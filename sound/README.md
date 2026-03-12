# Sound

Your instruments and mix. DSP and routing combined.

---

## What This Is

The Sound pillar encompasses everything audio:

- **DSP** — The synthesis and processing engines
- **Mixing** — The mixer and signal routing

---

## Layers

### [DSP](./dsp/) — Sound Substrate

The audio engine. DSP-first architecture with no framework dependencies:

- Synthesizer engines (Additive, Granular, Modal, Spectral, Chaos)
- Sample-based instruments with DDSP
- Effect processors
- Cross-platform: iOS, macOS, tvOS, visionOS, Windows, Linux

### [Mixing](./mixing/) — ConsoleX

The mixer that brings everything together:

- Channel strips with EQ, dynamics, sends
- Bus routing (aux, groups, master)
- Effect chains
- House Band channels

---

## Relationship

```
DSP                             MIXING
────                            ──────
Sound generation        →       Signal routing
Synths, Effects         →       Channels, Buses
Raw audio output        →       Mixed stereo master
```

DSP creates the sound. Mixing shapes it.
