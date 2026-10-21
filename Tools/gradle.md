# 🔨 Gradle

**Gradle with Kotlin DSL is the official build system for Android projects at ARC Labs Studio. This guide covers project configuration, dependency management with Version Catalogs, multi-module setup, build variants, performance optimization, and best practices for production-grade Android builds.**

---

## 🎯 Why Gradle?

**Gradle is the standard build system for Android development, endorsed by Google and tightly integrated with Android Studio. Combined with Kotlin DSL, it provides a type-safe, expressive, and maintainable build configuration.**

### Core Advantages

- **Official Android Build System** - First-class support from Google, Android Studio, and the entire Android ecosystem
- **Kotlin DSL (build.gradle.kts)** - Type-safe configuration with full IDE autocompletion, refactoring support, and compile-time error checking
- **Version Catalogs** - Centralized dependency management via `libs.versions.toml` for consistent versioning across all modules
- **Multi-Module Support** - First-class support for modularized architectures with fine-grained dependency graphs
- **Incremental Builds** - Smart caching and incremental compilation for fast build times
- **Extensible Plugin System** - Rich ecosystem of plugins for code generation (KSP), dependency injection (Hilt), static analysis (detekt, ktlint), and more
- **Convention Plugins** - Share build logic across modules with convention plugins in the `build-logic/` included build (the single mechanism at ARC Labs; `buildSrc` is not used)

### Why Kotlin DSL Over Groovy?

```kotlin
// ✅ Kotlin DSL - Type-safe, autocomplete works, compile-time checks
android {
    compileSdk = libs.versions.compileSdk.get().toInt()

    defaultConfig {
        minSdk = libs.versions.minSdk.get().toInt()
        targetSdk = libs.versions.targetSdk.get().toInt()
    }
}

dependencies {
    implementation(libs.compose.material3) // Autocomplete from version catalog
}
```

```groovy
// ❌ Groovy DSL - No type safety, no autocomplete, runtime errors
android {
    compileSdkVersion 36 // No IDE help, typos caught at runtime

    defaultConfig {
        minSdkVersion 26
        targetSdkVersion 36
    }
}

dependencies {
    implementation "androidx.compose.material3:material3:1.5.0-beta01" // String-based, error-prone
}
```

---

## 📐 Project Structure

**A well-organized multi-module Android project follows a clear directory convention. Each module has its own `build.gradle.kts` and the root project coordinates shared configuration.**

```
project-root/
├── build.gradle.kts              (root - plugins, repositories)
├── settings.gradle.kts           (module includes, plugin management)
├── gradle.properties             (JVM args, Kotlin config)
├── gradle/
│   ├── wrapper/
│   │   ├── gradle-wrapper.jar
│   │   └── gradle-wrapper.properties
│   └── libs.versions.toml        (version catalog)
├── app/
│   ├── build.gradle.kts          (app module - application plugin)
│   ├── proguard-rules.pro        (R8/ProGuard rules)
│   └── src/
│       ├── main/
│       │   ├── AndroidManifest.xml
│       │   ├── kotlin/
│       │   └── res/
│       ├── debug/
│       └── release/
├── feature/
│   ├── home/
│   │   ├── build.gradle.kts      (feature module - library plugin)
│   │   └── src/
│   ├── search/
│   │   ├── build.gradle.kts
│   │   └── src/
│   └── profile/
│       ├── build.gradle.kts
│       └── src/
├── core/
│   ├── database/
│   │   ├── build.gradle.kts      (core module - local storage)
│   │   └── src/
│   ├── domain/
│   │   ├── build.gradle.kts      (core module - business logic)
│   │   └── src/
│   ├── ui/
│   │   ├── build.gradle.kts      (core module - shared UI components)
│   │   └── src/
│   ├── model/
│   │   ├── build.gradle.kts      (core module - data models)
│   │   └── src/
│   └── testing/
│       ├── build.gradle.kts      (core module - test utilities)
│       └── src/
└── build-logic/                  (included build - convention plugins)
    ├── settings.gradle.kts
    └── convention/
        ├── build.gradle.kts
        └── src/main/kotlin/
            └── AndroidLibraryConventionPlugin.kt
```

### Module Dependency Rules

```
app -> feature:* -> core:*
feature:home -> core:domain, core:ui, core:model
core:database -> core:model
# core:network (Retrofit) only exists once a Vapor backend arrives; v1 apps talk to Firebase SDKs only
core:domain -> core:model (pure Kotlin, no Android dependencies)
```

---

## 📦 Version Catalogs (libs.versions.toml)

**Version Catalogs provide a single source of truth for all dependency versions. Located at `gradle/libs.versions.toml`, they enable type-safe dependency references across all modules.**

### 🗓️ ARC Labs 2026 Stack (Reference Table)

**This table is the canonical version reference for every ARC Labs Android document.** Other documents point here instead of pinning their own versions. Verified 2026-10-21; re-verify (`android docs`, Context7, release notes) before bumping.

| Layer | Choice | Version |
|---|---|---|
| Language | Kotlin (K2) | 2.4.21 |
| Build | Android Gradle Plugin / Gradle | 9.4.0 / 9.8.2 |
| JDK | JBR bundled with Android Studio (currently 25) for the IDE/CLI; **Gradle daemon pinned to JDK 21** (`gradle/gradle-daemon-jvm.properties` + foojay resolver) because detekt 1.23.8 fails on JDK 25 · bytecode target `jvmTarget` 17 | — / 21 |
| SDK | `compileSdk` = `targetSdk` | 36 |
| SDK | ⚠️ pending: compileSdk 37 required by Compose 1.12 stack (BOM 2026.09.00, material3 1.5.0-beta01, Nav3 1.2.0 declare `minCompileSdk` 37) — awaiting confirmation; targetSdk stays 36 | — |
| SDK | `minSdk` | 26 |
| Annotation processing | KSP (KSP2) — **no kapt** | 2.3.12 |
| UI | Compose BOM | 2026.09.00 |
| UI | material3 (M3 Expressive, pinned above the BOM) | 1.5.0-beta01 |
| UI | material3-adaptive | 1.3.0 |
| AndroidX | core-ktx / activity-compose | 1.19.1 / 1.13.0 |
| Navigation | Navigation 3 (`androidx.navigation3`) | 1.2.0 |
| Navigation | lifecycle-viewmodel-navigation3 (Lifecycle) | 2.11.0 |
| DI | Hilt · `androidx.hilt:hilt-lifecycle-viewmodel-compose` | 2.60.1 · 1.4.0 |
| Persistence | Room 3 (`androidx.room3`, KSP-only) | 3.0.3 |
| Persistence | DataStore | 1.2.1 |
| Serialization | kotlinx.serialization | 1.12.0 |
| Concurrency | kotlinx.coroutines | 1.11.0 |
| Images | Coil 3 | 3.6.3 |
| Backend | Firebase BoM · google-services plugin | 35.0.2 · 4.5.0 |
| Identity | Credential Manager · googleid | 1.6.0 · 1.2.1 |
| Payments | RevenueCat `purchases-android` | 10.28.1 |
| Maps | maps-compose | 9.0.1 |
| Tests | JUnit 5 Gradle plugin (mannodermaus, id `de.mannodermaus.android-junit`) · JUnit Jupiter BOM · JUnit 4 (Compose rules) | 2.0.2 · 5.14.4 · 4.13.2 |
| Tests | Turbine · Robolectric · Compose Preview Screenshot Testing | 1.2.1 · 4.17 · 0.0.1-alpha16 |
| Quality | detekt · compose-rules (`io.nlopez.compose.rules:detekt`) | 1.23.8 · 0.4.28 |
| Quality | ktlint (engine) · jlleitschuh ktlint Gradle plugin | 1.8.0 · 14.2.0 |
| Networking (Vapor phase only) | Retrofit 3 + OkHttp 5 + kotlinx.serialization converter | verify before use |

> Every version above except the Vapor-phase networking row was resolved by a Gradle build of ARCDevTools-Android 0.2.0 (`templates/gradle/libs.versions.toml`, every coordinate confirmed to exist, 2026-10-21). That template is the catalog projects actually copy; this table mirrors it. **"verify before use"** means the version is intentionally not pinned here — look it up (`android docs`, Context7, Maven Central) when the dependency is first added. Never guess a version.

**Toolchain facts verified by that build:**

- **Gradle daemon on JDK 21**: detekt 1.23.8 fails on JDK 25, so `gradle/gradle-daemon-jvm.properties` pins `toolchainVersion=21` and `settings.gradle.kts` applies the `org.gradle.toolchains.foojay-resolver-convention` plugin to provision it. detekt tasks set `jvmTarget = "17"`.
- **Screenshot tests** need `android.experimental.enableScreenshotTest=true` (in `gradle.properties` / the Compose convention plugin).
- **Robolectric 4.17** needs `--add-exports=java.base/jdk.internal.access=ALL-UNNAMED` on the unit-test JVM (`testOptions.unitTests.all { it.jvmArgs(...) }`).
- **Compose 1.12 test rules**: `createComposeRule` / `createAndroidComposeRule` from `androidx.compose.ui.test.junit4` are deprecated — use the `androidx.compose.ui.test.junit4.v2` variants.

### Catalog Template

```toml
[versions]
# Kotlin & Android
kotlin = "2.4.21"
agp = "9.4.0"
ksp = "2.3.12"

# SDK
compileSdk = "36"                   # pending: compileSdk 37 required by Compose 1.12 stack
targetSdk = "36"
minSdk = "26"

# Compose & UI
compose-bom = "2026.09.00"
material3 = "1.5.0-beta01"          # M3 Expressive — explicit pin overrides the BOM
material3-adaptive = "1.3.0"
activity-compose = "1.13.0"
core-ktx = "1.19.1"

# Navigation & Lifecycle
nav3 = "1.2.0"
lifecycle = "2.11.0"

# DI
hilt = "2.60.1"
androidx-hilt = "1.4.0"             # hilt-lifecycle-viewmodel-compose

# Persistence
room3 = "3.0.3"
datastore = "1.2.1"

# Kotlin libraries
kotlinx-serialization = "1.12.0"
kotlinx-coroutines = "1.11.0"

# Images
coil = "3.6.3"

# Firebase, identity, payments, maps
firebase-bom = "35.0.2"
google-services = "4.5.0"           # com.google.gms.google-services plugin
credentials = "1.6.0"
googleid = "1.2.1"
revenuecat = "10.28.1"
maps-compose = "9.0.1"

# Testing
junit5-plugin = "2.0.2"             # mannodermaus Android JUnit 5 Gradle plugin
junit-bom = "5.14.4"                # JUnit Jupiter
junit4 = "4.13.2"                   # Compose rule-based UI tests (vintage engine)
turbine = "1.2.1"
robolectric = "4.17"                # needs --add-exports=java.base/jdk.internal.access=ALL-UNNAMED
screenshot = "0.0.1-alpha16"        # Compose Preview Screenshot Testing; needs android.experimental.enableScreenshotTest=true

# Static analysis
detekt = "1.23.8"                   # runs with jvmTarget 17; fails on JDK 25 (daemon pinned to 21)
compose-rules-detekt = "0.4.28"     # last compose-rules line built for detekt 1.23.x
ktlint = "1.8.0"                    # ktlint engine
ktlint-gradle = "14.2.0"            # org.jlleitschuh.gradle.ktlint plugin

[libraries]
# ──────────────────────────────────────────────
# Compose (versions from the BOM, except material3)
# ──────────────────────────────────────────────
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics" }
compose-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
compose-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
compose-ui-test-junit4 = { group = "androidx.compose.ui", name = "ui-test-junit4" }
compose-ui-test-manifest = { group = "androidx.compose.ui", name = "ui-test-manifest" }
compose-material3 = { group = "androidx.compose.material3", name = "material3", version.ref = "material3" }
compose-material3-navigation-suite = { group = "androidx.compose.material3", name = "material3-adaptive-navigation-suite", version.ref = "material3" }
compose-material3-adaptive-navigation3 = { group = "androidx.compose.material3.adaptive", name = "adaptive-navigation3", version.ref = "material3-adaptive" }  # verify version alignment

# ──────────────────────────────────────────────
# AndroidX Core
# ──────────────────────────────────────────────
core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "core-ktx" }
activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "activity-compose" }

# ──────────────────────────────────────────────
# Lifecycle + Navigation 3
# ──────────────────────────────────────────────
lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version.ref = "lifecycle" }
lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycle" }
lifecycle-viewmodel-navigation3 = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-navigation3", version.ref = "lifecycle" }
navigation3-runtime = { group = "androidx.navigation3", name = "navigation3-runtime", version.ref = "nav3" }
navigation3-ui = { group = "androidx.navigation3", name = "navigation3-ui", version.ref = "nav3" }

# ──────────────────────────────────────────────
# Hilt (Dependency Injection) — KSP only
# ──────────────────────────────────────────────
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
hilt-android-testing = { group = "com.google.dagger", name = "hilt-android-testing", version.ref = "hilt" }
hilt-lifecycle-viewmodel-compose = { group = "androidx.hilt", name = "hilt-lifecycle-viewmodel-compose", version.ref = "androidx-hilt" }
javax-inject = { group = "javax.inject", name = "javax.inject", version = "1" }  # JSR-330 annotations for the pure-Kotlin domain module

# ──────────────────────────────────────────────
# Room 3 (Database) — KSP only, coroutine APIs
# ──────────────────────────────────────────────
room3-runtime = { group = "androidx.room3", name = "room3-runtime", version.ref = "room3" }
room3-compiler = { group = "androidx.room3", name = "room3-compiler", version.ref = "room3" }

# ──────────────────────────────────────────────
# DataStore
# ──────────────────────────────────────────────
datastore-preferences = { group = "androidx.datastore", name = "datastore-preferences", version.ref = "datastore" }

# ──────────────────────────────────────────────
# Kotlin
# ──────────────────────────────────────────────
kotlinx-serialization-json = { group = "org.jetbrains.kotlinx", name = "kotlinx-serialization-json", version.ref = "kotlinx-serialization" }
kotlinx-coroutines-core = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-core", version.ref = "kotlinx-coroutines" }
kotlinx-coroutines-android = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-android", version.ref = "kotlinx-coroutines" }
kotlinx-coroutines-test = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-test", version.ref = "kotlinx-coroutines" }

# ──────────────────────────────────────────────
# Images (Coil 3) — local/content URIs only; add a coil-network-* artifact only when remote images are needed (it brings an HTTP client: ask first)
# ──────────────────────────────────────────────
coil-compose = { group = "io.coil-kt.coil3", name = "coil-compose", version.ref = "coil" }

# ──────────────────────────────────────────────
# Firebase (versions from the BoM)
# ──────────────────────────────────────────────
firebase-bom = { group = "com.google.firebase", name = "firebase-bom", version.ref = "firebase-bom" }
firebase-auth = { group = "com.google.firebase", name = "firebase-auth" }
firebase-firestore = { group = "com.google.firebase", name = "firebase-firestore" }
firebase-functions = { group = "com.google.firebase", name = "firebase-functions" }
firebase-config = { group = "com.google.firebase", name = "firebase-config" }
firebase-analytics = { group = "com.google.firebase", name = "firebase-analytics" }
firebase-appcheck-playintegrity = { group = "com.google.firebase", name = "firebase-appcheck-playintegrity" }
firebase-ai = { group = "com.google.firebase", name = "firebase-ai" }

# ──────────────────────────────────────────────
# Identity, payments, maps
# ──────────────────────────────────────────────
credentials = { group = "androidx.credentials", name = "credentials", version.ref = "credentials" }
credentials-play-services-auth = { group = "androidx.credentials", name = "credentials-play-services-auth", version.ref = "credentials" }
googleid = { group = "com.google.android.libraries.identity.googleid", name = "googleid", version.ref = "googleid" }
revenuecat-purchases = { group = "com.revenuecat.purchases", name = "purchases", version.ref = "revenuecat" }
revenuecat-purchases-ui = { group = "com.revenuecat.purchases", name = "purchases-ui", version.ref = "revenuecat" }
maps-compose = { group = "com.google.maps.android", name = "maps-compose", version.ref = "maps-compose" }

# ──────────────────────────────────────────────
# Testing
# ──────────────────────────────────────────────
junit-bom = { group = "org.junit", name = "junit-bom", version.ref = "junit-bom" }
junit-jupiter = { group = "org.junit.jupiter", name = "junit-jupiter" }
junit-jupiter-params = { group = "org.junit.jupiter", name = "junit-jupiter-params" }
junit-platform-launcher = { group = "org.junit.platform", name = "junit-platform-launcher" }
junit-vintage-engine = { group = "org.junit.vintage", name = "junit-vintage-engine" }
junit4 = { group = "junit", name = "junit", version.ref = "junit4" }
turbine = { group = "app.cash.turbine", name = "turbine", version.ref = "turbine" }
robolectric = { group = "org.robolectric", name = "robolectric", version.ref = "robolectric" }
screenshot-validation-api = { group = "com.android.tools.screenshot", name = "screenshot-validation-api", version.ref = "screenshot" }
compose-rules-detekt = { group = "io.nlopez.compose.rules", name = "detekt", version.ref = "compose-rules-detekt" }

[bundles]
compose = [
    "compose-ui",
    "compose-ui-graphics",
    "compose-material3",
    "compose-ui-tooling-preview",
]
compose-debug = [
    "compose-ui-tooling",
    "compose-ui-test-manifest",
]
lifecycle = [
    "lifecycle-runtime-compose",
    "lifecycle-viewmodel-compose",
]
navigation = [
    "navigation3-runtime",
    "navigation3-ui",
    "lifecycle-viewmodel-navigation3",
]
testing = [
    "junit-jupiter",
    "junit-jupiter-params",
    "turbine",
    "kotlinx-coroutines-test",
]

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
android-library = { id = "com.android.library", version.ref = "agp" }
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
room3 = { id = "androidx.room3", version.ref = "room3" }
google-services = { id = "com.google.gms.google-services", version.ref = "google-services" }
junit5 = { id = "de.mannodermaus.android-junit", version.ref = "junit5-plugin" }
screenshot = { id = "com.android.compose.screenshot", version.ref = "screenshot" }
detekt = { id = "io.gitlab.arturbosch.detekt", version.ref = "detekt" }
ktlint = { id = "org.jlleitschuh.gradle.ktlint", version.ref = "ktlint-gradle" }
```

### What Changed in the 2026 Stack

- **AGP 9 built-in Kotlin**: Android modules no longer apply `org.jetbrains.kotlin.android`. `android.kotlinOptions {}` is gone — use `kotlin { compilerOptions { … } }`. Pure JVM modules (`:domain`) apply `org.jetbrains.kotlin.jvm`. See the official `agp-9-upgrade` skill.
- **No kapt**: the `kotlin-kapt` plugin is incompatible with built-in Kotlin, and Room 3 is KSP-only. Every processor (Hilt, Room) runs through KSP.
- **Navigation 3** replaces `navigation-compose`; `hilt-navigation-compose` is not used (Hilt ViewModels come from `hilt-lifecycle-viewmodel-compose`).
- **Room 3** lives in a new package and Maven group (`androidx.room3`); there is no `room-ktx` (every DAO API is coroutine-based).
- **Coil 3** moved to the `io.coil-kt.coil3` group; network loading is a separate artifact.
- **Screenshot tests**: AGP 9.4 uses the standalone `com.android.compose.screenshot` plugin (deprecated in favour of AGP test suites from AGP 9.5.0-alpha03). Migrate when the project moves to AGP ≥ 9.5.

### Using Bundles

**Bundles group related dependencies together so modules can add them in a single line.**

```kotlin
// ✅ Using bundles - clean and consistent
dependencies {
    implementation(libs.bundles.compose)
    implementation(libs.bundles.lifecycle)
    implementation(libs.bundles.navigation)
    debugImplementation(libs.bundles.compose.debug)
    testImplementation(libs.bundles.testing)
}
```

```kotlin
// ❌ Listing every dependency individually in every module
dependencies {
    implementation(libs.compose.ui)
    implementation(libs.compose.ui.graphics)
    implementation(libs.compose.material3)
    implementation(libs.compose.ui.tooling.preview)
    implementation(libs.lifecycle.runtime.compose)
    implementation(libs.lifecycle.viewmodel.compose)
    // ... 20 more lines
}
```

---

## 📐 Root build.gradle.kts

**The root build file declares all plugins used across the project but does NOT apply them. Each module applies only the plugins it needs.**

```kotlin
// Root build.gradle.kts
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.android.library) apply false
    alias(libs.plugins.kotlin.jvm) apply false
    alias(libs.plugins.kotlin.serialization) apply false
    alias(libs.plugins.compose.compiler) apply false
    alias(libs.plugins.hilt) apply false
    alias(libs.plugins.ksp) apply false
    alias(libs.plugins.room3) apply false
    alias(libs.plugins.google.services) apply false
    alias(libs.plugins.junit5) apply false
    alias(libs.plugins.screenshot) apply false
    alias(libs.plugins.detekt) apply false
    alias(libs.plugins.ktlint) apply false
}
```

> With AGP 9 built-in Kotlin there is no `org.jetbrains.kotlin.android` plugin to declare. Android modules get Kotlin support from AGP itself.

### Why `apply false`?

**Declaring plugins at the root with `apply false` resolves plugin versions once for the entire project. Each module then applies only the plugins it actually needs, avoiding unnecessary configuration overhead.**

```kotlin
// ✅ Root declares, modules apply
// root build.gradle.kts
plugins {
    alias(libs.plugins.hilt) apply false
}

// feature/home/build.gradle.kts
plugins {
    alias(libs.plugins.hilt)  // Applied here where it's needed
}
```

```kotlin
// ❌ Each module resolving its own plugin version
// feature/home/build.gradle.kts
plugins {
    id("com.google.dagger.hilt.android") version "2.60.1"  // Duplicated everywhere
}
```

---

## 📐 settings.gradle.kts

**The settings file configures plugin repositories, dependency repositories, and declares all modules in the project.**

```kotlin
pluginManagement {
    repositories {
        google {
            content {
                includeGroupByRegex("com\\.android.*")
                includeGroupByRegex("com\\.google.*")
                includeGroupByRegex("androidx.*")
            }
        }
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "MyApp"

// App module
include(":app")

// Feature modules
include(":feature:home")
include(":feature:search")
include(":feature:profile")

// Core modules
include(":core:database")
include(":core:domain")
include(":core:model")
include(":core:ui")
include(":core:testing")
```

### Repository Mode

**`FAIL_ON_PROJECT_REPOS` ensures all repositories are declared centrally in `settings.gradle.kts` rather than scattered across individual module build files.**

```kotlin
// ✅ Centralized in settings.gradle.kts
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
```

```kotlin
// ❌ Declaring repositories in individual modules
// feature/home/build.gradle.kts
repositories {
    google()       // Build will fail with FAIL_ON_PROJECT_REPOS
    mavenCentral()
}
```

---

## 📐 App Module build.gradle.kts

**The app module is the entry point of the application. It applies the `android.application` plugin and depends on all feature modules.**

> In ARC Labs projects the shared parts of these blocks (SDK levels, Java target, Compose, Hilt, zero-warnings, test setup) live in `build-logic/` convention plugins (`arclabs.android.application`, `arclabs.android.library`, `arclabs.android.feature`, `arclabs.android.compose`, `arclabs.android.hilt`, `arclabs.android.room`, `arclabs.jvm.library`, `arclabs.android.lint`). The expanded form below shows what those plugins configure. See [Module Structure](../Quality/module-structure.md#convention-plugins-pattern).

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.compose.compiler)
    alias(libs.plugins.hilt)
    alias(libs.plugins.ksp)
}

android {
    namespace = "com.arclabs.myapp"
    compileSdk = libs.versions.compileSdk.get().toInt()

    defaultConfig {
        applicationId = "com.arclabs.myapp"
        minSdk = libs.versions.minSdk.get().toInt()
        targetSdk = libs.versions.targetSdk.get().toInt()
        versionCode = 1
        versionName = "1.0.0"

        testInstrumentationRunner = "com.arclabs.myapp.HiltTestRunner"
    }

    buildTypes {
        debug {
            isDebuggable = true
            isMinifyEnabled = false
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-debug"
            enableUnitTestCoverage = true
        }

        release {
            isDebuggable = false
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    buildFeatures {
        compose = true
        buildConfig = true
    }

    // Zero warnings: Android Lint warnings fail the build
    lint {
        warningsAsErrors = true
        abortOnError = true
    }

    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
            excludes += "/META-INF/LICENSE*"
        }
    }
}

// AGP 9 built-in Kotlin: `kotlin.compilerOptions` replaces `android.kotlinOptions`.
// jvmTarget defaults to compileOptions.targetCompatibility (17).
kotlin {
    compilerOptions {
        allWarningsAsErrors = true
    }
}

dependencies {
    // Feature modules
    implementation(project(":feature:home"))
    implementation(project(":feature:search"))
    implementation(project(":feature:profile"))

    // Core modules
    implementation(project(":core:database"))
    implementation(project(":core:domain"))
    implementation(project(":core:model"))
    implementation(project(":core:ui"))

    // AndroidX Core
    implementation(libs.core.ktx)
    implementation(libs.activity.compose)

    // Compose (BOM + explicit material3 pin for M3 Expressive)
    implementation(platform(libs.compose.bom))
    implementation(libs.bundles.compose)
    implementation(libs.compose.material3.navigation.suite)
    debugImplementation(libs.bundles.compose.debug)

    // Lifecycle + Navigation 3
    implementation(libs.bundles.lifecycle)
    implementation(libs.bundles.navigation)

    // Hilt (KSP — never kapt)
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)
    implementation(libs.hilt.lifecycle.viewmodel.compose)

    // Testing (JUnit 5 via the mannodermaus plugin applied by the convention plugin)
    testImplementation(platform(libs.junit.bom))
    testImplementation(libs.bundles.testing)
    testRuntimeOnly(libs.junit.platform.launcher)
    androidTestImplementation(platform(libs.compose.bom))
    androidTestImplementation(libs.compose.ui.test.junit4)
}
```

---

## 📐 Library Module build.gradle.kts

**Feature and core modules use the `android.library` plugin. They expose only what they need and keep dependencies minimal.**

### Feature Module Example (feature/home)

With convention plugins (the ARC Labs default), a feature module is a few lines:

```kotlin
plugins {
    id("arclabs.android.feature")   // library + compose + hilt + nav3 + test setup + zero-warnings
}

android {
    namespace = "com.arclabs.myapp.feature.home"
}

dependencies {
    implementation(project(":core:ui"))
    implementation(libs.coil.compose)
}
```

Expanded, the convention plugin applies the equivalent of:

```kotlin
plugins {
    alias(libs.plugins.android.library)
    alias(libs.plugins.compose.compiler)
    alias(libs.plugins.kotlin.serialization)
    alias(libs.plugins.hilt)
    alias(libs.plugins.ksp)
    alias(libs.plugins.junit5)
}

android {
    compileSdk = libs.versions.compileSdk.get().toInt()

    defaultConfig {
        minSdk = libs.versions.minSdk.get().toInt()
        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        consumerProguardFiles("consumer-rules.pro")
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    buildFeatures {
        compose = true
    }

    lint {
        warningsAsErrors = true
        abortOnError = true
    }
}

kotlin {
    compilerOptions {
        allWarningsAsErrors = true
    }
}

dependencies {
    implementation(project(":core:domain"))
    implementation(project(":core:model"))

    implementation(platform(libs.compose.bom))
    implementation(libs.bundles.compose)
    debugImplementation(libs.bundles.compose.debug)
    implementation(libs.bundles.lifecycle)
    implementation(libs.bundles.navigation)
    implementation(libs.kotlinx.serialization.json)

    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)
    implementation(libs.hilt.lifecycle.viewmodel.compose)

    testImplementation(platform(libs.junit.bom))
    testImplementation(libs.bundles.testing)
    testRuntimeOnly(libs.junit.platform.launcher)
    testImplementation(project(":core:testing"))
}
```

### Core Module Example (core/database — Room 3)

```kotlin
plugins {
    alias(libs.plugins.android.library)
    alias(libs.plugins.ksp)
    alias(libs.plugins.room3)
    alias(libs.plugins.hilt)
}

android {
    namespace = "com.arclabs.myapp.core.database"
    compileSdk = libs.versions.compileSdk.get().toInt()

    defaultConfig {
        minSdk = libs.versions.minSdk.get().toInt()
        consumerProguardFiles("consumer-rules.pro")
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
}

// Required by the Room Gradle plugin; commit the generated schemas
room3 {
    schemaDirectory(layout.projectDirectory.dir("schemas").asFile.path)
}

dependencies {
    implementation(project(":core:model"))

    implementation(libs.room3.runtime)
    ksp(libs.room3.compiler)

    implementation(libs.kotlinx.coroutines.core)

    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)

    testImplementation(platform(libs.junit.bom))
    testImplementation(libs.bundles.testing)
    testRuntimeOnly(libs.junit.platform.launcher)
}
```

> Plugin id `androidx.room3`, extension `room3` (type `RoomExtension`); this `schemaDirectory(...)` form is the one used by the Gradle-verified ARCDevTools-Android 0.2.0 `arc.android.room` convention plugin.

### Pure Kotlin Module Example (core/domain)

```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm)   // the mannodermaus plugin is for Android modules only
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

kotlin {
    compilerOptions {
        jvmTarget = org.jetbrains.kotlin.gradle.dsl.JvmTarget.JVM_17
        allWarningsAsErrors = true
    }
}

dependencies {
    implementation(project(":core:model"))
    implementation(libs.kotlinx.coroutines.core)

    // JSR-330 annotations only (`@Inject constructor` on use cases) — no Hilt, no Android
    implementation(libs.javax.inject)

    testImplementation(platform(libs.junit.bom))
    testImplementation(libs.bundles.testing)
    testRuntimeOnly(libs.junit.platform.launcher)
}

tasks.test {
    useJUnitPlatform()
}
```

---

## 🔧 Build Variants

**Build variants combine build types and product flavors to produce different versions of your app from a single codebase.**

### Build Types

```kotlin
android {
    buildTypes {
        debug {
            isDebuggable = true
            isMinifyEnabled = false
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-debug"

            buildConfigField("String", "BASE_URL", "\"https://api-dev.arclabs.com\"")
            buildConfigField("Boolean", "ENABLE_LOGGING", "true")
        }

        create("staging") {
            initWith(getByName("debug"))
            applicationIdSuffix = ".staging"
            versionNameSuffix = "-staging"

            buildConfigField("String", "BASE_URL", "\"https://api-staging.arclabs.com\"")
            buildConfigField("Boolean", "ENABLE_LOGGING", "true")
        }

        release {
            isDebuggable = false
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )

            buildConfigField("String", "BASE_URL", "\"https://api.arclabs.com\"")
            buildConfigField("Boolean", "ENABLE_LOGGING", "false")
        }
    }
}
```

### Product Flavors

```kotlin
android {
    flavorDimensions += "environment"

    productFlavors {
        create("free") {
            dimension = "environment"
            applicationIdSuffix = ".free"
            versionNameSuffix = "-free"
            buildConfigField("Boolean", "IS_PREMIUM", "false")
            buildConfigField("Int", "MAX_FAVORITES", "5")
        }

        create("premium") {
            dimension = "environment"
            applicationIdSuffix = ".premium"
            versionNameSuffix = "-premium"
            buildConfigField("Boolean", "IS_PREMIUM", "true")
            buildConfigField("Int", "MAX_FAVORITES", "999")
        }
    }
}
```

### Accessing BuildConfig in Code

```kotlin
// ✅ Using BuildConfig fields for environment-specific values
class NetworkModule {
    fun provideBaseUrl(): String = BuildConfig.BASE_URL

    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder().apply {
            if (BuildConfig.ENABLE_LOGGING) {
                addInterceptor(HttpLoggingInterceptor().apply {
                    level = HttpLoggingInterceptor.Level.BODY
                })
            }
        }.build()
    }
}
```

```kotlin
// ❌ Hardcoding environment values
class NetworkModule {
    fun provideBaseUrl(): String = "https://api.arclabs.com" // Not configurable per variant
}
```

---

## 🏗️ ProGuard / R8

**R8 is the default code shrinker and obfuscator for Android release builds. ProGuard rules tell R8 which classes and members to keep.**

### Basic proguard-rules.pro

```proguard
# ──────────────────────────────────────────────
# General
# ──────────────────────────────────────────────
-keepattributes *Annotation*, InnerClasses, Signature, Exceptions
-keepattributes SourceFile, LineNumberTable

# ──────────────────────────────────────────────
# Kotlinx Serialization
# ──────────────────────────────────────────────
-keepattributes *Annotation*, InnerClasses
-dontnote kotlinx.serialization.AnnotationsKt

-keepclassmembers class kotlinx.serialization.json.** {
    *** Companion;
}
-keepclasseswithmembers class kotlinx.serialization.json.** {
    kotlinx.serialization.KSerializer serializer(...);
}

# Keep serializable classes and their serializers
-keep,includedescriptorclasses class com.arclabs.myapp.**$$serializer { *; }
-keepclassmembers class com.arclabs.myapp.** {
    *** Companion;
}
-keepclasseswithmembers class com.arclabs.myapp.** {
    kotlinx.serialization.KSerializer serializer(...);
}

# ──────────────────────────────────────────────
# Hilt / Dagger
# ──────────────────────────────────────────────
-keep class dagger.hilt.** { *; }
-keep class javax.inject.** { *; }
-keep class * extends dagger.hilt.android.internal.managers.ViewComponentManager$FragmentContextWrapper { *; }

# ──────────────────────────────────────────────
# Room 3
# ──────────────────────────────────────────────
# Room 3 (androidx.room3) generates Kotlin code only. Do not copy Room 2
# (androidx.room) keep rules; add rules only if R8 reports a concrete issue
# (use the official r8-analyzer skill to audit keep rules).

# ──────────────────────────────────────────────
# Retrofit / OkHttp — Vapor phase only (v1 apps have no HTTP client)
# ──────────────────────────────────────────────
-dontwarn okhttp3.**
-dontwarn okio.**
-dontwarn retrofit2.**
-keep class retrofit2.** { *; }
-keepattributes Signature
-keepattributes Exceptions

# Keep API interfaces
-keep,allowobfuscation,allowshrinking interface com.arclabs.myapp.core.network.api.**

# ──────────────────────────────────────────────
# Coroutines
# ──────────────────────────────────────────────
-keepnames class kotlinx.coroutines.internal.MainDispatcherFactory {}
-keepnames class kotlinx.coroutines.CoroutineExceptionHandler {}
-keepclassmembers class kotlinx.coroutines.** {
    volatile <fields>;
}

# ──────────────────────────────────────────────
# Compose
# ──────────────────────────────────────────────
-dontwarn androidx.compose.**
```

### Consumer ProGuard Rules

**Library modules should provide `consumer-rules.pro` for rules that consuming modules need.**

```proguard
# consumer-rules.pro in core/network
-keep class com.arclabs.myapp.core.network.model.** { *; }
```

```kotlin
// In the library module's build.gradle.kts
android {
    defaultConfig {
        consumerProguardFiles("consumer-rules.pro")
    }
}
```

---

## ⚡ Build Performance

**Optimizing Gradle build performance is critical for developer productivity. These settings can reduce build times by 30-60%.**

### gradle.properties

```properties
# ──────────────────────────────────────────────
# JVM Configuration
# ──────────────────────────────────────────────
org.gradle.jvmargs=-Xmx4g -XX:MaxMetaspaceSize=1g -XX:+HeapDumpOnOutOfMemoryError

# ──────────────────────────────────────────────
# Gradle Performance
# ──────────────────────────────────────────────
org.gradle.parallel=true
org.gradle.caching=true
org.gradle.configuration-cache=true
org.gradle.configureondemand=true

# ──────────────────────────────────────────────
# Kotlin Performance
# ──────────────────────────────────────────────
kotlin.incremental=true
kotlin.code.style=official

# ──────────────────────────────────────────────
# Android Specific
# ──────────────────────────────────────────────
android.nonTransitiveRClass=true
```

> AGP 9 removed the `android.defaults.buildfeatures.*` flags and defaults `android.useAndroidX` to `true`; enable `buildConfig` per module in the `android { buildFeatures {} }` block. After an AGP 9 migration, also remove `android.builtInKotlin`, `android.newDsl`, `android.uniquePackageNames` and `android.enableAppCompileTimeRClass` (official `agp-9-upgrade` skill).

### What Each Setting Does

| Setting | Effect |
|---------|--------|
| `org.gradle.jvmargs=-Xmx4g` | Allocates 4GB heap to the Gradle daemon |
| `org.gradle.parallel=true` | Builds independent modules in parallel |
| `org.gradle.caching=true` | Caches task outputs for reuse across builds |
| `org.gradle.configuration-cache=true` | Caches the build configuration phase |
| `kotlin.incremental=true` | Only recompiles changed Kotlin files |
| `android.nonTransitiveRClass=true` | Each module only sees its own R class (faster builds, smaller APK) |

### Profiling Builds

```bash
# Generate a build scan for performance analysis
./gradlew assembleDebug --scan

# Profile build with detailed timing
./gradlew assembleDebug --profile

# Show configuration time per project
./gradlew assembleDebug --configuration-cache-problems=warn
```

---

## 🔧 Useful Gradle Tasks

**Common commands for daily development workflows.**

### Build Tasks

| Task | Description |
|------|-------------|
| `./gradlew assembleDebug` | Build debug APK |
| `./gradlew assembleRelease` | Build release APK |
| `./gradlew bundleRelease` | Build release AAB (for Play Store) |
| `./gradlew installDebug` | Build and install debug on connected device |
| `./gradlew clean` | Delete all build outputs |

### Testing Tasks

| Task | Description |
|------|-------------|
| `./gradlew test` | Run all unit tests |
| `./gradlew testDebugUnitTest` | Run unit tests for debug variant |
| `./gradlew :feature:home:test` | Run tests for a specific module |
| `./gradlew connectedAndroidTest` | Run instrumented tests on device |
| `./gradlew createDebugUnitTestCoverageReport` | Generate unit test coverage report (AGP built-in JaCoCo, needs `enableUnitTestCoverage = true`) |
| `./gradlew validateDebugScreenshotTest` | Validate Compose preview screenshots against reference images |
| `./gradlew updateDebugScreenshotTest` | Record/update screenshot reference images |

### Code Quality Tasks

| Task | Description |
|------|-------------|
| `./gradlew lint` | Run Android Lint analysis |
| `./gradlew lintDebug` | Run lint for debug variant only |
| `./gradlew ktlintCheck` | Check Kotlin code style |
| `./gradlew ktlintFormat` | Auto-fix Kotlin code style issues |
| `./gradlew detekt` | Run detekt static analysis |

### Dependency Tasks

| Task | Description |
|------|-------------|
| `./gradlew dependencies` | Show full dependency tree |
| `./gradlew :app:dependencies --configuration releaseRuntimeClasspath` | Show release dependencies |
| `./gradlew dependencyUpdates` | Check for newer dependency versions |
| `./gradlew buildEnvironment` | Show buildscript dependencies |

### Module-Specific Execution

```bash
# ✅ Run tasks for a specific module
./gradlew :feature:home:test
./gradlew :core:database:lint
./gradlew :app:assembleDebug

# ✅ Run multiple tasks
./gradlew clean assembleDebug test

# ✅ Run with stacktrace for debugging
./gradlew assembleDebug --stacktrace

# ✅ Run with info-level logging
./gradlew assembleDebug --info
```

---

## 🔍 Troubleshooting

**Common Gradle issues and their solutions.**

### Dependency Resolution Failures

```
> Could not resolve all files for configuration ':app:debugRuntimeClasspath'.
```

**Solutions:**

```bash
# 1. Clear Gradle caches
./gradlew clean
rm -rf ~/.gradle/caches/

# 2. Refresh dependencies
./gradlew build --refresh-dependencies

# 3. Check for version conflicts
./gradlew :app:dependencies --configuration debugRuntimeClasspath | grep -i "FAILED"
```

### Version Conflicts

```kotlin
// ✅ Force a specific version when conflicts arise
configurations.all {
    resolutionStrategy {
        force("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.11.0")
    }
}
```

```kotlin
// ✅ Exclude transitive dependency causing conflict
dependencies {
    implementation(libs.some.library) {
        exclude(group = "org.jetbrains.kotlinx", module = "kotlinx-coroutines-core")
    }
}
```

### Build Cache Issues

```bash
# Clear local build cache
./gradlew cleanBuildCache

# Disable cache temporarily for debugging
./gradlew assembleDebug --no-build-cache

# Disable configuration cache temporarily
./gradlew assembleDebug -Dorg.gradle.configuration-cache=false
```

### KSP / Hilt Issues

```
> [ksp] InjectProcessingStep was unable to process 'MyViewModel' because 'SomeDependency' could not be resolved.
```

**Solutions:**

```kotlin
// 1. Ensure KSP is applied AFTER Hilt
plugins {
    alias(libs.plugins.hilt)
    alias(libs.plugins.ksp)  // KSP must come after Hilt
}

// 2. Ensure all @Inject dependencies are available
// Check that the module declaring SomeDependency is in the dependency graph
dependencies {
    implementation(project(":core:database")) // Missing dependency?
}
```

### Out of Memory Errors

```properties
# Increase JVM heap in gradle.properties
org.gradle.jvmargs=-Xmx6g -XX:MaxMetaspaceSize=1g -XX:+HeapDumpOnOutOfMemoryError
```

### Configuration Cache Failures

```
> Configuration cache state could not be cached
```

**Solutions:**

```bash
# Identify the incompatible task
./gradlew assembleDebug --configuration-cache-problems=warn

# Disable configuration cache temporarily
./gradlew assembleDebug -Dorg.gradle.configuration-cache=false
```

### Compose Compiler Compatibility

```
> Compose Compiler requires Kotlin version X but you are using Y.
```

**Solution:** The Compose Compiler plugin version must match the Kotlin version. Since Kotlin 2.0, the Compose Compiler is a Kotlin compiler plugin.

```toml
# In libs.versions.toml - both must use the same version
[versions]
kotlin = "2.4.21"

[plugins]
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
```

---

## ✅ Gradle Checklist

**Use this checklist when setting up a new project or auditing an existing one.**

### Project Setup

- [ ] Using Kotlin DSL (`build.gradle.kts`) for all build files
- [ ] Version Catalog (`libs.versions.toml`) for all dependencies
- [ ] Root `build.gradle.kts` only declares plugins with `apply false`
- [ ] `settings.gradle.kts` uses `FAIL_ON_PROJECT_REPOS`
- [ ] All modules included in `settings.gradle.kts`
- [ ] `gradle.properties` configured for optimal performance

### Each Module

- [ ] Unique `namespace` declared in `android {}` block
- [ ] `compileSdk`, `minSdk` referenced from version catalog
- [ ] `compileOptions` set to Java 17 (Kotlin `jvmTarget` follows it with AGP 9 built-in Kotlin)
- [ ] No `org.jetbrains.kotlin.android` / `kotlin-kapt` plugin, no `kotlinOptions {}`
- [ ] `allWarningsAsErrors = true` and Lint `warningsAsErrors = true` (via convention plugin)
- [ ] Dependencies use version catalog references (`libs.xxx`)
- [ ] Test dependencies include JUnit 5 (mannodermaus plugin), Turbine, coroutines-test; fakes from `:core:testing`
- [ ] Consumer ProGuard rules provided for library modules

### Build Variants

- [ ] Debug variant has `applicationIdSuffix = ".debug"`
- [ ] Release variant has `isMinifyEnabled = true` and `isShrinkResources = true`
- [ ] Staging variant configured (if applicable)
- [ ] `BuildConfig` fields for environment-specific values
- [ ] ProGuard/R8 rules cover serialization and Hilt (Retrofit only in the Vapor phase)

### Code Quality

- [ ] ktlint configured for Kotlin style checking
- [ ] detekt configured for static analysis
- [ ] Android Lint enabled with `warningsAsErrors`; no baseline hiding new code
- [ ] Pre-commit hooks run lint checks

### CI/CD

- [ ] Gradle wrapper committed to version control
- [ ] `gradle-wrapper.properties` specifies exact Gradle version
- [ ] CI uses `--no-daemon` for reproducible builds
- [ ] Build cache enabled in CI
- [ ] Dependency lock files used for reproducible builds

---

## 🚫 Common Mistakes

### Mistake 1: Hardcoded Dependency Versions

```kotlin
// ❌ WRONG - Hardcoded versions scattered across modules
dependencies {
    implementation("androidx.compose.material3:material3:1.2.1")
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.google.dagger:hilt-android:2.51.1")
}
```

```kotlin
// ✅ CORRECT - Version catalog references
dependencies {
    implementation(libs.compose.material3)
    implementation(libs.retrofit)
    implementation(libs.hilt.android)
}
```

**Why:** Hardcoded versions lead to inconsistencies across modules, make upgrades painful, and increase the risk of version conflicts.

---

### Mistake 2: Groovy DSL Instead of Kotlin DSL

```groovy
// ❌ WRONG - Groovy DSL (build.gradle)
android {
    compileSdkVersion 35
    defaultConfig {
        minSdkVersion 26
        targetSdkVersion 35
    }
}

dependencies {
    implementation "androidx.core:core-ktx:1.15.0"
}
```

```kotlin
// ✅ CORRECT - Kotlin DSL (build.gradle.kts)
android {
    compileSdk = libs.versions.compileSdk.get().toInt()
    defaultConfig {
        minSdk = libs.versions.minSdk.get().toInt()
        targetSdk = libs.versions.targetSdk.get().toInt()
    }
}

dependencies {
    implementation(libs.core.ktx)
}
```

**Why:** Kotlin DSL provides type safety, IDE autocompletion, compile-time error checking, and consistent syntax with the rest of the Kotlin codebase.

---

### Mistake 3: Missing ProGuard Rules for Release

```kotlin
// ❌ WRONG - Release build without shrinking or obfuscation
buildTypes {
    release {
        isMinifyEnabled = false  // No code shrinking!
        // No proguardFiles configured
    }
}
```

```kotlin
// ✅ CORRECT - Properly configured release build
buildTypes {
    release {
        isMinifyEnabled = true
        isShrinkResources = true
        proguardFiles(
            getDefaultProguardFile("proguard-android-optimize.txt"),
            "proguard-rules.pro"
        )
    }
}
```

**Why:** Without R8/ProGuard, release APKs contain unused code, are larger, and expose internal class and method names making reverse engineering trivial.

---

### Mistake 4: Using `implementation` When `api` Is Needed (or Vice Versa)

```kotlin
// ❌ WRONG - Using api when implementation suffices (leaks transitive deps)
// core/network/build.gradle.kts
dependencies {
    api(libs.retrofit)       // Exposes Retrofit to ALL consumers unnecessarily
    api(libs.okhttp)         // Exposes OkHttp to ALL consumers unnecessarily
    api(libs.okhttp.logging) // Exposes logging interceptor unnecessarily
}
```

```kotlin
// ✅ CORRECT - Use implementation by default, api only when consumers need the type
// core/network/build.gradle.kts
dependencies {
    implementation(libs.retrofit)       // Internal implementation detail
    implementation(libs.okhttp)         // Internal implementation detail
    implementation(libs.okhttp.logging) // Internal implementation detail
    api(project(":core:model"))         // Consumers need model classes from network APIs
}
```

**Why:** `api` exposes transitive dependencies to all consumers, increasing compile time and coupling. Use `implementation` by default and only use `api` when the dependency's types appear in the module's public API.

---

## 📚 Further Reading

### Official Documentation

- [Gradle User Guide](https://docs.gradle.org/current/userguide/userguide.html) - Comprehensive Gradle documentation
- [Android Gradle Plugin DSL Reference](https://developer.android.com/reference/tools/gradle-api) - AGP API reference
- [Version Catalogs](https://docs.gradle.org/current/userguide/platforms.html#sub:version-catalog) - Central dependency management
- [Kotlin DSL Primer](https://docs.gradle.org/current/userguide/kotlin_dsl.html) - Migrating from Groovy to Kotlin DSL

### Android-Specific Guides

- [Configure Your Build](https://developer.android.com/build) - Android build configuration guide
- [Shrink, Obfuscate, and Optimize](https://developer.android.com/build/shrink-code) - R8 and ProGuard guide
- [Build Variants](https://developer.android.com/build/build-variants) - Build types and product flavors
- [Modularization Guide](https://developer.android.com/topic/modularization) - Multi-module architecture

### Performance

- [Optimize Build Speed](https://developer.android.com/build/optimize-your-build) - Build performance tips
- [Profile Your Build](https://developer.android.com/build/profile-your-build) - Build profiling tools
- [Configuration Cache](https://docs.gradle.org/current/userguide/configuration_cache.html) - Configuration cache guide

### ARC Labs Standards

- [ARCKnowledge](https://github.com/arclabs-studio/ARCKnowledge) - ARC Labs knowledge base
- [ARCDevTools-Android](https://github.com/arclabs-studio/ARCDevTools-Android) - Development tools and configurations
