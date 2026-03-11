# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **System Atlas** — public architecture documentation for White Room, a private AI-powered music composition environment. It documents architecture, design decisions, patterns, and technology choices. The actual source code lives elsewhere.

## The Six Domains

White Room is organized into six interconnected domains:

```
FRONTEND (Application Layer)
    │ integrates all domains
    ▼
THEORY (Schillinger System)
    │ provides foundation for
    ▼
SONGWRITING (Creative Application)
    │ produces arrangements for
    ▼
MIXING (ConsoleX)
    │ routes to
    ▼
DSP (Sound Substrate)
    │ assisted by
    ▼
AI (ML Components)
```

### Frontend (`/frontend/`)
Swift application layer that integrates all domains:
- SwiftUI interface with tab-based navigation
- XCFramework consumption (Sound Substrate DSP)
- Platform support: iOS, macOS, tvOS, visionOS
- Audio session management
- MVVM architecture with Combine

### Theory (`/theory/`)
Mathematical foundation based on Joseph Schillinger's composition system (1946):
- **Rhythm**: Resultants, interference patterns, distribution
- **Melody**: Geometric pitch, contour, motivic development
- **Harmony**: Chord generation, voice leading, harmonic rhythm
- **Form**: Section types, energy curves, orchestration

### Songwriting (`/songwriting/`)
Creative application of theory:
- Song structure (AAA, AABA, Verse-Chorus)
- Hook construction using resultants
- Emotional arc design

### Mixing (`/mixing/`)
ConsoleX mixer architecture:
- Channel strips with EQ, dynamics, sends
- Group buses (Drums, Vocals, Instruments)
- Aux buses (Reverb, Delay, Chorus, Parallel Compression)
- Master bus with limiting and loudness metering
- House Band channels (7 default instruments)

### DSP (`/dsp/`)
Sound Substrate — the synthesis and audio processing engines:
- **DSP-first architecture**: Pure DSP layer, no JUCE dependencies
- **Synthesis engines**: Additive, Granular, Modal, Spectral, Chaos
- **Sample-based**: DDSP (Differentiable Digital Signal Processing) + open source samples
- **Effect processors**: Filter, Drive, Delay, Reverb, Chorus, Phaser, Compressor

### AI (`/ai/`)
Machine learning components:
- Composition Agent (generate rhythm/melody/harmony/bass)
- Analysis Agent (pitch/chord/key/tempo detection)
- Style Agent (learn and apply styles)
- Supervisor pattern for task routing

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
This enables consistent sound across all platforms with no vendor lock-in.

### Apple TV Threshold
**Built for Apple TV → Works everywhere.**

tvOS has no external synth/effect support. By targeting tvOS as the high water mark, White Room is self-contained and portable:
- Apple: iOS, macOS, tvOS, visionOS (via XCFramework)
- Desktop: Windows, Linux (VST3, CLAP, LV2)

### Real-Time Constraints
For DSP code:
- No allocation in audio thread
- No blocking operations (file I/O, network, mutex locks)
- Bounded execution within buffer period
- Performance budget: 4.5ms total per voice

## Instrument Canon

### Synthesizers (16)
| Instrument | Engine |
|------------|--------|
| NexSynth | FM (2/4/6-operator) |
| Kane | Wavetable/VA |
| Aether | Additive + Granular |
| LocalGal | Hybrid multi-engine |
| String | Additive |
| Breath / BreathLead | Physical model |
| Growl | Chaos + Additive |
| Choral / ChoirV2 | Spectral |
| Motion | Granular |
| Nature | Sample + Granular |
| Giants | Additive |
| MajorModulator | Modulation |
| CDC4046 | PLL synthesis |
| VoiceSynth | DDSP + ML (AI Voice Synthesis) |

### Samplers & Orchestral
- **SamSampler**: DDSP + Samples
- **Orchestral (12)**: Winds (5), Brass (4), Percussion (3) — all DDSP-enhanced
- **Keys (3)**: Piano, Rhodes, Rhodes3D
- **Drums (4)**: DrumMachine, DrumHybrid, DrumSamples, Drums

## Technology Stack

| Domain | Technologies |
|--------|-------------|
| Frontend | Swift, SwiftUI, Combine, XCFramework |
| Theory | Swift, Schillinger algorithms |
| Songwriting | Swift, NLP |
| Mixing | SwiftUI, Combine, AUv3 hosting |
| DSP | Pure C++20, DDSP, JUCE (optional wrapper), real-time audio |
| AI | Python, ML models, MCP agents |

## Document Conventions

- Architecture diagrams use ASCII art for portability
- Each domain has its own README.md in its subdirectory
- This repository contains **documentation only** — no implementation code
