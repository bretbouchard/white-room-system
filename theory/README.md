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

### Five-Tab Interface

1. **Form Tab**
   - Define section order
   - Set section lengths
   - Draw energy curve
   - Mark transitions

2. **Rhythm Tab**
   - Select resultant numbers
   - Choose interference mode
   - Set density
   - Apply swing

3. **Melody Tab**
   - Choose scale
   - Draw contour
   - Set interval constraints
   - Define motifs

4. **Harmony Tab**
   - Select progression
   - Set harmonic rhythm
   - Configure voice leading
   - Assign inversions

5. **Orchestration Tab**
   - Map roles to instruments
   - Set density per section
   - Define articulations
   - Control dynamics

---

## Data Flow

```
Form (structure)
    │
    ▼
Rhythm (timing grid)
    │
    ▼
Melody (pitch content)
    │
    ▼
Harmony (chord framework)
    │
    ▼
Orchestration (instrument assignment)
    │
    ▼
House Band (performance)
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
- Arden, Jeremy. *Schillinger System* (online resources)
