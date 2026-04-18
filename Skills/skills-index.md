# Skills Index

**Skills are knowledge containers that provide specialized guidance for specific tasks. Agents are autonomous executors that use tools to complete workflows. This index helps you find the right resource.**

---

## Quick Decision Guide

```
Need to...                                 Use
-------------------------------------------------------------------
Set up a new project/module                /arc-project-setup
Design architecture for a feature          /arc-android-architecture
Write Composables or ViewModels            /arc-presentation-layer
Implement API calls, Room, DataStore       /arc-data-layer
Write or review tests                      /arc-tdd-patterns
Review code quality, lint, docs            /arc-quality-standards
Final review before merge                  /arc-final-review
Audit entire project quality               /arc-audit
Make commits, create PRs                   /arc-workflow
Set up CI/CD workflows                     /arc-github-actions-ci
Project memory across sessions             /arc-memory
Parallel feature development               /arc-worktrees-workflow

Implement a feature with TDD (delegated)   → arc-kotlin-tdd agent
Review code (delegated)                    → arc-kotlin-reviewer agent
Debug a build/test failure                 → arc-kotlin-debugger agent
Manage Gradle dependencies                 → arc-gradle-manager agent
Navigate the codebase                      → arc-project-explorer agent
Start a Linear ticket                      → arc-linear-bridge agent
Publish a PR                               → arc-pr-publisher agent
Prepare a release                          → arc-release-orchestrator agent
```

---

## ARC Labs Skills (Internal Standards)

Skills live in `.claude/skills/<name>/SKILL.md` and are symlinked by ARCDevTools-Android.

### /arc-android-architecture
**Purpose**: Clean Architecture, MVVM, Hilt DI, Navigation Compose

- Clean Architecture layer rules (Presentation → Domain ← Data)
- ViewModel + StateFlow + sealed interface UiState
- Hilt module organization (@HiltViewModel, @Module, @Provides, @Binds)
- Type-safe Navigation Compose 2.8+ with @Serializable routes
- SOLID principles with Kotlin examples, dependency rule enforcement

**References**: Architecture/clean-architecture.md, mvvm.md, solid-principles.md, dependency-injection.md, navigation-compose.md

---

### /arc-presentation-layer
**Purpose**: Composables, @HiltViewModel, StateFlow, Material Design 3

- Screen/Content composable split pattern
- `collectAsStateWithLifecycle` (not `collectAsState`)
- Material Design 3 dynamic color + dark theme
- Compose performance: @Stable/@Immutable, derivedStateOf, remember
- Accessibility: contentDescription, Modifier.semantics, 48dp touch targets
- Clarification: `remember{}` for ephemeral UI state; business state in ViewModel

**References**: Layers/presentation.md

---

### /arc-data-layer
**Purpose**: Repositories, Retrofit, Room, DataStore, DTOs

- Repository pattern with @Binds Hilt module
- Retrofit + kotlinx.serialization + OkHttp interceptors
- Room: @Entity, @Dao, @Database, migrations, exportSchema = true
- DataStore Preferences + error handling
- DTO/Entity/mapper separation, network-first vs cache-first caching
- Sealed DomainError hierarchy

**References**: Layers/data.md

---

### /arc-tdd-patterns
**Purpose**: JUnit 5, MockK, Turbine, TDD workflow

**Stack (mandatory — never substitute)**:
- JUnit 5: `@Test`, `@Nested`, `@DisplayName` — never JUnit 4
- MockK: `every {}`, `coEvery {}`, `verify {}`, `coVerify {}`
- Turbine: `.test { awaitItem() }`, `cancelAndIgnoreRemainingEvents()`
- `runTest` + `StandardTestDispatcher` / `UnconfinedTestDispatcher`
- `createSUT()` factory pattern, Given-When-Then structure

**References**: Quality/testing.md

---

### /arc-quality-standards
**Purpose**: 9-domain review checklist, ktlint/detekt, accessibility, M3

- Architecture, Presentation, Domain, Data, Testing, Code Style, Documentation, Accessibility, Concurrency
- ktlint: `./gradlew ktlintCheck` / `./gradlew ktlintFormat`
- detekt: `./gradlew detekt`
- WCAG AA compliance, 48dp touch targets, edge-to-edge with WindowInsets
- KDoc standards for public APIs

**References**: Quality/code-review.md, code-style.md, documentation.md, readme-standards.md, module-structure.md, ui-guidelines.md, compose-performance.md

---

### /arc-workflow
**Purpose**: Git commits, branches, PRs, Plan Mode

- Conventional Commits: `feat(compose): add restaurant card`
- Branch naming: `feature/ARC-123-description`
- PR template with summary, type, testing, checklist
- Plan Mode: when and how to use
- Pre-PR gates: `./gradlew ktlintCheck`, `./gradlew test`

**References**: Workflow/git-commits.md, git-branches.md, plan-mode.md

---

### /arc-project-setup
**Purpose**: Version catalog, Gradle templates, multi-module, CI/CD

- Complete `libs.versions.toml` with 2025 versions
- `settings.gradle.kts` + `build.gradle.kts` templates (app, feature, library)
- Multi-module structure (`:app`, `:feature:*`, `:core:*`)
- ARCDevTools-Android git submodule integration
- Gradle performance: configuration cache, build cache, maxParallelForks

**References**: Tools/gradle.md, arcdevtools-android.md, android-studio.md, Projects/apps.md, libraries.md

---

### /arc-final-review
**Purpose**: Guided pre-merge review (you conduct it with Claude's help)

- 8-step process: diff → categorize → domain checklists → verify
- Verification gates: `./gradlew test ktlintCheck detekt lint`
- Output format: 🔴 Critical / 🟡 Important / 🔵 Improvements

> **Distinction**: For fully autonomous review, use the `arc-kotlin-reviewer` agent instead.

---

### /arc-audit
**Purpose**: Full project health check with A-F compliance grading

- 9 domains: Architecture (20pts), Testing (20pts), Presentation (15pts), Data (15pts), Code Style (10pts), Domain (10pts), Documentation (5pts), Accessibility (5pts), Concurrency (flags)
- Grading: A = 90-100 / B = 75-89 / C = 60-74 / D = 45-59 / F = <45
- Run quarterly, pre-release, or when onboarding to a project

---

### /arc-github-actions-ci
**Purpose**: GitHub Actions CI/CD for Android builds and releases

- `android-test.yml` — JUnit test runner with Gradle caching
- `android-lint.yml` — ktlintCheck + detekt + lint
- `android-release.yml` — signed AAB build from GitHub Secrets
- `android-distribute.yml` — Firebase App Distribution
- Versioning strategies, secrets reference table

---

### /arc-memory
**Purpose**: Persistent context across Claude Code sessions

- `/memory` directory structure (README, ARCHITECTURE, DECISIONS, PATTERNS, DEPENDENCIES, features/)
- Session start/end workflow
- CLAUDE.md integration
- Templates in `.claude/skills/arc-memory/templates/`

---

### /arc-worktrees-workflow
**Purpose**: Parallel feature development with git worktrees

- Worktree creation with submodule initialization
- Shell aliases: `zwn ARC-123`, `zwr ARC-123`, `zwl`
- Maximum 3 active worktrees
- Cleanup after PR merge

---

### /arc-android-security-audit
**Purpose**: OWASP Mobile Top 10 vulnerability scanner for Android apps

- Context detection (manifest, deps, permissions → identify risk profile and active verticals)
- 10 universal categories: Storage, Network/TLS, IPC, Auth, Crypto, WebView, Logging, Binary, UI, Supply Chain
- 4 vertical checklists: 🏦 Fintech, 🏥 Health/HIPAA, 🏢 Enterprise/MDM, 🛒 E-commerce
- Reporting: 🔴 Critical / 🟡 High / 🔵 Info (aligned with `/arc-final-review`)
- Kotlin 2.0 + Compose + Hilt + Room remediation examples

> **Adapted from** `MartinPSDev/skills/security/bug-hunter` (Apache-2.0). Attribution preserved in skill frontmatter.

---

### /arc-android-coroutines
**Purpose**: Kotlin Coroutines structured concurrency — scopes, dispatchers, cancellation, exception handling, Flow, testing

- CoroutineScope hierarchy and structured concurrency rules (never GlobalScope in production)
- Dispatchers (Main, IO, Default, Unconfined) — correct usage per layer; main-safe suspend functions
- Cancellation propagation: cooperative loops, NonCancellable cleanup, withTimeout semantics
- Exception handling: SupervisorJob, CoroutineExceptionHandler, launch vs async exception propagation
- Channels and Flow (cold vs hot, StateFlow, SharedFlow, collectLatest, produce)
- Android lifecycle integration (viewModelScope, lifecycleScope, repeatOnLifecycle, flowWithLifecycle)
- Testing with runTest, StandardTestDispatcher, advanceTimeBy, setMain/resetMain
- 32-entry triage playbook (topic/error → reference file)

**References**: 32 structured practice files in `references/`

> **Adapted from** `santimattius/structured-coroutines/kotlin-coroutines-skill` (MIT). Attribution preserved in skill frontmatter.

---

## ARC Labs Agents

Agents autonomously execute complete workflows. See [AGENTS.md](../AGENTS.md) for full documentation.

| Agent | Purpose | Model | R/O |
|-------|---------|-------|-----|
| `arc-kotlin-tdd` | TDD feature implementation | Sonnet | No |
| `arc-kotlin-reviewer` | 9-domain code review report | Sonnet | Yes |
| `arc-kotlin-debugger` | Build/test failure diagnosis | Sonnet | No |
| `arc-gradle-manager` | Gradle dependency management | Haiku | No |
| `arc-project-explorer` | Codebase navigation | Haiku | Yes |
| `arc-linear-bridge` | Ticket → branch + test skeleton | Haiku | No |
| `arc-pr-publisher` | PR creation + Linear update | Sonnet | No |
| `arc-release-orchestrator` | Version bump + release branch | Sonnet | No |
| `arc-play-distribution` | Firebase App Distribution | Haiku | No |
| `arc-play-store-listing` | Play Store ASO | Sonnet | No |
| `arc-room-migration` | Room schema migrations | Sonnet | No |
| `arc-dependency-auditor` | Dependency health audit | Haiku | Yes |
| `arc-kotlin-security-auditor` | OWASP Mobile security audit | Sonnet | Yes |

---

## Community Skills Integration (Audited)

These external skills were evaluated and selectively absorbed into ARC Labs skills. **Do not install them wholesale** — the following have been audited for conflicts.

### What We Absorbed

| Source | Content Absorbed | Into |
|--------|-----------------|------|
| `compose-skill` (aldefy) | 13 Compose reference guides (recomposition, modifiers, performance, theming) | `arc-android-presentation-layer` |
| `awesome-android-agent-skills` (new-silvermoon) | compose-navigation, compose-performance-audit, android-coroutines, android-retrofit, coil-compose, android-gradle-logic | Multiple skills |
| `claude-android-ninja` (Drjacky) | Security patterns, JaCoCo config, M3 Adaptive, convention plugins | `arc-android-quality-standards`, `arc-android-project-setup` |
| `bug-hunter` (MartinPSDev) | OWASP Mobile Top 10 scanner + vertical checklists (Fintech, Health, Enterprise, E-commerce) | Adapted into `arc-android-security-audit` (Apache-2.0 attribution) |
| `platform-design-skills` (ehmo) | Unique M3 rules: predictive back, edge-to-edge, gesture zones, notification channels | `arc-android-quality-standards`, `arc-android-presentation-layer` |
| `kotlin-specialist` (Jeffallan) | Coroutines/Flow advanced patterns, DSL idioms | `arc-android-architecture` |
| `kotlin-coroutines-skill` (santimattius) | Standalone skill: 32 structured concurrency practices + triage playbook (scopes, dispatchers, cancellation, Flow, testing) | `arc-android-coroutines` (standalone) |
| `claude-android-skill` (dpconde) | `libs.versions.toml` template, `settings.gradle.kts` template | `arc-android-project-setup` |

### Conflicts Found (Excluded)

| Source | Excluded Content | Reason |
|--------|-----------------|--------|
| awesome-skills | `android-testing` skill | JUnit 4 conflicts with ARC Labs' JUnit 5 mandate |
| claude-android-ninja | Testing philosophy (fakes-only, no MockK) | Conflicts with MockK mandate |
| claude-android-ninja | Navigation3 patterns | Too bleeding-edge; ARC Labs uses Navigation 2.x |
| claude-android-skill | Testing, optional domain layer | Multiple conflicts with ARC Labs architecture |
| compose-skill | 5 source code files (~2.3MB) | Would destroy context windows |

---

## MCP Servers

| Server | Purpose | Config |
|--------|---------|--------|
| **Context7** | Library docs (Compose, Hilt, Room, etc.) | See Tools/mcp-setup.md |
| **Linear** | Issue tracking | See Tools/mcp-setup.md |
| **GitHub** | Branch/PR management | See Tools/mcp-setup.md |
| **Android Source Explorer** | AOSP + Jetpack source exploration | See Tools/mcp-setup.md |
| **Android Docs MCP** | developer.android.com access | See Tools/mcp-setup.md |
| **Mobile MCP** | ADB emulator automation | See Tools/mcp-setup.md |
| **Play Store MCP** | Google Play Console (optional) | See Tools/mcp-setup.md |
| **Android Skills (Google)** | 6 official skills on-demand via MCP | See Tools/mcp-setup.md and Tools/android-cli.md |

---

## Common Scenarios

### 1. "I need to build a new feature"
```
/arc-android-architecture   → Design layers
/arc-tdd-patterns           → Write failing tests (RED)
/arc-data-layer             → Implement data access
/arc-presentation-layer     → Implement UI
/arc-quality-standards      → Review before PR
/arc-workflow               → Commit + PR

Or delegate: arc-kotlin-tdd agent handles the full TDD cycle
```

### 2. "I need to fix a bug"
```
/arc-tdd-patterns           → Write failing test reproducing bug
/arc-data-layer or
  /arc-presentation-layer   → Fix in appropriate layer
/arc-quality-standards      → Verify fix quality
/arc-workflow               → Commit (fix type)
```

### 3. "I need to review code"
```
/arc-quality-standards      → Domain-by-domain checklist
/arc-final-review           → Final structured review

Or delegate: arc-kotlin-reviewer agent produces 9-domain report
```

### 4. "I need to set up a new project"
```
/arc-project-setup          → Project structure + version catalog
/arc-android-architecture   → Set up Clean Architecture layers
/arc-github-actions-ci      → Configure CI/CD workflows
/arc-workflow               → Git branching + hooks
```

### 5. "I need to add a Room database column"
```
Delegate to: arc-room-migration agent (most conservative — confirms before any breaking change)
```

---

## Priority Order

1. **Architecture first** — get structure right before writing code
2. **Tests second** (TDD) — write tests before production code
3. **Implementation third** — build on solid foundations
4. **Quality last** — polish, document, review

---

## Creating Custom Skills

Skills live at `.claude/skills/<name>/SKILL.md` with this frontmatter:

```yaml
---
name: arc-<name>
description: |
  Multi-line description with trigger phrases users might say.
user-invocable: true
metadata:
  author: ARC Labs Studio
  version: "1.0.0"
---
```

Body structure: Title → Instructions → References → Common Mistakes → Examples → Related Skills

Reference docs go in `.claude/skills/<name>/references/` (copied from the repo's top-level docs).

To add a new skill:
1. Create `.claude/skills/arc-<name>/SKILL.md`
2. Copy relevant reference docs into `references/`
3. Add to this index and to CLAUDE.md skills table
4. Commit and update ARCDevTools-Android to symlink it

---

## Coverage Matrix

| Area | ARC Skills | Status |
|------|-----------|--------|
| Architecture (Clean, MVVM, Hilt) | Full | ✅ |
| UI (Jetpack Compose, M3) | Full | ✅ |
| Data (Room, Retrofit, DataStore) | Full | ✅ |
| Testing (JUnit 5, MockK, Turbine) | Full | ✅ |
| CI/CD (GitHub Actions) | Full | ✅ |
| Code Style (ktlint, detekt) | Full | ✅ |
| Accessibility (WCAG 2.2, M3) | Full | ✅ |
| Performance (Compose, Gradle) | Good | ✅ |
| Memory / Worktrees | Full | ✅ |
| Security (OWASP Mobile Top 10 + verticals) | Full | ✅ |
| Concurrency (Coroutines, Flow, Channels) | Full | ✅ |
| Kotlin Multiplatform | None | ❌ |
| Macrobenchmark | None | ❌ |

---

## Version Information

| Component | Version | Last Updated |
|-----------|---------|-------------|
| ARC Labs Skills | 2.2.0 | 2026-04-18 |
| ARCKnowledge-Android | 1.2.0 | 2026-04-18 |
| Skills Index | 2.2.0 | 2026-04-18 |
