# ⚡ Compose Performance

**A comprehensive guide to writing performant Jetpack Compose code. Covers recomposition, stability, lazy layouts, side effects, modifiers, debugging, and baseline profiles. Follow these patterns to deliver smooth 60fps experiences across all devices.**

---

## 🎯 Why Performance Matters

Jetpack Compose uses a **declarative** rendering model. Instead of mutating views directly, you
describe what the UI should look like for a given state, and the framework figures out the
minimal set of changes needed. This is powerful, but it introduces a class of performance
pitfalls that do not exist in the imperative View system.

### The Recomposition Model

Every time state changes, Compose **recomposes** the affected parts of the UI tree. If your
composables are written carelessly, a single state change can trigger a cascade of unnecessary
work, resulting in:

- **Janky scrolling** and dropped frames
- **Excessive CPU usage** draining battery
- **Sluggish interactions** that frustrate users
- **Memory pressure** from unnecessary allocations

### The 16ms Budget

At 60fps, each frame has approximately **16 milliseconds** to complete three phases:

1. **Composition** -- determine what composables are on screen
2. **Layout** -- measure and place each element
3. **Drawing** -- render pixels to the canvas

If any single frame exceeds 16ms, the user perceives a **dropped frame**. Multiple dropped
frames in succession create the perception of a laggy, unpolished app.

### Performance is a Feature

Users may not notice when performance is good, but they **always** notice when it is bad.
A restaurant app that stutters while scrolling a list of nearby places feels broken, even if
every feature works correctly. Performance is not an optimization you bolt on at the end -- it
is a quality attribute you design for from day one.

---

## 🔄 Understanding Recomposition

### What Triggers Recomposition

Recomposition occurs when **state read by a composable changes**. Compose tracks which
composables read which state objects and only re-executes the composables that are affected.

```kotlin
@Composable
fun Greeting(name: String) {
    // This composable recomposes when `name` changes
    Text(text = "Hello, $name!")
}
```

Triggers include:

- A `MutableState` value changes (`mutableStateOf`, `mutableStateListOf`)
- A `StateFlow` collected via `collectAsStateWithLifecycle()` emits a new value
- A `remember` key changes
- A parent composable recomposes and passes new parameters

### The Three Phases

```
State Change
    |
    v
[ Composition ]  -->  Determine which composables to call
    |
    v
[   Layout    ]  -->  Measure and place each node
    |
    v
[   Drawing   ]  -->  Render pixels on screen
```

Each phase can be skipped independently:

- If a composable's parameters haven't changed, **Composition** is skipped for that subtree
- If sizes and positions haven't changed, **Layout** is skipped
- If visual properties haven't changed, **Drawing** is skipped

### Smart Recomposition

Compose is **smart** about recomposition. It does not blindly re-execute the entire tree.
Instead, it uses **positional memoization** to skip composables whose inputs have not changed.

```kotlin
@Composable
fun UserProfile(user: UserUiModel) {
    Column {
        // Only recomposes if user.name changes
        Text(text = user.name)

        // Only recomposes if user.bio changes
        Text(text = user.bio)

        // This NEVER recomposes (no state dependency)
        Divider()
    }
}
```

### Recomposition Scope

Compose determines the **smallest possible scope** to recompose. A recomposition scope is
roughly equivalent to a non-inline composable function that returns `Unit`.

```kotlin
@Composable
fun CounterScreen() {
    var count by remember { mutableIntStateOf(0) }

    // This entire scope recomposes when count changes
    Column {
        // Recomposes because it reads `count`
        Text(text = "Count: $count")

        // Also recomposes (same scope as Text above)
        // Move to a separate composable to avoid this!
        ExpensiveWidget()

        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}
```

**Fix:** Extract composables to limit the recomposition scope.

```kotlin
@Composable
fun CounterScreen() {
    var count by remember { mutableIntStateOf(0) }

    Column {
        CounterText(count = count)
        ExpensiveWidget() // No longer recomposes when count changes
        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}

@Composable
private fun CounterText(count: Int) {
    // Recomposition is now scoped to just this composable
    Text(text = "Count: $count")
}
```

---

## 📊 Stability

Stability is the **single most important concept** for Compose performance. The Compose
compiler analyzes every parameter of every composable to determine whether it can safely skip
recomposition when the parent recomposes.

### @Stable and @Immutable

A type is **stable** if:

1. The result of `equals` will always return the same result for the same two instances
2. When a public property changes, Composition is notified
3. All public properties are also stable

```kotlin
// ✅ Stable by default (all val, primitive/String types)
data class UserUiModel(
    val id: String,
    val name: String,
    val isActive: Boolean,
)

// ✅ Stable by default (enum)
enum class SortOrder {
    NAME,
    DISTANCE,
    RATING,
}

// ✅ Stable by default (all val, all stable types)
data class FilterState(
    val query: String,
    val sortOrder: SortOrder,
    val maxDistance: Double,
)
```

```kotlin
// ❌ Unstable (mutable list parameter)
data class HomeUiModel(
    val users: List<UserUiModel>,  // List is not stable!
)

// ❌ Unstable (mutable property)
data class CounterState(
    var count: Int,  // var is not stable
)

// ❌ Unstable (contains unstable type)
data class RestaurantListState(
    val restaurants: List<Restaurant>,  // List is unstable
    val filters: Map<String, String>,   // Map is unstable
)
```

```kotlin
// ✅ Fix: Use kotlinx.collections.immutable
import kotlinx.collections.immutable.ImmutableList
import kotlinx.collections.immutable.ImmutableMap

data class HomeUiModel(
    val users: ImmutableList<UserUiModel>,
)

data class RestaurantListState(
    val restaurants: ImmutableList<Restaurant>,
    val filters: ImmutableMap<String, String>,
)
```

### @Immutable Annotation

Use `@Immutable` when you **guarantee** the class will never change after construction, but
the compiler cannot infer this automatically.

```kotlin
// ✅ Annotate when the compiler can't infer stability
@Immutable
data class ThemeConfig(
    val colors: ImmutableList<Color>,
    val typography: TypographySettings,
)
```

> **Warning:** `@Immutable` is a **contract**. If you annotate a class as `@Immutable` but
> mutate it, you will get incorrect UI behavior. Only use it when you are certain.

### @Stable Annotation

Use `@Stable` for types that may change but notify Compose when they do.

```kotlin
// ✅ Stable wrapper around mutable state
@Stable
class SearchState {
    var query by mutableStateOf("")
        private set

    var isLoading by mutableStateOf(false)
        private set

    fun updateQuery(newQuery: String) {
        query = newQuery
    }
}
```

### Stability Rules

| Type | Stable? | Notes |
|------|---------|-------|
| `Int`, `Long`, `Float`, `Double`, `Boolean` | ✅ Yes | Primitives are always stable |
| `String` | ✅ Yes | Immutable by design |
| `Enum` | ✅ Yes | Fixed set of values |
| `data class` (all `val`, all stable types) | ✅ Yes | Inferred by compiler |
| `List<T>` | ❌ No | Use `ImmutableList<T>` |
| `Map<K, V>` | ❌ No | Use `ImmutableMap<K, V>` |
| `Set<T>` | ❌ No | Use `ImmutableSet<T>` |
| Classes from external modules | ❌ Maybe | Depends on compiler analysis |
| `var` property in data class | ❌ No | Mutable = unstable |
| `LocalDateTime`, `Instant` | ❌ Maybe | Depends on the library version |

### Adding kotlinx.collections.immutable

In your `build.gradle.kts`:

```kotlin
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-collections-immutable:0.3.7")
}
```

Convert standard collections:

```kotlin
import kotlinx.collections.immutable.toImmutableList
import kotlinx.collections.immutable.toImmutableMap

// In your ViewModel or mapping layer
val uiModel = HomeUiModel(
    users = userList.toImmutableList(),
)
```

---

## 🧠 remember and derivedStateOf

### remember

`remember` stores a value across recompositions. Without it, expensive computations run
**every single time** the composable recomposes.

```kotlin
// ✅ Cache expensive computation
@Composable
fun RestaurantList(restaurants: List<Restaurant>) {
    val sortedRestaurants = remember(restaurants) {
        restaurants.sortedBy { it.name }
    }

    LazyColumn {
        items(sortedRestaurants, key = { it.id }) { restaurant ->
            RestaurantCard(restaurant)
        }
    }
}

// ❌ Recalculates on every recomposition
@Composable
fun RestaurantList(restaurants: List<Restaurant>) {
    val sortedRestaurants = restaurants.sortedBy { it.name } // Runs every time!

    LazyColumn {
        items(sortedRestaurants, key = { it.id }) { restaurant ->
            RestaurantCard(restaurant)
        }
    }
}
```

### remember with Multiple Keys

```kotlin
// ✅ Recompute only when either key changes
@Composable
fun FilteredResults(items: List<Item>, query: String) {
    val filteredItems = remember(items, query) {
        items.filter { it.name.contains(query, ignoreCase = true) }
    }

    ResultsList(filteredItems)
}
```

### rememberSaveable

Use `rememberSaveable` for state that should survive configuration changes (rotation, process
death).

```kotlin
// ✅ Survives configuration changes
@Composable
fun SearchBar() {
    var query by rememberSaveable { mutableStateOf("") }

    TextField(
        value = query,
        onValueChange = { query = it },
        placeholder = { Text("Search restaurants...") },
    )
}
```

### derivedStateOf

`derivedStateOf` creates a **derived state** that only triggers recomposition when the
**derived value** changes, not when the source state changes.

```kotlin
// ✅ Recompose only when derived value changes (boolean flip)
@Composable
fun ScrollToTopButton(listState: LazyListState) {
    val showButton by remember {
        derivedStateOf { listState.firstVisibleItemIndex > 0 }
    }

    AnimatedVisibility(visible = showButton) {
        FloatingActionButton(onClick = { /* scroll to top */ }) {
            Icon(Icons.Default.ArrowUpward, contentDescription = "Scroll to top")
        }
    }
}
```

Without `derivedStateOf`, the composable would recompose on **every scroll pixel** because
`firstVisibleItemIndex` changes frequently. With `derivedStateOf`, it only recomposes when
the boolean value flips between `true` and `false`.

### When NOT to Use derivedStateOf

```kotlin
// ❌ Wrong use of derivedStateOf (no filtering of changes)
@Composable
fun UserGreeting(firstName: String, lastName: String) {
    val fullName by remember {
        derivedStateOf { "$firstName $lastName" }
    }
    Text(text = "Hello, $fullName!")
}

// ✅ Just use direct computation (derivedStateOf adds overhead with no benefit)
@Composable
fun UserGreeting(firstName: String, lastName: String) {
    val fullName = "$firstName $lastName"
    Text(text = "Hello, $fullName!")
}
```

**Rule of thumb:** Use `derivedStateOf` when the derived value changes **less frequently**
than the source state. If they change at the same rate, `derivedStateOf` only adds overhead.

### Common derivedStateOf Use Cases

```kotlin
// ✅ Form validation (many fields, one boolean result)
val isFormValid by remember {
    derivedStateOf {
        name.isNotBlank() && email.contains("@") && password.length >= 8
    }
}

// ✅ Threshold detection (continuous value, discrete output)
val isScrolledPastHeader by remember {
    derivedStateOf { scrollState.value > headerHeight }
}

// ✅ Filtered count (large list, single number)
val activeUserCount by remember {
    derivedStateOf { users.count { it.isActive } }
}
```

---

## 📋 LazyColumn/LazyRow Performance

Lazy layouts are the Compose equivalent of `RecyclerView`. They only compose and lay out
items that are visible on screen. However, they can still be a major performance bottleneck
if used incorrectly.

### Stable Keys

Keys tell Compose how to **identify** items across recompositions. Without stable keys,
Compose uses the item's **position** as its identity, which causes unnecessary recomposition
whenever items are reordered, inserted, or removed.

```kotlin
// ✅ Provide unique stable keys
LazyColumn {
    items(items = restaurants, key = { it.id }) { restaurant ->
        RestaurantCard(restaurant)
    }
}

// ❌ No key (uses position, causes unnecessary recomposition on reorder)
LazyColumn {
    items(restaurants) { restaurant ->
        RestaurantCard(restaurant)
    }
}
```

### Why Keys Matter

Consider a list `[A, B, C]` that changes to `[B, A, C]`:

**Without keys (position-based):**
- Position 0: was A, now B -- recompose
- Position 1: was B, now A -- recompose
- Position 2: was C, still C -- skip

**With keys (identity-based):**
- Item A: moved from position 0 to 1 -- reuse, no recompose
- Item B: moved from position 1 to 0 -- reuse, no recompose
- Item C: still at position 2 -- skip

### contentType

When your lazy list contains **multiple item types**, use `contentType` to help Compose
reuse compositions more efficiently.

```kotlin
// ✅ Content type for heterogeneous lists
LazyColumn {
    items(
        items = feedItems,
        key = { it.id },
        contentType = { it.type },
    ) { item ->
        when (item) {
            is FeedItem.Header -> HeaderRow(item)
            is FeedItem.RestaurantCard -> RestaurantRow(item)
            is FeedItem.PromoBanner -> PromoRow(item)
            is FeedItem.CategorySection -> CategoryRow(item)
        }
    }
}
```

Without `contentType`, Compose may try to reuse the composition of a `HeaderRow` for a
`RestaurantRow`, which is wasteful. With `contentType`, it only reuses compositions of
the same type.

### Avoid Heavy Composables Inside Items

```kotlin
// ❌ Heavy computation inside each item
LazyColumn {
    items(restaurants, key = { it.id }) { restaurant ->
        val distance = calculateHaversineDistance(
            userLat, userLng, restaurant.lat, restaurant.lng,
        ) // Runs on every scroll!
        RestaurantCard(restaurant, distance)
    }
}

// ✅ Pre-compute outside the lazy list
@Composable
fun RestaurantListScreen(restaurants: List<Restaurant>, userLocation: Location) {
    val restaurantsWithDistance = remember(restaurants, userLocation) {
        restaurants.map { restaurant ->
            restaurant to calculateHaversineDistance(
                userLocation.lat, userLocation.lng,
                restaurant.lat, restaurant.lng,
            )
        }
    }

    LazyColumn {
        items(restaurantsWithDistance, key = { it.first.id }) { (restaurant, distance) ->
            RestaurantCard(restaurant, distance)
        }
    }
}
```

### Avoid Nesting Scrollables

```kotlin
// ❌ Nested scrollable (crashes or poor performance)
LazyColumn {
    item {
        LazyRow(items = horizontalItems) { /* ... */ }
    }
    item {
        Column(modifier = Modifier.verticalScroll(rememberScrollState())) {
            // This is another scrollable inside LazyColumn!
        }
    }
}

// ✅ Use fixed-height nested scrollables or flatten the structure
LazyColumn {
    item {
        LazyRow(
            modifier = Modifier.height(200.dp),
        ) {
            items(horizontalItems, key = { it.id }) { item ->
                HorizontalCard(item)
            }
        }
    }
}
```

### Item Animations

```kotlin
// ✅ Enable item animations with keys
LazyColumn {
    items(restaurants, key = { it.id }) { restaurant ->
        RestaurantCard(
            restaurant = restaurant,
            modifier = Modifier.animateItem(),
        )
    }
}
```

---

## 🔄 Side Effects Performance

Side effects are operations that escape the scope of a composable function, such as launching
coroutines, registering callbacks, or writing to shared state. Misused side effects can cause
performance problems.

### LaunchedEffect Keys

```kotlin
// ✅ Minimal keys -- only restart when necessary
@Composable
fun RestaurantDetailScreen(restaurantId: String, viewModel: RestaurantDetailViewModel) {
    LaunchedEffect(restaurantId) {
        viewModel.loadRestaurant(restaurantId)
    }
}

// ❌ Using Unit as key -- runs only once but never re-runs
LaunchedEffect(Unit) {
    viewModel.loadRestaurant(restaurantId)
    // If restaurantId changes, this does NOT re-run!
}

// ❌ Using an unstable key -- restarts too often
LaunchedEffect(someListThatChangesFrequently) {
    viewModel.loadData()
}
```

### rememberCoroutineScope for Event Handlers

```kotlin
// ✅ Use rememberCoroutineScope for event-driven coroutines
@Composable
fun SaveButton(viewModel: ProfileViewModel) {
    val scope = rememberCoroutineScope()

    Button(
        onClick = {
            scope.launch {
                viewModel.saveProfile()
            }
        },
    ) {
        Text("Save")
    }
}

// ❌ Don't use LaunchedEffect for one-shot user actions
@Composable
fun SaveButton(viewModel: ProfileViewModel) {
    var shouldSave by remember { mutableStateOf(false) }

    if (shouldSave) {
        LaunchedEffect(Unit) {
            viewModel.saveProfile()
            shouldSave = false
        }
    }

    Button(onClick = { shouldSave = true }) {
        Text("Save")
    }
}
```

### snapshotFlow for Converting State to Flow

```kotlin
// ✅ Efficient: only processes actual changes
LaunchedEffect(listState) {
    snapshotFlow { listState.firstVisibleItemIndex }
        .distinctUntilChanged()
        .collect { index ->
            analytics.trackScroll(index)
        }
}
```

### Avoid Creating New Lambdas Unnecessarily

```kotlin
// ❌ New lambda allocated on every recomposition
@Composable
fun ItemList(items: List<Item>, onItemClick: (String) -> Unit) {
    LazyColumn {
        items(items, key = { it.id }) { item ->
            ItemRow(
                item = item,
                onClick = { onItemClick(item.id) }, // New lambda every recomposition
            )
        }
    }
}

// ✅ Hoist the callback and use remember or a method reference
@Composable
fun ItemList(items: List<Item>, onItemClick: (String) -> Unit) {
    LazyColumn {
        items(items, key = { it.id }) { item ->
            ItemRow(
                item = item,
                onItemClick = onItemClick,
            )
        }
    }
}

@Composable
private fun ItemRow(item: Item, onItemClick: (String) -> Unit) {
    Row(
        modifier = Modifier.clickable { onItemClick(item.id) },
    ) {
        Text(text = item.name)
    }
}
```

---

## 📐 Modifier Best Practices

Modifiers can either help or hurt performance depending on how they are used.

### Lambda Modifiers for Frequently Changing Values

Some modifiers accept a **lambda** version that defers reading of state to the Layout or
Drawing phase, bypassing the Composition phase entirely.

```kotlin
// ✅ Lambda modifier -- only triggers Drawing phase, skips Composition
@Composable
fun AnimatedOffset(offsetX: State<Float>) {
    Box(
        modifier = Modifier
            .offset { IntOffset(x = offsetX.value.roundToInt(), y = 0) }
            .size(100.dp)
            .background(Color.Blue),
    )
}

// ❌ Direct modifier -- triggers full Composition + Layout + Drawing
@Composable
fun AnimatedOffset(offsetX: Float) {
    Box(
        modifier = Modifier
            .offset(x = offsetX.dp, y = 0.dp) // Recomposes on every change!
            .size(100.dp)
            .background(Color.Blue),
    )
}
```

### Common Lambda Modifier Alternatives

| Standard Modifier | Lambda Version | Phase Skipped |
|-------------------|---------------|---------------|
| `Modifier.offset(x, y)` | `Modifier.offset { IntOffset(x, y) }` | Composition |
| `Modifier.alpha(value)` | `Modifier.graphicsLayer { alpha = value }` | Composition + Layout |
| `Modifier.rotate(degrees)` | `Modifier.graphicsLayer { rotationZ = degrees }` | Composition + Layout |
| `Modifier.scale(scale)` | `Modifier.graphicsLayer { scaleX = s; scaleY = s }` | Composition + Layout |

### graphicsLayer for Animations

```kotlin
// ✅ All visual transforms in graphicsLayer (Layout + Composition skipped)
@Composable
fun PulsingIcon(scale: State<Float>, alpha: State<Float>) {
    Icon(
        imageVector = Icons.Default.Favorite,
        contentDescription = null,
        modifier = Modifier.graphicsLayer {
            scaleX = scale.value
            scaleY = scale.value
            this.alpha = alpha.value
        },
    )
}
```

### Avoid Reordering Modifiers Dynamically

```kotlin
// ❌ Conditional modifier chains can cause layout instability
@Composable
fun ConditionalCard(isHighlighted: Boolean) {
    Card(
        modifier = if (isHighlighted) {
            Modifier.padding(8.dp).border(2.dp, Color.Red)
        } else {
            Modifier.padding(16.dp)
        },
    ) {
        Text("Content")
    }
}

// ✅ Use a consistent modifier chain
@Composable
fun ConditionalCard(isHighlighted: Boolean) {
    Card(
        modifier = Modifier
            .padding(if (isHighlighted) 8.dp else 16.dp)
            .then(
                if (isHighlighted) Modifier.border(2.dp, Color.Red) else Modifier
            ),
    ) {
        Text("Content")
    }
}
```

### Reuse Modifier Instances

```kotlin
// ✅ Define constant modifiers outside the composable
private val CardModifier = Modifier
    .fillMaxWidth()
    .padding(horizontal = 16.dp, vertical = 8.dp)

@Composable
fun RestaurantCard(restaurant: Restaurant) {
    Card(modifier = CardModifier) {
        // ...
    }
}
```

---

## 🔧 Debugging Recomposition

### Layout Inspector

Android Studio's Layout Inspector shows **recomposition counts** directly on the component
tree. Enable it via:

1. Run your app in debug mode
2. Open **Tools > Layout Inspector**
3. Check **Show Recomposition Counts** in the toolbar

Look for composables with **high recomposition counts** -- these are your optimization
targets.

### Compose Compiler Metrics

The Compose compiler can generate **stability reports** that tell you exactly which classes
are stable, unstable, or skippable.

#### Enabling Compiler Metrics

In your module-level `build.gradle.kts`:

```kotlin
android {
    // ...
}

// For Kotlin 2.0+ with the Compose compiler Gradle plugin
composeCompiler {
    reportsDestination = layout.buildDirectory.dir("compose_compiler")
    metricsDestination = layout.buildDirectory.dir("compose_compiler")
}
```

#### Running the Report

```bash
./gradlew assembleRelease
```

This generates files in `build/compose_compiler/`:

- `<module>-classes.txt` -- Stability of each class
- `<module>-composables.txt` -- Skippability of each composable
- `<module>-composables.csv` -- Machine-readable composable data

#### Reading the Stability Report

```
// From app-classes.txt
stable class UserUiModel {
  stable val id: String
  stable val name: String
  stable val isActive: Boolean
}

unstable class HomeUiModel {
  unstable val users: List<UserUiModel>   // <-- Problem!
}
```

```
// From app-composables.txt
restartable skippable scheme("[androidx.compose.ui.UiComposable]") fun UserCard(
  stable user: UserUiModel
)

restartable scheme("[androidx.compose.ui.UiComposable]") fun HomeScreen(
  unstable model: HomeUiModel   // <-- Not skippable!
)
```

**Key terms:**
- **restartable** -- Compose can re-enter this composable during recomposition
- **skippable** -- Compose can skip this composable if inputs haven't changed
- **stable** -- The parameter type is stable
- **unstable** -- The parameter type is unstable (prevents skipping)

### Recomposition Highlighter

Add a debug modifier to visualize recompositions in your UI:

```kotlin
// Debug utility -- remove before release
fun Modifier.recompositionHighlighter(): Modifier = composed {
    val count = remember { mutableIntStateOf(0) }
    SideEffect { count.intValue++ }

    this.then(
        Modifier.drawWithContent {
            drawContent()
            val color = when (count.intValue) {
                0 -> Color.Transparent
                1 -> Color.Green.copy(alpha = 0.3f)
                else -> Color.Red.copy(alpha = 0.3f)
            }
            drawRect(color = color)
        }
    )
}
```

---

## 🏗️ Baseline Profiles

### What They Are

Baseline Profiles are **AOT compilation hints** that tell the Android Runtime (ART) which
code paths are critical to your app's startup and common user interactions. Without them,
your code runs in interpreted mode or JIT-compiled mode for the first several runs.

Benefits:

- **Faster app startup** (up to 30% improvement)
- **Smoother first-run experience** for new users
- **Reduced jank** during initial interactions
- **Consistent performance** from the very first launch

### Default Compose Baseline Profiles

Jetpack Compose libraries **ship with their own baseline profiles** since version 1.3. These
cover common Compose framework paths. However, your **app-specific** code is not included.

### Generating Custom Baseline Profiles

#### 1. Add the Baseline Profile Gradle Plugin

In your project-level `build.gradle.kts`:

```kotlin
plugins {
    id("androidx.baselineprofile") version "1.3.3" apply false
}
```

#### 2. Create a Baseline Profile Module

Use Android Studio: **File > New > Module > Baseline Profile Generator**.

This creates a `:baselineprofile` module with a test class.

#### 3. Write Profile Generation Tests

```kotlin
@RunWith(AndroidJUnit4::class)
@LargeTest
class BaselineProfileGenerator {

    @get:Rule
    val rule = BaselineProfileRule()

    @Test
    fun generateBaselineProfile() {
        rule.collect(
            packageName = "com.arclabs.favres",
        ) {
            // App startup
            pressHome()
            startActivityAndWait()

            // Critical user journeys
            device.findObject(By.text("Search")).click()
            device.waitForIdle()

            device.findObject(By.res("search_field")).text = "Pizza"
            device.waitForIdle()

            // Scroll restaurant list
            val list = device.findObject(By.res("restaurant_list"))
            list.setGestureMargin(device.displayWidth / 5)
            list.fling(Direction.DOWN)
            device.waitForIdle()
        }
    }
}
```

#### 4. Generate the Profile

```bash
./gradlew :app:generateBaselineProfile
```

The generated profile appears at `app/src/main/baseline-prof.txt`.

#### 5. Verify the Profile

```bash
./gradlew :app:assembleRelease
# Check that the APK contains the profile
```

### Startup Profiles

Startup profiles are a subset of baseline profiles focused specifically on app launch:

```kotlin
@RunWith(AndroidJUnit4::class)
class StartupProfileGenerator {
    @get:Rule
    val rule = BaselineProfileRule()

    @Test
    fun generateStartupProfile() {
        rule.collect(
            packageName = "com.arclabs.favres",
            includeInStartupProfile = true,
        ) {
            startActivityAndWait()
        }
    }
}
```

---

## ✅ Performance Checklist

Use this checklist when reviewing Compose code for performance issues:

### Stability
- [ ] All UI model classes use `val` properties only
- [ ] Collections use `ImmutableList`, `ImmutableMap`, `ImmutableSet` from kotlinx
- [ ] External types wrapped with `@Stable` or `@Immutable` when appropriate
- [ ] Compose compiler metrics show all critical composables as **skippable**

### remember and State
- [ ] Expensive computations wrapped in `remember` with correct keys
- [ ] `derivedStateOf` used when derived value changes less often than source
- [ ] `rememberSaveable` used for user-input state that survives config changes
- [ ] State hoisted appropriately to minimize recomposition scope

### Lazy Layouts
- [ ] All `LazyColumn`/`LazyRow` items have stable `key` parameters
- [ ] Heterogeneous lists use `contentType`
- [ ] No heavy computation inside item lambdas
- [ ] No nested scrollable containers without fixed dimensions
- [ ] Item animations use `animateItem()` modifier

### Modifiers
- [ ] Frequently animated values use lambda modifiers (`offset {}`, `graphicsLayer {}`)
- [ ] Modifier chains are consistent (no conditional reordering)
- [ ] Constant modifiers extracted to top-level `val` where practical

### Side Effects
- [ ] `LaunchedEffect` keys are minimal and correct
- [ ] `rememberCoroutineScope` used for event-driven coroutines
- [ ] `snapshotFlow` used for converting Compose state to Flow
- [ ] No unnecessary lambda allocations in hot paths

### Tooling
- [ ] Layout Inspector checked for unexpected recomposition counts
- [ ] Compose compiler metrics reviewed for unstable classes
- [ ] Baseline profiles generated for critical user journeys
- [ ] Performance tested on low-end devices (not just emulators)

---

## 🚫 Common Mistakes

### Mistake 1: Expensive Computation in Composable Body Without remember

```kotlin
// ❌ Runs regex parsing on EVERY recomposition
@Composable
fun FormattedDescription(rawHtml: String) {
    val cleanText = rawHtml
        .replace(Regex("<[^>]*>"), "")
        .replace("&amp;", "&")
        .replace("&lt;", "<")
        .replace("&gt;", ">")
        .trim()

    Text(text = cleanText)
}

// ✅ Cache the result with remember
@Composable
fun FormattedDescription(rawHtml: String) {
    val cleanText = remember(rawHtml) {
        rawHtml
            .replace(Regex("<[^>]*>"), "")
            .replace("&amp;", "&")
            .replace("&lt;", "<")
            .replace("&gt;", ">")
            .trim()
    }

    Text(text = cleanText)
}
```

### Mistake 2: Unstable Parameters Causing Unnecessary Recomposition

```kotlin
// ❌ List<Restaurant> is unstable -- RestaurantList is never skipped
@Composable
fun RestaurantList(restaurants: List<Restaurant>) {
    LazyColumn {
        items(restaurants, key = { it.id }) { restaurant ->
            RestaurantCard(restaurant)
        }
    }
}

// ✅ ImmutableList is stable -- RestaurantList can be skipped
@Composable
fun RestaurantList(restaurants: ImmutableList<Restaurant>) {
    LazyColumn {
        items(restaurants, key = { it.id }) { restaurant ->
            RestaurantCard(restaurant)
        }
    }
}
```

### Mistake 3: Missing Keys in LazyColumn

```kotlin
// ❌ No keys -- items are identified by position
LazyColumn {
    items(notifications) { notification ->
        NotificationRow(notification)
    }
}

// ✅ Stable keys -- items are identified by their unique ID
LazyColumn {
    items(
        items = notifications,
        key = { it.id },
    ) { notification ->
        NotificationRow(notification)
    }
}
```

### Mistake 4: Backwards Writes

A backwards write occurs when you **write to state** that was **already read** during the
current recomposition. This forces Compose to recompose again immediately.

```kotlin
// ❌ Backwards write -- triggers infinite recomposition loop
@Composable
fun CounterDisplay(viewModel: CounterViewModel) {
    val count = viewModel.count

    Text(text = "Count: $count")

    // Writing to state AFTER reading it = backwards write!
    viewModel.incrementReadCount()
}

// ✅ Use SideEffect for write-after-read
@Composable
fun CounterDisplay(viewModel: CounterViewModel) {
    val count = viewModel.count

    Text(text = "Count: $count")

    SideEffect {
        viewModel.incrementReadCount()
    }
}
```

### Mistake 5: Creating New Objects in Composition

```kotlin
// ❌ New list created on every recomposition
@Composable
fun TagChips(tags: List<String>) {
    val chipColors = tags.map { tag ->
        ChipColors(
            background = colorForTag(tag),
            content = Color.White,
        )
    }

    Row {
        chipColors.forEachIndexed { index, colors ->
            Chip(text = tags[index], colors = colors)
        }
    }
}

// ✅ Cache the computed list
@Composable
fun TagChips(tags: List<String>) {
    val chipColors = remember(tags) {
        tags.map { tag ->
            ChipColors(
                background = colorForTag(tag),
                content = Color.White,
            )
        }
    }

    Row {
        chipColors.forEachIndexed { index, colors ->
            Chip(text = tags[index], colors = colors)
        }
    }
}
```

---

## 📚 Further Reading

### Official Documentation

- [Jetpack Compose Performance](https://developer.android.com/develop/ui/compose/performance)
- [Compose Stability Explained](https://developer.android.com/develop/ui/compose/performance/stability)
- [Compose Phases](https://developer.android.com/develop/ui/compose/phases)
- [Compose Compiler Metrics](https://github.com/JetBrains/kotlin/blob/master/plugins/compose/design/compiler-metrics.md)
- [Baseline Profiles](https://developer.android.com/topic/performance/baselineprofiles/overview)

### Libraries

- [kotlinx.collections.immutable](https://github.com/Kotlin/kotlinx.collections.immutable) -- Immutable collection types for stable parameters
- [Compose Compiler Gradle Plugin](https://developer.android.com/develop/ui/compose/compiler) -- Compiler configuration and metrics

### Tools

- [Android Studio Layout Inspector](https://developer.android.com/studio/debug/layout-inspector) -- Visualize recomposition counts
- [Macrobenchmark](https://developer.android.com/topic/performance/benchmarking/macrobenchmark-overview) -- Measure real-world performance
- [Perfetto](https://perfetto.dev/) -- System-level tracing for deep performance analysis

### ARC Labs Studio Standards

- Refer to [Presentation Layer](../Layers/presentation.md) for ViewModel and state management patterns
- Refer to [Testing Standards](./testing.md) for testing Compose UI with performance in mind
