# White Room Intelligence

White Room uses AI as a governed participant in a live musical system, not as the owner of the song.

The authoritative song, performance state, engine state, preferences, and executable capabilities remain outside the language model. Models receive only the context needed for the current task and act through explicit White Room tools.

This is the White Room application of [GSA — Governed Stewardship Architecture](https://github.com/bretbouchard/gsa-system).

---

## Runtime Model

```text
                    User intent
                        |
                        v
              +-------------------+
              | Current Song      |
              | authoritative     |
              | musical state     |
              +---------+---------+
                        |
                controlled context
                        |
                        v
              +-------------------+
              | Model / Processor |
              | Apple Foundation  |
              | Models or other   |
              | governed model    |
              +---------+---------+
                        |
                 proposal/tool call
                        |
                        v
              +-------------------+
              | White Room Tools  |
              | bounded musical   |
              | capabilities      |
              +---------+---------+
                        |
                        v
              +-------------------+
              | Validation        |
              | structure, music, |
              | engine invariants |
              +---------+---------+
                        |
                        v
              +-------------------+
              | Song / Engine     |
              | accepted mutation |
              +---------+---------+
                        |
                        v
              +-------------------+
              | Observation       |
              | user/runtime      |
              | consequences      |
              +---------+---------+
                        |
                        +----> GSA Modeled World
```

A model can reason about the song and propose operations. It does not directly replace canonical song state or bypass engine contracts.

---

## AI v1 Composition

The intended White Room AI runtime combines four things:

1. **A capable language/reasoning model** — Apple Foundation Models is the primary local Apple path; the surrounding contracts are designed so model implementations can change.
2. **Music-domain expertise** — learned adapters and domain-specific musical knowledge provide specialized composition, performance, sound-design, and mixing behavior.
3. **The current Song object** — the authoritative live musical/project context supplied to the model in controlled form.
4. **White Room tools** — explicit executable capabilities for inspecting and changing composition, instruments, effects, automation, performance, and other governed state.

The model is therefore one processor in a larger system rather than the application itself.

---

## Authoritative State

White Room keeps truth in typed application state and engine state rather than conversational memory.

Examples include:

- notes, timing, sections, tracks, harmony, rhythm, form, and orchestration
- instrument and preset assignments
- mixer and effects state
- automation and performance parameters
- user-authored vs generated material
- current engine capabilities
- relevant user musical preferences and project intent
- expected outcomes for consequential product or system changes
- observed outcome evidence where applicable

A model may receive a projection of these objects. It does not become their source of truth.

---

## Tool Boundary

AI-driven changes use the same conceptual boundary as other application changes:

```text
Model
  |
  | structured request
  v
White Room tool contract
  |
  v
Policy / validation
  |
  v
WhiteRoomEngine / Song operation
  |
  v
Authoritative state mutation
  |
  v
Verification
  |
  v
Playable result
```

Useful tools can include bounded operations such as:

- inspect a section, track, harmonic region, or rhythmic pattern
- add, move, transform, or remove generated musical events
- change density or orchestration within explicit scope
- choose or modify an instrument/preset
- adjust mixer, effects, or automation values
- ask theory/analysis services for structured findings
- render or audition a proposed change

Tool contracts are preferable to arbitrary code or shell access because they make scope, validation, side effects, and testing explicit.

---

## A Typical Governed Musical Change

User request:

> Make the chorus denser without changing its harmony.

```text
1. Read the current Song and identify the chorus.
2. Preserve the existing harmonic structure as a hard constraint.
3. Inspect current density, instrumentation, note origins, and available voices.
4. Propose bounded rhythmic/orchestration changes.
5. Apply those changes through White Room tools.
6. Validate event integrity, timing, voice limits, and the preserved harmony constraint.
7. Commit the accepted Song mutation.
8. Play the result through the same engine used by the normal UI.
```

The important distinction is that the model does not regenerate an opaque song document. It operates against structured state through controlled capabilities.

---

## Failure Is Data

White Room's AI path follows the same testing philosophy as the rest of the system: generated output is not accepted because it sounds plausible in prose.

For example:

```text
requested transformation
        |
        v
generated event operations
        |
        v
music / structure validation
        |
      FAILED
        |
        v
diagnose invalid, dropped, duplicated, or out-of-range events
        |
        v
revised operations
        |
        v
validation rerun
        |
      PASSED
```

Where expected musical events are known, tests can verify that notes were not dropped, duplicated, moved outside the requested scope, or changed contrary to constraints.

---

## Reality Feedback and Outcome Validation

White Room participates in GSA's shared Reality Feedback Loop:

```text
Observe → Evidence → Interpret → Investigate → Decide → Plan → Govern → Execute → Verify → Observe
```

White Room may contribute evidence such as:

- crashes and regressions
- DSP glitches, underruns, and performance measurements
- repeated corrective actions
- abandoned or repeatedly retraced workflows
- recurring musical and tonal preferences
- mixer, instrument, and effect interaction patterns
- playback/render discrepancies
- publishing failures
- explicit user feedback

These observations are evidence, not automatic requirements. A repeated action or complaint may justify investigation, but White Room must not silently convert behavioral telemetry into autonomous product changes.

Implementation and outcome remain separate:

```text
Implementation: Not Started → Executing → Implemented → Verified
Outcome:        Not Observed → Observing → Outcome Validated | Outcome Failed | Inconclusive
```

A UI or engine change may pass every test and still fail to improve the workflow or musical result it was intended to improve. `Verified` therefore does not mean `Outcome Validated`.

Where appropriate, White Room should preserve the expected outcome before a consequential change and later return observed evidence to the GSA Modeled World and Historian. Failed or inconclusive outcomes may become new unknowns or governed follow-up work.

---

## Local Intelligence

White Room is designed to take advantage of Apple's on-device intelligence stack where it is appropriate, including Foundation Models on supported operating systems and hardware.

On-device reasoning is attractive for White Room because:

- the Song object can remain local
- latency is predictable
- there is no per-request cloud cost
- musical/project context does not need to be uploaded merely to make routine changes
- tools still enforce the same application contracts regardless of model source

The architecture does not require every intelligence capability to use the same provider. Models remain replaceable processors behind stable application contracts.

---

## Relationship to GSA

White Room intentionally uses a lighter GSA profile than engineering domains such as Volta.

The same general rules still apply:

- reality/state is owned outside the model
- models receive controlled projections
- actions happen through explicit capabilities
- important output is testable and inspectable
- durable preferences and project intent belong in application memory/state, not a prompt transcript
- observations become governed evidence rather than silently becoming requirements
- implementation verification and outcome validation remain distinct

White Room emphasizes fluid creative interaction rather than heavyweight planning. GSA provides the safety and state boundary without turning music-making into project-management UI.

---

## What This Demonstrates

White Room's intelligence layer sits inside a larger production system spanning:

- Swift application architecture
- C++20 real-time DSP
- a single controlled Swift/C++ FFI boundary
- Core ML and local ML inference
- Apple Foundation Models integration direction
- typed Song and performance state
- deterministic mock/live engine abstractions
- tool-driven agent interaction
- multi-platform Apple UI
- real-time audio constraints
- automated unit, integration, and end-to-end testing
- governed reality feedback and outcome validation

That combination is deliberate: the AI system has to operate against a real product and real-time engine rather than a standalone chat demo.
