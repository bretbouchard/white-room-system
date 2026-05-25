# White Room System

Architecture documentation for **White Room** -- a theory-powered music composition environment.

---

## The Concept

Hundreds of years ago, composing began in silence.
You sat in a quiet room. Every tool within reach. A blank sheet of staff paper waiting for the first mark.

White Room is that room -- rebuilt for today.

A canvas with depth. Instruments, effects, and theory surrounding you.
Present when needed. Invisible when not.

<br>

<div align="center">
*In the white room with black curtains, near the station...*

<sub>Pete Brown, "White Room" (Cream, 1968)</sub>
</div>

<br>

---

## What This Is

This is a **System Atlas** -- public architecture documentation for a private codebase.

- **What's here**: Architecture, design decisions, patterns, technology choices
- **What's not here**: Source code, algorithms, presets, implementation details

### Why This Exists

I'm Bret Bouchard. I've been building White Room for 3+ years as a solo developer. This System Atlas documents the architecture for:

- **Hiring managers** -- See the engineering behind the product
- **Potential collaborators** -- Understand the architecture before conversations
- **Future me** -- Document decisions while I still remember why I made them

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

---

## [Song](./song/)
Your composition. Theory, engine, and craft combined.

- **[Theory](./song/theory/)** -- Schillinger System
  - Rhythm resultants and interference patterns
  - Melodic contour and motivic development
  - Harmonic progressions and voice leading
  - Orchestration and form

- **[Composition Engine](./song/engine/)** -- BettaFish-MiroFish Layer
  - Forum Engine: Multi-member deliberation (6 Musical Specialists) -- TypeScript
  - Simulation Engine: Temporal state evolution -- TypeScript
  - Counterpoint Engine: Koechlin voice-leading, species rules -- Python
  - Ensemble Members: Bass, Harmony, Lead, Counterline, Texture
  - Renderer/Realizer: Simulation to musical output

- **[Songwriting](./song/songwriting/)** -- Creative Application
  - Song structure and arrangement
  - Hook construction
  - Emotional arc design

## [Sound](./sound/)
Your instruments, effects, and mix. DSP, modelers, and routing combined.

- **[DSP](./sound/dsp/)** -- Sound Substrate
  - DSP-first architecture (minimal JUCE dependencies: 7 kept, 2 removed)
  - 30+ instruments across 16 synthesizer families, samplers, orchestral, keys, and drums
  - 28 effect pedals
  - 10+ synthesis engines: FM, Wavetable/VA, Additive, Granular, Physical Modeling, Spectral, DDSP, Chaos, Modal, Sample-based

- **[Modelers](./sound/modelers/)** -- ISoundModeler System
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

- **[Mixing](./sound/mixing/)** -- ConsoleX Mixer Architecture
  - 16-channel mixer with per-channel EQ, compression, gate, saturation
  - Airwindows Console bus processing
  - Bus routing (aux, groups, master)
  - SoundModelerChain integration (modelers in the signal path)
  - Effect chains and pedal ordering

## [System](./system/)
The packaging layer. Frontend, engine kit, network, and ML combined.

- **[WhiteRoomKit](./system/white_room_kit/)** -- Engine Package
  - Standalone Swift Package (zero FFI, zero SwiftUI)
  - SongModels, SongAlgorithm enrichment, StateVector (16D)
  - WhiteRoomEngine protocol with 11 sub-APIs
  - LiveEngine (.live FFI) and MockEngine (.mock testing)
  - AppTheme protocol (WhiteRoomTheme)
  - White Room's engine layer -- pure Swift, fully testable

- **[Frontend](./system/frontend/)** -- White Room Application Layer
  - SwiftUI interface across iOS, iPadOS, macOS, tvOS, visionOS
  - Room Architecture: 3-area navigation (Song / Ensemble / Mix)
  - WhiteRoomEngine protocol via WhiteRoomKit
  - XCFramework integration with C++ DSP engine

- **[Network](./system/network/)** -- Pi 5 Network Engine
  - Headless network-attached synth engine on Raspberry Pi 5
  - WebSocket control (JSON commands, state push, binary song transfer)
  - WebSocket mirrors WhiteRoomKit API 1:1 (zero feature gap)
  - Pipewire audio infrastructure (sole daemon)
  - Buildroot system image with PREEMPT_RT kernel
  - CPU isolation, thermal management, crash recovery

- **[ML](./system/ml/)** -- Machine Learning
  - Core ML voice synthesis (DDSP)
  - Composition assistance and style classification
  - Audio analysis and mix coaching

---

## Architecture

### Engine Architecture

White Room consumes its engine through WhiteRoomKit -- a standalone Swift Package that abstracts local FFI, remote WebSocket, and mock testing behind a single protocol.

```
+-------------------------------+
|  WHITE ROOM APP               |
|  WhiteRoomTheme (neon studio) |
|  Composition workspace        |
|  Room Architecture            |
+---------------+---------------+
                |
                +-------- WhiteRoomKit -----------+
                         |  SongModels            |
                         |  WhiteRoomEngine       |
                         |  SongAlgorithm         |
                         |  StateVector (16D)     |
                         |  DSPTypes              |
                         |  AppTheme protocol     |
                         +----+-------------+-----+
                              |             |
                     LiveEngine(.live)  MockEngine(.mock)
                         (FFI)          (testing)
```

### Dual-Engine Transport

White Room connects to the C++ engine through WhiteRoomKit's protocol layer. The WhiteRoomEngine protocol abstracts local vs remote -- the UI never knows which engine is active.

```
iPad / iPhone / Mac / Apple TV            Pi 5 / CM5
+-------------------------------+     +-------------------------------+
|  SwiftUI UI                   |     |  WebSocket Control Server     |
|  Room Architecture            |     |  (JSON commands + state push) |
|  ViewModels                   |<--->|  Protocol v1.0                |
|                               |     |                               |
|  WhiteRoomEngine protocol     |     |  White Room Engine (C++)      |
|   +-- LiveEngine (FFI) ------+--+  |  +---------------------------+|
|   +-- MockEngine (testing)   |  |  |  | Sound Substrate (C++20)  ||
|   +-- Remote (WebSocket) ----+  |  |  | HouseBand Engine         ||
|                               |  +->|  | 30+ Instruments          ||
|  SongStateConverter           |     |  | 28 Effect Pedals         ||
|  (produces identical JSON     |     |  | ISoundModeler System     ||
|   for all paths)              |     |  | 16ch Mixer + Console     ||
+-------------------------------+     |  +---------------------------+|
                                      |                               |
                                      |  Pipewire (sole audio infra)  |
                                      |  +- JACK client compat        |
                                      |  +- AES67/RTP network out     |
                                      |  +- ALSA MIDI bridge          |
                                      |  +- Graph-based routing       |
                                      |                               |
                                      |  Buildroot system image       |
                                      |  +- Read-only rootfs          |
                                      |  +- PREEMPT_RT kernel         |
                                      |  +- CPU isolation (cores 2,3) |
                                      |  +- A/B partition + OTA       |
                                      +-------------------------------+
```

### Room Architecture

The White Room UI is organized into three areas. Users never leave the song to access other tools.

```
+--------------------------------------------------+
|  SONG AREA (default, full screen)                |
|  +- Piano Roll / Step Sequencer / Staff / Drums  |
|  +- Schillinger Overlay Lenses                   |
|  +- Density Curves                               |
|  +- Ambient Info Bubbles                         |
|  +- Transport with Section Ruler                 |
|                                                   |
|  [Ensemble Panel <]---- slides in from right      |
|  [Mix Panel <]-------- slides in from right       |
+--------------------------------------------------+
```

- **Song**: Full workspace. Piano roll, step sequencer, staff notation, drum pads. Schillinger analysis runs continuously with color-coded overlay lenses.
- **Ensemble**: Slide-in panel. Instrument selection, presets, deep editors per instrument type, modulation matrix.
- **Mix**: Slide-in panel. 16-channel mixer with ConsoleX bus processing, effect chains, sound modeler chains, AI mix coaching.

Platform adaptations:
- **iPad**: Panels slide from right, Song compresses but stays visible
- **iPhone**: Full-width sheet presentation
- **Mac**: Resizable side panels
- **Apple TV**: Transport-focused remote control (10-foot UI, Siri Remote)

### Information Flow

```
User taps Play
  -> WhiteRoomEngine.transport.play()
    -> [LiveEngine] wr_houseband_play() via FFI -> HouseBand::play() [C++]
    -> [MockEngine] Deterministic stub -> test assertion
    -> [RemoteEngine] WebSocket command -> Pi server -> HouseBand::play() [C++]
        -> TransportState::start()
        -> ProjectionEngine::render()
          -> SongBridge -> HouseBandBridgeEngine
            -> AudioGraph::processBlock()
              -> VoiceAllocator::allocate()
              -> SynthesisModules::render()
            -> SoundModelerChain (AmpCapture -> CabinetIR -> Console)
            -> 16-channel mixer (EQ/comp/gate/sat + Console bus)
          -> Audio buffer
            -> [Local] AVAudioEngine -> Speakers
            -> [Remote] Pipewire -> Network audio (AES67/RTP)
```

---

## WhiteRoomKit

White Room's engine package -- a standalone Swift Package that provides the full engine abstraction layer.

**Zero FFI. Zero SwiftUI. Pure Swift.**

### Package Structure

| Module | Purpose |
|--------|---------|
| SongModels | Core data models (SongDNA, FullSong, PerformanceParams, FormModel, HarmonyModel, RhythmModel, PitchModel, OrchestrationModel) |
| SongAlgorithm | Enriched song data (MotifDNA, HarmonicField, RhythmicIdentity, SpectralProfile, SpatialIdentity, RoleMap, TransformationLog, TensionReleaseCurve, DependencyGraph) |
| WhiteRoomEngine | Root protocol with 11 sub-APIs (Transport, Mixer, Schillinger, Effects, Sampler, Sequencer, SongState, Pedalboard, Automation, DomainKnowledge, Modeler) |
| StateVector | 16D psychoacoustic state math (SIMD16<Float>) with NaN guards, clamped to [0,1], fully immutable |
| DSPTypes | Parameter metadata, pedal types, modeler kinds, CPU classifications |
| AppTheme | Protocol for app theming (WhiteRoomTheme) |

### Engine Implementations

- **LiveEngine** (`.live`) -- Real C++ backend via FFI. Production use.
- **MockEngine** (`.mock`) -- Deterministic stubs. Testing, previews, SwiftUI Previews.

Swap implementations without code changes.

### Design Rules

1. Zero FFI -- No `@_silgen_name` declarations
2. Zero SwiftUI -- No view imports. Engine-only.
3. Leaf Package -- No local package dependencies. Stands alone.
4. Backward Compatible -- All enrichment fields optional with sensible defaults
5. Codable Everything -- All models serialize/deserialize with version fields
6. Immutable Models -- StateVector and core models use `let` properties with copy-on-write semantics

---

## Instrument Canon

### Synthesizers (16)
Nexsynth (FM), Kane (Wavetable/VA), Aether (Additive+Granular), LocalGal (Hybrid), String (Additive), Breath/BreathLead (Physical model), Growl (Chaos+Additive), Choral/ChoirV2 (Spectral), Motion (Granular), Nature (Sample+Granular), Giants (Additive), MajorModulator (Modulation), CDC4046 (PLL), VoiceSynth (DDSP+ML)

### Samplers and Orchestral
SamSampler (DDSP+Samples), Orchestral (12 winds/brass/perc), Keys (Piano, Rhodes, Rhodes3D), Drums (4 variants)

### Effect Pedals (28)
Reverb, Delay, Chorus, Flanger, Phaser, EQ, Compression, Gate, Saturation, Distortion, and more -- all running through Airwindows Console bus processing.

---

## Sound Modeler System

### ISoundModeler Interface

Every modeler implements the same abstract interface, making them interchangeable at runtime:

```
ISoundModeler (abstract base)
  +- prepare()  process()  reset()
  +- latencySamples()  kind()  cpuClass()
```

### Modeler Types (10 Kinds)

| Kind | Modeler | Description |
|------|---------|-------------|
| AmpCapture | AmpCaptureModeler | Neural amp simulation via NAM Core v0.5.2 |
| CabinetIR | CabinetIRModeler | Speaker/microphone impulse response convolution |
| RoomIR | RoomIRModeler | Room/space convolution reverb with predelay |
| Console | ConsoleModeler | Analog console summing (encode/decode stages) |
| Tape | TapeModeler | Tape saturation + HF rolloff |
| Transformer | TransformerModeler | Preamp/output transformer character |
| CircuitFilter | CircuitFilterModeler | ZDF ladder, SVF, OTA analog filter models |
| PhysicalInstrument | PhysicalInstrumentModeler | Plucked strings, modal resonators, drum membranes |
| Performance | PerformanceModeler | Seeded deterministic timing/velocity control curves |
| NeuralFX | NeuralFXModeler | Generic neural FX via ONNX/RTNeural |

### SoundModelerChain

The chain processor manages ordered modeler instances:

```
Input --> [Modeler 1] --> [Modeler 2] --> [Modeler 3]
          AmpCapture     CabinetIR       RoomIR
          (NAM Core)     (Convolution)   (Convolution)
```

Key behaviors:
- **Ping-pong scratch buffers** prevent aliasing artifacts when 3+ modelers chain together
- **Single-modeler fast path** bypasses scratch buffers for efficiency
- **Per-modeler bypass** with zero-latency passthrough
- **Background loading** via atomic swap -- zero audio dropouts during hot-swap
- **Wet/dry mix** per modeler with delay-compensated dry signal

### CPU Classification

| Class | Description | Devices |
|-------|-------------|---------|
| Cheap | Runs everywhere, every channel | All |
| Moderate | Reasonable CPU, limited instances | All |
| High | Significant CPU, 1-2 instances | iOS/Mac/Pi (not tvOS) |
| Extreme | Maximum quality, desktop/Pi only | Mac/Pi only |

### Tone3000 Community Integration

In-app browsing of the Tone3000 NAM model and IR ecosystem. Downloaded captures auto-import with community tags and ratings. Cloud sync via iCloud KVS. Offline mode for previously downloaded captures.

### Metadata Catalog

Every modeler file gets a unified `ModelerMetadata` entry with auto-tagging from NAM JSON metadata, WAV/AIFF headers, filename analysis, and format detection. Inverted index supports sub-100ms search across 500+ captures.

---

## Pi 5 Network Engine

### Architecture

The Pi 5 runs as a headless network-attached synth engine. No screen, no keyboard, no DAC. Digital audio over Ethernet. Controlled from any Apple device.

```
Engine (C++) -> Pipewire client -> Pipewire graph
                                      |
                            +---------+----------+
                            |                    |
                      Network audio          Pipewire MIDI
                      (AES67/RTP)        (native ALSA bridge)
```

### Technical Specifications

| Parameter | Value |
|-----------|-------|
| Platform | Pi 5 / CM5 (BCM2712, Cortex-A76 quad-core @ 2.4GHz) |
| SIMD | NEON 128-bit |
| Voices | 256-512 (16 channels x 16-32 voices) |
| Mixer | 16-channel with per-channel EQ/comp/gate/sat + Console bus |
| Audio output | Network only (AES67/RTP via Pipewire) |
| Audio infrastructure | Pipewire (sole daemon) |
| Latency (one-way) | < 10ms over Ethernet |
| Control | WebSocket JSON + binary song transfer |
| MIDI | Pipewire ALSA MIDI bridge (USB controllers) |
| CPU isolation | Cores 2,3 for audio; cores 0,1 for system + network |
| Kernel | PREEMPT_RT (mainline 6.12+) |
| System image | Buildroot, read-only squashfs + tmpfs overlay |
| Boot time | < 8 seconds to audio-ready |
| Crash recovery | Hardware watchdog + systemd, restart within 5s |
| Deployment | Docker cross-compile -> scp -> systemd restart (< 60s cycle) |

### Cross-Compilation

```
macOS (development host)
+-------------------------------+
|  Docker container              |
|  (debian:bookworm-slim)        |
|  +- gcc-aarch64-linux-gnu      |
|  +- Debian Bookworm sysroot    |
|  +- CMake + ninja              |
|                                |
|  Output: white-room-engine     |
|  (aarch64 ELF binary)          |
+----------+--------------------+
           | scp / rsync
           v
+-------------------------------+
|  Pi 5 (target)                 |
|  +- Binary dropped in place    |
|  +- systemd restarts service   |
+-------------------------------+
```

### JUCE Dependency

The C++ engine retains 7 JUCE modules and removes 2:

| Module | Status | Reason |
|--------|--------|--------|
| `juce_audio_devices` | Removed | Replaced by Pipewire |
| `juce_audio_utils` | Removed | GUI/audio utilities not needed |
| `juce_audio_basics` | Kept | `AudioBuffer<float>`, `MidiBuffer` throughout DSP |
| `juce_core` | Kept | `String`, `CriticalSection`, threading, containers |
| `juce_dsp` | Kept | DSP module abstractions for effect pedals |
| `juce_audio_processors` | Kept | Base class for instrument hierarchy (25+ files) |
| `juce_data_structures` | Kept | `UndoManager`, `ValueTree` for state management |
| `juce_audio_formats` | Kept | `AudioFormatManager` for sample loading |
| `juce_events` | Kept | Message thread, timers |

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

## Platform Strategy

Designed under the strictest constraints.

White Room was engineered to run on platforms that do not allow external audio plugins (starting with Apple TV). This forced the system to contain its entire synthesis, effects, and composition engine internally.

That decision produced a portable architecture where every instrument and effect is part of the core engine rather than an external dependency.

The result is a system that runs consistently across:

**Apple platforms:**
iOS, iPadOS, macOS, tvOS, visionOS

**Desktop platforms:**
Windows, Linux

**Plugin formats:**
AUv3, VST3, CLAP, LV2

**Network engine:**
Raspberry Pi 5 / Compute Module 5

By removing external dependencies, White Room ensures consistent sound, deterministic playback, and seamless cross-platform portability.

---

## Component Catalog

### WhiteRoomKit (Swift Package)

| Component | Purpose |
|-----------|---------|
| WhiteRoomEngine | Root protocol with 11 sub-APIs (Transport, Mixer, Schillinger, Effects, Sampler, Sequencer, SongState, Pedalboard, Automation, DomainKnowledge, Modeler) |
| SongModels | Core data models (SongDNA, FullSong, PerformanceParams, FormModel, HarmonyModel, RhythmModel, PitchModel, OrchestrationModel) |
| SongAlgorithm | Enrichment fields (MotifDNA, HarmonicField, RhythmicIdentity, SpectralProfile, SpatialIdentity, RoleMap, etc.) |
| StateVector | 16D psychoacoustic state representation (SIMD16<Float>, immutable, NaN-guarded) |
| AppTheme | Protocol for app-level theming (WhiteRoomTheme) |
| LiveEngine | FFI-backed engine implementation for production use |
| MockEngine | Deterministic engine implementation for testing and previews |
| DSPTypes | Parameter metadata, pedal types, modeler kinds, CPU classifications |

### Sound Modeler System (C++)

| Component | Purpose |
|-----------|---------|
| ISoundModeler | Abstract interface for all modelers (prepare, process, reset, latency, kind, cpuClass) |
| SoundModelerChain | Ordered modeler processor with ping-pong scratch buffers |
| AmpCaptureModeler | Neural amp simulation (NAM Core v0.5.2) |
| CabinetIRModeler | Speaker/microphone impulse response convolution |
| RoomIRModeler | Room/space convolution reverb with predelay |
| ConsoleModeler | Analog console summing emulation (encode/decode) |
| TapeModeler | Tape saturation + HF rolloff |
| TransformerModeler | Preamp/output transformer character |
| CircuitFilterModeler | ZDF analog filter models (ladder, SVF, OTA) |
| PhysicalInstrumentModeler | Karplus-Strong+, modal resonators |
| PerformanceModeler | Seeded deterministic control curves |
| NeuralFXModeler | ONNX/RTNeural generic neural FX |

### C++ Engine Layer

| Component | Purpose |
|-----------|---------|
| HouseBandBridgeEngine | Unified DSP engine for all synthesis |
| AudioGraph | Dynamic voice allocation and audio processing graph |
| VoiceAllocator | Deterministic voice management (mono/poly/drum policies) |
| ParameterManager | 10,000+ parameters with automation, smoothing, modulation |
| ModulationMatrix | LFO/envelope/random to destination routing |
| ParameterRegistry | Canonical ParamID namespace (487 params) |
| PresetManager | Factory/user presets with validation and morphing |
| HouseBand | 16-channel mixer, transport, song playback, orchestration |
| SongBridge | HouseBand to HouseBandBridge wiring |
| ProjectionEngine | Projects song data into renderable audio graphs |

### C++ DSP Layer

| Component | Purpose |
|-----------|---------|
| AdditiveModule | Harmonic synthesis with per-partial control |
| GranularModule | Granular synthesis with grain scheduling |
| SpectralModule | FFT-based spectral synthesis |
| ModalModule | Physical modeling via resonator banks |
| ChaosModule | Strange attractor synthesis (Lorenz/Rossler/Chen) |
| HarmonicBank | Additive partial engine with deterministic phase |
| GrainVoice | Individual grain processing |
| ModalBank | Resonator bank for physical modeling |
| DeterministicRNG | Seed-based reproducible random generation |

### C API / FFI Layer

| Component | Purpose |
|-----------|---------|
| HouseBandFFI | C FFI for HouseBand (Swift integration) |
| HouseBandBridgeFFI | C FFI for HouseBandBridgeEngine |
| AudioThreadMonitor | Real-time audio thread health monitoring |
| WR_Modeler_* | Generic C API for sound modeler operations |

### Swift Frontend

| Component | Purpose |
|-----------|---------|
| SharedModels | All data models (SongData, Preset, Instrument, etc.) |
| SharedFFI | @_silgen_name declarations and bridge implementations |
| SharedManagers | Preset discovery, song management, services |
| SharedAudio | HouseBandActor, NativeAudioEngine, AudioEngineBackend |
| ViewsDesign | Design tokens (colors, typography, spacing, animations) |
| ViewsCore | Platform adaptations, transitions, shared utilities |
| ViewsControls | Knobs, faders, mixer, waveform, strip views |
| SharedViewModels | State management (undo/redo, timeline, console) |
| SwiftFrontendCore | WhiteRoomCAPI, SchillingerGenerator, composition integration |

### Voice Core ML Pipeline

| Component | Purpose |
|-----------|---------|
| VoiceModelCoreML | Core ML model loading and inference |
| VoiceInferenceManager | Manages inference lifecycle and scheduling |
| VoiceThermalManager | Adaptive inference rate based on thermal state |
| VoiceModelBridge | FFI bridge to C++ DDSP engine |
| VoiceModelEngine | C++ state machine with DSP fallback, overlap-add |
| VoiceModelCAPI | C API bridge for voice model engine |

---

## Testing

| Dimension | Coverage |
|-----------|----------|
| C++ core | >= 90% |
| Swift frontend | >= 85% |
| WhiteRoomKit | >= 80% (all models Codable round-trip, StateVector NaN guards) |
| Sound Modelers | >= 80% (golden render tests for every modeler) |
| FFI / C API / Transport | 100% |
| Total test count | 804+ |
| Test frameworks | Google Test (C++), XCTest + Swift Testing (Swift), RapidCheck (property-based) |
| Fuzz/stress | libFuzzer harnesses, AFL++ on Pi, thermal stress, RT safety guards |

---

## Real-Time Constraints

For DSP code, these rules are enforced:

- No allocation in audio thread
- No blocking operations
- Bounded execution within buffer period (4.5ms total per voice)
- `mlockall()` for audio thread memory
- CPU affinity pinning to isolated cores
- `SCHED_FIFO` real-time priority on Linux
- `ScopedNoAllocation` guard verifies zero heap allocations in audio callback
- Lock-free SPSC queues for MIDI and parameter updates

---

## Document Conventions

- Architecture diagrams use ASCII art for portability
- Each pillar has its own README.md
- This repository contains **documentation only** -- no implementation code

---

## License

Documentation only. All implementation code remains proprietary.
