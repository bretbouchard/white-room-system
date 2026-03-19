# Frontend — Application Layer

The Swift application that brings together all White Room domains.

---

## What This Is

The frontend is the user-facing application that integrates:
- **Theory** — Schillinger composition engine
- **Songwriting** — Creative tools and workflows
- **Mixing** — ConsoleX mixer interface
- **DSP** — Sound Substrate via XCFramework
- **ML** — Composition assistance

---

## Technology Stack

| Technology | Purpose |
|------------|---------|
| **Swift** | Primary language |
| **SwiftUI** | Declarative UI framework |
| **Combine** | Reactive programming |
| **AVFoundation** | Audio I/O |
| **AudioToolbox** | AUv3 hosting |

---

## Platform Support

### Apple Ecosystem

| Platform | Status | Notes |
|----------|--------|-------|
| **iOS** | ✅ Primary | iPhone, iPad |
| **macOS** | ✅ Primary | Mac (Apple Silicon + Intel) |
| **tvOS** | ✅ Primary | Apple TV (the high water mark) |
| **visionOS** | ✅ Supported | Apple Vision Pro |

### The Apple TV Threshold

**Built for Apple TV → Works everywhere.**

tvOS has no external synth or effect support — no AudioUnit hosting, no plugin loading. By targeting tvOS as our high water mark, White Room is self-contained:

- No dependencies on third-party plugins
- All synthesis and effects built-in
- Portable to any platform

---

## XCFramework Integration

The frontend consumes **Sound Substrate** as an XCFramework:

```
WhiteRoom.xcframework/
├── Info.plist
├── ios-arm64/
│   └── WhiteRoom.framework/
├── ios-arm64_x86_64-simulator/
│   └── WhiteRoom.framework/
├── macos-arm64_x86_64/
│   └── WhiteRoom.framework/
├── tvos-arm64/
│   └── WhiteRoom.framework/
├── tvos-arm64_x86_64-simulator/
│   └── WhiteRoom.framework/
└── visionos-arm64_x86_64-simulator/
    └── WhiteRoom.framework/
```

### Why XCFramework

| Benefit | Description |
|---------|-------------|
| **Single bundle** | All Apple platforms + simulators |
| **AUv3 extension** | Audio unit embedded in framework |
| **Code signing** | App Store ready |
| **Slice optimization** | Platform-specific binaries |
| **Versioning** | Semantic versioning support |

### Integration Pattern

```swift
import WhiteRoom

// DSP engine access
let engine = WhiteRoomEngine()

// Load instrument
engine.loadInstrument(.kane)

// Route through ConsoleX
mixer.connect(engine.output, to: channelStrip)
```

---

## Architecture

### Layer Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    SWIFTUI VIEWS                            │
│              (Screens, Controls, Visualizers)               │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                    VIEW MODELS                              │
│           (State, Business Logic, Coordination)             │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                    DOMAIN SERVICES                          │
│        Theory │ Songwriting │ Mixing │ ML                   │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                    WHITE ROOM XCFRAMEWORK                   │
│                    (Sound Substrate DSP)                    │
└─────────────────────────────────────────────────────────────┘
```

### MVVM Pattern

```
View (SwiftUI)
    │ binds to
    ▼
ViewModel (ObservableObject)
    │ communicates with
    ▼
Domain Services
    │ calls into
    ▼
XCFramework (DSP)
```

---

## UI Components

### Main Interface

```
┌─────────────────────────────────────────────────────────────┐
│  [Theory] [Song] [Mix] [DSP] [ML]          ◀ ▶ ● ▶▶ ⏹     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    MAIN CONTENT AREA                        │
│              (Current tab's interface)                      │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  Transport: BPM 120 │ Key: C Major │ Time: 4/4 │ CPU: 23%  │
└─────────────────────────────────────────────────────────────┘
```

### Tab Structure

| Tab | Content |
|-----|---------|
| **Theory** | Schillinger five-tab interface (Form, Rhythm, Melody, Harmony, Orchestration) |
| **Song** | Lyrics, structure, hooks, emotional arc |
| **Mix** | ConsoleX mixer UI with channel strips |
| **DSP** | Instrument selection, preset management |
| **ML** | Composition assistance, analysis results |

---

## Audio Session Management

### iOS/tvOS/visionOS

```swift
import AVFoundation

// Configure audio session
let session = AVAudioSession.sharedInstance()
try session.setCategory(.playback, mode: .default)
try session.setActive(true)

// Handle interruptions
NotificationCenter.default.addObserver(
    forName: AVAudioSession.interruptionNotification,
    object: session,
    queue: .main
) { notification in
    // Handle interruption
}
```

### macOS

- Direct Core Audio access
- No session management required
- Multi-output device support

---

## Performance Considerations

### UI Thread

- All UI updates on main thread
- DSP callbacks never block UI
- State changes are atomic

### Memory

- Lazy loading of instruments
- Asset caching with eviction
- Background prefetching

### Battery (Mobile)

- CPU throttling awareness
- Screen refresh optimization
- Background audio support

---

## Future Expansion

The frontend layer is designed to accommodate additional technologies:

| Future | Status |
|--------|--------|
| Web frontend (React/SwiftWasm) | Planned |
| Android frontend | Planned |
| Desktop native (Windows/Linux) | Planned |
