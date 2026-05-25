# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **System Atlas** — public architecture documentation for White Room, a theory-powered music composition environment. It documents architecture, design decisions, patterns, and technology choices. The actual source code lives elsewhere.

---

## Three Pillars

White Room is organized into three pillars: **Song**, **Sound**, and **System**.

### Conceptual Flow

```
SYSTEM (Packaging)
    |
    v
SONG (Composition)
    |
    v
SOUND (Audio)
```

### Actual Data Flow

```
Swift Frontend
    |
    +-- WhiteRoomEngine protocol (11 sub-APIs)
    |       |
    |       +-- LiveEngine (.live) -- FFI --> C++20 DSP
    |       +-- MockEngine (.mock) -- Deterministic stubs
    |       +-- RemoteEngine -- WebSocket --> Pi 5 Engine
    |
    +-- FFI --> BettaFish-MiroFish (TS/Python)
    |                  |
    |                  +-- Forum Engine (TypeScript)
    |                  +-- Simulation Engine (TypeScript)
    |                  +-- Counterpoint Engine (Python/Koechlin)
    |                  |
    |                  +-- Triggers --> DSP
    |
    +-- FFI --> DSP (C++20) <-- Direct access
```

### Song (`/song/`)
Your composition. Theory, engine, and craft combined.

- **Theory** (`/song/theory/`) — Schillinger System
  - Rhythm resultants, interference patterns
  - Melodic contour, motivic development
  - Harmony, voice leading, form

- **Engine** (`/song/engine/`) — BettaFish-MiroFish Layer
  - Forum Engine: Multi-member deliberation (6 Musical Specialists) — TypeScript
  - Simulation Engine: Temporal state evolution — TypeScript
  - Counterpoint Engine: Koechlin voice-leading, species rules — Python
  - Ensemble Members: Bass, Harmony, Lead, Counterline, Texture
  - Renderer/Realizer: Simulation to musical output

- **Songwriting** (`/song/songwriting/`) — Creative Application
  - Song structure, arrangement
  - Hook construction
  - Emotional arc design

### Sound (`/sound/`)
Your instruments, effects, and mix. DSP, modelers, and routing combined.

- **DSP** (`/sound/dsp/`) — Sound Substrate
  - DSP-first architecture (minimal JUCE dependencies: 7 kept, 2 removed)
  - 30+ instruments across 16 synthesizer families, samplers, orchestral, keys, drums
  - 28 effect pedals
  - 10+ synthesis engines: FM, Wavetable/VA, Additive, Granular, Modal, Spectral, DDSP, Chaos, Physical Model, PLL
  - Cross-platform: iOS, macOS, tvOS, visionOS, Windows, Linux, Pi 5

- **Modelers** (`/sound/modelers/`) — ISoundModeler System
  - Generic modeler framework (ISoundModeler interface, 10 modeler kinds)
  - AmpCaptureModeler (NAM Core v0.5.2 neural amp simulation)
  - CabinetIR, RoomIR (partitioned convolution)
  - ConsoleModeler (analog summing encode/decode)
  - TapeModeler, TransformerModeler (analog color)
  - CircuitFilterModeler (ZDF analog filters)
  - PhysicalInstrumentModeler (Karplus-Strong+, modal resonators)
  - PerformanceModeler (seeded control curves)
  - NeuralFXModeler (ONNX/RTNeural)
  - Tone3000 community integration, metadata catalog, preview/comparison

- **Mixing** (`/sound/mixing/`) — ConsoleX
  - 16-channel mixer with per-channel EQ, compression, gate, saturation
  - Airwindows Console bus processing
  - Bus routing (aux, groups, master)
  - SoundModelerChain integration (modelers in the signal path)
  - De-zippered parameters (10ms ramp default)

### System (`/system/`)
The packaging layer. Frontend, engine kit, network, and ML combined.

- **WhiteRoomKit** (`/system/white_room_kit/`) — Engine Package
  - Standalone Swift Package (zero FFI, zero SwiftUI)
  - SongModels, SongAlgorithm enrichment, StateVector (16D)
  - WhiteRoomEngine protocol with 11 sub-APIs
  - LiveEngine (.live FFI) and MockEngine (.mock testing)
  - WhiteRoomKit is the engine layer for White Room

- **Frontend** (`/system/frontend/`) — White Room Application Layer
  - SwiftUI across iOS, iPadOS, macOS, tvOS, visionOS
  - Room Architecture: 3-area navigation (Song / Ensemble / Mix)
  - WhiteRoomEngine protocol via WhiteRoomKit
  - AppTheme protocol with WhiteRoomTheme (neon studio)
  - XCFramework integration with C++ DSP engine

- **Network** (`/system/network/`) — Pi 5 Network Engine
  - Headless synth engine on Raspberry Pi 5
  - WebSocket control (JSON + binary song transfer)
  - WebSocket mirrors WhiteRoomKit API 1:1 (zero feature gap)
  - Pipewire audio infrastructure
  - Buildroot system image with PREEMPT_RT kernel

- **ML** (`/system/ml/`) — Machine Learning
  - Core ML voice synthesis (DDSP) with 5 bundled models
  - Composition assistance and style classification
  - Audio analysis and mix coaching

---

## Key Architectural Decisions

### WhiteRoomKit — Engine Package

White Room consumes the engine through WhiteRoomKit:

```
+-------------------------------+
|  WHITE ROOM APP               |
|  WhiteRoomTheme (neon studio) |
|  Composition workspace        |
+---------------+---------------+
                |
                +-------- WhiteRoomKit
                         |  SongModels
                         |  WhiteRoomEngine
                         |  StateVector
                         |  DSPTypes
                         +----+-------------+
                              |             |
                     LiveEngine(.live)  MockEngine(.mock)
                         (FFI)          (testing)
```

The engine implementation can be swapped without code changes.

### WhiteRoomEngine Protocol

Swift connects to the C++ engine through WhiteRoomKit's protocol layer:

```
+-------------------------------+         +-------------------------------+
|  SWIFT APP                    |         |  ENGINE                       |
|  WhiteRoomEngine protocol     |         |                               |
|   +-- LiveEngine (.live) ---+--+      |  C++20 Sound Substrate       |
|   +-- MockEngine (.mock) ---+   |      |  HouseBand Engine             |
|   +-- RemoteEngine (WS) ----+   |      |  30+ Instruments              |
|                               |         |  28 Effect Pedals             |
|  SongStateConverter           |         |  ISoundModeler System         |
|  (identical JSON all paths)   |         |  16ch Mixer + Console         |
+-------------------------------+         +-------------------------------+
```

All paths produce identical results. The UI never knows which engine is active.

### AppTheme Protocol

White Room skins itself via protocol, not hardcoded colors:

```
AppTheme (protocol in WhiteRoomKit)
  +-- WhiteRoomTheme  (neon studio: electric pink, dark backgrounds)
```

Injected via SwiftUI Environment at app root. Views reference `.theme.accent`, not `WhiteRoomColors.electricPink`.

### Room Architecture

3-area navigation in White Room:

- **Song**: Full workspace (piano roll, step sequencer, staff, drums, Schillinger lenses, density curves)
- **Ensemble**: Slide-in panel (instrument deep editors, modulation matrix, presets)
- **Mix**: Slide-in panel (16-channel mixer, effect chains, ConsoleX)

### DSP-First Architecture

The audio engine is abstracted from platform APIs:

```
Application Layer (SwiftUI/Native UI)
         |
Sound Substrate (DSP Layer — minimal JUCE: 7 kept, 2 removed)
         |
ISoundModeler Layer (NAM, Convolution, Console, Tape, etc.)
         |
Platform Layer (AUv3 | VST3 | CLAP | LV2 | Standalone | Pipewire)
```

### Apple TV Threshold

**Built for Apple TV -> Works everywhere.**

tvOS has no external synth/effect support. By targeting tvOS, White Room is self-contained:
- Apple: iOS, macOS, tvOS, visionOS (via XCFramework)
- Desktop: Windows, Linux (VST3, CLAP, LV2)
- Network: Pi 5 (Pipewire)

### Real-Time Constraints

For DSP code:
- No allocation in audio thread
- No blocking operations
- Bounded execution within buffer period
- Performance budget: 4.5ms total per voice
- `ScopedNoAllocation` guard verifies zero heap allocations
- Lock-free SPSC queues for MIDI and parameter updates

---

## Instrument Canon

### Synthesizers (16)
Nexsynth (FM), Kane (Wavetable/VA), Aether (Additive+Granular), LocalGal (Hybrid), String (Additive), Breath/BreathLead (Physical model), Growl (Chaos+Additive), Choral/ChoirV2 (Spectral), Motion (Granular), Nature (Sample+Granular), Giants (Additive), MajorModulator (Modulation), CDC4046 (PLL), VoiceSynth (DDSP+ML)

### Samplers & Orchestral
SamSampler (DDSP+Samples), Orchestral (12 winds/brass/perc), Keys (Piano, Rhodes, Rhodes3D), Drums (4 variants)

### Effect Pedals (28)
28 effect pedals across time-based, modulation, dynamics, tonal, distortion, and utility categories.

### Sound Modelers (10 Kinds)
AmpCapture (NAM), CabinetIR, RoomIR, Console, Tape, Transformer, CircuitFilter, PhysicalInstrument, Performance, NeuralFX

---

## Technology Stack

| Pillar | Layer | Technologies |
|--------|-------|-------------|
| Song | Theory | Swift, Schillinger algorithms |
| Song | Engine (Forum/Sim) | TypeScript, Zod, Vitest |
| Song | Engine (Counterpoint) | Python, Koechlin system |
| Song | Songwriting | Swift |
| Sound | DSP | Pure C++20, DDSP, Core ML, real-time audio |
| Sound | Modelers | C++20, NAM Core, partitioned convolution, ONNX/RTNeural |
| Sound | Mixing | SwiftUI, Combine, AUv3 hosting |
| System | WhiteRoomKit | Swift Package, zero FFI, zero SwiftUI |
| System | Frontend | Swift, SwiftUI, Combine, XCFramework |
| System | Network | C++ (libwebsockets), Pipewire, Buildroot |
| System | ML | Core ML, Python, DDSP voice models |

---

## Document Conventions

- Architecture diagrams use ASCII art for portability
- Each pillar has its own README.md
- This repository contains **documentation only** — no implementation code
