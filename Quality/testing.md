# 🧪 Testing

**The definitive guide to testing Android/Kotlin applications at ARC Labs Studio. This document establishes the testing philosophy, patterns, frameworks, and standards that every ARC Labs Android project must follow. Tests are first-class citizens -- they document behavior, enable fearless refactoring, catch regressions before users do, and drive better software design.**

---

## 🎯 Testing Philosophy

### Core Principles

Our testing approach is built on four pillars:

1. **Tests Are Documentation** -- A well-written test suite is the most accurate and up-to-date documentation of how your system behaves. When a developer reads a test, they should understand the feature without reading the implementation.

2. **Tests Enable Refactoring** -- Without tests, refactoring is gambling. With tests, you can restructure internals with confidence that external behavior is preserved. Tests give you the freedom to improve your codebase continuously.

3. **Tests Catch Regressions** -- Every bug that reaches production is a missing test. When you fix a bug, write a test that proves it is fixed. That bug will never return.

4. **Tests Drive Design** -- Code that is hard to test is poorly designed. If you cannot test a class in isolation, its dependencies are too tightly coupled. Testing drives you toward better architecture.

### The Testing Pyramid

```
         /  UI   \          <- Few, slow, high confidence
        / Integr. \         <- Some, moderate speed
       /   Unit    \        <- Many, fast, focused
      /______________\
```

- **Unit Tests (70%)** -- Fast, isolated, test a single class or function
- **Integration Tests (20%)** -- Test collaboration between components (e.g., ViewModel + UseCase, DAO + Database)
- **UI Tests (10%)** -- Test rendered composables and user interactions

### Coverage Requirements

| Project Type | Minimum Coverage | Target Coverage | Enforced |
|---|---|---|---|
| **Android Libraries** | 80% | 100% | Yes (CI gate) |
| **Android Apps** | 60% | 80% | Yes (CI gate) |
| **Shared Modules (KMP)** | 80% | 100% | Yes (CI gate) |
| **Demo Apps** | 40% | 60% | No |

> **Note:** Coverage is measured by JaCoCo. Line coverage and branch coverage are both tracked. The minimum threshold applies to line coverage. Branch coverage should be within 10% of line coverage.

### What to Test

**ALWAYS test these layers:**

- **ViewModels** -- State transitions, event emissions, error handling, loading states
- **Use Cases** -- Business logic orchestration, input validation, error mapping
- **Repositories** -- Data source coordination, caching logic, error handling, mapping
- **Entities** -- Computed properties, validation rules, equality, business invariants
- **Utilities** -- Formatters, parsers, extensions, mappers, converters
- **DTOs / Mappers** -- Mapping from data layer models to domain models and back

**Test these when practical:**

- **Composable functions** -- Stateless content composables with known inputs
- **Room DAOs** -- Insert, query, update, delete with in-memory database
- **Network response parsing** -- JSON deserialization with Kotlinx Serialization
- **Navigation arguments** -- Route building and argument extraction

### What NOT to Test

**Do not test these (for now):**

- **Navigation flows** -- Deferred to integration testing phase; mocking NavController is brittle
- **Third-party library internals** -- Trust Retrofit, Room, Hilt; test your usage of them
- **Android framework classes** -- Activities, Fragments, Services (test ViewModels instead)
- **Generated code** -- Hilt modules, Room generated DAOs, Kotlin Serialization adapters
- **Trivial code** -- Simple data classes with no logic, direct delegations with no transformation

---

## 🧪 JUnit 5 Framework

### Why JUnit 5?

JUnit 5 (Jupiter) is the modern standard for JVM testing. It provides significant improvements over JUnit 4:

- **`@Nested`** -- Organize tests into logical groups within a single class
- **`@DisplayName`** -- Human-readable test descriptions that appear in reports
- **`@ParameterizedTest`** -- Run the same test with different inputs automatically
- **`@BeforeEach` / `@AfterEach`** -- Lifecycle hooks without `@Rule` boilerplate
- **`assertThrows<T>`** -- Type-safe exception assertions
- **Extensions API** -- Replace JUnit 4 `@Rule` with composable extensions
- **Kotlin-friendly** -- Backtick function names, reified generics, coroutine support

### Gradle Setup

```kotlin
// build.gradle.kts (module)
android {
    testOptions {
        unitTests.all {
            it.useJUnitPlatform()
        }
    }
}

dependencies {
    // JUnit 5
    testImplementation("org.junit.jupiter:junit-jupiter-api:5.10.2")
    testImplementation("org.junit.jupiter:junit-jupiter-params:5.10.2")
    testRuntimeOnly("org.junit.jupiter:junit-jupiter-engine:5.10.2")

    // MockK
    testImplementation("io.mockk:mockk:1.13.10")

    // Coroutines Test
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.8.1")

    // Turbine (Flow testing)
    testImplementation("app.cash.turbine:turbine:1.1.0")

    // Truth (optional, for fluent assertions)
    testImplementation("com.google.truth:truth:1.4.2")
}
```

### Basic Structure

Every test class follows the **Given-When-Then** pattern and uses `createSUT()` (System Under Test) factory:

```kotlin
import io.mockk.coEvery
import io.mockk.mockk
import kotlinx.coroutines.test.runTest
import org.junit.jupiter.api.DisplayName
import org.junit.jupiter.api.Nested
import org.junit.jupiter.api.Test
import app.cash.turbine.test
import kotlin.test.assertEquals
import kotlin.test.assertIs

class UserProfileViewModelTest {

    // MARK: - Dependencies

    private val getUserUseCase: GetUserUseCase = mockk()
    private val updateUserUseCase: UpdateUserUseCase = mockk()

    // MARK: - SUT Factory

    private fun createSUT() = UserProfileViewModel(
        getUserUseCase = getUserUseCase,
        updateUserUseCase = updateUserUseCase,
    )

    // MARK: - Tests

    @Nested
    @DisplayName("loadUser")
    inner class LoadUser {

        @Test
        @DisplayName("emits Loading then Success when user is found")
        fun `emits Loading then Success when user is found`() = runTest {
            // Given
            val user = User.fake()
            coEvery { getUserUseCase(any()) } returns user
            val viewModel = createSUT()

            // When
            viewModel.loadUser("123")

            // Then
            viewModel.uiState.test {
                assertEquals(UserUiState.Success(user), awaitItem())
                cancelAndIgnoreRemainingEvents()
            }
        }

        @Test
        @DisplayName("emits Error when use case throws UserNotFound")
        fun `emits Error when use case throws UserNotFound`() = runTest {
            // Given
            coEvery { getUserUseCase(any()) } throws DomainException.UserNotFound("123")
            val viewModel = createSUT()

            // When
            viewModel.loadUser("123")

            // Then
            viewModel.uiState.test {
                val state = awaitItem()
                assertIs<UserUiState.Error>(state)
                assertEquals("User not found", state.message)
                cancelAndIgnoreRemainingEvents()
            }
        }

        @Test
        @DisplayName("emits Error when use case throws NetworkException")
        fun `emits Error when use case throws NetworkException`() = runTest {
            // Given
            coEvery { getUserUseCase(any()) } throws DomainException.NetworkError("timeout")
            val viewModel = createSUT()

            // When
            viewModel.loadUser("123")

            // Then
            viewModel.uiState.test {
                val state = awaitItem()
                assertIs<UserUiState.Error>(state)
                assertEquals("Network error", state.message)
                cancelAndIgnoreRemainingEvents()
            }
        }
    }

    @Nested
    @DisplayName("updateUser")
    inner class UpdateUser {

        @Test
        @DisplayName("updates state to Success after successful update")
        fun `updates state to Success after successful update`() = runTest {
            // Given
            val updatedUser = User.fake(name = "Updated Name")
            coEvery { updateUserUseCase(any()) } returns updatedUser
            val viewModel = createSUT()

            // When
            viewModel.updateUser(updatedUser)

            // Then
            viewModel.uiState.test {
                val state = awaitItem()
                assertIs<UserUiState.Success>(state)
                assertEquals("Updated Name", state.user.name)
                cancelAndIgnoreRemainingEvents()
            }
        }
    }
}
```

### Key Conventions

| Convention | Rule |
|---|---|
| Class naming | `{ClassUnderTest}Test` |
| SUT factory | `private fun createSUT()` always |
| Dependencies | Declared as `private val` at class level |
| Test structure | Given / When / Then comments |
| Nested classes | One `@Nested inner class` per public method |
| Display names | Descriptive, reads like a sentence |
| Backtick names | Mirror the `@DisplayName` text |

---

## 🏗️ Testing Layers

### 1. Testing Entities (Domain Layer)

Entities contain pure business logic. They require **no mocking** -- just instantiate and assert.

```kotlin
class UserTest {

    @Nested
    @DisplayName("displayName")
    inner class DisplayName {

        @Test
        @DisplayName("returns full name when both first and last are present")
        fun `returns full name when both first and last are present`() {
            // Given
            val user = User.fake(firstName = "John", lastName = "Doe")

            // When
            val result = user.displayName

            // Then
            assertEquals("John Doe", result)
        }

        @Test
        @DisplayName("returns first name only when last name is empty")
        fun `returns first name only when last name is empty`() {
            // Given
            val user = User.fake(firstName = "John", lastName = "")

            // When
            val result = user.displayName

            // Then
            assertEquals("John", result)
        }

        @Test
        @DisplayName("returns email as fallback when name is empty")
        fun `returns email as fallback when name is empty`() {
            // Given
            val user = User.fake(firstName = "", lastName = "", email = "john@test.com")

            // When
            val result = user.displayName

            // Then
            assertEquals("john@test.com", result)
        }
    }

    @Nested
    @DisplayName("isActive")
    inner class IsActive {

        @Test
        @DisplayName("returns true when account is not deactivated and not expired")
        fun `returns true when account is not deactivated and not expired`() {
            // Given
            val user = User.fake(
                deactivatedAt = null,
                subscriptionExpiresAt = Clock.System.now() + 30.days,
            )

            // When / Then
            assertTrue(user.isActive)
        }

        @Test
        @DisplayName("returns false when account is deactivated")
        fun `returns false when account is deactivated`() {
            // Given
            val user = User.fake(
                deactivatedAt = Clock.System.now() - 1.days,
            )

            // When / Then
            assertFalse(user.isActive)
        }

        @Test
        @DisplayName("returns false when subscription has expired")
        fun `returns false when subscription has expired`() {
            // Given
            val user = User.fake(
                deactivatedAt = null,
                subscriptionExpiresAt = Clock.System.now() - 1.days,
            )

            // When / Then
            assertFalse(user.isActive)
        }
    }

    @Nested
    @DisplayName("validate")
    inner class Validate {

        @Test
        @DisplayName("returns valid when all fields are correct")
        fun `returns valid when all fields are correct`() {
            // Given
            val user = User.fake(
                email = "john@example.com",
                firstName = "John",
            )

            // When
            val result = user.validate()

            // Then
            assertIs<ValidationResult.Valid>(result)
        }

        @Test
        @DisplayName("returns invalid when email is malformed")
        fun `returns invalid when email is malformed`() {
            // Given
            val user = User.fake(email = "not-an-email")

            // When
            val result = user.validate()

            // Then
            assertIs<ValidationResult.Invalid>(result)
            assertTrue(result.errors.contains(ValidationError.InvalidEmail))
        }

        @Test
        @DisplayName("returns invalid when first name is blank")
        fun `returns invalid when first name is blank`() {
            // Given
            val user = User.fake(firstName = "   ")

            // When
            val result = user.validate()

            // Then
            assertIs<ValidationResult.Invalid>(result)
            assertTrue(result.errors.contains(ValidationError.BlankFirstName))
        }
    }
}
```

### 2. Testing Use Cases (Domain Layer)

Use cases orchestrate business logic and delegate to repository interfaces. They require **mocking of repository dependencies**.

```kotlin
class GetUserUseCaseTest {

    // MARK: - Dependencies

    private val userRepository: UserRepository = mockk()
    private val analyticsTracker: AnalyticsTracker = mockk(relaxed = true)

    // MARK: - SUT Factory

    private fun createSUT() = GetUserUseCase(
        userRepository = userRepository,
        analyticsTracker = analyticsTracker,
    )

    // MARK: - Tests

    @Nested
    @DisplayName("invoke")
    inner class Invoke {

        @Test
        @DisplayName("returns user from repository when found")
        fun `returns user from repository when found`() = runTest {
            // Given
            val expectedUser = User.fake(id = "123")
            coEvery { userRepository.getUser("123") } returns expectedUser
            val sut = createSUT()

            // When
            val result = sut("123")

            // Then
            assertEquals(expectedUser, result)
        }

        @Test
        @DisplayName("tracks analytics event on successful fetch")
        fun `tracks analytics event on successful fetch`() = runTest {
            // Given
            coEvery { userRepository.getUser(any()) } returns User.fake()
            val sut = createSUT()

            // When
            sut("123")

            // Then
            coVerify(exactly = 1) {
                analyticsTracker.track(
                    match { it is AnalyticsEvent.UserProfileViewed }
                )
            }
        }

        @Test
        @DisplayName("throws DomainException.UserNotFound when repository returns null")
        fun `throws UserNotFound when repository returns null`() = runTest {
            // Given
            coEvery { userRepository.getUser("999") } returns null
            val sut = createSUT()

            // When / Then
            val exception = assertThrows<DomainException.UserNotFound> {
                sut("999")
            }
            assertEquals("999", exception.userId)
        }

        @Test
        @DisplayName("maps IOException to DomainException.NetworkError")
        fun `maps IOException to NetworkError`() = runTest {
            // Given
            coEvery { userRepository.getUser(any()) } throws IOException("timeout")
            val sut = createSUT()

            // When / Then
            assertThrows<DomainException.NetworkError> {
                sut("123")
            }
        }

        @Test
        @DisplayName("does not track analytics on failure")
        fun `does not track analytics on failure`() = runTest {
            // Given
            coEvery { userRepository.getUser(any()) } throws IOException("timeout")
            val sut = createSUT()

            // When
            runCatching { sut("123") }

            // Then
            coVerify(exactly = 0) { analyticsTracker.track(any()) }
        }

        @Test
        @DisplayName("throws IllegalArgumentException for blank userId")
        fun `throws IllegalArgumentException for blank userId`() = runTest {
            // Given
            val sut = createSUT()

            // When / Then
            assertThrows<IllegalArgumentException> {
                sut("")
            }
        }
    }
}
```

```kotlin
class SearchRestaurantsUseCaseTest {

    private val restaurantRepository: RestaurantRepository = mockk()
    private val locationService: LocationService = mockk()

    private fun createSUT() = SearchRestaurantsUseCase(
        restaurantRepository = restaurantRepository,
        locationService = locationService,
    )

    @Nested
    @DisplayName("invoke")
    inner class Invoke {

        @Test
        @DisplayName("returns restaurants sorted by distance when location available")
        fun `returns restaurants sorted by distance when location available`() = runTest {
            // Given
            val userLocation = Location.fake(latitude = 40.7128, longitude = -74.0060)
            val restaurants = listOf(
                Restaurant.fake(name = "Far Place", distance = 5.0),
                Restaurant.fake(name = "Near Place", distance = 1.0),
                Restaurant.fake(name = "Mid Place", distance = 3.0),
            )
            coEvery { locationService.getCurrentLocation() } returns userLocation
            coEvery { restaurantRepository.search(any(), any()) } returns restaurants
            val sut = createSUT()

            // When
            val result = sut(SearchQuery(term = "pizza"))

            // Then
            assertEquals("Near Place", result[0].name)
            assertEquals("Mid Place", result[1].name)
            assertEquals("Far Place", result[2].name)
        }

        @Test
        @DisplayName("returns restaurants without sorting when location unavailable")
        fun `returns restaurants without sorting when location unavailable`() = runTest {
            // Given
            coEvery { locationService.getCurrentLocation() } returns null
            coEvery { restaurantRepository.search(any(), isNull()) } returns listOf(
                Restaurant.fake(name = "First"),
                Restaurant.fake(name = "Second"),
            )
            val sut = createSUT()

            // When
            val result = sut(SearchQuery(term = "pizza"))

            // Then
            assertEquals(2, result.size)
            assertEquals("First", result[0].name)
        }

        @Test
        @DisplayName("returns empty list for empty search term")
        fun `returns empty list for empty search term`() = runTest {
            // Given
            val sut = createSUT()

            // When
            val result = sut(SearchQuery(term = ""))

            // Then
            assertTrue(result.isEmpty())
            coVerify(exactly = 0) { restaurantRepository.search(any(), any()) }
        }
    }
}
```

### 3. Testing Repositories (Data Layer)

Repositories coordinate data sources. **Prefer fakes over mocks for data sources** -- fakes provide more realistic behavior and are easier to maintain.

```kotlin
class UserRepositoryImplTest {

    // MARK: - Dependencies

    private val remoteDataSource = FakeUserRemoteDataSource()
    private val localDataSource = FakeUserLocalDataSource()

    // MARK: - SUT Factory

    private fun createSUT() = UserRepositoryImpl(
        remoteDataSource = remoteDataSource,
        localDataSource = localDataSource,
    )

    // MARK: - Tests

    @Nested
    @DisplayName("getUser")
    inner class GetUser {

        @Test
        @DisplayName("returns cached user when available and not expired")
        fun `returns cached user when available and not expired`() = runTest {
            // Given
            val cachedUser = UserEntity.fake(id = "123", cachedAt = Clock.System.now())
            localDataSource.insertUser(cachedUser)
            val sut = createSUT()

            // When
            val result = sut.getUser("123")

            // Then
            assertEquals("123", result?.id)
            assertEquals(0, remoteDataSource.fetchCount)
        }

        @Test
        @DisplayName("fetches from remote when cache is expired")
        fun `fetches from remote when cache is expired`() = runTest {
            // Given
            val expiredUser = UserEntity.fake(
                id = "123",
                cachedAt = Clock.System.now() - 2.hours,
            )
            localDataSource.insertUser(expiredUser)
            remoteDataSource.addUser(UserDto.fake(id = "123", name = "Updated"))
            val sut = createSUT()

            // When
            val result = sut.getUser("123")

            // Then
            assertEquals("Updated", result?.name)
            assertEquals(1, remoteDataSource.fetchCount)
        }

        @Test
        @DisplayName("fetches from remote when cache is empty")
        fun `fetches from remote when cache is empty`() = runTest {
            // Given
            remoteDataSource.addUser(UserDto.fake(id = "123"))
            val sut = createSUT()

            // When
            val result = sut.getUser("123")

            // Then
            assertEquals("123", result?.id)
            assertEquals(1, remoteDataSource.fetchCount)
        }

        @Test
        @DisplayName("caches remote result locally after fetch")
        fun `caches remote result locally after fetch`() = runTest {
            // Given
            remoteDataSource.addUser(UserDto.fake(id = "123"))
            val sut = createSUT()

            // When
            sut.getUser("123")

            // Then
            val cached = localDataSource.getUser("123")
            assertNotNull(cached)
            assertEquals("123", cached.id)
        }

        @Test
        @DisplayName("returns cached user when remote fails")
        fun `returns cached user when remote fails`() = runTest {
            // Given
            val cachedUser = UserEntity.fake(
                id = "123",
                cachedAt = Clock.System.now() - 2.hours,
            )
            localDataSource.insertUser(cachedUser)
            remoteDataSource.shouldThrow = IOException("Network error")
            val sut = createSUT()

            // When
            val result = sut.getUser("123")

            // Then
            assertNotNull(result)
            assertEquals("123", result.id)
        }

        @Test
        @DisplayName("throws when both remote and cache fail")
        fun `throws when both remote and cache fail`() = runTest {
            // Given
            remoteDataSource.shouldThrow = IOException("Network error")
            val sut = createSUT()

            // When / Then
            assertThrows<IOException> {
                sut.getUser("nonexistent")
            }
        }
    }

    @Nested
    @DisplayName("observeUsers")
    inner class ObserveUsers {

        @Test
        @DisplayName("emits updated list when local data changes")
        fun `emits updated list when local data changes`() = runTest {
            // Given
            val sut = createSUT()

            // When / Then
            sut.observeUsers().test {
                assertEquals(emptyList(), awaitItem())

                localDataSource.insertUser(UserEntity.fake(id = "1"))
                assertEquals(1, awaitItem().size)

                localDataSource.insertUser(UserEntity.fake(id = "2"))
                assertEquals(2, awaitItem().size)

                cancelAndIgnoreRemainingEvents()
            }
        }
    }
}
```

**Fake Data Source Implementation:**

```kotlin
class FakeUserRemoteDataSource : UserRemoteDataSource {

    private val users = mutableMapOf<String, UserDto>()
    var fetchCount = 0
        private set
    var shouldThrow: Exception? = null

    override suspend fun fetchUser(userId: String): UserDto {
        shouldThrow?.let { throw it }
        fetchCount++
        return users[userId] ?: throw HttpException(404, "Not found")
    }

    override suspend fun fetchUsers(): List<UserDto> {
        shouldThrow?.let { throw it }
        fetchCount++
        return users.values.toList()
    }

    fun addUser(dto: UserDto) {
        users[dto.id] = dto
    }

    fun clear() {
        users.clear()
        fetchCount = 0
        shouldThrow = null
    }
}
```

```kotlin
class FakeUserLocalDataSource : UserLocalDataSource {

    private val users = mutableMapOf<String, UserEntity>()
    private val usersFlow = MutableStateFlow<List<UserEntity>>(emptyList())

    override suspend fun getUser(userId: String): UserEntity? = users[userId]

    override fun observeUsers(): Flow<List<UserEntity>> = usersFlow

    override suspend fun insertUser(entity: UserEntity) {
        users[entity.id] = entity
        usersFlow.value = users.values.toList()
    }

    override suspend fun deleteUser(userId: String) {
        users.remove(userId)
        usersFlow.value = users.values.toList()
    }

    override suspend fun clear() {
        users.clear()
        usersFlow.value = emptyList()
    }
}
```

### 4. Testing ViewModels (Presentation Layer)

ViewModels are the most critical layer to test. They manage UI state, handle user actions, and coordinate with use cases. Use **Turbine** for StateFlow/SharedFlow testing and **`runTest`** for coroutine support.

```kotlin
class HomeViewModelTest {

    // MARK: - Test Dispatcher Setup

    private val testDispatcher = UnconfinedTestDispatcher()

    @BeforeEach
    fun setup() {
        Dispatchers.setMain(testDispatcher)
    }

    @AfterEach
    fun tearDown() {
        Dispatchers.resetMain()
    }

    // MARK: - Dependencies

    private val getRestaurantsUseCase: GetRestaurantsUseCase = mockk()
    private val toggleFavoriteUseCase: ToggleFavoriteUseCase = mockk()

    // MARK: - SUT Factory

    private fun createSUT() = HomeViewModel(
        getRestaurantsUseCase = getRestaurantsUseCase,
        toggleFavoriteUseCase = toggleFavoriteUseCase,
    )

    // MARK: - Tests

    @Nested
    @DisplayName("init")
    inner class Init {

        @Test
        @DisplayName("initial state is Loading")
        fun `initial state is Loading`() = runTest {
            // Given
            coEvery { getRestaurantsUseCase() } returns emptyList()

            // When
            val viewModel = createSUT()

            // Then
            viewModel.uiState.test {
                // The first emission depends on your ViewModel implementation.
                // If loading starts in init, the first item may already be Loading.
                val state = awaitItem()
                assertIs<HomeUiState.Loading>(state)
                cancelAndIgnoreRemainingEvents()
            }
        }
    }

    @Nested
    @DisplayName("loadRestaurants")
    inner class LoadRestaurants {

        @Test
        @DisplayName("emits Success with restaurant list")
        fun `emits Success with restaurant list`() = runTest {
            // Given
            val restaurants = listOf(
                Restaurant.fake(name = "Pizza Place"),
                Restaurant.fake(name = "Burger Joint"),
            )
            coEvery { getRestaurantsUseCase() } returns restaurants
            val viewModel = createSUT()

            // When
            viewModel.loadRestaurants()

            // Then
            viewModel.uiState.test {
                val state = awaitItem()
                assertIs<HomeUiState.Success>(state)
                assertEquals(2, state.restaurants.size)
                assertEquals("Pizza Place", state.restaurants[0].name)
                cancelAndIgnoreRemainingEvents()
            }
        }

        @Test
        @DisplayName("emits Empty when no restaurants found")
        fun `emits Empty when no restaurants found`() = runTest {
            // Given
            coEvery { getRestaurantsUseCase() } returns emptyList()
            val viewModel = createSUT()

            // When
            viewModel.loadRestaurants()

            // Then
            viewModel.uiState.test {
                assertIs<HomeUiState.Empty>(awaitItem())
                cancelAndIgnoreRemainingEvents()
            }
        }

        @Test
        @DisplayName("emits Error with user-friendly message on failure")
        fun `emits Error with user-friendly message on failure`() = runTest {
            // Given
            coEvery { getRestaurantsUseCase() } throws DomainException.NetworkError("timeout")
            val viewModel = createSUT()

            // When
            viewModel.loadRestaurants()

            // Then
            viewModel.uiState.test {
                val state = awaitItem()
                assertIs<HomeUiState.Error>(state)
                assertTrue(state.message.isNotBlank())
                cancelAndIgnoreRemainingEvents()
            }
        }
    }

    @Nested
    @DisplayName("toggleFavorite")
    inner class ToggleFavorite {

        @Test
        @DisplayName("emits ShowFavoriteAdded event when adding favorite")
        fun `emits ShowFavoriteAdded event when adding favorite`() = runTest {
            // Given
            val restaurant = Restaurant.fake(id = "r1", isFavorite = false)
            coEvery { toggleFavoriteUseCase(any()) } returns restaurant.copy(isFavorite = true)
            coEvery { getRestaurantsUseCase() } returns listOf(restaurant)
            val viewModel = createSUT()

            // When
            viewModel.events.test {
                viewModel.toggleFavorite("r1")

                // Then
                val event = awaitItem()
                assertIs<HomeEvent.ShowFavoriteAdded>(event)
                assertEquals("r1", event.restaurantId)
            }
        }

        @Test
        @DisplayName("emits ShowError event when toggle fails")
        fun `emits ShowError event when toggle fails`() = runTest {
            // Given
            coEvery { toggleFavoriteUseCase(any()) } throws DomainException.NetworkError("fail")
            coEvery { getRestaurantsUseCase() } returns emptyList()
            val viewModel = createSUT()

            // When
            viewModel.events.test {
                viewModel.toggleFavorite("r1")

                // Then
                assertIs<HomeEvent.ShowError>(awaitItem())
            }
        }
    }

    @Nested
    @DisplayName("retry")
    inner class Retry {

        @Test
        @DisplayName("reloads restaurants when retry is invoked")
        fun `reloads restaurants when retry is invoked`() = runTest {
            // Given
            coEvery { getRestaurantsUseCase() } throws DomainException.NetworkError("fail")
            val viewModel = createSUT()
            viewModel.loadRestaurants()

            // Reset to success
            val restaurants = listOf(Restaurant.fake())
            coEvery { getRestaurantsUseCase() } returns restaurants

            // When
            viewModel.retry()

            // Then
            viewModel.uiState.test {
                val state = awaitItem()
                assertIs<HomeUiState.Success>(state)
                cancelAndIgnoreRemainingEvents()
            }
        }
    }
}
```

### 5. Testing Composables (UI Layer)

Compose UI tests verify that composables render correctly and respond to user interactions. Use `ComposeTestRule` and semantic matchers.

```kotlin
import androidx.compose.ui.test.assertIsDisplayed
import androidx.compose.ui.test.assertDoesNotExist
import androidx.compose.ui.test.junit4.createComposeRule
import androidx.compose.ui.test.onNodeWithContentDescription
import androidx.compose.ui.test.onNodeWithTag
import androidx.compose.ui.test.onNodeWithText
import androidx.compose.ui.test.performClick
import androidx.compose.ui.test.performTextInput
import org.junit.Rule
import org.junit.Test

class HomeScreenTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun `shows loading indicator when state is Loading`() {
        // Given / When
        composeTestRule.setContent {
            HomeContent(
                uiState = HomeUiState.Loading,
                onRetry = {},
                onRestaurantClick = {},
                onFavoriteToggle = {},
            )
        }

        // Then
        composeTestRule.onNodeWithTag("loading_indicator").assertIsDisplayed()
        composeTestRule.onNodeWithTag("restaurant_list").assertDoesNotExist()
        composeTestRule.onNodeWithTag("error_view").assertDoesNotExist()
    }

    @Test
    fun `shows restaurant names when state is Success`() {
        // Given
        val restaurants = listOf(
            Restaurant.fake(name = "Pizza Place"),
            Restaurant.fake(name = "Sushi Bar"),
            Restaurant.fake(name = "Taco Stand"),
        )

        // When
        composeTestRule.setContent {
            HomeContent(
                uiState = HomeUiState.Success(restaurants),
                onRetry = {},
                onRestaurantClick = {},
                onFavoriteToggle = {},
            )
        }

        // Then
        composeTestRule.onNodeWithText("Pizza Place").assertIsDisplayed()
        composeTestRule.onNodeWithText("Sushi Bar").assertIsDisplayed()
        composeTestRule.onNodeWithText("Taco Stand").assertIsDisplayed()
    }

    @Test
    fun `shows error message and retry button when state is Error`() {
        // Given / When
        composeTestRule.setContent {
            HomeContent(
                uiState = HomeUiState.Error("Something went wrong"),
                onRetry = {},
                onRestaurantClick = {},
                onFavoriteToggle = {},
            )
        }

        // Then
        composeTestRule.onNodeWithText("Something went wrong").assertIsDisplayed()
        composeTestRule.onNodeWithText("Retry").assertIsDisplayed()
    }

    @Test
    fun `calls onRetry when retry button is clicked`() {
        // Given
        var retryCalled = false
        composeTestRule.setContent {
            HomeContent(
                uiState = HomeUiState.Error("Error"),
                onRetry = { retryCalled = true },
                onRestaurantClick = {},
                onFavoriteToggle = {},
            )
        }

        // When
        composeTestRule.onNodeWithText("Retry").performClick()

        // Then
        assertTrue(retryCalled)
    }

    @Test
    fun `calls onRestaurantClick with correct ID when restaurant is tapped`() {
        // Given
        var clickedId: String? = null
        val restaurants = listOf(Restaurant.fake(id = "r1", name = "Pizza Place"))

        composeTestRule.setContent {
            HomeContent(
                uiState = HomeUiState.Success(restaurants),
                onRetry = {},
                onRestaurantClick = { clickedId = it },
                onFavoriteToggle = {},
            )
        }

        // When
        composeTestRule.onNodeWithText("Pizza Place").performClick()

        // Then
        assertEquals("r1", clickedId)
    }

    @Test
    fun `shows empty state when no restaurants available`() {
        // Given / When
        composeTestRule.setContent {
            HomeContent(
                uiState = HomeUiState.Empty,
                onRetry = {},
                onRestaurantClick = {},
                onFavoriteToggle = {},
            )
        }

        // Then
        composeTestRule.onNodeWithText("No restaurants found").assertIsDisplayed()
        composeTestRule.onNodeWithContentDescription("Empty state illustration").assertIsDisplayed()
    }
}
```

**Search Bar Compose Test:**

```kotlin
class SearchBarTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun `displays placeholder text when empty`() {
        composeTestRule.setContent {
            SearchBar(
                query = "",
                onQueryChange = {},
                onSearch = {},
            )
        }

        composeTestRule.onNodeWithText("Search restaurants...").assertIsDisplayed()
    }

    @Test
    fun `calls onQueryChange when text is entered`() {
        // Given
        var capturedQuery = ""
        composeTestRule.setContent {
            SearchBar(
                query = "",
                onQueryChange = { capturedQuery = it },
                onSearch = {},
            )
        }

        // When
        composeTestRule.onNodeWithTag("search_input").performTextInput("pizza")

        // Then
        assertEquals("pizza", capturedQuery)
    }

    @Test
    fun `shows clear button when query is not empty`() {
        composeTestRule.setContent {
            SearchBar(
                query = "pizza",
                onQueryChange = {},
                onSearch = {},
            )
        }

        composeTestRule.onNodeWithContentDescription("Clear search").assertIsDisplayed()
    }

    @Test
    fun `hides clear button when query is empty`() {
        composeTestRule.setContent {
            SearchBar(
                query = "",
                onQueryChange = {},
                onSearch = {},
            )
        }

        composeTestRule.onNodeWithContentDescription("Clear search").assertDoesNotExist()
    }
}
```

---

## 🎭 MockK Patterns

MockK is the standard mocking library for Kotlin. It provides first-class support for coroutines, extension functions, and Kotlin-specific constructs.

### Basic Mocking

```kotlin
// Create a mock
val repository: UserRepository = mockk()

// Stub a suspend function
coEvery { repository.getUser("123") } returns User.fake()

// Stub a regular function returning a Flow
every { repository.observeUsers() } returns flowOf(listOf(User.fake()))

// Stub a function to throw
coEvery { repository.getUser("999") } throws DomainException.UserNotFound("999")

// Stub a Unit-returning function
coEvery { repository.deleteUser(any()) } just Runs

// Stub with sequential returns
coEvery { repository.getUser(any()) } returnsMany listOf(
    User.fake(name = "First Call"),
    User.fake(name = "Second Call"),
)
```

### Verification

```kotlin
// Verify exact call count
coVerify(exactly = 1) { repository.getUser("123") }

// Verify never called
coVerify(exactly = 0) { repository.deleteUser(any()) }

// Verify with argument matcher
coVerify { repository.updateUser(match { it.name == "John" }) }

// Verify call order
coVerifyOrder {
    repository.getUser("123")
    repository.updateUser(any())
}

// Verify all specified calls happened (in any order)
coVerifyAll {
    repository.getUser("123")
    analyticsTracker.track(any())
}
```

### Relaxed Mocks

```kotlin
// All functions return default values without explicit stubbing
val repository: UserRepository = mockk(relaxed = true)

// Only Unit-returning functions are relaxed
val repository: UserRepository = mockk(relaxUnitFun = true)
```

> **Warning:** Use `relaxed = true` sparingly. It hides missing stubs and can mask test failures. Prefer explicit stubbing.

### Argument Capture

```kotlin
@Test
@DisplayName("passes correctly mapped user to repository")
fun `passes correctly mapped user to repository`() = runTest {
    // Given
    val userSlot = slot<User>()
    coEvery { repository.updateUser(capture(userSlot)) } just Runs
    val sut = createSUT()

    // When
    sut.updateUser(UserUpdateRequest(name = "Updated", email = "new@test.com"))

    // Then
    assertEquals("Updated", userSlot.captured.name)
    assertEquals("new@test.com", userSlot.captured.email)
}
```

```kotlin
@Test
@DisplayName("captures all analytics events during operation")
fun `captures all analytics events during operation`() = runTest {
    // Given
    val events = mutableListOf<AnalyticsEvent>()
    every { tracker.track(capture(events)) } just Runs
    val sut = createSUT()

    // When
    sut.performComplexOperation()

    // Then
    assertEquals(3, events.size)
    assertIs<AnalyticsEvent.OperationStarted>(events[0])
    assertIs<AnalyticsEvent.StepCompleted>(events[1])
    assertIs<AnalyticsEvent.OperationCompleted>(events[2])
}
```

### Mocking Extension Functions

```kotlin
// Top-level extension function mocking
mockkStatic("com.arclabs.util.StringExtKt")
every { "hello".capitalize() } returns "Hello"

// Object mocking
mockkObject(DateFormatter)
every { DateFormatter.format(any()) } returns "Jan 1, 2025"
```

### MockK Best Practices

| Practice | Status |
|---|---|
| Use `coEvery`/`coVerify` for suspend functions | ALWAYS |
| Use `every`/`verify` for regular functions | ALWAYS |
| Explicit stubbing over relaxed mocks | ALWAYS |
| Mock interfaces, not concrete classes | ALWAYS |
| Use `relaxed = true` sparingly | CAUTION |
| Mock data classes | NEVER |
| Mock what you own (use fakes for simple types) | AVOID |
| Mock static/top-level functions | LAST RESORT |

---

## 🌊 Turbine (Flow Testing)

[Turbine](https://github.com/cashapp/turbine) is the standard library for testing Kotlin Flows. It provides a clean API for asserting emissions, errors, and completion.

### Testing StateFlow

```kotlin
@Test
@DisplayName("emits states in correct order during data load")
fun `emits states in correct order during data load`() = runTest {
    // Given
    val restaurants = listOf(Restaurant.fake())
    coEvery { getRestaurantsUseCase() } returns restaurants
    val viewModel = createSUT()

    // When / Then
    viewModel.uiState.test {
        // Initial state
        assertIs<HomeUiState.Loading>(awaitItem())

        // Trigger load
        viewModel.loadRestaurants()

        // Final state
        val successState = awaitItem()
        assertIs<HomeUiState.Success>(successState)
        assertEquals(1, successState.restaurants.size)

        cancelAndIgnoreRemainingEvents()
    }
}
```

### Testing SharedFlow (One-Shot Events)

```kotlin
@Test
@DisplayName("emits navigation event when item is clicked")
fun `emits navigation event when item is clicked`() = runTest {
    // Given
    val viewModel = createSUT()

    // When / Then
    viewModel.events.test {
        viewModel.onItemClicked("123")

        val event = awaitItem()
        assertEquals(HomeEvent.NavigateToDetail("123"), event)
    }
}
```

### Testing Multiple Flow Emissions

```kotlin
@Test
@DisplayName("emits all intermediate states during complex operation")
fun `emits all intermediate states during complex operation`() = runTest {
    // Given
    val viewModel = createSUT()

    // When / Then
    viewModel.uiState.test {
        assertIs<HomeUiState.Idle>(awaitItem())

        viewModel.startSync()

        assertIs<HomeUiState.Syncing>(awaitItem())
        assertIs<HomeUiState.SyncProgress>(awaitItem())
        assertIs<HomeUiState.Success>(awaitItem())

        cancelAndIgnoreRemainingEvents()
    }
}
```

### Testing Flow from Repository

```kotlin
@Test
@DisplayName("repository emits updated user list when data changes")
fun `repository emits updated user list when data changes`() = runTest {
    // Given
    val fakeLocalSource = FakeUserLocalDataSource()
    val sut = UserRepositoryImpl(
        remoteDataSource = mockk(relaxed = true),
        localDataSource = fakeLocalSource,
    )

    // When / Then
    sut.observeUsers().test {
        assertEquals(emptyList(), awaitItem())

        fakeLocalSource.insertUser(UserEntity.fake(id = "1"))
        assertEquals(1, awaitItem().size)

        fakeLocalSource.insertUser(UserEntity.fake(id = "2"))
        assertEquals(2, awaitItem().size)

        fakeLocalSource.deleteUser("1")
        assertEquals(1, awaitItem().size)

        cancelAndIgnoreRemainingEvents()
    }
}
```

### Turbine Timeout Configuration

```kotlin
@Test
@DisplayName("handles slow emissions with custom timeout")
fun `handles slow emissions with custom timeout`() = runTest {
    val viewModel = createSUT()

    viewModel.uiState.test(timeout = 5.seconds) {
        viewModel.performSlowOperation()
        assertIs<HomeUiState.Success>(awaitItem())
        cancelAndIgnoreRemainingEvents()
    }
}
```

### Turbine Best Practices

| Practice | Status |
|---|---|
| Always call `cancelAndIgnoreRemainingEvents()` when done | ALWAYS |
| Use `awaitItem()` for expected emissions | ALWAYS |
| Use `expectNoEvents()` to assert no emissions | WHEN NEEDED |
| Use `awaitError()` for terminal errors | WHEN NEEDED |
| Set custom timeout for slow operations | WHEN NEEDED |
| Avoid `expectMostRecentItem()` in favor of `awaitItem()` | PREFER |

---

## ⚡ Coroutine Testing

### Setup with TestDispatcher

Every ViewModel test class must configure `Dispatchers.Main` before tests run:

```kotlin
class ViewModelTest {

    private val testDispatcher = UnconfinedTestDispatcher()

    @BeforeEach
    fun setup() {
        Dispatchers.setMain(testDispatcher)
    }

    @AfterEach
    fun tearDown() {
        Dispatchers.resetMain()
    }
}
```

> **Why `UnconfinedTestDispatcher`?** It executes coroutines eagerly, making state transitions synchronous and predictable. Use `StandardTestDispatcher` when you need to control coroutine execution timing explicitly.

### UnconfinedTestDispatcher vs StandardTestDispatcher

```kotlin
// UnconfinedTestDispatcher: coroutines run eagerly (most common)
@Test
fun `with unconfined dispatcher`() = runTest(UnconfinedTestDispatcher()) {
    val viewModel = createSUT()
    viewModel.loadData()
    // State is already updated because coroutines ran immediately
    assertIs<UiState.Success>(viewModel.uiState.value)
}

// StandardTestDispatcher: coroutines need explicit advancement
@Test
fun `with standard dispatcher`() = runTest(StandardTestDispatcher()) {
    val viewModel = createSUT()
    viewModel.loadData()
    // Must advance time for coroutines to execute
    advanceUntilIdle()
    assertIs<UiState.Success>(viewModel.uiState.value)
}
```

### Testing Delay and Timeout

```kotlin
@Test
@DisplayName("debounces search input by 300ms")
fun `debounces search input by 300ms`() = runTest {
    // Given
    val viewModel = createSUT()

    // When
    viewModel.onSearchQueryChanged("p")
    advanceTimeBy(100)
    viewModel.onSearchQueryChanged("pi")
    advanceTimeBy(100)
    viewModel.onSearchQueryChanged("piz")
    advanceTimeBy(100)
    viewModel.onSearchQueryChanged("pizza")

    // Then - search should NOT have been triggered yet
    coVerify(exactly = 0) { searchUseCase(any()) }

    // Advance past debounce window
    advanceTimeBy(300)

    // Now search should trigger with final query
    coVerify(exactly = 1) { searchUseCase(match { it.term == "pizza" }) }
}
```

### Testing Retry Logic

```kotlin
@Test
@DisplayName("retries failed request up to 3 times")
fun `retries failed request up to 3 times`() = runTest {
    // Given
    var callCount = 0
    coEvery { repository.fetchData() } answers {
        callCount++
        if (callCount < 3) throw IOException("fail") else Data.fake()
    }
    val sut = createSUT()

    // When
    val result = sut.fetchWithRetry()

    // Then
    assertNotNull(result)
    assertEquals(3, callCount)
}
```

### runTest Block

```kotlin
@Test
@DisplayName("suspend function returns expected result")
fun `suspend function returns expected result`() = runTest {
    // Given
    val sut = createSUT()

    // When
    val result = sut.fetchData()

    // Then
    assertEquals(expected, result)
}
```

---

## 📊 Parameterized Tests

JUnit 5's parameterized tests let you run the same assertion logic with multiple inputs, reducing duplication and improving coverage.

### @ValueSource

```kotlin
@ParameterizedTest
@ValueSource(strings = ["", " ", "   ", "\t", "\n"])
@DisplayName("rejects blank user IDs")
fun `rejects blank user IDs`(userId: String) = runTest {
    // Given
    val sut = createSUT()

    // When / Then
    assertThrows<IllegalArgumentException> {
        sut(userId)
    }
}

@ParameterizedTest
@ValueSource(ints = [0, -1, -100])
@DisplayName("rejects non-positive page numbers")
fun `rejects non-positive page numbers`(page: Int) = runTest {
    // Given
    val sut = createSUT()

    // When / Then
    assertThrows<IllegalArgumentException> {
        sut.loadPage(page)
    }
}
```

### @CsvSource

```kotlin
@ParameterizedTest
@CsvSource(
    "test@example.com, true",
    "user.name@domain.co.uk, true",
    "user+tag@example.com, true",
    "invalid-email, false",
    "@missing-local.com, false",
    "user@, false",
    "'', false",
    "user@domain, false",
)
@DisplayName("validates email addresses correctly")
fun `validates email addresses correctly`(email: String, expected: Boolean) {
    // Given
    val validator = EmailValidator()

    // When
    val result = validator.isValid(email)

    // Then
    assertEquals(expected, result, "Expected isValid('$email') to be $expected")
}
```

### @MethodSource

```kotlin
class PriceFormatterTest {

    @ParameterizedTest
    @MethodSource("priceTestCases")
    @DisplayName("formats prices correctly for different currencies")
    fun `formats prices correctly`(testCase: PriceTestCase) {
        // Given
        val formatter = PriceFormatter(locale = testCase.locale)

        // When
        val result = formatter.format(testCase.amount, testCase.currency)

        // Then
        assertEquals(testCase.expected, result)
    }

    companion object {
        @JvmStatic
        fun priceTestCases() = listOf(
            PriceTestCase(10.99, Currency.USD, Locale.US, "$10.99"),
            PriceTestCase(10.99, Currency.EUR, Locale.GERMANY, "10,99 EUR"),
            PriceTestCase(1000.0, Currency.JPY, Locale.JAPAN, "JPY 1,000"),
            PriceTestCase(0.0, Currency.USD, Locale.US, "$0.00"),
            PriceTestCase(999999.99, Currency.USD, Locale.US, "$999,999.99"),
        )
    }

    data class PriceTestCase(
        val amount: Double,
        val currency: Currency,
        val locale: Locale,
        val expected: String,
    )
}
```

### @EnumSource

```kotlin
@ParameterizedTest
@EnumSource(CuisineType::class)
@DisplayName("maps all cuisine types to display strings without throwing")
fun `maps all cuisine types to display strings`(cuisineType: CuisineType) {
    // Given
    val mapper = CuisineDisplayMapper()

    // When
    val result = mapper.toDisplayString(cuisineType)

    // Then
    assertTrue(result.isNotBlank(), "Display string for $cuisineType should not be blank")
}

@ParameterizedTest
@EnumSource(value = PriceRange::class, names = ["BUDGET", "MODERATE"])
@DisplayName("shows dollar signs for affordable price ranges")
fun `shows dollar signs for affordable price ranges`(priceRange: PriceRange) {
    val result = PriceRangeFormatter.format(priceRange)
    assertTrue(result.length <= 2, "Affordable ranges should show at most 2 dollar signs")
}
```

---

## 🔧 Test Helpers & Fakes

### Fake Factories

Every domain entity should have a `fake()` factory on its companion object for use in tests. Fakes use sensible defaults and allow overriding any property.

```kotlin
// User entity fake
fun User.Companion.fake(
    id: String = "user-123",
    email: String = "test@example.com",
    firstName: String = "Test",
    lastName: String = "User",
    avatarUrl: String? = null,
    isActive: Boolean = true,
    createdAt: Instant = Clock.System.now(),
    deactivatedAt: Instant? = null,
    subscriptionExpiresAt: Instant? = null,
) = User(
    id = id,
    email = email,
    firstName = firstName,
    lastName = lastName,
    avatarUrl = avatarUrl,
    isActive = isActive,
    createdAt = createdAt,
    deactivatedAt = deactivatedAt,
    subscriptionExpiresAt = subscriptionExpiresAt,
)

// Restaurant entity fake
fun Restaurant.Companion.fake(
    id: String = "restaurant-456",
    name: String = "Test Restaurant",
    cuisineType: CuisineType = CuisineType.ITALIAN,
    location: Location = Location.fake(),
    rating: Double = 4.5,
    priceRange: PriceRange = PriceRange.MODERATE,
    isFavorite: Boolean = false,
    distance: Double? = null,
) = Restaurant(
    id = id,
    name = name,
    cuisineType = cuisineType,
    location = location,
    rating = rating,
    priceRange = priceRange,
    isFavorite = isFavorite,
    distance = distance,
)

// Location value object fake
fun Location.Companion.fake(
    latitude: Double = 40.7128,
    longitude: Double = -74.0060,
    address: String = "123 Test Street",
    city: String = "New York",
    state: String = "NY",
    zipCode: String = "10001",
) = Location(
    latitude = latitude,
    longitude = longitude,
    address = address,
    city = city,
    state = state,
    zipCode = zipCode,
)

// DTO fakes
fun UserDto.Companion.fake(
    id: String = "user-123",
    name: String = "Test User",
    email: String = "test@example.com",
    avatarUrl: String? = null,
    createdAt: String = "2024-01-01T00:00:00Z",
) = UserDto(
    id = id,
    name = name,
    email = email,
    avatarUrl = avatarUrl,
    createdAt = createdAt,
)

// Entity (Room) fakes
fun UserEntity.Companion.fake(
    id: String = "user-123",
    name: String = "Test User",
    email: String = "test@example.com",
    cachedAt: Instant = Clock.System.now(),
) = UserEntity(
    id = id,
    name = name,
    email = email,
    cachedAt = cachedAt.toEpochMilliseconds(),
)
```

### Fake Repository Pattern

Fakes provide more realistic behavior than mocks for the data layer. They maintain internal state and can simulate complex scenarios.

```kotlin
class FakeUserRepository : UserRepository {

    private val users = mutableListOf<User>()
    private val usersFlow = MutableStateFlow<List<User>>(emptyList())
    private val favorites = mutableSetOf<String>()

    var shouldThrow: Exception? = null
    var getCallCount = 0
        private set
    var updateCallCount = 0
        private set

    override suspend fun getUser(userId: String): User? {
        shouldThrow?.let { throw it }
        getCallCount++
        return users.find { it.id == userId }
    }

    override suspend fun getUsers(): List<User> {
        shouldThrow?.let { throw it }
        return users.toList()
    }

    override fun observeUsers(): Flow<List<User>> = usersFlow

    override suspend fun updateUser(user: User) {
        shouldThrow?.let { throw it }
        updateCallCount++
        val index = users.indexOfFirst { it.id == user.id }
        if (index >= 0) {
            users[index] = user
        } else {
            users.add(user)
        }
        usersFlow.value = users.toList()
    }

    override suspend fun deleteUser(userId: String) {
        shouldThrow?.let { throw it }
        users.removeAll { it.id == userId }
        usersFlow.value = users.toList()
    }

    override suspend fun toggleFavorite(restaurantId: String): Boolean {
        shouldThrow?.let { throw it }
        return if (favorites.contains(restaurantId)) {
            favorites.remove(restaurantId)
            false
        } else {
            favorites.add(restaurantId)
            true
        }
    }

    // Test helpers

    fun addUser(user: User) {
        users.add(user)
        usersFlow.value = users.toList()
    }

    fun addUsers(vararg newUsers: User) {
        users.addAll(newUsers)
        usersFlow.value = users.toList()
    }

    fun clear() {
        users.clear()
        favorites.clear()
        usersFlow.value = emptyList()
        shouldThrow = null
        getCallCount = 0
        updateCallCount = 0
    }
}
```

### Test Extension Functions

```kotlin
/**
 * Asserts that a Flow emits exactly the expected items in order.
 */
suspend fun <T> Flow<T>.assertEmissions(vararg expected: T) {
    test {
        expected.forEach { expectedItem ->
            assertEquals(expectedItem, awaitItem())
        }
        cancelAndIgnoreRemainingEvents()
    }
}

/**
 * Collects all items from a Flow within a test scope.
 */
suspend fun <T> Flow<T>.collectAll(scope: TestScope): List<T> {
    val items = mutableListOf<T>()
    val job = scope.backgroundScope.launch {
        collect { items.add(it) }
    }
    scope.advanceUntilIdle()
    job.cancel()
    return items
}
```

---

## 📱 Room DAO Testing

Room DAOs should be tested with an in-memory database. These are instrumented tests that run on a device or emulator.

### Basic DAO Test Setup

```kotlin
import android.content.Context
import androidx.room.Room
import androidx.test.core.app.ApplicationProvider
import kotlinx.coroutines.test.runTest
import org.junit.jupiter.api.AfterEach
import org.junit.jupiter.api.BeforeEach
import org.junit.jupiter.api.DisplayName
import org.junit.jupiter.api.Nested
import org.junit.jupiter.api.Test
import kotlin.test.assertEquals
import kotlin.test.assertNotNull
import kotlin.test.assertNull

class UserDaoTest {

    private lateinit var database: AppDatabase
    private lateinit var userDao: UserDao

    @BeforeEach
    fun setup() {
        val context = ApplicationProvider.getApplicationContext<Context>()
        database = Room.inMemoryDatabaseBuilder(
            context,
            AppDatabase::class.java,
        ).allowMainThreadQueries().build()

        userDao = database.userDao()
    }

    @AfterEach
    fun tearDown() {
        database.close()
    }

    @Nested
    @DisplayName("insertUser")
    inner class InsertUser {

        @Test
        @DisplayName("inserts and retrieves a user by ID")
        fun `inserts and retrieves a user by ID`() = runTest {
            // Given
            val entity = UserEntity.fake(id = "user-1", name = "John Doe")

            // When
            userDao.insertUser(entity)
            val result = userDao.getUser("user-1")

            // Then
            assertNotNull(result)
            assertEquals("John Doe", result.name)
        }

        @Test
        @DisplayName("replaces existing user on conflict")
        fun `replaces existing user on conflict`() = runTest {
            // Given
            val original = UserEntity.fake(id = "user-1", name = "Original")
            val updated = UserEntity.fake(id = "user-1", name = "Updated")

            // When
            userDao.insertUser(original)
            userDao.insertUser(updated)
            val result = userDao.getUser("user-1")

            // Then
            assertNotNull(result)
            assertEquals("Updated", result.name)
        }
    }

    @Nested
    @DisplayName("deleteUser")
    inner class DeleteUser {

        @Test
        @DisplayName("removes user from database")
        fun `removes user from database`() = runTest {
            // Given
            val entity = UserEntity.fake(id = "user-1")
            userDao.insertUser(entity)

            // When
            userDao.deleteUser("user-1")

            // Then
            assertNull(userDao.getUser("user-1"))
        }

        @Test
        @DisplayName("does not throw when deleting nonexistent user")
        fun `does not throw when deleting nonexistent user`() = runTest {
            // When / Then (no exception)
            userDao.deleteUser("nonexistent")
        }
    }

    @Nested
    @DisplayName("observeUsers")
    inner class ObserveUsers {

        @Test
        @DisplayName("emits updated list when users change")
        fun `emits updated list when users change`() = runTest {
            userDao.observeUsers().test {
                // Initial emission
                assertEquals(emptyList(), awaitItem())

                // Insert first user
                userDao.insertUser(UserEntity.fake(id = "1", name = "Alice"))
                val afterFirst = awaitItem()
                assertEquals(1, afterFirst.size)
                assertEquals("Alice", afterFirst[0].name)

                // Insert second user
                userDao.insertUser(UserEntity.fake(id = "2", name = "Bob"))
                val afterSecond = awaitItem()
                assertEquals(2, afterSecond.size)

                // Delete first user
                userDao.deleteUser("1")
                val afterDelete = awaitItem()
                assertEquals(1, afterDelete.size)
                assertEquals("Bob", afterDelete[0].name)

                cancelAndIgnoreRemainingEvents()
            }
        }
    }

    @Nested
    @DisplayName("getUsersByCity")
    inner class GetUsersByCity {

        @Test
        @DisplayName("returns only users in the specified city")
        fun `returns only users in the specified city`() = runTest {
            // Given
            userDao.insertUser(UserEntity.fake(id = "1", city = "New York"))
            userDao.insertUser(UserEntity.fake(id = "2", city = "Los Angeles"))
            userDao.insertUser(UserEntity.fake(id = "3", city = "New York"))

            // When
            val result = userDao.getUsersByCity("New York")

            // Then
            assertEquals(2, result.size)
            assertTrue(result.all { it.city == "New York" })
        }

        @Test
        @DisplayName("returns empty list for city with no users")
        fun `returns empty list for city with no users`() = runTest {
            // Given
            userDao.insertUser(UserEntity.fake(id = "1", city = "New York"))

            // When
            val result = userDao.getUsersByCity("Chicago")

            // Then
            assertTrue(result.isEmpty())
        }
    }
}
```

### Testing Database Migrations

```kotlin
class MigrationTest {

    @get:Rule
    val helper = MigrationTestHelper(
        InstrumentationRegistry.getInstrumentation(),
        AppDatabase::class.java,
    )

    @Test
    fun `migration from v1 to v2 preserves user data`() {
        // Create database at version 1
        helper.createDatabase(TEST_DB_NAME, 1).apply {
            execSQL("INSERT INTO users (id, name, email) VALUES ('1', 'John', 'john@test.com')")
            close()
        }

        // Run migration to version 2
        val db = helper.runMigrationsAndValidate(TEST_DB_NAME, 2, true, Migration1To2)

        // Verify data preserved with new column default
        val cursor = db.query("SELECT * FROM users WHERE id = '1'")
        assertTrue(cursor.moveToFirst())
        assertEquals("John", cursor.getString(cursor.getColumnIndex("name")))
        // New column should have default value
        assertEquals("", cursor.getString(cursor.getColumnIndex("avatar_url")))
    }

    companion object {
        private const val TEST_DB_NAME = "migration-test"
    }
}
```

---

## 📋 Testing Checklist

Use this checklist before marking any task as complete.

### Coverage

- [ ] All public functions have at least one test
- [ ] Happy path is tested for every use case
- [ ] Error/failure paths are tested
- [ ] Edge cases are identified and tested (empty lists, null values, boundary conditions)
- [ ] Coverage meets or exceeds the minimum threshold for the project type
- [ ] Branch coverage is within 10% of line coverage
- [ ] No untested branches in critical business logic

### Quality

- [ ] Every test follows the Given/When/Then structure
- [ ] Every test class uses `createSUT()` factory
- [ ] Tests are independent (no shared mutable state between tests)
- [ ] Tests are deterministic (no flaky tests due to timing, ordering, or randomness)
- [ ] Test names clearly describe the behavior being verified
- [ ] `@Nested` classes group tests by method or feature
- [ ] `@DisplayName` annotations provide human-readable descriptions
- [ ] No logic in tests (no if/else, loops, or complex setup)
- [ ] Each test verifies one specific behavior

### Mocks & Fakes

- [ ] Dependencies are injected via constructor (not created internally)
- [ ] Mocks are used for interfaces, not concrete classes
- [ ] Fakes are preferred over mocks for data layer dependencies
- [ ] `coEvery`/`coVerify` used for suspend functions (not `every`/`verify`)
- [ ] Argument matchers are specific (prefer `match {}` over `any()` for verification)
- [ ] Relaxed mocks are justified and documented

### Coroutines

- [ ] `runTest {}` wraps all coroutine test code
- [ ] `Dispatchers.setMain(testDispatcher)` in `@BeforeEach`
- [ ] `Dispatchers.resetMain()` in `@AfterEach`
- [ ] Turbine used for Flow/StateFlow assertions
- [ ] `cancelAndIgnoreRemainingEvents()` called at end of Turbine blocks
- [ ] No `delay()` calls in tests (use `advanceTimeBy()` instead)

---

## 🚫 Common Mistakes

### 1. Testing Implementation Details Instead of Behavior

```kotlin
// WRONG: Tests internal implementation
@Test
fun `test internal cache map has correct size`() {
    val repo = UserRepositoryImpl(remote, local)
    repo.getUser("123")
    assertEquals(1, repo.cacheMap.size) // Leaks implementation detail
}
```

```kotlin
// CORRECT: Tests observable behavior
@Test
fun `returns cached user on second call without fetching remotely`() = runTest {
    // Given
    coEvery { remote.fetchUser("123") } returns UserDto.fake()
    val sut = createSUT()

    // When
    sut.getUser("123")
    sut.getUser("123")

    // Then
    coVerify(exactly = 1) { remote.fetchUser("123") } // Only called once = caching works
}
```

### 2. Shared Mutable State Between Tests

```kotlin
// WRONG: Shared mutable state
class BadTest {
    private val users = mutableListOf<User>() // Shared across tests!
    private val sut = UserRepository(users)   // Created once, shared!

    @Test
    fun `test one`() {
        users.add(User.fake(id = "1"))
        assertEquals(1, sut.getUsers().size)
    }

    @Test
    fun `test two`() {
        // FAILS if test one runs first -- users already has an item!
        assertEquals(0, sut.getUsers().size)
    }
}
```

```kotlin
// CORRECT: Fresh state per test
class GoodTest {
    private fun createSUT() = UserRepository(mutableListOf())

    @Test
    fun `test one`() {
        val sut = createSUT()
        sut.addUser(User.fake(id = "1"))
        assertEquals(1, sut.getUsers().size)
    }

    @Test
    fun `test two`() {
        val sut = createSUT()
        assertEquals(0, sut.getUsers().size) // Always passes
    }
}
```

### 3. Not Using runTest for Coroutines

```kotlin
// WRONG: Using runBlocking (JUnit 4 habit)
@Test
fun `fetch user`() = runBlocking {
    val result = sut.getUser("123")
    assertEquals(expected, result)
}
```

```kotlin
// CORRECT: Using runTest (JUnit 5 + coroutines-test)
@Test
fun `fetch user`() = runTest {
    val result = sut.getUser("123")
    assertEquals(expected, result)
}
```

> `runTest` provides virtual time control, proper exception handling, and integration with `TestDispatcher`. `runBlocking` blocks the thread and does not support `advanceTimeBy()` or `advanceUntilIdle()`.

### 4. Not Resetting Dispatchers.Main After Tests

```kotlin
// WRONG: Missing teardown -- leaks TestDispatcher to other test classes
class ViewModelTest {
    @BeforeEach
    fun setup() {
        Dispatchers.setMain(UnconfinedTestDispatcher())
    }

    // Missing @AfterEach! Other test classes will use this dispatcher.
}
```

```kotlin
// CORRECT: Always reset in @AfterEach
class ViewModelTest {
    private val testDispatcher = UnconfinedTestDispatcher()

    @BeforeEach
    fun setup() {
        Dispatchers.setMain(testDispatcher)
    }

    @AfterEach
    fun tearDown() {
        Dispatchers.resetMain()
    }
}
```

### 5. Testing Multiple Scenarios in One Test

```kotlin
// WRONG: Multiple scenarios in one test
@Test
fun `test user operations`() = runTest {
    // Test creation
    val user = sut.createUser("John")
    assertNotNull(user)

    // Test update
    sut.updateUser(user.copy(name = "Jane"))
    assertEquals("Jane", sut.getUser(user.id)?.name)

    // Test deletion
    sut.deleteUser(user.id)
    assertNull(sut.getUser(user.id))

    // If update fails, we never learn if delete works!
}
```

```kotlin
// CORRECT: One scenario per test
@Test
fun `createUser returns non-null user`() = runTest {
    val user = sut.createUser("John")
    assertNotNull(user)
}

@Test
fun `updateUser changes user name`() = runTest {
    val user = sut.createUser("John")
    sut.updateUser(user.copy(name = "Jane"))
    assertEquals("Jane", sut.getUser(user.id)?.name)
}

@Test
fun `deleteUser removes user from repository`() = runTest {
    val user = sut.createUser("John")
    sut.deleteUser(user.id)
    assertNull(sut.getUser(user.id))
}
```

---

## 🏷️ Test Organization

### Package Structure

```
src/
  main/kotlin/com/arclabs/app/
    domain/
      entity/User.kt
      usecase/GetUserUseCase.kt
    data/
      repository/UserRepositoryImpl.kt
      source/UserRemoteDataSource.kt
    presentation/
      home/HomeViewModel.kt
      home/HomeScreen.kt

  test/kotlin/com/arclabs/app/
    domain/
      entity/UserTest.kt
      usecase/GetUserUseCaseTest.kt
    data/
      repository/UserRepositoryImplTest.kt
    presentation/
      home/HomeViewModelTest.kt
    helpers/
      fakes/FakeUserRepository.kt
      fakes/FakeUserLocalDataSource.kt
      fakes/FakeUserRemoteDataSource.kt
      extensions/FlowTestExtensions.kt
      factories/UserFakes.kt

  androidTest/kotlin/com/arclabs/app/
    data/
      dao/UserDaoTest.kt
    presentation/
      home/HomeScreenTest.kt
```

### Naming Conventions

| Item | Convention | Example |
|---|---|---|
| Test class | `{ClassUnderTest}Test` | `UserProfileViewModelTest` |
| Test function | Backtick with behavior description | `` `emits Success when user found` `` |
| Nested class | Method or feature name | `inner class LoadUser` |
| Fake class | `Fake{Interface}` | `FakeUserRepository` |
| Factory function | `{Type}.Companion.fake()` | `User.fake()` |
| Test helper file | Descriptive name in `helpers/` | `FlowTestExtensions.kt` |

---

## 📐 Test Configuration

### JaCoCo Setup for Coverage

```kotlin
// build.gradle.kts (module)
plugins {
    id("jacoco")
}

android {
    buildTypes {
        debug {
            enableUnitTestCoverage = true
            enableAndroidTestCoverage = true
        }
    }
}

tasks.register<JacocoReport>("jacocoTestReport") {
    dependsOn("testDebugUnitTest")

    reports {
        xml.required.set(true)
        html.required.set(true)
    }

    val fileFilter = listOf(
        "**/R.class",
        "**/R$*.class",
        "**/BuildConfig.*",
        "**/Manifest*.*",
        "**/*Test*.*",
        "**/*_Impl*.*",          // Room generated
        "**/*_Factory*.*",       // Hilt generated
        "**/*_HiltModules*.*",   // Hilt generated
        "**/*Module_*.*",        // Dagger generated
        "**/*Directions*.*",     // Navigation generated
        "**/*Args*.*",           // Navigation generated
    )

    val debugTree = fileTree("${buildDir}/tmp/kotlin-classes/debug") {
        exclude(fileFilter)
    }

    sourceDirectories.setFrom(files("src/main/kotlin"))
    classDirectories.setFrom(debugTree)
    executionData.setFrom(
        fileTree(buildDir) {
            include("**/*.exec", "**/*.ec")
        }
    )
}
```

### CI Coverage Gate

```yaml
# .github/workflows/test.yml (excerpt)
- name: Run tests with coverage
  run: ./gradlew testDebugUnitTest jacocoTestReport

- name: Check coverage threshold
  run: |
    COVERAGE=$(cat build/reports/jacoco/jacocoTestReport/jacocoTestReport.xml | \
      grep -o 'INSTRUCTION.*missed="[0-9]*".*covered="[0-9]*"' | head -1 | \
      awk -F'"' '{missed=$2; covered=$4; printf "%.1f", covered/(missed+covered)*100}')
    echo "Coverage: ${COVERAGE}%"
    if (( $(echo "$COVERAGE < 60" | bc -l) )); then
      echo "Coverage ${COVERAGE}% is below 60% threshold"
      exit 1
    fi
```

---

## 📚 Further Reading

### Official Documentation

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [MockK Documentation](https://mockk.io/)
- [Turbine GitHub](https://github.com/cashapp/turbine)
- [Kotlinx Coroutines Test](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-test/)
- [Jetpack Compose Testing](https://developer.android.com/develop/ui/compose/testing)
- [Room Testing Guide](https://developer.android.com/training/data-storage/room/testing-db)
- [JaCoCo Documentation](https://www.jacoco.org/jacoco/trunk/doc/)

### ARC Labs Internal References

- [ARCKnowledge - Clean Architecture](../Architecture/clean-architecture.md)
- [ARCKnowledge - Presentation Layer](../Layers/presentation.md)
- [ARCDevTools - CI/CD Configuration](https://github.com/arclabs-studio/ARCDevTools)

### Recommended Books

- **Growing Object-Oriented Software, Guided by Tests** -- Steve Freeman, Nat Pryce
- **Unit Testing: Principles, Practices, and Patterns** -- Vladimir Khorikov
- **Working Effectively with Legacy Code** -- Michael Feathers

### Key Principles to Remember

1. **Test behavior, not implementation** -- Your tests should not break when you refactor internals.
2. **One assertion per concept** -- Each test should verify one logical behavior, even if it uses multiple `assertEquals` calls.
3. **Tests should be FIRST** -- Fast, Independent, Repeatable, Self-validating, Timely.
4. **The test is the first user of your API** -- If the test is awkward to write, the API needs improvement.
5. **Red, Green, Refactor** -- Write a failing test, make it pass, then improve the code. Never skip step one.

---

> **ARC Labs Studio** -- Quality is not negotiable. Every line of code deserves a test that proves it works.
