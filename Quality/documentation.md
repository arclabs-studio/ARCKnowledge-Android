# 📖 Documentation

**Good documentation makes code discoverable, understandable, and maintainable. ARC Labs uses KDoc and Dokka for all public APIs. This guide establishes comprehensive documentation standards for every Kotlin module, Jetpack Compose component, and architectural layer in our Android projects.**

---

## 🎯 Documentation Philosophy

Great documentation is not about restating the code in English. It is about capturing **intent**, **constraints**, and **context** that cannot be expressed through code alone.

### Core Principles

1. **Document the WHY, not the WHAT** -- The code already tells you what it does. Documentation should explain why it exists, what trade-offs were made, and what assumptions are in play.
2. **Keep docs close to code** -- KDoc lives with the source. External documentation drifts out of date. Prefer KDoc and inline comments over wiki pages or Confluence articles.
3. **All public APIs must have KDoc** -- If it is public, it is a contract. Every public class, interface, function, and property must be documented.
4. **Write for the maintainer, not yourself** -- Assume the reader has no context about your feature. They might be a new team member, or you six months from now.
5. **Documentation is part of the definition of done** -- A feature is not complete until its public API is documented, the CHANGELOG is updated, and Dokka generates without warnings.

### What Good Documentation Looks Like

- **Concise** -- Say what needs to be said, then stop.
- **Accurate** -- Outdated documentation is worse than no documentation. Keep it current.
- **Example-driven** -- A short code example is worth a paragraph of prose.
- **Layered** -- KDoc for API reference, README for overview, ADRs for decisions, CHANGELOG for history.

---

## 📝 KDoc Format

KDoc is the standard documentation format for Kotlin, processed by Dokka to generate API reference sites. KDoc combines Markdown syntax with special block tags like `@param`, `@return`, and `@throws`.

### General KDoc Structure

```kotlin
/**
 * Brief one-line summary of what this element does.
 *
 * Optional longer description that provides additional context,
 * explains behavior, constraints, or usage patterns. This section
 * supports full Markdown formatting including **bold**, *italic*,
 * [links], and code spans like `ClassName`.
 *
 * @param paramName Description of the parameter.
 * @return Description of the return value.
 * @throws ExceptionType Description of when this exception is thrown.
 * @see RelatedElement
 * @sample com.arclabs.samples.featureSample
 */
```

### Class Documentation

#### Regular Classes

```kotlin
/**
 * Retrieves user profiles from the repository.
 *
 * This use case validates the user ID, fetches the profile from the repository,
 * and ensures the user account is active before returning. It serves as the
 * single entry point for profile retrieval across all presentation layers.
 *
 * Usage:
 * ```
 * val user = getUserProfileUseCase("user-123")
 * ```
 *
 * @property userRepository The repository for accessing user data.
 * @property analyticsTracker Tracker for logging profile view events.
 * @see UserRepository
 * @see User
 */
class GetUserProfileUseCase @Inject constructor(
    private val userRepository: UserRepository,
    private val analyticsTracker: AnalyticsTracker,
)
```

#### Data Classes

```kotlin
/**
 * Represents a restaurant with its core attributes and location.
 *
 * This is the domain entity used throughout the application. It is mapped
 * from [RestaurantDto] at the data layer and should never contain
 * framework-specific annotations.
 *
 * @property id Unique identifier for the restaurant.
 * @property name Display name of the restaurant.
 * @property rating Average user rating from 0.0 to 5.0, or null if unrated.
 * @property location Geographic coordinates of the restaurant.
 * @property cuisineTypes List of cuisine categories this restaurant serves.
 * @property isOpenNow Whether the restaurant is currently accepting orders,
 *   based on the last sync with the remote API. This value may be stale
 *   if the device has been offline.
 */
data class Restaurant(
    val id: String,
    val name: String,
    val rating: Double?,
    val location: LatLng,
    val cuisineTypes: List<CuisineType>,
    val isOpenNow: Boolean,
)
```

#### Sealed Interfaces

```kotlin
/**
 * Represents the possible states of the home screen UI.
 *
 * The ViewModel exposes this as a [StateFlow] and the Compose UI
 * reacts to each state by rendering the appropriate content. Every
 * state transition is unidirectional: [Loading] -> [Success] or [Error].
 *
 * @see HomeViewModel.uiState
 */
sealed interface HomeUiState {

    /**
     * The initial loading state before any data is available.
     *
     * The UI should display a shimmer placeholder or progress indicator.
     */
    data object Loading : HomeUiState

    /**
     * Data has been successfully loaded and is ready for display.
     *
     * @property restaurants The list of restaurants to display.
     * @property lastUpdated Timestamp of the most recent data sync.
     */
    data class Success(
        val restaurants: List<Restaurant>,
        val lastUpdated: Instant,
    ) : HomeUiState

    /**
     * An error occurred during data loading.
     *
     * The UI should display an error message with a retry action.
     *
     * @property message A user-facing error description.
     * @property cause The underlying exception for logging purposes.
     */
    data class Error(
        val message: String,
        val cause: Throwable? = null,
    ) : HomeUiState
}
```

#### Abstract Classes

```kotlin
/**
 * Base class for all use cases that perform a single action and return a result.
 *
 * Subclasses must implement [execute] with their specific business logic.
 * The [invoke] operator handles coroutine dispatching and error mapping
 * so that subclasses do not need to manage threading or exception translation.
 *
 * All use cases execute on [Dispatchers.IO] by default. Override [dispatcher]
 * to change the execution context.
 *
 * @param P The type of the input parameter.
 * @param R The type of the result.
 * @see GetUserProfileUseCase for a concrete implementation.
 */
abstract class UseCase<in P, out R> {

    /**
     * The coroutine dispatcher used for executing the use case.
     *
     * Defaults to [Dispatchers.IO]. Override this property in subclasses
     * that require a different execution context (e.g., CPU-bound work
     * should use [Dispatchers.Default]).
     */
    open val dispatcher: CoroutineDispatcher = Dispatchers.IO

    /**
     * Executes the use case with the given [params].
     *
     * This method is called internally by [invoke] and should contain
     * the core business logic without concern for threading.
     *
     * @param params The input parameters for the use case.
     * @return The result of the operation.
     */
    protected abstract suspend fun execute(params: P): R

    /**
     * Invokes the use case, switching to the configured [dispatcher].
     *
     * @param params The input parameters.
     * @return The result from [execute].
     * @throws DomainException if the business logic encounters an error.
     */
    suspend operator fun invoke(params: P): R =
        withContext(dispatcher) { execute(params) }
}
```

### Function Documentation

#### Regular Functions

```kotlin
/**
 * Calculates the distance between two geographic coordinates using the
 * Haversine formula.
 *
 * The result is the great-circle distance, which is the shortest distance
 * between two points on the surface of a sphere. This does not account
 * for elevation differences.
 *
 * @param from The starting coordinates.
 * @param to The destination coordinates.
 * @return The distance in kilometers, rounded to two decimal places.
 */
fun calculateDistance(from: LatLng, to: LatLng): Double
```

#### Suspend Functions

```kotlin
/**
 * Fetches a user profile by ID.
 *
 * Validates the user ID is not blank and ensures the user is active.
 * This function is safe to call from any coroutine context as it
 * switches to [Dispatchers.IO] internally.
 *
 * @param userId The unique identifier of the user. Must not be blank.
 * @return The [User] entity if found and active.
 * @throws IllegalArgumentException if [userId] is blank.
 * @throws DomainException.UserInactive if the user account is deactivated.
 * @throws DomainException.UserNotFound if no user exists with the given ID.
 * @see UserRepository.getUser
 */
suspend operator fun invoke(userId: String): User
```

#### Operator Fun invoke()

Use cases in Clean Architecture expose an `operator fun invoke()` so they can be called like functions. Always document the full contract.

```kotlin
/**
 * Searches for restaurants matching the given query.
 *
 * Results are sorted by relevance and filtered to the user's current
 * region. If [query] is blank, returns an empty list immediately without
 * making a network request.
 *
 * The search debounces internally: calling this function multiple times
 * in rapid succession will cancel previous in-flight requests.
 *
 * @param query The search term. Supports partial matching on restaurant
 *   name and cuisine type. Case-insensitive.
 * @return A list of matching [Restaurant] entities, or an empty list
 *   if no results are found.
 * @throws NetworkException.NoConnection if the device is offline and
 *   no cached results are available.
 */
suspend operator fun invoke(query: String): List<Restaurant>
```

#### Extension Functions

```kotlin
/**
 * Converts this [RestaurantDto] to a domain [Restaurant] entity.
 *
 * Maps all fields from the network DTO to the domain model. The [rating]
 * field is clamped to the range 0.0..5.0 to guard against invalid API
 * responses. The [isOpenNow] flag is computed from [openingHours] and
 * the current system time.
 *
 * @return A new [Restaurant] entity with validated fields.
 * @see Restaurant
 */
fun RestaurantDto.toDomain(): Restaurant
```

#### Composable Functions

```kotlin
/**
 * Displays a restaurant card with image, name, rating, and distance.
 *
 * This composable follows Material 3 guidelines and supports both
 * light and dark themes. It uses [AsyncImage] for restaurant photos
 * with a shimmer placeholder during loading.
 *
 * @param restaurant The restaurant data to display.
 * @param onClick Called when the user taps the card. Receives the
 *   restaurant ID for navigation.
 * @param modifier Optional [Modifier] applied to the card root.
 */
@Composable
fun RestaurantCard(
    restaurant: Restaurant,
    onClick: (String) -> Unit,
    modifier: Modifier = Modifier,
)
```

### Property Documentation

#### StateFlow Properties

```kotlin
/**
 * The current UI state of the home screen.
 *
 * Emits [HomeUiState.Loading] initially, then transitions to
 * [HomeUiState.Success] or [HomeUiState.Error] after data loading
 * completes. Collectors receive the most recent state immediately
 * upon subscription (replay = 1).
 *
 * This flow never completes and survives configuration changes when
 * collected with [collectAsStateWithLifecycle].
 */
val uiState: StateFlow<HomeUiState>
```

#### lateinit Properties

```kotlin
/**
 * The navigation controller for the current navigation graph.
 *
 * Initialized in [onCreate] after [setContentView]. Accessing this
 * property before [onCreate] completes will throw
 * [UninitializedPropertyAccessException].
 */
lateinit var navController: NavHostController
```

#### Computed Properties

```kotlin
/**
 * Whether the restaurant list has any results to display.
 *
 * Derived from [uiState]. Returns `true` only when the state is
 * [HomeUiState.Success] and the restaurant list is not empty.
 */
val hasResults: Boolean
    get() = (uiState.value as? HomeUiState.Success)
        ?.restaurants
        ?.isNotEmpty() == true
```

### Constructor Documentation

#### @Inject Constructor with Hilt

When a class uses Hilt injection, document the constructor parameters using `@property` tags on the class KDoc. This keeps the documentation in one place.

```kotlin
/**
 * ViewModel for the restaurant list screen.
 *
 * Loads restaurants on initialization and exposes the result as
 * a [StateFlow] of [RestaurantListUiState]. Supports pull-to-refresh
 * and search filtering.
 *
 * @property getRestaurantsUseCase Retrieves all restaurants for the
 *   user's current region.
 * @property searchRestaurantsUseCase Searches restaurants by query string.
 * @property savedStateHandle Stores and restores the current search
 *   query across process death.
 * @see RestaurantListScreen
 */
@HiltViewModel
class RestaurantListViewModel @Inject constructor(
    private val getRestaurantsUseCase: GetRestaurantsUseCase,
    private val searchRestaurantsUseCase: SearchRestaurantsUseCase,
    private val savedStateHandle: SavedStateHandle,
) : ViewModel()
```

### Module Documentation

#### Hilt @Module @InstallIn

```kotlin
/**
 * Provides network-related dependencies for the application.
 *
 * All bindings in this module are scoped to [SingletonComponent], meaning
 * they live for the entire application lifecycle. This is appropriate for
 * expensive objects like [OkHttpClient] and [Retrofit] that should be
 * shared across all features.
 *
 * @see NetworkInterceptor for request/response logging configuration.
 * @see ApiService for the available API endpoints.
 */
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    /**
     * Provides the [OkHttpClient] configured with authentication,
     * logging, and timeout settings.
     *
     * Timeouts are set to 30 seconds for connect, read, and write.
     * The [AuthInterceptor] adds the bearer token to every request.
     *
     * @param authInterceptor Adds authentication headers to requests.
     * @param loggingInterceptor Logs HTTP request and response bodies
     *   in debug builds only.
     * @return A configured [OkHttpClient] instance.
     */
    @Provides
    @Singleton
    fun provideOkHttpClient(
        authInterceptor: AuthInterceptor,
        loggingInterceptor: HttpLoggingInterceptor,
    ): OkHttpClient = OkHttpClient.Builder()
        .addInterceptor(authInterceptor)
        .addInterceptor(loggingInterceptor)
        .connectTimeout(30, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .writeTimeout(30, TimeUnit.SECONDS)
        .build()
}
```

#### Hilt @Binds Modules

```kotlin
/**
 * Binds repository interfaces to their concrete implementations.
 *
 * This module uses [@Binds][dagger.Binds] instead of [@Provides][dagger.Provides]
 * because the implementations have their own `@Inject` constructors
 * and do not require manual instantiation.
 *
 * @see RepositoryModule for data-source bindings.
 */
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    /**
     * Binds [UserRepositoryImpl] to the [UserRepository] interface.
     *
     * The implementation uses Room for local caching and Retrofit
     * for remote data fetching with an offline-first strategy.
     */
    @Binds
    @Singleton
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl,
    ): UserRepository
}
```

### KDoc Tags Reference

#### @param

Documents a function parameter or a type parameter of a class, property, or function. Use one `@param` tag per parameter.

```kotlin
/**
 * Filters restaurants by cuisine type and minimum rating.
 *
 * @param cuisineType The cuisine category to filter by. Use [CuisineType.ALL]
 *   to skip cuisine filtering.
 * @param minRating The minimum rating threshold (inclusive). Must be in
 *   the range 0.0..5.0.
 * @param limit Maximum number of results to return. Defaults to 20.
 */
fun filterRestaurants(
    cuisineType: CuisineType,
    minRating: Double,
    limit: Int = 20,
): List<Restaurant>
```

#### @return

Documents the return value. Omit for `Unit`-returning functions.

```kotlin
/**
 * Attempts to authenticate the user with the given credentials.
 *
 * @param email The user's email address.
 * @param password The user's password.
 * @return A [Result] containing the [AuthToken] on success, or an
 *   [AuthException] on failure. The token includes both access and
 *   refresh tokens with their expiration timestamps.
 */
suspend fun login(email: String, password: String): Result<AuthToken>
```

#### @throws

Documents exceptions that a function may throw. List each possible exception separately.

```kotlin
/**
 * Saves a restaurant to the user's favorites.
 *
 * @param restaurantId The ID of the restaurant to favorite.
 * @throws DomainException.NotAuthenticated if the user is not logged in.
 * @throws DomainException.NotFound if the restaurant does not exist.
 * @throws DomainException.AlreadyExists if the restaurant is already
 *   in the user's favorites.
 * @throws NetworkException.NoConnection if the device is offline and
 *   the operation cannot be queued.
 */
suspend fun addFavorite(restaurantId: String)
```

#### @see

Creates a link to another element. Useful for referencing related classes, functions, or external documentation.

```kotlin
/**
 * Repository interface for restaurant data operations.
 *
 * @see RestaurantRepositoryImpl for the concrete implementation.
 * @see RestaurantLocalDataSource for the caching strategy.
 * @see RestaurantRemoteDataSource for the API integration.
 */
interface RestaurantRepository
```

#### @sample

References a function whose body will be included as a code example in the generated documentation.

```kotlin
/**
 * A composable that displays a paginated list with pull-to-refresh.
 *
 * @sample com.arclabs.samples.PaginatedListSamples.basicUsage
 * @sample com.arclabs.samples.PaginatedListSamples.withCustomEmptyState
 */
@Composable
fun <T> PaginatedList(
    items: LazyPagingItems<T>,
    onRefresh: () -> Unit,
    content: @Composable (T) -> Unit,
)
```

The referenced sample function (placed in a `samples` source set):

```kotlin
// In samples/com/arclabs/samples/PaginatedListSamples.kt
package com.arclabs.samples

fun basicUsage() {
    // val items = viewModel.restaurants.collectAsLazyPagingItems()
    // PaginatedList(
    //     items = items,
    //     onRefresh = viewModel::refresh,
    // ) { restaurant ->
    //     RestaurantCard(restaurant = restaurant)
    // }
}
```

#### @property

Documents a property of a class. Typically used in the class-level KDoc to describe constructor parameters.

```kotlin
/**
 * Configuration for the HTTP client.
 *
 * @property baseUrl The base URL for all API requests. Must include
 *   the trailing slash (e.g., `https://api.example.com/v1/`).
 * @property timeoutSeconds Connection and read timeout in seconds.
 *   Defaults to 30.
 * @property enableLogging Whether to log HTTP request and response
 *   bodies. Should be `false` in production builds.
 */
data class NetworkConfig(
    val baseUrl: String,
    val timeoutSeconds: Int = 30,
    val enableLogging: Boolean = false,
)
```

#### @since and @suppress

```kotlin
/**
 * Retrieves the user's notification preferences.
 *
 * @since 2.1.0
 */
suspend fun getNotificationPreferences(): NotificationPreferences

/**
 * Legacy method kept for binary compatibility. Use [getNotificationPreferences] instead.
 *
 * @suppress
 */
@Deprecated("Use getNotificationPreferences()", ReplaceWith("getNotificationPreferences()"))
suspend fun getNotifPrefs(): NotificationPreferences
```

---

## 📋 What to Document

### Always Document

- **All public classes, interfaces, and functions** -- These form your module's API contract.
- **Complex business logic** -- If a function has branching logic, calculations, or state machines, explain the rules.
- **Non-obvious decisions** -- If you chose approach A over approach B, document WHY.
- **Public API of libraries** -- Every public symbol in a shared module must have KDoc.
- **Side effects** -- If a function writes to disk, sends analytics, or mutates shared state, say so.
- **Thread safety** -- If a class is thread-safe or requires external synchronization, document it.
- **Architecture Decision Records** -- Major architectural choices get their own documents.

### Correct Examples

```kotlin
// ✅ Documents WHY the delay exists, not just WHAT it does
/**
 * Delays search execution by 300ms to debounce rapid user input.
 *
 * Without this delay, every keystroke would trigger a network request,
 * overwhelming the API and causing UI jank. The 300ms threshold was
 * chosen based on UX research showing it feels responsive while
 * reducing API calls by ~80%.
 */
private val searchDebounceMs = 300L
```

```kotlin
// ✅ Documents a complex mapping with business rules
/**
 * Maps the API price tier to a user-facing price label.
 *
 * The API returns numeric tiers (1-4) but the design system uses
 * dollar sign strings. Tier 0 (free) and unknown tiers default to
 * an empty string per product decision FAVRES-234.
 *
 * | Tier | Label |
 * |------|-------|
 * | 1    | $     |
 * | 2    | $$    |
 * | 3    | $$$   |
 * | 4    | $$$$  |
 *
 * @param tier The numeric price tier from the API.
 * @return The dollar-sign label, or an empty string for unknown tiers.
 */
fun mapPriceTier(tier: Int): String
```

### Do Not Document

```kotlin
// ❌ Don't document obvious getters/setters
/**
 * Gets the name. <-- Useless, adds noise
 */
fun getName(): String = name

// ❌ Don't document overridden methods unless behavior changes
/**
 * Returns a string representation. <-- Already documented in Any
 */
override fun toString(): String = "User($id)"
```

```kotlin
// ✅ DO document overridden methods when behavior is non-obvious
/**
 * Compares restaurants by distance from the user's location, then
 * by rating in descending order if distances are equal.
 *
 * This ordering matches the product requirement in FAVRES-567 where
 * nearby high-rated restaurants appear first.
 */
override fun compareTo(other: Restaurant): Int
```

```kotlin
// ❌ Don't restate the code in documentation
/**
 * Sets isLoading to true. <-- We can see that
 */
fun startLoading() {
    isLoading = true
}

// ✅ DO explain intent and side effects
/**
 * Triggers the loading state and cancels any pending search operations.
 *
 * Call this before initiating a new data fetch to ensure the UI
 * shows the loading indicator and previous results are cleared.
 */
fun startLoading() {
    searchJob?.cancel()
    _uiState.value = UiState.Loading
}
```

---

## 🔧 Dokka Setup

[Dokka](https://github.com/Kotlin/dokka) is the documentation engine for Kotlin. It processes KDoc comments and generates HTML, Markdown, or Javadoc output.

### Single-Module Setup

```kotlin
// build.gradle.kts (module-level)
plugins {
    id("org.jetbrains.dokka") version "1.9.20"
}

tasks.dokkaHtml {
    outputDirectory.set(layout.buildDirectory.dir("dokka/html"))

    dokkaSourceSets {
        named("main") {
            // Include module documentation
            includes.from("Module.md")

            // Link to Android SDK documentation
            externalDocumentationLink {
                url.set(java.net.URL("https://developer.android.com/reference/"))
                packageListUrl.set(
                    java.net.URL("https://developer.android.com/reference/android/package-list")
                )
            }

            // Link to Kotlinx Coroutines documentation
            externalDocumentationLink {
                url.set(java.net.URL("https://kotlinlang.org/api/kotlinx.coroutines/"))
            }

            // Suppress internal packages from generated docs
            perPackageOption {
                matchingRegex.set(".*\\.internal.*")
                suppress.set(true)
            }
        }
    }
}
```

### Multi-Module Setup

For projects with multiple Gradle modules, configure the root project to aggregate documentation from all subprojects.

```kotlin
// build.gradle.kts (root project)
plugins {
    id("org.jetbrains.dokka") version "1.9.20"
}

tasks.dokkaHtmlMultiModule {
    outputDirectory.set(layout.buildDirectory.dir("dokka/html"))
}

// Each submodule also applies the plugin:
// build.gradle.kts (each submodule)
plugins {
    id("org.jetbrains.dokka")
}
```

### Custom Templates and Styles

```kotlin
tasks.dokkaHtml {
    pluginConfiguration<org.jetbrains.dokka.base.DokkaBase, org.jetbrains.dokka.base.DokkaBaseConfiguration> {
        // Custom CSS for ARC Labs branding
        customStyleSheets = listOf(file("docs/styles/arclabs-dokka.css"))

        // Custom images (logo, favicon)
        customAssets = listOf(
            file("docs/assets/logo.svg"),
            file("docs/assets/favicon.ico"),
        )

        // Footer message
        footerMessage = "ARC Labs Studio - Generated Documentation"
    }
}
```

### Generating Documentation

```bash
# Single module HTML documentation
./gradlew dokkaHtml

# Single module GitHub-flavored Markdown
./gradlew dokkaGfm

# Single module Javadoc-compatible output
./gradlew dokkaJavadoc

# Multi-module aggregated HTML documentation
./gradlew dokkaHtmlMultiModule

# Generate and open in browser
./gradlew dokkaHtml && open build/dokka/html/index.html
```

### CI Integration

Add documentation generation to your CI pipeline to catch KDoc warnings early.

```yaml
# .github/workflows/docs.yml
name: Documentation

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  dokka:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - name: Generate documentation
        run: ./gradlew dokkaHtmlMultiModule --no-daemon
      - name: Check for Dokka warnings
        run: |
          if grep -r "WARNING" build/dokka/ 2>/dev/null; then
            echo "Dokka generated warnings. Fix KDoc issues."
            exit 1
          fi
      - name: Upload documentation
        if: github.ref == 'refs/heads/main'
        uses: actions/upload-pages-artifact@v3
        with:
          path: build/dokka/html
```

---

## 📓 CHANGELOG Format

ARC Labs follows [Keep a Changelog](https://keepachangelog.com/) for version history documentation. Every user-facing change must be recorded before a release.

### Structure

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- Restaurant search with cuisine type filtering (FAVRES-234)
- Pull-to-refresh on restaurant list screen

### Changed
- Updated Retrofit to 2.11.0

### Fixed
- Map annotations disappearing after screen rotation (FAVRES-345)

## [1.2.0] - 2026-02-15

### Added
- User favorites feature with offline sync
- Push notification preferences screen

### Changed
- Migrated from Moshi to Kotlin Serialization for JSON parsing
- Minimum SDK raised from 26 to 28

### Deprecated
- `RestaurantApi.getRestaurantsV1()` -- use `getRestaurantsV2()` instead

### Removed
- Legacy analytics SDK integration

### Fixed
- Crash when opening app with no network connection (FAVRES-289)
- Memory leak in MapViewModel (FAVRES-301)

### Security
- Updated OkHttp to 4.12.0 to address CVE-2023-XXXXX

## [1.1.0] - 2026-01-20
...
```

### Categories

| Category       | When to use                                        |
|----------------|----------------------------------------------------|
| **Added**      | New features                                       |
| **Changed**    | Changes in existing functionality                  |
| **Deprecated** | Features that will be removed in future releases   |
| **Removed**    | Features removed in this release                   |
| **Fixed**      | Bug fixes                                          |
| **Security**   | Vulnerability patches                              |

### Rules

1. Every user-facing PR adds an entry under `[Unreleased]`.
2. Entries reference the Linear issue ID (e.g., FAVRES-234).
3. On release, rename `[Unreleased]` to the version number and date.
4. Keep entries concise -- one line per change.

---

## 📐 Architecture Decision Records (ADRs)

An ADR captures a significant architectural decision, the context that led to it, and the consequences. They prevent revisiting the same debates and help new team members understand why the codebase looks the way it does.

### When to Create an ADR

- Choosing a library or framework (e.g., Room vs. SQLDelight)
- Changing the architecture pattern (e.g., adopting MVI)
- Modifying the module structure
- Adding a new cross-cutting concern (e.g., error handling strategy)
- Any decision that took more than 30 minutes of team discussion

### ADR Template

```markdown
# ADR-NNN: Title of Decision

## Status
Accepted | Proposed | Deprecated | Superseded by ADR-XXX

## Date
2026-02-25

## Context
What is the issue that we are seeing that is motivating this decision?
Describe the forces at play (technical, organizational, business).

## Decision
What is the change that we are proposing and/or doing?

## Alternatives Considered

### Alternative A: [Name]
- **Pros:** ...
- **Cons:** ...

### Alternative B: [Name]
- **Pros:** ...
- **Cons:** ...

## Consequences

### Positive
- What becomes easier or possible as a result of this change?

### Negative
- What becomes harder or riskier as a result of this change?

### Neutral
- What other changes or work does this decision require?

## References
- Link to discussion thread, RFC, or related ADRs
```

### Example ADR

```markdown
# ADR-003: Use Kotlin Serialization Instead of Moshi

## Status
Accepted

## Date
2026-01-10

## Context
Our project uses Moshi for JSON serialization. Moshi requires either
code generation (increasing build times by ~15%) or reflection (adding
~200KB to APK size). Kotlin Serialization is a first-party Kotlin
library with compiler plugin support, better multiplatform compatibility,
and native Kotlin type support.

## Decision
Migrate from Moshi to Kotlin Serialization for all JSON parsing.
Use the `kotlinx-serialization-json` artifact with `@Serializable`
annotations on all DTOs.

## Alternatives Considered

### Alternative A: Keep Moshi
- **Pros:** Already integrated, team familiarity
- **Cons:** Longer build times, reflection overhead, not multiplatform

### Alternative B: Gson
- **Pros:** Widely used, large community
- **Cons:** Reflection-based, poor Kotlin null safety, no multiplatform

## Consequences

### Positive
- 12% faster clean builds (no Moshi code generation)
- Better Kotlin type support (sealed classes, value classes)
- Path to Kotlin Multiplatform if needed in the future

### Negative
- Migration effort (~2 weeks across all modules)
- Team needs to learn new annotation patterns

## References
- Kotlin Serialization docs: https://kotlinlang.org/docs/serialization.html
- FAVRES-198: Migration tracking issue
```

### File Location

Store ADRs in the `docs/decisions/` directory of the project root, numbered sequentially:

```
docs/
  decisions/
    ADR-001-clean-architecture.md
    ADR-002-hilt-dependency-injection.md
    ADR-003-kotlin-serialization.md
```

---

## 💬 Inline Comments

Inline comments supplement KDoc for explaining complex implementation details within function bodies. Use them sparingly and only when the code alone is insufficient.

### When to Use Inline Comments

- **Complex algorithms** -- Explain the logic step by step.
- **Workarounds** -- Document why a hacky solution is necessary.
- **Non-obvious behavior** -- Clarify subtle platform quirks.
- **Performance-critical sections** -- Explain why a specific approach was chosen.

### When NOT to Use Inline Comments

- **To explain WHAT the code does** -- If the code is clear, the comment adds noise.
- **To disable code** -- Delete dead code; use version control to recover it.
- **To blame or apologize** -- Remove `// sorry about this` or `// I know this is bad`.

### TODO Format

Always include a Linear ticket reference and a brief description. TODOs without ticket references become permanent fixtures.

```kotlin
// ✅ Good TODO format
// TODO(FAVRES-456): Replace manual JSON parsing with Kotlin Serialization
// TODO(FAVRES-789): Add pagination support -- currently loads all results

// ❌ Bad TODO format
// TODO: fix this later
// TODO fix this
// FIXME something is wrong here
// HACK: this is terrible but works
```

### Inline Comment Examples

```kotlin
// ✅ Explains a non-obvious workaround
// Android 12 has a bug where WindowInsets.ime doesn't report correctly
// on first composition. We delay the check by one frame as a workaround.
// See: https://issuetracker.google.com/issues/XXXXXX
LaunchedEffect(Unit) {
    withFrameNanos { }
    isKeyboardVisible = WindowInsets.ime.getBottom(density) > 0
}
```

```kotlin
// ✅ Explains a business rule embedded in code
// Restaurants with fewer than 10 reviews are excluded from the
// "Top Rated" section to avoid statistical outliers (FAVRES-234).
val topRated = restaurants.filter { it.reviewCount >= 10 }
    .sortedByDescending { it.rating }
    .take(10)
```

```kotlin
// ❌ Noise -- the code is self-explanatory
// Increment the counter
counter++

// ❌ Noise -- restates the code
// Check if the list is empty
if (restaurants.isEmpty()) {
```

---

## 📁 Documentation Structure

A well-organized project has documentation at multiple levels:

| File/Directory            | Purpose                                      |
|---------------------------|----------------------------------------------|
| `README.md`               | Project overview, setup, quick start         |
| `CHANGELOG.md`            | Version history (Keep a Changelog format)    |
| `docs/decisions/`         | Architecture Decision Records                |
| `docs/guides/`            | Developer guides and how-tos                 |
| `Module.md`               | Dokka module-level documentation             |
| KDoc in source files       | API reference documentation                  |
| Inline comments            | Implementation detail explanations           |

See [README Standards](./readme-standards.md) for the README template.

---

## 📋 Documentation Checklist

Use this checklist before merging any PR.

### KDoc
- [ ] All public classes and interfaces have KDoc with a summary
- [ ] All public functions have `@param`, `@return`, and `@throws` tags
- [ ] All sealed interface/class variants are individually documented
- [ ] Data class properties are documented with `@property` tags
- [ ] Hilt modules and `@Provides` functions are documented
- [ ] `@see` tags link to related types where appropriate

### Code Comments
- [ ] Complex business logic has inline comments explaining WHY
- [ ] All TODOs reference a Linear ticket (e.g., `TODO(FAVRES-123)`)
- [ ] No commented-out code blocks (use version control instead)
- [ ] Workarounds reference the upstream issue or bug tracker link

### Project Documentation
- [ ] README.md follows [ARC Labs standards](./readme-standards.md)
- [ ] CHANGELOG.md has an entry under `[Unreleased]` for user-facing changes
- [ ] ADR created for any significant architectural decision
- [ ] Dokka generates without warnings (`./gradlew dokkaHtml`)

### Review
- [ ] Documentation is accurate and reflects the current implementation
- [ ] No spelling or grammar errors in public-facing documentation
- [ ] Code examples in KDoc compile and are correct

---

## ❌ Common Mistakes

### Mistake 1: Outdated KDoc

KDoc that contradicts the code is worse than no KDoc at all. When you change a function's behavior, update its documentation in the same commit.

```kotlin
// ❌ KDoc says it returns a User, but the function now returns Result<User>
/**
 * Fetches a user by ID.
 *
 * @return The [User] entity.
 */
suspend fun getUser(id: String): Result<User>

// ✅ KDoc matches the actual return type and behavior
/**
 * Fetches a user by ID.
 *
 * @return A [Result] containing the [User] on success, or a
 *   [DomainException] on failure.
 */
suspend fun getUser(id: String): Result<User>
```

### Mistake 2: Documenting WHAT Instead of WHY

```kotlin
// ❌ Restates the code -- adds no value
/**
 * Filters the list and takes the first 10 items.
 */
val topRestaurants = restaurants
    .filter { it.rating >= 4.0 }
    .take(10)

// ✅ Explains the business logic and reasoning
/**
 * Selects the top 10 restaurants with a 4-star minimum rating for the
 * "Featured" carousel. The limit of 10 matches the design spec in
 * Figma (FAVRES-567) and keeps the horizontal scroll performant.
 */
val topRestaurants = restaurants
    .filter { it.rating >= 4.0 }
    .take(10)
```

### Mistake 3: Missing @throws Documentation

```kotlin
// ❌ Callers don't know what exceptions to handle
/**
 * Saves the user profile.
 *
 * @param profile The profile to save.
 */
suspend fun saveProfile(profile: UserProfile)

// ✅ Every thrown exception is documented
/**
 * Saves the user profile to the remote API and local cache.
 *
 * @param profile The profile to save. The [UserProfile.email] field
 *   must be a valid email address.
 * @throws ValidationException if [profile] contains invalid fields.
 * @throws NetworkException.NoConnection if the device is offline.
 * @throws NetworkException.ServerError if the API returns a 5xx response.
 * @throws StorageException if the local database write fails.
 */
suspend fun saveProfile(profile: UserProfile)
```

### Mistake 4: No Code Examples for Complex APIs

```kotlin
// ❌ Users don't know how to use this
/**
 * Creates a paginated data source for restaurants.
 */
fun createRestaurantPager(config: PagerConfig): Flow<PagingData<Restaurant>>

// ✅ Includes a usage example
/**
 * Creates a paginated data source for restaurants.
 *
 * The returned [Flow] emits [PagingData] that can be collected in a
 * Composable using [collectAsLazyPagingItems].
 *
 * Usage:
 * ```
 * val pager = createRestaurantPager(PagerConfig(pageSize = 20))
 * val items = pager.collectAsLazyPagingItems()
 *
 * LazyColumn {
 *     items(items.itemCount) { index ->
 *         items[index]?.let { RestaurantCard(it) }
 *     }
 * }
 * ```
 *
 * @param config Pagination configuration including page size and
 *   prefetch distance.
 * @return A cold [Flow] of [PagingData] that loads pages on demand.
 */
fun createRestaurantPager(config: PagerConfig): Flow<PagingData<Restaurant>>
```

### Mistake 5: KDoc on Private Implementation Details

```kotlin
// ❌ Over-documenting private internals adds maintenance burden
/**
 * Temporary variable used to hold the intermediate computation
 * result of the restaurant distance calculation before applying
 * the Haversine correction factor.
 */
private var tempDistance: Double = 0.0

// ✅ Private members only need docs when behavior is complex
// Accumulates distance corrections for the Haversine formula.
// See calculateDistance() for the full algorithm.
private var distanceAccumulator: Double = 0.0
```

### Mistake 6: Forgetting Module Documentation

```kotlin
// ❌ Module has no top-level documentation -- Dokka generates a blank page
// (No Module.md file)

// ✅ Module.md file provides context for the generated docs
```

Create a `Module.md` at the module root:

```markdown
# Module feature-restaurants

Provides the restaurant browsing and search feature for the ARC Labs
Favorites app. This module contains the UI layer (Compose screens and
ViewModels) and depends on `:core:domain` for use cases and entities.

## Architecture

This feature follows the MVVM pattern with unidirectional data flow:

```
Screen (Compose) -> ViewModel -> UseCase -> Repository
```

## Key Classes

- [RestaurantListScreen] - Main screen showing all restaurants
- [RestaurantListViewModel] - Manages state for the list screen
- [RestaurantDetailScreen] - Detail view for a single restaurant
```

---

## 📚 Further Reading

- [KDoc Reference](https://kotlinlang.org/docs/kotlin-doc.html)
- [Dokka Documentation](https://kotlin.github.io/dokka/)
- [Keep a Changelog](https://keepachangelog.com/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [README Standards](./readme-standards.md)
- [Code Review Standards](./code-review.md)
- [Testing Standards](./testing.md)

---

**Remember**: Write documentation for the developer who will maintain this code in 6 months -- it might be you. 📖
