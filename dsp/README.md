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

## Instruments Using Sound Substrate

| Instrument | Primary Engine | Additional |
|------------|---------------|------------|
| **Aether** | Additive, Granular | Spectral |
| **Kane** | Modal, Chaos | Additive |
| **Choral** | Spectral | Granular |
| **Growl** | Chaos | Additive |
