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
- **Convention Plugins** - Share build logic across modules using `buildSrc` or composite builds

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
    compileSdkVersion 35 // No IDE help, typos caught at runtime

    defaultConfig {
        minSdkVersion 26
        targetSdkVersion 35
    }
}

dependencies {
    implementation "androidx.compose.material3:material3:1.2.1" // String-based, error-prone
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
│   ├── network/
│   │   ├── build.gradle.kts      (core module - networking)
│   │   └── src/
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
└── buildSrc/                     (optional - convention plugins)
    ├── build.gradle.kts
    └── src/main/kotlin/
        └── AndroidLibraryConventionPlugin.kt
```

### Module Dependency Rules

```
app -> feature:* -> core:*
feature:home -> core:domain, core:ui, core:model
core:network -> core:model
core:database -> core:model
core:domain -> core:model (pure Kotlin, no Android dependencies)
```

---

## 📦 Version Catalogs (libs.versions.toml)

**Version Catalogs provide a single source of truth for all dependency versions. Located at `gradle/libs.versions.toml`, they enable type-safe dependency references across all modules.**

```toml
[versions]
# Kotlin & Android
kotlin = "2.0.21"
agp = "8.7.3"
ksp = "2.0.21-1.0.28"

# Compose
compose-bom = "2024.12.01"
compose-navigation = "2.8.5"

# AndroidX
lifecycle = "2.8.7"
core-ktx = "1.15.0"
activity-compose = "1.9.3"
datastore = "1.1.1"

# Dependency Injection
hilt = "2.51.1"
hilt-navigation-compose = "1.2.0"

# Database
room = "2.6.1"

# Network
retrofit = "2.9.0"
okhttp = "4.12.0"
kotlinx-serialization = "1.7.3"

# Async
kotlinx-coroutines = "1.9.0"

# Images
coil = "2.7.0"

# Testing
junit5 = "5.10.3"
mockk = "1.13.13"
turbine = "1.2.0"

# Static Analysis
detekt = "1.23.7"
ktlint = "12.1.2"

# SDK Versions
compileSdk = "35"
minSdk = "26"
targetSdk = "35"

[libraries]
# ──────────────────────────────────────────────
# Compose
# ──────────────────────────────────────────────
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }
compose-material-icons = { group = "androidx.compose.material", name = "material-icons-extended" }
compose-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
compose-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
compose-ui-test-junit4 = { group = "androidx.compose.ui", name = "ui-test-junit4" }
compose-ui-test-manifest = { group = "androidx.compose.ui", name = "ui-test-manifest" }
compose-runtime = { group = "androidx.compose.runtime", name = "runtime" }

# ──────────────────────────────────────────────
# AndroidX Core
# ──────────────────────────────────────────────
core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "core-ktx" }
activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "activity-compose" }

# ──────────────────────────────────────────────
# Lifecycle
# ──────────────────────────────────────────────
lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version.ref = "lifecycle" }
lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycle" }

# ──────────────────────────────────────────────
# Navigation
# ──────────────────────────────────────────────
navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "compose-navigation" }

# ──────────────────────────────────────────────
# Hilt (Dependency Injection)
# ──────────────────────────────────────────────
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version.ref = "hilt-navigation-compose" }

# ──────────────────────────────────────────────
# Room (Database)
# ──────────────────────────────────────────────
room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }
room-testing = { group = "androidx.room", name = "room-testing", version.ref = "room" }

# ──────────────────────────────────────────────
# Network
# ──────────────────────────────────────────────
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-kotlinx-serialization = { group = "com.squareup.retrofit2", name = "converter-kotlinx-serialization", version.ref = "retrofit" }
okhttp = { group = "com.squareup.okhttp3", name = "okhttp", version.ref = "okhttp" }
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttp" }
kotlinx-serialization-json = { group = "org.jetbrains.kotlinx", name = "kotlinx-serialization-json", version.ref = "kotlinx-serialization" }

# ──────────────────────────────────────────────
# Coroutines
# ──────────────────────────────────────────────
kotlinx-coroutines-core = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-core", version.ref = "kotlinx-coroutines" }
kotlinx-coroutines-android = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-android", version.ref = "kotlinx-coroutines" }
kotlinx-coroutines-test = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-test", version.ref = "kotlinx-coroutines" }

# ──────────────────────────────────────────────
# Images
# ──────────────────────────────────────────────
coil-compose = { group = "io.coil-kt", name = "coil-compose", version.ref = "coil" }

# ──────────────────────────────────────────────
# DataStore
# ──────────────────────────────────────────────
datastore-preferences = { group = "androidx.datastore", name = "datastore-preferences", version.ref = "datastore" }

# ──────────────────────────────────────────────
# Testing
# ──────────────────────────────────────────────
junit5-api = { group = "org.junit.jupiter", name = "junit-jupiter-api", version.ref = "junit5" }
junit5-engine = { group = "org.junit.jupiter", name = "junit-jupiter-engine", version.ref = "junit5" }
junit5-params = { group = "org.junit.jupiter", name = "junit-jupiter-params", version.ref = "junit5" }
mockk = { group = "io.mockk", name = "mockk", version.ref = "mockk" }
mockk-android = { group = "io.mockk", name = "mockk-android", version.ref = "mockk" }
turbine = { group = "app.cash.turbine", name = "turbine", version.ref = "turbine" }

[bundles]
compose = [
    "compose-ui",
    "compose-ui-graphics",
    "compose-material3",
    "compose-ui-tooling-preview",
    "compose-material-icons",
]
compose-debug = [
    "compose-ui-tooling",
    "compose-ui-test-manifest",
]
lifecycle = [
    "lifecycle-runtime-compose",
    "lifecycle-viewmodel-compose",
]
network = [
    "retrofit",
    "retrofit-kotlinx-serialization",
    "okhttp",
    "okhttp-logging",
    "kotlinx-serialization-json",
]
room = [
    "room-runtime",
    "room-ktx",
]
testing = [
    "junit5-api",
    "junit5-params",
    "mockk",
    "turbine",
    "kotlinx-coroutines-test",
]

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
android-library = { id = "com.android.library", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
room = { id = "androidx.room", version.ref = "room" }
```

### Using Bundles

**Bundles group related dependencies together so modules can add them in a single line.**

```kotlin
// ✅ Using bundles - clean and consistent
dependencies {
    implementation(libs.bundles.compose)
    implementation(libs.bundles.lifecycle)
    implementation(libs.bundles.network)
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
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.serialization) apply false
    alias(libs.plugins.compose.compiler) apply false
    alias(libs.plugins.hilt) apply false
    alias(libs.plugins.ksp) apply false
    alias(libs.plugins.room) apply false
}
```

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
    id("com.google.dagger.hilt.android") version "2.51.1"  // Duplicated everywhere
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
include(":core:network")
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

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
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

        vectorDrawables {
            useSupportLibrary = true
        }
    }

    buildTypes {
        debug {
            isDebuggable = true
            isMinifyEnabled = false
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-debug"
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

    kotlinOptions {
        jvmTarget = "17"
    }

    buildFeatures {
        compose = true
        buildConfig = true
    }

    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
            excludes += "/META-INF/LICENSE*"
        }
    }
}

dependencies {
    // Feature modules
    implementation(project(":feature:home"))
    implementation(project(":feature:search"))
    implementation(project(":feature:profile"))

    // Core modules
    implementation(project(":core:network"))
    implementation(project(":core:database"))
    implementation(project(":core:domain"))
    implementation(project(":core:model"))
    implementation(project(":core:ui"))

    // AndroidX Core
    implementation(libs.core.ktx)
    implementation(libs.activity.compose)

    // Compose
    implementation(platform(libs.compose.bom))
    implementation(libs.bundles.compose)
    debugImplementation(libs.bundles.compose.debug)

    // Lifecycle
    implementation(libs.bundles.lifecycle)

    // Navigation
    implementation(libs.navigation.compose)

    // Hilt
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)
    implementation(libs.hilt.navigation.compose)

    // Testing
    testImplementation(libs.bundles.testing)
    testRuntimeOnly(libs.junit5.engine)
    androidTestImplementation(platform(libs.compose.bom))
    androidTestImplementation(libs.compose.ui.test.junit4)
}
```

---

## 📐 Library Module build.gradle.kts

**Feature and core modules use the `android.library` plugin. They expose only what they need and keep dependencies minimal.**

### Feature Module Example (feature/home)

```kotlin
plugins {
    alias(libs.plugins.android.library)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.compose.compiler)
    alias(libs.plugins.hilt)
    alias(libs.plugins.ksp)
}

android {
    namespace = "com.arclabs.myapp.feature.home"
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

    kotlinOptions {
        jvmTarget = "17"
    }

    buildFeatures {
        compose = true
    }
}

dependencies {
    // Core modules
    implementation(project(":core:domain"))
    implementation(project(":core:model"))
    implementation(project(":core:ui"))

    // Compose
    implementation(platform(libs.compose.bom))
    implementation(libs.bundles.compose)
    debugImplementation(libs.bundles.compose.debug)

    // Lifecycle
    implementation(libs.bundles.lifecycle)

    // Navigation
    implementation(libs.navigation.compose)

    // Hilt
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)
    implementation(libs.hilt.navigation.compose)

    // Images
    implementation(libs.coil.compose)

    // Testing
    testImplementation(libs.bundles.testing)
    testRuntimeOnly(libs.junit5.engine)
    testImplementation(project(":core:testing"))
}
```

### Core Module Example (core/network)

```kotlin
plugins {
    alias(libs.plugins.android.library)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.serialization)
    alias(libs.plugins.hilt)
    alias(libs.plugins.ksp)
}

android {
    namespace = "com.arclabs.myapp.core.network"
    compileSdk = libs.versions.compileSdk.get().toInt()

    defaultConfig {
        minSdk = libs.versions.minSdk.get().toInt()
        consumerProguardFiles("consumer-rules.pro")
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
    }
}

dependencies {
    // Core modules
    implementation(project(":core:model"))

    // Network
    implementation(libs.bundles.network)

    // Coroutines
    implementation(libs.kotlinx.coroutines.core)
    implementation(libs.kotlinx.coroutines.android)

    // Hilt
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)

    // Testing
    testImplementation(libs.bundles.testing)
    testRuntimeOnly(libs.junit5.engine)
}
```

### Pure Kotlin Module Example (core/domain)

```kotlin
plugins {
    id("java-library")
    alias(libs.plugins.kotlin.android)
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

dependencies {
    // Core modules
    implementation(project(":core:model"))

    // Coroutines
    implementation(libs.kotlinx.coroutines.core)

    // Hilt (compile-only for annotations)
    compileOnly(libs.hilt.android)

    // Testing
    testImplementation(libs.bundles.testing)
    testRuntimeOnly(libs.junit5.engine)
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
# Room
# ──────────────────────────────────────────────
-keep class * extends androidx.room.RoomDatabase
-keep @androidx.room.Entity class *
-dontwarn androidx.room.paging.**

# ──────────────────────────────────────────────
# Retrofit / OkHttp
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
android.useAndroidX=true
android.nonTransitiveRClass=true
android.defaults.buildfeatures.buildconfig=false
```

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
| `./gradlew jacocoTestReport` | Generate test coverage report |

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
./gradlew :core:network:lint
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
        force("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.9.0")
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
    implementation(project(":core:network")) // Missing dependency?
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
kotlin = "2.0.21"

[plugins]
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
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
- [ ] `compileOptions` and `kotlinOptions` set to Java 17
- [ ] Dependencies use version catalog references (`libs.xxx`)
- [ ] Test dependencies include JUnit5, MockK, Turbine
- [ ] Consumer ProGuard rules provided for library modules

### Build Variants

- [ ] Debug variant has `applicationIdSuffix = ".debug"`
- [ ] Release variant has `isMinifyEnabled = true` and `isShrinkResources = true`
- [ ] Staging variant configured (if applicable)
- [ ] `BuildConfig` fields for environment-specific values
- [ ] ProGuard rules cover serialization, Hilt, Room, Retrofit

### Code Quality

- [ ] ktlint configured for Kotlin style checking
- [ ] detekt configured for static analysis
- [ ] Android Lint enabled and baseline established
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
- [ARCDevTools](https://github.com/arclabs-studio/ARCDevTools) - Development tools and configurations
