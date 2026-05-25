# WhiteRoomKit — Engine Package

The Swift Package that powers White Room's engine layer.

---

## What This Is

WhiteRoomKit is a standalone Swift Package extracted from White Room's shared layer. It contains the song model types, engine protocol API, and DSP type definitions. Any target that imports WhiteRoomKit gets identical sounds, controls, and features through a protocol-based API surface.

**Zero FFI. Zero SwiftUI. Pure Swift.**

---

## Why It Exists

WhiteRoomKit separates the engine layer from the UI layer:

- Song models, engine protocols, and DSP types live in a clean, testable package
- The main app, AUv3 extension, and WebSocket server all consume the same API
- MockEngine enables deterministic testing and SwiftUI previews without the C++ backend

A leaf Swift Package with no circular dependencies, no FFI declarations, and no UI imports.

---

## Architecture

```
+---------------------------------------------------------------+
|  WHITE ROOM APP                                                |
|  (neon studio theme, AppTheme conformance)                     |
|                                                                |
|  +------------------------+                                   |
|  | App-Layer Code         |                                   |
|  | (FFI, views, assets)   |                                   |
|  +------------+-----------+                                   |
|               |                                                |
|  +------------v-----------------+                             |
|  |   WhiteRoomKit Swift Package |                             |
|  |                              |                             |
|  |  +-- SongModels              |                             |
|  |  +-- SongAlgorithm           |                             |
|  |  +-- WhiteRoomEngine protocol|                             |
|  |  +-- StateVector (16D)       |                             |
|  |  +-- DSPTypes                |                             |
|  +------+-----------------------+                             |
|         |                                                      |
|  +------v----------+  +------------------+                   |
|  | LiveEngine (.live)|  | MockEngine (.mock)|                   |
|  | (FFI to C++ DSP) |  | (deterministic)   |                   |
|  +------------------+  +------------------+                   |
+---------------------------------------------------------------+
```

---

## Package Structure

### SongModels

Core data models:

| Model | Purpose |
|-------|---------|
| `SongDNA` | Immutable song composition data |
| `FullSong` | Complete song with all sections, tracks, notes |
| `PerformanceParams` | Realization parameters (densities, presets, voicing) |
| `FormModel` | Section structure (intro, verse, chorus, bridge, outro) |
| `HarmonyModel` | Key, scale, chord progressions |
| `RhythmModel` | Time signature, tempo, density |
| `PitchModel` | Scale, contour, melodic patterns |
| `OrchestrationModel` | Instrument assignment across 16 channels |

### SongAlgorithm

Enriched song data with 9 computed fields:

| Field | Purpose |
|-------|---------|
| `MotifDNA` | Recurring melodic/rhythmic motifs with identity |
| `HarmonicField` | Tonal center, modality, tension profile |
| `RhythmicIdentity` | Subdivision, syncopation, clave alignment |
| `SpectralProfile` | Frequency distribution, brightness, warmth |
| `SpatialIdentity` | Stereo width, depth, imaging |
| `RoleMap` | Instrument roles (bass, lead, pad, texture, etc.) |
| `TransformationLog` | Schillinger operations that produced the song |
| `TensionReleaseCurve` | Harmonic tension over time |
| `DependencyGraph` | Inter-track rhythmic/harmonic dependencies |

All enrichment fields are optional and backward compatible. Existing songs without enrichment decode with sensible defaults.

### WhiteRoomEngine Protocol

The universal engine API — import WhiteRoomKit and control audio through this protocol:

```
WhiteRoomEngine (root protocol)
  +-- TransportAPI         (play, pause, stop, seek, position)
  +-- MixerAPI             (gain, pan, mute, solo per channel)
  +-- SchillingerAPI       (Books I-IV, resultants, grooves)
  +-- EffectsAPI           (pedal chains, parameters)
  +-- SamplerAPI           (note on/off, instrument selection)
  +-- SequencerAPI         (song structure, sections, tracks)
  +-- SongStateAPI         (load song, switch performance, duration)
  +-- PedalboardAPI        (pedal add/remove/reorder)
  +-- AutomationAPI        (parameter automation curves)
  +-- DomainKnowledgeAPI   (song analysis, structure info)
  +-- ModelerAPI           (sound modeler loading, bypass)
```

Two implementations:
- **LiveEngine** (`.live`) — Real C++ backend via FFI. Production use.
- **MockEngine** (`.mock`) — Deterministic stubs. Testing, previews, SwiftUI Previews.

Swap implementations without code changes.

### StateVector

16-dimensional psychoacoustic state representation:

```
StateVector (SIMD16<Float>)
  +- energy              (overall intensity)
  +- density             (note density)
  +- harmonicStability   (tonal stability)
  +- rhythmicStability   (groove consistency)
  +- brightness          (high frequency content)
  +- tension             (harmonic tension)
  +- spatialWidth        (stereo spread)
  +- spatialDepth        (reverb, distance)
  +- modalStability      (key adherence)
  +- syncopation         (off-beat emphasis)
  +- claveAlignment      (rhythmic pattern match)
  +- subdivision         (beat division complexity)
  +- articulation        (legato vs staccato)
  +- timbralComplexity   (harmonic richness)
  +- dynamicRange        (loudness variation)
  +- phraseLength        (phrase duration factor)
```

All values clamped to [0,1] with NaN guards. Fully immutable — `withDimension` returns new instances.

### DSPTypes

Parameter types, pedal types, and audio abstractions:

- `ParameterMeta` — Parameter metadata (range, skew, unit)
- `PedalType` — All 28+ effect pedal types
- `ModelerKind` — Sound modeler types (10 kinds)
- `CPUClass` — Modeler CPU requirements (cheap/moderate/high/extreme)

---

## Design Rules

1. **Zero FFI** — No `@_silgen_name` declarations. Verified by grep gate.
2. **Zero SwiftUI** — No view imports. Engine-only.
3. **Leaf Package** — No local package dependencies. Stands alone.
4. **Backward Compatible** — Re-export stubs via `@_exported import` for transparent migration.
5. **Codable Everything** — All models serialize/deserialize via Codable with version fields.
6. **Immutable Models** — StateVector and core models use `let` properties with copy-on-write semantics.

---

## Integration

### White Room App

```swift
import WhiteRoomKit

// Engine access via LiveEngine (FFI-backed)
let engine: WhiteRoomEngine = LiveEngine(houseBand: houseBandPtr)
engine.transport.play()
engine.mixer.setGain(channel: 0, gain: 0.8)
```

### AUv3 Extension

```swift
import WhiteRoomKit

// Same API in the extension
let engine: WhiteRoomEngine = LiveEngine(houseBand: houseBandPtr)
engine.sampler.noteOn(note: 60, velocity: 100, channel: 0)
```

### Testing

```swift
import WhiteRoomKit

let engine: WhiteRoomEngine = MockEngine()
engine.transport.play() // Deterministic, no FFI needed
XCTAssertEqual(engine.transport.isPlaying, true)
```

---

## Export Pipeline

White Room exports enriched SongAlgorithm as JSON:

- **iOS/tvOS**: App Group shared container
- **macOS**: Application Support directory
- **File**: Direct JSON file export/import

---

## Technology

| Aspect | Choice |
|--------|--------|
| Language | Swift 5.9+ |
| Package Manager | Swift Package Manager |
| Dependencies | Foundation only |
| Testing | XCTest + Swift Testing |
| Concurrency | Sendable, @MainActor, OSAllocatedUnfairLock |
| SIMD | SIMD16<Float> for state vector |

---

## Related Documentation

- [Frontend](../frontend/) — White Room app layer that consumes WhiteRoomKit
- [DSP](../../sound/dsp/) — C++ engine that LiveEngine wraps via FFI
- [Modelers](../../sound/modelers/) — Sound modeler system accessed via ModelerAPI
