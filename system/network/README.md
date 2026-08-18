# Network — Pi 5 Network Engine

Headless network-attached synth engine on Raspberry Pi 5.

---

## What This Is

The Pi 5 network path is an active White Room architecture and implementation
track. The repo contains real remote-engine, discovery, protocol, and Pi audio
backend code, but this path should not be described as fully release-qualified
until the physical Pi image and end-to-end system validation are complete.

Apple devices (iPad, iPhone, Mac, Apple TV) are intended to connect over WiFi
or Ethernet and control it via WebSocket. The Pi handles DSP; the Apple device
handles UI.

**The iPad becomes the face. The Pi becomes the brain.**

---

## Architecture

```
Apple Device (Controller)              Pi 5 / CM5 (Engine)
+-------------------------------+     +-------------------------------+
|  SwiftUI UI                   |     |  WebSocket Control Server     |
|  Room Architecture            |     |  (libwebsockets, C++)         |
|  Remote engine client         |<--->|  Protocol v1.0                |
|   +-- Remote (WebSocket) -----+--+  |                               |
+-------------------------------+  |  |  White Room Engine (C++)      |
                                  +->|  +-- Sound Substrate (C++20)  |
                                      |  +-- HouseBand Engine         |
                                      |  +-- 30+ Instruments          |
                                      |  +-- 28 Effect Pedals         |
                                      |  +-- 16ch Mixer + Console     |
                                      |  +----------------------------+
                                      |                               |
                                      |  Pipewire (sole audio infra)  |
                                      |  +-- AES67/RTP network out    |
                                      |  +-- ALSA MIDI bridge         |
                                      |  +-- JACK client compat       |
                                      |  +-- Graph-based routing      |
                                      |                               |
                                      |  Buildroot system image       |
                                      |  +-- Read-only squashfs       |
                                      |  +-- PREEMPT_RT kernel        |
                                      |  +-- CPU isolation (2,3)      |
                                      |  +-- A/B partition + OTA      |
                                      +-------------------------------+
```

---

## Why Pi 5

The iPad is a phenomenal controller but a constrained DSP host. The Pi offloads all audio compute:

- **256-512 voices at < 60% CPU** — iPad caps at ~64 voices with thermal throttling
- **16-channel mixer with Console processing** — impossible on iOS with limited RAM
- **Network audio output** — integrates with studio gear via AES67/ethernet
- **$97 module cost** — less than a single high-end synth plugin license
- **Headless reliability** — no notifications, no background apps, no iOS updates
- **USB MIDI** — hardware controllers connect directly, zero latency to engine
- **Deterministic scheduling** — PREEMPT_RT + CPU isolation, not iOS shared-driver model

---

## WebSocket Control Protocol

### Protocol v1.0

JSON-based bidirectional protocol over WebSocket:

**Client -> Server (Commands):**
- Transport: play, pause, stop, seek
- Mixer: set_gain, set_pan, set_mute, set_solo
- Song: load_song (chunked binary transfer)
- Instruments: set_instrument, set_preset
- Effects: set_effect_param, reorder_chain
- Fire-and-forget: note_on, note_off (no ACK, real-time MIDI)

**Server -> Client (State Push):**
- Transport state at 30fps (position, isPlaying)
- Metering at 10fps (levels, peaks)
- All other state on-change only
- Heartbeat every 1s

### Connection Lifecycle

```
1. Client connects to ws://pi:8080
2. Client sends: { "type": "hello", "protocolVersion": "1.0", "clientId": "..." }
3. Server responds: { "type": "welcome", "protocolVersion": "1.0" } or version error
4. Full state push (mixer, transport, loaded song)
5. Continuous state push + command processing
6. Heartbeat every 1s, connection lost after 3s silence
7. Reconnection: session token + lastProcessedSeq for seamless resume
```

### WhiteRoomKit API Mirroring

The target design is WhiteRoomKit parity over WebSocket JSON-RPC. A large
portion of that plumbing exists in the repo, but the system should be described
as pursuing parity rather than already guaranteeing a verified zero-gap mirror.

Client sends:
  { "method": "transport.play", "id": 1 }
  { "method": "mixer.setGain", "params": {"channel": 0, "gain": 0.8}, "id": 2 }
  { "method": "modeler.load", "params": {"path": "/models/amp.nam"}, "id": 3 }

Server responds:
  { "id": 1, "result": true }
  { "id": 2, "result": true }
  { "id": 3, "result": {"kind": "AmpCapture", "cpuClass": "moderate"} }

Handler names use domain.method format matching WhiteRoomEngine sub-API decomposition.
Treat full parity as a release goal, not a completed claim.

### Song Transfer

Large songs use binary frame chunked transfer:

```
1. Client sends: { "type": "load_song_begin", "size": 2400000 }
2. Client sends binary chunks (frames)
3. Client sends: { "type": "load_song_commit" } or { "type": "load_song_rollback" }
4. Song either loads completely or not at all — no partial state
```

### Security

- Command schema validation: reject malformed/unknown commands
- Rate limiting: 100 commands/sec sustained, 200/sec burst per client
- Resource limits: max 10MB message, max 8 connections, 30s idle timeout
- Secure logging: no credentials in logs
- Firewall: nftables policy-drop (only WebSocket + SSH open)

---

## Pipewire Audio Infrastructure

Pipewire runs as the sole audio daemon:

```
Engine (C++) -> Pipewire client -> Pipewire graph
                                      |
                            +---------+----------+
                            |                    |
                      Network audio          Pipewire MIDI
                      (AES67/RTP)        (native ALSA bridge)
```

Pipewire replaces: JACK, a2jmidid, separate ALSA bridges, separate audio routing.

### Audio Path

- Engine runs as Pipewire client at 48kHz
- Buffer size: 512 samples (10.6ms), target 256 (5.3ms)
- Network audio exits via Pipewire's built-in AES67/RTP module
- MIDI enters via Pipewire's native ALSA MIDI bridge (no a2jmidid)

---

## System Image (Buildroot)

### Operating System

Purpose-built embedded Linux via Buildroot:

| Component | Configuration |
|-----------|--------------|
| Kernel | Linux 6.12+ with PREEMPT_RT (CONFIG_PREEMPT_RT=y) |
| Root filesystem | Read-only squashfs + tmpfs overlay |
| CPU isolation | isolcpus=2,3 (cores 2,3 for audio only) |
| IRQ affinity | Network + USB IRQs on cores 0,1 |
| Init system | systemd |
| Audio daemon | Pipewire (sole) |
| Partition layout | A/B with rollback |

### Boot and Recovery

- Boot to audio-ready in < 8 seconds
- Engine crash -> automatic restart within 5 seconds (systemd + BCM2712 watchdog)
- Power yank safe: read-only rootfs survives corruption
- Core dump collection for post-mortem analysis
- State persistence: last song + preferences on separate data partition

### Deployment

```
macOS (development host)
+-------------------------------+
|  Docker container              |
|  (debian:bookworm-slim)        |
|  +-- gcc-aarch64-linux-gnu     |
|  +-- Debian Bookworm sysroot   |
|  +-- CMake + ninja             |
|                                |
|  Output: white-room-engine     |
|  (aarch64 ELF binary)          |
+----------+--------------------+
           | scp / rsync
           v
Pi 5 (target)
+-- Binary dropped in place
+-- systemd restarts service
+-- < 60s iteration cycle
```

Current status note as of August 5, 2026:

- the repo contains `RemoteAudioEngine`, Pi discovery, shared JSON-RPC models, and `pi_port` Pipewire backend code
- at least one release blocker remains around physical Pi 5 qualification and image/service verification
- some frontend remote flows still include simulation/TODO scaffolding while the full path is hardened

---

## Multi-Client Topology

- Multiple Apple devices connect to same Pi simultaneously
- First-connected is master (with auto-promotion on disconnect)
- Pi-authoritative star topology: single Pi = single truth
- mDNS/Bonjour discovery: Pi advertises `_whiteroom._tcp`
- Mac can run engine locally AND control a Pi

---

## Technical Specifications

| Parameter | Value |
|-----------|-------|
| Platform | Pi 5 / CM5 (BCM2712, Cortex-A76 quad-core @ 2.4GHz) |
| SIMD | NEON 128-bit (via Simd4f.h abstraction) |
| Voices | 256-512 (16 channels x 16-32 voices each) |
| Sample rate | 48kHz (configurable 44.1-96kHz) |
| Latency (one-way) | < 10ms over Ethernet (~3.3ms network + buffer) |
| CPU budget | ~50-60% for full engine + effects + network |
| RAM | 4GB minimum, < 2GB engine budget |
| Control | WebSocket JSON + binary song chunks |
| MIDI | Pipewire ALSA MIDI bridge (USB controllers) |
| Network audio bandwidth | ~20 Mbps per direction (16ch x 48kHz x 24-bit) |
| Module cost | ~$97 (CM5 + Ethernet + PCB) |

---

## JUCE Dependencies

The C++ engine retains 7 JUCE modules and removes 2:

| Module | Status | Reason |
|--------|--------|--------|
| `juce_audio_devices` | Removed | Replaced by Pipewire |
| `juce_audio_utils` | Removed | GUI/audio utilities not needed |
| `juce_audio_basics` | Kept | `AudioBuffer<float>`, `MidiBuffer` throughout DSP |
| `juce_core` | Kept | `String`, `CriticalSection`, threading, containers |
| `juce_dsp` | Kept | DSP module abstractions for effect pedals |
| `juce_audio_processors` | Kept | Base class for instrument hierarchy (25+ files) |
| `juce_data_structures` | Kept | `UndoManager`, `ValueTree` for state management |
| `juce_audio_formats` | Kept | `AudioFormatManager` for sample loading |
| `juce_events` | Kept | Message thread, timers |

---

## Real-Time Safety

Enforced rules on the Pi:

- `mlockall()` for audio thread memory
- CPU affinity pinning to isolated cores 2,3
- `SCHED_FIFO` real-time priority
- Zero heap allocations in audio callback (`ScopedNoAllocation` guard)
- Lock-free SPSC queues for MIDI and parameter updates
- `perf record` verifies zero lock acquisitions during audio callback
- Voice stealing with 256-sample minimum fade at CPU thresholds (70% warn, 80% steal, 90% kill)
