# White Room System

Architecture documentation for **White Room** — an AI-powered music composition environment.

This repository documents the six core domains of White Room:

---

## The Six Domains

### [Theory](./theory/) — Schillinger System
The mathematical foundation for composition.
- Rhythm resultants and interference patterns
- Melodic contour and motivic development
- Harmonic progressions and voice leading
- Orchestration and form

### [Songwriting](./songwriting/) — Creative Application
The craft of turning theory into songs.
- Lyric writing and phrasing
- Song structure and arrangement
- Hook construction
- Emotional arc design

### [Mixing](./mixing/) — ConsoleX
The mixing and signal routing infrastructure.
- ConsoleX mixer architecture
- Bus routing (aux, groups, master)
- Instrument channels
- Effect chains

### [DSP](./dsp/) — Sound Substrate
The synthesis and audio processing engines.
- **DSP-first architecture** — Not tied to JUCE or framework dependencies
- Synthesizer engines (Additive, Granular, Modal, Spectral, Chaos)
- Sample-based instruments with DDSP (Piano, Orchestral)
- Voice architecture and effect processors
- Cross-platform: iOS, macOS, **tvOS**, visionOS, Windows, Linux

### [AI](./ai/) — Machine Learning
The ML and intelligence components.
- Composition assistance
- Audio analysis
- Style modeling
- Agent orchestration

### [Frontend](./frontend/) — Application Layer
The Swift application that brings everything together.
- SwiftUI interface
- XCFramework integration
- Platform support (iOS, macOS, tvOS, visionOS)
- Future expansion (Web, Android, Desktop)

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                       FRONTEND                               │
│                    Application Layer                         │
│              (SwiftUI, XCFramework, Platform UI)             │
└─────────────────────────┬───────────────────────────────────┘
                          │ integrates all domains
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                         THEORY                               │
│                    Schillinger System                        │
│              (Form, Rhythm, Melody, Harmony)                 │
└─────────────────────────┬───────────────────────────────────┘
                          │ provides foundation for
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                      SONGWRITING                             │
│                    Creative Application                      │
│              (Lyrics, Hooks, Structure, Emotion)             │
└─────────────────────────┬───────────────────────────────────┘
                          │ produces arrangements for
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                         MIXING                               │
│                        ConsoleX                              │
│              (Channels, Buses, Routing)                      │
└─────────────────────────┬───────────────────────────────────┘
                          │ routes to
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                          DSP                                 │
│                    Sound Substrate                           │
│              (Synths, Effects, Voice)                        │
└─────────────────────────┬───────────────────────────────────┘
                          │ assisted by
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                           AI                                 │
│                    ML Components                             │
│              (Analysis, Assistance, Agents)                  │
└─────────────────────────────────────────────────────────────┘
```

---

## What This Is

This is a **System Atlas** — public architecture documentation for a private codebase.

- **What's here**: Architecture, design decisions, patterns, technology choices
- **What's not here**: Source code, algorithms, presets, implementation details

---

## Technology Stack

| Domain | Technologies |
|--------|-------------|
| Frontend | Swift, SwiftUI, Combine, XCFramework |
| Theory | Swift, Schillinger algorithms |
| Songwriting | Swift, natural language processing |
| Mixing | SwiftUI, Combine, AUv3 hosting |
| DSP | Pure C++20, DDSP, JUCE (optional wrapper), real-time audio |
| AI | Python, ML models, MCP agents |

---

## Platform Strategy

**Built for Apple TV → Works everywhere.**

Apple TV has no external synth or effect support. By targeting tvOS as our high water mark, we ensured White Room is self-contained and portable across all platforms:

- **Apple Ecosystem**: iOS, macOS, tvOS, visionOS (AUv3)
- **Desktop**: Windows, Linux (VST3, CLAP, LV2)
- **Mobile**: iOS, iPadOS (AUv3, Standalone)

---

## License

Documentation only. All implementation code remains proprietary.
