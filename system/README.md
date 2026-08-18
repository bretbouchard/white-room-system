# System

The packaging layer. Engine, application, network, and ML combined.

---

## What This Is

The System pillar is everything that makes White Room into a shipping app or a
near-term shipping system:

- **WhiteRoomKit** — White Room's engine package (zero FFI, zero SwiftUI)
- **Frontend** — White Room application (SwiftUI across current Apple targets)
- **Network** — Pi 5 network-attached synth engine architecture and implementation work
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

- SwiftUI interface across iOS, iPadOS, macOS, and tvOS
- Room Architecture: 3-area navigation (Song / Ensemble / Mix)
- WhiteRoomEngine protocol via WhiteRoomKit
- AppTheme protocol with WhiteRoomTheme (neon studio aesthetic)
- AudioEngineBackend protocol: local FFI or remote WebSocket
- XCFramework integration with C++ DSP engine

Distribution note as of August 5, 2026:

- iOS, tvOS, and macOS builds ship through one shared App Store Connect record
- VoiceSynth AUv3 is distributed through host apps, not as a standalone TestFlight app
- the current macOS app does not yet embed the VoiceSynth AUv3 extension

### [Network](./network/) — Pi 5 Network Engine

Headless network-attached synth engine effort:

- Raspberry Pi 5 running the full C++ audio engine
- Real remote-engine, discovery, and protocol code exists in the repo
- JSON commands + state push + binary song transfer are defined in the architecture
- Pipewire audio infrastructure and Buildroot image work are active implementation areas
- Physical qualification and release hardening are still ongoing

### [ML](./ml/) — Machine Learning

ML-powered features:

- Core ML voice synthesis (DDSP) with timbre XY pad and voice presets
- Composition assistance and style analysis
- Mostly on-device processing, with some cloud-facing code paths also present in the repo

---

## Relationship

```
WHITEROOMKIT         FRONTEND             NETWORK            ML
────────────         ────────             ───────            ──
Engine package       White Room app       Pi 5 synth engine  Core ML voice
SongModels           SwiftUI interface    WebSocket control  Composition assist
SongAlgorithm        Room Architecture    Mirrors Kit API    Audio analysis
StateVector 16D      WhiteRoomTheme       Pipewire audio     Style modeling
Engine protocol      4 Apple platforms    Buildroot appliance
LiveEngine + Mock    XCFramework
```

WhiteRoomKit is the foundation. Frontend is the face. Network is the remote brain. ML is the enhancement.

## Beta Invites

To request TestFlight access, email `bretbouchard@gmail.com` and include:

- target platform: `iOS/iPadOS`, `tvOS`, or `macOS`
- your TestFlight Apple ID email
- device model and OS version
- preferred AUv3 host app if you want to test VoiceSynth

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
|  |  iOS / iPadOS / macOS / tvOS                               ||
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
