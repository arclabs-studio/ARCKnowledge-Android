# 📦 Module Structure

**Modular architecture improves build times, enforces boundaries, and enables code reuse. ARC Labs Android projects use Gradle multi-module structure with convention plugins and strict dependency rules.**

---

## 🎯 Modularization Philosophy

### Why Modularize?

Modularization is not just a structural preference — it is an architectural decision that directly impacts build performance, team velocity, code quality, and long-term maintainability. Every ARC Labs Android project adopts multi-module architecture from the start.

### Core Principles

- **Single Responsibility:** Each module has one clear purpose and one reason to change
- **Compile-Time Boundaries:** Modules enforce dependency rules at compile time, not by convention
- **Feature Independence:** Feature modules are independent and testable in isolation
- **Minimal Public Surface:** Modules expose only what is necessary via `public` / `internal` visibility
- **Unidirectional Dependencies:** Dependencies flow in one direction — never circular

### Benefits

| Benefit | Description |
|---------|-------------|
| **Build Speed** | Gradle builds only changed modules and their dependents. Unchanged modules use cached artifacts, dramatically reducing incremental build times. |
| **Encapsulation** | Kotlin `internal` visibility works at the module level. Implementation details stay hidden from other modules. |
| **Reuse** | Core modules can be shared across multiple apps without modification. |
| **Testability** | Each module can be tested in isolation with its own test suite and mocked dependencies. |
| **Team Scalability** | Teams can own individual modules and work in parallel without merge conflicts. |
| **Enforced Architecture** | Clean Architecture layers become compile-time guarantees, not just folder conventions. |
| **Faster CI** | Parallel module compilation and caching reduce CI pipeline duration. |

---

## 📂 Standard Module Structure

### App Module Structure

The `app` module is the entry point. It wires together all features, sets up dependency injection, and owns the top-level navigation graph. It should contain as little code as possible.

```
app/
├── src/
│   ├── main/
│   │   ├── java/com/arclabs/myapp/
│   │   │   ├── MyApplication.kt              (@HiltAndroidApp)
│   │   │   ├── MainActivity.kt               (@AndroidEntryPoint)
│   │   │   ├── navigation/
│   │   │   │   ├── AppNavDisplay.kt           (Top-level NavDisplay + entryProvider)
│   │   │   │   ├── Navigator.kt               (NavigationState + Navigator)
│   │   │   │   └── TopLevelDestination.kt     (NavigationSuiteScaffold destinations)
│   │   │   └── di/
│   │   │       └── AppModule.kt               (App-scoped bindings)
│   │   ├── res/
│   │   │   ├── values/
│   │   │   │   ├── strings.xml
│   │   │   │   └── themes.xml
│   │   │   ├── drawable/
│   │   │   └── mipmap/
│   │   └── AndroidManifest.xml
│   ├── test/
│   │   └── java/com/arclabs/myapp/
│   │       └── navigation/
│   │           └── NavigatorTest.kt
│   └── androidTest/
│       └── java/com/arclabs/myapp/
│           └── MainActivityTest.kt
├── build.gradle.kts
└── proguard-rules.pro
```

**Key rules for the app module:**
- No business logic — only wiring and navigation
- Depends on all feature modules
- Depends on core modules as needed
- Owns the application-level Hilt component
- Contains ProGuard / R8 rules

### Feature Module Structure

Each feature module follows Clean Architecture internally, with `presentation`, `domain`, and `data` layers. A feature module is self-contained and owns its entire vertical slice.

```
feature/
├── home/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/arclabs/myapp/feature/home/
│   │   │   │   ├── presentation/
│   │   │   │   │   ├── HomeScreen.kt              (@Composable screen)
│   │   │   │   │   ├── HomeViewModel.kt           (@HiltViewModel)
│   │   │   │   │   ├── HomeUiState.kt             (UI state sealed class)
│   │   │   │   │   ├── HomeUiEvent.kt             (User action events)
│   │   │   │   │   ├── HomeNavigation.kt          (Route + nav extension)
│   │   │   │   │   └── components/
│   │   │   │   │       ├── HomeTopBar.kt
│   │   │   │   │       ├── HomeContentList.kt
│   │   │   │   │       └── HomeEmptyState.kt
│   │   │   │   ├── domain/
│   │   │   │   │   ├── model/
│   │   │   │   │   │   ├── HomeItem.kt            (Domain entity)
│   │   │   │   │   │   └── HomeFilter.kt          (Value object)
│   │   │   │   │   ├── usecase/
│   │   │   │   │   │   ├── GetHomeItemsUseCase.kt
│   │   │   │   │   │   └── RefreshHomeUseCase.kt
│   │   │   │   │   └── repository/
│   │   │   │   │       └── HomeRepository.kt      (Interface only)
│   │   │   │   ├── data/
│   │   │   │   │   ├── repository/
│   │   │   │   │   │   └── HomeRepositoryImpl.kt  (Implementation)
│   │   │   │   │   ├── remote/
│   │   │   │   │   │   ├── HomeApiService.kt      (Retrofit interface)
│   │   │   │   │   │   └── dto/
│   │   │   │   │   │       └── HomeItemDto.kt     (Network DTO)
│   │   │   │   │   ├── local/
│   │   │   │   │   │   ├── HomeDao.kt             (Room DAO)
│   │   │   │   │   │   └── entity/
│   │   │   │   │   │       └── HomeItemEntity.kt  (Database entity)
│   │   │   │   │   ├── mapper/
│   │   │   │   │   │   ├── HomeItemDtoMapper.kt
│   │   │   │   │   │   └── HomeItemEntityMapper.kt
│   │   │   │   │   └── di/
│   │   │   │   │       └── HomeDataModule.kt      (@Module for bindings)
│   │   │   │   └── di/
│   │   │   │       └── HomeDomainModule.kt        (@Module for use cases)
│   │   │   ├── res/
│   │   │   │   └── values/
│   │   │   │       └── strings.xml                (Feature-specific strings)
│   │   │   └── AndroidManifest.xml
│   │   ├── test/
│   │   │   └── java/com/arclabs/myapp/feature/home/
│   │   │       ├── presentation/
│   │   │       │   └── HomeViewModelTest.kt
│   │   │       ├── domain/
│   │   │       │   └── usecase/
│   │   │       │       ├── GetHomeItemsUseCaseTest.kt
│   │   │       │       └── RefreshHomeUseCaseTest.kt
│   │   │       └── data/
│   │   │           ├── repository/
│   │   │           │   └── HomeRepositoryImplTest.kt
│   │   │           └── mapper/
│   │   │               ├── HomeItemDtoMapperTest.kt
│   │   │               └── HomeItemEntityMapperTest.kt
│   │   └── androidTest/
│   │       └── java/com/arclabs/myapp/feature/home/
│   │           └── presentation/
│   │               └── HomeScreenTest.kt
│   └── build.gradle.kts
```

### Core Module Structures

Core modules provide shared infrastructure. Each core module has a focused responsibility.

#### core:network

```
core/network/
├── src/main/java/com/arclabs/myapp/core/network/
│   ├── NetworkClient.kt                 (OkHttpClient provider)
│   ├── RetrofitFactory.kt               (Retrofit instance builder)
│   ├── interceptor/
│   │   ├── AuthInterceptor.kt           (Token injection)
│   │   ├── LoggingInterceptor.kt        (Request/response logging)
│   │   └── ConnectivityInterceptor.kt   (Network check)
│   ├── model/
│   │   ├── NetworkResult.kt             (Sealed result wrapper)
│   │   └── ApiError.kt                  (Error model)
│   ├── adapter/
│   │   └── NetworkResultCallAdapter.kt  (Retrofit call adapter)
│   ├── di/
│   │   └── NetworkModule.kt             (@Module @InstallIn(SingletonComponent))
│   └── util/
│       └── SafeApiCall.kt               (Extension for safe API calls)
├── src/test/java/com/arclabs/myapp/core/network/
│   ├── interceptor/
│   │   └── AuthInterceptorTest.kt
│   └── util/
│       └── SafeApiCallTest.kt
└── build.gradle.kts
```

#### core:database

```
core/database/
├── src/main/java/com/arclabs/myapp/core/database/
│   ├── AppDatabase.kt                   (@Database definition)
│   ├── DatabaseFactory.kt               (Room.databaseBuilder helper)
│   ├── converter/
│   │   ├── DateConverter.kt             (TypeConverter for dates)
│   │   └── ListConverter.kt             (TypeConverter for lists)
│   ├── migration/
│   │   ├── Migration1to2.kt
│   │   └── MigrationHelper.kt
│   ├── di/
│   │   └── DatabaseModule.kt            (@Module @InstallIn(SingletonComponent))
│   └── util/
│       └── TransactionRunner.kt         (Transaction helper)
├── src/test/java/com/arclabs/myapp/core/database/
│   └── migration/
│       └── MigrationTest.kt
├── src/androidTest/java/com/arclabs/myapp/core/database/
│   └── AppDatabaseTest.kt
├── schemas/                             (Room schema exports for migration testing)
└── build.gradle.kts
```

#### core:common

```
core/common/
├── src/main/java/com/arclabs/myapp/core/common/
│   ├── result/
│   │   ├── Result.kt                    (Sealed class: Success/Error/Loading)
│   │   └── ResultExtensions.kt          (map, flatMap, onSuccess, onError)
│   ├── extension/
│   │   ├── StringExtensions.kt
│   │   ├── DateExtensions.kt
│   │   ├── FlowExtensions.kt
│   │   └── CoroutineExtensions.kt
│   ├── dispatcher/
│   │   ├── DispatcherProvider.kt        (Interface)
│   │   └── DefaultDispatcherProvider.kt (Implementation)
│   ├── di/
│   │   ├── CommonModule.kt
│   │   └── DispatcherModule.kt          (Provides @IoDispatcher, @DefaultDispatcher)
│   └── qualifier/
│       └── Qualifiers.kt               (@IoDispatcher, @DefaultDispatcher, @MainDispatcher)
├── src/test/java/com/arclabs/myapp/core/common/
│   ├── result/
│   │   └── ResultExtensionsTest.kt
│   └── extension/
│       └── StringExtensionsTest.kt
└── build.gradle.kts
```

#### core:ui

```
core/ui/
├── src/main/java/com/arclabs/myapp/core/ui/
│   ├── theme/
│   │   ├── Theme.kt                     (MaterialTheme wrapper)
│   │   ├── Color.kt                     (Brand colors)
│   │   ├── Typography.kt               (Type scale)
│   │   ├── Shape.kt                     (Shape definitions)
│   │   └── Spacing.kt                   (Custom spacing tokens)
│   ├── component/
│   │   ├── ArcButton.kt                (Branded button variants)
│   │   ├── ArcTextField.kt             (Branded text field)
│   │   ├── ArcTopAppBar.kt             (Branded top bar)
│   │   ├── ArcLoadingIndicator.kt      (Loading spinner)
│   │   └── ArcErrorState.kt            (Error display)
│   ├── modifier/
│   │   ├── ShimmerModifier.kt          (Shimmer loading effect)
│   │   └── ConditionalModifier.kt      (Conditional modifier extension)
│   ├── preview/
│   │   └── PreviewParameterProviders.kt (Shared preview providers)
│   └── util/
│       └── ComposeExtensions.kt        (Reusable compose utilities)
├── src/test/
└── build.gradle.kts
```

#### core:testing

```
core/testing/
├── src/main/java/com/arclabs/myapp/core/testing/
│   ├── rule/
│   │   ├── MainDispatcherRule.kt        (Coroutine test rule)
│   │   └── HiltTestRule.kt             (Custom Hilt rule)
│   ├── fake/
│   │   ├── FakeDispatcherProvider.kt   (Test dispatcher provider)
│   │   └── FakeNetworkMonitor.kt       (Test network monitor)
│   ├── factory/
│   │   └── TestDataFactory.kt          (Builder for test objects)
│   ├── extension/
│   │   ├── FlowTestExtensions.kt       (Flow testing helpers)
│   │   └── ComposeTestExtensions.kt    (Compose test helpers)
│   └── runner/
│       └── HiltTestRunner.kt           (Custom AndroidJUnitRunner)
└── build.gradle.kts
```

#### core:datastore

```
core/datastore/
├── src/main/java/com/arclabs/myapp/core/datastore/
│   ├── UserPreferences.kt               (Proto / Preferences DataStore)
│   ├── UserPreferencesSerializer.kt     (Proto serializer)
│   ├── PreferencesDataSource.kt         (DataStore wrapper)
│   ├── model/
│   │   ├── ThemeConfig.kt              (Theme preference model)
│   │   └── UserSettings.kt            (User settings model)
│   └── di/
│       └── DataStoreModule.kt          (@Module @InstallIn(SingletonComponent))
├── src/test/java/com/arclabs/myapp/core/datastore/
│   └── PreferencesDataSourceTest.kt
├── src/main/proto/
│   └── user_preferences.proto           (Proto schema, if using Proto DataStore)
└── build.gradle.kts
```

### Shared Domain Module (Optional)

When multiple features need to share domain entities or repository contracts, create a shared `:domain` module. This module contains **only** interfaces and data classes — no implementations.

```
domain/
├── src/main/java/com/arclabs/myapp/domain/
│   ├── model/
│   │   ├── User.kt                      (Shared User entity)
│   │   ├── Location.kt                  (Shared Location entity)
│   │   └── PagingConfig.kt              (Shared paging params)
│   ├── repository/
│   │   ├── UserRepository.kt            (Interface)
│   │   └── LocationRepository.kt        (Interface)
│   └── usecase/
│       └── GetCurrentUserUseCase.kt     (Cross-feature use case)
├── src/test/java/com/arclabs/myapp/domain/
│   └── usecase/
│       └── GetCurrentUserUseCaseTest.kt
└── build.gradle.kts
```

---

## 📦 Module Types

| Type | Purpose | Example | Depends On | Contains |
|------|---------|---------|------------|----------|
| `app` | Entry point, DI root, navigation shell | `:app` | `:feature:*`, `:core:*`, `:domain` | Application, MainActivity, NavDisplay |
| `feature` | Self-contained feature vertical slice | `:feature:home`, `:feature:search` | `:core:*`, `:domain` | Presentation + Domain + Data |
| `core` | Shared infrastructure and utilities | `:core:network`, `:core:database` | External libraries only | Infrastructure code |
| `domain` | Shared domain models and contracts | `:domain` | `:core:common` (optional) | Entities, interfaces, shared use cases |
| `core:ui` | Design system and shared composables | `:core:ui` | Compose libraries, `:core:common` | Theme, components, modifiers |
| `core:testing` | Shared test utilities | `:core:testing` | Test libraries, `:core:common` | Fakes, rules, extensions |

---

## 🔗 Dependency Rules

### Dependency Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│                        :app                             │
│  (entry point, navigation, DI root)                     │
└─────┬───────────┬───────────┬───────────┬───────────────┘
      │           │           │           │
      v           v           v           v
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│ :feature │ │ :feature │ │ :feature │ │ :feature │
│  :home   │ │ :search  │ │ :profile │ │ :settings│
└────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘
     │             │            │             │
     └──────┬──────┘────────────┘─────────────┘
            │
            v
     ┌─────────────┐
     │   :domain    │  (shared entities & contracts)
     └──────┬──────┘
            │
            v
┌───────────────────────────────────────────────────────┐
│                     :core:*                            │
│  ┌──────────┐ ┌──────────┐ ┌────────┐ ┌───────────┐  │
│  │ :network │ │ :database│ │ :ui    │ │ :common   │  │
│  └──────────┘ └──────────┘ └────────┘ └───────────┘  │
│  ┌──────────┐ ┌───────────┐                           │
│  │:datastore│ │ :testing  │                           │
│  └──────────┘ └───────────┘                           │
└───────────────────────────────────────────────────────┘
            │
            v
    [External Libraries]
```

### Rules by Module Type

**App module (`:app`):**
- CAN depend on: any `:feature:*` module, any `:core:*` module, `:domain`
- CANNOT depend on: nothing restricts it, but it should contain minimal code

**Feature modules (`:feature:*`):**
- CAN depend on: `:core:*` modules, `:domain`
- CANNOT depend on: other `:feature:*` modules, `:app`
- This is the most critical rule. Feature-to-feature dependencies break isolation.

**Core modules (`:core:*`):**
- CAN depend on: external libraries only, and optionally `:core:common`
- CANNOT depend on: `:feature:*`, `:app`, `:domain`
- Exception: `:core:testing` may depend on other `:core:*` modules to provide test fakes

**Domain module (`:domain`):**
- CAN depend on: `:core:common` (for shared types like Result)
- CANNOT depend on: `:feature:*`, `:app`, `:core:network`, `:core:database`
- Must remain framework-free (no Android imports, no Retrofit, no Room)

### Hilt Across Module Boundaries

Hilt uses `@InstallIn` to scope bindings to components. In a multi-module project, each module provides its own `@Module` that installs into the appropriate component.

```kotlin
// In :feature:home — data/di/HomeDataModule.kt
@Module
@InstallIn(SingletonComponent::class)
abstract class HomeDataModule {

    @Binds
    abstract fun bindHomeRepository(
        impl: HomeRepositoryImpl
    ): HomeRepository
}
```

```kotlin
// In :core:network — di/NetworkModule.kt
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    @Singleton
    fun provideOkHttpClient(
        authInterceptor: AuthInterceptor
    ): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(authInterceptor)
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(
        client: OkHttpClient
    ): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BuildConfig.BASE_URL)
            .client(client)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
}
```

Hilt merges all `@Module` classes from all modules at compile time into the app's component hierarchy. Feature modules do NOT need to know about each other — Hilt resolves the entire graph in `:app`.

### Sharing Entities Between Features

When two features need the same domain entity (e.g., `User`), **do not** create a dependency between features. Instead:

1. Place the shared entity in the `:domain` module
2. Both features depend on `:domain`
3. Each feature has its own repository implementation that provides that entity

```kotlin
// :domain/model/User.kt
data class User(
    val id: String,
    val name: String,
    val email: String
)

// :domain/repository/UserRepository.kt
interface UserRepository {
    suspend fun getCurrentUser(): User
    fun observeCurrentUser(): Flow<User>
}

// :feature:home uses UserRepository via injection
// :feature:profile uses UserRepository via injection
// The implementation lives in whichever module owns the data source
```

---

## 📐 build.gradle.kts Conventions

### Convention Plugins Pattern

Convention plugins centralize shared build logic so individual module `build.gradle.kts` files remain minimal and consistent. ARC Labs uses a `build-logic` included build for this purpose.

#### Project Structure with Convention Plugins

```
project-root/
├── build-logic/
│   ├── convention/
│   │   ├── src/main/kotlin/
│   │   │   ├── AndroidApplicationConventionPlugin.kt
│   │   │   ├── AndroidLibraryConventionPlugin.kt
│   │   │   ├── AndroidFeatureConventionPlugin.kt
│   │   │   ├── AndroidComposeConventionPlugin.kt
│   │   │   ├── AndroidHiltConventionPlugin.kt
│   │   │   └── AndroidTestConventionPlugin.kt
│   │   └── build.gradle.kts
│   ├── settings.gradle.kts
│   └── gradle.properties
├── app/
├── feature/
├── core/
├── domain/
├── gradle/
│   └── libs.versions.toml
├── settings.gradle.kts
└── build.gradle.kts
```

#### Convention Plugin Example: Android Feature Module

```kotlin
// build-logic/convention/src/main/kotlin/AndroidFeatureConventionPlugin.kt
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.kotlin.dsl.dependencies

class AndroidFeatureConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply("com.android.library")
                apply("org.jetbrains.kotlin.android")
                apply("org.jetbrains.kotlin.plugin.compose")
                apply("com.google.devtools.ksp")
                apply("com.google.dagger.hilt.android")
            }

            extensions.configure<com.android.build.gradle.LibraryExtension> {
                compileSdk = 35

                defaultConfig {
                    minSdk = 26
                    testInstrumentationRunner = "com.arclabs.myapp.core.testing.HiltTestRunner"
                }

                buildFeatures {
                    compose = true
                }

                compileOptions {
                    sourceCompatibility = JavaVersion.VERSION_17
                    targetCompatibility = JavaVersion.VERSION_17
                }
            }

            val libs = extensions.getByType<VersionCatalogsExtension>().named("libs")

            dependencies {
                add("implementation", project(":core:ui"))
                add("implementation", project(":core:common"))
                add("implementation", project(":domain"))

                add("implementation", libs.findLibrary("hilt.android").get())
                add("ksp", libs.findLibrary("hilt.compiler").get())
                add("implementation", libs.findLibrary("hilt.navigation.compose").get())

                add("implementation", libs.findLibrary("androidx.lifecycle.viewmodel.compose").get())
                add("implementation", libs.findLibrary("androidx.navigation.compose").get())

                add("testImplementation", project(":core:testing"))
                add("testImplementation", libs.findLibrary("junit5").get())
                add("testImplementation", libs.findLibrary("mockk").get())
                add("testImplementation", libs.findLibrary("turbine").get())
                add("testImplementation", libs.findLibrary("kotlinx.coroutines.test").get())
            }
        }
    }
}
```

#### Registering Convention Plugins

```kotlin
// build-logic/convention/build.gradle.kts
plugins {
    `kotlin-dsl`
}

dependencies {
    compileOnly(libs.android.gradlePlugin)
    compileOnly(libs.kotlin.gradlePlugin)
    compileOnly(libs.compose.gradlePlugin)
    compileOnly(libs.ksp.gradlePlugin)
}

gradlePlugin {
    plugins {
        register("androidApplication") {
            id = "arclabs.android.application"
            implementationClass = "AndroidApplicationConventionPlugin"
        }
        register("androidLibrary") {
            id = "arclabs.android.library"
            implementationClass = "AndroidLibraryConventionPlugin"
        }
        register("androidFeature") {
            id = "arclabs.android.feature"
            implementationClass = "AndroidFeatureConventionPlugin"
        }
        register("androidCompose") {
            id = "arclabs.android.compose"
            implementationClass = "AndroidComposeConventionPlugin"
        }
        register("androidHilt") {
            id = "arclabs.android.hilt"
            implementationClass = "AndroidHiltConventionPlugin"
        }
        register("androidTest") {
            id = "arclabs.android.test"
            implementationClass = "AndroidTestConventionPlugin"
        }
    }
}
```

#### Using Convention Plugins in Feature Modules

With convention plugins, a feature module's `build.gradle.kts` becomes minimal:

```kotlin
// feature/home/build.gradle.kts
plugins {
    id("arclabs.android.feature")
}

android {
    namespace = "com.arclabs.myapp.feature.home"
}

dependencies {
    implementation(project(":core:network"))
    implementation(project(":core:database"))
}
```

Compare this to the non-convention-plugin version:

```kotlin
// feature/home/build.gradle.kts (WITHOUT convention plugins — verbose)
plugins {
    id("com.android.library")
    id("org.jetbrains.kotlin.android")
    id("org.jetbrains.kotlin.plugin.compose")
    id("com.google.devtools.ksp")
    id("com.google.dagger.hilt.android")
}

android {
    namespace = "com.arclabs.myapp.feature.home"
    compileSdk = libs.versions.compileSdk.get().toInt()

    defaultConfig {
        minSdk = libs.versions.minSdk.get().toInt()
        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildFeatures {
        compose = true
    }
}

dependencies {
    implementation(project(":core:network"))
    implementation(project(":core:database"))
    implementation(project(":core:ui"))
    implementation(project(":core:common"))
    implementation(project(":domain"))

    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)
    implementation(libs.hilt.navigation.compose)

    testImplementation(project(":core:testing"))
    testImplementation(libs.junit5)
    testImplementation(libs.mockk)
    testImplementation(libs.turbine)
    testImplementation(libs.kotlinx.coroutines.test)
}
```

### settings.gradle.kts with Module Includes

```kotlin
// settings.gradle.kts
pluginManagement {
    includeBuild("build-logic")
    repositories {
        google()
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

// App
include(":app")

// Features
include(":feature:home")
include(":feature:search")
include(":feature:profile")
include(":feature:settings")

// Core
include(":core:network")
include(":core:database")
include(":core:common")
include(":core:ui")
include(":core:testing")
include(":core:datastore")

// Shared Domain
include(":domain")
```

---

## 📋 Version Catalogs (libs.versions.toml)

ARC Labs standardizes all dependency versions in a single `gradle/libs.versions.toml` file. No version strings should appear in any `build.gradle.kts` file.

```toml
[versions]
# Kotlin & Core
kotlin = "2.0.21"
kotlinx-coroutines = "1.9.0"
kotlinx-serialization = "1.7.3"
ksp = "2.0.21-1.0.28"

# Android
compileSdk = "35"
minSdk = "26"
targetSdk = "35"
agp = "8.7.3"

# Compose
compose-bom = "2024.12.01"
compose-compiler = "1.5.15"
navigation-compose = "2.8.5"
lifecycle = "2.8.7"
activity-compose = "1.9.3"

# Dependency Injection
hilt = "2.51.1"
hilt-navigation-compose = "1.2.0"

# Networking
retrofit = "2.9.0"
okhttp = "4.12.0"
gson = "2.11.0"

# Database
room = "2.6.1"

# DataStore
datastore = "1.1.1"
protobuf = "4.28.3"
protobuf-plugin = "0.9.4"

# Image Loading
coil = "2.7.0"

# Testing
junit5 = "5.11.3"
mockk = "1.13.13"
turbine = "1.2.0"
truth = "1.4.4"
androidx-test = "1.6.1"
compose-test = "1.7.6"
espresso = "3.6.1"
robolectric = "4.14.1"

# Tooling
detekt = "1.23.7"
ktlint = "12.1.2"
leakcanary = "2.14"
timber = "5.0.1"

[libraries]
# Kotlin
kotlinx-coroutines-core = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-core", version.ref = "kotlinx-coroutines" }
kotlinx-coroutines-android = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-android", version.ref = "kotlinx-coroutines" }
kotlinx-coroutines-test = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-test", version.ref = "kotlinx-coroutines" }
kotlinx-serialization-json = { group = "org.jetbrains.kotlinx", name = "kotlinx-serialization-json", version.ref = "kotlinx-serialization" }

# Compose
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
compose-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }
compose-runtime = { group = "androidx.compose.runtime", name = "runtime" }
compose-foundation = { group = "androidx.compose.foundation", name = "foundation" }
compose-animation = { group = "androidx.compose.animation", name = "animation" }
activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "activity-compose" }
navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "navigation-compose" }

# Lifecycle
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycle" }
androidx-lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version.ref = "lifecycle" }

# Hilt
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version.ref = "hilt-navigation-compose" }
hilt-android-testing = { group = "com.google.dagger", name = "hilt-android-testing", version.ref = "hilt" }

# Networking
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-converter-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "retrofit" }
okhttp = { group = "com.squareup.okhttp3", name = "okhttp", version.ref = "okhttp" }
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttp" }
gson = { group = "com.google.code.gson", name = "gson", version.ref = "gson" }

# Database
room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }
room-testing = { group = "androidx.room", name = "room-testing", version.ref = "room" }

# DataStore
datastore-preferences = { group = "androidx.datastore", name = "datastore-preferences", version.ref = "datastore" }
datastore-proto = { group = "androidx.datastore", name = "datastore", version.ref = "datastore" }

# Image Loading
coil-compose = { group = "io.coil-kt", name = "coil-compose", version.ref = "coil" }

# Testing
junit5 = { group = "org.junit.jupiter", name = "junit-jupiter", version.ref = "junit5" }
junit5-api = { group = "org.junit.jupiter", name = "junit-jupiter-api", version.ref = "junit5" }
junit5-params = { group = "org.junit.jupiter", name = "junit-jupiter-params", version.ref = "junit5" }
mockk = { group = "io.mockk", name = "mockk", version.ref = "mockk" }
mockk-android = { group = "io.mockk", name = "mockk-android", version.ref = "mockk" }
turbine = { group = "app.cash.turbine", name = "turbine", version.ref = "turbine" }
truth = { group = "com.google.truth", name = "truth", version.ref = "truth" }
robolectric = { group = "org.robolectric", name = "robolectric", version.ref = "robolectric" }
androidx-test-core = { group = "androidx.test", name = "core", version.ref = "androidx-test" }
androidx-test-runner = { group = "androidx.test", name = "runner", version.ref = "androidx-test" }
compose-ui-test = { group = "androidx.compose.ui", name = "ui-test-junit4" }
compose-ui-test-manifest = { group = "androidx.compose.ui", name = "ui-test-manifest" }
espresso = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "espresso" }

# Tooling
timber = { group = "com.jakewharton.timber", name = "timber", version.ref = "timber" }
leakcanary = { group = "com.squareup.leakcanary", name = "leakcanary-android", version.ref = "leakcanary" }

# Build Logic Dependencies (used in build-logic/convention)
android-gradlePlugin = { group = "com.android.tools.build", name = "gradle", version.ref = "agp" }
kotlin-gradlePlugin = { group = "org.jetbrains.kotlin", name = "kotlin-gradle-plugin", version.ref = "kotlin" }
compose-gradlePlugin = { group = "org.jetbrains.kotlin", name = "compose-compiler-gradle-plugin", version.ref = "kotlin" }
ksp-gradlePlugin = { group = "com.google.devtools.ksp", name = "com.google.devtools.ksp.gradle.plugin", version.ref = "ksp" }

[bundles]
compose = ["compose-ui", "compose-ui-tooling-preview", "compose-material3", "compose-runtime", "compose-foundation", "compose-animation"]
compose-debug = ["compose-ui-tooling", "compose-ui-test-manifest"]
networking = ["retrofit", "retrofit-converter-gson", "okhttp", "okhttp-logging"]
room = ["room-runtime", "room-ktx"]
testing = ["junit5", "mockk", "turbine", "truth", "kotlinx-coroutines-test"]
android-testing = ["mockk-android", "androidx-test-core", "androidx-test-runner", "compose-ui-test", "espresso"]

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
android-library = { id = "com.android.library", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
room = { id = "androidx.room", version.ref = "room" }
protobuf = { id = "com.google.protobuf", version.ref = "protobuf-plugin" }
```

---

## 🧩 When to Create a New Module

Creating modules has a cost (build configuration, Hilt wiring, Gradle sync time). Use these criteria to decide.

### Create a New Module When

- **Reuse:** The code will be used by two or more features or apps
- **Team Ownership:** A different team or developer will own this code
- **Build Performance:** The code changes infrequently and would benefit from caching
- **Encapsulation:** You need `internal` visibility to hide implementation details
- **Independent Testing:** The code needs its own isolated test suite
- **Distinct Lifecycle:** The code could be versioned or released independently

### Do NOT Create a New Module When

- The code is only used by a single feature and is unlikely to be shared
- The module would contain fewer than five files
- You are splitting purely for "cleanliness" without a practical benefit
- It would introduce a circular dependency that requires restructuring

### Decision Flowchart

```
Is this code used by multiple features?
├── Yes → Create a :core:* or :domain module
└── No
    └── Does this code represent a distinct feature with its own screens?
        ├── Yes → Create a :feature:* module
        └── No
            └── Is this infrastructure (network, database, etc.)?
                ├── Yes → Create a :core:* module
                └── No → Keep it in the existing module
```

---

## 🔌 Module Communication

Modules communicate through **interfaces defined in shared modules** (`:domain` or `:core:common`). Modules never directly reference concrete classes from other modules.

### Pattern: Interface in Domain, Implementation in Feature

```kotlin
// :domain/repository/UserRepository.kt
interface UserRepository {
    suspend fun getUser(id: String): Result<User>
    fun observeUser(id: String): Flow<User>
}

// :feature:profile/data/repository/UserRepositoryImpl.kt
class UserRepositoryImpl @Inject constructor(
    private val apiService: UserApiService,
    private val userDao: UserDao
) : UserRepository {
    override suspend fun getUser(id: String): Result<User> { ... }
    override fun observeUser(id: String): Flow<User> { ... }
}

// :feature:profile/data/di/ProfileDataModule.kt
@Module
@InstallIn(SingletonComponent::class)
abstract class ProfileDataModule {
    @Binds
    @Singleton
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}
```

Any feature that needs `UserRepository` receives it via Hilt injection without knowing which module provides the implementation.

### Pattern: Navigation Events for Cross-Feature Communication

Features should never call each other's screens directly. Instead, expose navigation keys and an `EntryProviderScope` extension (see [Navigation 3](../Architecture/navigation-3.md)).

```kotlin
// :feature:home/navigation/HomeEntries.kt
@Serializable
data object HomeKey : NavKey

fun EntryProviderScope<NavKey>.homeEntries(
    onNavigateToProfile: (userId: String) -> Unit,
    onNavigateToSearch: () -> Unit,
) {
    entry<HomeKey> {
        HomeRoute(
            onNavigateToProfile = onNavigateToProfile,
            onNavigateToSearch = onNavigateToSearch,
        )
    }
}

// :app/navigation/AppNavDisplay.kt — wires cross-feature navigation
val entryProvider = entryProvider<NavKey> {
    homeEntries(
        onNavigateToProfile = { userId -> navigator.navigate(ProfileKey(userId)) },
        onNavigateToSearch = { navigator.navigate(SearchKey) },
    )
    profileEntries(onBack = navigator::goBack)
    searchEntries(onBack = navigator::goBack)
}
```

### Pattern: Shared Event Bus (Use Sparingly)

For truly decoupled cross-feature events, use a shared event channel in `:core:common`.

```kotlin
// :core:common/event/AppEvent.kt
sealed interface AppEvent {
    data class UserLoggedOut(val reason: String) : AppEvent
    data object ThemeChanged : AppEvent
}

// :core:common/event/AppEventBus.kt
@Singleton
class AppEventBus @Inject constructor() {
    private val _events = MutableSharedFlow<AppEvent>(extraBufferCapacity = 10)
    val events: SharedFlow<AppEvent> = _events.asSharedFlow()

    suspend fun emit(event: AppEvent) {
        _events.emit(event)
    }
}
```

---

## 🧪 Testing in Multi-Module Projects

### Shared Test Utilities (`:core:testing`)

The `:core:testing` module provides shared test infrastructure. All feature modules include it as a `testImplementation` dependency.

```kotlin
// core/testing/build.gradle.kts
plugins {
    id("arclabs.android.library")
}

android {
    namespace = "com.arclabs.myapp.core.testing"
}

dependencies {
    api(libs.junit5)
    api(libs.mockk)
    api(libs.turbine)
    api(libs.truth)
    api(libs.kotlinx.coroutines.test)
    api(libs.hilt.android.testing)

    implementation(project(":core:common"))
}
```

Note the use of `api` instead of `implementation` — this exposes the test libraries transitively so feature modules do not need to declare them individually.

### MainDispatcherRule

```kotlin
// :core:testing/rule/MainDispatcherRule.kt
class MainDispatcherRule(
    private val dispatcher: TestDispatcher = UnconfinedTestDispatcher()
) : BeforeEachCallback, AfterEachCallback {

    override fun beforeEach(context: ExtensionContext?) {
        Dispatchers.setMain(dispatcher)
    }

    override fun afterEach(context: ExtensionContext?) {
        Dispatchers.resetMain()
    }
}
```

### Test Fakes vs Mocks

ARC Labs prefers **fakes over mocks** for repository-level testing. Fakes go in `:core:testing` when shared or in the feature's test source set when feature-specific.

```kotlin
// :core:testing/fake/FakeUserRepository.kt
class FakeUserRepository : UserRepository {

    private val users = mutableMapOf<String, User>()

    fun addUser(user: User) {
        users[user.id] = user
    }

    override suspend fun getUser(id: String): Result<User> {
        return users[id]
            ?.let { Result.Success(it) }
            ?: Result.Error(Exception("User not found"))
    }

    override fun observeUser(id: String): Flow<User> {
        return flowOf(users[id] ?: error("User not found"))
    }
}
```

### Test Structure in Feature Modules

```kotlin
// :feature:home/src/test/.../presentation/HomeViewModelTest.kt
@ExtendWith(MainDispatcherRule::class)
class HomeViewModelTest {

    private lateinit var fakeRepository: FakeHomeRepository
    private lateinit var getHomeItemsUseCase: GetHomeItemsUseCase
    private lateinit var sut: HomeViewModel

    @BeforeEach
    fun setUp() {
        fakeRepository = FakeHomeRepository()
        getHomeItemsUseCase = GetHomeItemsUseCase(fakeRepository)
        sut = HomeViewModel(getHomeItemsUseCase)
    }

    @Test
    fun `should emit loading then success when items load`() = runTest {
        // Given
        fakeRepository.addItems(listOf(testItem()))

        // When
        sut.loadItems()

        // Then
        sut.uiState.test {
            assertThat(awaitItem()).isEqualTo(HomeUiState.Loading)
            assertThat(awaitItem()).isEqualTo(HomeUiState.Success(listOf(testItem())))
        }
    }

    private fun testItem() = HomeItem(id = "1", title = "Test")
}
```

---

## 📈 Scaling Patterns: Monolith to Multi-Module

### Phase 1: Extract Core Modules

Start by extracting infrastructure that is clearly independent.

1. Create `:core:common` — move extensions, Result type, dispatchers
2. Create `:core:network` — move Retrofit setup, interceptors, API client
3. Create `:core:database` — move Room database, DAOs, migrations
4. Create `:core:ui` — move theme, shared composables, design tokens

### Phase 2: Extract Domain

1. Create `:domain` — move shared entities and repository interfaces
2. Keep implementations in `:app` for now

### Phase 3: Extract Features One by One

1. Pick the most independent feature (fewest dependencies on other features)
2. Create `:feature:x` module
3. Move presentation, domain, and data layers for that feature
4. Wire up Hilt modules
5. Update navigation in `:app`
6. Repeat for next feature

### Phase 4: Introduce Convention Plugins

Once you have three or more feature modules, the build boilerplate justifies convention plugins.

1. Create `build-logic` included build
2. Extract common plugin configurations
3. Replace verbose `build.gradle.kts` files with convention plugin IDs

### Migration Checklist

- [ ] All features compile and run after each module extraction
- [ ] Tests pass in every module independently (`./gradlew :feature:home:test`)
- [ ] No circular dependencies (run `./gradlew buildHealth` with dependency-analysis plugin)
- [ ] Hilt graph resolves correctly (`./gradlew :app:kspDebugKotlin`)
- [ ] ProGuard rules updated if needed
- [ ] CI builds each module in parallel where possible

---

## ✅ Module Structure Checklist

- [ ] Each module has a clear, single responsibility
- [ ] No circular dependencies between modules
- [ ] Feature modules do not depend on other feature modules
- [ ] Core modules do not depend on feature or app modules
- [ ] Version catalog used for all dependency versions (no hardcoded strings)
- [ ] Convention plugins used for shared build logic
- [ ] Each module has its own test suite with adequate coverage
- [ ] Shared test utilities live in `:core:testing`
- [ ] `internal` visibility used to hide implementation details
- [ ] `settings.gradle.kts` includes all modules
- [ ] Hilt modules use `@InstallIn` with appropriate component scope
- [ ] Navigation routes are defined per feature, wired in `:app`
- [ ] Shared domain entities live in `:domain`, not duplicated across features

---

## 🚫 Common Mistakes

### 1. God App Module

❌ **Putting all code in the `:app` module**

```kotlin
// app/src/main/java/com/arclabs/myapp/
// ├── ui/home/HomeScreen.kt
// ├── ui/search/SearchScreen.kt
// ├── ui/profile/ProfileScreen.kt
// ├── data/api/ApiService.kt
// ├── data/db/AppDatabase.kt
// ├── domain/model/User.kt
// ... 200+ files in one module
```

✅ **Extract features and core modules**

```kotlin
// :feature:home    — HomeScreen, HomeViewModel, HomeRepository
// :feature:search  — SearchScreen, SearchViewModel, SearchRepository
// :feature:profile — ProfileScreen, ProfileViewModel, ProfileRepository
// :core:network    — ApiService, interceptors
// :core:database   — AppDatabase, DAOs
// :domain          — User, shared entities
// :app             — Only Application, MainActivity, NavDisplay shell
```

### 2. Feature-to-Feature Dependency

❌ **Feature depending on another feature**

```kotlin
// feature/home/build.gradle.kts
dependencies {
    implementation(project(":feature:profile"))  // WRONG!
}

// feature/home/.../HomeScreen.kt
import com.arclabs.myapp.feature.profile.ProfileScreen  // WRONG!
```

✅ **Share through `:domain` or navigate via `:app`**

```kotlin
// feature/home/build.gradle.kts
dependencies {
    implementation(project(":domain"))   // Shared entities
    implementation(project(":core:ui"))  // Shared UI
    // No feature:profile dependency!
}

// Cross-feature navigation handled in :app/navigation/AppNavDisplay.kt
homeEntries(
    onNavigateToProfile = { userId -> navigator.navigate(ProfileKey(userId)) },
    onNavigateToSearch = { navigator.navigate(SearchKey) },
)
```

### 3. Leaking Implementation Details

❌ **Exposing internal classes as public**

```kotlin
// :core:network — exposing Retrofit internals
class NetworkModule {
    @Provides
    fun provideRetrofit(): Retrofit { ... }  // Retrofit leaks to all consumers
}

// :feature:home — directly using Retrofit
class HomeRepositoryImpl(
    private val retrofit: Retrofit  // Coupled to Retrofit!
) { ... }
```

✅ **Hide implementations behind interfaces**

```kotlin
// :core:network — expose only the abstraction
interface ApiClient {
    suspend fun <T> get(url: String, responseType: Class<T>): NetworkResult<T>
}

internal class RetrofitApiClient @Inject constructor(
    private val retrofit: Retrofit
) : ApiClient { ... }

// :feature:home — depends on the interface
class HomeRepositoryImpl @Inject constructor(
    private val apiClient: ApiClient  // No Retrofit knowledge!
) { ... }
```

### 4. Circular Dependencies

❌ **Two modules depending on each other**

```kotlin
// :core:network/build.gradle.kts
dependencies {
    implementation(project(":core:database"))  // network -> database
}

// :core:database/build.gradle.kts
dependencies {
    implementation(project(":core:network"))   // database -> network  CIRCULAR!
}
```

✅ **Extract shared types into `:core:common`**

```kotlin
// :core:common — owns shared types both modules need
// Result.kt, NetworkStatus.kt, etc.

// :core:network/build.gradle.kts
dependencies {
    implementation(project(":core:common"))  // network -> common
}

// :core:database/build.gradle.kts
dependencies {
    implementation(project(":core:common"))  // database -> common
}
// No circular dependency!
```

### 5. Missing `api` vs `implementation`

❌ **Using `implementation` for transitive dependencies that consumers need**

```kotlin
// :core:testing/build.gradle.kts
dependencies {
    implementation(libs.junit5)    // NOT exposed to feature tests
    implementation(libs.mockk)     // NOT exposed to feature tests
}

// :feature:home/build.gradle.kts
dependencies {
    testImplementation(project(":core:testing"))
    testImplementation(libs.junit5)    // Must redeclare!
    testImplementation(libs.mockk)     // Must redeclare!
}
```

✅ **Using `api` for dependencies that must be transitive**

```kotlin
// :core:testing/build.gradle.kts
dependencies {
    api(libs.junit5)     // Exposed transitively
    api(libs.mockk)      // Exposed transitively
    api(libs.turbine)    // Exposed transitively
}

// :feature:home/build.gradle.kts
dependencies {
    testImplementation(project(":core:testing"))
    // junit5, mockk, turbine are available automatically!
}
```

### 6. Hardcoded Versions in build.gradle.kts

❌ **Version strings scattered across module files**

```kotlin
// feature/home/build.gradle.kts
dependencies {
    implementation("com.google.dagger:hilt-android:2.51.1")       // Hardcoded!
    implementation("com.squareup.retrofit2:retrofit:2.9.0")       // Hardcoded!
}

// feature/search/build.gradle.kts
dependencies {
    implementation("com.google.dagger:hilt-android:2.48.1")       // DIFFERENT version!
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
}
```

✅ **All versions in `libs.versions.toml`**

```kotlin
// feature/home/build.gradle.kts
dependencies {
    implementation(libs.hilt.android)
    implementation(libs.retrofit)
}

// feature/search/build.gradle.kts
dependencies {
    implementation(libs.hilt.android)   // Same version, guaranteed
    implementation(libs.retrofit)
}
```

---

## 📚 Further Reading

- [Official Modularization Guide](https://developer.android.com/topic/modularization)
- [Now in Android Sample](https://github.com/android/nowinandroid)
- [Gradle Guide](../Tools/gradle.md)
- [Clean Architecture](../Architecture/clean-architecture.md)
- [Dependency Injection](../Architecture/dependency-injection.md)
- [Navigation 3](../Architecture/navigation-3.md)
- [Testing](testing.md)
- [Code Style](code-style.md)

---

**Remember**: Good modularization pays dividends in build speed, code reuse, and enforced architecture boundaries. Start modular from day one — retrofitting is always harder than building it right.
