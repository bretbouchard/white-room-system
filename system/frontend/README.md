# Frontend — Application Layer

The Swift application that brings together all White Room domains.

---

## What This Is

The frontend is the user-facing application that integrates:
- **Song** — Composition workspace with piano roll, step sequencer, Schillinger analysis
- **Ensemble** — Instrument selection, presets, deep editors, modulation
- **Mix** — 16-channel ConsoleX mixer with effect chains
- **DSP** — Sound Substrate via WhiteRoomKit (XCFramework or Pi 5 via WebSocket)
- **ML** — Core ML voice synthesis, composition assistance

---

## Technology Stack

| Technology | Purpose |
|------------|---------|
| **Swift** | Primary language |
| **SwiftUI** | Declarative UI framework |
| **Combine** | Reactive programming |
| **AVFoundation** | Audio I/O (local mode) |
| **AudioToolbox** | AUv3 hosting |
| **Core ML** | Voice synthesis inference |
| **WhiteRoomKit** | Engine access layer (Swift Package) |

---

## Room Architecture

The UI is organized into three areas. The user never leaves the song to access other tools.

```
+--------------------------------------------------+
|  SONG AREA (default, full screen)                |
|  +- Piano Roll / Step Sequencer / Staff / Drums  |
|  +- Schillinger Overlay Lenses                   |
|  +- Density Curves                               |
|  +- Ambient Info Bubbles                         |
|  +- Transport with Section Ruler                 |
|                                                   |
|  [Ensemble Panel]---- slides in from right        |
|  [Mix Panel]-------- slides in from right         |
+--------------------------------------------------+
```

### Song Area
The default landing view. Full workspace with:
- **Piano roll** (melodic tracks) and **step sequencer** (drum tracks), switchable via toggle
- **Staff notation** as a 4th view mode (Canvas-based, all platforms)
- **Drum pad performance surface** (MPC-style 4x4 grid with velocity and banks)
- **Schillinger overlay lenses** — color-coded overlays for Generators, Scale Degrees, Contour, Tension, Voice Leading
- **Density curves** — continuous lines at song/section/track levels with breakpoints and glitch-free changes
- **Ambient info bubbles** — contextual analysis insights surfaced inline
- **Transport with section ruler** — tick marks, labels, magnetic snap-to-section scrub
- **Song library** — blank songs, preset ensembles, templates, undo/redo

### Ensemble Panel
Slide-in panel for instrument management:
- Per-instrument **deep editors** with full parameter access
- Generalized **modulation matrix** (12x11) reusable across instruments
- LFOs (3), envelopes (3), modulation routing
- Instrument selection and preset browsing
- Sound Substrate editor shell as the template

### Mix Panel
Slide-in panel for mixing:
- 16-channel mixer with per-channel EQ, compression, gate, saturation
- ConsoleX bus processing (Airwindows Console)
- Effect pedal chains and ordering
- Gain, pan, mute, solo, send levels

---

## Platform Support

### Apple Ecosystem

| Platform | Layout | Notes |
|----------|--------|-------|
| **iPad** | Panels slide from right, Song compresses | Primary device |
| **iPhone** | Full-width sheet presentation | Adaptive layout |
| **Mac** | Resizable side panels (NavigationSplitView) | Native SwiftUI app |
| **Apple TV** | Transport-focused remote control (10-foot UI) | Siri Remote, no piano roll |
| **visionOS** | Standard Room Architecture | Spatial computing |

### The Apple TV Threshold

**Built for Apple TV -> Works everywhere.**

tvOS has no external synth or effect support. By targeting tvOS, White Room is self-contained across all platforms.

### Platform Adaptation

The `PlatformLayout` enum handles layout discrimination:

| Layout | Platform |
|--------|----------|
| `compactMobile` | iPhone portrait |
| `expandedMobile` | iPhone landscape |
| `tablet` | iPad |
| `desktop` | Mac native |
| `television` | Apple TV |

All Room Architecture views live in `SwiftFrontendShared` module. Platform adaptations use `PlatformAdaptor` protocol injection, not `#if os()` in view bodies.

---

## WhiteRoomKit Integration

The frontend consumes **WhiteRoomKit** as a Swift Package dependency. WhiteRoomKit is the engine access layer — it owns the `WhiteRoomEngine` protocol, all FFI declarations, and the model types that flow between the UI and the C++ engine.

**Key integration points:**
- All engine access goes through the `WhiteRoomEngine` protocol (never direct FFI calls from view code)
- `ModelerAPI` for sound modeler loading and control
- `SongModel` types come from WhiteRoomKit (no local copies in the frontend)
- Frontend imports WhiteRoomKit; it does not own or build it

---

## WhiteRoomEngine Protocol

The core abstraction for engine access, defined in WhiteRoomKit. This replaces the previous `AudioEngineBackend` as the single entry point for all engine operations.

```
WhiteRoomEngine (protocol, in WhiteRoomKit)
  +-- LiveEngine    (.live)  — FFI to C++ engine on device
  +-- MockEngine    (.mock)  — Deterministic testing, no FFI required
  +-- RemoteEngine  (.remote) — WebSocket to Pi 5 server
```

- Engine mode selection: `.live`, `.mock`, or `.remote` via protocol, not if/else
- `SongStateConverter` produces identical JSON for all three paths
- Transport push via WebSocket subscription (replaces 60fps FFI polling)
- Connection status UI shows latency, CPU, packet loss
- All 11 sub-APIs accessible through `WhiteRoomEngine`
- `MockEngine` provides deterministic testing without any C++ FFI dependency

---

## AppTheme Protocol

The visual design system, defined as a protocol in WhiteRoomKit with conformance in `ViewsDesign`.

### Protocol Location

- **Protocol** (`AppTheme`): WhiteRoomKit (engine layer owns the abstraction)
- **Conformance** (`WhiteRoomTheme`): ViewsDesign (UI layer owns the values)
- **Injection**: SwiftUI Environment at app root

### Semantic Properties

Views reference semantic properties, never hardcoded colors or values:

```swift
// Views reference:
.theme.accent          // not Color.pink
.theme.background      // not Color.black
.theme.textPrimary     // not Color.white
```

| Property Group | Properties |
|----------------|------------|
| **Accent** | `accent` |
| **Background hierarchy** | `background`, `backgroundSecondary`, `backgroundTertiary` |
| **Text hierarchy** | `textPrimary`, `textSecondary`, `textTertiary`, `textQuaternary` |
| **Card style** | `cardBackground`, `cardBorder`, `cardCornerRadius` |
| **Glass style** | `glassMaterial`, `glassOpacity`, `glassBorder` |
| **Typography** | `fontTitle`, `fontHeadline`, `fontBody`, `fontCaption` |
| **Spacing** | `spacingXS`, `spacingSM`, `spacingMD`, `spacingLG`, `spacingXL` |
| **Radius** | `radiusSM`, `radiusMD`, `radiusLG` |
| **Animation** | `animationFast`, `animationNormal`, `animationSlow` |
| **Visual character** | `visualCharacter` (pristine/aged/worn) |
| **Information density** | `informationDensity` (compact/normal/comfortable) |

### WhiteRoomTheme

The built-in conformance — a dark studio aesthetic with neon lab accents:

- Dark backgrounds with subtle layering (primary/secondary/tertiary)
- Electric pink accent (`#FF2D55`)
- Glass materials for overlays and panels
- Typography tuned for music production (high contrast at small sizes)

### Glass Material Forward Compatibility

The `glassMaterial` property returns `(any Sendable)?` — an opaque type that allows WhiteRoomKit to adopt iOS 26+ Liquid Glass when available, without requiring a minimum deployment target bump. Views consume this through the theme; the concrete material type is an implementation detail.

### InformationDensity

```swift
enum InformationDensity: String, CaseIterable {
    case compact      // Maximum content density (pro users, large displays)
    case normal       // Default balance
    case comfortable  // Relaxed spacing (accessibility, smaller screens)
}
```

Each case provides a localized display name via `localizedName`, suitable for settings UI.

---

## XCFramework Integration

The frontend consumes Sound Substrate as an XCFramework:

```
WhiteRoom.xcframework/
+-- ios-arm64/
+-- ios-arm64_x86_64-simulator/
+-- macos-arm64_x86_64/
+-- tvos-arm64/
+-- tvos-arm64_x86_64-simulator/
+-- visionos-arm64_x86_64-simulator/
```

### Module Structure

| Module | Purpose |
|--------|---------|
| `WhiteRoomKit` | Engine protocol, FFI declarations, model types (imported, not owned) |
| `SharedModels` | All data models (SongData, Preset, Instrument, etc.) |
| `SharedFFI` | @_silgen_name FFI declarations + bridge implementations |
| `SharedManagers` | Preset discovery, song management, services |
| `SharedAudio` | HouseBandActor, NativeAudioEngine |
| `ViewsDesign` | Design tokens, AppTheme conformance (WhiteRoomTheme), colors, typography, spacing, animations |
| `ViewsCore` | Platform adaptations, transitions, shared utilities |
| `ViewsControls` | Knobs, faders, mixer, waveform, strip views |
| `SharedViewModels` | State management (undo/redo, timeline, console) |
| `SwiftFrontendCore` | WhiteRoomCAPI, SchillingerGenerator |

---

## Architecture

### Layer Stack

```
+---------------------------------------------------------------+
|  SWIFTUI VIEWS (Room Architecture: Song / Ensemble / Mix)     |
+---------------------------+-----------------------------------+
                            |
+---------------------------v-----------------------------------+
|  VIEW MODELS (State, Business Logic, Coordination)           |
+---------------------------+-----------------------------------+
                            |
+---------------------------v-----------------------------------+
|  WHITEROOMKIT (Engine Access Layer)                          |
|  WhiteRoomEngine protocol | SongModel types | ModelerAPI    |
+---------------------------+-----------------------------------+
                            |
+---------------------------v-----------------------------------+
|  ENGINE IMPLEMENTATIONS                                       |
|  LiveEngine (FFI) | MockEngine (.mock) | RemoteEngine (WS)  |
+---------------------------+-----------------------------------+
                            |
+---------------------------v-----------------------------------+
|  C++ ENGINE (via XCFramework or Pi 5)                       |
+--------------------------------------------------------------+
```

### FFI Bridge

WhiteRoomKit's `LiveEngine` wraps all `@_silgen_name` declarations. The FFI surface is unchanged but is now accessed exclusively through the `WhiteRoomEngine` protocol, never directly from view code.

```
Swift (Views / ViewModels)
  -> WhiteRoomEngine protocol (in WhiteRoomKit)
    -> LiveEngine (wraps FFI)
      -> HouseBandActor (thread-safe access)
        -> wr_houseband_*() [C FFI]
          -> HouseBand [C++]
            -> ProjectionEngine -> SongBridge -> HouseBandBridgeEngine
              -> Audio Graph -> VoiceAllocator -> SynthesisModules
```

The FFI surface covers:
- `WR_HouseBand_*` — lifecycle, transport (play/pause/stop/seek)
- `WR_HouseBand_set_param_*` — parameter control (int/float/string)
- `WR_HouseBand_mixer_*` — mixer controls (gain/pan/mute/solo)
- `WR_HouseBand_member_*` — instrument/preset assignment
- `WR_HouseBand_schillinger_*` — Schillinger Books I-IV

`MockEngine` provides deterministic testing without any FFI dependency — views and view models can be fully tested without the C++ engine present.

---

## Schillinger Continuous Analysis

Analysis runs continuously as notes change:

- Dedicated worker thread pinned to core 0 or 1 (never audio cores)
- Results within 500ms of note change
- Lock-free MPSC queue to UI
- Color-coded overlay lenses on piano roll
- Bidirectional drum support (generate from theory + analyze against theory)
- Drum lock mode preserves locked hits during generation

---

## Song vs Performance Data Model

- **Song** = immutable DNA (notes, sections, Schillinger params)
- **Performance** = realization params (densities, presets, effects, voicing)
- Atomic swap at 5 levels: section, instrument, ensemble member, whole ensemble, full performance
- Share by reference: share Song DNA, recipients apply own Performance locally

---

## Strip Views

Instrument-grade channel strips with:
- ParameterMeta-driven design tokens
- InstrumentIcon protocol per instrument family
- VisualCharacter enum (pristine/aged/worn) for future character rendering
- StripBlock, Strip, StripValueControl components
- Cross-platform (no UIRectCorner, manual Path drawing)

---

## Real-Time Constraints

- UI thread never blocks audio
- All parameter changes de-zippered (smooth ramp)
- Atomic state changes
- CPU throttling awareness on mobile
- Thermal management for Core ML inference (adaptive rate scheduling)
