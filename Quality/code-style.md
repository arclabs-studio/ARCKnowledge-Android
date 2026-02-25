# 🎨 Code Style

**Comprehensive Kotlin and Android code style guide for ARC Labs Studio. This document defines formatting rules, naming conventions, tooling configuration, and idiomatic Kotlin patterns that every team member must follow. Consistency across the codebase is non-negotiable.**

---

## Table of Contents

1. [Style Philosophy](#-style-philosophy)
2. [File Structure](#-file-structure)
3. [Formatting Rules](#-formatting-rules)
4. [Naming Conventions](#-naming-conventions)
5. [ktlint Configuration](#-ktlint-configuration)
6. [detekt Configuration](#-detekt-configuration)
7. [Import Conventions](#-import-conventions)
8. [Kotlin Idioms](#-kotlin-idioms)
9. [Code Style Checklist](#-code-style-checklist)
10. [Common Violations](#-common-violations)
11. [Further Reading](#-further-reading)

---

## 🎯 Style Philosophy

**Code style is not about personal preference. It is about team productivity, readability, and long-term maintainability. Every line of code is read far more often than it is written.**

---

### Core Principles

1. **Consistency over personal preference** -- The team's style guide always wins over individual taste. If the guide says trailing commas, you use trailing commas.

2. **Automate enforcement** -- ktlint handles formatting, detekt catches code smells. Human reviewers should not waste time on style issues that tools can catch.

3. **Official Kotlin style guide as base** -- We adopt the [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html) as our foundation, with ARC Labs-specific extensions documented here.

4. **Readability first** -- When in doubt, choose the option that makes the code easier to read and understand at a glance.

5. **Minimal ceremony** -- Kotlin provides concise syntax for a reason. Use it. Avoid boilerplate and unnecessary verbosity.

### Style Enforcement Pipeline

```
Developer writes code
        |
        v
  ktlint --format (auto-fix formatting)
        |
        v
  detekt (analyze code smells)
        |
        v
  Pre-commit hook (block non-compliant code)
        |
        v
  CI pipeline (final gate)
```

### Non-Negotiable Rules

- All code MUST pass ktlint before merge
- All code MUST pass detekt with zero errors before merge
- Style violations in pull requests MUST be resolved before approval
- No suppression annotations without a code review comment explaining why

---

## 📐 File Structure

**Every Kotlin file follows a strict ordering. This makes it trivial to navigate unfamiliar code because you always know where to look.**

---

### Standard File Layout

```kotlin
// 1. Copyright header (if applicable)
/*
 * Copyright 2026 ARC Labs Studio
 * SPDX-License-Identifier: Apache-2.0
 */

// 2. Package declaration
package com.arclabs.app.feature.home

// 3. Imports (sorted, no wildcards)
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import javax.inject.Inject

// 4. Top-level declarations (one primary class/interface per file)
```

### ViewModel File Example

```kotlin
package com.arclabs.app.feature.home

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.arclabs.app.domain.model.Restaurant
import com.arclabs.app.domain.usecase.GetRestaurantsUseCase
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import javax.inject.Inject

/**
 * ViewModel responsible for managing the home screen state.
 *
 * Loads and displays a list of nearby restaurants based on
 * the user's current location preferences.
 */
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getRestaurantsUseCase: GetRestaurantsUseCase,
) : ViewModel() {

    // region Public Properties

    private val _uiState = MutableStateFlow<HomeUiState>(HomeUiState.Loading)
    val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()

    // endregion

    // region Lifecycle

    init {
        loadRestaurants()
    }

    // endregion

    // region Public Functions

    fun loadRestaurants() {
        viewModelScope.launch {
            _uiState.value = HomeUiState.Loading
            getRestaurantsUseCase()
                .onSuccess { restaurants ->
                    _uiState.value = HomeUiState.Success(restaurants)
                }
                .onFailure { error ->
                    handleError(error)
                }
        }
    }

    fun onRestaurantClicked(restaurant: Restaurant) {
        // Navigation handled by UI layer
    }

    // endregion

    // region Private Functions

    private fun handleError(error: Throwable) {
        _uiState.value = HomeUiState.Error(
            message = error.localizedMessage ?: "An unexpected error occurred",
        )
    }

    // endregion
}
```

### Region Ordering Within a Class

```kotlin
class MyClass {

    // region Constants / Companion Object
    companion object {
        private const val TAG = "MyClass"
    }
    // endregion

    // region Public Properties
    // endregion

    // region Private Properties
    // endregion

    // region Lifecycle / Init
    // endregion

    // region Public Functions
    // endregion

    // region Internal Functions
    // endregion

    // region Private Functions
    // endregion

    // region Nested Types / Sealed Classes
    // endregion
}
```

### Sealed Class / UI State File Example

```kotlin
package com.arclabs.app.feature.home

import com.arclabs.app.domain.model.Restaurant

/**
 * Represents the possible UI states for the home screen.
 */
sealed interface HomeUiState {

    /** Initial loading state. */
    data object Loading : HomeUiState

    /** Successfully loaded restaurants. */
    data class Success(
        val restaurants: List<Restaurant>,
    ) : HomeUiState

    /** An error occurred while loading. */
    data class Error(
        val message: String,
    ) : HomeUiState
}
```

### Composable File Example

```kotlin
package com.arclabs.app.feature.home.ui

import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.hilt.navigation.compose.hiltViewModel
import com.arclabs.app.feature.home.HomeUiState
import com.arclabs.app.feature.home.HomeViewModel

/**
 * Home screen displaying the list of nearby restaurants.
 */
@Composable
fun HomeScreen(
    viewModel: HomeViewModel = hiltViewModel(),
    onNavigateToDetail: (String) -> Unit,
) {
    val uiState by viewModel.uiState.collectAsState()

    HomeScreenContent(
        uiState = uiState,
        onRestaurantClicked = { restaurant ->
            onNavigateToDetail(restaurant.id)
        },
        onRetryClicked = viewModel::loadRestaurants,
    )
}

@Composable
private fun HomeScreenContent(
    uiState: HomeUiState,
    onRestaurantClicked: (Restaurant) -> Unit,
    onRetryClicked: () -> Unit,
) {
    Scaffold { paddingValues ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues),
        ) {
            when (uiState) {
                is HomeUiState.Loading -> CircularProgressIndicator()
                is HomeUiState.Success -> RestaurantList(
                    restaurants = uiState.restaurants,
                    onItemClicked = onRestaurantClicked,
                )
                is HomeUiState.Error -> ErrorContent(
                    message = uiState.message,
                    onRetryClicked = onRetryClicked,
                )
            }
        }
    }
}

@Preview(showBackground = true)
@Composable
private fun HomeScreenPreview() {
    HomeScreenContent(
        uiState = HomeUiState.Success(
            restaurants = listOf(/* preview data */),
        ),
        onRestaurantClicked = {},
        onRetryClicked = {},
    )
}
```

### File Naming Rules

| Type | Naming | Example |
|------|--------|---------|
| ViewModel | `{Feature}ViewModel.kt` | `HomeViewModel.kt` |
| UI State | `{Feature}UiState.kt` | `HomeUiState.kt` |
| Screen Composable | `{Feature}Screen.kt` | `HomeScreen.kt` |
| Repository Interface | `{Entity}Repository.kt` | `RestaurantRepository.kt` |
| Repository Impl | `{Entity}RepositoryImpl.kt` | `RestaurantRepositoryImpl.kt` |
| Use Case | `{Action}{Entity}UseCase.kt` | `GetRestaurantsUseCase.kt` |
| DI Module | `{Feature}Module.kt` | `HomeModule.kt` |
| Extension | `{Type}Extensions.kt` | `StringExtensions.kt` |

---

## 📏 Formatting Rules

**Automated formatting eliminates style debates in code review. Configure your IDE and CI pipeline to enforce these rules automatically.**

---

### Indentation

- **4 spaces** for indentation (never tabs)
- Continuation indent: **8 spaces**

```kotlin
// ✅ Correct: 4-space indentation
fun processRestaurants(
    restaurants: List<Restaurant>,
    filter: RestaurantFilter,
): List<Restaurant> {
    return restaurants
        .filter { it.rating >= filter.minRating }
        .sortedByDescending { it.rating }
}

// ❌ Wrong: 2-space indentation
fun processRestaurants(
  restaurants: List<Restaurant>,
  filter: RestaurantFilter,
): List<Restaurant> {
  return restaurants
    .filter { it.rating >= filter.minRating }
    .sortedByDescending { it.rating }
}
```

### Line Length

- **120 characters** maximum per line
- Break long lines at logical points

```kotlin
// ✅ Correct: line broken at a logical point
val filteredRestaurants = restaurants
    .filter { it.cuisine == selectedCuisine }
    .filter { it.priceRange <= maxPrice }
    .sortedByDescending { it.averageRating }

// ❌ Wrong: exceeds 120 characters on a single line
val filteredRestaurants = restaurants.filter { it.cuisine == selectedCuisine }.filter { it.priceRange <= maxPrice }.sortedByDescending { it.averageRating }
```

### Trailing Commas

**Always use trailing commas in multi-line declarations.** This produces cleaner diffs and makes reordering easier.

```kotlin
// ✅ Correct: trailing commas
data class Restaurant(
    val id: String,
    val name: String,
    val cuisine: CuisineType,
    val rating: Double,
    val priceRange: Int,
)

enum class CuisineType {
    ITALIAN,
    JAPANESE,
    MEXICAN,
    THAI,
    AMERICAN,
}

fun createRestaurant(
    name: String,
    cuisine: CuisineType,
    rating: Double,
) {
    // ...
}

// ❌ Wrong: missing trailing commas
data class Restaurant(
    val id: String,
    val name: String,
    val cuisine: CuisineType,
    val rating: Double,
    val priceRange: Int
)
```

### Brace Style (K&R / Egyptian Brackets)

Opening braces go on the same line as the declaration. Closing braces go on their own line.

```kotlin
// ✅ Correct: K&R style
class HomeViewModel @Inject constructor(
    private val useCase: GetRestaurantsUseCase,
) : ViewModel() {

    fun loadData() {
        viewModelScope.launch {
            // ...
        }
    }
}

// ❌ Wrong: Allman style (opening brace on new line)
class HomeViewModel @Inject constructor(
    private val useCase: GetRestaurantsUseCase,
) : ViewModel()
{
    fun loadData()
    {
        viewModelScope.launch
        {
            // ...
        }
    }
}
```

### Blank Lines

- **One blank line** between top-level declarations (classes, functions, properties)
- **One blank line** between member functions
- **No blank line** after an opening brace
- **No blank line** before a closing brace
- **One blank line** before a `return` statement in long functions (optional but encouraged)

```kotlin
// ✅ Correct: blank line usage
class UserRepository(
    private val apiService: ApiService,
    private val userDao: UserDao,
) {

    suspend fun getUser(id: String): User {
        return apiService.fetchUser(id)
    }

    suspend fun saveUser(user: User) {
        userDao.insert(user.toEntity())
    }
}

// ❌ Wrong: extra blank lines
class UserRepository(
    private val apiService: ApiService,
    private val userDao: UserDao,
) {

    suspend fun getUser(id: String): User {

        return apiService.fetchUser(id)

    }

}
```

### Multi-Line Function Signatures

When a function signature exceeds 120 characters, break parameters onto individual lines.

```kotlin
// ✅ Correct: each parameter on its own line
fun createRestaurantReview(
    restaurantId: String,
    userId: String,
    rating: Double,
    comment: String,
    visitDate: LocalDate,
): Result<Review> {
    // ...
}

// ❌ Wrong: parameters crammed onto one long line
fun createRestaurantReview(restaurantId: String, userId: String, rating: Double, comment: String, visitDate: LocalDate): Result<Review> {
    // ...
}
```

### Expression Wrapping

```kotlin
// ✅ Correct: chained calls wrapped with leading dot
val topRatedRestaurants = restaurants
    .filter { it.isOpen }
    .filter { it.rating >= 4.0 }
    .sortedByDescending { it.rating }
    .take(10)

// ✅ Correct: when expression on multiple lines
val label = when (status) {
    Status.OPEN -> "Open Now"
    Status.CLOSED -> "Closed"
    Status.CLOSING_SOON -> "Closing Soon"
}

// ✅ Correct: string template wrapping
val message = "Restaurant ${restaurant.name} has a rating of " +
    "${restaurant.rating} with ${restaurant.reviewCount} reviews"
```

---

## 📝 Naming Conventions

**Names are the most important form of documentation. A well-chosen name eliminates the need for a comment.**

---

### Full Reference Table

| Type | Convention | Example |
|------|-----------|---------|
| Package | `lowercase.dots` | `com.arclabs.app.feature` |
| Class / Interface | `PascalCase` | `UserRepository` |
| Abstract Class | `PascalCase` (no prefix) | `BaseViewModel` |
| Object | `PascalCase` | `NetworkModule` |
| Function | `camelCase` | `getUserProfile` |
| Property | `camelCase` | `isLoading` |
| Local Variable | `camelCase` | `restaurantList` |
| Constant (`const val`) | `UPPER_SNAKE_CASE` | `MAX_RETRIES` |
| Companion `val` (non-const) | `PascalCase` or `camelCase` | `DefaultTimeout` |
| Enum Value | `UPPER_SNAKE_CASE` | `LOADING` |
| Type Parameter | Single uppercase letter or `PascalCase` | `T`, `Key`, `Value` |
| Composable Function | `PascalCase` | `UserProfileScreen` |
| Backing Property | `_camelCase` | `_uiState` |
| Boolean Property | `is`/`has`/`should`/`can` prefix | `isActive`, `hasPermission` |
| Event Handler | `on{Event}` | `onSaveClicked` |
| Factory Function | `create{Type}` or matches type name | `createUser()` |
| Extension File | `{Type}Extensions.kt` | `StringExtensions.kt` |
| Test Class | `{Class}Test` | `HomeViewModelTest` |
| Test Function | backtick descriptive name | `` `should load restaurants when initialized` `` |

### Package Naming

```kotlin
// ✅ Correct: lowercase, dot-separated, feature-based
package com.arclabs.app.feature.home
package com.arclabs.app.data.repository
package com.arclabs.app.domain.usecase
package com.arclabs.app.core.network

// ❌ Wrong: uppercase, underscores, camelCase
package com.arclabs.app.Feature.Home
package com.arclabs.app.data_repository
package com.arclabs.app.domainUseCase
```

### Class and Interface Naming

```kotlin
// ✅ Correct: clear, descriptive PascalCase
interface RestaurantRepository
class RestaurantRepositoryImpl : RestaurantRepository
class GetRestaurantsUseCase
class HomeViewModel
sealed interface HomeUiState

// ❌ Wrong: prefixes, abbreviations, unclear names
interface IRestaurantRepository     // No "I" prefix
class RestaurantRepoImpl            // Don't abbreviate
class GetRestUC                     // Don't abbreviate
class HVM                           // Don't abbreviate
```

### Function Naming

```kotlin
// ✅ Correct: verb-first, descriptive
fun getRestaurantById(id: String): Restaurant
fun saveUserPreferences(preferences: UserPreferences)
fun calculateTotalPrice(items: List<OrderItem>): Double
fun isRestaurantOpen(restaurant: Restaurant): Boolean
suspend fun fetchNearbyRestaurants(location: Location): List<Restaurant>

// ❌ Wrong: noun-first, vague, abbreviated
fun restaurant(id: String): Restaurant          // Missing verb
fun prefs(preferences: UserPreferences)          // Abbreviated
fun calc(items: List<OrderItem>): Double         // Abbreviated
fun open(restaurant: Restaurant): Boolean        // Ambiguous
```

### Boolean Naming

```kotlin
// ✅ Correct: reads like a question
val isLoading: Boolean
val hasPermission: Boolean
val shouldRetry: Boolean
val canEdit: Boolean
val isRestaurantOpen: Boolean

// ❌ Wrong: does not read like a question
val loading: Boolean
val permission: Boolean
val retry: Boolean
val edit: Boolean
val restaurantOpen: Boolean
```

### Event Handler Naming

```kotlin
// ✅ Correct: on{Subject}{Action} pattern
fun onSaveClicked()
fun onRestaurantSelected(restaurant: Restaurant)
fun onSearchQueryChanged(query: String)
fun onPermissionGranted(permission: String)
fun onBackPressed()

// ❌ Wrong: inconsistent patterns
fun saveClick()
fun selectRestaurant(restaurant: Restaurant)
fun searchChanged(query: String)
fun permissionResult(permission: String)
fun goBack()
```

### Constant Naming

```kotlin
// ✅ Correct: UPPER_SNAKE_CASE for compile-time constants
companion object {
    const val MAX_RETRIES = 3
    const val BASE_URL = "https://api.arclabs.com"
    const val ANIMATION_DURATION_MS = 300L
    const val DEFAULT_PAGE_SIZE = 20
}

// ✅ Correct: PascalCase or camelCase for non-const companion vals
companion object {
    val DefaultPadding = 16.dp
    val defaultTimeout = 30.seconds
}

// ❌ Wrong: mixed conventions
companion object {
    const val maxRetries = 3               // Should be UPPER_SNAKE
    val DEFAULT_PADDING = 16.dp            // Non-const, use PascalCase
}
```

### Composable Naming

```kotlin
// ✅ Correct: PascalCase, noun-like for UI elements
@Composable
fun RestaurantCard(
    restaurant: Restaurant,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
) { /* ... */ }

@Composable
fun HomeScreen(
    viewModel: HomeViewModel = hiltViewModel(),
) { /* ... */ }

@Composable
fun ErrorContent(
    message: String,
    onRetryClicked: () -> Unit,
) { /* ... */ }

// ❌ Wrong: camelCase, verb-like naming
@Composable
fun showRestaurantCard(restaurant: Restaurant) { /* ... */ }

@Composable
fun renderHomeScreen() { /* ... */ }
```

---

## 🔧 ktlint Configuration

**ktlint is our automated code formatter. It enforces consistent formatting without manual effort. Every developer must integrate ktlint into their local workflow.**

---

### .editorconfig

Place this file at the project root:

```editorconfig
# .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 4
insert_final_newline = true
trim_trailing_whitespace = true
max_line_length = 120

[*.{kt,kts}]
ktlint_code_style = android_studio

# Trailing commas
ktlint_standard_trailing-comma-on-call-site = enabled
ktlint_standard_trailing-comma-on-declaration-site = enabled

# Indentation
ktlint_standard_indent = enabled

# Import ordering
ktlint_standard_import-ordering = enabled

# No wildcard imports
ktlint_standard_no-wildcard-imports = enabled

# Spacing
ktlint_standard_no-multi-spaces = enabled
ktlint_standard_no-trailing-spaces = enabled
ktlint_standard_no-blank-line-before-rbrace = enabled
ktlint_standard_no-consecutive-blank-lines = enabled

# Function signature
ktlint_standard_function-signature = enabled

# Wrapping
ktlint_standard_wrapping = enabled
ktlint_standard_argument-list-wrapping = enabled
ktlint_standard_parameter-list-wrapping = enabled

# Naming (enabled by default)
ktlint_standard_class-naming = enabled
ktlint_standard_function-naming = enabled
ktlint_standard_package-name = enabled
ktlint_standard_property-naming = enabled

# Annotation formatting
ktlint_standard_annotation = enabled

# Multiline expressions
ktlint_standard_multiline-expression-wrapping = enabled

# Disabled rules (if needed)
# ktlint_standard_filename = disabled

[*.md]
trim_trailing_whitespace = false

[*.{yml,yaml}]
indent_size = 2
```

### Gradle Integration (build.gradle.kts)

```kotlin
// build.gradle.kts (project-level)
plugins {
    id("org.jlleitschuh.gradle.ktlint") version "12.1.2"
}

// Configure ktlint
ktlint {
    version.set("1.5.0")
    android.set(true)
    verbose.set(true)
    outputToConsole.set(true)
    enableExperimentalRules.set(true)
    filter {
        exclude("**/generated/**")
        exclude("**/build/**")
    }
}
```

### Common ktlint Commands

```bash
# Check formatting (CI / pre-commit)
./gradlew ktlintCheck

# Auto-fix formatting issues
./gradlew ktlintFormat

# Check a specific module
./gradlew :app:ktlintCheck
./gradlew :feature:home:ktlintCheck
```

### Pre-Commit Hook Setup

```bash
#!/bin/sh
# .git/hooks/pre-commit

echo "Running ktlint..."
./gradlew ktlintCheck --daemon

if [ $? -ne 0 ]; then
    echo ""
    echo "ktlint check failed. Run './gradlew ktlintFormat' to auto-fix."
    echo "Then review changes and commit again."
    exit 1
fi
```

---

## 🔍 detekt Configuration

**detekt performs static analysis beyond formatting. It catches code smells, complexity issues, and potential bugs. Where ktlint cares about how code looks, detekt cares about how code behaves.**

---

### detekt.yml

```yaml
# detekt.yml

build:
  maxIssues: 0  # Zero tolerance for detekt issues

complexity:
  ComplexCondition:
    active: true
    threshold: 4
  ComplexMethod:
    active: true
    threshold: 15
    ignoreSingleWhenExpression: true
    ignoreSimpleWhenEntries: true
  LargeClass:
    active: true
    threshold: 600
  LongMethod:
    active: true
    threshold: 60
  LongParameterList:
    active: true
    functionThreshold: 6
    constructorThreshold: 8
    ignoreAnnotated:
      - "Composable"
      - "Inject"
  NestedBlockDepth:
    active: true
    threshold: 4
  TooManyFunctions:
    active: true
    thresholdInFiles: 15
    thresholdInClasses: 15
    thresholdInInterfaces: 10
    thresholdInObjects: 10
    thresholdInEnums: 5
    ignoreAnnotated:
      - "Composable"

naming:
  ClassNaming:
    active: true
    classPattern: "[A-Z][a-zA-Z0-9]*"
  FunctionNaming:
    active: true
    functionPattern: "[a-z][a-zA-Z0-9]*"
    ignoreAnnotated:
      - "Composable"
      - "Test"
  PackageNaming:
    active: true
    packagePattern: "[a-z]+(\\.[a-z][A-Za-z0-9]*)*"
  TopLevelPropertyNaming:
    active: true
    constantPattern: "[A-Z][A-Za-z0-9]*|[A-Z_][A-Z0-9_]*"
  VariableNaming:
    active: true
    variablePattern: "[a-z][a-zA-Z0-9]*"
  BooleanPropertyNaming:
    active: true
    allowedPattern: "^(is|has|should|can|was|will|are|do|does|need)"
  MatchingDeclarationName:
    active: true
    mustBeFirst: true

style:
  ForbiddenComment:
    active: true
    values:
      - "TODO:"
      - "FIXME:"
      - "HACK:"
    allowedPatterns: "TODO\\(\\w+\\)"  # Allow TODO(username)
  MagicNumber:
    active: true
    ignoreNumbers:
      - "-1"
      - "0"
      - "1"
      - "2"
    ignoreAnnotated:
      - "Composable"
      - "Preview"
  MaxLineLength:
    active: true
    maxLineLength: 120
    excludeCommentStatements: true
  ReturnCount:
    active: true
    max: 3
    excludeGuardClauses: true
  UnusedImports:
    active: true
  UnusedPrivateMember:
    active: true
    allowedNames: "_.*"
  WildcardImport:
    active: true
    excludeImports:
      - "kotlinx.coroutines.flow.*"
  ForbiddenMethodCall:
    active: true
    methods:
      - reason: "Use Timber instead of println"
        value: "kotlin.io.println"
      - reason: "Use Timber instead of print"
        value: "kotlin.io.print"
  UseDataClass:
    active: true
  UnnecessaryAbstractClass:
    active: true
  SerialVersionUIDInSerializableClass:
    active: false

exceptions:
  SwallowedException:
    active: true
    ignoredExceptionTypes:
      - "InterruptedException"
      - "CancellationException"
  TooGenericExceptionCaught:
    active: true
    exceptionNames:
      - "Exception"
      - "RuntimeException"
      - "Throwable"
    allowedExceptionNameRegex: "_|(ignored|expected).*"
  TooGenericExceptionThrown:
    active: true
    exceptionNames:
      - "Exception"
      - "RuntimeException"
      - "Throwable"

performance:
  SpreadOperator:
    active: true
  ForEachOnRange:
    active: true
  UnnecessaryTemporaryInstantiation:
    active: true

coroutines:
  GlobalCoroutineUsage:
    active: true
  SuspendFunSwallowedCancellation:
    active: true
  SuspendFunWithCoroutineScopeReceiver:
    active: true
```

### Gradle Integration

```kotlin
// build.gradle.kts (project-level)
plugins {
    id("io.gitlab.arturbosch.detekt") version "1.23.7"
}

detekt {
    buildUponDefaultConfig = true
    allRules = false
    config.setFrom("$rootDir/config/detekt/detekt.yml")
    baseline = file("$rootDir/config/detekt/baseline.xml")
    parallel = true
}

tasks.withType<io.gitlab.arturbosch.detekt.Detekt>().configureEach {
    reports {
        html.required.set(true)
        xml.required.set(true)
        sarif.required.set(true)
    }
}
```

### Common detekt Commands

```bash
# Run full detekt analysis
./gradlew detekt

# Generate baseline (for legacy projects migrating)
./gradlew detektBaseline

# Run on a specific module
./gradlew :app:detekt
./gradlew :feature:home:detekt
```

---

## 📦 Import Conventions

**Import organization affects readability. A consistent import order makes it easy to find dependencies at a glance.**

---

### Import Ordering

Imports are sorted alphabetically within each group, with groups separated by a blank line:

```kotlin
// 1. Kotlin standard library
import kotlin.collections.Map
import kotlin.coroutines.CoroutineContext

// 2. Kotlinx libraries
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.flow.StateFlow

// 3. Java / Javax
import java.time.LocalDate
import javax.inject.Inject

// 4. Android SDK
import android.content.Context
import android.os.Bundle

// 5. AndroidX libraries
import androidx.compose.foundation.layout.Column
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.lifecycle.ViewModel

// 6. Third-party libraries
import com.google.gson.Gson
import dagger.hilt.android.lifecycle.HiltViewModel
import retrofit2.http.GET

// 7. Project-internal imports
import com.arclabs.app.core.network.ApiService
import com.arclabs.app.domain.model.Restaurant
import com.arclabs.app.domain.usecase.GetRestaurantsUseCase
```

### Rules

```kotlin
// ✅ Correct: specific imports, alphabetically sorted
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding

// ❌ Wrong: wildcard import
import androidx.compose.foundation.layout.*

// ❌ Wrong: unused imports left in file
import androidx.compose.material3.Button  // Never used
import androidx.compose.material3.Text

// ❌ Wrong: unsorted imports
import androidx.compose.runtime.Composable
import androidx.compose.foundation.layout.Column  // Should come before runtime
```

### Alias Imports

Use alias imports sparingly and only to resolve genuine name conflicts:

```kotlin
// ✅ Correct: alias resolves a real conflict
import android.graphics.Color as AndroidColor
import androidx.compose.ui.graphics.Color as ComposeColor

// ❌ Wrong: unnecessary alias
import androidx.compose.ui.Modifier as Mod
```

---

## 🎯 Kotlin Idioms

**Idiomatic Kotlin is concise, expressive, and safe. These patterns leverage the language's strengths to produce clean, readable code.**

---

### Expression Functions

Use expression body for simple, single-expression functions:

```kotlin
// ✅ Correct: expression body for simple returns
fun Double.toCelsius(): Double = (this - 32) * 5 / 9

fun Restaurant.isHighlyRated(): Boolean = rating >= 4.5

fun List<Restaurant>.sortedByRating(): List<Restaurant> =
    sortedByDescending { it.rating }

// ❌ Wrong: block body for a simple return
fun Double.toCelsius(): Double {
    return (this - 32) * 5 / 9
}

// ✅ Correct: block body for multi-step logic
fun calculateDiscount(price: Double, membership: Membership): Double {
    val baseDiscount = membership.discountPercentage
    val seasonalBonus = if (isHolidaySeason()) 5.0 else 0.0
    return price * (baseDiscount + seasonalBonus) / 100
}
```

### when Expressions Over if-else Chains

```kotlin
// ✅ Correct: when expression
fun getStatusMessage(status: OrderStatus): String = when (status) {
    OrderStatus.PENDING -> "Your order is being prepared"
    OrderStatus.CONFIRMED -> "Your order has been confirmed"
    OrderStatus.IN_TRANSIT -> "Your order is on its way"
    OrderStatus.DELIVERED -> "Your order has been delivered"
    OrderStatus.CANCELLED -> "Your order was cancelled"
}

// ❌ Wrong: if-else chain for enum dispatch
fun getStatusMessage(status: OrderStatus): String {
    if (status == OrderStatus.PENDING) {
        return "Your order is being prepared"
    } else if (status == OrderStatus.CONFIRMED) {
        return "Your order has been confirmed"
    } else if (status == OrderStatus.IN_TRANSIT) {
        return "Your order is on its way"
    } else if (status == OrderStatus.DELIVERED) {
        return "Your order has been delivered"
    } else {
        return "Your order was cancelled"
    }
}
```

### Scope Functions Reference

| Function | Object Reference | Return Value | Use Case |
|----------|-----------------|--------------|----------|
| `let` | `it` | Lambda result | Null checks, transformations |
| `run` | `this` | Lambda result | Object configuration + compute |
| `with` | `this` | Lambda result | Grouping calls on an object |
| `apply` | `this` | Context object | Object configuration |
| `also` | `it` | Context object | Side effects (logging, validation) |

### let -- Null Safety and Transformations

```kotlin
// ✅ Correct: let for null check + transformation
val displayName = user?.name?.let { name ->
    "Welcome, $name"
} ?: "Welcome, Guest"

// ✅ Correct: let to limit scope
getRestaurantById(id)?.let { restaurant ->
    analytics.trackView(restaurant.id)
    showRestaurantDetail(restaurant)
}

// ❌ Wrong: unnecessary let
val name = "Hello".let { it.uppercase() }  // Just use "Hello".uppercase()
```

### apply -- Object Configuration

```kotlin
// ✅ Correct: apply for object initialization
val notification = NotificationCompat.Builder(context, CHANNEL_ID).apply {
    setSmallIcon(R.drawable.ic_notification)
    setContentTitle(title)
    setContentText(message)
    setPriority(NotificationCompat.PRIORITY_DEFAULT)
    setAutoCancel(true)
}.build()

// ❌ Wrong: repeated reference without apply
val builder = NotificationCompat.Builder(context, CHANNEL_ID)
builder.setSmallIcon(R.drawable.ic_notification)
builder.setContentTitle(title)
builder.setContentText(message)
builder.setPriority(NotificationCompat.PRIORITY_DEFAULT)
builder.setAutoCancel(true)
val notification = builder.build()
```

### also -- Side Effects

```kotlin
// ✅ Correct: also for logging / side effects
fun getUser(id: String): User? =
    userRepository.findById(id).also { user ->
        Timber.d("Fetched user: ${user?.name ?: "not found"}")
    }

// ✅ Correct: also for validation side effects
fun saveRestaurant(restaurant: Restaurant) {
    restaurant
        .also { require(it.name.isNotBlank()) { "Name must not be blank" } }
        .also { require(it.rating in 0.0..5.0) { "Rating must be 0-5" } }
        .let { repository.save(it) }
}
```

### run -- Configuration + Compute

```kotlin
// ✅ Correct: run to configure and compute in one step
val result = service.run {
    connectToEndpoint(baseUrl)
    authenticate(credentials)
    fetchData(query)
}
```

### with -- Grouping Calls

```kotlin
// ✅ Correct: with to group multiple property accesses
fun bindRestaurant(restaurant: Restaurant) {
    with(restaurant) {
        nameTextView.text = name
        ratingBar.rating = rating.toFloat()
        cuisineTextView.text = cuisine.displayName
        priceTextView.text = "$".repeat(priceRange)
    }
}
```

### Destructuring Declarations

```kotlin
// ✅ Correct: destructuring in loops
for ((id, name) in restaurantMap) {
    println("$id: $name")
}

// ✅ Correct: destructuring data classes
val (name, rating, cuisine) = restaurant

// ✅ Correct: destructuring in lambdas
restaurantMap.forEach { (id, name) ->
    Timber.d("Restaurant $id: $name")
}
```

### Null Safety Idioms

```kotlin
// ✅ Correct: Elvis operator for defaults
val displayName = user.name ?: "Unknown User"

// ✅ Correct: safe call chain
val cityName = user.address?.city?.name ?: "Unknown City"

// ✅ Correct: require / check for preconditions
fun processOrder(order: Order?) {
    requireNotNull(order) { "Order must not be null" }
    check(order.items.isNotEmpty()) { "Order must have at least one item" }
    // proceed with non-null, validated order
}

// ❌ Wrong: force unwrap
val name = user.name!!  // NEVER do this

// ❌ Wrong: manual null check when safe call works
if (user != null) {
    if (user.address != null) {
        if (user.address.city != null) {
            val cityName = user.address.city.name
        }
    }
}
```

### Collection Operations

```kotlin
// ✅ Correct: functional collection processing
val topRestaurants = restaurants
    .filter { it.isOpen && it.rating >= 4.0 }
    .sortedByDescending { it.rating }
    .take(5)
    .map { it.toSummary() }

// ✅ Correct: groupBy for categorization
val byCategory = restaurants.groupBy { it.cuisine }

// ✅ Correct: associate for map creation
val restaurantById = restaurants.associateBy { it.id }

// ❌ Wrong: manual loop when a collection function exists
val topRestaurants = mutableListOf<RestaurantSummary>()
for (restaurant in restaurants) {
    if (restaurant.isOpen && restaurant.rating >= 4.0) {
        topRestaurants.add(restaurant.toSummary())
    }
}
topRestaurants.sortByDescending { it.rating }
val result = topRestaurants.take(5)
```

### Sealed Interfaces and Exhaustive When

```kotlin
// ✅ Correct: sealed interface + exhaustive when
sealed interface NetworkResult<out T> {
    data class Success<T>(val data: T) : NetworkResult<T>
    data class Error(val exception: Throwable) : NetworkResult<Nothing>
    data object Loading : NetworkResult<Nothing>
}

fun <T> handleResult(result: NetworkResult<T>) {
    when (result) {
        is NetworkResult.Success -> showData(result.data)
        is NetworkResult.Error -> showError(result.exception)
        is NetworkResult.Loading -> showLoading()
        // No else needed -- compiler enforces exhaustiveness
    }
}
```

---

## 📋 Code Style Checklist

**Use this checklist before submitting a pull request. Every item should be satisfied.**

---

### Formatting

- [ ] All files formatted with `./gradlew ktlintFormat`
- [ ] No lines exceed 120 characters
- [ ] 4-space indentation (no tabs)
- [ ] Trailing commas on all multi-line declarations
- [ ] K&R brace style throughout
- [ ] Single blank line between members
- [ ] No blank line after opening brace or before closing brace
- [ ] Final newline at end of every file

### Naming

- [ ] Classes and interfaces use PascalCase
- [ ] Functions and properties use camelCase
- [ ] Constants use UPPER_SNAKE_CASE
- [ ] Composables use PascalCase
- [ ] Boolean properties start with is/has/should/can
- [ ] Event handlers follow on{Event} pattern
- [ ] Packages are lowercase with dots
- [ ] Test functions use descriptive backtick names

### Imports

- [ ] No wildcard imports
- [ ] No unused imports
- [ ] Imports sorted alphabetically within groups
- [ ] Proper group ordering (kotlin, kotlinx, java, android, androidx, third-party, project)

### Kotlin Idioms

- [ ] Expression body for single-expression functions
- [ ] `when` instead of long if-else chains
- [ ] Scope functions used appropriately
- [ ] Null safety via safe calls and Elvis (no `!!`)
- [ ] Collection operations instead of manual loops
- [ ] Destructuring where it improves readability

### Documentation

- [ ] All public classes have KDoc
- [ ] All public functions have KDoc
- [ ] Complex logic has inline comments explaining "why"
- [ ] No commented-out code

### Static Analysis

- [ ] `./gradlew ktlintCheck` passes with zero warnings
- [ ] `./gradlew detekt` passes with zero issues
- [ ] No suppression annotations without justification

---

## 🚫 Common Violations

**These are the five most frequently flagged issues in code review. Learn them, avoid them.**

---

### 1. Force Unwrapping with `!!`

```kotlin
// ❌ Wrong: force unwrap crashes at runtime
val name = user.name!!
val restaurant = restaurantMap[id]!!

// ✅ Correct: handle nullability safely
val name = user.name ?: "Unknown"
val restaurant = restaurantMap[id]
    ?: throw IllegalStateException("Restaurant $id not found")

// ✅ Correct: requireNotNull with a message
val restaurant = requireNotNull(restaurantMap[id]) {
    "Restaurant with id=$id was expected but not found"
}
```

### 2. Magic Numbers

```kotlin
// ❌ Wrong: magic numbers in logic
if (retryCount > 3) { /* ... */ }
delay(5000)
val padding = 16

// ✅ Correct: named constants
companion object {
    private const val MAX_RETRY_COUNT = 3
    private const val RETRY_DELAY_MS = 5_000L
}

private val ContentPadding = 16.dp

if (retryCount > MAX_RETRY_COUNT) { /* ... */ }
delay(RETRY_DELAY_MS)
```

### 3. Swallowed Exceptions

```kotlin
// ❌ Wrong: empty catch block hides failures
try {
    repository.saveUser(user)
} catch (e: Exception) {
    // silently swallowed
}

// ✅ Correct: log and/or propagate
try {
    repository.saveUser(user)
} catch (e: IOException) {
    Timber.e(e, "Failed to save user: ${user.id}")
    _uiState.value = UiState.Error("Could not save. Please try again.")
}
```

### 4. Wildcard Imports

```kotlin
// ❌ Wrong: wildcard import hides actual dependencies
import androidx.compose.foundation.layout.*
import kotlinx.coroutines.*

// ✅ Correct: explicit imports show what is used
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.launch
```

### 5. God Classes (Too Many Responsibilities)

```kotlin
// ❌ Wrong: ViewModel doing everything
class HomeViewModel : ViewModel() {
    // fetches data
    // formats strings
    // validates input
    // handles navigation
    // manages preferences
    // performs analytics
    // 500+ lines...
}

// ✅ Correct: single responsibility, delegated concerns
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getRestaurantsUseCase: GetRestaurantsUseCase,
    private val analyticsTracker: AnalyticsTracker,
) : ViewModel() {

    // Only orchestrates UI state based on use case results
    // Formatting lives in a mapper/formatter
    // Validation lives in the domain layer
    // Navigation is handled by the UI / Router
}
```

---

## 📚 Further Reading

**Authoritative references for Kotlin and Android code style.**

---

### Official References

- [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html) -- The official Kotlin style guide from JetBrains
- [Android Kotlin Style Guide](https://developer.android.com/kotlin/style-guide) -- Google's Kotlin style guide for Android
- [Kotlin API Design Best Practices](https://kotlinlang.org/docs/api-guidelines-introduction.html) -- Guidelines for designing Kotlin APIs

### Tooling Documentation

- [ktlint Official Documentation](https://pinterest.github.io/ktlint/) -- ktlint rules reference and configuration
- [detekt Official Documentation](https://detekt.dev/) -- detekt rules catalog and setup guide
- [ktlint Gradle Plugin](https://github.com/jlleitschuh/ktlint-gradle) -- Gradle integration for ktlint
- [detekt Gradle Plugin](https://detekt.dev/docs/gettingstarted/gradle/) -- Gradle integration for detekt

### Books and Guides

- [Effective Kotlin](https://effectivekotlin.com/) by Marcin Moskala -- Best practices for professional Kotlin developers
- [Kotlin in Action](https://www.manning.com/books/kotlin-in-action) by Dmitry Jemerov and Svetlana Isakova -- Comprehensive Kotlin reference
- [Android Development with Kotlin](https://developer.android.com/kotlin) -- Google's official Android + Kotlin resources

### ARC Labs Internal

- `ARCKnowledge-Android/Quality/` -- Full quality standards index
- `ARCKnowledge-Android/Architecture/` -- Architecture decision records
- `ARCKnowledge-Android/Layers/` -- Layer-specific guidelines

---

*This guide is maintained by ARC Labs Studio. Last updated: February 2026.*
