# Composition Engine

White Room's composition engine turns musical intent into bounded changes against the current Song rather than asking an AI model to regenerate an opaque song document.

The engine combines typed musical state, theory-aware analysis, deterministic composition operations, domain knowledge, and governed AI assistance.

---

## Core Rule

The **Song is authoritative**.

Models and algorithms may inspect it and propose changes, but accepted changes must flow through structured operations and preserve declared constraints.

```text
User intent
    |
    v
Current Song + selected scope
    |
    v
Analysis / theory / model reasoning
    |
    v
Structured composition operations
    |
    v
Validation
    |
    v
Accepted Song mutation
    |
    v
WhiteRoomEngine playback
```

---

## Composition Inputs

The engine can reason from:

- current notes and timing
- song form and section boundaries
- harmony and key/scale context
- rhythm and density
- motifs and transformations
- orchestration and track roles
- manually authored vs generated material
- instrument capabilities
- user constraints and musical preferences

The current Song supplies live project truth. The model does not need to infer the project from conversation history.

---

## Theory-Aware Operations

White Room's composition layer is built around explicit musical operations rather than unrestricted text generation.

Examples include:

- rhythmic generation and transformation
- density changes within song/section/track scope
- motif development and variation
- harmonic analysis and constrained reharmonization
- counterpoint and voice-leading operations
- orchestration changes
- section and form operations
- generated-note replacement while preserving manual material

Schillinger-derived tools provide mathematical structure for rhythm, melody, harmony, orchestration, and form. Other music-theory and learned-domain methods can participate through the same state/tool boundary.

---

## Governed AI Assistance

AI assistance follows the White Room intelligence architecture documented in [System / Intelligence](../../system/intelligence/).

```text
Model
  |
  v
bounded tool request
  |
  v
composition operation
  |
  v
constraint + event validation
  |
  v
Song state
```

A request such as:

> Make the chorus denser without changing its harmony.

becomes a scoped operation:

1. Resolve the chorus from current Song state.
2. Preserve harmonic content as a constraint.
3. Inspect current rhythmic density, voices, instrumentation, and note origins.
4. Propose additional/modified generated material only where allowed.
5. Validate timing, event identity, voice limits, and the harmonic constraint.
6. Commit only the accepted change.

---

## Manual Material Is Protected

White Room distinguishes user-authored material from generated material.

This allows operations such as density changes or regeneration to modify machine-generated content while preserving notes the musician intentionally placed.

That distinction is central to making AI assistance collaborative rather than destructive.

---

## Determinism and Testing

Where an operation can be deterministic, White Room makes it deterministic and seedable.

Tests can verify:

- expected notes are produced
- required notes are not dropped
- notes are not duplicated unexpectedly
- event timing remains in bounds
- transformations stay within the selected scope
- preserved harmonic/form constraints remain true
- serialization round-trips retain musical identity
- mock and live engine paths agree at their contracts

Generated output is therefore subject to the same engineering expectations as other product behavior.

---

## Relationship to the Engine

The composition layer does not render audio directly.

It produces/updates authoritative musical state that the `WhiteRoomEngine` abstraction realizes through the active engine implementation:

```text
Composition Engine
       |
       v
      Song
       |
       v
WhiteRoomEngine
   |      |      |
 Mock    Live   Remote
          |       |
         FFI   WebSocket
          |       |
          +--- C++20 DSP ---+
```

This separation keeps composition logic testable and prevents AI concerns from leaking into the real-time audio callback.

---

## GSA Relationship

White Room applies [GSA — Governed Stewardship Architecture](https://github.com/bretbouchard/gsa-system) in a deliberately lightweight creative form.

The shared principles are:

- authoritative state outside the model
- replaceable model processors
- explicit capabilities/tools
- preserved project intent and preferences
- inspectable changes
- validation before acceptance

Unlike Volta, White Room does not need heavy engineering approval workflows for ordinary composition. The goal is fluid creative interaction while retaining strong state and execution boundaries.
