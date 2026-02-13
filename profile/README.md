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
  <img src="https://img.shields.io/badge/IPA_size-940_KB-FF6B6B?style=flat-square" />
  <img src="https://img.shields.io/badge/languages-43-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/dependencies-0-brightgreen?style=flat-square" />
</p>

---

## What is Wythnos?

**Wythnos** (Welsh for "week") is a **privacy-first iOS app** that connects to your calendars and uses **on-device AI** to generate weekly insights, daily briefings, and intelligent chat about your schedule — all without sending a single byte to the cloud.

Think of it as a personal analyst for your time, living entirely on your iPhone.

- 📊 **Weekly Insights** — AI analyzes your past week's events
- 🔮 **Tomorrow Preview** — AI-generated daily briefing
- 💬 **Calendar Chat** — ask natural language questions about your schedule
- 🔒 **100% On-Device** — powered by Apple Foundation Models

---

## Sub-1MB Binary

The entire app — 14 modules, 75 tests, 43 languages, a genetic prompt evolution system — ships in a **940 KB IPA**.

| Metric | Size |
|---|---|
| .app bundle (uncompressed) | 6.2 MB |
| IPA (zip compressed) | 940 KB |
| Estimated App Store download | ~1.0–1.5 MB |
| Estimated install size | ~6–7 MB |

For context, Fantastical is ~80 MB. ChatGPT is ~250 MB. Structured is ~30 MB. Apple Calendar is ~15 MB. Wythnos is **50–250× smaller** than its competitors — with a full on-device AI engine inside.

### Where the Space Goes

| Category | Uncompressed | In IPA | % of IPA |
|---|---|---|---|
| Seed training data (10,000 JSONL entries) | 3.6 MB | ~306 KB | 32% |
| Executable code (app + 14 frameworks) | 2.2 MB | ~530 KB | 56% |
| Localization (43 languages × 6 bundles) | 80 KB | ~15 KB | 2% |
| Assets (.car color catalog) | 22 KB | ~5 KB | <1% |

### Why It's So Small

1. **Apple Foundation Models — zero model weight cost.** The AI model is pre-installed on iOS 26+ devices. No model binary to ship.
2. **Zero third-party dependencies.** No Alamofire, no Firebase, no Sentry, no Kingfisher, no Swinject. Pure Apple frameworks.
3. **No image assets.** SF Symbols for iconography, programmatic colors for theming. The `.car` asset catalog is 7 color definitions — 22 KB.

---

## Architecture: 14 Modules, Zero Dependencies

The app is a modular monorepo managed by [Tuist](https://tuist.dev). The `Project.swift` manifest defines everything — no `.xcodeproj` in the repo, no manual Xcode configuration drift.

```
Wythnos (App)
├── Core Modules (9)
│   ├── ServiceProvider     — Custom DI container (NSLock, protocol-based)
│   ├── WythnosLogger       — Structured os.log with level filtering
│   ├── AnalyticsService    — Pluggable analytics (os.log backend, no third-party)
│   ├── UserProfileService  — UserDefaults-backed preferences
│   ├── CalendarService     — EventKit abstraction
│   ├── CalendarStore       — CoreData persistence + delta sync
│   ├── LLMEngine           — Apple Foundation Models + self-evolving prompts
│   ├── SubscriptionService — StoreKit 2
│   └── DesignSystem        — Tokens, components, animations
└── Feature Modules (5)
    ├── OnboardingUI        — 7-step flow with calendar walkthrough
    ├── InsightsUI          — Weekly AI dashboard
    ├── ChatUI              — Conversational calendar queries
    ├── TomorrowUI          — Next-day AI briefing
    └── SettingsUI          — Profile, theme, subscription, about
```

Every module is its own framework target with a matching unit test target. Every commit in the 22-commit history compiles independently. The dependency graph is explicit and enforced by the build system.

### Architecture & Build System

| | |
|---|---|
| **Build System** | [Tuist](https://tuist.dev) — declarative `Project.swift` manifest for reproducible Xcode project generation |
| **Architecture** | Modular monorepo — isolated Core frameworks and Feature modules with explicit dependency graphs |
| **Dependency Injection** | Custom `ServiceProvider` container with protocol-based `DependencySpec` pattern — fully compile-time safe |
| **Concurrency** | Swift structured concurrency (`async/await`, `Actor`, `@Sendable`) throughout the entire stack |
| **Minimum Target** | iOS 26.0, iPhone only |

---

## Why Apple Foundation Models

We evaluated four alternatives before choosing Apple's on-device Foundation Models framework:

| Alternative | Why Not |
|---|---|
| ExecuTorch + Llama 3.2 1B | 500MB+ model binary, complex setup, battery drain |
| llama.cpp + GGUF | CPU-only on iOS, no Neural Engine, must ship model |
| MLC-LLM | Extra compilation step, no App Store precedent |
| Core ML + custom model | ML expertise for conversion, updates require app updates |

Apple Foundation Models wins because:

- **Zero download** — the model is pre-installed on iOS 26+ devices
- **Neural Engine optimized** — A17 Pro delivers ~35 TOPS with mxfp4 hardware support
- **OS-managed resources** — Apple handles memory, power, and thermal budgets
- **Privacy enforced by the OS** — not just our promise, Apple's guarantee
- **Free upgrades** — model improves with every iOS update
- **Streaming responses** — token-by-token generation via `AsyncThrowingStream`

Research basis: [*On-Device LLMs: State of the Union, 2026*](https://v-chandra.github.io/on-device-llms/) by Vikas Chandra & Raghuraman Krishnamoorthi (Meta AI Research).

---

## Self-Evolving Prompt System (Prompt Genes)

**Wythnos has zero hardcoded prompts.** Not a single string literal in the codebase constitutes a prompt to the LLM. Every prompt is dynamically assembled at runtime from composable units called **prompt genes**.

### What is a Prompt Gene?

A `PromptGene` is a Swift struct with:
- **Type** — one of 10 categories: `systemPersona`, `responseFormat`, `domainInstruction`, `contextTemplate`, `evolutionDirective`, `insightPattern`, `emotionalTone`, `languageMixing`, `errorRecovery`, `creativeExpression`
- **Content** — the actual prompt text
- **Fitness Score** — a float (0.0–1.0) updated with exponential moving average based on user feedback
- **Evolution Directive** — instructions that tell the system HOW the gene should mutate

### How Prompt Assembly Works

When the user asks a question, `DynamicPromptEngine.synthesizePrompt()` runs tournament selection across the gene pool:

1. Select the highest-fitness **persona** gene
2. Select the best **response format** gene
3. Select the most relevant **domain instruction** gene (based on intent classification)
4. Fill a **context template** gene with live calendar data
5. Add **emotional tone**, **language mixing**, and **error recovery** genes
6. Attach the current **evolution directive**

10% of the time, a random lower-fitness gene is selected instead — the exploration rate that prevents convergence on a local optimum.

### How Genes Evolve

After every interaction, genes are scored with an exponential moving average:

```swift
gene.fitnessScore = 0.1 * signal + 0.9 * gene.fitnessScore
```

- If fitness drops below **0.3** after 10+ uses → the gene **mutates** (new gene created with modified content based on its evolution directive)
- Bottom **20%** fitness genes get **pruned** after 30 days

The system has four **evolution stages**:

| Stage | Interactions | Behavior |
|---|---|---|
| 1 | 0–50 | **Learning** — language, format, time preferences |
| 2 | 50–200 | **Adapting** — priorities, habits, patterns |
| 3 | 200–500 | **Predicting** — anticipating needs before asked |
| 4 | 500+ | **Mastered** — proactive, personalized, trusted |

The app ships with ~30 seed genes across all 10 types. Each seed gene contains its own evolution directive — **the seeds are designed to be replaced**.

---

## Pattern Evolution Engine

The `PatternEvolutionEngine` discovers behavioral patterns from calendar data and user interactions:

- Meeting frequency trends (increasing/decreasing)
- Day-of-week preferences (busiest/quietest day)
- Time-of-day preferences (morning vs. afternoon person)
- Duration patterns (preference for short meetings)
- Work-life balance signals (weekend events, late meetings)
- Calendar usage distribution (primary vs. secondary calendars)
- Free time erosion (are free blocks shrinking?)

When a pattern is discovered with **>60% confidence**, it's **converted into a new prompt gene** and injected into the gene pool. The app literally creates new prompt intelligence from observed behavior.

---

## Data Flow

```
EventKit (System) → CalendarService → CalendarSyncManager → CalendarStore (CoreData)
                                                                    ↓
                                                           DynamicPromptEngine
                                                                    ↓
                                                      Apple Foundation Models (on-device)
                                                                    ↓
                                                               User-facing UI
```

- **Full sync:** Last 90 days + next 7 days
- **Incremental sync:** Since last sync + rolling 7-day window (on each foreground)
- **Background sync:** `BGTaskScheduler` with registered task identifier

---

## On-Device LLM Optimization Techniques

The architecture is designed to support 15+ optimization techniques as the AI layer matures:

### Inference Speed

| Technique | Description |
|---|---|
| **Speculative Decoding** | 135M draft model generates 5 candidate tokens, verified in one forward pass. Calendar responses have high predictability — acceptance rates can exceed 80%. |
| **Continuous Batching** | Multiple insight queries processed together during idle time for ~40% speedup |
| **KV-Cache Reuse** | Follow-up questions reuse cached KV states from system prompt and calendar context |

### Memory Optimization

| Technique | Description |
|---|---|
| **KV-Cache Paging** | Fixed-size pages from a memory pool to avoid fragmentation, with calendar-recency eviction |
| **Flash Attention** | Tiled attention via Metal Performance Shaders — O(N) memory instead of O(N²) |
| **Grouped-Query Attention (GQA)** | Fewer KV heads than query heads, reducing KV-cache memory by up to 8× |

### Quantization

| Technique | Description |
|---|---|
| **AWQ** (Activation-Aware Weight Quantization) | Preserves salient weight channels at higher precision. Temporal tokens stay at 8-bit. |
| **GPTQ** (Generalized Post-Training Quantization) | One-shot quantization using seed calendar dataset as calibration data |
| **K-Quant (GGUF)** | Per-block scaling with importance-weighted bit allocation. Q4_K_M saves 60% memory with <1% perplexity loss. |

### On-Device Training

| Technique | Description |
|---|---|
| **QLoRA** | Frozen 4-bit base model with trainable rank-8 adapter matrices (~2.5 MB per adapter — 1000× smaller than the model) |
| **Gradient Checkpointing** | Recompute activations during backward pass — 30% more compute, 60% less memory |
| **DPO** (Direct Preference Optimization) | Thumbs up/down pairs train the adapter directly — no reward model needed |

### Domain-Specific

| Technique | Description |
|---|---|
| **Calendar-Aware Attention** | Attention boundaries aligned with event boundaries, not fixed sliding windows |
| **Dynamic Token Budgets** | "next meeting?" gets 256 tokens; "monthly analysis" gets 4096 |
| **Batched Proactive Inference** | Pre-compute insights during idle GPU cycles for instant display |

### Model Lifecycle

| Technique | Description |
|---|---|
| **Model Distillation** | Knowledge transfer from larger teacher models into compact student models optimized for calendar domain |
| **Structured Pruning** | Remove entire channels (not individual weights) while preserving calendar-domain knowledge |
| **Adapter Versioning & Rollback** | QLoRA adapter checkpointing with automatic rollback if adaptation quality degrades |
| **Continuous Learning Pipeline** | Self-evolving prompt engine with evolutionary gene pool — prompts mutate and specialize based on user interaction patterns |

---

## Privacy Architecture

Privacy isn't a feature toggle. It's the architecture.

- **No network calls** for inference, training, or analytics
- **CoreData** for local persistence — programmatic `NSManagedObjectModel`, no `.xcdatamodeld`
- **MetricKit** for crash reporting — Apple-native, no Firebase Crashlytics, no Sentry
- **os.log** for analytics — structured, no Mixpanel, no Amplitude
- **StoreKit 2** for subscriptions — server-free receipt validation
- **EventKit** for calendar access — data never serialized to any external endpoint

The app's `Info.plist` contains two privacy-related keys. That's it.

---

## Technology Stack

### Apple Frameworks

- **SwiftUI** — Declarative UI with `@StateObject`, `@Published`, `@Environment`, `TabView`, `LazyVStack`
- **Foundation Models** (`FoundationModels` framework) — Apple's on-device LLM, iOS 26+, zero model download
- **EventKit** — Native calendar access with full read permissions
- **CoreData** — Local persistence for calendar snapshots, sync state, and training data
- **StoreKit 2** — Modern subscription and in-app purchase handling
- **MetricKit** — Crash reporting and performance metrics via `MXMetricManager` (no third-party crash SDK)
- **os.log** — Unified structured logging across all modules
- **BGTaskScheduler** — Background calendar sync scheduling

### Data & Persistence

- **CoreData** with programmatic `NSManagedObjectModel` — no `.xcdatamodeld` files, fully code-defined schema
- **Calendar delta sync** — 90-day lookback + 7-day forecast, diffed against local store
- **Actor-isolated stores** — `CalendarStore` is a Swift `actor` for thread-safe CoreData access

### UI & Design

- **Custom design system** — `WythnosColors`, `WythnosTypography`, `WythnosSpacing`, `WythnosCornerRadius`
- **Dark-first aesthetic** — black backgrounds, white/purple accents
- **Reusable components** — `WythnosPrimaryButton`, `WythnosTextField`, `WythnosLoadingView`, `WythnosCard`
- **Splash animation** — Pulsing circle → scale transition into 7-step onboarding flow

### Localization

**43 languages at launch.** Xcode String Catalogs (`.xcstrings`) with per-module bundles. Compiler-verified via `String(localized:bundle:)`.

Coverage spans Western Europe, Eastern Europe, Middle East (Arabic, Hebrew, Turkish), South Asia (Hindi, Bengali, Tamil), Southeast Asia (Thai, Vietnamese, Indonesian, Malay, Filipino), East Asia (Japanese, Korean, Chinese Simplified & Traditional), and Africa (Swahili).

### Quality & Testing

- **Swift Testing framework** (`@Suite`, `@Test`, `#expect`) — modern test API, no XCTest
- **75 unit tests** across **14 test targets** — all passing on iOS 26.2 Simulator (iPhone 17 Pro)
- **Protocol-based mocking** — services are testable via protocol conformance, no third-party mocking libraries
- **In-memory CoreData** — tests use `NSInMemoryStoreType` for fast, isolated data layer testing

### CI & Developer Experience

- **Two CI pipelines** on GitHub Actions:
  1. Unit tests — all 14 schemes sequentially on macOS 15
  2. Snapshot tests — matrix strategy across 3 devices (iPhone 17 Pro, Pro Max, 16e)
- Both use concurrency groups to cancel superseded runs and upload test artifacts
- **Tuist project generation** — `tuist generate` for deterministic Xcode project from `Project.swift` manifest
- **No CocoaPods / SPM for dependencies** — zero third-party dependencies, pure Apple frameworks
- **Module-level `Bundle.module`** — each feature module bundles its own resources and localization

---

## The Numbers

| Metric | Value |
|---|---|
| Swift source files | 56 |
| Lines of code | ~4,500 |
| Core modules | 9 |
| Feature modules | 5 |
| Unit tests | 75 (all passing) |
| Test targets | 14 |
| Localization bundles | 6 |
| Languages | 43 |
| CI pipelines | 2 |
| Git commits | 22 (each compiles independently) |
| Third-party dependencies | 0 |
| Hardcoded prompts | 0 |
| Network calls for AI | 0 |
| IPA size | 940 KB |
| Est. App Store download | ~1.0 MB |

---

## We're Looking for Collaborators

Wythnos is ambitious and we're looking for people who want to build something meaningful. If any of these resonate with you, we'd love to talk:

**🧠 On-Device ML / LLM Engineers**
Help us push the boundaries of what's possible with on-device intelligence — speculative decoding, QLoRA fine-tuning, KV-cache optimization, prompt evolution systems.

**📱 iOS Engineers**
SwiftUI, structured concurrency, modular architecture, CoreData — if you care about well-architected iOS apps, this is your playground.

**🎨 Design Engineers**
We're building a dark, minimal, typography-driven interface. If you think about spacing tokens and motion design, come shape the experience.

**🔬 Research-Oriented Builders**
Self-evolving prompt systems, behavioral pattern recognition, calendar-aware attention mechanisms — there's real research happening here.

---

<p align="center">
  <sub>Built with ❤️ by <a href="https://alp.me">Alp Özcan</a> from Amsterdam</sub><br/>
  <sub>100% on-device · Zero third-party dependencies · Privacy by architecture</sub><br/>
  <sub>Built with <a href="https://pi.dev">pi</a> + Anthropic Claude</sub>
</p>
