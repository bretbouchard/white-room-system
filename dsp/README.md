# DSP — Sound Substrate

The synthesis and audio processing engines for White Room.

---

## What Sound Substrate Is

Sound Substrate is the **audio engine** that powers White Room's instruments. It provides multiple synthesis methods through a unified interface:

- **Additive** — Harmonic partial synthesis
- **Granular** — Grain cloud processing
- **Modal** — Physical modeling resonance
- **Spectral** — FFT-based manipulation
- **Chaos** — Nonlinear dynamics

---

## DSP-First Architecture

### Philosophy

White Room is built on a **DSP-first** foundation, not tied to low-level framework dependencies. The audio engine is abstracted from any specific platform API:

```
┌─────────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                         │
│                    (SwiftUI / Native UI)                     │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                    SOUND SUBSTRATE                           │
│                    (Pure DSP Layer)                          │
│                                                              │
│   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │
│   │Additive │ │Granular │ │ Modal   │ │Spectral │ ...      │
│   └─────────┘ └─────────┘ └─────────┘ └─────────┘          │
│                                                              │
│   No JUCE dependencies | No platform-specific code          │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                    PLATFORM LAYER                            │
│   AudioUnit v3  │  VST3  │  CLAP  │  LV2  │  Standalone    │
└─────────────────────────────────────────────────────────────┘
```

This separation means:
- **Same DSP code** runs on all platforms
- **No vendor lock-in** to any single framework
- **Easy porting** to new platforms and formats
- **Consistent sound** across all targets

---

## Platform Coverage

### The Apple TV Threshold

**Apple TV was our high water mark.**

tvOS has no support for external synths or effects — no AudioUnit hosting, no plugin loading. If White Room works on Apple TV, it works everywhere:

| Platform | External Plugins | White Room |
|----------|------------------|------------|
| **tvOS** | ❌ Not supported | ✅ Built-in |
| **iOS** | ✅ AUv3 hosting | ✅ Native |
| **macOS** | ✅ AU/VST3/CLAP | ✅ Native |
| **visionOS** | ⚠️ Limited | ✅ Native |
| **Windows** | ✅ VST3/CLAP | ✅ Portable |
| **Linux** | ✅ LV2/CLAP | ✅ Portable |

**We built for the TV so we could work everywhere.**

### Supported Formats

| Format | Platforms | Status |
|--------|-----------|--------|
| **AUv3** | iOS, macOS, tvOS, visionOS | ✅ Primary |
| **VST3** | macOS, Windows, Linux | ✅ Supported |
| **CLAP** | macOS, Windows, Linux | ✅ Supported |
| **LV2** | Linux | ✅ Supported |
| **Standalone** | All platforms | ✅ Development |

For Apple platforms, Sound Substrate is distributed as **XCFramework**. See [Frontend](../frontend/) for integration details.

---

## Sample-Based Instruments

### Open Source + DDSP

In addition to synthesis engines, White Room includes sample-based instruments built with:

- **Open source sample libraries** — High-quality recordings, community-maintained
- **DDSP (Differentiable Digital Signal Processing)** — Neural synthesis that combines DSP with ML

### Available Instruments

| Instrument | Source | Technology |
|------------|--------|------------|
| **Piano** | Open source samples | DDSP-enhanced |
| **Orchestral** | Open source samples | DDSP-enhanced |

**DDSP advantages:**
- Realistic timbral transitions
- Expressive control beyond velocity
- Smaller memory footprint than pure sampling
- Combines neural networks with interpretable DSP

---

## Synthesis Engines

### Additive

Sum of sinusoidal partials with independent control.

**How it works**:
```
Fundamental (f₀)
    │
    ├── Partial 1: f₀      × A₁
    ├── Partial 2: 2f₀     × A₂
    ├── Partial 3: 3f₀     × A₃
    ├── Partial 4: 4f₀     × A₄
    │   ...
    └── Partial N: Nf₀     × Aₙ
                    │
                    ▼
                Sum → Output
```

**Parameters**:
| Parameter | Range | Description |
|-----------|-------|-------------|
| Partial 1-16 | 0-1 | Amplitude of each partial |
| Brightness | 0-1 | Overall high frequency content |
| Detune | ±cents | Spread of partials |
| Inharmonic | 0-1 | Deviation from harmonic series |

**Best for**: Bells, organs, pads, evolving textures

---

### Granular

Sound composed of thousands of tiny grains.

**How it works**:
```
Source Audio
    │
    ▼
┌─────────────────────────────────────────────┐
│              Grain Cloud                     │
│                                              │
│   •    •  •     •      •   •    •           │
│     •     •    •   •      •     •    •      │
│   •    •      •     •       •    •          │
│                                              │
│  Each grain: 1-100ms, independent envelope  │
└─────────────────────────────────────────────┘
    │
    ▼
Output
```

**Parameters**:
| Parameter | Range | Description |
|-----------|-------|-------------|
| Grain size | 1-100ms | Duration of each grain |
| Density | 1-100 | Grains per second |
| Position | 0-1 | Where in source to read |
| Randomness | 0-1 | Variation in position/size |
| Pitch | ±2 oct | Transposition |

**Best for**: Textures, atmospheres, time-stretching

---

### Modal

Physical modeling of resonant objects.

**How it works**:
```
Exciter (strike, pluck, blow)
    │
    ▼
┌─────────────────────────────────────────────┐
│              Modal Bank                      │
│                                              │
│   Mode 1: f₁, Q₁, A₁                        │
│   Mode 2: f₂, Q₂, A₂                        │
│   Mode 3: f₃, Q₃, A₃                        │
│   ...                                        │
│   Mode N: fₙ, Qₙ, Aₙ                        │
│                                              │
│   Each mode = resonant filter (bandpass)    │
└─────────────────────────────────────────────┘
    │
    ▼
Output
```

**Parameters**:
| Parameter | Range | Description |
|-----------|-------|-------------|
| Material | preset | Preset resonance (wood, metal, glass) |
| Decay | 0.1-10s | Sustain length |
| Brightness | 0-1 | High frequency content |
| Inharmonicity | 0-1 | Non-harmonic modes |

**Best for**: Bells, strings, membranes, resonant bodies

---

### Spectral

FFT-based sound manipulation.

**How it works**:
```
Input Signal
    │
    ▼
┌─────────────────────────────────────────────┐
│         FFT (Fast Fourier Transform)         │
│                                              │
│   Time Domain → Frequency Domain             │
│                                              │
│   Bin 1: f=43Hz,  mag=0.5, phase=45°        │
│   Bin 2: f=86Hz,  mag=0.8, phase=120°       │
│   Bin 3: f=129Hz, mag=0.3, phase=200°       │
│   ...                                        │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│         Spectral Processing                  │
│                                              │
│   - Shift frequencies (transposition)        │
│   - Scale magnitudes (filtering)             │
│   - Freeze phases (spectral freeze)          │
│   - Blur across time (spectral smoothing)    │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│              Inverse FFT                     │
│                                              │
│   Frequency Domain → Time Domain             │
└─────────────────────────────────────────────┘
    │
    ▼
Output Signal
```

**Parameters**:
| Parameter | Range | Description |
|-----------|-------|-------------|
| Shift | ±2 oct | Frequency shifting |
| Blur | 0-1 | Temporal smoothing |
| Freeze | 0-1 | Hold current spectrum |
| Formant | 0-1 | Preserve formants |

**Best for**: Vocoding, morphing, spectral filtering

---

### Chaos

Nonlinear dynamics and noise.

**How it works**:
```
Initial State (x₀, y₀, z₀)
    │
    ▼
┌─────────────────────────────────────────────┐
│         Chaotic System (e.g., Lorenz)       │
│                                              │
│   dx/dt = σ(y - x)                          │
│   dy/dt = x(ρ - z) - y                      │
│   dz/dt = xy - βz                           │
│                                              │
│   σ, ρ, β = system parameters               │
└─────────────────────────────────────────────┘
    │
    ▼
Output (map chaotic variable to audio)
```

**Parameters**:
| Parameter | Range | Description |
|-----------|-------|-------------|
| Chaos | 0-1 | Amount of chaotic behavior |
| Complexity | 1-10 | Dimensionality of attractor |
| Rate | 0.1-10 | Speed of evolution |
| Filter | LP/HP/BP | Output filtering |

**Best for**: Noise, unpredictable textures, organic sounds

---

## Instrument Architecture

### Voice Management

```
Note On
    │
    ▼
┌────────────────────┐
│   Voice Allocator  │
│                    │
│  Voice 1: [free]  │
│  Voice 2: [C4]    │
│  Voice 3: [E4]    │
│  Voice 4: [free]  │
│  ...               │
└────────┬───────────┘
         │
         ▼
┌────────────────────┐
│  Steal if needed   │
│  (oldest/quietest) │
└────────┬───────────┘
         │
         ▼
┌────────────────────┐
│   Voice Instance   │
│                    │
│  - Synthesis Engine│
│  - Amp Envelope    │
│  - Filter Envelope │
│  - LFO Modulation  │
└────────────────────┘
```

### Voice Structure

```
          ┌──────────────────────────────────────────────────────┐
          │                     VOICE                              │
          │                                                        │
MIDI ────▶│  Oscillator ──▶ Filter ──▶ Amplifier ──▶ Pan ──▶ Mix  │
Note      │      │            │           │                         │
          │      │            │           │                         │
          │  ┌───┴───┐    ┌───┴───┐   ┌───┴───┐                   │
          │  │  LFO  │    │  LFO  │   │  LFO  │                   │
          │  └───────┘    └───────┘   └───────┘                   │
          │       │            │           │                         │
          │       └────────────┴───────────┘                         │
          │              Modulation                                   │
          │                                                          │
          └──────────────────────────────────────────────────────────┘
```

---

## Effect Processors

### Built-in Effects

| Effect | Description |
|--------|-------------|
| **Filter** | LP/HP/BP/Notch, resonance, envelope |
| **Drive** | Soft/hard clipping, waveshaping |
| **Delay** | Tempo-sync, feedback, ping-pong |
| **Reverb** | Algorithmic, room size, decay |
| **Chorus** | Modulated delays, width |
| **Phaser** | All-pass filters, stages |
| **Compressor** | Threshold, ratio, attack, release |

### Effect Chain

```
Instrument Output
       │
       ▼
┌──────────────┐
│   Filter     │
└──────┬───────┘
       ▼
┌──────────────┐
│   Drive      │
└──────┬───────┘
       ▼
┌──────────────┐
│   Delay      │
└──────┬───────┘
       ▼
┌──────────────┐
│   Reverb     │
└──────┬───────┘
       ▼
    Output
```

---

## Real-Time Constraints

### The Rules

1. **No allocation in audio thread**
   - Pre-allocate all buffers
   - Pre-allocate voice pool

2. **No blocking operations**
   - No file I/O
   - No network calls
   - No mutex locks

3. **Bounded execution**
   - Must complete within buffer period
   - At 44.1kHz, 256 samples = 5.8ms
   - At 96kHz, 256 samples = 2.7ms

### Performance Budget

| Component | Budget |
|-----------|--------|
| Synthesis engine | 2ms |
| Filter | 0.5ms |
| Effects | 1.5ms |
| Voice mixing | 0.5ms |
| **Total per voice** | **4.5ms** |

---

## Parameter System

### Thread-Safe Access

```cpp
// UI Thread - Write
parameter->setValue(newValue);

// Audio Thread - Read (atomic, no locks)
float value = parameter->getAtomicValue();
```

### Parameter Types

| Type | Range | Skew |
|------|-------|------|
| Linear | 0-1 | None |
| Frequency | 20-20000Hz | Logarithmic |
| Time | 0.001-10s | Logarithmic |
| Decibels | -∞ to +12dB | Linear |
| Percent | 0-100% | Linear |

### Smoothing

All parameter changes are smoothed to prevent clicks:
```
currentValue = currentValue + (targetValue - currentValue) × smoothCoeff
```

---

## Instrument Canon

### Synthesizers (16)

| Instrument | Engine | Description |
|------------|--------|-------------|
| **NexSynth** | FM | 2/4/6-operator FM synthesis |
| **Kane** | Wavetable/VA | Virtual analog with wavetables |
| **Aether** | Additive + Granular | Ambient/textural soundscapes |
| **LocalGal** | Hybrid | Local Galactic multi-engine |
| **String** | Additive | String machine emulation |
| **Breath** | Physical model | Breath-controlled synthesis |
| **BreathLead** | Physical model | Expressive breath lead |
| **Growl** | Chaos + Additive | Aggressive, evolving textures |
| **Choral** | Spectral | Choir/vocal textures |
| **ChoirV2** | Spectral | Enhanced choir ensemble |
| **Motion** | Granular | Evolving motion textures |
| **Nature** | Sample + Granular | Environmental soundscapes |
| **Giants** | Additive | Massive, epic sounds |
| **MajorModulator** | Modulation | Dedicated modulation synth |
| **CDC4046** | PLL | PLL-based synthesis (4046 chip emulation) |
| **VoiceSynth** | DDSP + ML | **AI Voice Synthesis** (see below) |

### AI Voice Synthesis — VoiceSynth

**Real-time text-to-speech synthesis with zero external dependencies.**

```
┌─────────────────────────────────────────────────────────────┐
│                    VOICE SYNTH PIPELINE                       │
│                                                              │
│   Phase 1: Text → Phoneme (G2P Dictionary)                  │
│            "HELLO" → ["HH", "EH", "L", "OW"]                 │
│                                                              │
│   Phase 2: Phoneme → Formant (ML Model)                     │
│            ["HH", "EH", "L", "OW"] → F1/F2/F3 frequencies    │
│            100x faster than real-time target                 │
│                                                              │
│   Phase 3: Formant → Audio (DDSP Oscillator)                │
│            Formant frequencies → Additive synthesis output   │
└─────────────────────────────────────────────────────────────┘
```

**Parameters:**
| Parameter | Range | Description |
|-----------|-------|-------------|
| Speaking Rate | 1-50 phonemes/sec | Speed of speech |
| Pitch | 80-400 Hz | Fundamental frequency |
| Volume | 0-100% | Output amplitude |
| Text | string | Text to synthesize |

**MIDI Controllable:** Note-on triggers synthesis with pitch, velocity.

### Samplers

| Instrument | Technology | Description |
|------------|------------|-------------|
| **SamSampler** | DDSP + Samples | Multi-sample playback with DDSP enhancement |

### Orchestral (12)

All orchestral instruments use **open source samples + DDSP**:

**Winds (5):** Flute, Clarinet, Oboe, Bassoon, Saxophone

**Brass (4):** Trumpet, Trombone, French Horn, Tuba

**Percussion (3):** Timpani, Cymbals, MalletPercussion (Marimba, Vibraphone, Xylophone, etc.)

### Keys/Pianos (3)

| Instrument | Technology | Description |
|------------|------------|-------------|
| **Piano** | DDSP + Samples | Acoustic piano with neural enhancement |
| **Rhodes** | Physical model | Electric piano emulation |
| **Rhodes3D** | Physical model + Spatial | Spatial Rhodes with 3D imaging |

### Drums (4)

| Instrument | Technology | Description |
|------------|------------|-------------|
| **DrumMachine** | Synthesis | Electronic drum synthesis |
| **DrumHybrid** | Samples + Synthesis | Hybrid acoustic/electronic |
| **DrumSamples** | Samples | Acoustic drum samples |
| **Drums** | Samples | General drum kit |

---

## Shared DSP Backend

Many instruments share the same underlying DSP engines:

```
┌─────────────────────────────────────────────────────────────┐
│                    PURE DSP LAYER                             │
│                   (No JUCE Dependencies)                      │
│                                                              │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│   │ KanePure │  │ Additive │  │ Granular │  │ Spectral │   │
│   │   DSP    │  │  Engine   │  │  Engine   │  │  Engine   │   │
│   └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│         │              │              │              │          │
│         └──────────────┴──────────────┴──────────────┘          │
│                              │                                 │
│   Instruments: Kane, Aether, LocalGal, Growl, Giants,            │
│   Breath, Choral, Motion, String, VoiceSynth...                 │
└─────────────────────────────────────────────────────────────┘
```

**Merged Instruments:** Several instruments were consolidated because they shared the same synth backend. This reduces code duplication and ensures consistent sound across the ecosystem.
