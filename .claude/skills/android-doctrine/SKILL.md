---
name: android-doctrine
description: >
  ARC's binding Android ruleset (Kotlin + Jetpack Compose). Read BEFORE writing or modifying
  any Android code — screens, composables, ViewModels, use cases, repositories, Room, Gradle
  modules, navigation, theming, DI. Triggers on "Android", "Kotlin", "Compose", "composable",
  "@Composable", "@Preview", "ViewModel", "@HiltViewModel", "StateFlow", "UiState",
  "collectAsStateWithLifecycle", "use case", "repository", "Room", "DAO", "DataStore",
  "Hilt", "KSP", "kapt", "Gradle module", "build-logic", "convention plugin",
  "libs.versions.toml", "Navigation 3", "NavDisplay", "NavKey", "back stack",
  "NavigationSuiteScaffold", "Material 3", "M3 Expressive", "theme", "dynamic color",
  "edge-to-edge", "predictive back", "per-app language", "16 KB page size", "Keystore",
  "allWarningsAsErrors", "detekt", "ktlint", "add a dependency". Governs architecture and ARC
  house rules; delegates API detail to Google's android-skills plugin.
---

# Android Doctrine (ARC Labs)

**Precedence — scoped, not blanket.** This skill governs **architecture and ARC house rules**.
API detail belongs to Google's official `android-skills` plugin (routing table below) and to
`android docs search` — they win on how an API behaves. If one contradicts a rule here, say so
rather than silently picking one. Versions are not pinned here: they live in the project's
`gradle/libs.versions.toml` (reference table in ARCKnowledge-Android). **Never invent an API** —
Nav3, Room 3 and M3 Expressive are young; verify with `android docs`, the plugin references or
Context7 before writing against them.

The constitution's prime directives and non-negotiables apply unchanged: no speculation, minimum
change, zero warnings, no secrets in source, never modify a test to make it pass.

## Core rules (always apply)

1. **Layers, one direction.** UI → Domain ← Data.
   - **UI:** Compose + `@HiltViewModel`. Each screen exposes one `StateFlow<UiState>` where
     `UiState` is a `sealed interface` (Loading / Content / Error…); one-shot events (navigate,
     snackbar) travel separately from state. Collect with `collectAsStateWithLifecycle()` —
     never `collectAsState()` for screen state. The VM orchestrates state only.
   - **Domain:** pure Kotlin JVM module — no Android, no Compose, no Room, no Firebase imports.
     Entities, use cases as classes with a single `operator fun invoke`, repository
     *interfaces*, and a `sealed` `DomainException` hierarchy. Business logic lives here.
   - **Data:** Room 3 is the single source of truth (offline-first); Firebase SDKs are remote
     data sources; DTOs never cross into Domain — mappers at the boundary. Repositories expose
     `Flow` from Room for reactive reads.
2. **Modules.** `app` · `domain` · `core:*` · `feature:*`. **No `feature → feature`
   dependency** — shared code moves to `core:*`. Build config through convention plugins in
   `build-logic/` (never `buildSrc`, never copy-pasted `android {}` blocks) and every
   dependency through the version catalog — no literal coordinates in module build files.
3. **Navigation 3 only.** Back stack = `NavBackStack<NavKey>` with `@Serializable` keys,
   rendered by `NavDisplay` + `entryProvider`. Top-level destinations in
   `NavigationSuiteScaffold` (bar ↔ rail adapts by window size). Modals (dialog, bottom sheet)
   are keys on the back stack shown through a scene strategy, not ad-hoc `showDialog` state.
   Exception: a transient error alert that is part of the screen's `UiState` (e.g. a form's
   submit failure) is rendered from that state and dismissed through a ViewModel event — it is
   not a destination. User-initiated confirmations (sign out, delete) are always keys.
   ViewModels scoped per entry (`rememberViewModelStoreNavEntryDecorator`). Predictive back
   stays enabled. No Nav2 `NavHost`, no custom Coordinator/Router replacing the back stack.
4. **Material 3 Expressive theming.** One `AppTheme` with the brand color scheme in
   `core:designsystem`; **dynamic color off** (brand identity); light + dark schemes; M3
   typography scale (`MaterialTheme.typography.*`) — no hardcoded `sp` sizes in composables,
   no color literals outside the theme. Native M3 components only; a custom look on an M3
   component goes through its parameters/styles, never a parallel re-implementation.
5. **DI = Hilt + KSP.** Never `kapt` — any `kapt` plugin or configuration is a violation.
   Constructor injection; bind interfaces in Hilt modules; no service locators.
6. **Zero business logic in composables.** Validation, transformation, persistence and
   orchestration live in use cases / the ViewModel. Composables receive state + lambdas.
   Screen-level composable takes the VM; everything below it is stateless and previewable.
7. **Previews.** Every composable ships a `@Preview` fed from centralized sample data
   (`core:ui` sample-data objects / `PreviewParameterProvider`) — never inline invented
   literals. Previews are also the screenshot-test oracle (`android-testing-doctrine`).
8. **Platform rules.** Edge-to-edge forced (`enableEdgeToEdge()`, insets handled — see
   `edge-to-edge`); per-app language via `AppCompatDelegate.setApplicationLocales`;
   any native `.so` must be 16 KB page-aligned; accessibility: every interactive element has
   a TalkBack label (`contentDescription` / semantics), touch targets ≥ 48dp, text in `sp`
   through the type scale.
9. **Zero warnings.** Kotlin `allWarningsAsErrors = true`, Android Lint
   `warningsAsErrors = true` with `abortOnError`, detekt `maxIssues: 0`, ktlint 0 violations.
   No `@Suppress` without a comment citing why and an approval. Baselines are not allowed to
   hide new code.
10. **Third-party allowlist.** Hilt, Firebase, RevenueCat, Coil, maps-compose · tests: JUnit 5
    (mannodermaus plugin), Turbine · tooling: detekt, ktlint. **Anything else → ask first.**
    AndroidX/Jetpack and Kotlin first-party libraries are not third-party.
11. **Modern APIs only.** No deprecated or soft-deprecated APIs in new code. AVOID → PREFER:
    `LiveData` → `StateFlow` · `kapt` → KSP · Nav2 `NavHost` → Nav3 `NavDisplay` ·
    `SharedPreferences` → DataStore · `AsyncTask`/`Handler`/callbacks → coroutines ·
    `GlobalScope` → `viewModelScope`/injected scope · `startActivityForResult` → Activity
    Result APIs · `SimpleDateFormat` → `java.time` · Gson/Moshi → kotlinx.serialization ·
    XML layouts → Compose · `!!` → `?.`/`?:`/`requireNotNull` with a message (test fixtures
    excepted).
12. **Secrets.** Keys and tokens in the Android Keystore (encrypted at rest) — never in source,
    `BuildConfig`, `strings.xml`, DataStore/SharedPreferences plaintext, or logs. Obfuscated is
    still a secret. Server-side keys stay server-side (proxy via Cloud Functions).
13. **Concurrency.** Coroutines + Flow only; inject `CoroutineDispatcher`s (`core:common`),
    never hardcode `Dispatchers.IO` in classes under test. Structured concurrency — no
    `GlobalScope`, no fire-and-forget without an owning scope.

## Tooling policy

`./gradlew` + the `android` CLI are the sanctioned tools (the Xcode-MCP equivalent) — see
`arc-cli-android`. Android Studio GUI is optional. `mobile-mcp` only when the CLI genuinely
cannot do something — ask first. The verification loop is `android-verify` /
`arc-verify-android`.

## Routing — Google `android-skills` plugin (API detail)

| Area | Skill |
|---|---|
| Nav3 setup, scenes, deep links, Hilt + ViewModel per entry, results | `android-skills:navigation-3` |
| Predictive back, back events | `android-skills:navigation-event` |
| Window sizes, nav bar ↔ rail, list-detail, state across config changes | `android-skills:adaptive` |
| Edge-to-edge, insets, IME, system bar contrast | `android-skills:edge-to-edge` |
| Compose Styles API, component theming | `android-skills:styles` |
| Recomposition, stability, lazy lists performance | `android-skills:jetpack-compose-performance` |
| AGP 9 migration | `android-skills:agp-9-upgrade` |
| R8 / keep rules | `android-skills:r8-analyzer` |
| Test infrastructure, screenshot tests | `android-skills:testing-setup` (rules: `android-testing-doctrine`) |
| Permissions, intents security | `android-skills:android-permissions-security`, `android-skills:android-intent-security` |
| Credential Manager restore, verified email | `android-skills:restore-credentials`, `android-skills:verified-email` |
| Play Billing upgrade (under RevenueCat) | `android-skills:play-billing-library-version-upgrade` |
| Play policy | `android-skills:play-policy-insights` |
| ANRs, profiling | `android-skills:anr-audit-and-fix`, `android-skills:android-profiler` |
| `android` CLI usage | `arc-cli-android` (ARC policy) → `android-cli` (Google reference) |

Firebase: `arc-mcp-firebase`. RevenueCat: `arc-mcp-revenuecat`.

## Scoping rule

Fix and comment only the code you were asked to touch. Never flag, migrate or mention issues in
out-of-scope code — no "I also noticed…" trailing offers.
