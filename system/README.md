# System

The application layer. Frontend, Intelligence, and ML combined.

---

## What This Is

The System pillar is everything that makes White Room an app:

- **Frontend** — The Swift application
- **Intelligence** — Multi-agent generative composition (BettaFish-MiroFish)
- **ML** — Machine learning assistance

---

## Layers

### [Frontend](./frontend/) — Application Layer

The Swift application that brings everything together:

- SwiftUI interface
- XCFramework integration
- Platform support (iOS, macOS, tvOS, visionOS)

### [Intelligence](./intelligence/) — BettaFish-MiroFish Layer

Multi-agent generative composition system:

- **Forum Engine** — Multi-agent deliberation (6 specialist agents)
- **Simulation Engine** — Temporal state evolution
- **Musical Actor Agents** — Kick, Bass, Harmony, Lead, Counterline, Texture
- **Renderer/Realizer** — Simulation to musical output

### [ML](./ml/) — Machine Learning

ML assistance for composition:

- Composition suggestions
- Audio analysis
- Style modeling

---

## Relationship

```
FRONTEND          INTELLIGENCE              ML
────────          ───────────               ──
User interface    Generative core           Optional assistance
Swift, SwiftUI    TypeScript, Zod           Python, ML models
Platform layer    Multi-agent system        Analysis models
```

Frontend is the app. Intelligence is the generative core. ML is optional assistance.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND                              │
│  SwiftUI + Combine + AVFoundation + XCFramework             │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                     INTELLIGENCE                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Intent → Agents → Forum Engine → Simulation Engine  │    │
│  │         → Actor Agents → Renderer                   │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                          ML                                  │
│  Python models for optional composition assistance           │
└─────────────────────────────────────────────────────────────┘
```
