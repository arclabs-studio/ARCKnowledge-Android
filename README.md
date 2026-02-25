# ARCKnowledge-Android

**The complete Android development knowledge base for ARC Labs Studio.**

ARCKnowledge-Android guides AI agents (primarily Claude Code) in building high-quality Android applications using Kotlin, Jetpack Compose, and Clean Architecture. It is the Android counterpart to [ARCKnowledge](https://github.com/arclabs-studio/ARCKnowledge) (iOS/Swift).

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
| Language | Kotlin 2.0+ |
| UI | Jetpack Compose + Material Design 3 |
| Architecture | Clean Architecture + MVVM |
| DI | Hilt (Dagger) |
| Networking | Retrofit + OkHttp + Kotlinx Serialization |
| Persistence | Room (database), DataStore (key-value) |
| Testing | JUnit 5 + MockK + Turbine + Compose Testing |
| Linting | ktlint + detekt |
| Build | Gradle with Kotlin DSL + Version Catalogs |
| Concurrency | Kotlin Coroutines + Flow |
| Navigation | Navigation Compose (type-safe) |
| Images | Coil |
| CI/CD | GitHub Actions |

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
│   ├── navigation-compose.md      Type-safe Navigation Compose
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
- Available skills with slash commands
- Critical rules (never break)
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
| Navigation | ARCNavigation (Router) | Navigation Compose (type-safe) |
| Testing | Swift Testing | JUnit 5 + MockK + Turbine |
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
