# White Room System

Architecture documentation for **White Room** — an AI-powered music composition environment.

This repository documents the four core domains of White Room:

---

## The Four Domains

### [Theory](./theory/) — Schillinger System
The mathematical foundation for composition.
- Rhythm resultants and interference patterns
- Melodic contour and motivic development
- Harmonic progressions and voice leading
- Orchestration and form

### [Mixing](./mixing/) — ConsoleX
The mixing and signal routing infrastructure.
- ConsoleX mixer architecture
- Bus routing (aux, groups, master)
- Instrument channels
- Effect chains

### [DSP](./dsp/) — Sound Substrate
The synthesis and audio processing engines.
- Synthesizer engines (Additive, Granular, Modal, Spectral, Chaos)
- Voice architecture
- Effect processors
- Real-time audio constraints

### [AI](./ai/) — Machine Learning
The ML and intelligence components.
- Composition assistance
- Audio analysis
- Style modeling
- Agent orchestration

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                         THEORY                               │
│                    Schillinger System                        │
│              (Form, Rhythm, Melody, Harmony)                 │
└─────────────────────────┬───────────────────────────────────┘
                          │ generates
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
                          │ processes
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
| Theory | Swift, Schillinger algorithms |
| Mixing | SwiftUI, Combine, AUv3 hosting |
| DSP | JUCE, C++20, real-time audio |
| AI | Python, ML models, MCP agents |

---

## License

Documentation only. All implementation code remains proprietary.
