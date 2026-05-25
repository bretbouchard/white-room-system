# Sound

Your instruments, tone, and mix. DSP, modelers, and routing combined.

---

## What This Is

The Sound pillar encompasses everything audio:

- **DSP** — The synthesis and processing engines
- **Modelers** — Neural and analog tone shaping
- **Mixing** — The mixer and signal routing

---

## Layers

### [DSP](./dsp/) — Sound Substrate

The audio engine. DSP-first architecture with minimal JUCE dependencies (7 kept, 2 removed):

- 30+ instruments across 16 synthesizer families, samplers, orchestral, keys, and drums
- 10+ synthesis engines: FM, Wavetable/VA, Additive, Granular, Physical Modeling, Spectral, DDSP, Chaos, Modal, Sample-based
- 28 effect pedals
- Cross-platform: iOS, macOS, tvOS, visionOS, Windows, Linux

### [Modelers](./modelers/) — ISoundModeler System

Neural and analog modelers for real-time audio processing:

- ISoundModeler generic interface (10 modeler kinds)
- AmpCaptureModeler (NAM Core v0.5.2 neural amp simulation)
- CabinetIR, RoomIR (partitioned convolution impulse responses)
- ConsoleModeler (analog console summing encode/decode)
- TapeModeler, TransformerModeler (analog color)
- CircuitFilterModeler (ZDF analog filter models)
- PhysicalInstrumentModeler (Karplus-Strong+, modal resonators)
- PerformanceModeler (seeded deterministic control curves)
- NeuralFXModeler (ONNX/RTNeural generic neural FX)
- SoundModelerChain with ping-pong scratch buffers
- Tone3000 community integration and metadata catalog
- Model preview and A/B comparison

### [Mixing](./mixing/) — ConsoleX

The mixer that brings everything together:

- 16-channel mixer with per-channel EQ, compression, gate, saturation
- Airwindows Console bus processing
- Bus routing (aux, groups, master)
- Effect chains and pedal ordering
- De-zippered parameter changes (configurable 10ms ramp default)

---

## Relationship

```
DSP                             MODELERS                    MIXING
────                            ─────────                    ──────
Sound generation        ->      Tone shaping         ->      Signal routing
30+ Instruments         ->      Amp/cab/IR capture   ->      16-channel mixer
28 Effect pedals        ->      Analog color         ->      ConsoleX bus processing
Raw audio output        ->      Neural FX            ->      Mixed stereo master
```

DSP creates the sound. Modelers shape the tone. Mixing routes the result.
