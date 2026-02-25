# 📦 Android Libraries

**Standards and guidelines for creating, structuring, publishing, and maintaining reusable Android libraries at ARC Labs Studio.**

---

## 📖 Table of Contents

1. [Library Philosophy](#-library-philosophy)
2. [Library Types](#-library-types)
3. [Standard Library Structure](#-standard-library-structure)
4. [build.gradle.kts Template](#-buildgradlekts-template)
5. [Publishing Setup](#-publishing-setup)
6. [API Design Principles](#-api-design-principles)
7. [Naming Conventions](#-naming-conventions)
8. [Versioning](#-versioning)
9. [Testing Requirements](#-testing-requirements)
10. [Documentation Requirements](#-documentation-requirements)
11. [Sample / Demo App Pattern](#-sample--demo-app-pattern)
12. [Dependency Management](#-dependency-management)
13. [ProGuard / R8 Consumer Rules](#-proguard--r8-consumer-rules)
14. [Checklist for New Libraries](#-checklist-for-new-libraries)
15. [Common Mistakes](#-common-mistakes)
16. [Further Reading](#-further-reading)

---

## 🧭 Library Philosophy

At ARC Labs Studio, libraries are first-class citizens. Every library we ship must be:

- **Reusable** - designed for consumption across multiple apps and teams.
- **Well-tested** - 100% code coverage is the baseline, not the goal.
- **Documented** - KDoc on every public symbol, a complete README, and a maintained CHANGELOG.
- **Minimal** - expose only what consumers need; hide everything else.
- **Stable** - follow Semantic Versioning so consumers can upgrade with confidence.

> A library that is not tested is not a library. It is a liability.

### Guiding Principles

1. **Simple, Lovable, Complete** - The API should be easy to discover, pleasant to use, and cover the full use case without forcing workarounds.
2. **Quality Over Speed** - Never rush a library release. A broken library breaks every app that depends on it.
3. **Modular by Design** - Keep libraries focused on a single responsibility. Prefer many small libraries over one large monolith.
4. **Native First** - Prefer Jetpack Compose over legacy View-based UI. Prefer Kotlin coroutines over RxJava. Prefer modern Jetpack APIs over deprecated alternatives.

### What Belongs in a Library

| Belongs | Does NOT Belong |
|---------|-----------------|
| Generic UI components | App-specific screens |
| Network abstractions | API endpoint definitions |
| Database utilities | App-specific schemas |
| Logging infrastructure | Business rules |
| Design system tokens | Feature logic |
| Common extensions | App navigation |
| Testing utilities | App configuration |

---

## 🗂 Library Types

ARC Labs recognizes four primary library categories. Every new library must identify which category it belongs to before development begins.

### 1. UI Component Libraries

Provide reusable Compose components, themes, and design tokens.

```kotlin
// Example: arc-design module
@Composable
fun ArcButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    style: ArcButtonStyle = ArcButtonStyle.Primary,
    enabled: Boolean = true,
    isLoading: Boolean = false,
) {
    Button(
        onClick = onClick,
        modifier = modifier,
        enabled = enabled && !isLoading,
        colors = style.toButtonColors(),
        shape = ArcTheme.shapes.medium,
    ) {
        if (isLoading) {
            CircularProgressIndicator(
                modifier = Modifier.size(18.dp),
                strokeWidth = 2.dp,
                color = ArcTheme.colorScheme.onPrimary,
            )
        } else {
            Text(
                text = text,
                style = ArcTheme.typography.labelLarge,
            )
        }
    }
}
```

**Characteristics:**
- Depend on Jetpack Compose and Material Design 3.
- Ship Compose previews for every component.
- Include a demo/catalog app showing every variant.

### 2. Utility Libraries

Provide pure Kotlin or Android utility functions with zero UI dependencies.

```kotlin
// Example: arc-core module
object DateUtils {

    fun Instant.toRelativeString(clock: Clock = Clock.System): String {
        val duration = clock.now() - this
        return when {
            duration < 1.minutes -> "Just now"
            duration < 1.hours -> "${duration.inWholeMinutes}m ago"
            duration < 1.days -> "${duration.inWholeHours}h ago"
            else -> "${duration.inWholeDays}d ago"
        }
    }
}
```

**Characteristics:**
- Minimal dependencies (ideally only `kotlin-stdlib` and `kotlinx` libraries).
- Pure functions preferred; no hidden state.
- Multiplatform-friendly when possible.

### 3. Data Libraries

Encapsulate data access: networking, persistence, caching.

```kotlin
// Example: arc-network module
interface ApiClient {
    suspend fun <T : Any> get(
        endpoint: String,
        responseType: KClass<T>,
        headers: Map<String, String> = emptyMap(),
    ): Result<T>

    suspend fun <T : Any> post(
        endpoint: String,
        body: Any,
        responseType: KClass<T>,
        headers: Map<String, String> = emptyMap(),
    ): Result<T>
}
```

**Characteristics:**
- Expose protocol/interface abstractions, not concrete implementations.
- Provide default implementations behind factory functions.
- Never leak OkHttp, Retrofit, or Room types to consumers.

### 4. Domain Libraries

Contain shared business logic, entities, and use-case definitions.

```kotlin
// Example: arc-domain module
data class Restaurant(
    val id: String,
    val name: String,
    val cuisine: CuisineType,
    val rating: Double,
    val location: LatLng,
)

interface SearchRestaurantsUseCase {
    suspend operator fun invoke(query: SearchQuery): Result<List<Restaurant>>
}
```

**Characteristics:**
- Zero Android framework dependencies (pure Kotlin).
- Define repository interfaces; never implement them.
- Entities are immutable data classes.

### Current ARC Labs Libraries

| Library | Type | Purpose | Status |
|---------|------|---------|--------|
| ARCDesignSystem | UI Component | Material3 theme, shared composables, design tokens | Planned |
| ARCDevTools-Android | Utility | Tooling, linting (ktlint/detekt), CI/CD config | Planned |
| ARCLogger-Android | Utility | Structured logging with tag-based filtering | Planned |
| ARCNetworking-Android | Data | Retrofit wrappers, interceptors, error handling | Planned |
| ARCStorage-Android | Data | Room wrappers, DataStore utilities, cache management | Planned |
| ARCTesting-Android | Utility | Test utilities, fakes, assertion helpers | Planned |

### Library Dependency Graph

```
ARCDesignSystem
    └── (no ARC dependencies, only Compose)

ARCLogger-Android
    └── (no ARC dependencies, standalone)

ARCNetworking-Android
    └── ARCLogger-Android

ARCStorage-Android
    └── ARCLogger-Android

ARCTesting-Android
    └── ARCLogger-Android

ARCDevTools-Android
    └── (build tool, no runtime dependency)
```

---

## 🏗 Standard Library Structure

Every ARC Labs Android library follows a consistent Gradle module layout.

### Simple Library (Single Module)

```
arc-feature/
├── build.gradle.kts
├── consumer-proguard-rules.pro
├── src/
│   ├── main/
│   │   ├── AndroidManifest.xml
│   │   ├── kotlin/
│   │   │   └── com/arclabs/feature/
│   │   │       ├── FeatureApi.kt
│   │   │       ├── internal/
│   │   │       │   ├── FeatureImpl.kt
│   │   │       │   └── FeatureMapper.kt
│   │   │       ├── model/
│   │   │       │   ├── FeatureEntity.kt
│   │   │       │   └── FeatureConfig.kt
│   │   │       └── di/
│   │   │           └── FeatureModule.kt
│   │   └── res/
│   │       └── values/
│   │           └── strings.xml
│   ├── test/
│   │   └── kotlin/
│   │       └── com/arclabs/feature/
│   │           ├── FeatureApiTest.kt
│   │           ├── internal/
│   │           │   ├── FeatureImplTest.kt
│   │           │   └── FeatureMapperTest.kt
│   │           └── fixtures/
│   │               └── FeatureFixtures.kt
│   └── androidTest/
│       └── kotlin/
│           └── com/arclabs/feature/
│               └── FeatureInstrumentedTest.kt
├── README.md
└── CHANGELOG.md
```

### Multi-Module Library

For larger libraries that benefit from separation of concerns:

```
arc-feature/
├── arc-feature-core/
│   ├── src/
│   │   ├── main/kotlin/com/arclabs/feature/core/
│   │   └── test/kotlin/com/arclabs/feature/core/
│   └── build.gradle.kts
├── arc-feature-compose/
│   ├── src/
│   │   ├── main/kotlin/com/arclabs/feature/compose/
│   │   └── test/kotlin/com/arclabs/feature/compose/
│   └── build.gradle.kts
├── arc-feature-testing/
│   ├── src/
│   │   ├── main/kotlin/com/arclabs/feature/testing/
│   │   └── test/kotlin/com/arclabs/feature/testing/
│   └── build.gradle.kts
├── demo/
│   ├── src/
│   │   └── main/kotlin/com/arclabs/feature/demo/
│   └── build.gradle.kts
├── build.gradle.kts
├── settings.gradle.kts
├── gradle/
│   └── libs.versions.toml
├── README.md
└── CHANGELOG.md
```

### Module Naming Conventions

| Module | Purpose | Artifact ID |
|--------|---------|-------------|
| `arc-feature-core` | Core functionality, no Android UI | `arc-feature-core` |
| `arc-feature-compose` | Compose UI components | `arc-feature-compose` |
| `arc-feature-testing` | Test utilities and fakes | `arc-feature-testing` |
| `demo` | Demo app demonstrating usage | (not published) |

### Key Directory Conventions

| Directory | Purpose |
|---|---|
| `src/main/kotlin/.../internal/` | Implementation details hidden from consumers via `internal` visibility. |
| `src/main/kotlin/.../model/` | Public data classes and sealed types. |
| `src/main/kotlin/.../di/` | Hilt modules for dependency injection. |
| `src/test/` | Unit tests (JUnit 5 + MockK). Runs on JVM. |
| `src/androidTest/` | Instrumented tests. Use sparingly; prefer JVM tests. |
| `src/test/.../fixtures/` | Shared test data factories and builders. |
| `consumer-proguard-rules.pro` | ProGuard rules shipped to consumers. |

### Visibility Rules

```kotlin
// ✅ Correct: Public API is explicit and minimal
public class FeatureApi(
    private val impl: FeatureImpl,
) {
    public fun doSomething(): Result<String> = impl.execute()
}

// ✅ Correct: Implementation details are internal
internal class FeatureImpl(
    private val repository: FeatureRepository,
) {
    fun execute(): Result<String> {
        // ...
    }
}
```

```kotlin
// ❌ Incorrect: Leaking implementation to consumers
public class FeatureImpl(  // Should be internal!
    public val repository: FeatureRepository,  // Should be private!
) {
    public fun execute(): Result<String> { /* ... */ }
}
```

---

## 📝 build.gradle.kts Template

Every library module uses Gradle Kotlin DSL with version catalogs.

### Library Module

```kotlin
plugins {
    alias(libs.plugins.android.library)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    alias(libs.plugins.hilt.android)  // Only if DI is needed
    alias(libs.plugins.ksp)           // Only if annotation processing is needed
    `maven-publish`
}

android {
    namespace = "com.arclabs.feature"
    compileSdk = libs.versions.compileSdk.get().toInt()

    defaultConfig {
        minSdk = libs.versions.minSdk.get().toInt()

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        consumerProguardFiles("consumer-proguard-rules.pro")
    }

    buildTypes {
        release {
            isMinifyEnabled = false
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
        freeCompilerArgs += listOf(
            "-opt-in=kotlinx.coroutines.ExperimentalCoroutinesApi",
        )
    }

    buildFeatures {
        compose = true  // Only for UI libraries
    }

    publishing {
        singleVariant("release") {
            withSourcesJar()
            withJavadocJar()
        }
    }

    testOptions {
        unitTests.all {
            it.useJUnitPlatform()
        }
    }
}

kotlin {
    explicitApi()
}

dependencies {
    // Core
    implementation(libs.androidx.core.ktx)
    implementation(libs.kotlinx.coroutines.android)

    // Compose (UI libraries only)
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.compose.ui)
    implementation(libs.androidx.compose.material3)
    implementation(libs.androidx.compose.ui.tooling.preview)
    debugImplementation(libs.androidx.compose.ui.tooling)

    // Hilt (only if DI is needed)
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)

    // Testing
    testImplementation(platform(libs.junit.bom))
    testImplementation(libs.junit.jupiter.api)
    testImplementation(libs.junit.jupiter.params)
    testRuntimeOnly(libs.junit.jupiter.engine)
    testImplementation(libs.mockk)
    testImplementation(libs.kotlinx.coroutines.test)
    testImplementation(libs.turbine)
    testImplementation(libs.truth)
}
```

### Version Catalog (libs.versions.toml)

Libraries reference a shared version catalog at the project root.

```toml
[versions]
compileSdk = "35"
minSdk = "26"
targetSdk = "35"
kotlin = "2.1.0"
agp = "8.7.3"
compose-bom = "2025.01.00"
coroutines = "1.9.0"
hilt = "2.53.1"
junit = "5.11.4"
mockk = "1.13.14"
turbine = "1.2.0"
truth = "1.4.4"
ktlint = "12.1.2"
detekt = "1.23.7"
dokka = "1.9.20"

[libraries]
androidx-core-ktx = { module = "androidx.core:core-ktx", version = "1.15.0" }
kotlinx-coroutines-android = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-android", version.ref = "coroutines" }
kotlinx-coroutines-test = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-test", version.ref = "coroutines" }

# Compose
androidx-compose-bom = { module = "androidx.compose:compose-bom", version.ref = "compose-bom" }
androidx-compose-ui = { module = "androidx.compose.ui:ui" }
androidx-compose-material3 = { module = "androidx.compose.material3:material3" }
androidx-compose-ui-tooling = { module = "androidx.compose.ui:ui-tooling" }
androidx-compose-ui-tooling-preview = { module = "androidx.compose.ui:ui-tooling-preview" }
androidx-compose-ui-test = { module = "androidx.compose.ui:ui-test-junit4" }

# Hilt
hilt-android = { module = "com.google.dagger:hilt-android", version.ref = "hilt" }
hilt-compiler = { module = "com.google.dagger:hilt-compiler", version.ref = "hilt" }

# Testing
junit-bom = { module = "org.junit:junit-bom", version.ref = "junit" }
junit-jupiter-api = { module = "org.junit.jupiter:junit-jupiter-api" }
junit-jupiter-params = { module = "org.junit.jupiter:junit-jupiter-params" }
junit-jupiter-engine = { module = "org.junit.jupiter:junit-jupiter-engine" }
mockk = { module = "io.mockk:mockk", version.ref = "mockk" }
turbine = { module = "app.cash.turbine:turbine", version.ref = "turbine" }
truth = { module = "com.google.truth:truth", version.ref = "truth" }

[plugins]
android-library = { id = "com.android.library", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
hilt-android = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
ksp = { id = "com.google.devtools.ksp", version = "2.1.0-1.0.29" }
ktlint = { id = "org.jlleitschuh.gradle.ktlint", version.ref = "ktlint" }
detekt = { id = "io.gitlab.arturbosch.detekt", version.ref = "detekt" }
dokka = { id = "org.jetbrains.dokka", version.ref = "dokka" }
```

### Publishing Configuration

```kotlin
publishing {
    publications {
        register<MavenPublication>("release") {
            groupId = "com.arclabs"
            artifactId = "arc-feature"
            version = project.findProperty("VERSION_NAME") as? String ?: "0.0.1-SNAPSHOT"

            afterEvaluate {
                from(components["release"])
            }

            pom {
                name.set("ARC Feature")
                description.set("Feature library for ARC Labs apps.")
                url.set("https://github.com/ARC-Labs-Studio/arc-feature-android")

                licenses {
                    license {
                        name.set("Proprietary")
                    }
                }

                developers {
                    developer {
                        id.set("arclabs")
                        name.set("ARC Labs Studio")
                        email.set("dev@arclabs.com")
                    }
                }
            }
        }
    }

    repositories {
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/ARC-Labs-Studio/arc-feature-android")
            credentials {
                username = System.getenv("GITHUB_ACTOR")
                    ?: project.findProperty("gpr.user") as? String
                password = System.getenv("GITHUB_TOKEN")
                    ?: project.findProperty("gpr.token") as? String
            }
        }
    }
}
```

---

## 🚀 Publishing Setup

ARC Labs libraries are published to Maven Local for local development and GitHub Packages for team-wide distribution.

### Maven Local (Development)

```bash
# Publish to ~/.m2/repository
./gradlew :arc-feature:publishToMavenLocal
```

Consumers reference it with:

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        mavenLocal()
        google()
        mavenCentral()
    }
}
```

### GitHub Packages (Production)

Add the publishing configuration from the build.gradle.kts template above. The GitHub Packages repository block handles authentication via environment variables or `gradle.properties`.

### Consuming Published Libraries

```kotlin
// settings.gradle.kts of the consuming app
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/ARC-Labs-Studio/*")
            credentials {
                username = providers.gradleProperty("gpr.user").orNull
                    ?: System.getenv("GITHUB_ACTOR")
                password = providers.gradleProperty("gpr.token").orNull
                    ?: System.getenv("GITHUB_TOKEN")
            }
        }
    }
}

// build.gradle.kts of the consuming module
dependencies {
    implementation("com.arclabs:arc-feature:1.2.0")
}
```

### Publishing CI Workflow

```yaml
# .github/workflows/publish.yml
name: Publish Library

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read

    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4

      - name: Run Tests
        run: ./gradlew test

      - name: Publish to GitHub Packages
        run: ./gradlew publish -PVERSION_NAME=${{ github.event.release.tag_name }}
        env:
          GITHUB_ACTOR: ${{ github.actor }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 🎯 API Design Principles

### 1. Minimal Surface Area

Expose only what consumers need. Everything else is `internal` or `private`.

```kotlin
// ✅ Correct: Small, focused public API
public interface ImageLoader {
    suspend fun load(url: String): Result<Bitmap>
    suspend fun preload(urls: List<String>)
    fun clearCache()
}
```

```kotlin
// ❌ Incorrect: Leaking internals to the public API
public interface ImageLoader {
    suspend fun load(url: String): Result<Bitmap>
    suspend fun preload(urls: List<String>)
    fun clearCache()
    fun getDiskCacheSize(): Long        // Consumer doesn't need this
    fun getMemoryCacheHitRate(): Double  // Consumer doesn't need this
    fun setOkHttpClient(client: OkHttpClient) // Leaking implementation detail
}
```

### 2. Extension-Friendly

Design APIs so consumers can extend behavior without modifying library code.

```kotlin
// ✅ Correct: Interface-based, consumers can provide their own implementation
public interface Logger {
    fun log(level: LogLevel, tag: String, message: String, throwable: Throwable? = null)
}

public enum class LogLevel { DEBUG, INFO, WARN, ERROR }

// Library ships a default
public class AndroidLogger : Logger {
    override fun log(level: LogLevel, tag: String, message: String, throwable: Throwable?) {
        when (level) {
            LogLevel.DEBUG -> Log.d(tag, message, throwable)
            LogLevel.INFO -> Log.i(tag, message, throwable)
            LogLevel.WARN -> Log.w(tag, message, throwable)
            LogLevel.ERROR -> Log.e(tag, message, throwable)
        }
    }
}

// Consumer can plug in their own
class CrashlyticsLogger : Logger {
    override fun log(level: LogLevel, tag: String, message: String, throwable: Throwable?) {
        if (level == LogLevel.ERROR) {
            Firebase.crashlytics.recordException(throwable ?: RuntimeException(message))
        }
    }
}
```

### 3. Backward Compatible

Never break existing consumers when releasing minor or patch versions.

```kotlin
// ✅ Correct: Adding optional parameters preserves binary compatibility
// v1.0.0
public fun fetchUser(id: String): Result<User>

// v1.1.0 - backward compatible addition
public fun fetchUser(
    id: String,
    includeProfile: Boolean = false, // New, optional parameter with default
): Result<User>
```

```kotlin
// ❌ Incorrect: Changing existing parameter types breaks consumers
// v1.0.0
public fun fetchUser(id: String): Result<User>

// v1.1.0 - BREAKING CHANGE
public fun fetchUser(id: Long): Result<User> // Changed String to Long!
```

### 4. Kotlin-Idiomatic

Use Kotlin language features to create expressive, safe APIs.

```kotlin
// ✅ Correct: Builder DSL for complex configuration
public class ArcClient private constructor(
    public val baseUrl: String,
    public val timeout: Duration,
    public val logger: Logger?,
) {
    public class Builder {
        private var baseUrl: String = ""
        private var timeout: Duration = 30.seconds
        private var logger: Logger? = null

        public fun baseUrl(url: String): Builder = apply { baseUrl = url }
        public fun timeout(duration: Duration): Builder = apply { timeout = duration }
        public fun logger(logger: Logger): Builder = apply { this.logger = logger }

        public fun build(): ArcClient {
            require(baseUrl.isNotBlank()) { "baseUrl must not be blank" }
            return ArcClient(baseUrl, timeout, logger)
        }
    }
}

// DSL extension
public fun arcClient(block: ArcClient.Builder.() -> Unit): ArcClient =
    ArcClient.Builder().apply(block).build()

// Usage
val client = arcClient {
    baseUrl("https://api.arclabs.com")
    timeout(15.seconds)
    logger(AndroidLogger())
}
```

### 5. Result-Based Error Handling

Never throw exceptions across library boundaries. Use `Result<T>` or sealed classes.

```kotlin
// ✅ Correct: Explicit error modeling
public sealed interface NetworkError {
    data class HttpError(val code: Int, val body: String?) : NetworkError
    data class ConnectionError(val cause: Throwable) : NetworkError
    data object Timeout : NetworkError
    data class Unknown(val cause: Throwable) : NetworkError
}

public suspend fun fetchData(): Result<Data>
```

```kotlin
// ❌ Incorrect: Throwing exceptions the consumer may not expect
public suspend fun fetchData(): Data // Throws IOException, HttpException, etc.
```

### 6. Composable API Design

Follow Jetpack Compose conventions for UI component libraries.

```kotlin
// ✅ Correct: Follows Compose conventions
// 1. Composable functions are PascalCase
// 2. First meaningful parameter is the primary content
// 3. Modifier is always a parameter with default
// 4. Content lambda is last parameter

@Composable
public fun ArcCard(
    modifier: Modifier = Modifier,
    elevation: Dp = 2.dp,
    shape: Shape = MaterialTheme.shapes.medium,
    colors: CardColors = CardDefaults.cardColors(),
    content: @Composable ColumnScope.() -> Unit,
) {
    Card(
        modifier = modifier,
        elevation = CardDefaults.cardElevation(defaultElevation = elevation),
        shape = shape,
        colors = colors,
        content = content,
    )
}
```

### 7. Extension Functions for Discoverability

```kotlin
// Core function on the interface
public interface Logger {
    fun log(level: LogLevel, tag: String, message: String)
}

// Convenient extensions
public fun Logger.debug(tag: String, message: String) = log(LogLevel.DEBUG, tag, message)
public fun Logger.error(tag: String, message: String) = log(LogLevel.ERROR, tag, message)
public fun Logger.debug(tag: String, lazyMessage: () -> String) {
    log(LogLevel.DEBUG, tag, lazyMessage())
}
```

---

## 📛 Naming Conventions

Consistent naming across all ARC Labs libraries eliminates ambiguity and improves discoverability.

### Module Names

| Convention | Example |
|---|---|
| Prefix with `arc-` | `arc-design`, `arc-network`, `arc-analytics` |
| Lowercase with hyphens | `arc-feature-flags` (not `arcFeatureFlags`) |
| Singular nouns preferred | `arc-image` (not `arc-images`) |

### Package Names

```
com.arclabs.<module>
com.arclabs.<module>.internal
com.arclabs.<module>.model
com.arclabs.<module>.di
```

Examples:
- `com.arclabs.network`
- `com.arclabs.network.internal`
- `com.arclabs.design.component`

### Artifact Names (Maven Coordinates)

```
com.arclabs:arc-<name>:<version>
```

Examples:
- `com.arclabs:arc-design:2.1.0`
- `com.arclabs:arc-network:1.5.3`
- `com.arclabs:arc-analytics:0.8.0`

### Class / Interface Names

| Type | Pattern | Example |
|---|---|---|
| Public API entry point | `Arc<Feature>` | `ArcImageLoader`, `ArcClient` |
| Interface | Descriptive name | `ImageLoader`, `AnalyticsTracker` |
| Implementation | `Default<Feature>` or `<Feature>Impl` | `DefaultImageLoader` |
| Hilt Module | `<Feature>Module` | `NetworkModule` |
| Compose Component | `Arc<Component>` | `ArcButton`, `ArcTextField` |
| Test Class | `<ClassName>Test` | `DefaultImageLoaderTest` |
| Test Fixtures | `<Feature>Fixtures` | `RestaurantFixtures` |

---

## 🔢 Versioning

All ARC Labs libraries follow **Semantic Versioning 2.0.0** (`MAJOR.MINOR.PATCH`).

### Version Meaning

| Component | When to increment | Example |
|---|---|---|
| **MAJOR** | Breaking API changes | `1.x.x` -> `2.0.0` |
| **MINOR** | New features, backward compatible | `1.2.x` -> `1.3.0` |
| **PATCH** | Bug fixes, backward compatible | `1.2.3` -> `1.2.4` |

### Pre-Release Versions

```
0.x.x          - Initial development, API may change at any time
1.0.0-alpha.1  - Feature-complete alpha, API still unstable
1.0.0-beta.1   - API frozen, bug fixes only
1.0.0-rc.1     - Release candidate, production-ready unless critical bugs found
1.0.0          - Stable release
```

### Version Rules

1. **NEVER** release a `1.0.0` without at least one consumer app validating the API.
2. **NEVER** make breaking changes in MINOR or PATCH releases.
3. **ALWAYS** tag releases in Git: `git tag v1.2.3`.
4. **ALWAYS** update `CHANGELOG.md` before every release.
5. **ALWAYS** set the version in `gradle.properties`:

```properties
# gradle.properties
VERSION_NAME=1.2.3
VERSION_CODE=10203
```

### What Constitutes a Breaking Change

| Breaking | Not Breaking |
|----------|-------------|
| Remove public function | Add new public function |
| Change function signature | Add optional parameter with default |
| Change return type | Add new class |
| Remove public class | Deprecate (with replacement) |
| Change behavior silently | Fix a documented bug |
| Rename public API | Add internal functions |

### Deprecation Policy

```kotlin
// Step 1: Deprecate with a replacement (MINOR release)
@Deprecated(
    message = "Use fetchUserById() instead.",
    replaceWith = ReplaceWith("fetchUserById(id)"),
    level = DeprecationLevel.WARNING,
)
public fun getUser(id: String): Result<User> = fetchUserById(id)

public fun fetchUserById(id: String): Result<User> { /* ... */ }

// Step 2: Escalate to ERROR (next MINOR release)
@Deprecated(
    message = "Use fetchUserById() instead.",
    replaceWith = ReplaceWith("fetchUserById(id)"),
    level = DeprecationLevel.ERROR,
)
public fun getUser(id: String): Result<User> = fetchUserById(id)

// Step 3: Remove in the next MAJOR release
```

---

## 🧪 Testing Requirements

Libraries have the strictest testing requirements at ARC Labs. If it ships as a library, it must be tested exhaustively.

### Coverage Targets

| Metric | Target |
|---|---|
| Line coverage | **100%** |
| Branch coverage | **95%+** |
| Public API | Every public function and class tested |
| Edge cases | Empty inputs, null handling, boundary values |
| Error handling | All error paths tested |

### Test Stack

| Tool | Purpose |
|---|---|
| JUnit 5 | Test framework and runner |
| MockK | Mocking and stubbing |
| Turbine | Testing Kotlin Flows |
| Truth | Fluent assertions |
| Coroutines Test | `runTest`, `TestDispatcher`, `advanceUntilIdle` |
| Compose UI Test | Compose component testing |
| Robolectric | Unit tests that need Android framework |

### Test Structure (Given-When-Then)

Every test follows the Given-When-Then pattern with a `makeSUT()` factory.

```kotlin
import com.google.common.truth.Truth.assertThat
import io.mockk.coEvery
import io.mockk.mockk
import kotlinx.coroutines.test.runTest
import org.junit.jupiter.api.DisplayName
import org.junit.jupiter.api.Nested
import org.junit.jupiter.api.Tag
import org.junit.jupiter.api.Test

@Tag("unit")
@DisplayName("DefaultImageLoader")
class DefaultImageLoaderTest {

    private val mockCache: ImageCache = mockk(relaxed = true)
    private val mockFetcher: ImageFetcher = mockk()

    private fun makeSUT(): DefaultImageLoader {
        return DefaultImageLoader(
            cache = mockCache,
            fetcher = mockFetcher,
        )
    }

    @Nested
    @DisplayName("load()")
    inner class Load {

        @Test
        @DisplayName("should return cached image when available")
        fun returnsCachedImage() = runTest {
            // Given
            val sut = makeSUT()
            val expectedBitmap = createTestBitmap()
            coEvery { mockCache.get("https://example.com/img.png") } returns expectedBitmap

            // When
            val result = sut.load("https://example.com/img.png")

            // Then
            assertThat(result.isSuccess).isTrue()
            assertThat(result.getOrNull()).isEqualTo(expectedBitmap)
        }

        @Test
        @DisplayName("should fetch from network when cache misses")
        fun fetchesFromNetworkOnCacheMiss() = runTest {
            // Given
            val sut = makeSUT()
            val expectedBitmap = createTestBitmap()
            coEvery { mockCache.get(any()) } returns null
            coEvery { mockFetcher.fetch("https://example.com/img.png") } returns expectedBitmap

            // When
            val result = sut.load("https://example.com/img.png")

            // Then
            assertThat(result.isSuccess).isTrue()
            assertThat(result.getOrNull()).isEqualTo(expectedBitmap)
        }

        @Test
        @DisplayName("should return failure when both cache and network fail")
        fun returnsFailureWhenAllSourcesFail() = runTest {
            // Given
            val sut = makeSUT()
            coEvery { mockCache.get(any()) } returns null
            coEvery { mockFetcher.fetch(any()) } throws IOException("Network error")

            // When
            val result = sut.load("https://example.com/img.png")

            // Then
            assertThat(result.isFailure).isTrue()
        }
    }
}
```

### Testing Flows with Turbine

```kotlin
@Test
@DisplayName("should emit loading then success states")
fun emitsCorrectStates() = runTest {
    // Given
    val sut = makeSUT()

    // When / Then
    sut.state.test {
        assertThat(awaitItem()).isEqualTo(State.Idle)

        sut.loadData()

        assertThat(awaitItem()).isEqualTo(State.Loading)
        assertThat(awaitItem()).isInstanceOf(State.Success::class.java)

        cancelAndConsumeRemainingEvents()
    }
}
```

### Testing Compose Components

```kotlin
class ArcButtonTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun `displays text correctly`() {
        // Given
        composeTestRule.setContent {
            ArcButton(
                text = "Save",
                onClick = {},
            )
        }

        // Then
        composeTestRule
            .onNodeWithText("Save")
            .assertIsDisplayed()
    }

    @Test
    fun `invokes onClick when tapped`() {
        // Given
        var clicked = false
        composeTestRule.setContent {
            ArcButton(
                text = "Save",
                onClick = { clicked = true },
            )
        }

        // When
        composeTestRule
            .onNodeWithText("Save")
            .performClick()

        // Then
        assertThat(clicked).isTrue()
    }

    @Test
    fun `shows loading indicator when isLoading is true`() {
        // Given
        composeTestRule.setContent {
            ArcButton(
                text = "Save",
                onClick = {},
                isLoading = true,
            )
        }

        // Then
        composeTestRule
            .onNodeWithContentDescription("Loading")
            .assertIsDisplayed()
        composeTestRule
            .onNodeWithText("Save")
            .assertDoesNotExist()
    }
}
```

### Test Fixtures

Shared test data lives in a `fixtures/` directory.

```kotlin
// src/test/kotlin/com/arclabs/feature/fixtures/RestaurantFixtures.kt
object RestaurantFixtures {

    fun createRestaurant(
        id: String = "restaurant-001",
        name: String = "Test Restaurant",
        cuisine: CuisineType = CuisineType.ITALIAN,
        rating: Double = 4.5,
    ): Restaurant = Restaurant(
        id = id,
        name = name,
        cuisine = cuisine,
        rating = rating,
        location = LatLng(37.7749, -122.4194),
    )

    fun createRestaurantList(count: Int = 5): List<Restaurant> =
        (1..count).map { index ->
            createRestaurant(
                id = "restaurant-$index",
                name = "Restaurant $index",
                rating = 3.0 + (index * 0.3),
            )
        }
}
```

---

## 📚 Documentation Requirements

### KDoc on All Public APIs

Every public class, function, property, and interface must have KDoc.

```kotlin
/**
 * Loads images from network or cache with automatic memory management.
 *
 * Usage:
 * ```kotlin
 * val loader = ArcImageLoader.create(context)
 * val bitmap = loader.load("https://example.com/image.png").getOrNull()
 * ```
 *
 * @see ImageCache for cache configuration options.
 * @since 1.0.0
 */
public class ArcImageLoader internal constructor(
    private val cache: ImageCache,
    private val fetcher: ImageFetcher,
) {

    /**
     * Loads an image from the given [url].
     *
     * The image is served from cache if available, otherwise fetched
     * from the network and cached for future requests.
     *
     * @param url The fully qualified HTTP/HTTPS URL of the image.
     * @return [Result.success] with the decoded [Bitmap], or [Result.failure]
     *         with a [NetworkError] if the load fails.
     * @throws IllegalArgumentException if [url] is blank.
     */
    public suspend fun load(url: String): Result<Bitmap> {
        require(url.isNotBlank()) { "Image URL must not be blank" }
        // ...
    }
}
```

### README Template

Every library must include a README with the following sections:

```markdown
# arc-feature

Brief one-line description of the library.

## Installation

### Gradle

```kotlin
dependencies {
    implementation("com.arclabs:arc-feature:1.0.0")
}
```

## Quick Start

Minimal code to get started.

## Features

- Feature 1
- Feature 2
- Feature 3

## Usage

### Basic Usage
### Advanced Usage
### Configuration

## API Reference

Link to generated KDoc.

## Requirements

- Min SDK: 26
- Kotlin: 2.0+
- Compose BOM: 2025.01.00+

## License

Proprietary - ARC Labs Studio
```

See [README Standards](../Quality/readme-standards.md) for the full template.

### CHANGELOG Template

Follow [Keep a Changelog](https://keepachangelog.com/) format:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [1.2.0] - 2026-02-20

### Added
- New `preload()` method for batch image preloading.
- Support for WebP animated images.

### Changed
- Improved cache eviction strategy for large images.

### Fixed
- Memory leak when loading images in RecyclerView.

## [1.1.0] - 2026-01-15

### Added
- Disk cache support with configurable size limit.
```

### Dokka Generation

```bash
# Generate HTML documentation
./gradlew dokkaHtml

# Generate for all modules
./gradlew dokkaHtmlMultiModule

# Output: build/dokka/html/index.html
```

---

## 📱 Sample / Demo App Pattern

Every UI library and most non-trivial utility libraries must ship a demo app.

### Structure

```
arc-feature/
├── arc-feature/          # The library module
│   └── ...
├── demo/                 # The demo app module
│   ├── build.gradle.kts
│   └── src/main/
│       ├── AndroidManifest.xml
│       └── kotlin/com/arclabs/feature/demo/
│           ├── DemoApp.kt
│           ├── DemoActivity.kt
│           ├── DemoNavigation.kt
│           └── screens/
│               ├── CatalogScreen.kt
│               └── DetailScreen.kt
├── build.gradle.kts
└── settings.gradle.kts
```

### Demo App Requirements

1. **Named `demo`** as the module name, with app ID `com.arclabs.<library>.demo`.
2. **Catalog screen** that shows every public component or API entry point.
3. **Interactive** - consumers can tweak parameters and see results.
4. **No production data** - use only mock/sample data.
5. **Compose only** - no XML layouts.
6. **Light and dark theme** previews included.
7. **Not published** as an artifact.

### Demo build.gradle.kts

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
}

android {
    namespace = "com.arclabs.design.demo"
    compileSdk = libs.versions.compileSdk.get().toInt()

    defaultConfig {
        applicationId = "com.arclabs.design.demo"
        minSdk = libs.versions.minSdk.get().toInt()
        targetSdk = libs.versions.targetSdk.get().toInt()
        versionCode = 1
        versionName = "1.0.0"
    }

    buildFeatures {
        compose = true
    }
}

dependencies {
    implementation(project(":arc-design"))

    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.compose.material3)
    implementation(libs.androidx.activity.compose)
    implementation(libs.androidx.navigation.compose)
}
```

### Example Catalog Screen

```kotlin
@Composable
fun CatalogScreen(
    onNavigateToDetail: (String) -> Unit,
) {
    LazyColumn(
        modifier = Modifier.fillMaxSize(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp),
    ) {
        item {
            Text(
                text = "ARC Design Catalog",
                style = MaterialTheme.typography.headlineMedium,
            )
        }

        item {
            SectionHeader("Buttons")
            ArcButton(text = "Primary", onClick = {}, style = ArcButtonStyle.Primary)
            Spacer(modifier = Modifier.height(8.dp))
            ArcButton(text = "Secondary", onClick = {}, style = ArcButtonStyle.Secondary)
            Spacer(modifier = Modifier.height(8.dp))
            ArcButton(text = "Loading", onClick = {}, isLoading = true)
            Spacer(modifier = Modifier.height(8.dp))
            ArcButton(text = "Disabled", onClick = {}, enabled = false)
        }

        item {
            SectionHeader("Text Fields")
            ArcTextField(value = "Sample input", onValueChange = {}, label = "Label")
        }
    }
}
```

---

## 📦 Dependency Management

### Version Catalogs

All dependencies are managed through a single `libs.versions.toml` at the root of the repository. Libraries must NEVER hardcode dependency versions in `build.gradle.kts`.

```kotlin
// ✅ Correct: Reference from version catalog
dependencies {
    implementation(libs.kotlinx.coroutines.android)
    implementation(libs.hilt.android)
}
```

```kotlin
// ❌ Incorrect: Hardcoded versions
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.9.0")
    implementation("com.google.dagger:hilt-android:2.53.1")
}
```

### Transitive Dependency Rules

1. **Minimize transitive dependencies.** Use `api` only when the dependency type appears in the library's public API surface.
2. **Prefer `implementation` over `api`.** Implementation details must not leak to consumers.
3. **Never expose third-party types in public APIs.** Wrap them in library-owned types.

```kotlin
// ✅ Correct: OkHttp is an implementation detail
dependencies {
    implementation(libs.okhttp)       // Hidden from consumers
    implementation(libs.retrofit)     // Hidden from consumers
    api(libs.kotlinx.coroutines.core) // Exposed because suspend functions are in the API
}
```

```kotlin
// ❌ Incorrect: Leaking implementation dependencies
dependencies {
    api(libs.okhttp)    // Consumers now depend on OkHttp!
    api(libs.retrofit)  // Consumers now depend on Retrofit!
}
```

### BOM (Bill of Materials) for Multi-Artifact Libraries

For libraries with multiple artifacts, publish a BOM so consumers get compatible versions automatically.

```kotlin
// BOM module: arc-bom/build.gradle.kts
plugins {
    `java-platform`
    `maven-publish`
}

dependencies {
    constraints {
        api("com.arclabs:arc-design:1.2.0")
        api("com.arclabs:arc-network:1.5.3")
        api("com.arclabs:arc-analytics:0.8.0")
        api("com.arclabs:arc-core:2.0.1")
    }
}
```

```kotlin
// Consumer usage
dependencies {
    implementation(platform("com.arclabs:arc-bom:2026.02.00"))
    implementation("com.arclabs:arc-design")   // Version from BOM
    implementation("com.arclabs:arc-network")  // Version from BOM
}
```

---

## 🛡 ProGuard / R8 Consumer Rules

Libraries must ship consumer ProGuard rules so that consumers' R8 optimization does not break library functionality.

### consumer-proguard-rules.pro

```proguard
# Keep public API
-keep public class com.arclabs.feature.** {
    public *;
}

# Keep data classes used for serialization
-keepclassmembers class com.arclabs.feature.model.** {
    <fields>;
    <init>(...);
}

# Keep Hilt generated components (if using Hilt)
-keep class com.arclabs.feature.di.** { *; }

# Keep enum values
-keepclassmembers enum com.arclabs.feature.** {
    **[] $VALUES;
    public *;
}

# Keep Kotlin metadata for reflection
-keep class kotlin.Metadata { *; }
```

### Rules for Different Library Types

| Library Type | Key ProGuard Considerations |
|---|---|
| UI Component | Keep Compose functions annotated with `@Composable`. Generally safe without extra rules. |
| Utility | Minimal rules; mostly pure Kotlin. |
| Data / Network | Keep serialization models, Retrofit interfaces. |
| Domain | Keep data classes and sealed interfaces. |

### Testing Consumer Rules

Verify that consumer rules work by enabling R8 in the demo app:

```kotlin
// demo/build.gradle.kts
android {
    buildTypes {
        release {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro",
            )
        }
    }
}
```

Build the demo app in release mode and verify no crashes or missing classes:

```bash
./gradlew :demo:assembleRelease
```

---

## ✅ Checklist for New Libraries

Use this checklist every time you create a new ARC Labs Android library.

### Project Setup

- [ ] Module name follows `arc-<name>` convention.
- [ ] Package name follows `com.arclabs.<name>` convention.
- [ ] `build.gradle.kts` uses the standard template with version catalog references.
- [ ] `kotlin { explicitApi() }` is enabled.
- [ ] `consumer-proguard-rules.pro` is present and configured.
- [ ] `minSdk` is set to `26`.
- [ ] JUnit 5 configured with `useJUnitPlatform()`.

### API Design

- [ ] All implementation details are `internal` or `private`.
- [ ] No third-party types leaked in public API.
- [ ] All public functions return `Result<T>` or sealed error types instead of throwing.
- [ ] Builder DSL provided for complex configuration.
- [ ] Default parameter values provided where sensible.
- [ ] Extension functions used for convenience APIs.

### Testing

- [ ] 100% line coverage achieved.
- [ ] 95%+ branch coverage achieved.
- [ ] All tests follow Given-When-Then pattern.
- [ ] `makeSUT()` factory used in all test classes.
- [ ] Flows tested with Turbine.
- [ ] Test fixtures in `fixtures/` directory.
- [ ] Tests run on CI with every PR.

### Documentation

- [ ] KDoc on every public class, function, and property.
- [ ] `README.md` follows the standard template.
- [ ] `CHANGELOG.md` initialized with `[Unreleased]` section.
- [ ] Code examples in KDoc are compilable.
- [ ] Dokka generates without errors.

### Publishing

- [ ] `maven-publish` plugin applied.
- [ ] Group ID, artifact ID, and version configured.
- [ ] `singleVariant("release")` with sources and javadoc JARs.
- [ ] GitHub Packages publishing configured.
- [ ] CI workflow for automated publishing on release.

### Demo App

- [ ] `demo` module created (for UI and complex libraries).
- [ ] Catalog screen showing all components/features.
- [ ] Demo app builds and runs without errors.
- [ ] Release build with R8 verified.
- [ ] Light and dark theme included.

### Code Quality

- [ ] ktlint passes with zero violations.
- [ ] detekt passes with zero violations.
- [ ] No `TODO` or `FIXME` comments in released code.
- [ ] No unused dependencies.
- [ ] Library builds in isolation (no app module dependency).

---

## ❌ Common Mistakes

### 1. Leaking Implementation Dependencies

```kotlin
// ❌ WRONG: Exposing Retrofit Response type to consumers
public interface UserApi {
    suspend fun getUser(id: String): Response<UserDto>  // Retrofit type leaked!
}
```

```kotlin
// ✅ CORRECT: Library-owned return type
public interface UserApi {
    suspend fun getUser(id: String): Result<User>
}
```

### 2. Missing `internal` Visibility

```kotlin
// ❌ WRONG: Everything is public by default
class UserRepositoryImpl(           // Implicitly public!
    val apiClient: ApiClient,       // Implicitly public property!
) : UserRepository {
    override suspend fun getUser(id: String): Result<User> { /* ... */ }
}
```

```kotlin
// ✅ CORRECT: Implementation is hidden
internal class UserRepositoryImpl(
    private val apiClient: ApiClient,
) : UserRepository {
    override suspend fun getUser(id: String): Result<User> { /* ... */ }
}
```

### 3. Hardcoded Dependency Versions

```kotlin
// ❌ WRONG: Version hardcoded in build.gradle.kts
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.9.0")
}
```

```kotlin
// ✅ CORRECT: Version from catalog
dependencies {
    implementation(libs.kotlinx.coroutines.android)
}
```

### 4. Throwing Exceptions Across Boundaries

```kotlin
// ❌ WRONG: Throwing exceptions consumers may not catch
public suspend fun loadProfile(userId: String): UserProfile {
    val response = api.fetchProfile(userId)
    if (!response.isSuccessful) {
        throw HttpException(response)  // Consumer may not expect this!
    }
    return response.body()!!           // Force unwrap!
}
```

```kotlin
// ✅ CORRECT: Returning Result with explicit error types
public suspend fun loadProfile(userId: String): Result<UserProfile> =
    runCatching {
        val response = api.fetchProfile(userId)
        if (!response.isSuccessful) {
            error("HTTP ${response.code()}: ${response.errorBody()?.string()}")
        }
        response.body() ?: error("Empty response body")
    }
```

### 5. Using `api` Instead of `implementation`

```kotlin
// ❌ WRONG: All dependencies are `api` scope
dependencies {
    api(libs.okhttp)
    api(libs.moshi)
    api(libs.retrofit)
}
```

```kotlin
// ✅ CORRECT: Only types in the public API use `api` scope
dependencies {
    implementation(libs.okhttp)
    implementation(libs.moshi)
    implementation(libs.retrofit)
    api(libs.kotlinx.coroutines.core) // Exposed in public suspend functions
}
```

### 6. Missing Consumer ProGuard Rules

```kotlin
// ❌ WRONG: No consumer-proguard-rules.pro
// Consumer enables R8 -> library data classes get stripped -> runtime crash
```

```kotlin
// ✅ CORRECT: Ship consumer rules in consumer-proguard-rules.pro
// -keep class com.arclabs.feature.model.** { *; }
```

### 7. Tests Without Structure

```kotlin
// ❌ WRONG: No structure, no factory, no Given-When-Then
class UserRepoTest {
    @Test
    fun test1() {
        val repo = UserRepositoryImpl(FakeApiClient())
        val result = runBlocking { repo.getUser("1") }
        assert(result.isSuccess)
    }
}
```

```kotlin
// ✅ CORRECT: Structured, readable, maintainable
@Tag("unit")
@DisplayName("UserRepositoryImpl")
class UserRepositoryImplTest {

    private val mockApi: ApiClient = mockk()

    private fun makeSUT(): UserRepositoryImpl {
        return UserRepositoryImpl(apiClient = mockApi)
    }

    @Test
    @DisplayName("should return user when API succeeds")
    fun returnsUserOnSuccess() = runTest {
        // Given
        val sut = makeSUT()
        val expectedUser = UserFixtures.createUser()
        coEvery { mockApi.getUser("1") } returns expectedUser

        // When
        val result = sut.getUser("1")

        // Then
        assertThat(result.getOrNull()).isEqualTo(expectedUser)
    }
}
```

### 8. Breaking API Without Major Version Bump

```kotlin
// ❌ WRONG: Breaking change in a minor release
// Version 1.0.0
fun log(message: String)

// Version 1.1.0 - Existing callers break!
fun log(message: String, level: LogLevel)
```

```kotlin
// ✅ CORRECT: Backward compatible addition
// Version 1.1.0 - Add default parameter
fun log(message: String, level: LogLevel = LogLevel.DEBUG)
```

### 9. No CHANGELOG Updates

```markdown
<!-- ❌ WRONG: Empty or missing CHANGELOG -->
```

```markdown
<!-- ✅ CORRECT: Updated with every release -->
## [1.3.0] - 2026-02-25

### Added
- New `ArcChip` component with selectable and dismissible variants.

### Fixed
- `ArcButton` ripple effect not matching theme colors.
```

### 10. Skipping the Demo App

```
❌ WRONG: "The library is simple enough, it doesn't need a demo."

✅ CORRECT: Every UI library ships a demo. Even simple ones.
   - Validates the API ergonomics in a real app context.
   - Serves as living documentation.
   - Catches integration issues early.
```

### 11. Not Using Explicit API Mode

```kotlin
// ❌ WRONG: Relying on Kotlin's default public visibility
class FeatureManager {           // Public by accident
    fun doInternalThing() { }    // Public by accident
    fun doPublicThing() { }
}
```

```kotlin
// ✅ CORRECT: Enable explicit API mode in build.gradle.kts
// kotlin { explicitApi() }
// Now the compiler forces you to declare visibility:
public class FeatureManager {
    internal fun doInternalThing() { }
    public fun doPublicThing() { }
}
```

### 12. Including Business Logic in Libraries

```kotlin
// ❌ WRONG: Business logic in a library
class RestaurantValidator {
    fun isValidForBooking(restaurant: Restaurant): Boolean {
        return restaurant.isOpen && restaurant.hasAvailableSeats
    }
}
```

```kotlin
// ✅ CORRECT: Generic, reusable utilities
class Validator<T> {
    fun validate(value: T, rules: List<ValidationRule<T>>): ValidationResult {
        return rules.fold(ValidationResult.Valid as ValidationResult) { acc, rule ->
            if (acc is ValidationResult.Invalid) acc else rule.validate(value)
        }
    }
}
```

---

## 📖 Further Reading

- [App Architecture](./apps.md) - Standards for Android app projects that consume libraries.
- [Testing Standards](../Quality/testing.md) - Full testing guidelines including JUnit 5, MockK, and Turbine patterns.
- [Code Style](../Quality/code-style.md) - ktlint and detekt configuration for all ARC Labs projects.
- [README Standards](../Quality/readme-standards.md) - Template and rules for library README files.
- [Module Structure](../Quality/module-structure.md) - How to organize modules in multi-module projects.
- [Documentation Standards](../Quality/documentation.md) - KDoc and Dokka requirements.
- [Git Commits](../Workflow/git-commits.md) - Conventional Commits standards and formatting.
- [Git Branches](../Workflow/git-branches.md) - Branch naming, workflow, and release flow.
- [Dependency Injection](../Architecture/dependency-injection.md) - Hilt DI patterns and scoping strategy.
- [Compose Performance](../Quality/compose-performance.md) - Best practices for Jetpack Compose in libraries and apps.
- [Gradle Configuration](../Tools/gradle.md) - Gradle Kotlin DSL patterns and conventions.

---

**Remember**: Libraries are the foundation. Build them solid, document them well, test them thoroughly.
