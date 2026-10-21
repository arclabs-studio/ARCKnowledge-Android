# 🧭 Navigation 3

**Navigation for ARC Labs Android apps is built on Jetpack Navigation 3 (`androidx.navigation3`, 1.2.0). The back stack is a plain, observable list of `@Serializable` `NavKey`s that you own; `NavDisplay` renders it, an `entryProvider` resolves each key to content, and scene strategies decide how entries are laid out (single pane, dialog, bottom sheet, list-detail). This guide covers setup, keys, the back stack, top-level tabs with `NavigationSuiteScaffold`, modals, ViewModels scoped per entry, adaptive layouts, predictive back, deep links, and testing.**

> **Navigation 2 (`navigation-compose`, `NavHost`, `NavController`) is not used in new ARC Labs code.** This document replaces the former `navigation-compose.md`.
>
> API names in this document were verified against the official `android/skills` plugin (`navigation-3` skill, last updated 2026-10-17) and the Nav3 recipes it ships. Nav3 is young: before writing code, re-check signatures with the `navigation-3` skill or Context7 (`/websites/developer_android_develop_ui_compose`). Never invent an API.

---

## 🎯 Why Navigation 3

Navigation 3 is a navigation library designed exclusively for Compose. Compared to Navigation 2:

- **You own the back stack** — it is a `SnapshotStateList`-like list of keys. Navigating forward is `add(key)`, going back is `removeLastOrNull()`. No `NavController`, no graph DSL, no `popUpTo` gymnastics.
- **Simpler Compose integration** — `NavDisplay` observes the back stack and recomposes; nothing to "bind".
- **Adaptive by design** — a scene can read more than one entry from the back stack at the same time (list-detail, supporting pane), so layouts adapt to window size without separate code paths.
- **Entry-scoped state** — each `NavEntry` gets its own saveable state, `LifecycleOwner` and (with the add-on) `ViewModelStore`.
- **Predictive back built in** — `NavDisplay` handles system back and runs predictive-back animations.

### Comparison: iOS NavigationStack vs Android Navigation 3

| Concept | iOS (SwiftUI) | Android (Navigation 3) |
|---|---|---|
| Navigation container | `NavigationStack` | `NavDisplay` |
| Navigation state | `NavigationPath` / `[Route]` | `NavBackStack<NavKey>` (from `rememberNavBackStack`) |
| Route definition | `Hashable` enum/struct | `@Serializable` object/data class implementing `NavKey` |
| Destination mapping | `.navigationDestination(for:)` | `entryProvider { entry<Key> { … } }` |
| Navigate forward | `path.append(route)` | `backStack.add(key)` |
| Navigate back | `path.removeLast()` | `backStack.removeLastOrNull()` |
| Tabs | `TabView` + `Tab` | `NavigationSuiteScaffold` + one back stack per top-level key |
| Sheets / alerts | `.sheet` / `.alert` | Keys with `BottomSheetSceneStrategy` / `DialogSceneStrategy` metadata |
| Coordinator | Not used (ARC iOS forbids a custom Router) | Not used — no custom Coordinator on Android either |

The mental model is now almost identical on both platforms: a typed, value-based path that the UI renders.

---

## 🚀 Setup

### Requirements

- `compileSdk` **36** or later (ARC Labs uses 36; pending: compileSdk 37 required by Compose 1.12 stack — Nav3 1.2.0 declares `minCompileSdk` 37).
- Kotlin Serialization plugin applied to every module that declares keys.
- Compose-only destinations (Navigation 3 has no Fragment/View destinations).

### Version Catalog (`libs.versions.toml`)

```toml
[versions]
nav3 = "1.2.0"
lifecycle = "2.11.0"            # lifecycle-viewmodel-navigation3 version from the official Nav3 migration guide
material3-adaptive = "1.3.0"
kotlinx-serialization = "1.12.0" # verify exact patch before use

[libraries]
androidx-navigation3-runtime = { module = "androidx.navigation3:navigation3-runtime", version.ref = "nav3" }
androidx-navigation3-ui = { module = "androidx.navigation3:navigation3-ui", version.ref = "nav3" }
androidx-lifecycle-viewmodel-navigation3 = { module = "androidx.lifecycle:lifecycle-viewmodel-navigation3", version.ref = "lifecycle" }
# Material adaptive scenes (list-detail, supporting pane) for Navigation 3 — verify version alignment with material3-adaptive before use
androidx-material3-adaptive-navigation3 = { module = "androidx.compose.material3.adaptive:adaptive-navigation3", version.ref = "material3-adaptive" }
# NavigationSuiteScaffold (bottom bar <-> rail); version managed by the Compose BOM / material3 pin
androidx-material3-adaptive-navigation-suite = { module = "androidx.compose.material3:material3-adaptive-navigation-suite" }
kotlinx-serialization-json = { module = "org.jetbrains.kotlinx:kotlinx-serialization-json", version.ref = "kotlinx-serialization" }
```

### Module `build.gradle.kts`

In ARC Labs projects these lines live in the `arc.android.feature` / `arc.android.compose` convention plugins in `build-logic/` (see [Gradle](../Tools/gradle.md)); feature modules never repeat them.

```kotlin
plugins {
    alias(libs.plugins.kotlin.serialization)
}

dependencies {
    implementation(libs.androidx.navigation3.runtime)
    implementation(libs.androidx.navigation3.ui)
    implementation(libs.androidx.lifecycle.viewmodel.navigation3)
    implementation(libs.kotlinx.serialization.json)
}
```

```kotlin
// ❌ Navigation 2 artifacts are not allowed in new code
implementation("androidx.navigation:navigation-compose:…")
implementation("androidx.hilt:hilt-navigation-compose:…")
```

---

## 🔑 Keys

Every destination is a `@Serializable` key implementing `NavKey`. Serializable keys are what make `rememberNavBackStack` survive configuration changes and process death.

```kotlin
import androidx.navigation3.runtime.NavKey
import kotlinx.serialization.Serializable

// ✅ Destinations without arguments: data object
@Serializable
data object HomeKey : NavKey

// ✅ Destinations with arguments: data class, arguments are properties
@Serializable
data class RestaurantDetailKey(val restaurantId: String) : NavKey

// ✅ Modals are keys too (rendered by a scene strategy, see "Modals")
@Serializable
data class DeleteRestaurantDialogKey(val restaurantId: String) : NavKey
```

### Key Rules

- ✅ Keys carry **identifiers**, never whole domain objects (`restaurantId`, not `Restaurant`).
- ✅ Keys are immutable `data object` / `data class` values.
- ✅ Each feature module owns its keys in a small `api`/`navigation` package; `:app` composes them.
- ❌ Never use strings as routes.
- ❌ Never put non-serializable types (lambdas, `Context`, `Bitmap`) in a key.

---

## 🏗️ Back Stack + NavDisplay + entryProvider

The minimal setup: a persistent back stack, a `NavDisplay`, and an `entryProvider` built with the DSL.

```kotlin
@Composable
fun SimpleNavigation() {
    val backStack = rememberNavBackStack(HomeKey)

    NavDisplay(
        backStack = backStack,
        onBack = { backStack.removeLastOrNull() },
        entryProvider = entryProvider {
            entry<HomeKey> {
                HomeRoute(
                    onRestaurantClick = dropUnlessResumed { id: String ->
                        backStack.add(RestaurantDetailKey(id))
                    },
                )
            }
            entry<RestaurantDetailKey> { key ->
                RestaurantDetailRoute(restaurantId = key.restaurantId)
            }
        },
    )
}
```

> `dropUnlessResumed` (from `androidx.lifecycle.compose`) prevents double-taps from pushing the same key twice while a transition is running. The official recipes wrap navigation clicks with it; ARC Labs does the same.

### Encapsulating Feature Entries

Feature modules expose an extension on `EntryProviderScope<NavKey>` instead of a `NavGraphBuilder` extension:

```kotlin
// :feature:library — navigation/LibraryEntries.kt
fun EntryProviderScope<NavKey>.libraryEntries(
    onOpenRestaurant: (String) -> Unit,
    onBack: () -> Unit,
) {
    entry<HomeKey> { HomeRoute(onRestaurantClick = onOpenRestaurant) }
    entry<RestaurantDetailKey> { key ->
        RestaurantDetailRoute(restaurantId = key.restaurantId, onBack = onBack)
    }
}

// :app — composes every feature
val entryProvider = entryProvider<NavKey> {
    libraryEntries(onOpenRestaurant = navigator::openRestaurant, onBack = navigator::goBack)
    searchEntries(/* … */)
}
```

Feature modules never depend on each other; cross-feature navigation is wired in `:app` through these lambdas. For Hilt-injected entry builders (multibinding `EntryProviderScope` installers), follow the official `modular-hilt` recipe in the `navigation-3` skill.

---

## 📱 Top-Level Tabs with NavigationSuiteScaffold

ARC Labs apps with tabs keep **one back stack per top-level key** and render the tab selector with `NavigationSuiteScaffold`, which shows a bottom navigation bar on compact windows and a navigation rail on medium/expanded windows automatically.

### Navigation State and Navigator

This is the state holder pattern from the official Navigation 2 → 3 migration guide. Copy it into `:app` (or `:core:ui`) and do not change `rememberSerializable` to `rememberSaveable`.

```kotlin
@Composable
fun rememberNavigationState(
    startRoute: NavKey,
    topLevelRoutes: Set<NavKey>,
): NavigationState {
    val topLevelRoute = rememberSerializable(
        startRoute, topLevelRoutes,
        serializer = MutableStateSerializer(NavKeySerializer()),
    ) {
        mutableStateOf(startRoute)
    }
    val backStacks = topLevelRoutes.associateWith { key -> rememberNavBackStack(key) }

    return remember(startRoute, topLevelRoutes) {
        NavigationState(startRoute = startRoute, topLevelRoute = topLevelRoute, backStacks = backStacks)
    }
}

class NavigationState(
    val startRoute: NavKey,
    topLevelRoute: MutableState<NavKey>,
    val backStacks: Map<NavKey, NavBackStack<NavKey>>,
) {
    var topLevelRoute: NavKey by topLevelRoute
    val stacksInUse: List<NavKey>
        get() = if (topLevelRoute == startRoute) listOf(startRoute) else listOf(startRoute, topLevelRoute)
}

class Navigator(val state: NavigationState) {
    fun navigate(route: NavKey) {
        if (route in state.backStacks.keys) {
            state.topLevelRoute = route
        } else {
            state.backStacks[state.topLevelRoute]?.add(route)
        }
    }

    fun goBack() {
        val currentStack = state.backStacks[state.topLevelRoute] ?: return
        if (currentStack.lastOrNull() == state.topLevelRoute) {
            state.topLevelRoute = state.startRoute
        } else {
            currentStack.removeLastOrNull()
        }
    }
}
```

> ARC Labs deviation from the official snippet: `goBack()` returns early instead of calling `error(...)` when a stack is missing, so the Navigator can never crash the app.

`NavigationState.toEntries(entryProvider)` (also from the migration guide) decorates each stack and flattens the stacks in use into the list `NavDisplay` renders. ARC Labs adds the ViewModel decorator so every entry gets its own `ViewModelStore`:

```kotlin
@Composable
fun NavigationState.toEntries(
    entryProvider: (NavKey) -> NavEntry<NavKey>,
): SnapshotStateList<NavEntry<NavKey>> {
    val decoratedEntries = backStacks.mapValues { (_, stack) ->
        rememberDecoratedNavEntries(
            backStack = stack,
            entryDecorators = listOf(
                rememberSaveableStateHolderNavEntryDecorator<NavKey>(),
                rememberViewModelStoreNavEntryDecorator(), // verify generic signature when implementing
            ),
            entryProvider = entryProvider,
        )
    }
    return stacksInUse.flatMap { decoratedEntries[it].orEmpty() }.toMutableStateList()
}
```

### The App Shell

```kotlin
enum class TopLevelDestination(
    val key: NavKey,
    val icon: ImageVector,
    @StringRes val labelRes: Int,
) {
    HOME(HomeKey, Icons.Filled.Home, R.string.nav_home),
    FAVORITES(FavoritesKey, Icons.Filled.Favorite, R.string.nav_favorites),
    PENDING(PendingKey, Icons.Filled.Bookmark, R.string.nav_pending),
    MAP(MapKey, Icons.Filled.Map, R.string.nav_map),
}

@Composable
fun AppShell() {
    val navigationState = rememberNavigationState(
        startRoute = HomeKey,
        topLevelRoutes = TopLevelDestination.entries.map { it.key }.toSet(),
    )
    val navigator = remember(navigationState) { Navigator(navigationState) }

    NavigationSuiteScaffold(
        navigationSuiteItems = {
            TopLevelDestination.entries.forEach { destination ->
                item(
                    selected = destination.key == navigationState.topLevelRoute,
                    onClick = { navigator.navigate(destination.key) },
                    icon = { Icon(destination.icon, contentDescription = null) },
                    label = { Text(stringResource(destination.labelRes)) },
                )
            }
        },
    ) {
        AppNavDisplay(navigationState = navigationState, navigator = navigator)
    }
}
```

> **Verify before use:** the `navigationSuiteItems { item(...) }` form is the long-standing `NavigationSuiteScaffold` API. Newer Material 3 releases also offer an overload taking `NavigationSuiteItem` composables (the official `adaptive` skill refers to a `navigationItems` parameter). Confirm which overload is current for material3 **1.5.0-beta01** with the `adaptive` skill or Context7 before writing the shell.

### Navigation Suite Rules

- ✅ Selected tab = `destination.key == navigationState.topLevelRoute` (no hierarchy traversal as in Nav2).
- ✅ Hide the navigation suite on immersive screens with `rememberNavigationSuiteScaffoldState()` + `show()`/`hide()` from a `LaunchedEffect` (official `adaptive` skill, Step 2.1).
- ✅ `NavigationSuiteScaffold` does **not** propagate `PaddingValues`: apply window insets on each screen (list `contentPadding`, FAB padding). Never put `safeDrawingPadding()` on the scaffold itself (official `edge-to-edge` skill).
- ❌ Never hand-build a `Scaffold` + `NavigationBar` + `NavigationRail` switch on window size — that is what `NavigationSuiteScaffold` is for.

---

## 🪟 Modals: Dialogs and Bottom Sheets as Keys

Modals are destinations like any other: they are keys on the back stack, so they survive rotation, process death and predictive back. A **scene strategy** decides that an entry renders as an overlay.

```kotlin
@Composable
fun AppNavDisplay(navigationState: NavigationState, navigator: Navigator) {
    val bottomSheetStrategy = remember { BottomSheetSceneStrategy<NavKey>() }
    val dialogStrategy = remember { DialogSceneStrategy<NavKey>() }

    val entryProvider = entryProvider<NavKey> {
        entry<HomeKey> { HomeRoute(onRestaurantClick = { id -> navigator.navigate(RestaurantDetailKey(id)) }) }
        entry<RestaurantDetailKey> { key ->
            RestaurantDetailRoute(
                restaurantId = key.restaurantId,
                onDeleteClick = { navigator.navigate(DeleteRestaurantDialogKey(key.restaurantId)) },
                onFilterClick = { navigator.navigate(VisitFilterSheetKey) },
            )
        }
        // Dialog: built-in strategy, metadata on the entry
        entry<DeleteRestaurantDialogKey>(metadata = DialogSceneStrategy.dialog()) { key ->
            DeleteRestaurantDialog(restaurantId = key.restaurantId, onDismiss = navigator::goBack)
        }
        // Bottom sheet: strategy copied from the official recipe into :core:ui
        entry<VisitFilterSheetKey>(metadata = BottomSheetSceneStrategy.bottomSheet()) {
            VisitFilterSheet(onDone = navigator::goBack)
        }
    }

    NavDisplay(
        entries = navigationState.toEntries(entryProvider),
        onBack = { navigator.goBack() },
        // Overlay strategies first (official guidance for the bottom sheet strategy)
        sceneStrategies = listOf(bottomSheetStrategy, dialogStrategy),
    )
}
```

- `DialogSceneStrategy` ships in Navigation 3 (`androidx.navigation3.scene.DialogSceneStrategy`).
- `BottomSheetSceneStrategy` is **not** part of the library: copy the implementation from the official `bottomsheet` recipe (in the `navigation-3` skill) into `:core:ui` once and reuse it. It wraps `ModalBottomSheet` in an `OverlayScene`.
- ❌ Do not show modals with local `var showDialog by remember { … }` state when the modal is a navigation destination (it is lost on process death and is invisible to back handling).
- ✅ Purely transient, in-screen confirmations (e.g. a snackbar, a menu) stay as local UI state — they are not destinations.

---

## 🧠 ViewModel per Entry

Add `rememberViewModelStoreNavEntryDecorator()` (from `lifecycle-viewmodel-navigation3`) to the entry decorators. Each `NavEntry` then owns a `ViewModelStore`: a new ViewModel per key instance, cleared when the entry is popped.

With Hilt, pass the key to the ViewModel through **assisted injection** (official `passingarguments` recipe):

```kotlin
@HiltViewModel(assistedFactory = RestaurantDetailViewModel.Factory::class)
class RestaurantDetailViewModel @AssistedInject constructor(
    @Assisted private val key: RestaurantDetailKey,
    private val observeRestaurant: ObserveRestaurantUseCase,
) : ViewModel() {

    val uiState: StateFlow<RestaurantDetailUiState> = observeRestaurant(key.restaurantId)
        .map { RestaurantDetailUiState.Success(it) }
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), RestaurantDetailUiState.Loading)

    @AssistedFactory
    interface Factory {
        fun create(key: RestaurantDetailKey): RestaurantDetailViewModel
    }
}

// In the entryProvider
entry<RestaurantDetailKey> { key ->
    val viewModel = hiltViewModel<RestaurantDetailViewModel, RestaurantDetailViewModel.Factory>(
        creationCallback = { factory -> factory.create(key) },
    )
    RestaurantDetailRoute(viewModel = viewModel, onBack = navigator::goBack)
}
```

- `hiltViewModel` is imported from `androidx.hilt.lifecycle.viewmodel.compose` (as in the official recipe), not from the Navigation 2 `hilt-navigation-compose` artifact. **Verify** the `androidx.hilt:hilt-lifecycle-viewmodel-compose` version against the Hilt 2.60.1 / KSP 2.3.12 toolchain when implementing.
- ✅ The key is the single source of navigation arguments — no `SavedStateHandle.toRoute()`.
- ❌ Never pass the `Navigator`, `NavBackStack` or `NavigationState` into a ViewModel.

---

## 🧭 ViewModel Navigation Events

ViewModels decide **when** to navigate; the route composable decides **how**, by calling the `Navigator` through lambdas. The ViewModel never sees navigation types.

```kotlin
sealed interface HomeEvent {
    data class OpenRestaurant(val restaurantId: String) : HomeEvent
}

@HiltViewModel
class HomeViewModel @Inject constructor(/* use cases */) : ViewModel() {
    private val _events = Channel<HomeEvent>(Channel.BUFFERED)
    val events: Flow<HomeEvent> = _events.receiveAsFlow()

    fun onRestaurantClicked(id: String) {
        viewModelScope.launch { _events.send(HomeEvent.OpenRestaurant(id)) }
    }
}

@Composable
fun HomeRoute(
    onOpenRestaurant: (String) -> Unit,
    viewModel: HomeViewModel = hiltViewModel(),
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val lifecycleOwner = LocalLifecycleOwner.current // entry-scoped owner provided by NavDisplay

    LaunchedEffect(viewModel, lifecycleOwner) {
        lifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
            viewModel.events.collect { event ->
                when (event) {
                    is HomeEvent.OpenRestaurant -> onOpenRestaurant(event.restaurantId)
                }
            }
        }
    }

    HomeScreen(uiState = uiState, onRestaurantClick = viewModel::onRestaurantClicked)
}
```

For direct UI actions with no logic before navigating, pass the lambda straight through (`onClick = onOpenRestaurant`) — no event round-trip needed.

### Returning Results

Navigation 3 replaces `previousBackStackEntry.savedStateHandle` with an event bus: add `rememberResultEventBusNavEntryDecorator()` to the decorators, send with `LocalResultEventBus.current.sendResult(...)`, receive with `ResultEffect<T> { … }`. Results are **in-memory** and do not survive process death; if a result must persist, put it in the destination key or in a `SavedStateHandle`-backed ViewModel. See the `results-event` / `results-state` recipes in the `navigation-3` skill.

---

## 📐 Adaptive Layouts (List-Detail)

Multi-pane layouts are scene strategies, not separate screens. Use `adaptive-navigation3`:

```kotlin
val listDetailStrategy = rememberListDetailSceneStrategy<NavKey>()

entry<HomeKey>(
    metadata = ListDetailSceneStrategy.listPane(
        detailPlaceholder = { SelectRestaurantPlaceholder() },
    ),
) { HomeRoute(/* … */) }

entry<RestaurantDetailKey>(metadata = ListDetailSceneStrategy.detailPane()) { key -> /* … */ }
```

- Pass `listDetailStrategy` in `sceneStrategies` after the overlay strategies.
- The official `adaptive` skill mandates the Navigation 3 `SceneStrategy` approach; do **not** use `ListDetailPaneScaffold` / `SupportingPaneScaffold` directly.
- On a list-detail layout the detail pane must not show a back arrow, and full-screen detail modes must be disabled.
- Only use list-detail when the detail does not need the full screen (e.g. not for photo galleries).

---

## ⬅️ Predictive Back

- `NavDisplay` handles system back for you: it calls `onBack` and animates with `predictivePopTransitionSpec` (customisable together with `transitionSpec` / `popTransitionSpec`, also per entry through metadata — see the `animations` recipe).
- `android:enableOnBackInvokedCallback` must **not** be set to `false`. It defaults to `true` on API 36+; ARC Labs targets 36, so leave the manifest attribute out unless a dependency forces it.
- For screen-level interception (unsaved changes, in-screen state), use the Navigation Event APIs described in the official `navigation-event` skill (`NavigationBackHandler` with predictive progress) instead of the legacy `BackHandler`. Verify the exact signature with that skill before use.
- ❌ Never consume back in a way that leaves the back stack and the UI out of sync — back must always end in `navigator.goBack()` or an explicit key removal.

---

## 🔗 Deep Links

Deep links are resolved **outside** the UI: match the incoming `Intent` in the Activity and build the initial back stack from the result.

```kotlin
val restaurantMatcher = UriDeepLinkMatcher(
    DeepLinkUri("https://favres.app/restaurant/{restaurantId}"),
    serializer<RestaurantDetailKey>(),
)
```

Match in `onCreate` and `onNewIntent` with `DeepLinkRequest(intent = intent)` and seed the back stack (a synthetic back stack keeps "up" behaviour correct). Follow the `deeplinks-*` recipes in the `navigation-3` skill; App Links also need the `intent-filter` with `android:autoVerify="true"` in the manifest and the `assetlinks.json` on the domain. Review exported components with the official `android-intent-security` skill.

---

## 🧪 Testing Navigation

Because the back stack is plain state, most navigation logic is testable without a device.

- **Navigator logic** — unit-test the `Navigator` against a real `NavigationState` (no mocks): navigate to a top-level key switches tabs; navigate to a child key pushes on the current stack; `goBack()` at a tab root returns to the start tab.
- **Shell behaviour** — Compose UI tests (`createComposeRule` from `androidx.compose.ui.test.junit4.v2` — the non-v2 rules are deprecated in Compose 1.12; JUnit 4 runner) executed on the JVM with **Robolectric**: click a tab item, assert the destination content is displayed.
- **Layout** — Compose Preview Screenshot Testing (`@PreviewTest`) of the shell at compact / medium / expanded widths to prove the bar ↔ rail switch.

```kotlin
@RunWith(AndroidJUnit4::class) // Robolectric
class AppShellTest {
    @get:Rule val composeRule = createComposeRule()

    @Test
    fun selectingFavoritesTabShowsFavorites() {
        // Given
        composeRule.setContent { ArcTheme { AppShell() } }

        // When
        composeRule.onNodeWithText("Favorites").performClick()

        // Then
        composeRule.onNodeWithTag(FavoritesTestTags.SCREEN).assertIsDisplayed()
    }
}
```

> A test that only asserts that `backStack.add(x)` adds `x` tests the library, not our code: delete it. Test **our** decisions (which key, which stack, what happens at the root).

---

## ✅ Navigation Checklist

- [ ] Navigation 3 artifacts only (`navigation3-runtime`, `navigation3-ui`, `lifecycle-viewmodel-navigation3`); no `navigation-compose`
- [ ] Every key is `@Serializable` and implements `NavKey`; keys carry ids, not objects
- [ ] Back stacks created with `rememberNavBackStack` (or the `NavigationState` holder for tabs)
- [ ] `NavDisplay.onBack` always routes through `navigator.goBack()` / `removeLastOrNull()`
- [ ] `rememberSaveableStateHolderNavEntryDecorator` + `rememberViewModelStoreNavEntryDecorator` installed
- [ ] ViewModels receive their key via Hilt assisted injection; never a `Navigator`
- [ ] Tabs rendered with `NavigationSuiteScaffold`; insets applied per screen
- [ ] Dialogs and bottom sheets are keys rendered by `DialogSceneStrategy` / `BottomSheetSceneStrategy`
- [ ] List-detail via `ListDetailSceneStrategy` where it makes sense
- [ ] Predictive back not disabled; custom back via Navigation Event APIs
- [ ] Navigation clicks wrapped in `dropUnlessResumed`
- [ ] Navigator logic and shell covered by JVM tests + screenshot tests

---

## 🚫 Common Mistakes

### 1. Porting Nav2 Habits (`NavController`, Graphs, `popUpTo`)

```kotlin
// ❌ WRONG — Navigation 2 in new code
NavHost(navController, startDestination = Home) {
    composable<Home> { HomeScreen() }
}
navController.navigate(Detail(id)) { popUpTo(Home) { inclusive = false } }
```

```kotlin
// ✅ CORRECT — the back stack is a list you own
backStack.add(RestaurantDetailKey(id))
backStack.removeLastOrNull()
```

### 2. Navigation Types Inside the ViewModel

```kotlin
// ❌ WRONG
class HomeViewModel(private val backStack: NavBackStack<NavKey>) : ViewModel()
```

```kotlin
// ✅ CORRECT — emit an event or expose a lambda-driven API; the route composable navigates
viewModelScope.launch { _events.send(HomeEvent.OpenRestaurant(id)) }
```

### 3. Missing the ViewModel Decorator

Without `rememberViewModelStoreNavEntryDecorator()`, `hiltViewModel()` scopes to the Activity: two `RestaurantDetailKey("a")` / `RestaurantDetailKey("b")` entries silently share one ViewModel.

### 4. Modals as Local State

```kotlin
// ❌ WRONG — lost on process death, invisible to predictive back
var showDelete by remember { mutableStateOf(false) }
if (showDelete) DeleteRestaurantDialog(onDismiss = { showDelete = false })
```

```kotlin
// ✅ CORRECT — a key rendered by DialogSceneStrategy
navigator.navigate(DeleteRestaurantDialogKey(restaurantId))
```

### 5. Non-Serializable Keys

```kotlin
// ❌ WRONG — crashes when the back stack is saved
@Serializable data class DetailKey(val restaurant: Restaurant, val onDone: () -> Unit) : NavKey
```

```kotlin
// ✅ CORRECT
@Serializable data class RestaurantDetailKey(val restaurantId: String) : NavKey
```

---

## 📚 Further Reading

- **Official `navigation-3` skill** (`android/skills` plugin): migration guide + recipes (basic, dialog, bottom sheet, common UI, multiple back stacks, passing arguments with Hilt, results, deep links, list-detail)
- **Official `adaptive` skill**: `NavigationSuiteScaffold`, list-detail / supporting pane scenes
- **Official `navigation-event` skill**: predictive back handlers
- [Navigation 3 — Android Developers](https://developer.android.com/guide/navigation/navigation-3)
- [Navigation 3 recipes](https://github.com/android/nav3-recipes)
- [Presentation Layer](../Layers/presentation.md) · [MVVM](mvvm.md) · [UI Guidelines](../Quality/ui-guidelines.md)
