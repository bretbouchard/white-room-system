# ML — Machine Learning

ML assistance for White Room.

---

## What This Is

The ML layer provides assistance for music composition:

- **Composition suggestions** — Ideas when you're stuck
- **Audio analysis** — Understand existing audio
- **Style modeling** — Learn from reference tracks
- **Task orchestration** — Coordinate multiple ML tasks

---

## ML Architecture

### Supervisor Pattern

```
                    ┌─────────────────────┐
                    │   Supervisor Agent   │
                    │                     │
                    │  - Task routing     │
                    │  - Result synthesis │
                    │  - Error handling   │
                    └──────────┬──────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
           ▼                   ▼                   ▼
    ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
    │  Composition │   │   Analysis   │   │   Style      │
    │    Agent     │   │    Agent     │   │   Agent      │
    └──────────────┘   └──────────────┘   └──────────────┘
           │                   │                   │
           ▼                   ▼                   ▼
    ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
    │  Generation  │   │  Extraction  │   │   Transfer   │
    │    Tools     │   │    Tools     │   │    Tools     │
    └──────────────┘   └──────────────┘   └──────────────┘
```

---

## Composition Agent

### Capabilities

| Task | Description |
|------|-------------|
| Generate rhythm | Create rhythmic patterns from description |
| Generate melody | Create melodic lines with constraints |
| Generate harmony | Create chord progressions |
| Generate bass | Create bass lines |
| Suggest orchestration | Recommend instrument assignments |

### Input Format

```json
{
  "task": "generate_melody",
  "context": {
    "key": "C major",
    "tempo": 120,
    "style": "pop",
    "range": ["C4", "C5"],
    "length_bars": 8
  },
  "constraints": {
    "max_interval": 5,
    "avoid_repetition": true,
    "contour": "arch"
  }
}
```

### Output Format

```json
{
  "result": {
    "notes": [
      {"pitch": 60, "start": 0.0, "duration": 0.5, "velocity": 0.8},
      {"pitch": 64, "start": 0.5, "duration": 0.25, "velocity": 0.7},
      {"pitch": 67, "start": 0.75, "duration": 0.5, "velocity": 0.9}
    ],
    "explanation": "Arch-shaped melody starting on C4, peaking at G4"
  }
}
```

---

## Analysis Agent

### Capabilities

| Task | Description |
|------|-------------|
| Pitch detection | Identify fundamental frequency |
| Chord recognition | Identify chord symbols |
| Key detection | Determine musical key |
| Tempo estimation | Detect BPM |
| Section detection | Find song sections |

### Audio Input

```json
{
  "task": "analyze_audio",
  "audio": {
    "sample_rate": 44100,
    "duration": 30.0,
    "format": "float32"
  },
  "analysis_type": ["key", "tempo", "chords"]
}
```

### Analysis Output

```json
{
  "result": {
    "key": {
      "tonic": "C",
      "mode": "major",
      "confidence": 0.92
    },
    "tempo": {
      "bpm": 120,
      "confidence": 0.95
    },
    "chords": [
      {"symbol": "C", "start": 0.0, "duration": 4.0},
      {"symbol": "Am", "start": 4.0, "duration": 4.0},
      {"symbol": "F", "start": 8.0, "duration": 4.0},
      {"symbol": "G", "start": 12.0, "duration": 4.0}
    ]
  }
}
```

---

## Style Agent

### Capabilities

| Task | Description |
|------|-------------|
| Learn style | Extract patterns from reference |
| Apply style | Transfer learned style to new material |
| Blend styles | Combine multiple style references |

### Style Learning

```json
{
  "task": "learn_style",
  "references": [
    {"audio": "reference1.wav", "weight": 1.0},
    {"audio": "reference2.wav", "weight": 0.5}
  ],
  "aspects": ["rhythm", "melody", "harmony"]
}
```

### Style Application

```json
{
  "task": "apply_style",
  "style_id": "learned_style_123",
  "content": {
    "melody": [60, 64, 67, 72],
    "rhythm": [1.0, 0.5, 0.5, 1.0]
  },
  "strength": 0.7
}
```

---

## Tool Registry

### Available Tools

| Tool | Agent | Description |
|------|-------|-------------|
| `generate_rhythm` | Composition | Create rhythmic pattern |
| `generate_melody` | Composition | Create melodic line |
| `generate_harmony` | Composition | Create chord progression |
| `analyze_key` | Analysis | Detect musical key |
| `analyze_chords` | Analysis | Recognize chords |
| `analyze_tempo` | Analysis | Estimate tempo |
| `learn_style` | Style | Extract style patterns |
| `apply_style` | Style | Transfer style |

### Tool Contract

Each tool follows the MCP (Model Context Protocol) format:

```typescript
interface Tool {
  name: string;
  description: string;
  inputSchema: JSONSchema;
  outputSchema: JSONSchema;
}

interface ToolCall {
  tool: string;
  input: object;
}

interface ToolResult {
  success: boolean;
  output?: object;
  error?: string;
}
```

---

## Integration with White Room

### User Interactions

```
User: "Make the melody more interesting"
        │
        ▼
┌─────────────────────┐
│   Supervisor Agent  │
└──────────┬──────────┘
           │
           ▼
    Understanding: "vary_melody"
           │
           ▼
┌─────────────────────┐
│  Composition Agent  │
│                     │
│  1. Analyze current │
│  2. Identify "flat" │
│     sections        │
│  3. Generate        │
│     variations      │
│  4. Return options  │
└──────────┬──────────┘
           │
           ▼
    UI shows 3 options:
    - Add passing tones
    - Vary rhythm
    - Change contour
           │
           ▼
    User selects → Applied
```

### Context Management

The ML system maintains context about the current composition:

```json
{
  "song_context": {
    "key": "C major",
    "tempo": 120,
    "time_signature": "4/4",
    "sections": ["verse", "chorus", "bridge"],
    "current_section": "chorus",
    "instruments": ["drums", "bass", "chords", "lead"],
    "recent_changes": [
      {"action": "generate_melody", "timestamp": "2024-01-15T10:30:00Z"}
    ]
  }
}
```

---

## ML Models

### Model Types

| Model | Purpose | Size |
|-------|---------|------|
| Melody Generator | Create melodic lines | ~50M params |
| Harmony Predictor | Predict chord progressions | ~30M params |
| Style Encoder | Extract style embeddings | ~100M params |
| Audio Analyzer | Pitch/chord detection | ~20M params |

### Inference Requirements

| Metric | Target |
|--------|--------|
| Latency (simple) | < 100ms |
| Latency (complex) | < 2s |
| Memory | < 500MB |
| GPU | Optional (CPU fallback) |

---

## Error Handling

### Fallback Behavior

```
Agent Request
     │
     ▼
┌────────────────┐
│ Try ML Solution │
└───────┬────────┘
        │
        ▼
    Success? ──Yes──▶ Return Result
        │
        No
        │
        ▼
┌────────────────┐
│ Fallback:      │
│ Rule-based     │
│ Generation     │
└───────┬────────┘
        │
        ▼
    Return Safe Result
```

### Error Types

| Error | Handling |
|-------|----------|
| Model not loaded | Use rule-based fallback |
| Timeout | Return partial result |
| Invalid input | Validate and correct |
| Out of memory | Clear cache, retry |

---

## Privacy

### Data Handling

- **User compositions**: Processed locally, not stored
- **Audio analysis**: On-device, no cloud upload
- **Style learning**: Patterns extracted, source discarded
- **Telemetry**: Opt-in, anonymized

### Local Processing

All ML inference runs locally on device:
- No API calls to external services
- No user data leaves the device
- Models run on-device (CoreML on Apple)

---

## Future Capabilities

| Feature | Status |
|---------|--------|
| Real-time collaboration | Planned |
| Voice commands | Research |
| Gesture control | Research |
| Emotional analysis | Research |
