<p align="center">
  <img alt="Wythnos" width="80" height="80" src="https://avatars.githubusercontent.com/u/248208152?s=200&v=4" />
</p>

<h1 align="center">Wythnos</h1>

<p align="center">
  <b>On-device AI calendar intelligence for iOS.</b><br/>
  Your schedule, analyzed privately — nothing leaves your phone.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/platform-iOS_26+-000000?style=flat-square&logo=apple&logoColor=white" />
  <img src="https://img.shields.io/badge/swift-6.0-F05138?style=flat-square&logo=swift&logoColor=white" />
  <img src="https://img.shields.io/badge/swiftui-declarative_UI-0070E0?style=flat-square&logo=swift&logoColor=white" />
  <img src="https://img.shields.io/badge/AI-on--device_LLM-8B5CF6?style=flat-square&logo=apple&logoColor=white" />
  <img src="https://img.shields.io/badge/privacy-100%25_local-22C55E?style=flat-square&logo=shieldsdotio&logoColor=white" />
  <img src="https://img.shields.io/badge/languages-43-blue?style=flat-square" />
</p>

---

## What is Wythnos?

**Wythnos** (Welsh for "week") is a **privacy-first iOS app** that connects to your calendars and uses **on-device AI** to generate weekly insights, daily briefings, and intelligent chat about your schedule — all without sending a single byte to the cloud.

Think of it as a personal analyst for your time, living entirely on your iPhone.

- 📊 **Weekly Insights** — AI analyzes your past week's events
- 🔮 **Tomorrow Preview** — AI-generated daily briefing  
- 💬 **Calendar Chat** — ask natural language questions about your schedule
- 🔔 **Smart Notifications** — Weekly summaries, meeting reminders with deep links to Zoom/Meet
- 🎯 **Pattern Discovery** — AI learns your habits and surfaces insights
- 🔒 **100% On-Device** — powered by Apple Foundation Models

---

## Core Features

### 🤖 Self-Evolving AI System

**Zero hardcoded prompts.** The AI dynamically assembles responses from a pool of "prompt genes" that evolve based on your feedback:

- **10 gene types**: persona, format, domain instructions, context templates, emotional tone, language mixing, error recovery, evolution directives, insight patterns, creative expression
- **4 evolution stages**: Learning → Adapting → Predicting → Mastered (at 500+ interactions)
- **Continuous improvement**: Low-fitness genes mutate, high-fitness genes proliferate

### 📱 Push Notifications (NEW)

| Type | When | Content |
|------|------|---------|
| **Weekly Summary** | Sunday 6 PM | AI-generated week recap with stats |
| **Week Preview** | Monday 8 AM | Upcoming week briefing |
| **Meeting Reminders** | Before events | Join links for Zoom/Google Meet/Teams |
| **Insight Milestones** | When discovered | Pattern discoveries with 80%+ confidence |

**Deep Link Support:**
- Zoom, Google Meet, Microsoft Teams, Webex, Skype, Jitsi
- Internal: `wythnos://insights`, `wythnos://tomorrow`, `wythnos://chat`

### 💰 Intelligence-Based Monetization

Unlike fixed-day trials, Wythnos uses an **AI maturity model**:

```
Install → Full Pro (silent, day 1)
   ↓
Maturity Trigger (50 interactions OR 3 patterns OR 21 days)
   ↓
48-Hour Grace Period (gentle inline hints)
   ↓
Free Tier (3 chats/day) with inline upgrade cards
```

**Pricing Tiers:**
- **Free**: 3 AI chats/day, basic insights
- **Pro Monthly**: $1.99 — Unlimited chat, deep analysis, all patterns
- **Pro Annual**: $14.99 — 37% savings
- **Lifetime**: $39.99 — Pay once, own forever

### 🎨 Nyx Design System

Dark, futuristic UI inspired by sci-fi interfaces:

- **Dark only**: #000000 backgrounds, no light mode
- **Matrix green**: #00FF88 for AI accents
- **Animations**: Character-by-character typing, number count-ups, spring physics
- **Haptics**: Feedback on every meaningful interaction
- **Custom tab bar**: Centered "W" glyph with glow effect

---

## Architecture

### 15 Modules, Clean Dependencies

```
Wythnos (App)
├── Core Modules (10)
│   ├── ServiceProvider     — Custom DI container
│   ├── WythnosLogger       — Structured logging
│   ├── AnalyticsService    — TelemetryDeck + os.log backends
│   ├── UserProfileService  — Preferences & onboarding state
│   ├── CalendarService     — EventKit abstraction
│   ├── CalendarStore       — CoreData persistence
│   ├── LLMEngine           — Apple Foundation Models + prompt evolution
│   ├── SubscriptionService — StoreKit 2 with TrialManager
│   ├── NotificationService — Local push notifications with deep linking
│   └── DesignSystem        — Nyx tokens, components, animations
└── Feature Modules (5)
    ├── OnboardingUI        — 7-step boot sequence
    ├── InsightsUI          — Weekly AI dashboard with stats
    ├── ChatUI              — Conversational calendar queries
    ├── TomorrowUI          — Next-day AI briefing
    └── SettingsUI          — System settings, subscription, Neural Engine stats
```

### Build System

| | |
|---|---|
| **Build System** | [Tuist](https://tuist.dev) — declarative `Project.swift` manifest |
| **Architecture** | Modular monorepo with explicit dependency graphs |
| **Dependency Injection** | Custom `ServiceProvider` with protocol-based `DependencySpec` |
| **Concurrency** | Swift structured concurrency (`async/await`, `Actor`) throughout |
| **Minimum Target** | iOS 26.0, iPhone only |

---

## Why Apple Foundation Models

Apple Foundation Models wins because:

- **Zero download** — pre-installed on iOS 26+ devices
- **Neural Engine optimized** — A17 Pro delivers ~35 TOPS with mxfp4
- **OS-managed resources** — Apple handles memory, power, thermal
- **Privacy enforced by OS** — not just our promise, Apple's guarantee
- **Free upgrades** — model improves with iOS updates
- **Streaming responses** — token-by-token via `AsyncThrowingStream`

---

## Pattern Evolution Engine

Discovers behavioral patterns from calendar data:

- Meeting frequency trends (increasing/decreasing)
- Day-of-week preferences (busiest/quietest day)
- Time-of-day preferences (morning vs. afternoon person)
- Duration patterns (preference for short meetings)
- Work-life balance signals (weekend events, late meetings)
- Free time erosion (are free blocks shrinking?)

When confidence > 60%, patterns become new prompt genes.

---

## Notification System Architecture

```
┌─────────────────────────────────────────────────────────┐
│              NotificationController                     │
├─────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Scheduler   │  │  ContentGen  │  │ DeepLinkHandler│ │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
         ↓                    ↓                 ↓
    Weekly/              LLM-generated     Zoom/Meet/Teams
    Meeting              personalized      URL scheme
    Insights             text              detection
```

**LLM Content Generation:** Notifications are personalized to user's language and calendar context using the same prompt gene system as chat.

---

## Data Flow

```
EventKit → CalendarService → CalendarSyncManager → CalendarStore (CoreData)
                                                            ↓
                              DynamicPromptEngine ← Prompt Gene Pool
                                                            ↓
                                               Apple Foundation Models
                                                            ↓
                              ┌─────────────────┬──────────┴──────────┐
                              ↓                 ↓                     ↓
                         User-facing      Push Notifications     Pattern
                             UI            (Local, on-device)    Detection
```

- **Full sync:** Last 90 days + next 7 days
- **Incremental sync:** Since last sync + rolling window
- **Background sync:** BGTaskScheduler

---

## On-Device LLM Optimization Techniques

| Category | Technique | Description |
|----------|-----------|-------------|
| **Inference Speed** | Speculative Decoding | 135M draft model, 80%+ acceptance |
| | Continuous Batching | ~40% speedup during idle |
| **Memory** | KV-Cache Reuse | Follow-up queries faster |
| | Flash Attention | O(N) vs O(N²) memory |
| **Quantization** | AWQ/GPTQ | 60% memory savings |
| **Training** | QLoRA | Rank-8 adapters, 1000× smaller |
| | DPO | Direct preference optimization |

---

## Privacy Architecture

- **No network calls** for inference, training, or core analytics
- **CoreData** — local persistence, programmatic schema
- **TelemetryDeck** — privacy-first analytics (EU-hosted, no PII)
- **MetricKit** — crash reporting, no third-party SDK
- **StoreKit 2** — server-free receipt validation
- **EventKit** — data never serialized externally

---

## Technology Stack

### Apple Frameworks
- **SwiftUI** — Declarative UI with `@StateObject`, `@Environment`
- **Foundation Models** — iOS 26+ on-device LLM
- **EventKit** — Native calendar access
- **CoreData** — Local persistence
- **StoreKit 2** — Subscriptions
- **UserNotifications** — Local push notifications
- **MetricKit** — Crash reporting
- **BGTaskScheduler** — Background sync

### Third-Party Dependencies
- **TelemetryDeck** (1 dependency) — Privacy-first analytics

### Data & Persistence
- Programmatic `NSManagedObjectModel` — no `.xcdatamodeld`
- Delta sync with 90-day lookback
- Actor-isolated stores for thread safety

### UI & Design
- **Nyx Design System** — Tokens, components, animations
- Dark-first aesthetic (#000000 backgrounds)
- 43 languages via String Catalogs

### Testing
- **Swift Testing framework** — `@Suite`, `@Test`, `#expect`
- **75+ unit tests** across test targets
- Protocol-based mocking
- In-memory CoreData for tests

---

## The Numbers

| Metric | Value |
|---|---|
| Swift source files | 60+ |
| Core modules | 10 |
| Feature modules | 5 |
| Unit tests | 75+ |
| Languages | 43 |
| Dependencies | 1 (TelemetryDeck) |
| Hardcoded prompts | 0 |
| Network calls for AI | 0 |

---

## Documentation

| Document | Purpose |
|----------|---------|
| [Push Notification Strategy](../../Docs/PushNotificationStrategy.md) | Notification architecture |
| [Notification Integration](../../Docs/NotificationIntegrationGuide.md) | Setup guide |
| [Testing with Mock Data](../../Docs/TestingWithMockData.md) | Debug modes |
| [Monetization Strategy](../../Done/monetization-strategy.md) | Revenue model |
| [Nyx Design System](../../Done/redesign-nyx-design-system.md) | Design specs |
| [Hugging Face Models](../../Done/HUGGING_FACE_MODELS.md) | Vision/LLM options |

---

## LLM Stress Testing & Hardening

Wythnos ships with a **fully automated hardening pipeline** that validates every AI response pattern before production. The pipeline simulates 192 real-world calendar scenarios derived from 5 Hugging Face dataset archetypes and runs 10 validation checks per pair.

### Run it yourself

```bash
cd Wythnos
make harden-llm
```

### Pipeline

```
make transform-data          make test-simulation          make deploy-genes
       ↓                            ↓                            ↓
  Python script              Swift validation engine       Verify gene injection
  generates 192 pairs        scores each pair on 10        into SeedHardeningProvider
  from 5 HF archetypes       dimensions, writes report     and SeedDataLoader
```

### Dataset Coverage (192 pairs)

| Source Archetype | Count | What it tests |
|---|---|---|
| **MeetingBank** | 40 | Dense transcript summarization — budgets, deadlines, action items |
| **MultiWOZ** | 40 | Multi-step scheduling — gap finding, rescheduling, conflict-aware booking |
| **GUM** | 30 | Emotional context — overwhelmed users, work-life balance signals |
| **Taskmaster** | 30 | Natural language booking — "Book a dentist on Friday at 3" |
| **AMI Corpus** | 30 | Pattern recognition — trend detection, week-over-week comparison |
| **Edge Cases** | 10 | Empty calendars, ambiguous names, timezone, off-topic refusal, all-day events, recurring events, back-to-back fatigue, calendar math |
| **Multilingual** | 12 | Turkish, Spanish, French, German, Japanese, Korean, Welsh, Arabic, Dutch, Portuguese, Italian |

### 10 Validation Checks

| # | Check | What it catches |
|---|---|---|
| 1 | `response_nonempty` | Missing or blank responses |
| 2 | `context_nonempty` | Queries without calendar context |
| 3 | `time_grounding` | Responses that don't reference real times from context |
| 4 | `no_hallucinated_names` | Person names invented by the model |
| 5 | `no_empty_cal_hallucination` | Fabricated events on empty calendars |
| 6 | `offtopic_refusal` | Off-topic queries that aren't properly refused |
| 7 | `conflict_flagged` | Overlapping events not flagged with ⚠️ |
| 8 | `empathy_present` | Emotional queries answered without warmth |
| 9 | `locale_match` | Non-English queries answered in English |
| 10 | `contains_numbers` | Analytical responses missing numerical data |

### Results

```
Total:      192 pairs
Passed:     192 (100.0%)

  edgeCase              ████████████████████ 100%
  emotionalContext      ████████████████████ 100%
  eventCreation         ████████████████████ 100%
  meetingIntelligence   ████████████████████ 100%
  multiStepScheduling   ████████████████████ 100%
  multilingual          ████████████████████ 100%
  patternRecognition    ████████████████████ 100%
```

### Feedback Loop

Stress test learnings are fed back into the app's initial seed data via `SeedHardeningProvider` → `SeedDataLoader`, ensuring every install starts with intelligence hardened against known failure modes. This follows the **Teacher-Student Curriculum** pattern from DeepMind research and **RLAIF** from Anthropic.

### Language Intelligence Tiering

The LLM supports 43 UI languages but uses a **tiered intelligence model**:

| Tier | Languages | Strategy |
|---|---|---|
| **Primary** | EN, ES, FR, DE, ZH, JA, KO, PT, IT, NL, TR, AR | Full native prompting |
| **Secondary** | All other 31 languages | Translated Prompting (English reasoning → local output) |

Reports are saved to `Reports/hardening_results.json` (machine-readable) and `Reports/hardening_summary.txt` (human-readable).

---

## We're Looking for Collaborators

**🧠 On-Device ML / LLM Engineers**  
Speculative decoding, QLoRA, KV-cache optimization, prompt evolution

**📱 iOS Engineers**  
SwiftUI, structured concurrency, modular architecture

**🎨 Design Engineers**  
Dark interfaces, motion design, typography systems

**🔬 Research-Oriented Builders**  
Behavioral pattern recognition, calendar-aware attention

---

<p align="center">
  <sub>Built with ❤️ by <a href="https://alp.me">Alp Özcan</a> from Amsterdam</sub><br/>
  <sub>100% on-device · Privacy by architecture · Intelligence-based monetization</sub><br/>
  <sub>Built with <a href="https://pi.dev">pi</a> + Anthropic Claude</sub>
</p>
