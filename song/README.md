# Song

Your composition. Theory, engine, and craft combined.

---

## What This Is

The Song pillar encompasses everything related to musical composition:

- **Theory** — The mathematical foundation (Schillinger System) running as continuous reactive analysis
- **Engine** — The generative composition system (BettaFish-MiroFish)
- **Songwriting** — The creative application of theory
- **Song Workspace** — The unified composition interface

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

### [Engine](./engine/) — BettaFish-MiroFish Layer

The generative composition system (**active R&D** — the TypeScript engine runs standalone in the SDK and is not yet invoked by the shipping Swift app; see [Engine](./engine/) for integration status):

- **Forum Engine** — Multi-member deliberation (6 Musical Specialists)
- **Simulation Engine** — Temporal state evolution
- **Counterpoint Engine** — Koechlin voice-leading, species rules
- **Ensemble Members** — Bass, Harmony, Lead, Counterline, Texture
- **Renderer/Realizer** — Simulation to musical output

### [Songwriting](./songwriting/) — Creative Application

The craft of turning theory into songs:

- Song structure and arrangement
- Hook construction
- Emotional arc design

---

## Song Workspace

The Song Area is the default landing view in White Room's Room Architecture. It provides a unified composition workspace:

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
- Inline mute/solo/volume (no deep navigation needed)

### Density Curves

Continuous density control at three levels:

```
Song Level Density ---------> Overall density curve
  |
  +-- Section Override -----> Per-section density breakpoints
       |
       +-- Track Override --> Per-track density shaping
```

- Strategies: linear, exponential, stepped, Schillinger resultant
- Manual note preservation: density changes never affect user-placed notes
- Note origin tracking: every note tagged 'manual' or 'generated' at DSP level
- Glitch-free changes with configurable crossfade (default 10ms)

### Ambient Information

Contextual analysis insights surfaced inline:
- "This section is in Lydian mode"
- "Density spike at bar 17"
- Tap bubble for inline editing of the underlying value
- Tap bubble for deep-link navigation (density bubble -> density editor)
- Priority filtering: only 3-5 most impactful insights visible

### Song vs Performance Data Model

- **Song** = immutable DNA (notes, sections, Schillinger params)
- **Performance** = realization params (densities, presets, effects, voicing)
- Atomic swap at 5 levels: section, instrument, ensemble member, whole ensemble, full performance
- Share Song DNA by reference; recipients apply own Performance locally

---

## Relationship

```
THEORY              ENGINE                  SONGWRITING
------              ------                  -----------
Schillinger   ->    BettaFish-MiroFish  ->   Creative choices
Math rules    ->    Generative system   ->   Artistic judgment
Foundation    ->    Real-time decisions ->   Hooks, Arc, Structure

                        |
                        v
                  SONG WORKSPACE
                  Piano Roll | Step Seq | Staff | Drum Pads
                  Overlay Lenses | Density Curves | Ambient Info
```

Theory provides the foundation. Engine transforms it into music. Songwriting makes it memorable. The workspace brings it all together.
