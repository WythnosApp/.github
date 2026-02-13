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
</p>

---

## What is Wythnos?

Wythnos is a **privacy-first iOS app** that connects to your calendars and uses **on-device AI** to generate weekly insights, daily briefings, and intelligent chat about your schedule — all without sending a single byte to the cloud.

Think of it as a personal analyst for your time, living entirely on your iPhone.

---

## How It's Built

Wythnos is a **modular, production-grade iOS application** built with modern Apple technologies and a strict separation of concerns.

### Architecture & Build System

| | |
|---|---|
| **Build System** | [Tuist](https://tuist.dev) — declarative `Project.swift` manifest for reproducible Xcode project generation |
| **Architecture** | Modular monorepo — isolated Core frameworks and Feature modules with explicit dependency graphs |
| **Dependency Injection** | Custom `ServiceProvider` container with protocol-based `DependencySpec` pattern — fully compile-time safe |
| **Concurrency** | Swift structured concurrency (`async/await`, `Actor`, `@Sendable`) throughout the entire stack |
| **Minimum Target** | iOS 26.0, iPhone only |

### Core Modules

| Module | Responsibility |
|---|---|
| `ServiceProvider` | DI container — protocol-based service registration and resolution |
| `WythnosLogger` | Structured logging via `os.log` with level-based filtering |
| `AnalyticsService` | Privacy-respecting, pluggable analytics backend (os.log based — no third-party SDKs) |
| `CalendarService` | EventKit abstraction — authorization flow, event fetching, permission handling |
| `CalendarStore` | CoreData persistence layer for calendar snapshots with delta sync |
| `LLMEngine` | On-device LLM inference via Apple Foundation Models framework |
| `SubscriptionService` | StoreKit 2 integration for subscription management |
| `UserProfileService` | Local user preferences and onboarding state via UserDefaults |
| `DesignSystem` | Shared design tokens (colors, typography, spacing, corner radii) and reusable components |

### Feature Modules

| Feature | What it does |
|---|---|
| `OnboardingUI` | Multi-step onboarding flow with calendar permission request and splash animation |
| `InsightsUI` | AI-generated weekly summary — meeting hours, busiest day, category breakdown, pattern detection |
| `TomorrowUI` | Tomorrow's schedule preview with AI-powered daily briefing |
| `ChatUI` | Conversational interface to ask anything about your schedule history |
| `SettingsUI` | User profile, theme preferences, subscription management |

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

### On-Device AI

Wythnos uses **Apple's Foundation Models framework** for all AI features:

- **Zero model download** — the system model is pre-installed on-device
- **Neural Engine optimized** — hardware-accelerated inference on Apple silicon
- **4-bit quantization** — mxfp4 hardware support for efficient on-device inference
- **Streaming responses** — token-by-token generation via `AsyncThrowingStream`
- **OS-managed resources** — Apple handles memory and power budgets automatically
- **Complete privacy** — all inference happens locally, no network calls

### On-Device LLM Optimization Techniques

The project is architected to support advanced on-device optimization as the AI layer evolves:

| Technique | Description |
|---|---|
| **Mixed-Precision Quantization (AWQ/GPTQ)** | Activation-aware weight quantization — preserving salient weights (top 1% by activation magnitude) at higher precision while aggressively quantizing the rest |
| **Speculative Decoding** | Tiny draft model generates candidate tokens, verified in parallel by the main model for 2-3× speedup |
| **KV-Cache Paging** | Fixed-size page allocation from a memory pool to avoid fragmentation, with calendar-recency eviction |
| **Flash Attention** | Tiled attention computation via Metal Performance Shaders — O(N) memory instead of O(N²) |
| **Grouped-Query Attention (GQA)** | Fewer KV heads than query heads, reducing KV-cache memory by up to 8× |
| **QLoRA On-Device Fine-Tuning** | Quantized Low-Rank Adaptation — frozen 4-bit base model with trainable rank-8 adapter matrices (~2.5 MB per adapter) |
| **Structured Pruning** | Remove entire channels (not individual weights) while preserving calendar-domain knowledge |
| **Calendar-Aware Attention** | Attention boundaries aligned with calendar event boundaries instead of fixed sliding windows |
| **Dynamic Token Budget Allocation** | Intelligent context sizing per query type — "next meeting?" gets 256 tokens, "monthly analysis" gets 4096 |
| **Batched Proactive Inference** | Multiple analytical queries processed in a single forward pass during idle GPU time |
| **Model Distillation** | Knowledge transfer from larger teacher models into compact student models optimized for calendar domain |
| **Gradient Checkpointing** | Recompute activations during backward pass — trades 30% compute for 60% less memory during on-device training |
| **KV-Cache Reuse** | Cached KV states from system prompt and calendar context are reused across follow-up questions |
| **Continuous Learning Pipeline** | Self-evolving prompt engine with evolutionary gene pool — prompts mutate and specialize based on user interaction patterns |
| **Adapter Versioning & Rollback** | QLoRA adapter checkpointing with automatic rollback if adaptation quality degrades |

### Data & Persistence

- **CoreData** with programmatic `NSManagedObjectModel` — no `.xcdatamodeld` files, fully code-defined schema
- **Calendar delta sync** — 90-day lookback + 7-day forecast, diffed against local store
- **Actor-isolated stores** — `CalendarStore` is a Swift `actor` for thread-safe CoreData access

### UI & Design

- **Custom design system** — `WythnosColors`, `WythnosTypography`, `WythnosSpacing`, `WythnosCornerRadius`
- **Dark-first aesthetic** — black backgrounds, white/purple accents
- **Reusable components** — `WythnosPrimaryButton`, `WythnosTextField`, `WythnosLoadingView`, `WythnosCard`
- **Localization-ready** — `.xcstrings` catalogs per feature module with `String(localized:bundle:)` API
- **Splash animation** — Pulsing circle → scale transition into onboarding flow

### Quality & Testing

- **Swift Testing framework** (`@Suite`, `@Test`, `#expect`) — modern test API, no XCTest
- **Unit tests per module** — every Core and Feature module has its own test target
- **Protocol-based mocking** — services are testable via protocol conformance, no third-party mocking libraries
- **In-memory CoreData** — tests use `NSInMemoryStoreType` for fast, isolated data layer testing

### CI & Developer Experience

- **Tuist project generation** — `tuist generate` for deterministic Xcode project from `Project.swift` manifest
- **No CocoaPods / SPM for dependencies** — zero third-party dependencies, pure Apple frameworks
- **Module-level `Bundle.module`** — each feature module bundles its own resources and localization

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
  <sub>Built with ❤️ by <a href="https://alp.me">Alp Özcan</a></sub><br/>
  <sub>100% on-device · Zero third-party dependencies · Privacy by architecture</sub>
</p>
