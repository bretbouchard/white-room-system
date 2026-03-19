# BettaFish-MiroFish Intelligence Layer

Multi-member generative composition system for White Room.

---

## Overview

The Intelligence Layer transforms user intent into musical output through a sophisticated multi-member system:

1. **Intent Interpretation** — Natural language or UI selection → structured intent
2. **Musical Specialists** — Domain experts propose musical decisions
3. **Forum Engine (BettaFish)** — Deliberation, conflict resolution, synthesis
4. **Simulation Engine (MiroFish)** — Temporal state evolution
5. **Ensemble Members** — Real-time musical decisions
6. **Renderer/Realizer** — Simulation output to playable music

---

## Architecture

```
User Intent
    │
    ▼
┌─────────────────────────────────────┐
│     Intent Interpretation Layer     │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│       Musical Specialists (6)       │
│  Structure | Harmony | Rhythm       │
│  Motif | Emotion | Expression       │
└─────────────────┬───────────────────┘
                  │ SpecialistProposal[]
                  ▼
┌─────────────────────────────────────┐
│      Forum Engine (BettaFish)       │
│  • Collection: Parallel execution   │
│  • Conflict Detection: Overlap score│
│  • Synthesis: Priority-weighted     │
│  • Plan Generation: CompositionPlanIR│
└─────────────────┬───────────────────┘
                  │ CompositionPlanIR
                  ▼
┌─────────────────────────────────────┐
│    Simulation Engine (MiroFish)     │
│  • WorldState (12 variables)        │
│  • Tick Loop: Bar → Phrase → Section│
│  • Checkpoints: Branch/restore      │
│  • Event Emission: SimulationTimeline│
└─────────────────┬───────────────────┘
                  │ SimulationTimeline
                  ▼
┌─────────────────────────────────────┐
│        Ensemble Members (8)         │
│  Bass | HarmonyBed | Lead           │
│  Counterline | Texture              │
│  Listener | EnergyCtrl | SectionGate│
└─────────────────┬───────────────────┘
                  │
                  │ EnsembleAction[]
                  ▼
┌─────────────────────────────────────┐
│        Renderer/Realizer            │
│  VoiceMapper → MIDI → Audio Export  │
└─────────────────┬───────────────────┘
                  │
                  ▼
         PatternIR / SongIR
```

---

## Components

### Musical Specialists (6)

| Specialist | Domain | Responsibilities |
|------------|--------|------------------|
| **Structure** | Form | Section layout, repetition, phrase length, tension curve |
| **Harmony** | Harmony | Key, scale, chord movement, modulation |
| **Rhythm** | Rhythm | Density, pulse, syncopation, emphasis |
| **Motif** | Melody | Recurrence, variation, transformation |
| **Emotion** | Expression | Intended affect, energy curve, mood |
| **Expression** | Dynamics | Velocity, articulation, timing |

### Forum Engine (BettaFish)

**Purpose:** Multi-member deliberation and synthesis

**Process:**
1. **Collection** — Execute all specialists in parallel
2. **Conflict Detection** — Score overlaps between proposals
3. **Synthesis** — Priority-weighted merge of proposals
4. **Plan Generation** — Output unified CompositionPlanIR

**Output:** `CompositionPlanIR` — A complete, conflict-free composition plan

### Simulation Engine (MiroFish)

**Purpose:** Temporal state evolution

**WorldState Variables:**

| Variable | Range | Purpose |
|----------|-------|---------|
| `energyLevel` | 0-1 | Overall intensity |
| `density` | 0-1 | Note density |
| `harmonicTension` | 0-1 | Chord tension |
| `motifFamiliarity` | 0-1 | Theme recognition |
| `noveltyBudget` | 0-1 | Remaining novelty |
| `grooveTightness` | 0-1 | Rhythmic precision |
| `arrangementOccupancy` | 0-1 | Active voices ratio |
| `emotionalVector` | [-1,1]² | Valence/Arousal |
| `barIndex` | 0+ | Current position |
| `phraseIndex` | 0+ | Current phrase |
| `sectionIndex` | 0+ | Current section |
| `tickInBar` | 0-3 | Quarter note position |

**Output:** `SimulationTimeline` — Bar-by-bar event sequence

### Ensemble Members (8)

**Musical Ensemble:**

| Member | Role | Triggers |
|--------|------|----------|
| **Bass** | Low-end foundation | Root motion, rhythm |
| **HarmonyBed** | Chord voicing, pad | Harmonic changes |
| **Lead** | Melodic content | Motif development |
| **Counterline** | Counter-melody | Complementary motion |
| **Texture** | Atmospheric elements | Density changes |

**Meta Members:**

| Member | Role | Monitors |
|--------|------|----------|
| **Listener** | Responds to density, surprise | WorldState |
| **EnergyCtrl** | Manages energy curve | Energy trajectory |
| **SectionGate** | Controls transitions | Section boundaries |

### Renderer/Realizer

**Purpose:** Convert simulation to musical representation

**Components:**
- **VoiceMapper** — Ensemble member → instrument slot mapping with priority and voice stealing
- **MIDIRenderer** — Timeline → MIDI file
- **AudioExporter** — MIDI → WAV/FLAC/MP3
- **SheetMusicRenderer** — Timeline → MusicXML/PDF/LilyPond
- **ExportCoordinator** — Orchestrates multi-format exports

---

## Data Schemas

All schemas use Zod for runtime validation with TypeScript type inference.

### Core Schemas

```typescript
// Ensemble member output format
SpecialistProposalSchema

// Forum Engine output
CompositionPlanIRSchema

// Simulation Engine output
SimulationTimelineSchema

// State variables
WorldStateSchema

// Explainability log
DecisionSchema

// Accountability system
FeatureRegistrySchema
```

---

## Implementation

**Location:** `sdk/packages/core/src/` in main repository

**Subdirectories:**
```
├── schemas/bettafish/     # Phase 33.1
├── forum-engine/          # Phase 33.2
├── simulation/            # Phase 33.3
├── ensemble/              # Phase 33.4
└── renderer/              # Phase 33.6
```

**Technology Stack:**
- TypeScript 5.3+
- Zod 3.22+ (runtime validation)
- Vitest 1.0+ (testing with 100% coverage)
- Node.js EventEmitter (streaming)

---

## UI Introspection (Phase 33.5)

Users can observe and understand the intelligence layer:

- **Timeline View** — Bar-by-bar state evolution
- **Energy/Tension Overlay** — Visual curves on timeline
- **Ensemble Contribution Panel** — Per-decision breakdown
- **Rationale Inspector** — "Why did this happen?" queries
- **State Graph** — Visual state machine
- **Traceability Interface** — Full provenance tracking

---

## Design Principles

1. **Determinism** — Same seed → identical output
2. **Explainability** — Every decision traceable to members
3. **Testability** — 100% coverage requirement
4. **Streaming** — Real-time UI updates via events
5. **Extensibility** — New members can be added easily

---

## Related Documentation

- [BettaFish-MiroFish Plan](../../../.planning/phases/33-bettaFish-miroFish/BETTAFISH-MIROFISH-PLAN.md)
- [Phase 33.1: Foundation Schemas](../../../.planning/phases/33.1-foundation-schemas/)
- [Phase 33.2: Forum Engine Core](../../../.planning/phases/33.2-forum-engine-core/)
- [Phase 33.3: Simulation Engine Core](../../../.planning/phases/33.3-simulation-engine-core/)
- [Phase 33.4: Musical Actor Agents](../../../.planning/phases/33.4-musical-actor-agents/)
- [Phase 33.5: UI Introspection](../../../.planning/phases/33.5-ui-introspection/)
- [Phase 33.6: Renderer/Realizer](../../../.planning/phases/33.6-renderer-realizer/)

---

*Phase 33 — BettaFish-MiroFish Intelligence Layer*
*Status: In Development*
