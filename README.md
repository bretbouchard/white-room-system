# White Room System

Architecture documentation for **White Room** — a theory-powered music composition environment.

---

## The Concept

Hundreds of years ago, composing began in silence.
You sat in a quiet room. Every tool within reach. A blank sheet of staff paper waiting for the first mark.

White Room is that room — rebuilt for today.

A canvas with depth. Instruments, effects, and theory surrounding you.
Present when needed. Invisible when not.

<br>

<div align="center">
*In the white room with black curtains, near the station…*

<sub>Pete Brown, "White Room" (Cream, 1968)</sub>
</div>

<br>

---

## Project Status

**Working prototype, not yet released.**

White Room is built and running — [synthesis engines](./sound/dsp/), [Schillinger composition](./song/theory/), [multi-agent intelligence](./system/intelligence/), [cross-platform audio](./sound/dsp/).

**Current focus:** UI/UX Experience

---

## Three Sections

White Room is built on three sections: **Song**, **Sound**, and **System**.

### [Song](./song/)
Your composition. Theory and craft combined.

- **[Theory](./song/theory/)** — Schillinger System
  - [Rhythm resultants](./song/theory/) and interference patterns
  - [Melodic contour](./song/theory/) and motivic development
  - [Harmonic progressions](./song/theory/) and voice leading
  - Orchestration and form

- **[Songwriting](./song/songwriting/)** — Creative Application
  - Song structure and arrangement
  - Synths, Samplers, Orchestral, Keys, Drums
  - Hook construction
  - Emotional arc design

### [Sound](./sound/)
Your instruments and mix. DSP and routing combined.

- **[DSP](./sound/dsp/)** — Sound Substrate
  - [DSP-first architecture](./sound/dsp/) (no framework dependencies)
  - 27 [instruments](./sound/dsp/) across 5 categories, powered by 10+ synthesis engines
  - 15+ [effects](./sound/dsp/)

- **[Mixing](./sound/mixing/)** — [ConsoleX](./sound/mixing/) Mixer Architecture
  - [Bus routing](./sound/mixing/) (aux, groups, master)
  - Effect chains
  - [Channel strips](./sound/mixing/)

### [System](./system/)
The application layer. Frontend, Intelligence, and ML combined.

- **[Frontend](./system/frontend/)** — Application Layer
  - [SwiftUI](./system/frontend/) interface
  - [XCFramework](./system/frontend/) integration
  - Platform support (iOS, macOS, tvOS, visionOS)

- **[Intelligence](./system/intelligence/)** — [BettaFish-MiroFish](./system/intelligence/) Layer
  - [Forum Engine](./system/intelligence/): Multi-agent deliberation (6 specialist agents)
  - [Simulation Engine](./system/intelligence/): Temporal state evolution
  - [Musical Actor Agents](./system/intelligence/): Kick, Bass, Harmony, Lead, Texture
  - [Renderer/Realizer](./system/intelligence/): Simulation to musical output

- **[ML](./system/ml/)** — Machine Learning
  - [Composition assistance](./system/ml/)
  - [Audio analysis](./system/ml/)
  - [Style modeling](./system/ml/)

---

## Architecture Overview

```mermaid
flowchart LR
    subgraph SYSTEM["SYSTEM"]
        Frontend["Frontend<br/>SwiftUI"]
        Intelligence["Intelligence<br/>BettaFish-MiroFish"]
        ML["ML<br/>Python"]
        Frontend --> Intelligence
        Intelligence ~~~ ML
    end

    subgraph SONG["SONG"]
        Theory["Theory<br/>Schillinger"]
        Songwriting["Songwriting<br/>Creative"]
        Theory ~~~ Songwriting
    end

    subgraph SOUND["SOUND"]
        DSP["DSP<br/>C++20"]
        Mixing["Mixing<br/>ConsoleX"]
        DSP ~~~ Mixing
    end

    SYSTEM --> SONG --> SOUND
```

### [Intelligence Layer](./system/intelligence/) (BettaFish-MiroFish)

The Intelligence Layer sits between user intent and musical output:

```
User Intent
    ↓
Intent Interpretation
    ↓
Specialist Agents (6): Structure | Harmony | Rhythm | Motif | Emotion | Expression
    ↓
Forum Engine (BettaFish): Multi-agent deliberation → CompositionPlanIR
    ↓
Simulation Engine (MiroFish): Temporal state evolution → SimulationTimeline
    ↓
Musical Actor Agents (9): Kick | Bass | Harmony | Lead | Counterline | Texture
    ↓
Renderer/Realizer: Simulation → PatternIR/SongIR
    ↓
Audio Output
```

---

## What This Is

This is a **System Atlas** — public architecture documentation for a private codebase.

- **What's here**: Architecture, design decisions, patterns, technology choices
- **What's not here**: Source code, algorithms, presets, implementation details

### Why This Exists

I'm Bret Bouchard. I've been building White Room for 3+ years as a solo developer. This System Atlas demonstrates the depth and complexity of the project for:

- **Hiring managers** — See the engineering behind the product
- **Potential collaborators** — Understand the architecture before conversations
- **Future me** — Document decisions while I still remember why I made them

### What This Shows

If you're evaluating me as a candidate, this repo demonstrates:

| Skill | Evidence |
|-------|----------|
| **Systems Architecture** | 3-layer design (Song → Sound → System) with clean boundaries |
| **Real-time Audio** | Pure C++20 [DSP](./sound/dsp/), lock-free processing, cross-platform |
| **AI/Agent Systems** | [Multi-agent deliberation](./system/intelligence/), temporal simulation, explainable decisions |
| **Cross-Platform** | iOS, macOS, tvOS, visionOS, Windows, Linux, 4 plugin formats |
| **Music Theory** | Full [Schillinger System](./song/theory/) implementation |
| **Documentation** | Comprehensive architecture docs, decision logs |

The private codebase is available for review under NDA.

---

## Technology Stack

| Section | Layer | Technologies | Purpose |
|---------|-------|--------------|---------|
| [Song](./song/) | [Theory](./song/theory/) | Swift, Schillinger algorithms | Mathematical composition |
| Song | [Songwriting](./song/songwriting/) | Swift | Creative application |
| [Sound](./sound/) | [DSP](./sound/dsp/) | Pure C++20, DDSP, real-time audio | Synthesis engines |
| Sound | [Mixing](./sound/mixing/) | SwiftUI, Combine, AUv3 hosting | ConsoleX mixer |
| [System](./system/) | [Frontend](./system/frontend/) | Swift, SwiftUI, Combine, XCFramework | Application layer |
| System | [Intelligence](./system/intelligence/) | TypeScript, Zod, Node.js, Vitest | Multi-agent composition |
| System | [ML](./system/ml/) | Python, ML models | Optional assistance |

---

## Platform Strategy

Designed under the strictest constraints.

White Room was engineered to run on platforms that do not allow external audio plugins. This forced the system to contain its entire synthesis, effects, and composition engine internally.

That decision produced a portable architecture where every instrument and effect is part of the core engine rather than an external dependency.

The result is a system that runs consistently across:

**Apple platforms:** iOS, macOS, tvOS, visionOS
**Desktop platforms:** Windows, Linux
**Mobile platforms:** iOS, iPadOS

**Plugin formats supported:**

AUv3 • VST3 • CLAP • LV2

By removing external dependencies, White Room ensures consistent sound, deterministic playback, and seamless cross-platform portability.

---

## License

Documentation only. All implementation code remains proprietary.
