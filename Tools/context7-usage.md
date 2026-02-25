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

## 📚 Further Reading

- [MCP Setup Guide](./mcp-setup.md)
- [Gradle Version Catalogs](./gradle.md)
- [Skills Index](../Skills/skills-index.md)
- [Context7 Documentation](https://context7.com)

---

**Remember**: When in doubt, query Context7. Five seconds of documentation lookup prevents five hours of debugging a wrong API call.
