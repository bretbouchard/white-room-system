# Mixing — ConsoleX

The mixing and signal routing infrastructure for White Room.

---

## What ConsoleX Is

ConsoleX is the mixer that brings together all audio sources in White Room:
- **Instruments** from House Band
- **Effects** for processing
- **Audio tracks** for recording
- **Buses** for grouping and sends

---

## Architecture

```
                        ┌─────────────────────────────────────────┐
                        │              MASTER BUS                  │
                        │            (Stereo Output)              │
                        └────────────────▲────────────────────────┘
                                         │
        ┌────────────────────────────────┼────────────────────────────────┐
        │                                │                                │
   ┌────┴────┐                      ┌────┴────┐                      ┌────┴────┐
   │  AUX 1  │                      │  GROUP  │                      │  AUX 2  │
   │ (Reverb)│                      │   BUS   │                      │ (Delay) │
   └────▲────┘                      └────▲────┘                      └────▲────┘
        │                                │                                │
   ┌────┴────┐                      ┌────┴────┐                      ┌────┴────┐
   │ Reverb  │                      │ Drums   │                      │  Delay  │
   │  Unit   │                      │  Group  │                      │  Unit   │
   └─────────┘                      └─────────┘                      └─────────┘
        ▲                                ▲                                ▲
        │         ┌──────────────────────┼──────────────────────┐        │
        │         │                      │                      │        │
   ┌────┴────┐ ┌──┴────┐ ┌────────┐ ┌────┴────┐ ┌────────┐ ┌────┴────┐ ┌─┴────┐
   │  Lead   │ │  Pad  │ │ Chords │ │  Drums  │ │  Bass  │ │ Texture │ │ FX   │
   │ Channel │ │Channel│ │Channel │ │ Channel │ │Channel │ │ Channel │ │Channel│
   └─────────┘ └───────┘ └────────┘ └─────────┘ └────────┘ └─────────┘ └──────┘
        │         │         │          │         │          │         │
        ▼         ▼         ▼          ▼         ▼          ▼         ▼
   ┌─────────────────────────────────────────────────────────────────────────┐
   │                           HOUSE BAND                                     │
   │                    (Virtual Instrument Ensemble)                         │
   └─────────────────────────────────────────────────────────────────────────┘
```

---

## Channel Strip

Each channel in ConsoleX has:

### Input Section
- **Source selector**: Which instrument/track feeds this channel
- **Input gain**: Pre-fader level
- **Phase invert**: Polarity flip

### EQ Section
- **Low shelf**: 80Hz, ±12dB
- **Low mid**: 250Hz, Q=1.0, ±12dB
- **High mid**: 2.5kHz, Q=1.0, ±12dB
- **High shelf**: 12kHz, ±12dB

### Dynamics Section
- **Compressor**: Threshold, ratio, attack, release, knee
- **Gate**: Threshold, attack, hold, release

### Fader Section
- **Volume fader**: -∞ to +12dB
- **Pan knob**: L100 to R100
- **Mute**: Cut signal
- **Solo**: Isolate channel

### Sends
- **Aux 1-4**: Post-fader sends to effect buses
- **Pre-fader option**: For parallel processing

---

## Buses

### Group Buses
Combine related channels:
- **Drums**: Kick, snare, hats, toms → Drum bus
- **Vocals**: Lead, backing, harmonies → Vocal bus
- **Instruments**: Synths, pads, leads → Instrument bus

### Aux Buses
Parallel processing sends:
- **Aux 1**: Reverb (shared space)
- **Aux 2**: Delay (time-based)
- **Aux 3**: Chorus/Modulation
- **Aux 4**: Parallel compression

### Master Bus
Final stereo output:
- **Limiter**: Brick-wall protection
- **Stereo width**: Mono to super-wide
- **Master EQ**: Tonal balance
- **Loudness metering**: LUFS readout

---

## House Band Channels

The virtual ensemble:

| Channel | Role | Default Instrument |
|---------|------|-------------------|
| 1 | Drums | Kane (percussion mode) |
| 2 | Bass | Growl |
| 3 | Chords | Aether (pad mode) |
| 4 | Lead | Kane |
| 5 | Pad | Aether |
| 6 | Texture | Choral |
| 7 | FX | FilterGate |

Each channel:
- Receives MIDI from Schillinger Engine
- Routes to assigned instrument
- Has full channel strip processing
- Can send to aux buses

---

## Signal Flow

```
MIDI In
   │
   ▼
┌──────────┐
│  House   │
│  Band    │──────┐
│Instrument│      │
└──────────┘      │
   │              │
   ▼              ▼
┌──────────┐  ┌──────────┐
│ Channel  │  │   Aux    │
│  Strip   │  │   Send   │
└────┬─────┘  └────┬─────┘
     │             │
     ▼             ▼
┌──────────┐  ┌──────────┐
│  Group   │  │   FX     │
│   Bus    │  │  Return  │
└────┬─────┘  └────┬─────┘
     │             │
     └──────┬──────┘
            ▼
     ┌────────────┐
     │   Master   │
     │    Bus     │
     └─────┬──────┘
           ▼
      ┌─────────┐
      │  Output │
      └─────────┘
```

---

## ConsoleX UI

### Layout

```
┌─────────────────────────────────────────────────────────────────────┐
│  Transport:  ▶ ▶▶ ⏹ ⏺ │  BPM: 120 │  Time: 1.2.3.4 │  CPU: 23%   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐                 │
│  │ 1 │ │ 2 │ │ 3 │ │ 4 │ │ 5 │ │ 6 │ │ 7 │ │MST│   Channels      │
│  │   │ │   │ │   │ │   │ │   │ │   │ │   │ │   │                 │
│  │ ▓ │ │ ▓ │ │ ▓ │ │ ▓ │ │ ▓ │ │ ▓ │ │ ▓ │ │ ▓ │   Meters       │
│  │   │ │   │ │   │ │   │ │   │ │   │ │   │ │   │                 │
│  └───┘ └───┘ └───┘ └───┘ └───┘ └───┘ └───┘ └───┘                 │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    SELECTED CHANNEL                          │   │
│  │  EQ:      ░░▓▓░░      ░░░▓░░░      ░░▓▓░░      ░░░░▓░░    │   │
│  │          80Hz       250Hz       2.5kHz      12kHz           │   │
│  │                                                              │   │
│  │  Dynamics:  ▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░                          │   │
│  │            Threshold -12dB  Ratio 4:1  Attack 10ms          │   │
│  │                                                              │   │
│  │  Sends:    Aux1: 0.3  Aux2: 0.1  Aux3: 0.0  Aux4: 0.0      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Interactions

- **Click channel**: Select for detailed view
- **Drag fader**: Adjust volume
- **Double-click fader**: Reset to 0dB
- **Shift+drag**: Fine adjustment
- **Right-click**: Context menu (routing, presets)

---

## Automation

### Automatable Parameters

| Parameter | Range | Automation Mode |
|-----------|-------|-----------------|
| Volume | -∞ to +12dB | Touch, Latch, Write |
| Pan | L100 to R100 | Touch, Latch, Write |
| Mute | On/Off | Touch |
| Send levels | 0 to 1 | Touch, Latch, Write |
| Plugin parameters | Varies | Touch, Latch, Write |

### Automation Modes

- **Read**: Plays back automation
- **Touch**: Writes while touching, returns to previous on release
- **Latch**: Writes while touching, stays at last position
- **Write**: Overwrites all existing automation

---

## Technical Constraints

### Latency Budget

| Component | Budget |
|-----------|--------|
| Input processing | 1ms |
| Channel strip | 1ms |
| Bus routing | 0.5ms |
| Master processing | 2ms |
| **Total** | **4.5ms** |

### CPU Usage

- Target: < 50% total
- Per channel: < 3%
- Master bus: < 5%

### Thread Safety

- All fader/pan reads: Atomic
- Buffer processing: Lock-free
- UI updates: Main thread only
