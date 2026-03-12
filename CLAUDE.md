# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **System Atlas** — public architecture documentation for White Room, a theory-powered music composition environment. It documents architecture, design decisions, patterns, and technology choices. The actual source code lives elsewhere.

---

## Three Pillars

White Room is organized into three pillars: **Song**, **Sound**, and **System**.

```
SYSTEM(Frontend + ML)
    │
    ▼
SONG (Theory + Songwriting)
    │
    ▼
SOUND (DSP + Mixing)
```

### Song (`/song/`)
Your composition. Theory and craft combined.

- **Theory** (`/song/theory/`) — Schillinger System
  - Rhythm resultants, interference patterns
  - Melodic contour, motivic development
  - Harmony, voice leading, form

- **Songwriting** (`/song/songwriting/`) — Creative Application
  - Song structure, arrangement
  - Hook construction
  - Emotional arc design

### Sound (`/sound/`)
Your instruments and mix. DSP and routing combined.

- **DSP** (`/sound/dsp/`) — Sound Substrate
  - DSP-first architecture (no framework dependencies)
  - Synthesis engines: Additive, Granular, Modal, Spectral, Chaos
  - Sample-based instruments with DDSP
  - Cross-platform: iOS, macOS, tvOS, visionOS, Windows, Linux

- **Mixing** (`/sound/mixing/`) — ConsoleX
  - Mixer architecture
  - Bus routing (aux, groups, master)
  - Channel strips, effect chains

### System (`/system/`)
The application layer. Frontend and ML combined.

- **Frontend** (`/system/frontend/`) — Application Layer
  - SwiftUI interface
  - XCFramework integration
  - Platform support (iOS, macOS, tvOS, visionOS)

- **ML** (`/system/ml/`) — Machine Learning
  - Composition assistance
  - Audio analysis
  - Style modeling

---

## Key Architectural Decisions

### DSP-First Architecture
The audio engine is abstracted from platform APIs:
```
Application Layer (SwiftUI/Native UI)
         │
Sound Substrate (Pure DSP Layer — No JUCE dependencies)
         │
Platform Layer (AUv3 | VST3 | CLAP | LV2 | Standalone)
```

### Apple TV Threshold
**Built for Apple TV → Works everywhere.**

tvOS has no external synth/effect support. By targeting tvOS, White Room is self-contained:
- Apple: iOS, macOS, tvOS, visionOS (via XCFramework)
- Desktop: Windows, Linux (VST3, CLAP, LV2)

### Real-Time Constraints
For DSP code:
- No allocation in audio thread
- No blocking operations
- Bounded execution within buffer period
- Performance budget: 4.5ms total per voice

---

## Instrument Canon

### Synthesizers (16)
NexSynth (FM), Kane (Wavetable/VA), Aether (Additive+Granular), LocalGal (Hybrid), String (Additive), Breath/BreathLead (Physical model), Growl (Chaos+Additive), Choral/ChoirV2 (Spectral), Motion (Granular), Nature (Sample+Granular), Giants (Additive), MajorModulator (Modulation), CDC4046 (PLL), VoiceSynth (DDSP+ML)

### Samplers & Orchestral
SamSampler (DDSP+Samples), Orchestral (12 winds/brass/perc), Keys (Piano, Rhodes, Rhodes3D), Drums (4 variants)

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

## Document Conventions

- Architecture diagrams use ASCII art for portability
- Each pillar has its own README.md
- This repository contains **documentation only** — no implementation code
