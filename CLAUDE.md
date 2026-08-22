# CLAUDE.md

This repository is the public **System Atlas** for White Room, a theory-powered music composition and performance environment. It documents architecture, design decisions, integration boundaries, and technology choices. The private implementation lives elsewhere.

## Core public rule

Keep this atlas aligned with the current system. Do not resurrect retired experimental architectures or present research concepts as shipping product architecture.

White Room uses [GSA — Governed Stewardship Architecture](https://github.com/bretbouchard/gsa-system) in a lightweight creative form:

- authoritative Song and engine state live outside models
- models receive controlled project context
- models act through explicit White Room tools
- accepted changes are validated before becoming authoritative state
- models are replaceable processors, not the product database or execution authority

## Three pillars

### Song
Composition, theory, songwriting, and structured musical operations.

- Schillinger-derived analysis and generation
- typed Song and Performance state
- theory-aware composition operations
- governed AI assistance through White Room tools
- manual/generated note distinction and preservation

### Sound
Real-time instruments, effects, modelers, mixer, and routing.

- C++20 Sound Substrate
- 30+ instruments
- 28 effect pedals
- 10 sound-modeler kinds
- ConsoleX mixing architecture
- hard real-time constraints and lock-free communication

### System
Packaging and integration layer.

- Swift / SwiftUI application
- WhiteRoomKit engine abstraction
- Live, Mock, and Remote engine paths
- controlled Swift/C++ FFI boundary
- Core ML and local intelligence
- Apple-platform and network-engine support

## Current data flow

```text
User / UI / governed AI
          |
          v
Authoritative Song + Performance State
          |
          v
WhiteRoomEngine protocol
     |          |          |
   Mock       Live       Remote
              |            |
             FFI        WebSocket
              |            |
              +---- C++20 engine ----+
                       |
                       v
                 DSP / Mixer / Audio
```

AI-driven composition follows the same product contracts:

```text
Current Song -> model/domain reasoning -> White Room tool -> validation -> accepted Song mutation
```

AI logic must not be placed in the real-time audio callback.

## WhiteRoomKit

WhiteRoomKit is the Swift engine abstraction layer. It keeps application code independent of the active engine implementation and makes deterministic testing possible.

Key ideas:

- typed Song models
- `WhiteRoomEngine` root protocol with domain APIs
- `LiveEngine` for production FFI
- `MockEngine` for deterministic tests and previews
- remote engine transport behind the same conceptual contract
- serialization/versioning boundaries

## Intelligence

Current public intelligence documentation lives in `system/intelligence/README.md`.

The intended AI runtime combines:

1. a capable local/reasoning model, with Apple Foundation Models as the primary Apple path
2. music-domain expertise/adapters
3. the authoritative current Song object
4. explicit White Room tools

Do not describe a model as directly owning or replacing Song state.

## Real-time constraints

For DSP-facing code and documentation, preserve these principles:

- no allocation in the audio thread
- no blocking operations
- bounded work per buffer period
- lock-free communication for time-critical data
- model inference and expensive planning stay off the render callback
- failures degrade safely where a deterministic DSP fallback exists

## Public documentation rules

- distinguish shipped/current capability from active development and plans
- prefer concrete architecture and verified implementation facts over speculative claims
- do not expose proprietary implementation code, private prompts, credentials, private datasets, or unpublished product details
- link to GSA where the general governed-agent architecture is relevant rather than re-explaining the full governance stack in every White Room document
- keep White Room focused on making music; do not turn the public architecture into project-management UI
