# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-10-21

2026 stack refresh.

### Added
- `Architecture/navigation-3.md`: Navigation 3 guide (NavBackStack + `@Serializable` NavKey, NavDisplay + entryProvider, NavigationSuiteScaffold tabs, dialogs and bottom sheets as keys with scene strategies, ViewModel per entry, adaptive list-detail, predictive back, deep links, testing)
- Canonical "ARC Labs 2026 Stack" version table and catalog template in `Tools/gradle.md`
- Binding rules in `CLAUDE.md` mirroring the iOS constitution: zero warnings, third-party allowlist, modern APIs only, secrets in the Android Keystore, tooling policy (`./gradlew` + `android` CLI), tautological tests deleted, fakes over mocks, previews fed from centralized sample data, definition of done
- Networking policy: v1 apps have no HTTP client (Firebase SDKs only); Retrofit 3 + OkHttp 5 + kotlinx.serialization converter in the Vapor phase

### Changed
- Stack: Kotlin 2.4.21, AGP 9.4.0 (built-in Kotlin), Gradle 9.8.2, JBR bundled with Android Studio + jvmTarget 17, compileSdk/targetSdk 36, minSdk 26, Compose BOM 2026.09.00, material3 1.5.0-beta01, Hilt 2.60.1 + KSP 2.3.12, Room 3 3.0.3, DataStore 1.2.1, Coil 3.6.3, Firebase BoM 35.0.2, detekt 1.23.8, ktlint 1.8
- Room 2 → Room 3 (`androidx.room3`, KSP-only, suspend + Flow DAOs, driver-based)
- Material 3 → Material 3 Expressive with brand colors (dynamic color off)
- Coil 2 → Coil 3
- `build-logic/` convention plugins are the single mechanism (`buildSrc` removed); `kapt` removed everywhere
- Coverage gate unified at 80% line coverage
- `Skills/skills-index.md` lists the official `android/skills` plugin (33 skills) and the real ARC skills (`android-doctrine`, `android-testing-doctrine`, `android-verify`, `arc-cli-android`) and agents (`arc-verify-android`, `arc-audit-material`)
- `Projects/apps.md`: FavRes-Android in progress; ARC*-Android libraries marked as planned extractions from FavRes-Android `:core:*` modules
- `_audit/audit-report.md` refreshed

### Removed
- `Architecture/navigation-compose.md` (replaced by `navigation-3.md`)
- References to non-existent skills (`/arc-*`) and scripts (`arcdevtools-setup`, `setup-skills.sh`)

### Fixed
- Contradictory version pins across documents
- Changelog entry for the first release relabelled from 1.0.0 to 0.1.0 to match the `v0.1.0` tag

## [0.1.0] - 2026-02-25

### Added
- Initial ARCKnowledge-Android knowledge base
- Architecture documentation: Clean Architecture, MVVM, SOLID, DI, Navigation, Singletons
- Layer documentation: Presentation, Domain, Data
- Quality documentation: Testing, Code Style, Code Review, Documentation, Module Structure, README Standards, UI Guidelines, Compose Performance
- Tools documentation: Gradle, Android Studio, ARCDevTools-Android
- Workflow documentation: Git Commits, Git Branches, Plan Mode
- Skills index with ARC Labs Android skills
- CLAUDE.md entry point for AI agents
