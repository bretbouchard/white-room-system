# Sound Modelers — ISoundModeler System

Neural and analog modelers for real-time audio processing.

---

## What This Is

The Sound Modeler system provides a generic framework for loading and running audio modelers — neural amp captures, cabinet/room impulse responses, analog console emulation, tape saturation, and more. Every modeler implements the same `ISoundModeler` interface, runs through the same `SoundModelerChain`, and is controlled from the same UI.

White Room becomes a host, manager, and metadata layer over the existing ecosystem of captures, IRs, and models — not just another plugin.

---

## Architecture

```
+---------------------------------------------------------------+
|                    SOUNDMODELER SYSTEM                         |
|                                                                |
|  +----------------------------------------------------------+ |
|  |              SOUND MODELER CHAIN                          | |
|  |                                                           | |
|  |  Input --> [Modeler 1] --> [Modeler 2] --> [Modeler 3]   | |
|  |            AmpCapture     CabinetIR      RoomIR           | |
|  |            (NAM Core)     (Convolution)  (Convolution)    | |
|  |                                                           | |
|  |  Ping-pong scratch buffers for 3+ modelers               | |
|  |  Single-modeler fast path bypasses scratch buffers        | |
|  |  Per-modeler bypass with zero-latency passthrough         | |
|  +----------------------------------------------------------+ |
|                                                                |
|  +----------------------------------------------------------+ |
|  |              ISOUNDMODELER INTERFACE                      | |
|  |  prepare()  process()  reset()                            | |
|  |  latencySamples()  kind()  cpuClass()                     | |
|  +----------------------------------------------------------+ |
|                                                                |
|  +----------------------------------------------------------+ |
|  |              METADATA & CATALOG                           | |
|  |  ModelerMetadata + AutoTagging + SearchIndex              | |
|  |  Tone3000 Community Integration                           | |
|  |  Import pipeline (.nam, .wav, .aiff)                      | |
|  +----------------------------------------------------------+ |
+---------------------------------------------------------------+
```

---

## ISoundModeler Interface

Every modeler implements the same abstract interface:

```cpp
class ISoundModeler {
    virtual void prepare(double sampleRate, int maxBlockSize) = 0;
    virtual void process(AudioBlock& block) = 0;
    virtual void reset() = 0;
    virtual int latencySamples() const = 0;
    virtual ModelerKind kind() const = 0;
    virtual CPUClass cpuClass() const = 0;
};
```

This means:
- Any modeler plugs into any slot in the chain
- Modelers are interchangeable at runtime
- UI doesn't need to know what type of modeler it's controlling
- New modeler types can be added without changing existing code

---

## Modeler Types (10 Kinds)

| Kind | Modeler | Description |
|------|---------|-------------|
| **AmpCapture** | AmpCaptureModeler | Neural amp simulation via NAM Core v0.5.2 |
| **CabinetIR** | CabinetIRModeler | Speaker/microphone impulse response convolution |
| **RoomIR** | RoomIRModeler | Room/space convolution reverb with predelay |
| **Console** | ConsoleModeler | Analog console summing (encode/decode stages) |
| **Tape** | TapeModeler | Tape saturation + HF rolloff |
| **Transformer** | TransformerModeler | Preamp/output transformer character |
| **CircuitFilter** | CircuitFilterModeler | ZDF ladder, SVF, OTA analog filter models |
| **PhysicalInstrument** | PhysicalInstrumentModeler | Plucked strings, modal resonators, drum membranes |
| **Performance** | PerformanceModeler | Seeded deterministic timing/velocity control curves |
| **NeuralFX** | NeuralFXModeler | Generic neural FX via ONNX/RTNeural |

---

## Signal Chain

The minimum viable signal chain for guitar processing:

```
Guitar Input
    |
    v
[AmpCaptureModeler] (NAM Core)
    |  Neural amp simulation
    v
[CabinetIRModeler]
    |  Speaker/mic impulse response
    v
[RoomIRModeler]
    |  Room/space convolution reverb
    v
[ConsoleModeler]
    |  Analog summing emulation
    v
ConsoleX Mixer -> Master Output
```

But modelers work anywhere in the signal path — not just guitar. Tape on a vocal. Transformer on a drum bus. CircuitFilter on a synth lead. The chain is fully configurable.

---

## SoundModelerChain

The chain processor manages ordered modeler instances:

```
SoundModelerChain
  +- modelers[] (ordered list)
  +- scratchA, scratchB (ping-pong buffers)
  +- Process: modeler[0](input -> scratchA)
               modeler[1](scratchA -> scratchB)
               modeler[2](scratchB -> output)
```

Key behaviors:
- **Ping-pong scratch buffers** prevent aliasing artifacts when 3+ modelers chain together
- **Single-modeler fast path** bypasses scratch buffers for efficiency
- **Per-modeler bypass** with zero-latency passthrough
- **Background loading** via atomic swap — zero audio dropouts during hot-swap
- **Wet/dry mix** per modeler with delay-compensated dry signal
- **Default stereo** (2-channel); multi-channel deferred until needed

---

## Modeler Details

### AmpCaptureModeler (NAM Core)

Neural amp simulation using NAM Core v0.5.2:

- Loads `.nam` files (Neural Amp Modeler format)
- Real-time inference with deterministic output
- Architectures: Linear, ConvNet, LSTM, WaveNet
- Receptive field extracted from model metadata
- Sample rate mismatch detection before loading
- 25MB model size cap
- Path validation: extension check, size limit, canonical resolution, traversal detection
- Background model loading via atomic swap
- Soft-clip output (tanh-based limiting, not hard clip)

**FFI Bridge:** 8 `wr_modeler_*` C functions for Swift integration.

### CabinetIRModeler

Speaker/microphone impulse response convolution:

- Loads WAV/AIFF IR files
- Supports mono/mono, mono/stereo, stereo/stereo configurations
- High-cut/low-cut filtering for tone shaping
- Shared partitioned convolution backend with RoomIR
- Background loading with cache

### RoomIRModeler

Room/space convolution reverb:

- Partitioned FFT convolution for IRs of any length
- Predelay, decay scaling, wet/dry mix controls
- Crossfade on IR change (never clicks)
- Short ambience IRs run on tvOS budget
- Long IRs can be disabled on constrained devices

### ConsoleModeler

Analog console summing emulation (White Room identity):

- Two-stage: ConsoleEncode (per-channel, pre-sum) + ConsoleDecode (per-bus, post-sum)
- Three modes: Pure (near bit-transparent single channel), Classic, Color
- DC blocking after nonlinear bus stage
- Inspired by Airwindows PurestConsole architecture
- Signal chain: Track inserts -> ConsoleEncode -> Bus Sum -> ConsoleDecode -> Bus processors

### TapeModeler

Analog tape machine characteristics:

- Stage 1: input trim -> tanh/cubic saturation -> gentle HF rolloff -> output trim
- Speed, bias, and mix controls
- CPU class "cheap" — runs on every channel

### TransformerModeler

Analog transformer character:

- Low-frequency saturation + asymmetric saturation
- Bandwidth shaping + subtle phase shift
- Core types: Clean, Neve-ish, API-ish, Tube Output
- CPU class "cheap"

### CircuitFilterModeler

Analog synth filter models:

- Zero-delay feedback (ZDF) topologies
- Filter types: Ladder LP, SVF (LP/HP/BP/Notch), OTA LP
- Stable at extreme resonance (clean self-oscillation)
- Drive before/inside filter with optional oversampling
- Key tracking parameter routes MIDI note to cutoff modulation
- Parameter smoothing on cutoff and resonance

### PhysicalInstrumentModeler

Physical modeling synthesis (keeps Apple TV self-contained):

- Plucked string (Karplus-Strong+) with brightness and damping control
- Modal resonator bank (bells, bodies, metallic resonances)
- Drum membrane via modal bank
- Excitation source: pluck, strike, bow
- No external plugin dependency

### PerformanceModeler

Seeded deterministic control curves:

- Produces control data (timing offsets, velocity, articulation), NOT audio
- Section-aware (verse/chorus/bridge/drop adapt energy and density)
- Role-aware (lead/bass/pad/drum/arp patterns match musical role)
- Same seed + params = identical curves every time
- Output: noteTimeOffsetsMs[], velocityMultipliers[], ccCurves[], articulations[]

### NeuralFXModeler

Generic neural FX loading:

- ONNX or RTNeural model formats
- Strict CPU budget enforcement
- Falls back to bypass if model exceeds budget on current device
- Quality tiers: eco/normal/high for device-adaptive rendering

---

## CPU Classification

| Class | Description | Devices |
|-------|-------------|---------|
| **Cheap** | Runs everywhere, every channel | All |
| **Moderate** | Reasonable CPU, limited instances | All |
| **High** | Significant CPU, 1-2 instances | iOS/Mac/Pi (not tvOS) |
| **Extreme** | Maximum quality, desktop/Pi only | Mac/Pi only |

tvOS rejects High/Extreme CPU models. iOS triggers model unload on memory pressure.

---

## Metadata & Catalog

### Unified Metadata Schema

Every modeler file gets a `ModelerMetadata` entry:

| Field | Purpose |
|-------|---------|
| id | Unique identifier |
| kind | ModelerKind (10 types) |
| tags[] | Auto-generated + user tags |
| sampleRate | Model sample rate |
| lengthSamples | IR/model length |
| cpuClass | CPU budget requirement |
| recommendedFor[] | Suggested use cases |
| source | Origin (file, Tone3000, etc.) |
| author | Creator name |
| license | License type |
| format | File format (.nam, .wav, .aiff) |
| fileHash | Content hash for deduplication |
| importDate | When it was imported |

### Auto-Tagging

On import, files are automatically tagged from:
- NAM JSON metadata (author, amp, tubes)
- WAV/AIFF headers (sample rate, channels, length)
- Filename and path analysis (brand, model, mic type)
- Format detection (.nam, .wav, .aiff auto-detected)

### Search Index

Inverted index supports:
- Tag-based queries
- Kind-based filtering
- Text-based search
- CPU class filtering
- Sub-100ms results across 500+ captures

---

## Tone3000 Community Integration

In-app browsing of the Tone3000 NAM model and IR ecosystem:

- Browse/search community captures with filters
- Downloaded captures auto-import with community tags and ratings
- Community tags merge with auto-generated tags
- Cloud sync preserves libraries across devices (iCloud KVS)
- Offline mode: previously downloaded captures work without network
- Storage management: total capture storage view, delete from UI, warnings at 80%

---

## Preview & Comparison

Users can audition before committing:

- **Preview render**: 2-4 second deterministic audio clip from any modeler using reference input
- **A/B comparison**: Toggle between two loaded models with crossfade
- **Side-by-side**: Metadata + waveform + CPU cost for two models simultaneously
- **Preview cache**: Avoids re-rendering same model+input combination
- **CPU budget guard**: Refuses to preview Extreme CPUClass models on constrained devices

---

## Preset Schema (UPFS)

Unified Preset Format for Sound Modelers:

- Versioned JSON serialization for modeler chains
- Round-trip guarantee: serialize -> deserialize -> identical chain state
- Forward-compatible ModelerKind handling
- Quality tiers (eco/normal/high) per preset

---

## Integration

### C++ Layer

```
ISoundModeler (abstract base)
  +-- AmpCaptureModeler (wraps NAM Core)
  +-- CabinetIRModeler (convolution)
  +-- RoomIRModeler (convolution)
  +-- ConsoleModeler (encode/decode DSP)
  +-- TapeModeler (saturation + rolloff)
  +-- TransformerModeler (asymmetric saturation)
  +-- CircuitFilterModeler (ZDF filters)
  +-- PhysicalInstrumentModeler (Karplus-Strong+, modal)
  +-- PerformanceModeler (control curves)
  +-- NeuralFXModeler (ONNX/RTNeural)
```

### FFI Bridge

Generic `WR_Modeler_*` C API (not NAM-specific for future reuse):
- `wr_modeler_load` / `wr_modeler_unload`
- `wr_modeler_process`
- `wr_modeler_set_param`
- `wr_modeler_get_info`
- `wr_modeler_bypass`
- `wr_modeler_get_latency`
- `wr_modeler_get_cpu_class`
- `kind_hint` field identifies modeler type (e.g., "AmpCapture" for NAM)

### Swift Layer

Via WhiteRoomKit `ModelerAPI`:
- Model loading with progress and error states
- Model selection UI (`NAMModelPickerView`, generic modeler picker)
- Model controls UI (mix, bypass, model-specific params)
- Model catalog browsing and search
- Preview rendering

### Pedal Integration

Modelers integrate into the existing pedal system via adapter pattern:
- `NeuralAmpPedalAdapter` wraps `AmpCaptureModeler` behind `GuitarPedalPureDSP`
- Adapter owns mix parameter, passes mix=1.0 to modeler
- PedalFactory registers modeler pedals alongside existing DSP pedals
- Existing `WR_Pedalboard_SetParameter` FFI works for modeler parameters

---

## Platform Constraints

| Platform | Behavior |
|----------|----------|
| **tvOS** | Rejects High/Extreme CPU models; graceful degradation |
| **iOS** | Memory pressure triggers model unload; thermal-aware loading |
| **macOS** | Full capability; all modeler types and sizes |
| **Pi 5** | Full capability; dedicated cores for processing |

---

## Testing

- **Golden render tests**: Every modeler has determinism tests (same input + params = same output hash)
- **CPU profiling harness**: Measures process() time per modeler at 44.1k/48k with 128/256/512 frames
- **Full chain integration**: Guitar -> AmpCapture -> CabinetIR -> RoomIR -> Console -> Master produces expected output
- **tvOS fallback verification**: Every modeler either runs within budget or degrades cleanly
- **Coverage**: >= 80% for all modeler code

---

## Related Documentation

- [DSP](../dsp/) — Sound Substrate synthesis engines
- [Mixing](../mixing/) — ConsoleX mixer and signal routing
- [WhiteRoomKit](../../system/white_room_kit/) — ModelerAPI in the engine protocol
- [Frontend](../../system/frontend/) — UI for modeler controls
