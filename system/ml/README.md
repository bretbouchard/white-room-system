# ML — Machine Learning

ML-powered features for White Room.

---

## What This Is

The ML layer provides:

- **Voice Synthesis** — Core ML DDSP voice instrument with real-time timbre control
- **Composition Assistance** — AI chat, ML suggestions, mix coaching
- **Audio Analysis** — Style classification, preset recommendations
- **Voice/Choir** — Multi-voice choir mode with individual controls

All inference runs on-device. No cloud. No API calls. No user data leaves the device.

---

## Core ML Voice Synthesis

The primary ML feature is real-time DDSP voice synthesis using 5 bundled Core ML models:

### Bundled Models

| Model | File | Purpose |
|-------|------|---------|
| **ConditionEncoder** | `VoiceConditionEncoder.mlpackage` | Encodes conditioning parameters for the voice |
| **NSF** | `VoiceNSF.mlpackage` | Neural Source Filter — generates audio from conditioning |
| **Vocoder** | `VoiceVocoder.mlpackage` | Vocodes the neural output to final audio |
| **SpeakerEncoder** | `VoiceSpeakerEncoder.mlpackage` | Encodes speaker identity for voice presets |
| **PhonemeToFormant** | `PhonemeToFormant.mlpackage` | Converts phonemes to formant frequencies |

### Voice Pipeline

```
Text Input
    |
    v
[Phase 1: G2P] "HELLO" -> ["HH", "EH", "L", "OW"]
    |
    v
[Phase 2: PhonemeToFormant (Core ML)]
    ["HH", "EH", "L", "OW"] -> F1/F2/F3 frequencies
    |
    v
[Phase 3: SpeakerEncoder (Core ML)]
    Speaker identity -> conditioning vector
    |
    v
[Phase 4: ConditionEncoder (Core ML)]
    Merge phoneme + speaker + VoiceControlIR (16-dim)
    |
    v
[Phase 5: NSF + Vocoder (Core ML)]
    Conditioning -> audio frames
    |
    v
[Phase 6: DDSP Synthesis (C++)]
    FormantFilterBank + HarmonicOscillatorBank + GlottalPulseShaper
    |
    v
Audio Output
```

### Swift Components

| Component | Purpose |
|-----------|---------|
| `VoiceModelCoreML` | Core ML model loading + inference |
| `VoiceInferenceManager` | Manages inference lifecycle and scheduling |
| `VoiceThermalManager` | Adaptive inference rate based on ProcessInfo.thermalState |
| `VoiceModelAudioBuffer` | Audio buffer for render thread |
| `VoiceModelBridge` | FFI bridge to C++ DDSP engine |
| `VoiceEngineBootstrapper` | Concurrent model loading (async let), DSP fallback on failure |

### C++ Components

| Component | Purpose |
|-----------|---------|
| `VoiceModelEngine` | State machine with DSP fallback, overlap-add, ring buffers |
| `VoiceModelCAPI` | C API bridge for Swift FFI |
| `VoiceModelRingBuffer` | Lock-free ring buffer for inference -> render |
| `VoiceDDSP` | Complete DSP chain (FormantFilterBank, HarmonicOscillatorBank, GlottalPulseShaper) |
| `VoiceControlIR` | 16-dim control parameter struct |

### Three-Tier Rendering

| Tier | Description | Latency |
|------|-------------|---------|
| **Real-time** | Live inference at adaptive rate (100Hz normal, 50Hz thermal) | ~10ms |
| **Offline** | Pre-render voice frames for playback | N/A |
| **Cached** | FNV-1a hashed inference cache for repeated frames | ~0ms |

### Overlap-Add (OLA) Engine

- Polyphase sinc interpolation (16-tap, 64 phases) over linear to avoid aliasing
- 80% max overlap (higher causes phase artifacts at 256-sample frames)
- Voice frames resampled from inference rate to audio rate

### Thermal Management

`VoiceThermalManager` observes `ProcessInfo.thermalState` via `AsyncStream`:

| Thermal State | Inference Rate | Ramp Time |
|---------------|---------------|-----------|
| Normal | 100Hz | N/A |
| Fair | 75Hz | ~250ms |
| Serious | 50Hz | ~500ms |
| Critical | 25Hz | ~700ms |

Rate changes use `DispatchSourceTimer.schedule()` on the active timer — no recreation, no kernel calls. Smooth 10% per 100ms tick ramp rate.

### Voice UI

- **Timbre XY Pad** — 2D control for voice timbre (breathy <-> resonant, dark <-> bright)
- **6 Dimension Sliders** — Individual voice parameter control
- **Voice Presets** — Soprano, alto, tenor, bass, falsetto, whisper
- **Thermal Badge** — Toolbar badge showing thermal state + inference rate
- **Rendering Indicator** — Three-tier status (real-time / offline / cached)
- **Choir Mode** — Multi-voice choir with individual voice controls and blend

---

## AI Chat and Composition Assistance

### Chat Interface

Compact popup/sidebar for natural language music direction:

- "Make the bridge more dense"
- "Add a counter-melody"
- "Change the harmony to Lydian"

Chat triggers generation via Schillinger engine. Responses include Schillinger-aware context.

### ML Suggestions

UI affordances for:
- **Preset recommendations** — Based on current song context
- **Mix coaching** — AI suggests EQ, compression, level adjustments
- **Style classification** — ML classifies song style and suggests complementary changes
- **Semantic preset search** — Find presets by description, not just name

---

## Integration

### Model Loading

Core ML models load concurrently via `async let` at app startup. DSP fallback activates on any model load failure — the voice instrument always works, even without ML models.

### FFI Bridge

```
Swift (VoiceInferenceManager)
  -> VoiceModelBridge (FFI)
    -> VoiceModelCAPI (C)
      -> VoiceModelEngine (C++)
        -> VoiceDDSP (C++ DSP chain)
          -> AudioGraph (mixer output)
```

### On-Device Processing

All ML inference runs locally:
- No API calls to external services
- No user data leaves the device
- Core ML models run on Neural Engine / GPU / CPU (Core ML auto-selects)
- DSP fallback ensures voice always works
