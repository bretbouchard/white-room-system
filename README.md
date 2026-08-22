# White Room System

**A theory-powered music composition environment with a real-time C++ audio engine, native Apple application stack, local ML, and governed AI operating against authoritative musical state.**

White Room is a music-writing and performance system built around a simple idea: the musician should stay inside the song while composition, theory, instruments, effects, mixing, and intelligent assistance remain available around them.

This repository is the public **System Atlas** for a private implementation. It documents the architecture, engineering boundaries, technology choices, and current product direction without publishing proprietary source code.

---

## Why this system is interesting

White Room is not an AI chat interface wrapped around a DAW.

It combines several systems that normally live separately:

- structured song/composition state
- mathematical music-theory tooling
- real-time synthesis and DSP
- instruments, effects, modelers, and mixing
- Swift / SwiftUI application architecture
- C++20 real-time audio
- a controlled Swift ↔ C++ FFI boundary
- Core ML and local inference
- multi-platform Apple UI
- local and remote engine implementations
- governed model/tool interaction

The AI layer must work against the same real product state and engine contracts as the rest of the application.

---

## Three pillars

White Room is organized into **Song**, **Sound**, and **System**.

```text
SYSTEM
application, engine abstraction, ML, networking
        |
        v
SONG
composition, theory, form, musical state
        |
        v
SOUND
instruments, effects, modelers, mixer, audio
```

### [Song](./song/)

The composition layer.

- Schillinger-derived theory and reactive analysis
- piano roll, step sequencer, staff, and drum workflows
- typed Song / Performance separation
- density, form, harmony, rhythm, motif, and orchestration operations
- preservation of user-authored vs generated material
- governed AI assistance through structured composition tools

### [Sound](./sound/)

The real-time audio layer.

- C++20 Sound Substrate
- 30+ instruments across multiple synthesis families
- 28 effect pedals
- physical, modal, spectral, granular, FM, additive, wavetable/VA, sample, neural and other synthesis/modeling paths
- ISoundModeler abstraction for amp, cabinet, room, console, tape, transformer, filter, physical-instrument, performance and neural processing
- 16-channel mixer with Console-style bus processing

### [System](./system/)

The application and integration layer.

- Swift / SwiftUI app architecture
- WhiteRoomKit engine abstraction
- Live, Mock, and Remote engine paths
- C++ FFI and WebSocket transports behind stable application contracts
- Core ML / DDSP voice pipeline
- local intelligence and Apple Foundation Models direction
- iOS, iPadOS, macOS, tvOS and related Apple-platform work

---

## Current engine boundary

The UI and AI layers do not directly own the DSP engine.

```text
Swift Frontend
     |
     v
WhiteRoomEngine protocol
     |
 +---+-------------------+
 |                       |
 v                       v
MockEngine            LiveEngine
 deterministic           |
 tests/previews           v
                    Swift/C++ FFI
                          |
                          v
                    C++20 Engine
                          |
                          v
               instruments / DSP / mixer
```

A RemoteEngine path uses the same conceptual application boundary while transporting commands/state over the network.

This separation makes UI, composition and AI work testable without putting model or application concerns inside the real-time render callback.

---

## Governed intelligence

White Room applies [GSA — Governed Stewardship Architecture](https://github.com/bretbouchard/gsa-system) in a deliberately lightweight creative form.

The central rule is:

> **The Song is authoritative. The model is a processor.**

The intended AI runtime combines:

1. **A reasoning model** — Apple Foundation Models is the primary local Apple path; model implementations remain replaceable.
2. **Music-domain expertise** — learned adapters and structured domain knowledge.
3. **The current Song** — authoritative live project context.
4. **White Room tools** — bounded executable capabilities for composition, performance, sound, mix and analysis.

```text
User intent
    |
    v
Current Song / project state
    |
    | controlled context
    v
Model + music-domain reasoning
    |
    | structured tool request
    v
White Room tool
    |
    v
Validation / engine contract
    |
    v
Accepted state mutation
    |
    v
Playable result
```

The model does not replace the Song document, silently become long-term application memory, or bypass executable contracts.

See [System / Intelligence](./system/intelligence/) for the detailed architecture.

---

## Example AI-assisted change

A request such as:

> **Make the chorus denser without changing its harmony.**

can become a constrained system operation:

```text
resolve chorus from current Song
        |
preserve harmony as constraint
        |
inspect density, instrumentation and note origins
        |
propose bounded generated-event changes
        |
apply through composition tools
        |
validate event integrity + preserved constraints
        |
accept mutation
        |
play through WhiteRoomEngine
```

If generated events are missing, duplicated, outside scope, or violate the requested harmonic constraint, the operation fails validation and is revised rather than accepted because the model sounded confident.

---

## WhiteRoomKit

WhiteRoomKit is the Swift-side engine abstraction.

Its purpose is to give the application one typed interface to musical state and executable engine capabilities while allowing the implementation behind it to change.

Key characteristics include:

- typed Song and performance models
- `WhiteRoomEngine` protocol with domain-specific APIs
- deterministic `MockEngine` for tests and previews
- `LiveEngine` for the production C++ path
- remote transport behind the same conceptual contract
- serializable/versioned state models
- separation between application models and real-time implementation details

This is a major reason AI assistance can operate against normal application capabilities rather than becoming a special privileged execution path.

---

## Real-time audio engineering

The Sound layer is built around hard real-time constraints.

Important rules include:

- no heap allocation on the audio thread
- no blocking work in the render callback
- bounded execution per audio buffer
- lock-free queues for time-critical control/event flow
- expensive inference and planning stay outside real-time rendering
- safe fallback paths where ML/model resources are unavailable

The system includes a broad instrument and effect estate, but those private implementations are intentionally not reproduced in this public repository.

See [Sound / DSP](./sound/dsp/) and [Sound / Modelers](./sound/modelers/) for the architectural atlas.

---

## Local ML and VoiceSynth

White Room contains a real-time voice/ML path built around Core ML and C++ DDSP processing.

The documented architecture includes:

```text
text / phoneme conditioning
        |
Core ML model pipeline
        |
Swift inference scheduling
        |
controlled FFI
        |
C++ DDSP / render buffers
        |
AudioGraph
```

The design includes DSP fallback when model loading fails, thermal-aware inference scheduling, cached/offline rendering paths, and render-thread isolation from ML work.

See [System / ML](./system/ml/) for the detailed pipeline.

---

## Testing philosophy

White Room treats generated behavior as product behavior.

Testing is therefore not limited to whether an LLM returns syntactically valid text. Depending on the subsystem, tests can assert things such as:

- expected musical events are present
- notes are not dropped or duplicated
- operations stay within requested scope
- manual material remains preserved
- serialization round-trips retain identity
- engine protocols behave consistently across mock/live boundaries
- DSP obeys real-time constraints
- model-loading failures degrade safely
- UI and end-to-end product flows continue to work

The goal is **no vibes-based acceptance**: where the system can know what correct behavior is, it should test it.

---

## What this public atlas demonstrates

For an engineer evaluating the project, White Room is an example of integrating AI into a large stateful native application rather than building an isolated model demo.

It spans:

| Area | Representative architecture |
|---|---|
| Agentic AI | governed context + explicit tools + authoritative state |
| Native application | Swift, SwiftUI, typed application models |
| Systems integration | Swift ↔ C++ FFI, protocol abstraction |
| Real-time systems | C++20 DSP, lock-free audio/control boundaries |
| ML | Core ML, DDSP, local inference and fallbacks |
| Music domain | theory, composition, instruments, effects, mixer |
| Testing | deterministic mocks, music-specific assertions, E2E flows |
| Networking | remote engine/WebSocket architecture |
| Platforms | iOS/iPadOS/macOS/tvOS-oriented architecture |

---

## Public vs private

This repository intentionally contains architecture documentation rather than the production implementation.

Public here:

- system boundaries
- design rationale
- technology choices
- data/execution flow
- integration patterns
- current vs planned status where relevant

Kept private:

- production source code
- proprietary DSP implementations and presets
- internal training data
- private prompts/configuration
- credentials
- unpublished product state

---

## Related projects

### [GSA — Governed Stewardship Architecture](https://github.com/bretbouchard/gsa-system)
The general architecture used to keep models, state, evidence, capabilities and side effects separate.

### [Volta System](https://github.com/bretbouchard/volta-system)
The same governed-system ideas applied to electronics design, where requirements and deterministic verification must survive all the way to manufacturing.

---

## Status

White Room is under active development. This atlas should describe current implementation honestly and label experimental/planned capabilities rather than presenting them as shipped.
