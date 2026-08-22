# Song

Your composition. Theory, engine, and craft combined.

---

## What This Is

The Song pillar encompasses everything related to musical composition:

- **Theory** — mathematical and musical analysis running continuously against the song
- **Composition Engine** — structured, theory-aware and AI-assisted musical operations
- **Songwriting** — creative application of theory and form
- **Song Workspace** — the unified composition interface

The authoritative Song object is the source of truth. Algorithms and models inspect and transform that structured state through bounded operations rather than replacing it with opaque generated documents.

---

## Layers

### [Theory](./theory/) — Schillinger System

The mathematical foundation for composition, running as continuous reactive analysis:

- Rhythm resultants and interference patterns
- Melodic contour and motivic development
- Harmonic progressions and voice leading
- Orchestration and form
- Color-coded overlay lenses on piano roll (Generators, Scale Degrees, Contour, Tension, Voice Leading)
- Bidirectional drum support (generate from theory + analyze against theory)
- Drum lock mode (preserve specific hits during generation)
- Density curves at song/section/track levels with breakpoint editing

### [Composition Engine](./engine/) — Structured Musical Operations

The composition engine combines typed Song state, theory-aware analysis, deterministic algorithms, domain knowledge, and governed AI assistance.

- intent resolved against the current Song rather than conversational memory
- bounded composition operations instead of unrestricted document generation
- manual/user-authored material can be protected from regeneration
- harmony, form, scope, timing, and other constraints can be preserved explicitly
- generated output is validated before becoming authoritative Song state
- the same Song is realized through mock, live, or remote engine implementations

### [Songwriting](./songwriting/) — Creative Application

The craft of turning theory into songs:

- Song structure and arrangement
- Hook construction
- Emotional arc design

---

## Song Workspace

The Song Area is the default landing view in White Room's Room Architecture. It provides a unified composition workspace.

### View Modes

| Mode | Description | Tracks |
|------|-------------|--------|
| **Piano Roll** | Standard piano roll with note editing | Melodic |
| **Step Sequencer** | Grid-based pattern editing | Drums |
| **Staff Notation** | Canvas-based musical staff renderer | All |
| **Drum Pads** | MPC-style 4x4 grid with velocity and banks | Drums |

### Song Overview

- Section list with tick marks and density curve overlay
- Mini track previews per section
- Section CRUD: add, remove, reorder, duplicate
- Section markers in transport ruler
- Auto-routing: new tracks auto-assign to appropriate HouseBand channel

### Track Listing

Per-section track listing with:

- Instrument name and preset name
- Mini waveform preview
- Density indicator
- Inline mute/solo/volume

### Density Curves

Continuous density control at three levels:

```text
Song Level Density ---------> Overall density curve
  |
  +-- Section Override -----> Per-section density breakpoints
       |
       +-- Track Override --> Per-track density shaping
```

- Strategies: linear, exponential, stepped, Schillinger resultant
- Manual note preservation: density changes do not need to destroy user-placed notes
- Note-origin tracking distinguishes manual and generated events
- Changes are designed for glitch-free playback

### Ambient Information

Contextual analysis insights can be surfaced inline:

- "This section is in Lydian mode"
- "Density spike at bar 17"
- Tap a bubble for inline editing of the underlying value
- Deep-link from analysis to the relevant editor
- Priority filtering keeps only the most useful insights visible

### Song vs Performance Data Model

- **Song** = musical identity and authored structure: notes, sections, theory parameters and relationships
- **Performance** = realization parameters: densities, presets, effects, voicing and other performance choices

This separation allows the same musical identity to be realized differently without rewriting the composition itself.

---

## AI and the Song

White Room's governed intelligence architecture is documented in [System / Intelligence](../system/intelligence/).

At a high level:

```text
User intent
    |
    v
Current Song
    |
    v
Model / theory / domain reasoning
    |
    v
White Room composition tools
    |
    v
Validation
    |
    v
Accepted Song mutation
```

The model is a processor, not the source of truth. Song state, engine capabilities, constraints, user preferences, and executable operations live in the application.

---

## Relationship

```text
THEORY                 COMPOSITION ENGINE              SONGWRITING
------                 ------------------              -----------
Schillinger       ->   structured operations      ->   Creative choices
Analysis          ->   governed AI assistance     ->   Artistic judgment
Math / rules      ->   validation + transformation->   Hooks, arc, structure

                              |
                              v
                        SONG WORKSPACE
              Piano Roll | Step Seq | Staff | Drum Pads
              Overlay Lenses | Density Curves | Ambient Info
```

Theory provides structured understanding. The composition engine turns intent into bounded musical change. Songwriting supplies artistic direction. The workspace keeps all three inside the same song rather than forcing the musician into separate tools.
