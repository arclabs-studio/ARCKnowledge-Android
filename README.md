# ARCKnowledge-Android

**The complete Android development knowledge base for ARC Labs Studio.**

ARCKnowledge-Android guides AI agents (primarily Claude Code) in building high-quality Android applications using Kotlin, Jetpack Compose, and Clean Architecture. It is the Android counterpart to [ARCKnowledge](https://github.com/arclabs-studio/ARCKnowledge) (iOS/Swift).

**Version:** 0.2.0 — 2026 stack refresh (see [CHANGELOG](CHANGELOG.md)).

---

## Philosophy

1. **Simple, Lovable, Complete** — Every feature should be intuitive, delightful, and fully realized
2. **Quality Over Speed** — Write code that lasts, not code that works once
3. **Modular by Design** — Build reusable components that serve multiple projects
4. **Professional Standards** — Indie doesn't mean amateur; maintain enterprise-level quality
5. **Native First** — Leverage Android Jetpack and Material Design 3 before external dependencies

## Tech Stack

| Category | Technology |
|----------|-----------|
| Language | Kotlin 2.4.21 (K2) |
| UI | Jetpack Compose (BOM 2026.09.00) + Material 3 Expressive (material3 1.5.0-beta01) |
| Architecture | Clean Architecture + MVVM |
| DI | Hilt 2.60.1 + KSP 2.3.12 (no kapt) |
| Backend / Networking | Firebase SDKs (BoM 35.0.2) — no HTTP client in v1; Retrofit 3 + OkHttp 5 + kotlinx.serialization when a Vapor backend arrives |
| Persistence | Room 3 (`androidx.room3` 3.0.3), DataStore 1.2.1 |
| Testing | JUnit 5 (mannodermaus plugin) + fakes + Turbine + Robolectric + Compose Preview Screenshot Testing |
| Linting | ktlint 1.8 + detekt 1.23.8 + Android Lint (zero warnings) |
| Build | Gradle 9.8.2 + AGP 9.4.0, Kotlin DSL, Version Catalogs, `build-logic/` convention plugins |
| Concurrency | Kotlin Coroutines + Flow |
| SDK | compileSdk / targetSdk 36, minSdk 26 |
| Navigation | Navigation 3 (`NavDisplay` + `@Serializable` `NavKey`s) |
| Images | Coil 3 |
| CI/CD | GitHub Actions |

> Full version table: [Tools/gradle.md → ARC Labs 2026 Stack](Tools/gradle.md#-arc-labs-2026-stack-reference-table).

## Repository Structure

```
ARCKnowledge-Android/
├── CLAUDE.md                      ← Entry point for AI agents
├── README.md
├── LICENSE
├── CHANGELOG.md
├── .gitignore
├── .kotlin-version
│
├── Architecture/
│   ├── clean-architecture.md      Clean Architecture layers & boundaries
│   ├── mvvm.md                    MVVM pattern (ViewModel + StateFlow)
│   ├── solid-principles.md        SOLID with Kotlin examples
│   ├── dependency-injection.md    Hilt DI patterns
│   ├── navigation-3.md            Navigation 3 (NavDisplay, NavKey, scenes)
│   └── singletons.md              When & how to use singletons
│
├── Layers/
│   ├── presentation.md            Composables, ViewModels, StateFlow
│   ├── domain.md                  Entities, Use Cases, Repository interfaces
│   └── data.md                    Retrofit, Room, DataStore, DTOs
│
├── Quality/
│   ├── testing.md                 JUnit 5, MockK, Turbine, Compose testing
│   ├── code-style.md              ktlint + detekt configuration
│   ├── code-review.md             Review checklist & process
│   ├── compose-performance.md     Recomposition, stability, optimization
│   ├── documentation.md           KDoc + Dokka
│   ├── module-structure.md        Gradle multi-module organization
│   ├── readme-standards.md        README templates & requirements
│   └── ui-guidelines.md           Material Design 3, accessibility, localization
│
├── Tools/
│   ├── gradle.md                  Gradle Kotlin DSL, version catalogs, tasks
│   ├── android-studio.md          IDE configuration & debugging
│   ├── arcdevtools-android.md     Centralized tooling & CI/CD
│   ├── mcp-setup.md               MCP server configuration
│   └── context7-usage.md          Context7 documentation query patterns
│
├── Projects/
│   ├── apps.md                    Android app standards & patterns
│   └── libraries.md               Android library development guide
│
├── Workflow/
│   ├── git-commits.md             Conventional Commits standards
│   ├── git-branches.md            Git Flow branch naming
│   └── plan-mode.md               Structured planning for complex tasks
│
└── Skills/
    └── skills-index.md            AI agent skill routing & discovery
```

## Installation

### As a Git Submodule (Recommended)

```bash
# Add to your Android project
git submodule add https://github.com/arclabs-studio/ARCKnowledge-Android.git ARCKnowledge

# Initialize after cloning
git submodule update --init --recursive
```

### Direct Clone

```bash
git clone https://github.com/arclabs-studio/ARCKnowledge-Android.git
```

## Usage

### For AI Agents (Claude Code)

The entry point is **CLAUDE.md**. It provides:
- Core philosophy and values
- Skill routing (ARC Android skills + the official `android/skills` plugin)
- Critical rules and binding rules (zero warnings, third-party allowlist, modern APIs, secrets, tooling policy, tests)
- Quick architecture reference with code examples
- Code style and testing essentials

**Progressive disclosure**: Start with CLAUDE.md. Load specific documents only when needed for a task.

### For Developers

Browse the documentation by category:
1. **Architecture/** — How to structure code
2. **Layers/** — What goes where
3. **Quality/** — How to maintain quality
4. **Tools/** — How to use the toolchain
5. **Workflow/** — How to collaborate

## Key Differences from ARCKnowledge (iOS)

| Concept | iOS (ARCKnowledge) | Android (ARCKnowledge-Android) |
|---------|-------------------|-------------------------------|
| UI Framework | SwiftUI | Jetpack Compose |
| State Management | @Observable | ViewModel + StateFlow |
| DI | Manual (init params) | Hilt (@Inject) |
| Navigation | `NavigationStack` + `navigationDestination` | Navigation 3 (`NavDisplay` + `NavKey`) |
| Testing | Swift Testing | JUnit 5 + fakes + Turbine |
| Linting | SwiftLint + SwiftFormat | ktlint + detekt |
| Build System | SPM (Package.swift) | Gradle (build.gradle.kts) |
| Documentation | DocC | Dokka |

## Contributing

1. Create a branch: `docs/description`
2. Make changes following existing document patterns
3. Ensure all cross-references are valid
4. Submit a PR for review

### Document Format

Every document follows this structure:
- Emoji + Title (`# 🎯 Title`)
- Bold description + horizontal rule
- Sections with emojis
- ✅/❌ for correct/incorrect patterns
- ```kotlin code blocks with full examples
- Checklist section
- Common Mistakes with ❌/✅ examples
- Further Reading with relative links

## License

MIT License — see [LICENSE](LICENSE) for details.

---

**ARC Labs Studio** — Building delightful apps, one module at a time. 🚀
