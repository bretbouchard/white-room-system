# System

The packaging layer. Engine, application, network, and ML combined.

---

## What This Is

The System pillar is everything that makes White Room into a shipping app:

- **WhiteRoomKit** — White Room's engine package (zero FFI, zero SwiftUI)
- **Frontend** — White Room application (SwiftUI across all Apple platforms)
- **Network** — Pi 5 network-attached synth engine
- **ML** — Machine learning (voice synthesis, composition assistance)

---

## Layers

### [WhiteRoomKit](./white_room_kit/) — White Room's Engine Package

Standalone Swift Package with zero FFI and zero SwiftUI dependencies. White Room's engine package.

- **SongModels** — Core data models for songs, instruments, presets, Schillinger parameters
- **SongAlgorithm** — Enrichment layer: models gain computed properties, validations, and transformations
- **StateVector** — 16-dimensional psychoacoustic math (valence, energy, density, brightness, etc.)
- **WhiteRoomEngine protocol** — 11 sub-APIs:
  - Transport, Mixer, Schillinger, Effects, Sampler, Sequencer
  - SongState, Pedalboard, Automation, DomainKnowledge, Modeler
- **LiveEngine** (.live) — Real engine via FFI to C++ Sound Substrate
- **MockEngine** (.mock) — Deterministic testing engine

### [Frontend](./frontend/) — White Room Application Layer

The White Room application that brings together all domains:

- SwiftUI interface across iOS, iPadOS, macOS, tvOS, visionOS
- Room Architecture: 3-area navigation (Song / Ensemble / Mix)
- WhiteRoomEngine protocol via WhiteRoomKit
- AppTheme protocol with WhiteRoomTheme (neon studio aesthetic)
- AudioEngineBackend protocol: local FFI or remote WebSocket
- XCFramework integration with C++ DSP engine

### [Network](./network/) — Pi 5 Network Engine

Headless network-attached synth engine:

- Raspberry Pi 5 running the full C++ audio engine
- WebSocket control server mirrors WhiteRoomKit API 1:1
- JSON commands + state push + binary song transfer
- Pipewire audio infrastructure (sole daemon)
- Buildroot system image with PREEMPT_RT kernel, CPU isolation, crash recovery

### [ML](./ml/) — Machine Learning

ML-powered features:

- Core ML voice synthesis (DDSP) with timbre XY pad and voice presets
- Composition assistance and style classification
- Audio analysis and mix coaching

---

## Relationship

```
WHITEROOMKIT         FRONTEND             NETWORK            ML
────────────         ────────             ───────            ──
Engine package       White Room app       Pi 5 synth engine  Core ML voice
SongModels           SwiftUI interface    WebSocket control  Composition assist
SongAlgorithm        Room Architecture    Mirrors Kit API    Audio analysis
StateVector 16D      WhiteRoomTheme       Pipewire audio     Style modeling
Engine protocol      5 Apple platforms    Buildroot appliance
LiveEngine + Mock    XCFramework
```

WhiteRoomKit is the foundation. Frontend is the face. Network is the remote brain. ML is the enhancement.

---

## Architecture

```
+---------------------------------------------------------------+
|                    APPLICATION LAYER                           |
|                                                               |
|  +-----------------------------------------------------------+|
|  |  WHITE ROOM (Frontend)                                     ||
|  |  SwiftUI + Room Architecture                               ||
|  |  WhiteRoomTheme (neon studio aesthetic)                    ||
|  |  Song / Ensemble / Mix                                     ||
|  |  5 Apple platforms                                         ||
|  +----------------------------+-------------------------------+|
                               |                               +-------------------------------+
                   +-----------v-----------+                   |  PI 5 (Network Engine)        |
                   |  WHITEROOMKIT         |                   |                               |
                   |  Swift Package        |                   |  WebSocket Control Server     |
                   |  Zero FFI/SwiftUI     |                   |  (mirrors WhiteRoomKit 1:1)   |
                   |                       |                   |                               |
                   |  SongModels           |        WiFi/      |  White Room Engine (C++)      |
                   |  SongAlgorithm        |<------->Ethernet  |  +-- Sound Substrate (C++20)  |
                   |  StateVector (16D)    |                   |  +-- HouseBand Engine         |
                   |                       |                   |  +-- 30+ Instruments          |
                   |  WhiteRoomEngine      |                   |  +-- 28 Effect Pedals         |
                   |  +-- LiveEngine (FFI) |                   |  +-- 16ch Mixer + Console     |
                   |  +-- MockEngine (test)|                   |                               |
                   +-----------+-----------+                   |  Pipewire (sole audio infra)  |
                               |                               |  Buildroot system image       |
                   +-----------v-----------+                   +-------------------------------+
                   |  C++ ENGINE            |
                   |  Sound Substrate       |
                   |  HouseBand Engine      |
                   |  30+ Instruments       |
                   |  28 Effect Pedals      |
                   |  16ch Mixer + Console  |
                   +------------------------+
```

### WhiteRoomEngine Protocol

The core abstraction wrapping local and remote engines:

```
WhiteRoomEngine (protocol) — 11 sub-APIs
  Transport | Mixer | Schillinger | Effects | Sampler
  Sequencer | SongState | Pedalboard | Automation
  DomainKnowledge | Modeler
    +-- LiveEngine (.live) — FFI to C++ Sound Substrate
    +-- MockEngine (.mock) — Deterministic testing
```

White Room uses the WhiteRoomEngine protocol from WhiteRoomKit. LiveEngine talks to the C++ engine via FFI on-device, or the same commands travel over WebSocket to the Pi 5. The UI never knows which engine is active.

### App Theming

White Room provides its visual identity through AppTheme:

```
AppTheme (protocol)
  +-- WhiteRoomTheme — Neon studio aesthetic (White Room)
```
