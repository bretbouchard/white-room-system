# White Room System

Architecture documentation for **White Room** — a theory-powered music composition environment.

---

## The Concept

Hundreds of years ago, composing began in silence.
You sat in a quiet room. Every tool within reach. A blank sheet of staff paper waiting for the first mark.

White Room is that room — rebuilt for today.

A canvas with depth. Instruments, effects, and theory surrounding you.
Present when needed. Invisible when not.

*In the white room with black curtains, near the station…*

---

## Three Pillars

White Room is built on three pillars: **Song**, **Sound**, and **System**.

### [Song](./song/)
Your composition. Theory and craft combined.

- **[Theory](./song/theory/)** — Schillinger System
  - Rhythm resultants and interference patterns
  - Melodic contour and motivic development
  - Harmonic progressions and voice leading
  - Orchestration and form

- **[Songwriting](./song/songwriting/)** — Creative Application
  - Song structure and arrangement
  - Hook construction
  - Emotional arc design

### [Sound](./sound/)
Your instruments and mix. DSP and routing combined.

- **[DSP](./sound/dsp/)** — Sound Substrate
  - DSP-first architecture (no framework dependencies)
  - Synthesizer engines (Additive, Granular, Modal, Spectral, Chaos)
  - Sample-based instruments with DDSP
  - Cross-platform: iOS, macOS, tvOS, visionOS, Windows, Linux

- **[Mixing](./sound/mixing/)** — ConsoleX
  - Mixer architecture
  - Bus routing (aux, groups, master)
  - Effect chains
  - Channel strips

### [System](./system/)
The application layer. Frontend and ML combined.

- **[Frontend](./system/frontend/)** — Application Layer
  - SwiftUI interface
  - XCFramework integration
  - Platform support (iOS, macOS, tvOS, visionOS)

- **[ML](./system/ml/)** — Machine Learning
  - Composition assistance
  - Audio analysis
  - Style modeling

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                         SYSTEM                               │
│              Frontend + ML (Application Layer)               │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                          SONG                                │
│              Theory + Songwriting (Composition)              │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                          SOUND                               │
│              DSP + Mixing (Audio Engine)                     │
└─────────────────────────────────────────────────────────────┘
```

---

## What This Is

This is a **System Atlas** — public architecture documentation for a private codebase.

- **What's here**: Architecture, design decisions, patterns, technology choices
- **What's not here**: Source code, algorithms, presets, implementation details

---

## Technology Stack

| Pillar | Layer | Technologies |
|--------|-------|-------------|
| Song | Theory | Swift, Schillinger algorithms |
| Song | Songwriting | Swift |
| Sound | DSP | Pure C++20, DDSP, real-time audio |
| Sound | Mixing | SwiftUI, Combine, AUv3 hosting |
| System | Frontend | Swift, SwiftUI, Combine, XCFramework |
| System | ML | Python, ML models |

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
