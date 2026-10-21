# ARCKnowledge-Android — Audit Report

**Date**: 2026-10-21
**Scope**: v0.2.0 "2026 stack refresh" (branch `feature/2026-stack-refresh`)
**Previous audit**: 2026-02-25 (v0.1.0) — superseded by this report

---

## Executive Summary

v0.1.0 was well structured but pinned a 2024 stack (Kotlin 2.0.21, AGP 8.7, Compose BOM 2024.12, Navigation Compose 2.8, Room 2, Coil 2, Retrofit 2.9) with contradictory versions across documents, listed nine skills that never existed, referenced ARCDevTools scripts that do not exist, and lacked the binding rules of the iOS constitution. v0.2.0 refreshes the knowledge base to the 2026 stack with judgment (no rewrite): structure, tone and language (English) are unchanged.

---

## Findings from the 2026-02-25 Audit

| # | Issue | Status |
|---|-------|--------|
| 1 | CLAUDE.md missing Context7/MCP reference, reading list, Gradle reference | Resolved in v0.1.0 |
| 2 | `MoshiConverterFactory` in singletons.md | Resolved; remaining `GsonConverterFactory` usages also replaced by kotlinx.serialization in v0.2.0 |
| 3 | `kapt` for the Hilt compiler in apps.md | Resolved; v0.2.0 removes `kapt` everywhere (KSP only) |
| 4 | JUnit 4 alongside JUnit 5 in apps.md | Resolved; JUnit 5 via the mannodermaus plugin, JUnit 4 only for Compose rule-based UI tests |
| 5 | Broken references in compose-performance.md | Resolved |

## Findings Fixed in v0.2.0

| Area | Finding | Fix |
|------|---------|-----|
| Versions | 2024 pins, contradictory across 10+ documents | Canonical 2026 table + catalog template in `Tools/gradle.md`; other documents point to it |
| Build | `kotlin-android` plugin and `kotlinOptions` (incompatible with AGP 9 built-in Kotlin); `buildSrc` vs `build-logic` | AGP 9 snippets; `build-logic/` convention plugins as the single mechanism |
| Navigation | Navigation Compose 2.8 | `Architecture/navigation-3.md` (Navigation 3) replaces `navigation-compose.md`; all examples and links updated |
| Data | Room 2, Coil 2, Retrofit as default | Room 3, Coil 3, networking policy (v1: no HTTP client) |
| UI | Plain M3 with dynamic color | M3 Expressive, brand color scheme, dynamic color off |
| Testing | MockK as default, coverage thresholds 60/80/100 | Fakes over mocks, unified 80% gate, tautological-test rule, previews + screenshot tests |
| Rules | No zero-warnings, allowlist, modern-API, secrets or tooling policy | "Binding Rules" section in `CLAUDE.md` |
| Skills | Nine non-existent `/arc-*` skills | Real ARC Android skills + official `android/skills` plugin |
| Tooling | `arcdevtools-setup`, `setup-skills.sh` do not exist | Real `arc-setup`, repository layout and Makefile targets |
| Projects | FavRes-Android "Planned"; non-existent ARC*-Android libraries | FavRes-Android "In progress"; libraries "planned — extracted from FavRes-Android `:core:*`" |
| Changelog | Entry labelled 1.0.0 while the released tag is v0.1.0 | Entry relabelled 0.1.0 |

---

## Open Items ("verify before use")

ARCDevTools-Android 0.2.0 resolved the stack with a successful Gradle build (`templates/gradle/libs.versions.toml`, every coordinate confirmed to exist, 2026-10-21). Backfilled from it: core-ktx 1.19.1, activity-compose 1.13.0, hilt-lifecycle-viewmodel-compose 1.4.0, google-services 4.5.0, googleid 1.2.1, JUnit Jupiter BOM 5.14.4 (+ JUnit 4 4.13.2 via vintage engine), Turbine 1.2.1, Robolectric 4.17, ktlint Gradle plugin 14.2.0, compose-rules 0.4.28, kotlinx.serialization 1.12.0, coroutines 1.11.0; plugin ids `de.mannodermaus.android-junit` and `androidx.room3` (extension `room3`); screenshot flag, Robolectric `--add-exports`, JDK 21 daemon for detekt, `junit4.v2` Compose test rules.

Still open (each marked "verify" where it appears):

| Item | Where |
|------|-------|
| **compileSdk 37** — Compose BOM 2026.09.00, material3 1.5.0-beta01, Nav3 1.2.0, Lifecycle 2.11.0 and core 1.19.1 declare `minCompileSdk` 37; docs keep 36 with a "pending" note until the user confirms (targetSdk stays 36) | `Tools/gradle.md`, `Tools/android-studio.md`, `Quality/module-structure.md`, `Architecture/navigation-3.md`, `README.md` |
| Versions not in the verified catalog: Dokka, kotlinx-datetime, kotlinx-collections-immutable, baselineprofile plugin, Retrofit 3 / OkHttp 5 | `Quality/documentation.md`, `Layers/domain.md`, `Quality/compose-performance.md`, `Tools/gradle.md` |
| `NavigationSuiteScaffold` overload in material3 1.5.0-beta01; `rememberViewModelStoreNavEntryDecorator` generic signature; window-info entry point | `Architecture/navigation-3.md`, `Quality/ui-guidelines.md` |
| `MaterialExpressiveTheme` parameters and opt-in status | `Quality/ui-guidelines.md`, `Layers/presentation.md`, `Projects/apps.md` |
| Room 3: builder overloads, `Migration(SQLiteConnection)`, `AndroidSQLiteDriver` artifact | `Layers/data.md`, `Quality/testing.md` |
| Coil 3 `SingletonImageLoader` / `ImageResult.image` / `toBitmap()` | `Architecture/singletons.md` |
| Kotlin extension type registered by AGP 9 built-in Kotlin (convention plugins) | `Quality/module-structure.md` |
| Firestore coroutine helpers without `-ktx` artifacts | `Layers/data.md` |
| `NavigationBackHandler` (Navigation Event) signature | `Architecture/navigation-3.md` |
| Convention plugin ids: this KB uses `arclabs.android.*`, ARCDevTools-Android 0.2.0 ships `arc.android.*` — align | `Tools/gradle.md`, `Quality/module-structure.md`, `Architecture/navigation-3.md` |

## Known Remaining Debt

- Many long-form examples (testing, solid-principles, mvvm, clean-architecture, domain, libraries) still illustrate test doubles with MockK. The binding rules and `Quality/testing.md` mark these as legacy; rewriting every example with fakes is a follow-up work unit.
- Retrofit/OkHttp examples remain as Vapor-phase reference and were written against Retrofit 2 / OkHttp 4 APIs.
- `Tools/arcdevtools-android.md` workflow YAML and custom ktlint/detekt rule sections describe intended behaviour; reconcile with ARCDevTools-Android v0.2.0 when it ships.
