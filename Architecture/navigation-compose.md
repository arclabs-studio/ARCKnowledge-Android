# 🧭 Navigation Compose

**Type-safe navigation patterns for Jetpack Compose using Navigation 2.8.0+ with kotlinx.serialization. This guide covers route definitions, NavHost setup, nested graphs, deep links, bottom navigation, ViewModel-driven navigation events, and testing strategies for Android apps built with ARC Labs standards.**

---

## 🎯 Navigation in Android

### Overview of Navigation Compose

Navigation Compose is the official Jetpack library for handling in-app navigation in Compose-based Android applications. It provides a declarative API for defining navigation graphs, handling argument passing between destinations, deep link support, and integration with the system back button.

Starting with Navigation **2.8.0**, the library introduced **type-safe navigation** using Kotlin serialization. This eliminates the error-prone string-based route system that was previously required, replacing it with strongly-typed route objects and data classes.

### Why Type-Safe Navigation

Before Navigation 2.8.0, routes were defined as strings with manually constructed argument placeholders:

```kotlin
// ❌ Old string-based approach (Navigation < 2.8.0)
composable("user_profile/{userId}") { backStackEntry ->
    val userId = backStackEntry.arguments?.getString("userId") ?: ""
    UserProfileScreen(userId = userId)
}

// Navigating required manual string construction
navController.navigate("user_profile/$userId")
```

This approach had significant drawbacks:

- **No compile-time safety** - Typos in route strings cause runtime crashes
- **Manual argument parsing** - Extracting arguments from the back stack entry is verbose and error-prone
- **No default values** - Optional arguments required complex route patterns with query parameters
- **Refactoring risk** - Renaming a route required updating every string reference

Type-safe navigation solves all of these problems:

```kotlin
// ✅ Type-safe approach (Navigation 2.8.0+)
@Serializable
data class UserProfile(val userId: String)

composable<UserProfile> { backStackEntry ->
    val route: UserProfile = backStackEntry.toRoute()
    UserProfileScreen(userId = route.userId)
}

// Navigating is type-safe
navController.navigate(UserProfile(userId = "abc123"))
```

### Comparison: iOS NavigationStack/Router vs Android NavHost/NavController

| Concept | iOS (SwiftUI) | Android (Compose) |
|---|---|---|
| Navigation container | `NavigationStack` | `NavHost` |
| Navigation state | `NavigationPath` | `NavHostController` |
| Route definition | `enum` or `Hashable` struct | `@Serializable` object/data class |
| Navigate forward | `path.append(route)` | `navController.navigate(route)` |
| Navigate back | `dismiss()` / `path.removeLast()` | `navController.popBackStack()` |
| Nested graphs | Nested `NavigationStack` | `navigation<Graph>(...)` |
| Deep links | `onOpenURL` | `navDeepLink` |
| Tab navigation | `TabView` | `NavigationBar` + per-tab `NavHost` |
| Coordinator pattern | Router/Coordinator | ViewModel navigation events |

Both platforms share the philosophy of declarative navigation with typed routes. The key difference is that iOS uses value-type enums with associated values while Android uses serializable classes. The navigation controller concept is similar, with `NavHostController` serving the same role as `NavigationPath` combined with the `NavigationStack` environment.

---

## 🚀 Setup

### Dependencies

Add the Navigation Compose and Kotlin Serialization dependencies to your project.

**Project-level `build.gradle.kts`:**

```kotlin
plugins {
    alias(libs.plugins.kotlin.serialization) apply false
}
```

**Module-level `build.gradle.kts`:**

```kotlin
plugins {
    alias(libs.plugins.kotlin.serialization)
}

dependencies {
    implementation(libs.androidx.navigation.compose)
    implementation(libs.kotlinx.serialization.json)
}
```

**Version catalog (`libs.versions.toml`):**

```toml
[versions]
navigation-compose = "2.8.5"
kotlinx-serialization = "1.7.3"
kotlin = "2.1.0"

[libraries]
androidx-navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "navigation-compose" }
kotlinx-serialization-json = { group = "org.jetbrains.kotlinx", name = "kotlinx-serialization-json", version.ref = "kotlinx-serialization" }

[plugins]
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
```

### Gradle Plugin Setup

The Kotlin Serialization compiler plugin is required for `@Serializable` annotations to work. Make sure both the plugin and the runtime library are configured:

1. Apply the `kotlin-serialization` plugin to every module that defines `@Serializable` route classes
2. Add the `kotlinx-serialization-json` runtime dependency
3. Ensure your Kotlin version is compatible with the serialization plugin version

```kotlin
// ✅ Correct: Plugin applied and dependency added
plugins {
    alias(libs.plugins.kotlin.serialization)
}

dependencies {
    implementation(libs.kotlinx.serialization.json)
    implementation(libs.androidx.navigation.compose)
}
```

```kotlin
// ❌ Incorrect: Missing plugin - @Serializable will not work
plugins {
    // No serialization plugin
}

dependencies {
    implementation(libs.kotlinx.serialization.json) // Runtime only is not enough
    implementation(libs.androidx.navigation.compose)
}
```

---

## 📐 Defining Routes

Routes are defined as Kotlin classes annotated with `@Serializable`. The type of class you use depends on whether the route accepts arguments.

### Routes Without Arguments

Use `@Serializable object` for destinations that do not require any arguments:

```kotlin
import kotlinx.serialization.Serializable

@Serializable
object Home

@Serializable
object Search

@Serializable
object Favorites

@Serializable
object SettingsHome
```

### Routes With Arguments

Use `@Serializable data class` for destinations that require arguments:

```kotlin
@Serializable
data class UserProfile(val userId: String)

@Serializable
data class RestaurantDetail(val restaurantId: String)

@Serializable
data class OrderConfirmation(
    val orderId: String,
    val totalAmount: Double,
)
```

### Routes With Default Arguments

Default values make arguments optional. When navigating, you can omit arguments that have defaults:

```kotlin
@Serializable
data class Settings(val section: String = "general")

@Serializable
data class SearchResults(
    val query: String,
    val page: Int = 1,
    val sortBy: String = "relevance",
)

@Serializable
data class MapView(
    val latitude: Double = 0.0,
    val longitude: Double = 0.0,
    val zoomLevel: Float = 12.0f,
)
```

### Organizing Routes

For larger applications, organize routes into sealed interfaces or separate files by feature:

```kotlin
// navigation/Routes.kt

package com.arclabs.app.navigation

import kotlinx.serialization.Serializable

// Top-level destinations
@Serializable
object Home

@Serializable
object Search

@Serializable
object Favorites

@Serializable
object Profile

// Feature: Restaurant
@Serializable
data class RestaurantDetail(val restaurantId: String)

@Serializable
data class RestaurantReviews(val restaurantId: String)

@Serializable
data class RestaurantMenu(
    val restaurantId: String,
    val categoryId: String? = null,
)

// Feature: User
@Serializable
data class UserProfile(val userId: String)

@Serializable
data class EditProfile(val userId: String)

// Feature: Settings
@Serializable
object SettingsHome

@Serializable
object SettingsNotifications

@Serializable
object SettingsPrivacy

@Serializable
data class SettingsDetail(val section: String = "general")

// Navigation graph markers
@Serializable
object SettingsGraph

@Serializable
object OnboardingGraph
```

### Supported Argument Types

Navigation Compose with serialization supports the following argument types:

```kotlin
@Serializable
data class ExampleRoute(
    val stringArg: String,           // String
    val intArg: Int,                 // Int
    val longArg: Long,               // Long
    val floatArg: Float,             // Float
    val doubleArg: Double,           // Double
    val boolArg: Boolean,            // Boolean
    val optionalArg: String? = null, // Nullable with default
)
```

> **Note:** Complex types (lists, custom objects) are serialized as JSON strings in the route. For large data payloads, pass an ID and retrieve the data from a shared repository or ViewModel instead.

---

## 🏗️ NavHost Setup

The `NavHost` composable is the container that hosts your navigation graph. It requires a `NavHostController` and a start destination.

### Basic NavHost

```kotlin
@Composable
fun AppNavHost(
    navController: NavHostController = rememberNavController(),
    modifier: Modifier = Modifier,
) {
    NavHost(
        navController = navController,
        startDestination = Home,
        modifier = modifier,
    ) {
        composable<Home> {
            HomeScreen(
                onUserClicked = { userId ->
                    navController.navigate(UserProfile(userId))
                },
                onSearchClicked = {
                    navController.navigate(Search)
                },
            )
        }
        composable<UserProfile> { backStackEntry ->
            val route: UserProfile = backStackEntry.toRoute()
            UserProfileScreen(
                userId = route.userId,
                onBackClicked = {
                    navController.popBackStack()
                },
            )
        }
        composable<Search> {
            SearchScreen(
                onResultClicked = { restaurantId ->
                    navController.navigate(RestaurantDetail(restaurantId))
                },
                onBackClicked = {
                    navController.popBackStack()
                },
            )
        }
        composable<RestaurantDetail> { backStackEntry ->
            val route: RestaurantDetail = backStackEntry.toRoute()
            RestaurantDetailScreen(
                restaurantId = route.restaurantId,
                onBackClicked = {
                    navController.popBackStack()
                },
            )
        }
    }
}
```

### Hosting NavHost in an Activity

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AppTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background,
                ) {
                    AppNavHost()
                }
            }
        }
    }
}
```

### NavHost with Transition Animations

```kotlin
@Composable
fun AppNavHost(
    navController: NavHostController = rememberNavController(),
    modifier: Modifier = Modifier,
) {
    NavHost(
        navController = navController,
        startDestination = Home,
        modifier = modifier,
        enterTransition = {
            slideIntoContainer(
                towards = AnimatedContentTransitionScope.SlideDirection.Left,
                animationSpec = tween(300),
            )
        },
        exitTransition = {
            slideOutOfContainer(
                towards = AnimatedContentTransitionScope.SlideDirection.Left,
                animationSpec = tween(300),
            )
        },
        popEnterTransition = {
            slideIntoContainer(
                towards = AnimatedContentTransitionScope.SlideDirection.Right,
                animationSpec = tween(300),
            )
        },
        popExitTransition = {
            slideOutOfContainer(
                towards = AnimatedContentTransitionScope.SlideDirection.Right,
                animationSpec = tween(300),
            )
        },
    ) {
        composable<Home> {
            HomeScreen(
                onNavigate = { navController.navigate(it) },
            )
        }
        // ... other destinations
    }
}
```

---

## 🔗 Navigation Patterns

### Navigate Forward

Use `navController.navigate(route)` to push a new destination onto the back stack:

```kotlin
// Simple navigation
navController.navigate(UserProfile(userId = "user_123"))

// Navigation with options
navController.navigate(UserProfile(userId = "user_123")) {
    launchSingleTop = true
}
```

### Navigate Back

Use `navController.popBackStack()` to return to the previous destination:

```kotlin
// Pop the current destination
navController.popBackStack()

// Pop back to a specific destination
navController.popBackStack<Home>(inclusive = false)

// Pop back to a specific destination and remove it too
navController.popBackStack<Home>(inclusive = true)
```

### Navigate with Pop (Replace)

Navigate to a new destination while removing destinations from the back stack:

```kotlin
// Navigate to Home and clear the entire back stack (e.g., after login)
navController.navigate(Home) {
    popUpTo(navController.graph.startDestinationId) {
        inclusive = true
    }
}

// Navigate to a destination, popping everything up to Home (but keeping Home)
navController.navigate(RestaurantDetail(restaurantId = "rest_456")) {
    popUpTo<Home> {
        inclusive = false
    }
}
```

### Single Top

Prevent multiple copies of the same destination on the back stack:

```kotlin
// ✅ Prevents duplicate destinations
navController.navigate(Search) {
    launchSingleTop = true
}
```

### Restore State

Save and restore state when navigating between tabs:

```kotlin
navController.navigate(destination) {
    popUpTo(navController.graph.findStartDestination().id) {
        saveState = true
    }
    launchSingleTop = true
    restoreState = true
}
```

### Conditional Navigation

```kotlin
// ✅ Navigate based on auth state
@Composable
fun AppNavHost(
    isLoggedIn: Boolean,
    navController: NavHostController = rememberNavController(),
) {
    NavHost(
        navController = navController,
        startDestination = if (isLoggedIn) Home else Login,
    ) {
        composable<Login> {
            LoginScreen(
                onLoginSuccess = {
                    navController.navigate(Home) {
                        popUpTo<Login> { inclusive = true }
                    }
                },
            )
        }
        composable<Home> {
            HomeScreen()
        }
    }
}
```

---

## 📂 Nested Navigation Graphs

Nested graphs help organize navigation into logical groups and enable feature-level modularity.

### Defining a Nested Graph

```kotlin
// Route markers for nested graphs
@Serializable
object SettingsGraph

@Serializable
object SettingsHome

@Serializable
object SettingsNotifications

@Serializable
object SettingsPrivacy

@Serializable
data class SettingsDetail(val section: String = "general")
```

### Extension Function Pattern

Define nested graphs as extension functions on `NavGraphBuilder` for clean separation:

```kotlin
fun NavGraphBuilder.settingsGraph(navController: NavHostController) {
    navigation<SettingsGraph>(startDestination = SettingsHome) {
        composable<SettingsHome> {
            SettingsHomeScreen(
                onNotificationsClicked = {
                    navController.navigate(SettingsNotifications)
                },
                onPrivacyClicked = {
                    navController.navigate(SettingsPrivacy)
                },
                onDetailClicked = { section ->
                    navController.navigate(SettingsDetail(section))
                },
                onBackClicked = {
                    navController.popBackStack()
                },
            )
        }
        composable<SettingsNotifications> {
            SettingsNotificationsScreen(
                onBackClicked = {
                    navController.popBackStack()
                },
            )
        }
        composable<SettingsPrivacy> {
            SettingsPrivacyScreen(
                onBackClicked = {
                    navController.popBackStack()
                },
            )
        }
        composable<SettingsDetail> { backStackEntry ->
            val route: SettingsDetail = backStackEntry.toRoute()
            SettingsDetailScreen(
                section = route.section,
                onBackClicked = {
                    navController.popBackStack()
                },
            )
        }
    }
}
```

### Onboarding Graph Example

```kotlin
@Serializable
object OnboardingGraph

@Serializable
object OnboardingWelcome

@Serializable
object OnboardingPermissions

@Serializable
object OnboardingComplete

fun NavGraphBuilder.onboardingGraph(navController: NavHostController) {
    navigation<OnboardingGraph>(startDestination = OnboardingWelcome) {
        composable<OnboardingWelcome> {
            WelcomeScreen(
                onContinue = {
                    navController.navigate(OnboardingPermissions)
                },
            )
        }
        composable<OnboardingPermissions> {
            PermissionsScreen(
                onContinue = {
                    navController.navigate(OnboardingComplete)
                },
                onBackClicked = {
                    navController.popBackStack()
                },
            )
        }
        composable<OnboardingComplete> {
            CompleteScreen(
                onFinish = {
                    navController.navigate(Home) {
                        popUpTo<OnboardingGraph> { inclusive = true }
                    }
                },
            )
        }
    }
}
```

### Using Nested Graphs in the Main NavHost

```kotlin
@Composable
fun AppNavHost(
    navController: NavHostController = rememberNavController(),
    modifier: Modifier = Modifier,
) {
    NavHost(
        navController = navController,
        startDestination = Home,
        modifier = modifier,
    ) {
        composable<Home> {
            HomeScreen(
                onSettingsClicked = {
                    navController.navigate(SettingsGraph)
                },
            )
        }

        // Include nested graphs
        settingsGraph(navController)
        onboardingGraph(navController)
    }
}
```

---

## 🔗 Deep Links

Deep links allow external apps and URLs to navigate directly to specific screens in your application.

### Manifest Setup

Add intent filters to your `AndroidManifest.xml`:

```xml
<activity
    android:name=".MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>

    <!-- Deep link intent filter -->
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
            android:scheme="https"
            android:host="app.arclabs.com" />
    </intent-filter>

    <!-- Custom scheme -->
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="arclabs" />
    </intent-filter>
</activity>
```

### Defining Deep Links in Composables

Use `navDeepLink` inside your `composable` destinations:

```kotlin
composable<RestaurantDetail>(
    deepLinks = listOf(
        navDeepLink<RestaurantDetail>(
            basePath = "https://app.arclabs.com/restaurant",
        ),
        navDeepLink<RestaurantDetail>(
            basePath = "arclabs://restaurant",
        ),
    ),
) { backStackEntry ->
    val route: RestaurantDetail = backStackEntry.toRoute()
    RestaurantDetailScreen(
        restaurantId = route.restaurantId,
        onBackClicked = {
            navController.popBackStack()
        },
    )
}

composable<UserProfile>(
    deepLinks = listOf(
        navDeepLink<UserProfile>(
            basePath = "https://app.arclabs.com/user",
        ),
    ),
) { backStackEntry ->
    val route: UserProfile = backStackEntry.toRoute()
    UserProfileScreen(
        userId = route.userId,
        onBackClicked = {
            navController.popBackStack()
        },
    )
}
```

### Testing Deep Links

Test deep links using `adb` from the command line:

```bash
# Test HTTPS deep link
adb shell am start -a android.intent.action.VIEW \
    -d "https://app.arclabs.com/restaurant/rest_123" \
    com.arclabs.app

# Test custom scheme deep link
adb shell am start -a android.intent.action.VIEW \
    -d "arclabs://restaurant/rest_123" \
    com.arclabs.app

# Test user profile deep link
adb shell am start -a android.intent.action.VIEW \
    -d "https://app.arclabs.com/user/user_456" \
    com.arclabs.app
```

### Handling Deep Links in the Activity

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val navController = rememberNavController()
            AppTheme {
                AppNavHost(navController = navController)
            }
        }
    }

    override fun onNewIntent(intent: Intent) {
        super.onNewIntent(intent)
        // Handle deep links when the activity is already running
        // NavController handles this automatically if you pass the intent
    }
}
```

---

## 📱 Bottom Navigation

Integrating Navigation Compose with Material 3 `NavigationBar` for tab-based navigation.

### Defining Tab Destinations

```kotlin
enum class TopLevelDestination(
    val route: Any,
    val selectedIcon: ImageVector,
    val unselectedIcon: ImageVector,
    val label: String,
) {
    HOME(
        route = Home,
        selectedIcon = Icons.Filled.Home,
        unselectedIcon = Icons.Outlined.Home,
        label = "Home",
    ),
    SEARCH(
        route = Search,
        selectedIcon = Icons.Filled.Search,
        unselectedIcon = Icons.Outlined.Search,
        label = "Search",
    ),
    FAVORITES(
        route = Favorites,
        selectedIcon = Icons.Filled.Favorite,
        unselectedIcon = Icons.Outlined.FavoriteBorder,
        label = "Favorites",
    ),
    PROFILE(
        route = Profile,
        selectedIcon = Icons.Filled.Person,
        unselectedIcon = Icons.Outlined.Person,
        label = "Profile",
    ),
}
```

### Main Scaffold with Bottom Navigation

```kotlin
@Composable
fun MainScreen() {
    val navController = rememberNavController()
    val navBackStackEntry by navController.currentBackStackEntryAsState()
    val currentDestination = navBackStackEntry?.destination

    // Determine if bottom bar should be visible
    val showBottomBar = TopLevelDestination.entries.any { destination ->
        currentDestination?.hasRoute(destination.route::class) == true
    }

    Scaffold(
        bottomBar = {
            if (showBottomBar) {
                AppBottomBar(
                    destinations = TopLevelDestination.entries,
                    currentDestination = currentDestination,
                    onNavigateToDestination = { destination ->
                        navController.navigate(destination.route) {
                            // Pop up to the start destination to avoid
                            // building up a large stack of destinations
                            popUpTo(navController.graph.findStartDestination().id) {
                                saveState = true
                            }
                            // Avoid multiple copies of the same destination
                            launchSingleTop = true
                            // Restore state when re-selecting a previously selected tab
                            restoreState = true
                        }
                    },
                )
            }
        },
    ) { innerPadding ->
        AppNavHost(
            navController = navController,
            modifier = Modifier.padding(innerPadding),
        )
    }
}
```

### Bottom Bar Composable

```kotlin
@Composable
fun AppBottomBar(
    destinations: List<TopLevelDestination>,
    currentDestination: NavDestination?,
    onNavigateToDestination: (TopLevelDestination) -> Unit,
    modifier: Modifier = Modifier,
) {
    NavigationBar(modifier = modifier) {
        destinations.forEach { destination ->
            val selected = currentDestination?.hasRoute(
                destination.route::class,
            ) == true

            NavigationBarItem(
                selected = selected,
                onClick = { onNavigateToDestination(destination) },
                icon = {
                    Icon(
                        imageVector = if (selected) {
                            destination.selectedIcon
                        } else {
                            destination.unselectedIcon
                        },
                        contentDescription = destination.label,
                    )
                },
                label = {
                    Text(text = destination.label)
                },
            )
        }
    }
}
```

### Complete NavHost with Bottom Navigation Destinations

```kotlin
@Composable
fun AppNavHost(
    navController: NavHostController,
    modifier: Modifier = Modifier,
) {
    NavHost(
        navController = navController,
        startDestination = Home,
        modifier = modifier,
    ) {
        // Tab destinations
        composable<Home> {
            HomeScreen(
                onRestaurantClicked = { restaurantId ->
                    navController.navigate(RestaurantDetail(restaurantId))
                },
            )
        }
        composable<Search> {
            SearchScreen(
                onResultClicked = { restaurantId ->
                    navController.navigate(RestaurantDetail(restaurantId))
                },
            )
        }
        composable<Favorites> {
            FavoritesScreen(
                onItemClicked = { restaurantId ->
                    navController.navigate(RestaurantDetail(restaurantId))
                },
            )
        }
        composable<Profile> {
            ProfileScreen(
                onSettingsClicked = {
                    navController.navigate(SettingsGraph)
                },
                onEditProfileClicked = { userId ->
                    navController.navigate(EditProfile(userId))
                },
            )
        }

        // Detail destinations (no bottom bar)
        composable<RestaurantDetail> { backStackEntry ->
            val route: RestaurantDetail = backStackEntry.toRoute()
            RestaurantDetailScreen(
                restaurantId = route.restaurantId,
                onBackClicked = { navController.popBackStack() },
            )
        }
        composable<EditProfile> { backStackEntry ->
            val route: EditProfile = backStackEntry.toRoute()
            EditProfileScreen(
                userId = route.userId,
                onBackClicked = { navController.popBackStack() },
            )
        }

        // Nested graphs
        settingsGraph(navController)
    }
}
```

---

## 🧭 ViewModel Navigation Events

ViewModels should never hold a reference to `NavController`. Instead, they emit navigation events that the Composable layer observes and acts upon.

### Defining Navigation Events

```kotlin
sealed interface NavigationEvent {
    data class GoToProfile(val userId: String) : NavigationEvent
    data class GoToRestaurant(val restaurantId: String) : NavigationEvent
    data class GoToSettings(val section: String = "general") : NavigationEvent
    data object GoBack : NavigationEvent
    data object GoToLogin : NavigationEvent
}
```

### ViewModel with Navigation Events

```kotlin
class HomeViewModel(
    private val getRestaurantsUseCase: GetRestaurantsUseCase,
) : ViewModel() {

    private val _navigationEvent = MutableSharedFlow<NavigationEvent>()
    val navigationEvent: SharedFlow<NavigationEvent> = _navigationEvent.asSharedFlow()

    private val _uiState = MutableStateFlow(HomeUiState())
    val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()

    fun onRestaurantClicked(restaurantId: String) {
        viewModelScope.launch {
            _navigationEvent.emit(
                NavigationEvent.GoToRestaurant(restaurantId),
            )
        }
    }

    fun onProfileClicked(userId: String) {
        viewModelScope.launch {
            _navigationEvent.emit(
                NavigationEvent.GoToProfile(userId),
            )
        }
    }

    fun onSettingsClicked() {
        viewModelScope.launch {
            _navigationEvent.emit(
                NavigationEvent.GoToSettings(),
            )
        }
    }

    fun onLogout() {
        viewModelScope.launch {
            // Perform logout logic
            _navigationEvent.emit(NavigationEvent.GoToLogin)
        }
    }
}
```

### Collecting Navigation Events in Composables

```kotlin
@Composable
fun HomeRoute(
    navController: NavHostController,
    viewModel: HomeViewModel = hiltViewModel(),
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val lifecycle = LocalLifecycleOwner.current.lifecycle

    // ✅ Lifecycle-aware collection of navigation events
    LaunchedEffect(lifecycle) {
        lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
            viewModel.navigationEvent.collect { event ->
                when (event) {
                    is NavigationEvent.GoToProfile -> {
                        navController.navigate(UserProfile(event.userId))
                    }
                    is NavigationEvent.GoToRestaurant -> {
                        navController.navigate(RestaurantDetail(event.restaurantId))
                    }
                    is NavigationEvent.GoToSettings -> {
                        navController.navigate(SettingsDetail(event.section))
                    }
                    NavigationEvent.GoBack -> {
                        navController.popBackStack()
                    }
                    NavigationEvent.GoToLogin -> {
                        navController.navigate(Login) {
                            popUpTo(navController.graph.startDestinationId) {
                                inclusive = true
                            }
                        }
                    }
                }
            }
        }
    }

    HomeScreen(
        uiState = uiState,
        onRestaurantClicked = viewModel::onRestaurantClicked,
        onProfileClicked = viewModel::onProfileClicked,
        onSettingsClicked = viewModel::onSettingsClicked,
    )
}
```

### Alternative: Callback Pattern (Simpler Cases)

For simpler screens, you can skip the SharedFlow pattern and use direct callbacks:

```kotlin
// ✅ Simple callback pattern - good for screens with few navigation actions
@Composable
fun HomeRoute(
    navController: NavHostController,
    viewModel: HomeViewModel = hiltViewModel(),
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    HomeScreen(
        uiState = uiState,
        onRestaurantClicked = { restaurantId ->
            navController.navigate(RestaurantDetail(restaurantId))
        },
        onProfileClicked = { userId ->
            navController.navigate(UserProfile(userId))
        },
    )
}
```

Choose the SharedFlow pattern when:
- Navigation depends on async operations (network calls, database queries)
- The ViewModel needs to perform logic before navigation
- You have complex navigation flows (e.g., logout clears state then navigates)

Choose the callback pattern when:
- Navigation is a direct response to a UI action
- No business logic runs before navigating

---

## 🧪 Testing Navigation

### Setting Up Test Dependencies

```kotlin
// build.gradle.kts
dependencies {
    androidTestImplementation(libs.androidx.navigation.testing)
    androidTestImplementation(libs.androidx.compose.ui.test.junit4)
}
```

### Testing with TestNavHostController

```kotlin
class NavigationTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    private lateinit var navController: TestNavHostController

    @Before
    fun setup() {
        composeTestRule.setContent {
            navController = TestNavHostController(LocalContext.current).apply {
                navigatorProvider.addNavigator(ComposeNavigator())
            }
            AppNavHost(navController = navController)
        }
    }

    @Test
    fun appNavHost_startsAtHomeScreen() {
        // Then
        composeTestRule
            .onNodeWithText("Home")
            .assertIsDisplayed()
    }

    @Test
    fun appNavHost_navigatesToUserProfile() {
        // When
        composeTestRule
            .onNodeWithText("View Profile")
            .performClick()

        // Then
        val currentRoute = navController.currentBackStackEntry?.toRoute<UserProfile>()
        assertNotNull(currentRoute)
    }

    @Test
    fun appNavHost_navigatesBackFromDetail() {
        // Given - Navigate to a detail screen
        composeTestRule.runOnUiThread {
            navController.navigate(RestaurantDetail(restaurantId = "rest_123"))
        }

        // When - Press back
        composeTestRule.runOnUiThread {
            navController.popBackStack()
        }

        // Then - Back at home
        composeTestRule
            .onNodeWithText("Home")
            .assertIsDisplayed()
    }
}
```

### Testing ViewModel Navigation Events

```kotlin
class HomeViewModelNavigationTest {

    private lateinit var viewModel: HomeViewModel
    private val fakeGetRestaurantsUseCase = FakeGetRestaurantsUseCase()

    @Before
    fun setup() {
        viewModel = HomeViewModel(
            getRestaurantsUseCase = fakeGetRestaurantsUseCase,
        )
    }

    @Test
    fun onRestaurantClicked_emitsGoToRestaurantEvent() = runTest {
        // Given
        val restaurantId = "rest_123"
        val events = mutableListOf<NavigationEvent>()
        val job = launch {
            viewModel.navigationEvent.toList(events)
        }

        // When
        viewModel.onRestaurantClicked(restaurantId)
        advanceUntilIdle()

        // Then
        assertEquals(1, events.size)
        assertEquals(
            NavigationEvent.GoToRestaurant(restaurantId),
            events.first(),
        )

        job.cancel()
    }

    @Test
    fun onLogout_emitsGoToLoginEvent() = runTest {
        // Given
        val events = mutableListOf<NavigationEvent>()
        val job = launch {
            viewModel.navigationEvent.toList(events)
        }

        // When
        viewModel.onLogout()
        advanceUntilIdle()

        // Then
        assertEquals(1, events.size)
        assertEquals(NavigationEvent.GoToLogin, events.first())

        job.cancel()
    }
}
```

### Testing Deep Links

```kotlin
@Test
fun deepLink_navigatesToRestaurantDetail() {
    // Given
    val deepLinkUri = Uri.parse("https://app.arclabs.com/restaurant/rest_123")

    // When
    composeTestRule.runOnUiThread {
        navController.navigate(deepLinkUri)
    }

    // Then
    val currentRoute = navController.currentBackStackEntry
        ?.toRoute<RestaurantDetail>()
    assertEquals("rest_123", currentRoute?.restaurantId)
}
```

---

## ✅ Navigation Checklist

Use this checklist when implementing navigation in your Android Compose application:

- [ ] **Dependencies** - `navigation-compose` and `kotlinx-serialization` are added to `build.gradle.kts`
- [ ] **Serialization plugin** - `kotlin-serialization` plugin is applied in the module
- [ ] **Route definitions** - All routes use `@Serializable` objects or data classes
- [ ] **NavHost** - A single `NavHost` is set up with `rememberNavController()`
- [ ] **Start destination** - The start destination is a type-safe route, not a string
- [ ] **Argument extraction** - `backStackEntry.toRoute<T>()` is used to extract route arguments
- [ ] **Back navigation** - Every detail screen provides a way to go back
- [ ] **Single top** - Tab navigation uses `launchSingleTop = true`
- [ ] **State restoration** - Tab navigation uses `saveState` and `restoreState`
- [ ] **Bottom bar visibility** - Bottom bar is hidden on detail screens
- [ ] **ViewModel separation** - ViewModels never reference `NavController`
- [ ] **Lifecycle awareness** - Navigation event collection uses `repeatOnLifecycle`
- [ ] **Deep links** - Deep links are configured in both manifest and composable destinations
- [ ] **Nested graphs** - Feature groups are organized as nested navigation graphs
- [ ] **Transition animations** - Enter/exit animations are configured for a polished feel
- [ ] **Tests** - Navigation tests verify routing, argument passing, and deep links

---

## 🚫 Common Mistakes

### 1. Passing NavController to ViewModel

```kotlin
// ❌ WRONG: ViewModel holds NavController reference
class HomeViewModel(
    private val navController: NavHostController,
) : ViewModel() {
    fun onItemClicked(id: String) {
        navController.navigate(RestaurantDetail(id)) // Direct navigation in ViewModel
    }
}
```

This violates separation of concerns. The ViewModel belongs to the domain/presentation boundary and should not depend on Android framework navigation classes. It also makes the ViewModel untestable without mocking the NavController.

```kotlin
// ✅ CORRECT: ViewModel emits events, Composable handles navigation
class HomeViewModel : ViewModel() {
    private val _navigationEvent = MutableSharedFlow<NavigationEvent>()
    val navigationEvent: SharedFlow<NavigationEvent> = _navigationEvent.asSharedFlow()

    fun onItemClicked(id: String) {
        viewModelScope.launch {
            _navigationEvent.emit(NavigationEvent.GoToRestaurant(id))
        }
    }
}

// In Composable
LaunchedEffect(lifecycle) {
    lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.navigationEvent.collect { event ->
            when (event) {
                is NavigationEvent.GoToRestaurant -> {
                    navController.navigate(RestaurantDetail(event.restaurantId))
                }
                // ...
            }
        }
    }
}
```

### 2. Using String Routes Instead of Type-Safe

```kotlin
// ❌ WRONG: String-based routes are error-prone
composable("restaurant/{restaurantId}") { backStackEntry ->
    val id = backStackEntry.arguments?.getString("restaurantId") ?: ""
    RestaurantDetailScreen(restaurantId = id)
}

// Navigating with string interpolation
navController.navigate("restaurant/$restaurantId")
```

String routes have no compile-time safety. A typo in the route string or argument name causes a runtime crash instead of a compile error.

```kotlin
// ✅ CORRECT: Type-safe routes with serialization
@Serializable
data class RestaurantDetail(val restaurantId: String)

composable<RestaurantDetail> { backStackEntry ->
    val route: RestaurantDetail = backStackEntry.toRoute()
    RestaurantDetailScreen(restaurantId = route.restaurantId)
}

// Navigating is type-checked at compile time
navController.navigate(RestaurantDetail(restaurantId = restaurantId))
```

### 3. Not Handling Back Stack Properly

```kotlin
// ❌ WRONG: Not clearing back stack after login
navController.navigate(Home)
// User can press back and return to the login screen!

// ❌ WRONG: Building up duplicate destinations on tab switches
navController.navigate(Search)
navController.navigate(Search) // Creates a second Search on the stack
```

Improper back stack management leads to confusing user experiences where pressing back leads to unexpected screens or duplicate destinations stack up.

```kotlin
// ✅ CORRECT: Clear back stack after authentication
navController.navigate(Home) {
    popUpTo(navController.graph.startDestinationId) {
        inclusive = true
    }
}

// ✅ CORRECT: Tab navigation with proper back stack handling
navController.navigate(Search) {
    popUpTo(navController.graph.findStartDestination().id) {
        saveState = true
    }
    launchSingleTop = true
    restoreState = true
}
```

### 4. Collecting Navigation Events Without Lifecycle Awareness

```kotlin
// ❌ WRONG: Collecting without lifecycle awareness
LaunchedEffect(Unit) {
    viewModel.navigationEvent.collect { event ->
        when (event) {
            is NavigationEvent.GoToProfile -> {
                navController.navigate(UserProfile(event.userId))
            }
            NavigationEvent.GoBack -> navController.popBackStack()
        }
    }
}
```

Collecting a `SharedFlow` without lifecycle awareness means navigation events can be processed when the app is in the background, causing crashes or unexpected behavior.

```kotlin
// ✅ CORRECT: Lifecycle-aware collection
val lifecycle = LocalLifecycleOwner.current.lifecycle

LaunchedEffect(lifecycle) {
    lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.navigationEvent.collect { event ->
            when (event) {
                is NavigationEvent.GoToProfile -> {
                    navController.navigate(UserProfile(event.userId))
                }
                NavigationEvent.GoBack -> navController.popBackStack()
            }
        }
    }
}
```

The `repeatOnLifecycle(Lifecycle.State.STARTED)` call ensures that collection only happens when the Composable is at least in the STARTED state. Events emitted while the app is in the background are buffered and delivered when the lifecycle resumes.

---

## 📚 Further Reading

- **Official Documentation:** [Navigation Compose - Android Developers](https://developer.android.com/develop/ui/compose/navigation)
- **Type Safety in Navigation:** [Type safe navigation in Compose](https://developer.android.com/guide/navigation/design/type-safety)
- **Kotlinx Serialization:** [Kotlin Serialization Guide](https://kotlinlang.org/docs/serialization.html)
- **Navigation Testing:** [Test navigation - Android Developers](https://developer.android.com/guide/navigation/testing)
- **Now in Android Sample:** [GitHub - android/nowinandroid](https://github.com/android/nowinandroid) (Reference implementation of Navigation Compose with type safety)
- **Deep Links:** [Create deep links to app content](https://developer.android.com/training/app-links/deep-linking)
- **Material 3 NavigationBar:** [Navigation bar - Material Design 3](https://m3.material.io/components/navigation-bar)
