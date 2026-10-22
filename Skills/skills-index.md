# Skills Index

**Skills are AI agent context files that provide specialized knowledge for specific tasks. ARC Labs Android work uses two families: the official Google `android/skills` plugin for API detail, and four ARC Labs skills (plus two agents) for house rules and verification. Skills are knowledge containers that load guidance; agents are autonomous executors that use tools to complete a workflow. This index helps you find the right one.**

---

## Skill Sources Overview

| Source | Type | Focus | Count |
|--------|------|-------|-------|
| **ARC Labs** (this repo's `.claude/skills/`; agents in ARCAgentStack `config/agents/`) | Internal standards | House rules, testing doctrine, verification loop, CLI policy | 4 skills + 2 agents |
| **Google** (`android/skills` Claude plugin, v1.0.16) | Official | API detail and migrations across the Android platform | 33 skills |
| **MCP / docs** | Documentation | Context7 (`/websites/developer_android_develop_ui_compose`), `android docs` | — |

**Precedence:** Google's skills win on *how an API behaves*; ARC skills win on *house rules* (architecture, allowlist, zero warnings, testing doctrine). If they contradict each other, say so instead of silently picking one.

> The `/arc-android-*`, `/arc-*-layer`, `/arc-tdd-patterns`, … skills and the 13 agents (`arc-kotlin-tdd`, `arc-gradle-manager`, `arc-room-migration`, …) listed in earlier versions of this index (1.0.0 and 0.2.0) have no definition files in ARCAgentStack, `~/.claude` or this repository, and have been removed from the routing (see [AGENTS.md](../AGENTS.md)).

---

## Quick Decision Guide

```
Need to...                                      Use
-----------------------------------------------------------------------------------------
Write or change any Android code                android-doctrine (+ the Google skill for the API area)
Navigation, back stack, dialogs, sheets         navigation-3 · predictive back: navigation-event
Bar <-> rail, list-detail, window sizes          adaptive
Insets, edge-to-edge, IME                       edge-to-edge
Component theming / Styles API                  styles
Recomposition and performance                   jetpack-compose-performance
Write or audit tests                            android-testing-doctrine (+ testing-setup)
Build, install, run, emulator, CLI              arc-cli-android (+ android-cli)
Verify a change (build/tests/screenshots/app)   android-verify / agent arc-verify-android
Close a UI work unit                            agent arc-audit-material
AGP 9 / Gradle migration                        agp-9-upgrade
R8 keep rules                                   r8-analyzer
Permissions / intents security                  android-permissions-security / android-intent-security
Play Billing upgrade (under RevenueCat)         play-billing-library-version-upgrade
Play policy check before release                play-policy-insights
Commits, branches, PRs                          Workflow/git-commits.md, Workflow/git-branches.md
```

---

## ARC Labs Skills (`.claude/skills/`)

Thin skills that encode ARC Labs house rules; API detail is delegated to the Google plugin. They live in this repository's `.claude/skills/` and are symlinked into each project's `.claude/skills/` by ARCDevTools-Android's `arc-setup` (`scripts/setup-skills.sh`), like ARCKnowledge on iOS.

### `android-doctrine`

**Purpose**: ARC's binding Android ruleset — read **before writing or modifying any Android code**.

**Covers**: layers (UI → Domain ← Data), modules (`app` · `domain` · `core:*` · `feature:*`, no feature → feature), `build-logic/` convention plugins, Navigation 3 rules, Material 3 Expressive theming (dynamic color off), Hilt + KSP (never kapt), zero business logic in composables, previews from centralized sample data, platform rules (edge-to-edge, per-app language, 16 KB pages, accessibility), zero warnings, third-party allowlist, modern APIs only, secrets, concurrency. Includes the routing table to Google skills.

### `android-testing-doctrine`

**Purpose**: What makes an Android test real, and the gate every test must pass — read **before creating any `*Test.kt`**.

**Covers**: behavioral tests against an independent oracle, deleting tautological tests, fakes over mocks, JUnit 5, `runTest`, Turbine, Robolectric + Compose UI tests, Compose Preview Screenshot Testing, coverage, TDD, shared golden test vectors with iOS.

### `android-verify`

**Purpose**: The verification loop — the external verifier for Kotlin/Compose work.

**Covers**: zero-warning `./gradlew build`, `testDebugUnitTest`, `validateDebugScreenshotTest`, then install and walk the app on a headless emulator with the `android` CLI (`android run`, `android layout`, `android screen capture`). The main agent iterates until it is green.

### `arc-cli-android`

**Purpose**: ARC's tooling policy and companion for the official `android` CLI — read **before any Android tooling task**.

**Covers**: `./gradlew` + `android` CLI as the sanctioned tools (the Xcode MCP equivalent), Android Studio GUI optional, `mobile-mcp` only when the CLI cannot do it; key commands, recipes and gotchas. Google's `android-cli` skill is the command reference.

### Agents

| Agent | Role | When |
|---|---|---|
| `arc-verify-android` | Runs the `android-verify` loop and reports pass/fail | After any change; iterate until it returns green |
| `arc-audit-material` | Audits UI work against Material 3 / platform rules / TalkBack | Before closing any UI-affecting work unit — one blocker and the unit does not close |

See [AGENTS.md](../AGENTS.md) for agent documentation.

---

## Google Skills (`android/skills` plugin)

Official, AI-optimized instructions from developer.android.com. Install once per machine:

```
/plugin marketplace add android/skills
/plugin install android-skills@android-skills
```

(or `android skills add --all` with the Android CLI). Invoke as `android-skills:<name>`.

| Area | Skill | What it covers |
|---|---|---|
| Build | `agp-9-upgrade` | Migrating to AGP 9: built-in Kotlin, new DSL, KSP/kapt, BuildConfig |
| Camera | `camerax` | CameraX guidance |
| Cars | `android-for-cars-parked-apps` | Video, games and browsers for parked cars |
| Device AI | `appfunctions` | Exposing app workflows as AppFunctions |
| Device AI | `litert-lm` | On-device open LLMs with LiteRT-LM |
| Device AI | `litert-runtime` | LiteRT (`.tflite`) inference |
| Device AI | `ml-kit-genai-prompt-api` | ML Kit GenAI Prompt API |
| Dev tools | `android-cli` | Installing and using the `android` CLI |
| Identity | `restore-credentials` | Restore Credentials (Credential Manager) |
| Identity | `verified-email` | Verified email retrieval |
| Compose | `adaptive` | Adaptive UI: `NavigationSuiteScaffold`, Nav3 list-detail/supporting-pane scenes, grids |
| Compose | `jetpack-compose-performance` | Diagnosing and optimizing Compose performance |
| Compose | `migrate-xml-views-to-jetpack-compose` | XML → Compose migration workflow |
| Compose | `styles` | Compose Styles API for component theming |
| Media | `media3-cast-integration` | Google Cast with Media3 |
| Navigation | `navigation-3` | Nav3 setup, migration from Nav2, recipes (dialogs, bottom sheets, tabs, Hilt ViewModels, results, deep links) |
| Navigation | `navigation-event` | Back gestures and predictive back animations |
| Performance | `anr-audit-and-fix` | ANR risk audit and stack-dump diagnosis |
| Performance | `r8-analyzer` | R8 keep-rule analysis |
| Play | `engage-sdk-integration` | Play Engage SDK |
| Play | `play-billing-library-version-upgrade` | Play Billing Library upgrades |
| Play | `play-policy-insights` | Google Play policy audit |
| Profilers | `android-profiler` | Recording and analyzing performance data |
| Security | `android-intent-security` | Intent security |
| Security | `android-permissions-security` | Permissions and IPC component audit |
| System | `edge-to-edge` | Adaptive edge-to-edge and insets |
| System | `glance-canonical-layouts` | Home screen widgets with Glance |
| Testing | `testing-setup` | Testing strategy, infrastructure, screenshot tests |
| TV | `leanback-to-compose-tv-migration` | Leanback → Compose for TV |
| Wear | `wear-compose-m3` | Wear Compose Material 3 |
| Wear | `wear-widgets` | Wear OS widgets with Glance Wear |
| XR | `display-glasses-with-jetpack-compose-glimmer` | Projected XR apps for display glasses |
| XR | `projected-hardware` | Hardware access for projected XR |

---

## MCP Servers

| Server | Purpose | Config |
|--------|---------|--------|
| **Context7** | Library docs (Compose, Nav3, Hilt, Room 3, etc.) | See Tools/mcp-setup.md |
| **Linear** | Issue tracking | See Tools/mcp-setup.md |
| **GitHub** | Branch/PR management | See Tools/mcp-setup.md |
| **Android Source Explorer** | AOSP + Jetpack source exploration | See Tools/mcp-setup.md |
| **Android Docs MCP** | developer.android.com access | See Tools/mcp-setup.md |
| **Mobile MCP** | ADB emulator automation (only when the `android` CLI cannot do it, after asking) | See Tools/mcp-setup.md |
| **Play Store MCP** | Google Play Console (optional) | See Tools/mcp-setup.md |
| **Android Skills (Google)** | The 33 official skills above (`android/skills` plugin) | See Tools/mcp-setup.md and Tools/android-cli.md |

---

## Common Scenarios

### 1. "I need to build a new feature"

```
android-doctrine            --> Layers, modules, UiState, rules
android-testing-doctrine    --> Failing tests first (fakes, Turbine, screenshot previews)
navigation-3 / adaptive     --> Keys, entries, scenes for the new screens
android-verify              --> Build (zero warnings) → tests → screenshots → emulator
arc-audit-material          --> Close the UI work unit
```

### 2. "I need to fix a bug"
```
android-testing-doctrine    --> Failing test that reproduces the bug
android-doctrine            --> Fix in the right layer
android-verify              --> Green loop
```

### 3. "I need to set up a new project"

```
arc-cli-android             --> android create, SDK, emulator, JAVA_HOME
android-doctrine            --> Module layout, build-logic, catalog (Tools/gradle.md)
testing-setup               --> JUnit 5, Robolectric, screenshot tests
edge-to-edge                --> Insets from day one
```

### 4. "I need to upgrade the build"

```
agp-9-upgrade               --> AGP / built-in Kotlin / DSL migration
Tools/gradle.md             --> Canonical 2026 version table
android-verify              --> Zero-warning build + tests
```

---

## Priority Order

1. **Architecture first** — get structure right before writing code
2. **Tests second** (TDD) — write tests before production code
3. **Implementation third** — build on solid foundations
4. **Quality last** — polish, document, review

---

## Creating Custom Skills

ARC Android skills are authored in this repository under `.claude/skills/<name>/SKILL.md` (frontmatter `name` + `description` with triggers; see the `arc-agent-authoring` skill for the format), then added to this index and to the CLAUDE.md skills table. Agents are authored in **ARCAgentStack** (`config/agents/`).

---

## Coverage Matrix

| Gap | Description | Suggested Source |
|-----|-------------|------------------|
| Firebase on Android | Auth, Firestore, Functions, Remote Config, App Check | `arc-mcp-firebase` + Firebase docs via Context7 |
| RevenueCat on Android | Purchases, Paywall Compose, Customer Center | `arc-mcp-revenuecat` + RevenueCat docs |
| Room 3 | Migrations, drivers, testing | Context7 / `android docs` (no dedicated skill yet) |
| Material 3 Expressive | Expressive components and motion | Context7 / `android docs`; audited by `arc-audit-material` |
| Kotlin Multiplatform | Not covered | — |
| Macrobenchmark | Not covered | — |

If you identify a repeated workflow that is not covered, open a Linear issue with the label `skill-request`.

---

## Version Information

| Component | Version | Last Updated |
|-----------|---------|-------------|
| Google `android/skills` plugin | 1.0.16 (33 skills) | 2026-10-21 |
| ARC Android skills (`.claude/skills/`) + agents (ARCAgentStack) | 4 skills + 2 agents | 2026-10-21 |
| ARCKnowledge-Android | 0.3.0 | 2026-10-21 |

---

## Further Reading

- [CLAUDE.md](../CLAUDE.md) — binding rules and skill routing
- [Navigation 3](../Architecture/navigation-3.md)
- [Testing Standards](../Quality/testing.md)
- [Gradle — 2026 stack table](../Tools/gradle.md)
- [Git Commits](../Workflow/git-commits.md) · [Git Branches](../Workflow/git-branches.md) · [Plan Mode](../Workflow/plan-mode.md)
- [Android skills documentation](https://developer.android.com/tools/agents/android-skills)

---

**Note**: Skills are living documents. When you find gaps or outdated information, flag it for updates.
