# Mixing — ConsoleX

The mixing and signal routing infrastructure for White Room.

---

## What ConsoleX Is

ConsoleX is the 16-channel mixer that brings together all audio sources in White Room:
- **Instruments** from House Band (up to 32 voices per channel)
- **28 effect pedals** for processing
- **Console bus processing** via Airwindows Console
- **Buses** for grouping and sends

---

## Architecture

```
                        +------------------------------------------+
                        |              MASTER BUS                  |
                        |  [SoundModelerChain] (optional insert)  |
                        |  Console Bus (Airwindows Console)       |
                        |  +- Limiter +- Stereo Width +- EQ       |
                        +------------------+-----------------------+
                                           |
        +----------------+------------------+-------------------+
        |                |                  |                   |
   +----+----+      +----+----+       +----+----+         +----+----+
   |  AUX 1  |      |  AUX 2  |       | GROUP 1 |        | GROUP N |
   | (Reverb)|      | (Delay) |       | (Drums) |        |         |
   +----+----+      +----+----+       +----+----+         +---------+
        |                |                  |
   +----+----+      +----+----+       +----+----+
   | Reverb  |      |  Delay  |       | Ch 1-4  |
   |  Pedal   |      |  Pedal  |       | (mixed) |
   +---------+      +---------+       +---------+
        ^                ^                  ^
        |         +------+------+---+---+---+---------+
        |         |      |      |   |   |   |         |
   +----+----+ +--+----+ +--+--+ +-+-+ +-+-+ +--+--+ +--+--+
   |  Ch 1   | | Ch 2  | |Ch 3 | |Ch4| |Ch5| |Ch 6 | |Ch16 |
   | Drums   | | Bass  | |Lead | |Pad| |FX | | Tex | | ... |
   | EQ/Cmp/ | |EQ/Cmp/ |EQ/Cm | |   | |   | |     | |     |
   | Gate/Sat| |Gate/Sat| |     | |   | |   | |     | |     |
   | [SMC]   | | [SMC]  | |     | |   | |   | |     | |     |
   +---------+ +-------+ +-----+ +---+ +---+ +-----+ +-----+
        |         |        |       |     |     |       |
        v         v        v       v     v     v       v
   +----------------------------------------------------------+
   |                    HOUSE BAND ENGINE                       |
   |              30+ Instruments | 28 Effect Pedals           |
   +----------------------------------------------------------+

   [SMC] = SoundModelerChain (optional per-channel or bus insert)
```

---

## 16-Channel Mixer

### Channel Strip

Each of the 16 channels has:

- **Per-channel EQ** — 4-band parametric
- **Compressor** — Threshold, ratio, attack, release, knee
- **Gate** — Threshold, attack, hold, release
- **Saturation** — Analog-style warmth and drive
- **Console Bus** — Airwindows Console processing per channel

### Per-Channel Processing Chain

```
Instrument Output
       |
       v
[EQ: Lo/LoMid/HiMid/Hi]
       |
       v
[Compressor: Thresh/Ratio/Attack/Release]
       |
       v
[Gate: Thresh/Attack/Hold/Release]
       |
       v
[Saturation: Drive/Tone]
       |
       v
[Console Bus Processing (Airwindows)]
       |
       v
[Fader: Volume + Pan + Mute + Solo]
       |
       v
[Send: Aux 1-4]
       |
       v
To Group/Master Bus
```

### Sound Modeler Chain Integration

Sound modelers can be inserted at multiple points in the mixing path:

Per-Channel Insert Points:
  Instrument Output -> [EQ] -> [Compressor] -> [Gate] -> [Saturation] -> [SoundModelerChain] -> [Console Bus] -> [Fader]

Bus Insert Points:
  Bus Input -> [ConsoleEncode] -> [Bus Sum] -> [ConsoleDecode] -> [SoundModelerChain] -> [Bus Processing]

Master Insert Points:
  Master Input -> [SoundModelerChain] -> [Console Bus] -> [Limiter] -> [Stereo Width] -> [Output]

### De-Zippering

All continuous parameters (gain, pan, send levels, effect wet/dry) are applied with configurable de-zippering. Default: 10ms ramp. No zipper noise on any output path.

---

## Buses

### Group Buses
Combine related channels:
- **Drums**: Kick, snare, hats, toms -> Drum bus
- **Vocals**: Lead, backing, harmonies -> Vocal bus
- **Instruments**: Synths, pads, leads -> Instrument bus

### Aux Buses
Parallel processing sends:
- **Aux 1**: Reverb (shared space)
- **Aux 2**: Delay (time-based)
- **Aux 3**: Chorus/Modulation
- **Aux 4**: Parallel compression

### Master Bus
Final stereo output with Airwindows Console bus processing:
- **Console bus**: Analog console summing emulation
- **Limiter**: Brick-wall protection
- **Stereo width**: Mono to super-wide
- **Master EQ**: Tonal balance

---

## House Band Channels

The virtual ensemble across 16 channels:

| Channels | Role | Typical Instruments |
|----------|------|-------------------|
| 1-4 | Rhythm section | Drums (4 variants), Percussion |
| 5-8 | Bass and harmony | Growl, Aether, LocalGal |
| 9-12 | Lead and counterline | Kane, NexSynth, Breath |
| 13-16 | Texture and FX | Choral, Motion, Nature, VoiceSynth |

Each channel:
- Receives MIDI from Schillinger Engine or manual input
- Routes to assigned instrument via SongBridge
- Has full channel strip processing (EQ/comp/gate/sat)
- Can send to aux buses
- Supports up to 32 voices (512 total across 16 channels on Pi 5)

---

## Control Paths

Mixer controls work over both local FFI and remote WebSocket:

```
User moves fader
  -> ViewsControls (fader)
    -> AudioEngineBackend.setMixerGain()
      -> [Local] wr_houseband_mixer_set_gain() via FFI -> ParameterManager
      -> [Remote] WebSocket command -> Pi server -> ParameterManager
        -> LinearSmoothedValue::setTarget()
          -> Smoothed value applied in next audio block
```

---

## Automation

### Automatable Parameters

| Parameter | Range | Automation Mode |
|-----------|-------|-----------------|
| Volume | -inf to +12dB | Touch, Latch, Write |
| Pan | L100 to R100 | Touch, Latch, Write |
| Mute | On/Off | Touch |
| Send levels | 0 to 1 | Touch, Latch, Write |
| Effect parameters | Varies | Touch, Latch, Write |

---

## Technical Constraints

### Latency Budget

| Component | Budget |
|-----------|--------|
| Input processing | 1ms |
| Channel strip (EQ/comp/gate/sat) | 1ms |
| Bus routing | 0.5ms |
| Console bus processing | 0.5ms |
| Master processing | 1.5ms |
| **Total** | **4.5ms** |

### CPU Usage

- Target: < 50% total (60% on Pi 5)
- Per channel: < 3%
- Master bus: < 5%

### Thread Safety

- All fader/pan reads: Atomic
- Buffer processing: Lock-free
- UI updates: Main thread only
- RT safety: Zero heap allocations in audio callback
