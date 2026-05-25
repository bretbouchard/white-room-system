# Theory — Schillinger System

The mathematical foundation for composition in White Room.

---

## What This Is

White Room's composition engine is built on **Joseph Schillinger's** mathematical approach to music composition (1946). Instead of intuitive composition, Schillinger provides systematic methods for generating musical elements from numerical relationships.

---

## Core Concepts

### Rhythm

#### Resultants
A resultant is a rhythmic pattern derived from the interaction of two or more uniform patterns.

```
Example: Resultant of 3 and 4

Pattern A (3):  X . . X . . X . . X . . X . .
Pattern B (4):  X . . . X . . . X . . . X . .
────────────────────────────────────────────
Resultant:      X . . X X . X . X X . . X X .
```

The resultant contains attacks from both patterns, creating syncopated rhythms.

#### Interference
When two or more patterns overlap, they create interference — the combination reveals emergent patterns.

```
3 against 4 creates a 12-beat cycle
4 against 5 creates a 20-beat cycle
```

#### Distribution
How events are placed across available time slots:
- **Uniform**: Evenly spaced
- **Random**: Stochastic placement
- **Weighted**: Probability curves
- **Clustered**: Grouped events

---

### Melody

#### Geometric Pitch
Pitch relationships mapped to geometric space:
- Linear sequences (ascending/descending)
- Arches (rise and fall)
- Random walk
- Intervallic patterns

#### Contour
The shape of a melody over time:
- Rising
- Falling
- Arch (rise then fall)
- Inverted arch (fall then rise)
- Oscillating

#### Motivic Development
Techniques for developing musical ideas:
- **Sequence**: Repeat at different pitch level
- **Inversion**: Flip intervals
- **Retrograde**: Play backwards
- **Augmentation**: Stretch durations
- **Diminution**: Compress durations

---

### Harmony

#### Chord Generation
Chords derived from:
- Scale degrees (diatonic harmony)
- Voice leading rules
- Functional progressions (T-D-T)
- Chromatic interpolation

#### Voice Leading
Rules for moving voices between chords:
- Minimize movement (nearest chord tone)
- Avoid parallel fifths/octaves (optional)
- Resolve tendency tones (7→1, 4→3)
- Stay within vocal ranges

#### Harmonic Rhythm
How frequently chords change:
- One chord per bar
- Two chords per bar
- Variable (fast/slow sections)

---

### Form

#### Section Types
- **Intro**: Opening material
- **Verse**: Main narrative content
- **Chorus**: Hook, memorable refrain
- **Bridge**: Contrasting material
- **Pre-chorus**: Builds to chorus
- **Outro**: Closing material

#### Energy Curve
Overall intensity over time:
```
Energy
   │     ╭──╮      ╭────╮
   │    ╱    ╲    ╱      ╲
   │   ╱      ╲  ╱        ╲
   │  ╱        ╲╱          ╲
   └──────────────────────────▶ Time
     Intro Verse Chorus Bridge Outro
```

---

## Implementation in White Room

### Continuous Reactive Analysis

Schillinger analysis runs continuously as notes change — not as a separate tab or mode. A dedicated worker thread (pinned to core 0 or 1, never audio cores) processes analysis via lock-free MPSC queue, with results appearing within 500ms of any note change.

### Overlay Lenses

Color-coded overlays render directly on the piano roll, toggleable by the user:

| Lens | Color | Shows |
|------|-------|-------|
| **Generators** | Blue | Which Schillinger generators produced each note |
| **Scale Degrees** | Green | Scale degree of each pitch |
| **Contour** | Yellow | Melodic contour (rising/falling/arch) |
| **Tension** | Red | Harmonic tension level |
| **Voice Leading** | Purple | Voice leading paths between chords |

### Bidirectional Drum Support

The Schillinger system works bidirectionally for drums:
- **Generate**: Schillinger params -> drum pattern (resultants, interference, density)
- **Analyze**: Existing drum pattern -> Schillinger report (what theory it follows)
- **Lock mode**: Lock specific drum hits (e.g., hi-hat pattern) and regenerate everything else

### Density Curves

Schillinger density parameters drive continuous density curves visible at song, section, and track levels:
- Density curve rendered as a continuous line (not a knob)
- Inherited hierarchy: song -> section -> track (each can override independently)
- Breakpoints on the curve for manual shaping
- Density strategies: linear, exponential, stepped, Schillinger resultant
- Manual notes always preserved during density changes
- Note origin tracking: every note tagged as 'manual' or 'generated' at DSP level

### Analysis Pipeline

```
Note Input Change
       |
       v
[Worker Thread] (core 0 or 1, SCHED_OTHER)
       |
       v
[ReverseEngineeringBridge]
       |
       +-- PitchModel
       +-- RhythmModel
       +-- HarmonyModel
       |
       v
[Lock-free MPSC Queue] -> UI Thread
       |
       v
[Overlay Lens Rendering on Piano Roll]
       |
       +-- Ambient Info Bubbles (contextual analysis insights)
       +-- Density Curve Updates
       +-- Spinner -> Checkmark on completion
```

---

## Data Flow

```
Form (structure)
    |
    v
Rhythm (timing grid)
    |
    v
Melody (pitch content)
    |
    v
Harmony (chord framework)
    |
    v
Orchestration (instrument assignment)
    |
    v
Density (density curves at song/section/track levels)
    |
    v
House Band (16-channel performance)
```

---

## Key Schillinger Terms

| Term | Definition |
|------|------------|
| Resultant | Pattern from combined uniform patterns |
| Interference | Overlap of multiple patterns |
| Grouping | Organizing beats into phrases |
| Distribution | Event placement across slots |
| Geometric Pitch | Spatial pitch relationships |
| Balance | Symmetry in time and pitch |

---

## Further Reading

- Schillinger, Joseph. *The Schillinger System of Musical Composition*. 1946.
- Koechlin, Charles. *Précis des règles du contrepoint*. 1926.
- Arden, Jeremy. *Schillinger System* (online resources)
