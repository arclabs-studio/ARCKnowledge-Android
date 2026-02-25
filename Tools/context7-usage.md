# 📚 Context7 Usage Guide

**Context7 provides real-time library documentation for AI agents working on Android projects. This guide covers when to use it, how to query it effectively, and the library IDs for the ARC Labs tech stack.**

---

## 🎯 What Is Context7?

Context7 is an MCP (Model Context Protocol) server that resolves library names to documentation. Instead of relying on training data that may be outdated, agents query Context7 for **current, version-specific documentation** including:

- API reference and method signatures
- Code examples from official sources
- Migration guides between versions
- Configuration options and defaults
- Known issues and workarounds

---

## 🔍 When to Use Context7

### Always Use Context7 When:

| Scenario | Why |
|----------|-----|
| Implementing a new library feature | Get current API, not guessed signatures |
| Debugging a library-specific error | Find known issues and workarounds |
| Upgrading a dependency version | Get the migration guide for that specific version |
| Configuring library options | Get the current default values and available options |
| Writing ProGuard/R8 rules | Get the library's recommended keep rules |
| Setting up Gradle plugins | Get the current plugin ID and configuration DSL |

### Don't Use Context7 When:

| Scenario | Why |
|----------|-----|
| Writing pure Kotlin logic | Standard language features don't change often |
| Applying ARC Labs patterns | Use ARCKnowledge-Android instead |
| Making architectural decisions | Use Architecture/ docs instead |
| Writing tests (structure) | Use Quality/testing.md instead |

### Decision Rule

> **If you're about to write code that calls a library API, check Context7 first.**
> Training data may have the API wrong. Context7 has the current truth.

---

## 📖 Query Patterns for ARC Labs Tech Stack

### Jetpack Compose

```
Query: "Jetpack Compose"
Use for: Composable APIs, Modifier chains, state management, side effects,
         animation, layout, Material 3 components
```

**Example queries:**
- "How to use LaunchedEffect with a key parameter"
- "AnimatedVisibility enter and exit transitions"
- "Compose State hoisting best practices"
- "collectAsStateWithLifecycle usage"
- "Modifier.nestedScroll with TopAppBar"

### Hilt (Dagger)

```
Query: "Dagger Hilt Android"
Use for: @HiltViewModel, @AndroidEntryPoint, @Module, @InstallIn,
         @Provides, @Binds, @Singleton, testing with Hilt
```

**Example queries:**
- "@HiltViewModel with SavedStateHandle"
- "Hilt testing with @UninstallModules"
- "@Binds vs @Provides when to use each"
- "Custom Hilt scope for feature modules"
- "Hilt AssistedInject setup"

### Room

```
Query: "Android Room Database"
Use for: @Entity, @Dao, @Database, @Query, migrations,
         TypeConverters, Flow queries, testing
```

**Example queries:**
- "Room auto-migration setup"
- "Room TypeConverter for Kotlin Serialization"
- "Room database testing with in-memory database"
- "Room @Upsert annotation usage"
- "Room FTS4 full-text search setup"

### Retrofit + OkHttp

```
Query: "Retrofit" or "OkHttp"
Use for: API service interfaces, interceptors, call adapters,
         converter factories, timeout configuration
```

**Example queries:**
- "Retrofit with Kotlin Serialization converter"
- "OkHttp interceptor for authentication token refresh"
- "Retrofit suspend function return types"
- "OkHttp certificate pinning setup"

### Kotlin Coroutines + Flow

```
Query: "Kotlin Coroutines" or "Kotlin Flow"
Use for: suspend functions, Flow operators, StateFlow, SharedFlow,
         structured concurrency, dispatchers, testing
```

**Example queries:**
- "StateFlow vs SharedFlow differences"
- "Flow.combine with multiple sources"
- "Coroutine exception handling with SupervisorJob"
- "Testing coroutines with runTest and advanceUntilIdle"

### Navigation Compose

```
Query: "Jetpack Navigation Compose"
Use for: NavHost, NavController, type-safe arguments,
         deep links, nested graphs, bottom navigation
```

**Example queries:**
- "Type-safe navigation with Kotlin Serialization routes"
- "Nested navigation graphs in Compose"
- "Deep link handling in Navigation Compose"
- "Bottom navigation with Navigation Compose"

### Coil

```
Query: "Coil Image Loading"
Use for: AsyncImage, ImageRequest, transformations,
         caching, placeholder/error handling
```

**Example queries:**
- "Coil AsyncImage with placeholder and error"
- "Coil disk cache configuration"
- "Coil image transformations (circle crop, blur)"

### DataStore

```
Query: "Android DataStore"
Use for: Preferences DataStore, Proto DataStore,
         migration from SharedPreferences
```

**Example queries:**
- "DataStore Preferences read and write"
- "Proto DataStore with Kotlin Serialization"
- "Migrate from SharedPreferences to DataStore"

### JUnit 5

```
Query: "JUnit 5" or "JUnit Jupiter"
Use for: @Test, @Nested, @DisplayName, @BeforeEach,
         @ParameterizedTest, assertions, lifecycle
```

**Example queries:**
- "JUnit 5 @Nested test classes"
- "JUnit 5 @ParameterizedTest with @MethodSource"
- "JUnit 5 assertThrows for exception testing"

### MockK

```
Query: "MockK"
Use for: mockk(), every {}, verify {}, coEvery {},
         coVerify {}, slot, capture, relaxed mocks
```

**Example queries:**
- "MockK coEvery and coVerify for suspend functions"
- "MockK verify call order"
- "MockK slot and capture for argument inspection"

### Turbine

```
Query: "Turbine Flow Testing"
Use for: Flow testing, awaitItem(), awaitComplete(),
         awaitError(), expectNoEvents()
```

**Example queries:**
- "Turbine test StateFlow emissions"
- "Turbine awaitItem with timeout"
- "Turbine testing multiple Flow emissions"

---

## 🔄 Workflow: Using Context7 in Practice

### Step 1: Identify the Library

Before implementing, identify which libraries you'll call:

```
Task: "Add restaurant search with debounce"
Libraries involved:
  - Jetpack Compose (SearchBar component)
  - Kotlin Flow (debounce operator)
  - Room (search query)
  - Hilt (DI wiring)
```

### Step 2: Resolve Library IDs

Use Context7's `resolve-library-id` tool:

```
resolve-library-id("Jetpack Compose")
resolve-library-id("Kotlin Flow")
resolve-library-id("Android Room")
```

### Step 3: Fetch Relevant Documentation

Use `get-library-docs` with the resolved ID and a focused topic:

```
get-library-docs(id: "<compose-id>", topic: "SearchBar component")
get-library-docs(id: "<flow-id>", topic: "debounce operator")
get-library-docs(id: "<room-id>", topic: "LIKE query with Flow return type")
```

### Step 4: Implement with Confidence

Now you have current API signatures, correct parameter names, and official examples. Implement the feature knowing the code will compile.

---

## 📐 Version Verification Pattern

When dependency versions matter (e.g., for libs.versions.toml), use Context7 to verify the latest stable version:

```
Query: "What is the latest stable version of [library]?"

Libraries to verify:
- Kotlin: 2.0.21  # verify latest
- Compose BOM: 2024.12.01  # verify latest
- Hilt: 2.51.1  # verify latest
- Room: 2.6.1  # verify latest
- Retrofit: 2.9.0  # verify latest
- OkHttp: 4.12.0  # verify latest
- Coil: 2.7.0  # verify latest
- Navigation Compose: 2.8.5  # verify latest
- Kotlinx Serialization: 1.7.3  # verify latest
- Kotlinx Coroutines: 1.9.0  # verify latest
- JUnit 5: 5.10.3  # verify latest
- MockK: 1.13.13  # verify latest
- Turbine: 1.2.0  # verify latest
```

> **Note**: Versions marked `# verify latest` should be checked against Context7
> or Maven Central before updating libs.versions.toml.

---

## ✅ Context7 Checklist

- [ ] Context7 MCP is installed and responding (see [MCP Setup](./mcp-setup.md))
- [ ] Library IDs resolved for the ARC Labs tech stack
- [ ] Documentation fetched before implementing library-specific code
- [ ] Version numbers verified against latest stable releases
- [ ] Migration guides consulted when upgrading dependencies

---

## ❌ Common Mistakes

### 1. Not Querying Before Implementing

```kotlin
// ❌ Agent guesses API from training data
@Composable
fun SearchBar(query: String, onQueryChange: (String) -> Unit) {
    TextField(value = query, onValueChange = onQueryChange) // Wrong component
}

// ✅ Agent queries Context7 first, gets current Material3 SearchBar API
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun SearchBar(
    query: String,
    onQueryChange: (String) -> Unit,
    onSearch: (String) -> Unit,
    active: Boolean,
    onActiveChange: (Boolean) -> Unit,
) {
    SearchBar(
        inputField = {
            SearchBarDefaults.InputField(
                query = query,
                onQueryChange = onQueryChange,
                onSearch = onSearch,
                expanded = active,
                onExpandedChange = onActiveChange,
            )
        },
        expanded = active,
        onExpandedChange = onActiveChange,
    ) {
        // Search suggestions
    }
}
```

### 2. Using Outdated API Signatures

```kotlin
// ❌ Old Navigation Compose API (pre-2.8.0)
composable("profile/{userId}") { backStackEntry ->
    val userId = backStackEntry.arguments?.getString("userId")
    ProfileScreen(userId = userId ?: "")
}

// ✅ Current type-safe Navigation Compose API (2.8.0+)
@Serializable
data class ProfileRoute(val userId: String)

composable<ProfileRoute> { backStackEntry ->
    val route = backStackEntry.toRoute<ProfileRoute>()
    ProfileScreen(userId = route.userId)
}
```

### 3. Mixing Library Versions

```toml
# ❌ Mismatched versions that may cause runtime crashes
[versions]
compose-bom = "2024.01.00"  # January 2024
lifecycle = "2.8.7"          # December 2024 -- may require newer Compose

# ✅ Compatible version set verified via Context7
[versions]
compose-bom = "2024.12.01"  # December 2024 # verify latest
lifecycle = "2.8.7"          # Compatible with December BOM # verify latest
```

---

## 🌐 Android Developers Official Documentation

Context7 covers **library-level** documentation (API references, code examples). For **platform-level** guidance — architecture patterns, design guidelines, quality standards, device-specific development — use the official Android Developers site directly.

### Site Structure

The Android Developers site is organized into three main portals:

| Portal | URL | Purpose |
|--------|-----|---------|
| **Get Started** | `developer.android.com/get-started` | Onboarding, training courses, tutorials |
| **Design & Plan** | `developer.android.com/design` | UI design, architecture, quality, security |
| **Develop** | `developer.android.com/develop` | Core APIs, tools, libraries, device tech |

### Get Started (`/get-started`)

Use for onboarding, learning paths, and platform overview.

| Section | URL | When to Use |
|---------|-----|-------------|
| Hello World | `/get-started/overview` | Setting up first Android project |
| Training Courses | `/courses` | Structured learning paths for Android |
| Tutorials / Codelabs | `/get-started/codelabs` | Step-by-step guided exercises |
| Compose for Teams | `/develop/ui/compose/adopt` | Compose adoption strategy for existing projects |
| Kotlin for Android | `/kotlin` | Kotlin language features for Android |
| Latest Updates | `/latest-updates` | What's new in the latest Android release |
| Jetpack & Compose Releases | `/jetpack/androidx/versions` | Version history of AndroidX libraries |

### Design & Plan (`/design`)

Use for architecture decisions, UI patterns, quality standards, and security.

| Section | URL | When to Use |
|---------|-----|-------------|
| **UI Design** | | |
| Design for Android | `/design/ui` | Mobile design principles and patterns |
| Mobile | `/design/ui/mobile` | Phone-specific design guidelines |
| Adaptive UI | `/design/ui/large-screens` | Tablet, foldable, and desktop layouts |
| Widgets | `/design/ui/widget` | Home screen widget design |
| **Architecture** | | |
| Architecture Guide | `/topic/architecture/intro` | Official architecture recommendations |
| Navigation Principles | `/guide/navigation/navigation-principles` | Navigation design patterns |
| Modularization | `/topic/modularization` | Multi-module project structure |
| Testing Fundamentals | `/training/testing/fundamentals` | Test strategy and pyramid |
| **Quality** | | |
| Quality Overview | `/quality` | App quality dimensions and standards |
| Accessibility | `/guide/topics/ui/accessibility` | Making apps accessible to all users |
| Technical Quality | `/quality/technical` | Performance, stability, security |
| **Security** | | |
| Security Overview | `/security` | Security best practices |
| Privacy | `/privacy` | Privacy guidelines and permissions |
| Permissions | `/privacy#app-permissions` | Runtime permissions handling |

### Develop (`/develop`)

Use for implementation guidance, API references, and tooling.

| Section | URL | When to Use |
|---------|-----|-------------|
| **Core Areas** | | |
| User Interfaces | `/develop/ui` | Compose, Views, layouts, theming |
| Background Work | `/develop/background-work` | WorkManager, services, coroutines |
| Data and Files | `/guide/topics/data` | Storage, databases, content providers |
| Connectivity | `/develop/connectivity` | Network, Bluetooth, NFC |
| Samples | `/samples` | Official sample code projects |
| **Tools & Workflow** | | |
| Android Studio | `/studio/write` | IDE features, debugging, profiling |
| Build Projects | `/build/gradle-build-overview` | Gradle configuration, build variants |
| Testing | `/training/testing` | Testing tools and strategies |
| Performance | `/topic/performance/overview` | Performance profiling and optimization |
| Command-line Tools | `/tools` | adb, emulator, sdkmanager |
| **Libraries** | | |
| Jetpack Libraries | `/jetpack/androidx/explorer` | Browse all AndroidX libraries |
| Compose Libraries | `/jetpack/androidx/releases/compose` | Compose release notes and versions |
| API Reference | `/reference/packages` | Full Android API reference |
| **Device Tech** | | |
| Adaptive UI | `/guide/topics/large-screens/get-started-with-large-screens` | Large screen development |
| Wear OS | `/training/wearables` | Wearable app development |
| Android for Cars | `/training/cars` | Android Auto and Automotive OS |
| Android TV | `/training/tv` | TV app development |

### When to Use Android Developers vs. Context7

| Question | Source |
|----------|--------|
| "What parameters does `LazyColumn` accept?" | **Context7** (API reference) |
| "How should I structure navigation in a multi-module app?" | **Android Developers** (Architecture guide) |
| "What's the latest Room version?" | **Context7** (library docs) |
| "How do I implement edge-to-edge on Android 15?" | **Android Developers** (platform guide) |
| "What's the correct Hilt annotation for a ViewModel?" | **Context7** (library docs) |
| "How should I handle runtime permissions?" | **Android Developers** (privacy guide) |
| "What Compose BOM includes which library versions?" | **Context7** (release docs) |
| "How do I make my app accessible with TalkBack?" | **Android Developers** (accessibility guide) |
| "What's the recommended app architecture?" | **Android Developers** (architecture intro) |
| "How does `StateFlow.collectAsStateWithLifecycle` work?" | **Context7** (API reference) |

### Web Search Pattern

When the agent needs information from Android Developers, use this pattern:

```
Search: "site:developer.android.com <topic>"

Examples:
- "site:developer.android.com room migration"
- "site:developer.android.com compose performance"
- "site:developer.android.com navigation type safe"
- "site:developer.android.com workmanager constraints"
```

Or fetch directly when you know the URL:

```
Fetch: https://developer.android.com/topic/architecture/intro
Fetch: https://developer.android.com/develop/ui/compose/state
Fetch: https://developer.android.com/training/testing/fundamentals
```

---

## 📚 Further Reading

- [MCP Setup Guide](./mcp-setup.md)
- [Gradle Version Catalogs](./gradle.md)
- [Skills Index](../Skills/skills-index.md)
- [Context7 Documentation](https://context7.com)
- [Android Developers](https://developer.android.com)
- [Jetpack Library Explorer](https://developer.android.com/jetpack/androidx/explorer)
- [Compose Release Notes](https://developer.android.com/jetpack/androidx/releases/compose)

---

**Remember**: Context7 for library APIs, Android Developers for platform guidance. When in doubt, check both — five seconds of lookup prevents five hours of debugging.
