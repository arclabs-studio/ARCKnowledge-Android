# 🎨 Presentation Layer

**The Presentation Layer is the user-facing surface of the application. It is responsible for rendering UI, collecting user interactions, and reflecting application state visually. In Android with Jetpack Compose, this layer is built entirely with `@Composable` functions, `@HiltViewModel`-annotated ViewModels, `StateFlow` for reactive state, and Jetpack Navigation for screen transitions. The Presentation Layer contains zero business logic -- all decisions, transformations, and validations are delegated to the Domain Layer via Use Cases.**

---

## 🎯 Purpose

The Presentation Layer exists to:

- **Handle the user interface** -- render screens, components, and visual feedback using Jetpack Compose.
- **Manage UI state** -- ViewModels hold and expose `StateFlow` representing the current state of each screen.
- **React to user interactions** -- button clicks, text input, swipes, and gestures are captured and forwarded to ViewModels.
- **Navigate between screens** -- Jetpack Navigation Compose handles all screen transitions declaratively.
- **Contain zero business logic** -- no validation, no data transformation, no calculation. All of that belongs in the Domain Layer.

### What Belongs Here

| Belongs in Presentation | Does NOT Belong in Presentation |
|------------------------|---------------------------------|
| Composable functions | Business rules |
| ViewModels | Data validation logic |
| UI State sealed interfaces | Repository implementations |
| Navigation graphs | Network calls |
| Theme definitions | Database queries |
| String resources | Data mapping/transformation |
| Animation logic | Use Case implementations |
| Accessibility semantics | Entity definitions |

---

## 📂 Directory Structure

```
presentation/
├── features/
│   ├── home/
│   │   ├── HomeScreen.kt
│   │   ├── HomeViewModel.kt
│   │   ├── HomeUiState.kt
│   │   ├── HomeEvent.kt
│   │   └── components/
│   │       ├── RestaurantCard.kt
│   │       ├── RestaurantList.kt
│   │       ├── SearchBar.kt
│   │       └── FilterChipRow.kt
│   ├── userprofile/
│   │   ├── UserProfileScreen.kt
│   │   ├── UserProfileViewModel.kt
│   │   ├── UserProfileUiState.kt
│   │   └── components/
│   │       ├── ProfileHeader.kt
│   │       ├── ProfileStats.kt
│   │       └── FavoritesList.kt
│   ├── restaurantdetail/
│   │   ├── RestaurantDetailScreen.kt
│   │   ├── RestaurantDetailViewModel.kt
│   │   ├── RestaurantDetailUiState.kt
│   │   └── components/
│   │       ├── MenuSection.kt
│   │       ├── ReviewCard.kt
│   │       └── LocationMap.kt
│   └── settings/
│       ├── SettingsScreen.kt
│       ├── SettingsViewModel.kt
│       └── SettingsUiState.kt
├── navigation/
│   ├── AppNavDisplay.kt
│   ├── Keys.kt
│   └── Navigator.kt
├── theme/
│   ├── Color.kt
│   ├── Theme.kt
│   ├── Type.kt
│   └── Shape.kt
└── components/
    ├── LoadingIndicator.kt
    ├── ErrorContent.kt
    ├── EmptyContent.kt
    ├── PullToRefreshContainer.kt
    └── AsyncImageWithPlaceholder.kt
```

### Directory Rules

- **Each feature gets its own package** under `features/`.
- **Screen + ViewModel + UiState + Event** live together in the feature package.
- **Feature-specific components** go into a `components/` sub-package within the feature.
- **Shared components** go into the top-level `components/` package.
- **Navigation** is centralized in the `navigation/` package.
- **Theming** is centralized in the `theme/` package.

---

## 🖼️ Composables (@Composable)

Composables are the building blocks of the UI. We organize them into three tiers: **Screen**, **Content**, and **Component** composables.

### Screen Composables

**Screen composables are the top-level entry point for each feature. They connect to the ViewModel, collect state, and delegate rendering to Content composables.**

A Screen composable:
- Receives a ViewModel (typically via `hiltViewModel()`)
- Collects `StateFlow` using `collectAsStateWithLifecycle()`
- Collects one-shot events via `LaunchedEffect`
- Passes state down and lambdas up
- Contains NO layout logic itself

```kotlin
@Composable
fun UserProfileScreen(
    viewModel: UserProfileViewModel = hiltViewModel(),
    onNavigateBack: () -> Unit,
    onNavigateToSettings: () -> Unit,
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is UserProfileEvent.NavigateToSettings -> onNavigateToSettings()
                is UserProfileEvent.NavigateToFavorite -> { /* handle */ }
            }
        }
    }

    UserProfileContent(
        uiState = uiState,
        onRetryClicked = viewModel::loadProfile,
        onNavigateBack = onNavigateBack,
        onSettingsClicked = viewModel::onSettingsClicked,
        onFavoriteToggled = viewModel::onFavoriteToggled,
    )
}
```

**Screen composable rules:**

- ✅ Name follows `{Feature}Screen` convention
- ✅ ViewModel injected with default parameter `hiltViewModel()`
- ✅ Navigation callbacks passed as lambda parameters
- ✅ State collected with `collectAsStateWithLifecycle()`
- ✅ Events collected inside `LaunchedEffect(Unit)`
- ❌ No layout code (Scaffold, Column, Row, etc.)
- ❌ No direct `remember` or `mutableStateOf` for business state
- ❌ No string resources or hardcoded text

---

### Content Composables

**Content composables are stateless. They receive all data as parameters and emit all interactions as lambda callbacks. They own the layout structure for the screen.**

```kotlin
@Composable
fun UserProfileContent(
    uiState: UserProfileUiState,
    onRetryClicked: () -> Unit,
    onNavigateBack: () -> Unit,
    onSettingsClicked: () -> Unit,
    onFavoriteToggled: (String) -> Unit,
    modifier: Modifier = Modifier,
) {
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text(stringResource(R.string.user_profile_title)) },
                navigationIcon = {
                    IconButton(onClick = onNavigateBack) {
                        Icon(
                            imageVector = Icons.AutoMirrored.Filled.ArrowBack,
                            contentDescription = stringResource(R.string.navigate_back),
                        )
                    }
                },
                actions = {
                    IconButton(onClick = onSettingsClicked) {
                        Icon(
                            imageVector = Icons.Filled.Settings,
                            contentDescription = stringResource(R.string.settings),
                        )
                    }
                },
            )
        },
        modifier = modifier,
    ) { padding ->
        when (uiState) {
            is UserProfileUiState.Loading -> {
                LoadingIndicator(
                    modifier = Modifier
                        .padding(padding)
                        .fillMaxSize(),
                )
            }
            is UserProfileUiState.Success -> {
                ProfileDetails(
                    user = uiState.user,
                    favorites = uiState.favorites,
                    onFavoriteToggled = onFavoriteToggled,
                    modifier = Modifier.padding(padding),
                )
            }
            is UserProfileUiState.Error -> {
                ErrorContent(
                    message = uiState.message,
                    onRetry = onRetryClicked,
                    modifier = Modifier
                        .padding(padding)
                        .fillMaxSize(),
                )
            }
        }
    }
}
```

**Content composable rules:**

- ✅ Name follows `{Feature}Content` convention
- ✅ Completely stateless -- all state passed via parameters
- ✅ All user interactions forwarded via lambdas
- ✅ Accepts a trailing `modifier: Modifier = Modifier` parameter
- ✅ Handles all branches of the `UiState` sealed interface
- ❌ No ViewModel reference
- ❌ No `collectAsStateWithLifecycle()` calls
- ❌ No side effects (`LaunchedEffect`, `DisposableEffect`)

---

### Component Composables

**Component composables are small, reusable UI elements. They follow Material Design 3, are self-contained, and can be used across features.**

```kotlin
@Composable
fun RestaurantCard(
    restaurant: Restaurant,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
) {
    Card(
        onClick = onClick,
        modifier = modifier.fillMaxWidth(),
        shape = MaterialTheme.shapes.medium,
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surface,
        ),
    ) {
        Row(
            modifier = Modifier
                .padding(16.dp)
                .fillMaxWidth(),
            verticalAlignment = Alignment.CenterVertically,
        ) {
            AsyncImage(
                model = restaurant.imageUrl,
                contentDescription = stringResource(
                    R.string.restaurant_image_description,
                    restaurant.name,
                ),
                modifier = Modifier
                    .size(64.dp)
                    .clip(MaterialTheme.shapes.small),
                contentScale = ContentScale.Crop,
            )
            Spacer(modifier = Modifier.width(16.dp))
            Column(modifier = Modifier.weight(1f)) {
                Text(
                    text = restaurant.name,
                    style = MaterialTheme.typography.titleMedium,
                    maxLines = 1,
                    overflow = TextOverflow.Ellipsis,
                )
                Spacer(modifier = Modifier.height(4.dp))
                Text(
                    text = restaurant.cuisineType.displayName,
                    style = MaterialTheme.typography.bodyMedium,
                    color = MaterialTheme.colorScheme.onSurfaceVariant,
                )
                Spacer(modifier = Modifier.height(4.dp))
                Row(verticalAlignment = Alignment.CenterVertically) {
                    Icon(
                        imageVector = Icons.Filled.Star,
                        contentDescription = null,
                        tint = MaterialTheme.colorScheme.primary,
                        modifier = Modifier.size(16.dp),
                    )
                    Spacer(modifier = Modifier.width(4.dp))
                    Text(
                        text = restaurant.rating.toString(),
                        style = MaterialTheme.typography.labelMedium,
                    )
                }
            }
            Icon(
                imageVector = Icons.AutoMirrored.Filled.KeyboardArrowRight,
                contentDescription = null,
                tint = MaterialTheme.colorScheme.onSurfaceVariant,
            )
        }
    }
}
```

#### Shared Components

These live in the top-level `components/` package and are used across multiple features.

```kotlin
@Composable
fun LoadingIndicator(
    modifier: Modifier = Modifier,
) {
    Box(
        modifier = modifier,
        contentAlignment = Alignment.Center,
    ) {
        CircularProgressIndicator(
            color = MaterialTheme.colorScheme.primary,
        )
    }
}
```

```kotlin
@Composable
fun ErrorContent(
    message: String,
    onRetry: () -> Unit,
    modifier: Modifier = Modifier,
) {
    Column(
        modifier = modifier,
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center,
    ) {
        Icon(
            imageVector = Icons.Filled.Warning,
            contentDescription = null,
            tint = MaterialTheme.colorScheme.error,
            modifier = Modifier.size(48.dp),
        )
        Spacer(modifier = Modifier.height(16.dp))
        Text(
            text = message,
            style = MaterialTheme.typography.bodyLarge,
            color = MaterialTheme.colorScheme.onSurface,
            textAlign = TextAlign.Center,
            modifier = Modifier.padding(horizontal = 32.dp),
        )
        Spacer(modifier = Modifier.height(24.dp))
        Button(onClick = onRetry) {
            Text(text = stringResource(R.string.retry))
        }
    }
}
```

```kotlin
@Composable
fun EmptyContent(
    title: String,
    description: String,
    modifier: Modifier = Modifier,
    icon: ImageVector = Icons.Filled.SearchOff,
) {
    Column(
        modifier = modifier,
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center,
    ) {
        Icon(
            imageVector = icon,
            contentDescription = null,
            tint = MaterialTheme.colorScheme.onSurfaceVariant,
            modifier = Modifier.size(64.dp),
        )
        Spacer(modifier = Modifier.height(16.dp))
        Text(
            text = title,
            style = MaterialTheme.typography.titleMedium,
            color = MaterialTheme.colorScheme.onSurface,
        )
        Spacer(modifier = Modifier.height(8.dp))
        Text(
            text = description,
            style = MaterialTheme.typography.bodyMedium,
            color = MaterialTheme.colorScheme.onSurfaceVariant,
            textAlign = TextAlign.Center,
            modifier = Modifier.padding(horizontal = 48.dp),
        )
    }
}
```

---

### Composable Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Screen | `{Feature}Screen` | `HomeScreen`, `UserProfileScreen` |
| Content | `{Feature}Content` | `HomeContent`, `UserProfileContent` |
| Component | Descriptive noun | `RestaurantCard`, `SearchBar` |
| Preview | `{Component}Preview` | `RestaurantCardPreview` |
| Shared | Descriptive noun | `LoadingIndicator`, `ErrorContent` |
| Dialog | `{Feature}Dialog` | `ConfirmDeleteDialog` |
| BottomSheet | `{Feature}BottomSheet` | `FilterBottomSheet` |

### Composable Parameter Ordering

Follow this consistent parameter ordering for every composable:

```kotlin
@Composable
fun ExampleComponent(
    // 1. Required state parameters
    title: String,
    isSelected: Boolean,
    // 2. Optional state parameters with defaults
    subtitle: String = "",
    // 3. Event callback lambdas
    onClick: () -> Unit,
    onLongClick: () -> Unit = {},
    // 4. Modifier (always last with default)
    modifier: Modifier = Modifier,
) {
    // ...
}
```

---

## 🧠 ViewModels

**ViewModels are the bridge between the Presentation Layer and the Domain Layer. They hold UI state, react to user actions, invoke Use Cases, and expose state via `StateFlow`.**

### @HiltViewModel Pattern

Every ViewModel follows the `@HiltViewModel` + `@Inject constructor` pattern for dependency injection via Hilt.

```kotlin
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getRestaurantsUseCase: GetRestaurantsUseCase,
    private val searchRestaurantsUseCase: SearchRestaurantsUseCase,
    private val toggleFavoriteUseCase: ToggleFavoriteUseCase,
) : ViewModel() {

    private val _uiState = MutableStateFlow<HomeUiState>(HomeUiState.Loading)
    val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()

    private val _events = MutableSharedFlow<HomeEvent>()
    val events: SharedFlow<HomeEvent> = _events.asSharedFlow()

    private var currentSearchQuery: String = ""

    init {
        loadRestaurants()
    }

    fun loadRestaurants() {
        viewModelScope.launch {
            _uiState.value = HomeUiState.Loading
            try {
                val restaurants = getRestaurantsUseCase()
                _uiState.value = HomeUiState.Success(
                    restaurants = restaurants,
                    isRefreshing = false,
                )
            } catch (e: Exception) {
                _uiState.value = HomeUiState.Error(
                    message = e.message ?: "Failed to load restaurants",
                )
            }
        }
    }

    fun onSearchQueryChanged(query: String) {
        currentSearchQuery = query
        viewModelScope.launch {
            try {
                val results = searchRestaurantsUseCase(query)
                _uiState.value = HomeUiState.Success(
                    restaurants = results,
                    isRefreshing = false,
                )
            } catch (e: Exception) {
                _uiState.value = HomeUiState.Error(
                    message = e.message ?: "Search failed",
                )
            }
        }
    }

    fun onRefresh() {
        viewModelScope.launch {
            val currentState = _uiState.value
            if (currentState is HomeUiState.Success) {
                _uiState.value = currentState.copy(isRefreshing = true)
            }
            try {
                val restaurants = if (currentSearchQuery.isNotEmpty()) {
                    searchRestaurantsUseCase(currentSearchQuery)
                } else {
                    getRestaurantsUseCase()
                }
                _uiState.value = HomeUiState.Success(
                    restaurants = restaurants,
                    isRefreshing = false,
                )
            } catch (e: Exception) {
                _uiState.value = HomeUiState.Error(
                    message = e.message ?: "Refresh failed",
                )
            }
        }
    }

    fun onRestaurantClicked(restaurantId: String) {
        viewModelScope.launch {
            _events.emit(HomeEvent.NavigateToDetail(restaurantId))
        }
    }

    fun onFavoriteToggled(restaurantId: String) {
        viewModelScope.launch {
            try {
                toggleFavoriteUseCase(restaurantId)
                loadRestaurants()
            } catch (e: Exception) {
                _events.emit(HomeEvent.ShowSnackbar("Failed to update favorite"))
            }
        }
    }
}
```

### ViewModel with SavedStateHandle

When a ViewModel needs to receive navigation arguments:

```kotlin
@HiltViewModel
class RestaurantDetailViewModel @Inject constructor(
    private val getRestaurantDetailUseCase: GetRestaurantDetailUseCase,
    private val getRestaurantReviewsUseCase: GetRestaurantReviewsUseCase,
    savedStateHandle: SavedStateHandle,
) : ViewModel() {

    private val restaurantId: String = checkNotNull(
        savedStateHandle["restaurantId"],
    )

    private val _uiState = MutableStateFlow<RestaurantDetailUiState>(
        RestaurantDetailUiState.Loading,
    )
    val uiState: StateFlow<RestaurantDetailUiState> = _uiState.asStateFlow()

    init {
        loadDetail()
    }

    fun loadDetail() {
        viewModelScope.launch {
            _uiState.value = RestaurantDetailUiState.Loading
            try {
                val detail = getRestaurantDetailUseCase(restaurantId)
                val reviews = getRestaurantReviewsUseCase(restaurantId)
                _uiState.value = RestaurantDetailUiState.Success(
                    restaurant = detail,
                    reviews = reviews,
                )
            } catch (e: Exception) {
                _uiState.value = RestaurantDetailUiState.Error(
                    message = e.message ?: "Failed to load details",
                )
            }
        }
    }
}
```

### ViewModel with Combine/Merge Flows

When a ViewModel needs to combine multiple data sources:

```kotlin
@HiltViewModel
class DashboardViewModel @Inject constructor(
    private val getUserProfileUseCase: GetUserProfileUseCase,
    private val getRecentOrdersUseCase: GetRecentOrdersUseCase,
    private val getNotificationsUseCase: GetNotificationsUseCase,
) : ViewModel() {

    private val _uiState = MutableStateFlow<DashboardUiState>(DashboardUiState.Loading)
    val uiState: StateFlow<DashboardUiState> = _uiState.asStateFlow()

    init {
        loadDashboard()
    }

    fun loadDashboard() {
        viewModelScope.launch {
            _uiState.value = DashboardUiState.Loading
            try {
                coroutineScope {
                    val profileDeferred = async { getUserProfileUseCase() }
                    val ordersDeferred = async { getRecentOrdersUseCase() }
                    val notificationsDeferred = async { getNotificationsUseCase() }

                    val profile = profileDeferred.await()
                    val orders = ordersDeferred.await()
                    val notifications = notificationsDeferred.await()

                    _uiState.value = DashboardUiState.Success(
                        profile = profile,
                        recentOrders = orders,
                        notifications = notifications,
                    )
                }
            } catch (e: Exception) {
                _uiState.value = DashboardUiState.Error(
                    message = e.message ?: "Failed to load dashboard",
                )
            }
        }
    }
}
```

### ViewModel Rules

- ✅ Use `@HiltViewModel` + `@Inject constructor`
- ✅ Expose immutable `StateFlow` (never `MutableStateFlow` publicly)
- ✅ Use `sealed interface` for UiState
- ✅ Use `viewModelScope` for all coroutines
- ✅ Delegate all business logic to Use Cases
- ✅ Use `SavedStateHandle` for navigation arguments
- ✅ Use `SharedFlow` for one-shot events (navigation, snackbar)
- ✅ Keep public functions as the only entry points for user actions
- ❌ **No** Android framework references (`Context`, `Activity`, `Fragment`, `View`)
- ❌ **No** business logic (validation, transformation, calculation)
- ❌ **No** direct repository access -- always go through Use Cases
- ❌ **No** `Navigator` / back stack reference -- emit events instead
- ❌ **No** `MutableStateFlow` exposed publicly
- ❌ **No** `LiveData` -- use `StateFlow` exclusively
- ❌ **No** `GlobalScope` or manual `CoroutineScope` creation

---

## 📊 UI State Pattern

### Sealed Interface for UiState

Every screen defines its own `UiState` as a sealed interface. This guarantees exhaustive handling in `when` expressions.

```kotlin
sealed interface HomeUiState {
    data object Loading : HomeUiState

    data class Success(
        val restaurants: List<Restaurant>,
        val isRefreshing: Boolean = false,
    ) : HomeUiState

    data class Error(val message: String) : HomeUiState
}
```

### Complex UiState with Multiple Sub-States

For screens with multiple independent sections:

```kotlin
sealed interface RestaurantDetailUiState {
    data object Loading : RestaurantDetailUiState

    data class Success(
        val restaurant: RestaurantDetail,
        val reviews: List<Review>,
        val isFavorite: Boolean = false,
        val selectedTab: DetailTab = DetailTab.MENU,
    ) : RestaurantDetailUiState

    data class Error(val message: String) : RestaurantDetailUiState
}

enum class DetailTab {
    MENU,
    REVIEWS,
    INFO,
}
```

### Form UiState

For screens with user input:

```kotlin
data class CreateReviewUiState(
    val rating: Int = 0,
    val comment: String = "",
    val isSubmitting: Boolean = false,
    val validationErrors: Map<String, String> = emptyMap(),
    val isSubmitEnabled: Boolean = false,
)
```

### One-Shot Events with SharedFlow

Events that should be consumed exactly once (navigation, snackbar, toast) use `SharedFlow`:

```kotlin
sealed interface HomeEvent {
    data class NavigateToDetail(val restaurantId: String) : HomeEvent
    data class ShowSnackbar(val message: String) : HomeEvent
    data object NavigateToSearch : HomeEvent
    data class ShareRestaurant(val url: String) : HomeEvent
}
```

### Collecting State and Events in Screen Composables

```kotlin
@Composable
fun HomeScreen(
    viewModel: HomeViewModel = hiltViewModel(),
    onNavigateToDetail: (String) -> Unit,
    onNavigateToSearch: () -> Unit,
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val snackbarHostState = remember { SnackbarHostState() }

    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is HomeEvent.NavigateToDetail -> {
                    onNavigateToDetail(event.restaurantId)
                }
                is HomeEvent.ShowSnackbar -> {
                    snackbarHostState.showSnackbar(event.message)
                }
                is HomeEvent.NavigateToSearch -> {
                    onNavigateToSearch()
                }
                is HomeEvent.ShareRestaurant -> { /* launch share intent */ }
            }
        }
    }

    HomeContent(
        uiState = uiState,
        snackbarHostState = snackbarHostState,
        onRestaurantClicked = viewModel::onRestaurantClicked,
        onSearchQueryChanged = viewModel::onSearchQueryChanged,
        onRefresh = viewModel::onRefresh,
        onFavoriteToggled = viewModel::onFavoriteToggled,
    )
}
```

### UiState Rules

- ✅ Always use `sealed interface` for screen-level UiState
- ✅ Always include `Loading`, `Success`, and `Error` variants
- ✅ Use `data object` for state variants without data (e.g., `Loading`)
- ✅ Use `data class` for state variants with data
- ✅ Default values for optional fields in `Success` states
- ✅ Handle every branch in `when` expressions
- ❌ Never use nullable UiState (`UiState?`)
- ❌ Never combine unrelated state into a single data class

---

## 🧭 Navigation (Navigation 3)

ARC Labs apps use **Jetpack Navigation 3**. Full guide: [Navigation 3](../Architecture/navigation-3.md). The essentials for the presentation layer:

### Keys

```kotlin
// Keys.kt (owned by the feature module)
@Serializable
data object HomeKey : NavKey

@Serializable
data object SearchKey : NavKey

@Serializable
data class RestaurantDetailKey(val restaurantId: String) : NavKey

@Serializable
data class UserProfileKey(val userId: String) : NavKey

@Serializable
data object SettingsKey : NavKey
```

### Entries

Each feature contributes entries to the app's `entryProvider`; the Screen composables only receive lambdas.

```kotlin
// LibraryEntries.kt
fun EntryProviderScope<NavKey>.libraryEntries(navigator: Navigator) {
    entry<HomeKey> {
        HomeRoute(
            onNavigateToDetail = { restaurantId -> navigator.navigate(RestaurantDetailKey(restaurantId)) },
            onNavigateToSearch = { navigator.navigate(SearchKey) },
            onNavigateToProfile = { userId -> navigator.navigate(UserProfileKey(userId)) },
        )
    }
    entry<RestaurantDetailKey> { key ->
        val viewModel = hiltViewModel<RestaurantDetailViewModel, RestaurantDetailViewModel.Factory>(
            creationCallback = { factory -> factory.create(key) },
        )
        RestaurantDetailRoute(viewModel = viewModel, onNavigateBack = navigator::goBack)
    }
}
```

### Top-Level Navigation

Tabs use `NavigationSuiteScaffold` (bottom bar on compact windows, navigation rail on larger ones) with one back stack per top-level key. See [Navigation 3 → Top-Level Tabs](../Architecture/navigation-3.md#-top-level-tabs-with-navigationsuitescaffold) for the `NavigationState` / `Navigator` holder and the shell.

### Navigation Rules

- ✅ Keys are `@Serializable` and implement `NavKey`; they carry ids only
- ✅ Pass navigation callbacks as lambdas to Screen composables
- ✅ Dialogs and bottom sheets are keys rendered by scene strategies
- ✅ One `ViewModel` per entry (`rememberViewModelStoreNavEntryDecorator`), key injected with Hilt assisted injection
- ❌ **Never** pass the `Navigator`, `NavBackStack` or `NavigationState` into a ViewModel
- ❌ **Never** pass navigation objects deeper than the Route composable
- ❌ **Never** use Navigation 2 (`NavHost`, `NavController`) or string routes

---

## 🎨 Theming (Material 3 Expressive)

### Color Scheme

```kotlin
// Color.kt
val md_theme_light_primary = Color(0xFF6750A4)
val md_theme_light_onPrimary = Color(0xFFFFFFFF)
val md_theme_light_primaryContainer = Color(0xFFEADDFF)
val md_theme_light_onPrimaryContainer = Color(0xFF21005D)
val md_theme_light_secondary = Color(0xFF625B71)
val md_theme_light_onSecondary = Color(0xFFFFFFFF)
val md_theme_light_secondaryContainer = Color(0xFFE8DEF8)
val md_theme_light_onSecondaryContainer = Color(0xFF1D192B)
val md_theme_light_error = Color(0xFFB3261E)
val md_theme_light_onError = Color(0xFFFFFFFF)
val md_theme_light_background = Color(0xFFFFFBFE)
val md_theme_light_onBackground = Color(0xFF1C1B1F)
val md_theme_light_surface = Color(0xFFFFFBFE)
val md_theme_light_onSurface = Color(0xFF1C1B1F)

val md_theme_dark_primary = Color(0xFFD0BCFF)
val md_theme_dark_onPrimary = Color(0xFF381E72)
val md_theme_dark_primaryContainer = Color(0xFF4F378B)
val md_theme_dark_onPrimaryContainer = Color(0xFFEADDFF)
val md_theme_dark_secondary = Color(0xFFCCC2DC)
val md_theme_dark_onSecondary = Color(0xFF332D41)
val md_theme_dark_secondaryContainer = Color(0xFF4A4458)
val md_theme_dark_onSecondaryContainer = Color(0xFFE8DEF8)
val md_theme_dark_error = Color(0xFFF2B8B5)
val md_theme_dark_onError = Color(0xFF601410)
val md_theme_dark_background = Color(0xFF1C1B1F)
val md_theme_dark_onBackground = Color(0xFFE6E1E5)
val md_theme_dark_surface = Color(0xFF1C1B1F)
val md_theme_dark_onSurface = Color(0xFFE6E1E5)

val LightColorScheme = lightColorScheme(
    primary = md_theme_light_primary,
    onPrimary = md_theme_light_onPrimary,
    primaryContainer = md_theme_light_primaryContainer,
    onPrimaryContainer = md_theme_light_onPrimaryContainer,
    secondary = md_theme_light_secondary,
    onSecondary = md_theme_light_onSecondary,
    secondaryContainer = md_theme_light_secondaryContainer,
    onSecondaryContainer = md_theme_light_onSecondaryContainer,
    error = md_theme_light_error,
    onError = md_theme_light_onError,
    background = md_theme_light_background,
    onBackground = md_theme_light_onBackground,
    surface = md_theme_light_surface,
    onSurface = md_theme_light_onSurface,
)

val DarkColorScheme = darkColorScheme(
    primary = md_theme_dark_primary,
    onPrimary = md_theme_dark_onPrimary,
    primaryContainer = md_theme_dark_primaryContainer,
    onPrimaryContainer = md_theme_dark_onPrimaryContainer,
    secondary = md_theme_dark_secondary,
    onSecondary = md_theme_dark_onSecondary,
    secondaryContainer = md_theme_dark_secondaryContainer,
    onSecondaryContainer = md_theme_dark_onSecondaryContainer,
    error = md_theme_dark_error,
    onError = md_theme_dark_onError,
    background = md_theme_dark_background,
    onBackground = md_theme_dark_onBackground,
    surface = md_theme_dark_surface,
    onSurface = md_theme_dark_onSurface,
)
```

### Typography

```kotlin
// Type.kt
val AppTypography = Typography(
    displayLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 57.sp,
        lineHeight = 64.sp,
        letterSpacing = (-0.25).sp,
    ),
    headlineLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 32.sp,
        lineHeight = 40.sp,
    ),
    titleLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 22.sp,
        lineHeight = 28.sp,
    ),
    titleMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.15.sp,
    ),
    bodyLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp,
    ),
    bodyMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.25.sp,
    ),
    labelLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.1.sp,
    ),
    labelMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 12.sp,
        lineHeight = 16.sp,
        letterSpacing = 0.5.sp,
    ),
)
```

### Shapes

```kotlin
// Shape.kt
val AppShapes = Shapes(
    extraSmall = RoundedCornerShape(4.dp),
    small = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(12.dp),
    large = RoundedCornerShape(16.dp),
    extraLarge = RoundedCornerShape(28.dp),
)
```

### Theme Composable

ARC Labs uses **Material 3 Expressive** with the brand color scheme — **dynamic color off**. Full guidance: [UI Guidelines → Material 3 Expressive](../Quality/ui-guidelines.md#material-3-expressive).

```kotlin
// Theme.kt (:core:designsystem) — verify MaterialExpressiveTheme parameters/opt-in for material3 1.5.0-beta01
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit,
) {
    MaterialExpressiveTheme(
        colorScheme = if (darkTheme) DarkColorScheme else LightColorScheme,
        motionScheme = MotionScheme.expressive(),
        typography = AppTypography,
        shapes = AppShapes,
        content = content,
    )
}
```

### Theming Rules

- ✅ Always use `MaterialTheme.colorScheme` for colors
- ✅ Always use `MaterialTheme.typography` for text styles
- ✅ Always use `MaterialTheme.shapes` for shapes
- ✅ Brand light and dark color schemes; dynamic color is off
- ✅ Prefer Material 3 Expressive components and motion over custom implementations
- ❌ **Never** hardcode colors directly in composables (literals live only in `:core:designsystem`)
- ❌ **Never** hardcode text sizes directly
- ❌ **Never** use deprecated `MaterialTheme.colors` (Material 2)

---

## ♿ Accessibility

Accessibility is not optional. Every composable must be usable with TalkBack, Switch Access, and other assistive technologies.

### Content Descriptions

Every non-decorative `Icon` and `Image` must have a meaningful `contentDescription`:

```kotlin
// ✅ Correct -- meaningful description
Icon(
    imageVector = Icons.Filled.Favorite,
    contentDescription = stringResource(R.string.add_to_favorites),
    tint = MaterialTheme.colorScheme.primary,
)

// ✅ Correct -- decorative icon, null is appropriate
Icon(
    imageVector = Icons.Filled.Star,
    contentDescription = null, // Decorative, rating value is in adjacent text
)

// ❌ Wrong -- missing description for interactive icon
Icon(
    imageVector = Icons.Filled.Delete,
    contentDescription = null, // NOT OK for interactive icons!
)
```

### Semantic Properties

Use `Modifier.semantics` for custom accessibility behavior:

```kotlin
@Composable
fun RatingBar(
    rating: Float,
    maxRating: Int = 5,
    modifier: Modifier = Modifier,
) {
    Row(
        modifier = modifier.semantics {
            contentDescription = "Rating: $rating out of $maxRating stars"
            stateDescription = "$rating stars"
        },
    ) {
        repeat(maxRating) { index ->
            Icon(
                imageVector = if (index < rating.toInt()) {
                    Icons.Filled.Star
                } else {
                    Icons.Outlined.Star
                },
                contentDescription = null, // Parent has the semantic info
                tint = if (index < rating.toInt()) {
                    MaterialTheme.colorScheme.primary
                } else {
                    MaterialTheme.colorScheme.onSurfaceVariant
                },
                modifier = Modifier.size(20.dp),
            )
        }
    }
}
```

### Minimum Touch Target

All clickable elements must have a minimum touch target of 48dp:

```kotlin
// ✅ Correct -- IconButton guarantees 48dp touch target
IconButton(onClick = onDeleteClicked) {
    Icon(
        imageVector = Icons.Filled.Delete,
        contentDescription = stringResource(R.string.delete_item),
    )
}

// ❌ Wrong -- Icon is only 24dp, no touch target guarantee
Icon(
    imageVector = Icons.Filled.Delete,
    contentDescription = stringResource(R.string.delete_item),
    modifier = Modifier
        .clickable { onDeleteClicked() }
        .size(24.dp), // Too small!
)

// ✅ Correct -- using sizeIn to guarantee minimum touch target
Icon(
    imageVector = Icons.Filled.Delete,
    contentDescription = stringResource(R.string.delete_item),
    modifier = Modifier
        .clickable { onDeleteClicked() }
        .sizeIn(minWidth = 48.dp, minHeight = 48.dp),
)
```

### Heading Semantics

Mark screen titles and section headers for TalkBack navigation:

```kotlin
Text(
    text = stringResource(R.string.section_title),
    style = MaterialTheme.typography.headlineMedium,
    modifier = Modifier.semantics { heading() },
)
```

### Accessibility Checklist

- ✅ All interactive elements have `contentDescription`
- ✅ All clickable areas are at least 48dp x 48dp
- ✅ All text uses `stringResource()` (supports localization)
- ✅ Color is never the only indicator of state
- ✅ Screen titles use heading semantics
- ✅ Lists announce item count via `Modifier.semantics`
- ✅ Loading states announced via `LiveRegion`

---

## 🧪 Testing Presentation Layer

### Testing ViewModels with Turbine

Turbine is the standard library for testing `Flow` emissions, including `StateFlow` and `SharedFlow`.

```kotlin
class HomeViewModelTest {

    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()

    private val getRestaurantsUseCase: GetRestaurantsUseCase = mockk()
    private val searchRestaurantsUseCase: SearchRestaurantsUseCase = mockk()
    private val toggleFavoriteUseCase: ToggleFavoriteUseCase = mockk()

    private fun createSUT(
        getRestaurants: GetRestaurantsUseCase = getRestaurantsUseCase,
        searchRestaurants: SearchRestaurantsUseCase = searchRestaurantsUseCase,
        toggleFavorite: ToggleFavoriteUseCase = toggleFavoriteUseCase,
    ) = HomeViewModel(
        getRestaurantsUseCase = getRestaurants,
        searchRestaurantsUseCase = searchRestaurants,
        toggleFavoriteUseCase = toggleFavorite,
    )

    @Test
    fun `loadRestaurants emits Loading then Success when use case succeeds`() = runTest {
        // Given
        val restaurants = listOf(
            Restaurant.fake(id = "1", name = "Pizza Place"),
            Restaurant.fake(id = "2", name = "Burger Joint"),
        )
        coEvery { getRestaurantsUseCase() } returns restaurants

        // When
        val viewModel = createSUT()

        // Then
        viewModel.uiState.test {
            assertEquals(HomeUiState.Loading, awaitItem())
            assertEquals(
                HomeUiState.Success(restaurants = restaurants, isRefreshing = false),
                awaitItem(),
            )
        }
    }

    @Test
    fun `loadRestaurants emits Loading then Error when use case fails`() = runTest {
        // Given
        coEvery { getRestaurantsUseCase() } throws RuntimeException("Network error")

        // When
        val viewModel = createSUT()

        // Then
        viewModel.uiState.test {
            assertEquals(HomeUiState.Loading, awaitItem())
            val errorState = awaitItem()
            assertTrue(errorState is HomeUiState.Error)
            assertEquals("Network error", (errorState as HomeUiState.Error).message)
        }
    }

    @Test
    fun `onSearchQueryChanged emits Success with search results`() = runTest {
        // Given
        val allRestaurants = listOf(Restaurant.fake(name = "Pizza Place"))
        val searchResults = listOf(Restaurant.fake(name = "Pizza Place"))
        coEvery { getRestaurantsUseCase() } returns allRestaurants
        coEvery { searchRestaurantsUseCase("pizza") } returns searchResults

        val viewModel = createSUT()

        viewModel.uiState.test {
            skipItems(2) // Skip Loading + initial Success

            // When
            viewModel.onSearchQueryChanged("pizza")

            // Then
            assertEquals(
                HomeUiState.Success(restaurants = searchResults, isRefreshing = false),
                awaitItem(),
            )
        }
    }

    @Test
    fun `onRestaurantClicked emits NavigateToDetail event`() = runTest {
        // Given
        coEvery { getRestaurantsUseCase() } returns emptyList()
        val viewModel = createSUT()

        // When & Then
        viewModel.events.test {
            viewModel.onRestaurantClicked("restaurant-123")
            assertEquals(
                HomeEvent.NavigateToDetail("restaurant-123"),
                awaitItem(),
            )
        }
    }

    @Test
    fun `onFavoriteToggled emits ShowSnackbar when use case fails`() = runTest {
        // Given
        coEvery { getRestaurantsUseCase() } returns emptyList()
        coEvery { toggleFavoriteUseCase(any()) } throws RuntimeException("Failed")
        val viewModel = createSUT()

        // When & Then
        viewModel.events.test {
            viewModel.onFavoriteToggled("restaurant-123")
            assertEquals(
                HomeEvent.ShowSnackbar("Failed to update favorite"),
                awaitItem(),
            )
        }
    }
}
```

### MainDispatcherRule for Testing

```kotlin
class MainDispatcherRule(
    private val testDispatcher: TestDispatcher = UnconfinedTestDispatcher(),
) : TestWatcher() {

    override fun starting(description: Description) {
        Dispatchers.setMain(testDispatcher)
    }

    override fun finished(description: Description) {
        Dispatchers.resetMain()
    }
}
```

### Testing Composables with Compose Test

```kotlin
class RestaurantCardTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun `displays restaurant name`() {
        // Given
        val restaurant = Restaurant.fake(name = "Pizza Place")

        // When
        composeTestRule.setContent {
            AppTheme {
                RestaurantCard(
                    restaurant = restaurant,
                    onClick = {},
                )
            }
        }

        // Then
        composeTestRule
            .onNodeWithText("Pizza Place")
            .assertIsDisplayed()
    }

    @Test
    fun `displays cuisine type`() {
        // Given
        val restaurant = Restaurant.fake(
            cuisineType = CuisineType.ITALIAN,
        )

        // When
        composeTestRule.setContent {
            AppTheme {
                RestaurantCard(
                    restaurant = restaurant,
                    onClick = {},
                )
            }
        }

        // Then
        composeTestRule
            .onNodeWithText("Italian")
            .assertIsDisplayed()
    }

    @Test
    fun `invokes onClick when card is tapped`() {
        // Given
        var clicked = false
        val restaurant = Restaurant.fake()

        composeTestRule.setContent {
            AppTheme {
                RestaurantCard(
                    restaurant = restaurant,
                    onClick = { clicked = true },
                )
            }
        }

        // When
        composeTestRule
            .onNodeWithText(restaurant.name)
            .performClick()

        // Then
        assertTrue(clicked)
    }

    @Test
    fun `displays rating`() {
        // Given
        val restaurant = Restaurant.fake(rating = 4.5f)

        composeTestRule.setContent {
            AppTheme {
                RestaurantCard(
                    restaurant = restaurant,
                    onClick = {},
                )
            }
        }

        // Then
        composeTestRule
            .onNodeWithText("4.5")
            .assertIsDisplayed()
    }
}
```

### Testing Screen Composables with State Variants

```kotlin
class HomeContentTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun `shows loading indicator when state is Loading`() {
        composeTestRule.setContent {
            AppTheme {
                HomeContent(
                    uiState = HomeUiState.Loading,
                    snackbarHostState = SnackbarHostState(),
                    onRestaurantClicked = {},
                    onSearchQueryChanged = {},
                    onRefresh = {},
                    onFavoriteToggled = {},
                )
            }
        }

        composeTestRule
            .onNode(hasTestTag("loading_indicator"))
            .assertIsDisplayed()
    }

    @Test
    fun `shows error content with retry button when state is Error`() {
        composeTestRule.setContent {
            AppTheme {
                HomeContent(
                    uiState = HomeUiState.Error("Something went wrong"),
                    snackbarHostState = SnackbarHostState(),
                    onRestaurantClicked = {},
                    onSearchQueryChanged = {},
                    onRefresh = {},
                    onFavoriteToggled = {},
                )
            }
        }

        composeTestRule
            .onNodeWithText("Something went wrong")
            .assertIsDisplayed()

        composeTestRule
            .onNodeWithText("Retry")
            .assertIsDisplayed()
    }

    @Test
    fun `shows restaurant list when state is Success`() {
        val restaurants = listOf(
            Restaurant.fake(name = "Pizza Place"),
            Restaurant.fake(name = "Burger Joint"),
        )

        composeTestRule.setContent {
            AppTheme {
                HomeContent(
                    uiState = HomeUiState.Success(
                        restaurants = restaurants,
                        isRefreshing = false,
                    ),
                    snackbarHostState = SnackbarHostState(),
                    onRestaurantClicked = {},
                    onSearchQueryChanged = {},
                    onRefresh = {},
                    onFavoriteToggled = {},
                )
            }
        }

        composeTestRule
            .onNodeWithText("Pizza Place")
            .assertIsDisplayed()

        composeTestRule
            .onNodeWithText("Burger Joint")
            .assertIsDisplayed()
    }
}
```

### Testing Rules

- ✅ Use `createSUT()` factory method in every test class
- ✅ Follow Given-When-Then structure
- ✅ Use Turbine `.test {}` for StateFlow and SharedFlow assertions
- ✅ Use `MainDispatcherRule` for ViewModel tests
- ✅ Use `createComposeRule()` from `androidx.compose.ui.test.junit4.v2` for composable tests (non-v2 deprecated in Compose 1.12)
- ✅ Test all UiState branches (Loading, Success, Error)
- ✅ Test user interactions (clicks, input)
- ✅ Use `Restaurant.fake()` and similar factory methods for test data
- ❌ Never test implementation details (private fields, internal state)
- ❌ Never use `Thread.sleep()` -- use Turbine's `awaitItem()`

---

## 📐 Preview Best Practices

### Single Preview

```kotlin
@Preview(showBackground = true)
@Composable
private fun RestaurantCardPreview() {
    AppTheme {
        RestaurantCard(
            restaurant = Restaurant.fake(
                name = "Trattoria Roma",
                cuisineType = CuisineType.ITALIAN,
                rating = 4.5f,
            ),
            onClick = {},
        )
    }
}
```

### Multi-Configuration Previews

```kotlin
@Preview(
    name = "Light",
    showBackground = true,
)
@Preview(
    name = "Dark",
    showBackground = true,
    uiMode = UI_MODE_NIGHT_YES,
)
@Composable
private fun RestaurantCardPreview() {
    AppTheme {
        RestaurantCard(
            restaurant = Restaurant.fake(),
            onClick = {},
        )
    }
}
```

### Preview with Different States

```kotlin
@Preview(showBackground = true)
@Composable
private fun HomeContentLoadingPreview() {
    AppTheme {
        HomeContent(
            uiState = HomeUiState.Loading,
            snackbarHostState = SnackbarHostState(),
            onRestaurantClicked = {},
            onSearchQueryChanged = {},
            onRefresh = {},
            onFavoriteToggled = {},
        )
    }
}

@Preview(showBackground = true)
@Composable
private fun HomeContentSuccessPreview() {
    AppTheme {
        HomeContent(
            uiState = HomeUiState.Success(
                restaurants = listOf(
                    Restaurant.fake(name = "Pizza Place"),
                    Restaurant.fake(name = "Sushi Bar"),
                    Restaurant.fake(name = "Taco Stand"),
                ),
                isRefreshing = false,
            ),
            snackbarHostState = SnackbarHostState(),
            onRestaurantClicked = {},
            onSearchQueryChanged = {},
            onRefresh = {},
            onFavoriteToggled = {},
        )
    }
}

@Preview(showBackground = true)
@Composable
private fun HomeContentErrorPreview() {
    AppTheme {
        HomeContent(
            uiState = HomeUiState.Error("Failed to load restaurants. Please check your connection."),
            snackbarHostState = SnackbarHostState(),
            onRestaurantClicked = {},
            onSearchQueryChanged = {},
            onRefresh = {},
            onFavoriteToggled = {},
        )
    }
}
```

### Preview Annotation for Reuse

```kotlin
@Preview(
    name = "Phone",
    showBackground = true,
    device = Devices.PIXEL_7,
)
@Preview(
    name = "Phone Dark",
    showBackground = true,
    device = Devices.PIXEL_7,
    uiMode = UI_MODE_NIGHT_YES,
)
@Preview(
    name = "Tablet",
    showBackground = true,
    device = Devices.PIXEL_TABLET,
)
@Preview(
    name = "Foldable",
    showBackground = true,
    device = Devices.PIXEL_FOLD,
)
annotation class DevicePreview
```

Usage:

```kotlin
@DevicePreview
@Composable
private fun HomeContentPreview() {
    AppTheme {
        HomeContent(
            uiState = HomeUiState.Success(
                restaurants = listOf(Restaurant.fake()),
                isRefreshing = false,
            ),
            snackbarHostState = SnackbarHostState(),
            onRestaurantClicked = {},
            onSearchQueryChanged = {},
            onRefresh = {},
            onFavoriteToggled = {},
        )
    }
}
```

### Preview Rules

- ✅ Wrap every preview in `AppTheme {}`
- ✅ Provide both light and dark previews
- ✅ Preview all UiState branches
- ✅ Use `Restaurant.fake()` and similar factory methods
- ✅ Mark preview functions as `private`
- ✅ Name previews descriptively: `{Component}{State}Preview`
- ❌ Never use real data or API calls in previews
- ❌ Never make preview functions public

---

## ✅ Presentation Layer Checklist

Use this checklist when reviewing or creating Presentation Layer code.

### Composables

- [ ] Screen composables follow `{Feature}Screen` naming
- [ ] Content composables are stateless -- no ViewModel reference
- [ ] Component composables accept `modifier: Modifier = Modifier` as last parameter
- [ ] All strings use `stringResource()` -- no hardcoded text
- [ ] All colors reference `MaterialTheme.colorScheme`
- [ ] All text styles reference `MaterialTheme.typography`
- [ ] All shapes reference `MaterialTheme.shapes`
- [ ] Interactive elements have `contentDescription`
- [ ] Touch targets are at least 48dp x 48dp
- [ ] Previews exist for all components in light and dark mode
- [ ] Previews exist for all UiState branches

### ViewModels

- [ ] Annotated with `@HiltViewModel`
- [ ] Uses `@Inject constructor` for dependencies
- [ ] Exposes only immutable `StateFlow` (not `MutableStateFlow`)
- [ ] Uses `sealed interface` for UiState
- [ ] Uses `SharedFlow` for one-shot events
- [ ] All coroutines launched in `viewModelScope`
- [ ] All business logic delegated to Use Cases
- [ ] No Android framework imports (`android.content`, `android.app`, etc.)
- [ ] No direct repository access
- [ ] Navigation handled via events or lambdas, never navigation objects

### Navigation

- [ ] Keys defined as `@Serializable` data objects/classes implementing `NavKey`
- [ ] Navigation callbacks passed as lambdas
- [ ] `Navigator` / back stack never passed beyond Route composables
- [ ] Top-level tabs use `NavigationSuiteScaffold` with one back stack per tab
- [ ] Deep links configured where needed

### Testing

- [ ] ViewModel tests use `MainDispatcherRule`
- [ ] ViewModel tests use Turbine for Flow assertions
- [ ] Composable tests use `createComposeRule()` (`junit4.v2`)
- [ ] All UiState branches tested
- [ ] User interactions tested (click, input, scroll)
- [ ] One-shot events tested
- [ ] `createSUT()` factory method used
- [ ] Given-When-Then structure followed

---

## 🚫 Common Mistakes

### 1. Business Logic in Composable

```kotlin
// ❌ Wrong -- validation logic in composable
@Composable
fun CheckoutScreen(viewModel: CheckoutViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    var email by remember { mutableStateOf("") }
    var isEmailValid by remember { mutableStateOf(false) }

    // Business logic does NOT belong here!
    isEmailValid = email.contains("@") && email.contains(".")

    TextField(
        value = email,
        onValueChange = { email = it },
        isError = !isEmailValid,
    )
}

// ✅ Correct -- validation delegated to ViewModel/UseCase
@Composable
fun CheckoutScreen(viewModel: CheckoutViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    CheckoutContent(
        uiState = uiState,
        onEmailChanged = viewModel::onEmailChanged,
        onSubmit = viewModel::onSubmit,
    )
}
```

### 2. Mutable State Exposed from ViewModel

```kotlin
// ❌ Wrong -- MutableStateFlow exposed publicly
@HiltViewModel
class ProfileViewModel @Inject constructor(
    private val getUserProfileUseCase: GetUserProfileUseCase,
) : ViewModel() {
    val uiState = MutableStateFlow<ProfileUiState>(ProfileUiState.Loading) // Mutable!
}

// ✅ Correct -- immutable StateFlow exposed
@HiltViewModel
class ProfileViewModel @Inject constructor(
    private val getUserProfileUseCase: GetUserProfileUseCase,
) : ViewModel() {
    private val _uiState = MutableStateFlow<ProfileUiState>(ProfileUiState.Loading)
    val uiState: StateFlow<ProfileUiState> = _uiState.asStateFlow()
}
```

### 3. Not Using collectAsStateWithLifecycle

```kotlin
// ❌ Wrong -- collectAsState does not respect lifecycle
@Composable
fun HomeScreen(viewModel: HomeViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsState() // Continues collecting in background!
    // ...
}

// ✅ Correct -- collectAsStateWithLifecycle respects lifecycle
@Composable
fun HomeScreen(viewModel: HomeViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    // Collection stops when lifecycle goes below STARTED
    // ...
}
```

### 4. Hardcoded Strings Instead of String Resources

```kotlin
// ❌ Wrong -- hardcoded strings
@Composable
fun ErrorContent(message: String, onRetry: () -> Unit) {
    Column {
        Text("Something went wrong")  // Not localizable!
        Button(onClick = onRetry) {
            Text("Try Again")  // Not localizable!
        }
    }
}

// ✅ Correct -- string resources
@Composable
fun ErrorContent(message: String, onRetry: () -> Unit) {
    Column {
        Text(stringResource(R.string.error_generic_title))
        Text(message)
        Button(onClick = onRetry) {
            Text(stringResource(R.string.retry))
        }
    }
}
```

### 5. Missing Content Descriptions on Interactive Icons

```kotlin
// ❌ Wrong -- no content description on clickable icon
IconButton(onClick = onDeleteClicked) {
    Icon(
        imageVector = Icons.Filled.Delete,
        contentDescription = null,  // TalkBack cannot describe this!
    )
}

// ✅ Correct -- meaningful content description
IconButton(onClick = onDeleteClicked) {
    Icon(
        imageVector = Icons.Filled.Delete,
        contentDescription = stringResource(R.string.delete_restaurant),
    )
}
```

### 6. Navigation Objects Passed into ViewModel

```kotlin
// ❌ Wrong -- ViewModel holds the navigation back stack
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val backStack: NavBackStack<NavKey>,  // UI navigation state in the ViewModel!
) : ViewModel() {
    fun onItemClicked(id: String) {
        backStack.add(RestaurantDetailKey(id))  // ViewModel should NOT navigate!
    }
}

// ✅ Correct -- ViewModel emits navigation events
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getRestaurantsUseCase: GetRestaurantsUseCase,
) : ViewModel() {
    private val _events = MutableSharedFlow<HomeEvent>()
    val events: SharedFlow<HomeEvent> = _events.asSharedFlow()

    fun onItemClicked(id: String) {
        viewModelScope.launch {
            _events.emit(HomeEvent.NavigateToDetail(id))
        }
    }
}
```

### 7. Using LiveData Instead of StateFlow

```kotlin
// ❌ Wrong -- LiveData is legacy in Compose-first apps
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getRestaurantsUseCase: GetRestaurantsUseCase,
) : ViewModel() {
    private val _uiState = MutableLiveData<HomeUiState>(HomeUiState.Loading)
    val uiState: LiveData<HomeUiState> = _uiState
}

// ✅ Correct -- StateFlow is the standard for Compose
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getRestaurantsUseCase: GetRestaurantsUseCase,
) : ViewModel() {
    private val _uiState = MutableStateFlow<HomeUiState>(HomeUiState.Loading)
    val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()
}
```

---

## 📚 Further Reading

### Official Documentation

- [Jetpack Compose Documentation](https://developer.android.com/develop/ui/compose)
- [State and Jetpack Compose](https://developer.android.com/develop/ui/compose/state)
- [Side Effects in Compose](https://developer.android.com/develop/ui/compose/side-effects)
- [Navigation in Compose](https://developer.android.com/guide/navigation/design/type-safety)
- [ViewModel Overview](https://developer.android.com/topic/libraries/architecture/viewmodel)
- [StateFlow and SharedFlow](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow)
- [Lifecycle-Aware Collection](https://developer.android.com/topic/libraries/architecture/lifecycle)
- [Material Design 3 for Compose](https://developer.android.com/develop/ui/compose/designsystems/material3)
- [Accessibility in Compose](https://developer.android.com/develop/ui/compose/accessibility)

### Testing

- [Testing Compose Layouts](https://developer.android.com/develop/ui/compose/testing)
- [Turbine - Flow Testing Library](https://github.com/cashapp/turbine)
- [MockK - Mocking Library for Kotlin](https://mockk.io/)

### Architecture

- [Guide to App Architecture](https://developer.android.com/topic/architecture)
- [UI Layer Documentation](https://developer.android.com/topic/architecture/ui-layer)
- [UI Events](https://developer.android.com/topic/architecture/ui-layer/events)
- [UI State Production](https://developer.android.com/topic/architecture/ui-layer/state-production)

### Libraries

- [Hilt - Dependency Injection](https://developer.android.com/training/dependency-injection/hilt-android)
- [Coil - Image Loading for Compose](https://coil-kt.github.io/coil/compose/)
- [Accompanist - Compose Utilities](https://google.github.io/accompanist/)
