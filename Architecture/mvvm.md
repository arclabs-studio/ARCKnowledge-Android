# 🏗️ MVVM Architecture

**Model-View-ViewModel architecture pattern for Android applications at ARC Labs Studio. This is the Android adaptation of our iOS MVVM+C pattern — notably, Android does NOT use a Coordinator layer. Navigation is handled through Jetpack Navigation and event-driven patterns instead.**

---

## 🎯 MVVM in Android

### Overview

MVVM (Model-View-ViewModel) is the primary architectural pattern for all ARC Labs Android projects. Unlike our iOS counterpart which uses MVVM+C (with Coordinators), the Android implementation omits the Coordinator layer entirely. Android's navigation ecosystem — Jetpack Navigation, NavHost, and NavController — provides a declarative, type-safe navigation mechanism that replaces the need for a dedicated Coordinator object.

### Why MVVM Fits Android

MVVM is the natural architectural choice for modern Android development for several reasons:

1. **Lifecycle-Aware ViewModels:** Android's `ViewModel` class from the Architecture Components library survives configuration changes (screen rotation, theme changes). This eliminates an entire class of bugs related to state loss.

2. **Compose's Declarative UI:** Jetpack Compose is inherently compatible with MVVM. Composables are functions that accept state and emit UI — they are the View layer. ViewModels expose state via `StateFlow`, and Compose re-renders automatically when state changes.

3. **Unidirectional Data Flow:** MVVM enforces a clear direction: events flow up (View to ViewModel), state flows down (ViewModel to View). This makes debugging predictable and state management explicit.

4. **Separation of Concerns:** Business logic lives in Use Cases (Domain layer), UI logic lives in ViewModels, and rendering lives in Composables. Each layer is independently testable.

5. **First-Class Hilt Support:** `@HiltViewModel` provides seamless dependency injection into ViewModels, making it trivial to inject Use Cases and other dependencies.

### Architecture Comparison

| Aspect | MVVM | MVC | MVP | MVI |
|---|---|---|---|---|
| **View-Logic Coupling** | Low | High | Medium | Low |
| **Testability** | High | Low | High | High |
| **State Management** | StateFlow | Manual | Manual | Reducer-based |
| **Compose Compatibility** | Native | Poor | Poor | Good |
| **Boilerplate** | Low | Low | High | Medium |
| **Learning Curve** | Medium | Low | Medium | High |
| **ARC Labs Standard** | Yes | No | No | No |

MVVM strikes the best balance between simplicity and structure. MVI adds unnecessary complexity for most features (reducer functions, intent classes), while MVP requires interface-heavy contracts that Compose makes obsolete. MVC conflates View and Controller concerns, leading to untestable code.

---

## 📐 Component Responsibilities

Each component in the MVVM pattern has a strict, well-defined responsibility. Violating these boundaries leads to tightly coupled, untestable code.

### View (@Composable)

The View layer consists entirely of `@Composable` functions. These functions are **pure UI renderers** — they accept state as parameters and emit user events via callback lambdas. They contain **zero business logic**.

**Responsibilities:**
- Render UI based on the current `UiState`
- Forward user interactions to the ViewModel via callback lambdas
- Handle UI-only concerns (animations, transitions, scroll state)
- Collect `StateFlow` from the ViewModel and pass values to stateless composables

**Rules:**
- Composables NEVER call repository methods directly
- Composables NEVER perform data transformations
- Composables NEVER hold business state (use `remember` only for UI state like scroll position)
- Composables NEVER instantiate ViewModels — they receive them from the navigation graph

#### Stateless Composable Example

```kotlin
@Composable
fun UserProfileScreen(
    uiState: UserProfileUiState,
    onRetryClicked: () -> Unit,
    onEditClicked: () -> Unit,
    modifier: Modifier = Modifier,
) {
    when (uiState) {
        is UserProfileUiState.Loading -> LoadingIndicator()
        is UserProfileUiState.Success -> UserProfileContent(
            user = uiState.user,
            onEditClicked = onEditClicked,
            modifier = modifier,
        )
        is UserProfileUiState.Error -> ErrorContent(
            message = uiState.message,
            onRetryClicked = onRetryClicked,
        )
    }
}
```

#### Stateful Composable (Route-Level)

The route-level composable is the bridge between the ViewModel and the stateless UI. It collects state and wires callbacks:

```kotlin
@Composable
fun UserProfileRoute(
    viewModel: UserProfileViewModel = hiltViewModel(),
    onNavigateToEdit: (String) -> Unit,
    onNavigateBack: () -> Unit,
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val context = LocalContext.current

    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is UserProfileEvent.NavigateToEdit -> onNavigateToEdit(event.userId)
                is UserProfileEvent.ShowSnackbar -> {
                    // Handle snackbar display
                }
            }
        }
    }

    UserProfileScreen(
        uiState = uiState,
        onRetryClicked = viewModel::onRetryClicked,
        onEditClicked = viewModel::onEditClicked,
    )
}
```

#### Composable Sub-Components

Break complex screens into small, focused composables:

```kotlin
@Composable
private fun UserProfileContent(
    user: User,
    onEditClicked: () -> Unit,
    modifier: Modifier = Modifier,
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp),
    ) {
        UserAvatar(
            avatarUrl = user.avatarUrl,
            name = user.name,
        )
        UserInfoSection(
            name = user.name,
            email = user.email,
            memberSince = user.memberSince,
        )
        EditProfileButton(
            onClick = onEditClicked,
        )
    }
}

@Composable
private fun UserAvatar(
    avatarUrl: String?,
    name: String,
    modifier: Modifier = Modifier,
) {
    Box(
        modifier = modifier.size(120.dp),
        contentAlignment = Alignment.Center,
    ) {
        if (avatarUrl != null) {
            AsyncImage(
                model = avatarUrl,
                contentDescription = "Avatar for $name",
                modifier = Modifier
                    .fillMaxSize()
                    .clip(CircleShape),
                contentScale = ContentScale.Crop,
            )
        } else {
            Icon(
                imageVector = Icons.Default.Person,
                contentDescription = "Default avatar for $name",
                modifier = Modifier.size(64.dp),
            )
        }
    }
}

@Composable
private fun UserInfoSection(
    name: String,
    email: String,
    memberSince: String,
    modifier: Modifier = Modifier,
) {
    Column(
        modifier = modifier.fillMaxWidth(),
        verticalArrangement = Arrangement.spacedBy(8.dp),
    ) {
        Text(
            text = name,
            style = MaterialTheme.typography.headlineMedium,
        )
        Text(
            text = email,
            style = MaterialTheme.typography.bodyLarge,
            color = MaterialTheme.colorScheme.onSurfaceVariant,
        )
        Text(
            text = "Member since $memberSince",
            style = MaterialTheme.typography.bodySmall,
            color = MaterialTheme.colorScheme.onSurfaceVariant,
        )
    }
}

@Composable
private fun EditProfileButton(
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
) {
    Button(
        onClick = onClick,
        modifier = modifier.fillMaxWidth(),
    ) {
        Icon(
            imageVector = Icons.Default.Edit,
            contentDescription = null,
            modifier = Modifier.size(18.dp),
        )
        Spacer(modifier = Modifier.width(8.dp))
        Text(text = "Edit Profile")
    }
}
```

### ViewModel

The ViewModel is the central coordinator of the MVVM pattern. It manages UI state, handles user events, and delegates business logic to Use Cases. It **NEVER** contains business logic itself — it orchestrates.

**Responsibilities:**
- Expose UI state via `StateFlow<UiState>`
- Handle user events (click, swipe, input)
- Delegate to Use Cases for business operations
- Emit one-shot events via `SharedFlow` (navigation, snackbars, toasts)
- Transform domain models into UI-friendly formats when necessary

**Rules:**
- ALWAYS annotate with `@HiltViewModel`
- ALWAYS use `@Inject constructor` for dependencies
- NEVER reference `Context`, `Activity`, `Fragment`, or any Android framework class
- NEVER expose `MutableStateFlow` — only expose the read-only `StateFlow`
- NEVER perform business logic (validation, calculation, transformation of domain data)
- ALWAYS use `viewModelScope` for coroutines

#### Full ViewModel Example

```kotlin
@HiltViewModel
class UserProfileViewModel @Inject constructor(
    private val getUserProfileUseCase: GetUserProfileUseCase,
    private val updateUserProfileUseCase: UpdateUserProfileUseCase,
    savedStateHandle: SavedStateHandle,
) : ViewModel() {

    private val userId: String = checkNotNull(savedStateHandle["userId"])

    private val _uiState = MutableStateFlow<UserProfileUiState>(UserProfileUiState.Loading)
    val uiState: StateFlow<UserProfileUiState> = _uiState.asStateFlow()

    private val _events = MutableSharedFlow<UserProfileEvent>()
    val events: SharedFlow<UserProfileEvent> = _events.asSharedFlow()

    init {
        loadUserProfile()
    }

    fun onRetryClicked() {
        loadUserProfile()
    }

    fun onEditClicked() {
        viewModelScope.launch {
            _events.emit(UserProfileEvent.NavigateToEdit(userId))
        }
    }

    fun onSaveClicked(updatedName: String, updatedEmail: String) {
        viewModelScope.launch {
            _uiState.value = UserProfileUiState.Loading
            updateUserProfileUseCase(
                UpdateUserProfileUseCase.Params(
                    userId = userId,
                    name = updatedName,
                    email = updatedEmail,
                )
            )
                .onSuccess { user ->
                    _uiState.value = UserProfileUiState.Success(user)
                    _events.emit(UserProfileEvent.ShowSnackbar("Profile updated"))
                }
                .onFailure { error ->
                    _uiState.value = UserProfileUiState.Error(
                        message = error.message ?: "Failed to update profile",
                    )
                }
        }
    }

    private fun loadUserProfile() {
        viewModelScope.launch {
            _uiState.value = UserProfileUiState.Loading
            getUserProfileUseCase(userId)
                .onSuccess { user ->
                    _uiState.value = UserProfileUiState.Success(user)
                }
                .onFailure { error ->
                    _uiState.value = UserProfileUiState.Error(
                        message = error.message ?: "Failed to load profile",
                    )
                }
        }
    }
}
```

### Model (Domain Entities + Use Cases)

The Model layer encompasses the Domain layer — entities, Use Cases, and repository interfaces. This layer is **pure Kotlin** with no Android dependencies.

#### Domain Entities

Entities are simple data classes that represent core business objects:

```kotlin
data class User(
    val id: String,
    val name: String,
    val email: String,
    val avatarUrl: String?,
    val memberSince: String,
)
```

#### Use Cases

Use Cases encapsulate a single business operation. They use `operator fun invoke()` for clean call-site syntax:

```kotlin
class GetUserProfileUseCase @Inject constructor(
    private val userRepository: UserRepository,
) {
    suspend operator fun invoke(userId: String): Result<User> {
        return userRepository.getUserProfile(userId)
    }
}
```

```kotlin
class UpdateUserProfileUseCase @Inject constructor(
    private val userRepository: UserRepository,
) {
    data class Params(
        val userId: String,
        val name: String,
        val email: String,
    )

    suspend operator fun invoke(params: Params): Result<User> {
        return userRepository.updateUserProfile(
            userId = params.userId,
            name = params.name,
            email = params.email,
        )
    }
}
```

#### Repository Interfaces

Repository interfaces define the contract between the Domain and Data layers. They live in the Domain layer:

```kotlin
interface UserRepository {
    suspend fun getUserProfile(userId: String): Result<User>
    suspend fun updateUserProfile(
        userId: String,
        name: String,
        email: String,
    ): Result<User>
    fun observeUserProfile(userId: String): Flow<User>
}
```

---

## 🔄 State Management

State management is the backbone of any MVVM implementation. At ARC Labs, we use a strict pattern based on sealed interfaces for UI state and dedicated event channels for one-shot actions.

### Sealed Interface UiState Pattern

Every screen defines a sealed interface that exhaustively represents all possible UI states:

```kotlin
sealed interface UserProfileUiState {
    data object Loading : UserProfileUiState
    data class Success(val user: User) : UserProfileUiState
    data class Error(val message: String) : UserProfileUiState
}
```

This pattern provides:
- **Exhaustive `when` expressions:** The compiler ensures every state is handled
- **Type safety:** Each state carries only the data it needs
- **Clarity:** The set of possible states is immediately visible

#### Complex State Example

For screens with multiple independent data sections, compose states together:

```kotlin
sealed interface DashboardUiState {
    data object Loading : DashboardUiState

    data class Success(
        val userName: String,
        val recentOrders: List<OrderSummary>,
        val recommendations: List<Product>,
        val notificationCount: Int,
    ) : DashboardUiState

    data class PartialSuccess(
        val userName: String,
        val recentOrders: List<OrderSummary>,
        val recommendationsError: String?,
    ) : DashboardUiState

    data class Error(val message: String) : DashboardUiState
}
```

### StateFlow for Persistent State

`StateFlow` is the standard mechanism for exposing UI state from a ViewModel. It is lifecycle-aware when collected with `collectAsStateWithLifecycle()` in Compose.

```kotlin
// In ViewModel
private val _uiState = MutableStateFlow<UserProfileUiState>(UserProfileUiState.Loading)
val uiState: StateFlow<UserProfileUiState> = _uiState.asStateFlow()

// In Composable
@Composable
fun UserProfileRoute(viewModel: UserProfileViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    UserProfileScreen(uiState = uiState)
}
```

**Key properties of StateFlow:**
- Always has a current value (no null initial state needed)
- Replays the latest value to new collectors
- Only emits when the value actually changes (structural equality)
- Survives configuration changes when held in a ViewModel

### SharedFlow for One-Shot Events

Navigation, snackbar messages, and toasts are one-shot events that should NOT be replayed on configuration changes. Use `SharedFlow` for these:

```kotlin
sealed interface UserProfileEvent {
    data class NavigateToEdit(val userId: String) : UserProfileEvent
    data class ShowSnackbar(val message: String) : UserProfileEvent
    data object NavigateBack : UserProfileEvent
}

// In ViewModel
private val _events = MutableSharedFlow<UserProfileEvent>()
val events: SharedFlow<UserProfileEvent> = _events.asSharedFlow()

// Emitting events
viewModelScope.launch {
    _events.emit(UserProfileEvent.ShowSnackbar("Profile saved"))
}
```

### Collecting Events in Compose

```kotlin
@Composable
fun UserProfileRoute(
    viewModel: UserProfileViewModel = hiltViewModel(),
    onNavigateToEdit: (String) -> Unit,
    onNavigateBack: () -> Unit,
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val snackbarHostState = remember { SnackbarHostState() }

    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is UserProfileEvent.NavigateToEdit -> onNavigateToEdit(event.userId)
                is UserProfileEvent.ShowSnackbar -> {
                    snackbarHostState.showSnackbar(event.message)
                }
                is UserProfileEvent.NavigateBack -> onNavigateBack()
            }
        }
    }

    Scaffold(
        snackbarHost = { SnackbarHost(hostState = snackbarHostState) },
    ) { paddingValues ->
        UserProfileScreen(
            uiState = uiState,
            onRetryClicked = viewModel::onRetryClicked,
            onEditClicked = viewModel::onEditClicked,
            modifier = Modifier.padding(paddingValues),
        )
    }
}
```

---

## 🧭 Navigation Pattern

In Android MVVM, there is **no Coordinator**. Navigation is handled through a combination of Jetpack Navigation's `NavController`, callback lambdas, and ViewModel-emitted events.

### Core Principles

1. **ViewModels NEVER hold a reference to `NavController`** — this is an Android framework class
2. **Navigation decisions originate in ViewModels** — but are executed by Composables
3. **NavController is managed at the NavHost level** — passed down via lambda callbacks
4. **Type-safe navigation routes** — use Kotlin serialization for route arguments

### Navigation Event Flow

The ViewModel decides _when_ to navigate by emitting events. The Composable decides _how_ to navigate by collecting those events and calling `NavController`:

```kotlin
// Step 1: ViewModel emits a navigation event
@HiltViewModel
class OrderListViewModel @Inject constructor(
    private val getOrdersUseCase: GetOrdersUseCase,
) : ViewModel() {

    private val _events = MutableSharedFlow<OrderListEvent>()
    val events: SharedFlow<OrderListEvent> = _events.asSharedFlow()

    fun onOrderClicked(orderId: String) {
        viewModelScope.launch {
            _events.emit(OrderListEvent.NavigateToDetail(orderId))
        }
    }
}

sealed interface OrderListEvent {
    data class NavigateToDetail(val orderId: String) : OrderListEvent
}
```

```kotlin
// Step 2: Route-level Composable collects and navigates
@Composable
fun OrderListRoute(
    viewModel: OrderListViewModel = hiltViewModel(),
    onNavigateToDetail: (String) -> Unit,
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is OrderListEvent.NavigateToDetail -> {
                    onNavigateToDetail(event.orderId)
                }
            }
        }
    }

    OrderListScreen(
        uiState = uiState,
        onOrderClicked = viewModel::onOrderClicked,
    )
}
```

```kotlin
// Step 3: NavHost wires navigation callbacks to NavController
@Composable
fun AppNavHost(
    navController: NavHostController = rememberNavController(),
    modifier: Modifier = Modifier,
) {
    NavHost(
        navController = navController,
        startDestination = "orders",
        modifier = modifier,
    ) {
        composable("orders") {
            OrderListRoute(
                onNavigateToDetail = { orderId ->
                    navController.navigate("orders/$orderId")
                },
            )
        }
        composable(
            route = "orders/{orderId}",
            arguments = listOf(
                navArgument("orderId") { type = NavType.StringType }
            ),
        ) {
            OrderDetailRoute(
                onNavigateBack = { navController.popBackStack() },
                onNavigateToTracking = { trackingId ->
                    navController.navigate("tracking/$trackingId")
                },
            )
        }
    }
}
```

### Alternative: Callback Lambdas Without SharedFlow

For simple screens where navigation is a direct response to a user action, you can skip `SharedFlow` and use callback lambdas directly:

```kotlin
@HiltViewModel
class SettingsViewModel @Inject constructor(
    private val logoutUseCase: LogoutUseCase,
) : ViewModel() {

    val uiState: StateFlow<SettingsUiState> = // ...

    fun onLogoutClicked(onLogoutComplete: () -> Unit) {
        viewModelScope.launch {
            logoutUseCase()
            onLogoutComplete()
        }
    }
}
```

This approach is simpler but less decoupled. Prefer `SharedFlow` for complex navigation scenarios and callback lambdas for straightforward cases.

---

## 📊 Data Flow Diagram

The following diagram illustrates the complete data flow through all layers of the MVVM architecture:

```
┌─────────────────────────────────────────────────────────────────┐
│                        PRESENTATION LAYER                       │
│                                                                 │
│   User Action                                                   │
│       │                                                         │
│       ▼                                                         │
│   ┌───────────────────┐         ┌───────────────────────┐      │
│   │   @Composable     │         │     ViewModel         │      │
│   │   (View)          │────────▶│  @HiltViewModel       │      │
│   │                   │  Event  │                       │      │
│   │   - Renders UI    │         │  - Manages UiState    │      │
│   │   - Emits events  │         │  - Delegates to       │      │
│   │                   │◀────────│    Use Cases           │      │
│   │                   │  State  │                       │      │
│   └───────────────────┘         └───────────┬───────────┘      │
│                                             │                   │
└─────────────────────────────────────────────┼───────────────────┘
                                              │ invoke()
┌─────────────────────────────────────────────┼───────────────────┐
│                         DOMAIN LAYER        │                   │
│                                             ▼                   │
│   ┌─────────────────────┐         ┌─────────────────────┐      │
│   │   Entities          │         │   Use Cases          │      │
│   │                     │◀────────│                      │      │
│   │   - data class User │         │  - operator invoke() │      │
│   │   - data class Order│         │  - Single operation  │      │
│   └─────────────────────┘         └──────────┬──────────┘      │
│                                              │                  │
│   ┌──────────────────────────────────────────┘                  │
│   │   Repository Interface                                      │
│   │   (defined in Domain, implemented in Data)                  │
│   └──────────────────────┬──────────────────────────────────────┘
│                          │
└──────────────────────────┼───────────────────────────────────────┘
                           │
┌──────────────────────────┼───────────────────────────────────────┐
│                     DATA LAYER                                    │
│                          ▼                                        │
│   ┌──────────────────────────────────────┐                       │
│   │   Repository Implementation          │                       │
│   │                                      │                       │
│   │   - Coordinates data sources         │                       │
│   │   - Handles caching strategy         │                       │
│   └──────┬───────────────────┬───────────┘                       │
│          │                   │                                    │
│          ▼                   ▼                                    │
│   ┌──────────────┐   ┌──────────────┐                            │
│   │ Remote Source │   │ Local Source  │                           │
│   │ (Retrofit)   │   │ (Room)        │                           │
│   └──────────────┘   └──────────────┘                            │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

### Simplified Flow

```
User Action → Composable → ViewModel → UseCase → Repository
                ↑                                      ↓
                └──── StateFlow (UiState) ─────────────┘
```

### Key Points

- **Events flow UP:** User taps a button in the Composable, which calls a ViewModel method
- **State flows DOWN:** ViewModel updates `StateFlow`, Composable recomposes with new state
- **Business logic stays in the MIDDLE:** Use Cases in the Domain layer handle all business rules
- **Data layer is an implementation detail:** The Domain layer only knows about repository interfaces

---

## 🔗 Connecting Layers with Hilt

Hilt (Dagger under the hood) is the dependency injection framework used at ARC Labs for all Android projects. It wires together every layer of the MVVM architecture.

### Module Structure

#### Data Module — Provides Repository Implementations

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    @Singleton
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl,
    ): UserRepository

    @Binds
    @Singleton
    abstract fun bindOrderRepository(
        impl: OrderRepositoryImpl,
    ): OrderRepository
}
```

#### Data Module — Provides Data Sources

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DataSourceModule {

    @Provides
    @Singleton
    fun provideUserApi(retrofit: Retrofit): UserApi {
        return retrofit.create(UserApi::class.java)
    }

    @Provides
    @Singleton
    fun provideUserDao(database: AppDatabase): UserDao {
        return database.userDao()
    }
}
```

#### Network Module — Provides Retrofit and OkHttp

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(HttpLoggingInterceptor().apply {
                level = HttpLoggingInterceptor.Level.BODY
            })
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BuildConfig.BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
}
```

### ViewModel Injection

ViewModels are automatically injectable with `@HiltViewModel`:

```kotlin
@HiltViewModel
class UserProfileViewModel @Inject constructor(
    private val getUserProfileUseCase: GetUserProfileUseCase,
    private val updateUserProfileUseCase: UpdateUserProfileUseCase,
    savedStateHandle: SavedStateHandle,
) : ViewModel() {
    // Use Cases are automatically injected by Hilt
    // SavedStateHandle provides navigation arguments
}
```

### Use Case Injection

Use Cases receive their dependencies through constructor injection. No `@Module` is needed for Use Cases because Hilt can construct them automatically when their dependencies are available:

```kotlin
class GetUserProfileUseCase @Inject constructor(
    private val userRepository: UserRepository,
) {
    suspend operator fun invoke(userId: String): Result<User> {
        return userRepository.getUserProfile(userId)
    }
}
```

### Repository Implementation

```kotlin
@Singleton
class UserRepositoryImpl @Inject constructor(
    private val userApi: UserApi,
    private val userDao: UserDao,
    private val userMapper: UserMapper,
) : UserRepository {

    override suspend fun getUserProfile(userId: String): Result<User> {
        return runCatching {
            val response = userApi.getUserProfile(userId)
            val entity = userMapper.mapToDomain(response)
            userDao.insertUser(userMapper.mapToLocal(entity))
            entity
        }
    }

    override suspend fun updateUserProfile(
        userId: String,
        name: String,
        email: String,
    ): Result<User> {
        return runCatching {
            val response = userApi.updateUserProfile(userId, name, email)
            val entity = userMapper.mapToDomain(response)
            userDao.insertUser(userMapper.mapToLocal(entity))
            entity
        }
    }

    override fun observeUserProfile(userId: String): Flow<User> {
        return userDao.observeUser(userId).map { localUser ->
            userMapper.mapToDomain(localUser)
        }
    }
}
```

### Full Wiring: Module to ViewModel to Composable

The complete dependency chain looks like this:

```
Hilt Modules provide:
  └── Retrofit → UserApi
  └── AppDatabase → UserDao
  └── UserApi + UserDao → UserRepositoryImpl (bound as UserRepository)

UserRepository is injected into:
  └── GetUserProfileUseCase
  └── UpdateUserProfileUseCase

Use Cases are injected into:
  └── UserProfileViewModel (@HiltViewModel)

ViewModel is created by:
  └── hiltViewModel() in UserProfileRoute (@Composable)
```

---

## 🧪 Testing MVVM

Every layer of the MVVM architecture is independently testable. At ARC Labs, we target 100% coverage for packages and 80%+ for apps.

### ViewModel Testing

ViewModel tests verify that the ViewModel correctly manages state in response to events and Use Case results. We use [Turbine](https://github.com/cashapp/turbine) for testing `Flow` emissions and `kotlinx-coroutines-test` for controlling coroutine execution.

```kotlin
class UserProfileViewModelTest {

    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()

    private val getUserProfileUseCase = mockk<GetUserProfileUseCase>()
    private val updateUserProfileUseCase = mockk<UpdateUserProfileUseCase>()

    private fun makeSUT(userId: String = "user-123"): UserProfileViewModel {
        val savedStateHandle = SavedStateHandle(mapOf("userId" to userId))
        return UserProfileViewModel(
            getUserProfileUseCase = getUserProfileUseCase,
            updateUserProfileUseCase = updateUserProfileUseCase,
            savedStateHandle = savedStateHandle,
        )
    }

    @Test
    fun `should emit loading then success when profile loads successfully`() = runTest {
        // Given
        val expectedUser = User(
            id = "user-123",
            name = "Jane Doe",
            email = "jane@example.com",
            avatarUrl = null,
            memberSince = "2024-01-15",
        )
        coEvery { getUserProfileUseCase("user-123") } returns Result.success(expectedUser)

        // When
        val sut = makeSUT()

        // Then
        sut.uiState.test {
            assertThat(awaitItem()).isEqualTo(UserProfileUiState.Loading)
            assertThat(awaitItem()).isEqualTo(UserProfileUiState.Success(expectedUser))
        }
    }

    @Test
    fun `should emit loading then error when profile fails to load`() = runTest {
        // Given
        coEvery { getUserProfileUseCase("user-123") } returns
            Result.failure(Exception("Network error"))

        // When
        val sut = makeSUT()

        // Then
        sut.uiState.test {
            assertThat(awaitItem()).isEqualTo(UserProfileUiState.Loading)
            assertThat(awaitItem()).isEqualTo(
                UserProfileUiState.Error("Network error")
            )
        }
    }

    @Test
    fun `should reload profile when retry is clicked`() = runTest {
        // Given
        val expectedUser = User(
            id = "user-123",
            name = "Jane Doe",
            email = "jane@example.com",
            avatarUrl = null,
            memberSince = "2024-01-15",
        )
        coEvery { getUserProfileUseCase("user-123") } returnsMany listOf(
            Result.failure(Exception("Network error")),
            Result.success(expectedUser),
        )
        val sut = makeSUT()

        sut.uiState.test {
            // Initial load fails
            assertThat(awaitItem()).isEqualTo(UserProfileUiState.Loading)
            assertThat(awaitItem()).isInstanceOf(UserProfileUiState.Error::class.java)

            // When
            sut.onRetryClicked()

            // Then
            assertThat(awaitItem()).isEqualTo(UserProfileUiState.Loading)
            assertThat(awaitItem()).isEqualTo(UserProfileUiState.Success(expectedUser))
        }
    }

    @Test
    fun `should emit navigation event when edit is clicked`() = runTest {
        // Given
        coEvery { getUserProfileUseCase("user-123") } returns
            Result.success(mockk(relaxed = true))
        val sut = makeSUT()

        // Then
        sut.events.test {
            // When
            sut.onEditClicked()

            assertThat(awaitItem()).isEqualTo(
                UserProfileEvent.NavigateToEdit("user-123")
            )
        }
    }
}
```

### MainDispatcherRule

A reusable test rule for replacing `Dispatchers.Main` in unit tests:

```kotlin
class MainDispatcherRule(
    private val dispatcher: TestDispatcher = UnconfinedTestDispatcher(),
) : TestWatcher() {

    override fun starting(description: Description) {
        Dispatchers.setMain(dispatcher)
    }

    override fun finished(description: Description) {
        Dispatchers.resetMain()
    }
}
```

### Composable Testing

Use `ComposeTestRule` to test Composable UI behavior:

```kotlin
class UserProfileScreenTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun `should display loading indicator when state is loading`() {
        // Given
        composeTestRule.setContent {
            UserProfileScreen(
                uiState = UserProfileUiState.Loading,
                onRetryClicked = {},
                onEditClicked = {},
            )
        }

        // Then
        composeTestRule
            .onNodeWithTag("loading_indicator")
            .assertIsDisplayed()
    }

    @Test
    fun `should display user name when state is success`() {
        // Given
        val user = User(
            id = "user-123",
            name = "Jane Doe",
            email = "jane@example.com",
            avatarUrl = null,
            memberSince = "2024-01-15",
        )

        composeTestRule.setContent {
            UserProfileScreen(
                uiState = UserProfileUiState.Success(user),
                onRetryClicked = {},
                onEditClicked = {},
            )
        }

        // Then
        composeTestRule
            .onNodeWithText("Jane Doe")
            .assertIsDisplayed()
    }

    @Test
    fun `should display error message and retry button when state is error`() {
        // Given
        composeTestRule.setContent {
            UserProfileScreen(
                uiState = UserProfileUiState.Error("Network error"),
                onRetryClicked = {},
                onEditClicked = {},
            )
        }

        // Then
        composeTestRule
            .onNodeWithText("Network error")
            .assertIsDisplayed()
        composeTestRule
            .onNodeWithText("Retry")
            .assertIsDisplayed()
    }

    @Test
    fun `should call onRetryClicked when retry button is tapped`() {
        // Given
        var retryClicked = false

        composeTestRule.setContent {
            UserProfileScreen(
                uiState = UserProfileUiState.Error("Network error"),
                onRetryClicked = { retryClicked = true },
                onEditClicked = {},
            )
        }

        // When
        composeTestRule
            .onNodeWithText("Retry")
            .performClick()

        // Then
        assertThat(retryClicked).isTrue()
    }
}
```

---

## ✅ MVVM Checklist

Use this checklist when implementing or reviewing any MVVM feature:

**View (@Composable):**
- [ ] Stateless screen composable accepts `UiState` and callback lambdas
- [ ] Route-level composable collects `StateFlow` with `collectAsStateWithLifecycle()`
- [ ] One-shot events collected in `LaunchedEffect(Unit)`
- [ ] No business logic in any composable
- [ ] No direct ViewModel references below the route-level composable
- [ ] Sub-components are small, focused, and reusable
- [ ] Preview functions exist for all significant composables

**ViewModel:**
- [ ] Annotated with `@HiltViewModel`
- [ ] Uses `@Inject constructor`
- [ ] Exposes `StateFlow<UiState>` (not `MutableStateFlow`)
- [ ] Exposes `SharedFlow<Event>` for one-shot events (not `MutableSharedFlow`)
- [ ] All coroutines use `viewModelScope`
- [ ] No Android framework imports (`Context`, `Activity`, etc.)
- [ ] No business logic — delegates to Use Cases
- [ ] Navigation arguments read from `SavedStateHandle`

**Model (Domain):**
- [ ] Entities are `data class` with no Android dependencies
- [ ] Use Cases have single `operator fun invoke()` method
- [ ] Repository interfaces defined in Domain layer
- [ ] No framework dependencies (pure Kotlin)

**State Management:**
- [ ] `UiState` is a sealed interface with `Loading`, `Success`, `Error` variants
- [ ] One-shot events use `SharedFlow` (not `StateFlow`)
- [ ] State updates are atomic (single `_uiState.value = ...` assignment)

**Dependency Injection:**
- [ ] Repository implementations bound with `@Binds`
- [ ] Data sources provided with `@Provides`
- [ ] No manual instantiation of dependencies

**Testing:**
- [ ] ViewModel tests use `makeSUT()` factory
- [ ] ViewModel tests use `MainDispatcherRule`
- [ ] Flow assertions use Turbine
- [ ] Composable tests verify all UiState branches
- [ ] Composable tests verify callback invocations

---

## 🚫 Common Mistakes

### Mistake 1: Business Logic in ViewModel

The ViewModel should orchestrate, not compute. Business rules belong in Use Cases.

```kotlin
// ❌ WRONG: Business logic in ViewModel
@HiltViewModel
class OrderViewModel @Inject constructor(
    private val orderRepository: OrderRepository,
) : ViewModel() {

    fun calculateTotal(items: List<OrderItem>): Double {
        var total = 0.0
        for (item in items) {
            total += item.price * item.quantity
            if (item.quantity > 10) {
                total *= 0.9 // 10% bulk discount
            }
        }
        if (total > 100) {
            total -= 15.0 // loyalty discount
        }
        return total
    }
}
```

```kotlin
// ✅ CORRECT: Business logic in Use Case
class CalculateOrderTotalUseCase @Inject constructor() {

    operator fun invoke(items: List<OrderItem>): Double {
        val subtotal = items.sumOf { it.price * it.quantity }
        val bulkDiscount = items
            .filter { it.quantity > 10 }
            .sumOf { it.price * it.quantity * 0.1 }
        val loyaltyDiscount = if (subtotal > 100) 15.0 else 0.0
        return subtotal - bulkDiscount - loyaltyDiscount
    }
}

@HiltViewModel
class OrderViewModel @Inject constructor(
    private val calculateOrderTotalUseCase: CalculateOrderTotalUseCase,
) : ViewModel() {

    fun onItemsChanged(items: List<OrderItem>) {
        val total = calculateOrderTotalUseCase(items)
        _uiState.value = _uiState.value.copy(total = total)
    }
}
```

### Mistake 2: Mutable State Exposed Publicly

Exposing `MutableStateFlow` allows any component to modify ViewModel state, breaking unidirectional data flow.

```kotlin
// ❌ WRONG: Mutable state is public
@HiltViewModel
class ProfileViewModel @Inject constructor() : ViewModel() {
    val uiState = MutableStateFlow<ProfileUiState>(ProfileUiState.Loading)
    val events = MutableSharedFlow<ProfileEvent>()
}
```

```kotlin
// ✅ CORRECT: Only immutable types are public
@HiltViewModel
class ProfileViewModel @Inject constructor() : ViewModel() {
    private val _uiState = MutableStateFlow<ProfileUiState>(ProfileUiState.Loading)
    val uiState: StateFlow<ProfileUiState> = _uiState.asStateFlow()

    private val _events = MutableSharedFlow<ProfileEvent>()
    val events: SharedFlow<ProfileEvent> = _events.asSharedFlow()
}
```

### Mistake 3: Collecting Flows Without Lifecycle Awareness

Collecting flows without lifecycle awareness causes updates to be processed when the app is in the background, wasting resources and potentially causing crashes.

```kotlin
// ❌ WRONG: Not lifecycle-aware
@Composable
fun ProfileRoute(viewModel: ProfileViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsState() // No lifecycle awareness!

    ProfileScreen(uiState = uiState)
}
```

```kotlin
// ✅ CORRECT: Lifecycle-aware collection
@Composable
fun ProfileRoute(viewModel: ProfileViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    ProfileScreen(uiState = uiState)
}
```

`collectAsStateWithLifecycle()` automatically stops collection when the Composable is not in a `STARTED` or `RESUMED` lifecycle state.

### Mistake 4: ViewModel Directly References Android Framework Classes

ViewModels should be pure Kotlin classes that can be unit tested without Robolectric or instrumented tests.

```kotlin
// ❌ WRONG: ViewModel depends on Android framework
@HiltViewModel
class SettingsViewModel @Inject constructor(
    private val context: Context, // Android framework class!
) : ViewModel() {

    fun getAppVersion(): String {
        return context.packageManager
            .getPackageInfo(context.packageName, 0)
            .versionName ?: "Unknown"
    }

    fun showToast(message: String) {
        Toast.makeText(context, message, Toast.LENGTH_SHORT).show() // UI in ViewModel!
    }
}
```

```kotlin
// ✅ CORRECT: Abstract Android dependencies behind interfaces
interface AppInfoProvider {
    fun getAppVersion(): String
}

class AndroidAppInfoProvider @Inject constructor(
    @ApplicationContext private val context: Context,
) : AppInfoProvider {
    override fun getAppVersion(): String {
        return context.packageManager
            .getPackageInfo(context.packageName, 0)
            .versionName ?: "Unknown"
    }
}

@HiltViewModel
class SettingsViewModel @Inject constructor(
    private val appInfoProvider: AppInfoProvider,
) : ViewModel() {

    private val _events = MutableSharedFlow<SettingsEvent>()
    val events: SharedFlow<SettingsEvent> = _events.asSharedFlow()

    fun getAppVersion(): String {
        return appInfoProvider.getAppVersion()
    }

    fun onActionCompleted(message: String) {
        viewModelScope.launch {
            _events.emit(SettingsEvent.ShowSnackbar(message))
        }
    }
}
```

---

## 📚 Further Reading

### Official Android Documentation
- [Guide to App Architecture](https://developer.android.com/topic/architecture)
- [ViewModel Overview](https://developer.android.com/topic/libraries/architecture/viewmodel)
- [StateFlow and SharedFlow](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow)
- [Jetpack Compose Navigation](https://developer.android.com/jetpack/compose/navigation)
- [Dependency Injection with Hilt](https://developer.android.com/training/dependency-injection/hilt-android)
- [UI State Production](https://developer.android.com/topic/architecture/ui-layer/state-production)

### Libraries
- [Hilt](https://dagger.dev/hilt/) - Dependency injection
- [Turbine](https://github.com/cashapp/turbine) - Flow testing
- [MockK](https://mockk.io/) - Kotlin mocking framework
- [Truth](https://truth.dev/) - Fluent assertions

### ARC Labs Internal
- Architecture Decision Records (ADRs) in the project wiki
- ARC Labs Android Code Style Guide
- ARC Labs Testing Standards
