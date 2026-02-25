# 🔍 Code Review

**Code review ensures quality, consistency, and knowledge sharing across the team. Every change is reviewed before merge.**

---

## 🎯 Review Philosophy

### Collaborative Culture

Code reviews at ARC Labs Studio are **collaborative conversations**, not gatekeeping exercises. The reviewer and the author work together toward the same goal: shipping high-quality, maintainable Android code that follows our standards.

**Principles:**

- **Assume good intent.** The author made deliberate choices. Ask questions before assuming mistakes.
- **Teach, don't lecture.** If you see a pattern violation, explain the _why_ behind the standard.
- **Praise good work.** Call out elegant solutions, thorough tests, and clear naming.
- **Stay humble.** Every engineer, regardless of seniority, can learn from a review.

### Knowledge Sharing

Reviews are the single most effective way to spread architectural knowledge across the team. When you review code:

- You learn about parts of the codebase you haven't touched.
- You expose the author to different perspectives and patterns.
- You create a shared understanding of conventions that documentation alone cannot achieve.

### Consistency Enforcement

Our codebase serves as a teaching tool. New team members read existing code to learn ARC Labs conventions. Every merged PR either reinforces or undermines that consistency. Reviews are the last human checkpoint before code becomes the standard others will follow.

### Review Scope

- **Every PR** requires at least one approval before merge.
- **Critical modules** (authentication, payments, data persistence) require two approvals.
- **Self-review first.** Authors must review their own diff before requesting review. Catch the obvious issues yourself.

---

## 📋 Review Checklist

### Architecture

Verify that Clean Architecture boundaries are respected. The dependency rule flows inward: Presentation depends on Domain, Domain depends on nothing, Data depends on Domain.

- [ ] Clean Architecture layers respected (no reverse dependencies)
- [ ] Business logic in Use Cases, not ViewModels or Composables
- [ ] Repository interfaces in Domain, implementations in Data
- [ ] Hilt DI properly configured with correct scopes
- [ ] No framework imports in the Domain layer
- [ ] Use Cases have a single responsibility
- [ ] DTOs mapped to domain entities at the Data layer boundary

**Reverse Dependency Violation:**

```kotlin
// domain/repository/UserRepository.kt
import retrofit2.Response // !! Framework import in Domain layer

interface UserRepository {
    suspend fun getUser(id: String): Response<UserDto> // !! DTO leaking into Domain
}
```

```kotlin
// domain/repository/UserRepository.kt
interface UserRepository {
    suspend fun getUser(id: String): Result<User> // Domain entity, no framework types
}
```

**Business Logic in ViewModel:**

```kotlin
// presentation/viewmodel/OrderViewModel.kt
@HiltViewModel
class OrderViewModel @Inject constructor(
    private val repository: OrderRepository
) : ViewModel() {

    fun placeOrder(items: List<CartItem>) {
        viewModelScope.launch {
            val subtotal = items.sumOf { it.price * it.quantity }
            val tax = subtotal * 0.08
            val total = subtotal + tax
            if (total > 500) {
                applyDiscount(total, 0.10)
            }
            repository.submitOrder(items, total) // Business logic belongs in a Use Case
        }
    }
}
```

```kotlin
// domain/usecase/PlaceOrderUseCase.kt
class PlaceOrderUseCase @Inject constructor(
    private val repository: OrderRepository,
    private val pricingCalculator: PricingCalculator
) {
    suspend operator fun invoke(items: List<CartItem>): Result<Order> {
        val total = pricingCalculator.calculate(items)
        return repository.submitOrder(items, total)
    }
}

// presentation/viewmodel/OrderViewModel.kt
@HiltViewModel
class OrderViewModel @Inject constructor(
    private val placeOrder: PlaceOrderUseCase
) : ViewModel() {

    fun onPlaceOrderClick(items: List<CartItem>) {
        viewModelScope.launch {
            placeOrder(items)
                .onSuccess { _uiState.update { it.copy(orderPlaced = true) } }
                .onFailure { _uiState.update { it.copy(error = it.message) } }
        }
    }
}
```

### Code Quality

- [ ] No `!!` operator (use safe calls, `let`, `requireNotNull` with message)
- [ ] No hardcoded strings in UI (use `R.string` resources)
- [ ] No magic numbers (use named constants or enums)
- [ ] No commented-out code (use version control instead)
- [ ] No TODO without a Linear ticket reference
- [ ] No dead code (unused functions, unreachable branches)
- [ ] ktlint and detekt pass cleanly
- [ ] Functions are short and focused (under 30 lines as a guideline)
- [ ] Naming is clear and intention-revealing
- [ ] Error messages are descriptive and actionable

**The `!!` Operator:**

```kotlin
// Never do this
val userName = user!!.name
val email = prefs.getString("email", null)!!
```

```kotlin
// Safe alternatives
val userName = user?.name ?: "Unknown"
val email = requireNotNull(prefs.getString("email", null)) {
    "Email must be set before accessing user profile"
}
```

**Hardcoded Strings:**

```kotlin
// Never do this
Text(text = "Welcome back!")
Button(onClick = onSubmit) {
    Text("Submit Order")
}
```

```kotlin
// Use string resources
Text(text = stringResource(R.string.welcome_back))
Button(onClick = onSubmit) {
    Text(stringResource(R.string.submit_order))
}
```

**Magic Numbers:**

```kotlin
// Never do this
if (password.length < 8) { /* ... */ }
delay(3000)
LazyColumn {
    items(50) { /* ... */ }
}
```

```kotlin
// Use named constants
companion object {
    private const val MIN_PASSWORD_LENGTH = 8
    private const val DEBOUNCE_MILLIS = 3_000L
    private const val PAGE_SIZE = 50
}

if (password.length < MIN_PASSWORD_LENGTH) { /* ... */ }
delay(DEBOUNCE_MILLIS)
```

**Dead Code and TODOs:**

```kotlin
// Never do this
fun calculateTotal(items: List<Item>): Double {
    // val oldTotal = items.map { it.price }.sum()
    // return oldTotal * 1.08
    val total = items.sumOf { it.price }
    return total * TAX_RATE
}

// TODO: fix this later
fun fetchData() { /* ... */ }
```

```kotlin
// Remove dead code; reference tickets in TODOs
fun calculateTotal(items: List<Item>): Double {
    val subtotal = items.sumOf { it.price }
    return subtotal * TAX_RATE
}

// TODO(FAVRES-234): Add retry logic for network failures
fun fetchData() { /* ... */ }
```

### Kotlin Idioms

- [ ] `data class` for value types and DTOs
- [ ] `sealed interface` for UI state, events, and navigation
- [ ] `operator fun invoke()` for Use Cases
- [ ] Extension functions used appropriately (not abused)
- [ ] Null safety handled idiomatically
- [ ] Scope functions (`let`, `run`, `apply`, `also`, `with`) used correctly
- [ ] `when` expressions are exhaustive
- [ ] Immutable collections preferred (`List` over `MutableList` in public APIs)
- [ ] `Result` or custom sealed types for error handling

**Data Classes:**

```kotlin
// Regular class for a value type is wasteful
class UserProfile(
    val id: String,
    val name: String,
    val email: String
) {
    override fun equals(other: Any?): Boolean { /* manual implementation */ }
    override fun hashCode(): Int { /* manual implementation */ }
    override fun toString(): String { /* manual implementation */ }
}
```

```kotlin
// data class generates equals, hashCode, toString, copy
data class UserProfile(
    val id: String,
    val name: String,
    val email: String
)
```

**Sealed Interfaces for State:**

```kotlin
// Using strings or enums with associated data is fragile
data class UiState(
    val isLoading: Boolean = false,
    val data: List<Item>? = null,
    val error: String? = null
)
// Problem: isLoading=true AND error="fail" is a valid but nonsensical state
```

```kotlin
// Sealed interface makes illegal states unrepresentable
sealed interface UiState {
    data object Loading : UiState
    data class Success(val items: List<Item>) : UiState
    data class Error(val message: String) : UiState
}
```

**Extension Functions:**

```kotlin
// Keep extensions focused and discoverable
fun String.isValidEmail(): Boolean =
    Patterns.EMAIL_ADDRESS.matcher(this).matches()

fun List<CartItem>.totalPrice(): Double =
    sumOf { it.price * it.quantity }

// Avoid: Extension that doesn't logically belong to the receiver type
fun String.saveToDatabase() { /* ... */ } // String shouldn't know about databases
```

**Scope Functions:**

```kotlin
// Avoid nested scope functions or unclear usage
user?.let {
    it.name.let { name ->
        it.email.let { email ->
            updateProfile(name, email) // Deeply nested, hard to read
        }
    }
}
```

```kotlin
// Clear, flat scope function usage
user?.let { activeUser ->
    updateProfile(activeUser.name, activeUser.email)
}

// Use apply for object configuration
val intent = Intent(context, DetailActivity::class.java).apply {
    putExtra(EXTRA_ID, itemId)
    addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
}

// Use also for side effects
val user = repository.getUser(id).also { user ->
    analytics.trackUserLoaded(user.id)
}
```

### Compose Patterns

- [ ] Composables are stateless where possible
- [ ] State hoisted to the appropriate level
- [ ] Preview functions included for all significant composables
- [ ] `Modifier` is the first optional parameter and has a default of `Modifier`
- [ ] Modifier chains are ordered correctly (clickable before padding, etc.)
- [ ] Accessibility: `contentDescription` on images and icons
- [ ] Dark theme verified
- [ ] Material3 components used throughout
- [ ] No side effects in composable body (use `LaunchedEffect`, `SideEffect`)
- [ ] Slot API pattern used for flexible composables
- [ ] String resources used (no hardcoded strings in composables)

**Stateless Composables and State Hoisting:**

```kotlin
// Stateful composable is hard to test and reuse
@Composable
fun SearchBar() {
    var query by remember { mutableStateOf("") }
    var results by remember { mutableStateOf(emptyList<Item>()) }

    LaunchedEffect(query) {
        results = api.search(query) // Side effect mixed with UI
    }

    TextField(value = query, onValueChange = { query = it })
    LazyColumn {
        items(results) { item -> ItemRow(item) }
    }
}
```

```kotlin
// Stateless composable with hoisted state
@Composable
fun SearchBar(
    query: String,
    onQueryChange: (String) -> Unit,
    results: List<Item>,
    modifier: Modifier = Modifier
) {
    Column(modifier = modifier) {
        TextField(value = query, onValueChange = onQueryChange)
        LazyColumn {
            items(results, key = { it.id }) { item ->
                ItemRow(item)
            }
        }
    }
}

@Preview(showBackground = true)
@Composable
private fun SearchBarPreview() {
    AppTheme {
        SearchBar(
            query = "Pizza",
            onQueryChange = {},
            results = listOf(
                Item(id = "1", name = "Pizza Place"),
                Item(id = "2", name = "Pizza House")
            )
        )
    }
}
```

**Modifier Parameter Convention:**

```kotlin
// Modifier must be the first optional parameter with a Modifier default
@Composable
fun ProfileCard(
    user: User,                     // Required parameters first
    modifier: Modifier = Modifier,  // Modifier is first optional param
    onClick: () -> Unit = {}        // Other optional parameters after
) {
    Card(modifier = modifier.clickable(onClick = onClick)) {
        // ...
    }
}
```

**Modifier Chain Ordering:**

```kotlin
// Incorrect ordering: padding applied before clickable creates
// a clickable area smaller than the visual bounds
Box(
    modifier = Modifier
        .padding(16.dp)
        .clickable { onClick() }
)
```

```kotlin
// Correct ordering: clickable area includes the padding
Box(
    modifier = Modifier
        .clickable { onClick() }
        .padding(16.dp)
)
```

**Side Effects:**

```kotlin
// Never perform side effects directly in the composable body
@Composable
fun UserScreen(viewModel: UserViewModel) {
    viewModel.loadUser() // Called on every recomposition!

    val state by viewModel.uiState.collectAsStateWithLifecycle()
    // ...
}
```

```kotlin
// Use LaunchedEffect for suspend side effects
@Composable
fun UserScreen(viewModel: UserViewModel) {
    LaunchedEffect(Unit) {
        viewModel.loadUser() // Called once when entering composition
    }

    val state by viewModel.uiState.collectAsStateWithLifecycle()
    // ...
}
```

### Testing

- [ ] All Use Cases have unit tests
- [ ] All ViewModels have unit tests
- [ ] Tests follow Given-When-Then pattern
- [ ] MockK used for interfaces, fakes for simple types
- [ ] Turbine used for Flow testing
- [ ] Coverage meets requirements (80% apps, 100% libraries)
- [ ] Edge cases tested (empty lists, null values, error states)
- [ ] Test names describe behavior, not implementation
- [ ] No flaky tests (no `delay`, no real network calls)
- [ ] Test data created with factory functions

**Given-When-Then with MockK:**

```kotlin
@Test
fun `should return user profile when repository succeeds`() = runTest {
    // Given
    val expectedUser = User(id = "1", name = "Alice", email = "alice@test.com")
    coEvery { mockRepository.getUser("1") } returns Result.success(expectedUser)

    val useCase = GetUserUseCase(mockRepository)

    // When
    val result = useCase("1")

    // Then
    assertThat(result.isSuccess).isTrue()
    assertThat(result.getOrNull()).isEqualTo(expectedUser)
    coVerify(exactly = 1) { mockRepository.getUser("1") }
}

@Test
fun `should return error when repository fails`() = runTest {
    // Given
    val exception = IOException("Network error")
    coEvery { mockRepository.getUser(any()) } returns Result.failure(exception)

    val useCase = GetUserUseCase(mockRepository)

    // When
    val result = useCase("1")

    // Then
    assertThat(result.isFailure).isTrue()
    assertThat(result.exceptionOrNull()).isInstanceOf(IOException::class.java)
}
```

**Turbine for Flow Testing:**

```kotlin
@Test
fun `should emit loading then success when data loads`() = runTest {
    // Given
    val items = listOf(Item(id = "1", name = "Item 1"))
    coEvery { mockUseCase() } returns flowOf(Result.success(items))

    val viewModel = ItemListViewModel(mockUseCase)

    // When & Then
    viewModel.uiState.test {
        assertThat(awaitItem()).isEqualTo(UiState.Loading)
        assertThat(awaitItem()).isEqualTo(UiState.Success(items))
        cancelAndIgnoreRemainingEvents()
    }
}
```

**Test Data Factory:**

```kotlin
// Use factory functions for test data, not constructors scattered everywhere
object TestFactory {
    fun createUser(
        id: String = "user-1",
        name: String = "Test User",
        email: String = "test@example.com"
    ) = User(id = id, name = name, email = email)

    fun createOrder(
        id: String = "order-1",
        items: List<CartItem> = listOf(createCartItem()),
        total: Double = 29.99
    ) = Order(id = id, items = items, total = total)
}

// Usage in tests
@Test
fun `should display user name`() = runTest {
    // Given
    val user = TestFactory.createUser(name = "Alice")
    // ...
}
```

### Performance

- [ ] `remember` used for expensive computations in composables
- [ ] `LazyColumn`/`LazyRow` has stable keys
- [ ] No unnecessary recompositions (use `derivedStateOf` where appropriate)
- [ ] `withContext(Dispatchers.IO)` for I/O operations in repositories
- [ ] Images loaded with Coil and properly sized
- [ ] Large lists use paging (Paging 3)
- [ ] No blocking calls on the main thread
- [ ] `Immutable`/`Stable` annotations used where the compiler cannot infer stability

**`remember` for Expensive Computations:**

```kotlin
// Recomputed on every recomposition
@Composable
fun FormattedPrice(items: List<CartItem>) {
    val total = items.sumOf { it.price * it.quantity } // Recalculated every recomposition
    val formatted = NumberFormat.getCurrencyInstance().format(total)
    Text(text = formatted)
}
```

```kotlin
// Computed only when items change
@Composable
fun FormattedPrice(items: List<CartItem>) {
    val formatted = remember(items) {
        val total = items.sumOf { it.price * it.quantity }
        NumberFormat.getCurrencyInstance().format(total)
    }
    Text(text = formatted)
}
```

**LazyColumn Keys:**

```kotlin
// Without stable keys, Compose cannot efficiently diff the list
LazyColumn {
    items(users) { user ->
        UserRow(user)
    }
}
```

```kotlin
// Stable keys enable efficient recomposition when the list changes
LazyColumn {
    items(users, key = { it.id }) { user ->
        UserRow(user)
    }
}
```

**`derivedStateOf`:**

```kotlin
// Recomposes every time scrollState changes (every pixel)
@Composable
fun ScrollAwareHeader(scrollState: LazyListState) {
    val showButton = scrollState.firstVisibleItemIndex > 0 // Triggers recomposition on every scroll
    AnimatedVisibility(visible = showButton) {
        ScrollToTopButton()
    }
}
```

```kotlin
// Only recomposes when the derived boolean value actually changes
@Composable
fun ScrollAwareHeader(scrollState: LazyListState) {
    val showButton by remember {
        derivedStateOf { scrollState.firstVisibleItemIndex > 0 }
    }
    AnimatedVisibility(visible = showButton) {
        ScrollToTopButton()
    }
}
```

**`withContext` for I/O:**

```kotlin
// Repository implementation must switch to IO dispatcher
class UserRepositoryImpl @Inject constructor(
    private val api: UserApi,
    private val dao: UserDao,
    @IoDispatcher private val ioDispatcher: CoroutineDispatcher
) : UserRepository {

    override suspend fun getUser(id: String): Result<User> =
        withContext(ioDispatcher) {
            try {
                val dto = api.getUser(id)
                val entity = dto.toDomain()
                dao.insertUser(entity.toEntity())
                Result.success(entity)
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
}
```

### Concurrency

- [ ] `viewModelScope` used for ViewModel coroutines
- [ ] Structured concurrency followed (no `GlobalScope`)
- [ ] Dispatchers injected, not hardcoded
- [ ] `StateFlow` used for UI state (not `LiveData` in new code)
- [ ] `SharedFlow` used for one-shot events
- [ ] `flowOn` used to change upstream dispatcher
- [ ] `collectAsStateWithLifecycle` used in Compose (not `collectAsState`)
- [ ] Cancellation handled correctly

**Dispatcher Injection:**

```kotlin
// Hardcoded dispatcher is untestable
class SyncWorker @Inject constructor(
    private val repository: SyncRepository
) {
    suspend fun sync() {
        withContext(Dispatchers.IO) { // Cannot replace in tests
            repository.syncAll()
        }
    }
}
```

```kotlin
// Injected dispatcher is testable
class SyncWorker @Inject constructor(
    private val repository: SyncRepository,
    @IoDispatcher private val ioDispatcher: CoroutineDispatcher
) {
    suspend fun sync() {
        withContext(ioDispatcher) { // Can be replaced with TestDispatcher
            repository.syncAll()
        }
    }
}

// In your DI module
@Module
@InstallIn(SingletonComponent::class)
object DispatcherModule {
    @Provides
    @IoDispatcher
    fun provideIoDispatcher(): CoroutineDispatcher = Dispatchers.IO
}

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class IoDispatcher
```

**Structured Concurrency:**

```kotlin
// GlobalScope leaks coroutines and ignores lifecycle
fun sendAnalytics(event: AnalyticsEvent) {
    GlobalScope.launch { // Never do this
        analyticsService.send(event)
    }
}
```

```kotlin
// Use a proper scope tied to a lifecycle
@HiltViewModel
class AnalyticsViewModel @Inject constructor(
    private val analyticsService: AnalyticsService
) : ViewModel() {

    fun sendAnalytics(event: AnalyticsEvent) {
        viewModelScope.launch {
            analyticsService.send(event)
        }
    }
}
```

**`collectAsStateWithLifecycle`:**

```kotlin
// collectAsState keeps collecting even when the app is in the background
@Composable
fun UserScreen(viewModel: UserViewModel = hiltViewModel()) {
    val state by viewModel.uiState.collectAsState() // Wastes resources in background
    // ...
}
```

```kotlin
// collectAsStateWithLifecycle respects the lifecycle
@Composable
fun UserScreen(viewModel: UserViewModel = hiltViewModel()) {
    val state by viewModel.uiState.collectAsStateWithLifecycle() // Pauses in background
    // ...
}
```

### Security

- [ ] No hardcoded API keys, tokens, or secrets in source code
- [ ] Secrets stored in `local.properties` or injected via CI environment variables
- [ ] ProGuard/R8 rules configured for release builds
- [ ] HTTPS enforced for all network calls (no cleartext traffic)
- [ ] Sensitive data not logged (no passwords, tokens, PII in log statements)
- [ ] `EncryptedSharedPreferences` used for sensitive local storage
- [ ] No sensitive data in Intent extras without encryption
- [ ] Certificate pinning considered for critical APIs

### Git

- [ ] Conventional commit messages (`feat:`, `fix:`, `refactor:`, etc.)
- [ ] Branch naming follows convention (`feature/ISSUE-ID-description`)
- [ ] PR description is clear and complete
- [ ] PR is scoped to a single concern (not mixing features, refactors, and fixes)
- [ ] No merge commits in the branch (rebase onto main)
- [ ] Commit history is clean and logical
- [ ] No large generated files committed (build outputs, .idea files)

---

## 🤖 AI-Generated Code Review

AI coding assistants accelerate development but introduce specific failure modes. Apply extra scrutiny to AI-generated or AI-assisted code in these areas.

### Deprecated API Usage

AI models are trained on historical data and frequently suggest deprecated APIs.

```kotlin
// AI commonly generates deprecated Compose APIs
@Composable
fun MyList(items: List<Item>) {
    // LazyColumnFor was removed in Compose 1.0
    LazyColumnFor(items) { item ->
        Text(item.name)
    }
}
```

```kotlin
// Current API
@Composable
fun MyList(items: List<Item>) {
    LazyColumn {
        items(items, key = { it.id }) { item ->
            Text(item.name)
        }
    }
}
```

**Common deprecated patterns AI generates:**
- `LazyColumnFor` / `LazyRowFor` (removed in Compose 1.0)
- `accompanist` libraries for functionality now in Compose (system UI controller, pager, etc.)
- `LiveData` instead of `StateFlow` in new ViewModels
- `@ExperimentalMaterialApi` for APIs that are now stable in Material3
- `rememberCoroutineScope` where `LaunchedEffect` is more appropriate
- `ViewModelProvider.Factory` instead of Hilt's `@HiltViewModel`

### Hallucinated Compose APIs

AI models sometimes invent APIs that do not exist.

```kotlin
// These APIs do not exist (AI hallucinations)
Modifier.shimmer()                    // Not a real Modifier
Modifier.borderRadius(12.dp)          // CSS leaking into Compose
Text(text = "Hello", bold = true)     // Not a real parameter
LazyColumn(reverseScroll = true)      // Not a real parameter
```

```kotlin
// Real equivalents
Modifier.clip(RoundedCornerShape(12.dp))
Text(text = "Hello", fontWeight = FontWeight.Bold)
LazyColumn(reverseLayout = true)
```

**Always verify:** If a Compose API looks unfamiliar, check the official documentation before approving.

### Wrong Hilt Annotations

AI frequently confuses Hilt and Dagger annotations or applies them to the wrong targets.

```kotlin
// Wrong: @Inject on a Composable parameter
@Composable
fun UserScreen(@Inject viewModel: UserViewModel) { /* ... */ }

// Wrong: Missing @HiltViewModel
class UserViewModel @Inject constructor(
    private val useCase: GetUserUseCase
) : ViewModel()

// Wrong: @Provides in the wrong place
@HiltViewModel
class UserViewModel @Provides constructor( // @Provides is for modules, not constructors
    private val useCase: GetUserUseCase
) : ViewModel()
```

```kotlin
// Correct Hilt usage
@HiltViewModel
class UserViewModel @Inject constructor(
    private val useCase: GetUserUseCase
) : ViewModel()

@Composable
fun UserScreen(viewModel: UserViewModel = hiltViewModel()) {
    // ...
}
```

### Incorrect Coroutine Patterns

AI models frequently generate coroutine code with subtle bugs.

```kotlin
// AI-generated: blocking call disguised as a coroutine
suspend fun loadData(): List<Item> {
    return api.getItems().execute().body()!! // Blocking call + !! operator
}

// AI-generated: launch inside a launch
viewModelScope.launch {
    launch { // Unnecessary nested launch
        val data = repository.getData()
        _uiState.value = UiState.Success(data)
    }
}
```

```kotlin
// Correct suspend function
suspend fun loadData(): List<Item> =
    withContext(ioDispatcher) {
        api.getItems() // Retrofit suspend function, non-blocking
    }

// Flat coroutine, no unnecessary nesting
viewModelScope.launch {
    val data = repository.getData()
    _uiState.value = UiState.Success(data)
}
```

### Missing Null Safety

AI models trained on Java-heavy codebases often produce Kotlin code that ignores null safety.

```kotlin
// AI-generated Java-style null handling
fun processUser(user: User?) {
    if (user != null) {
        if (user.address != null) {
            if (user.address.city != null) {
                displayCity(user.address.city)
            }
        }
    }
}
```

```kotlin
// Idiomatic Kotlin null safety
fun processUser(user: User?) {
    val city = user?.address?.city ?: return
    displayCity(city)
}
```

---

## 🚦 Review Severity Levels

Use severity prefixes in review comments so the author can prioritize responses.

### Blocking

**Must be fixed before merge.** The PR cannot be approved with blocking issues outstanding.

Examples:
- Clean Architecture violation (wrong layer dependency)
- Missing tests for business logic
- Security vulnerability (hardcoded secret, cleartext HTTP)
- Crash-prone code (`!!` on nullable external input)
- Data loss potential
- Memory leak (unscoped coroutine, unclosed resource)

**Comment format:**
> **blocking:** This ViewModel directly calls the Retrofit API, bypassing the repository layer. Business logic must go through a Use Case. See [Clean Architecture](../Architecture/clean-architecture.md).

### Major

**Should be fixed before merge** unless the author provides a strong justification with a follow-up ticket.

Examples:
- Missing error handling on a network call
- No accessibility labels on interactive elements
- Hardcoded strings in the UI
- Performance issue (missing `remember`, no LazyColumn keys)
- Test that doesn't actually assert behavior

**Comment format:**
> **major:** This `LazyColumn` is missing stable keys, which will cause full recomposition on every list update. Add `key = { it.id }`.

### Minor

**Should be fixed but won't block merge.** Can be addressed in a follow-up if the PR is time-sensitive.

Examples:
- Naming could be more descriptive
- Missing KDoc on a public function
- Slightly verbose code that could use an extension function
- Test naming doesn't follow convention

**Comment format:**
> **minor:** Consider renaming `doStuff()` to `syncUserProfile()` for clarity.

### Nit

**Optional style preference.** The author is free to take or leave it.

Examples:
- Import ordering preference beyond what ktlint enforces
- Blank line placement
- Alternative (equally valid) Kotlin idiom
- Trailing comma preference

**Comment format:**
> **nit:** I'd put a blank line before this `return` for readability, but it's fine either way.

---

## 🗣️ Review Response Etiquette

### Giving Feedback

- **Be specific.** Point to the exact line. Explain what is wrong and why.
- **Provide solutions.** Don't just say "this is wrong." Show what the correct code looks like.
- **Use questions for ambiguity.** "Did you consider using `sealed interface` here?" is better than "This should be a sealed interface" when you're not certain of the context.
- **Batch related comments.** If the same issue appears in five places, leave one detailed comment and reference it.
- **Separate blocking from optional.** Use the severity prefixes so the author knows what must change.
- **Acknowledge trade-offs.** If the author's approach has merits, say so even if you prefer a different approach.
- **Time-box your review.** If a PR is too large to review in 30 minutes, ask the author to split it.

### Receiving Feedback

- **Don't take it personally.** The review is about the code, not about you.
- **Respond to every comment.** Even if the response is "Done" or "Good point, fixed."
- **Explain your reasoning.** If you disagree with a suggestion, explain why. "I kept it this way because..." is a valid response.
- **Ask for clarification.** If a comment is unclear, ask. Don't guess at what the reviewer meant.
- **Say thank you.** When a reviewer catches a real bug or teaches you something, acknowledge it.

### Resolving Disagreements

1. Discuss in the PR comments with code examples.
2. If no resolution after two rounds, bring in a third engineer.
3. If it's a standards question, update the ARCKnowledge-Android docs to prevent future ambiguity.
4. The reviewer has the final call on blocking issues. The author has the final call on nits.

---

## ✅ Approval Flow

### When to Approve

- All **blocking** and **major** issues are resolved.
- All tests pass in CI.
- ktlint and detekt pass.
- You understand the change well enough to support it in production.

### When to Request Changes

- There are **blocking** issues that must be fixed.
- Tests are missing for new business logic.
- The PR introduces a security risk.
- The code would be difficult to maintain or extend.

### When to Comment (without approval or request-changes)

- You reviewed part of the PR but not all of it (e.g., you reviewed the domain layer but not the UI).
- You have questions that need answers before you can decide.
- You want to share context or suggestions but another reviewer is the primary approver.

### Approval Requirements by PR Type

| PR Type | Approvals | Notes |
|---|---|---|
| Feature | 1 | Standard review |
| Bug fix | 1 | Verify the fix addresses the root cause, not just symptoms |
| Hotfix | 1 | Expedited review, but still requires full checklist |
| Refactor | 1 | Extra scrutiny on test coverage for behavioral preservation |
| Critical module | 2 | Auth, payments, data persistence |
| Architecture change | 2 | Changes to base classes, DI setup, module structure |

---

## 🤖 Automated Checks

These checks run in CI before any human reviews the PR. If automated checks fail, the author must fix them before requesting review.

### What Automation Should Catch

| Tool | What It Catches |
|---|---|
| **ktlint** | Code formatting, import ordering, spacing |
| **detekt** | Code smells, complexity, naming conventions |
| **Android Lint** | Android-specific issues, deprecated API usage, accessibility |
| **Unit tests** | Behavioral regressions |
| **Build** | Compilation errors, unresolved dependencies |
| **Dependency check** | Vulnerable or outdated dependencies |

### What Automation Cannot Catch

These require human review:

- Architecture violations (layer boundaries)
- Business logic correctness
- Appropriate error handling
- UX quality and accessibility completeness
- Naming clarity and domain accuracy
- Test quality (tests that pass but don't meaningfully assert behavior)
- Performance characteristics under real-world data
- Security implications of design decisions

### Pre-Review Author Checklist

Before requesting review, the author should verify:

1. `./gradlew ktlintCheck` passes
2. `./gradlew detekt` passes
3. `./gradlew testDebugUnitTest` passes
4. `./gradlew lintDebug` passes (no new warnings)
5. Self-review of the diff is complete
6. PR description explains the _why_, not just the _what_
7. Screenshots or screen recordings attached for UI changes

---

## 🚫 Common Anti-Patterns Caught in Reviews

### 1. God ViewModel

```kotlin
// ViewModel doing everything: API calls, business logic, formatting, navigation
@HiltViewModel
class RestaurantViewModel @Inject constructor(
    private val api: RestaurantApi,
    private val locationManager: LocationManager,
    private val prefs: SharedPreferences
) : ViewModel() {

    fun loadRestaurants() {
        viewModelScope.launch {
            val location = locationManager.getLastKnownLocation()
            val response = api.getNearby(location.latitude, location.longitude)
            val restaurants = response.body()?.map { dto ->
                Restaurant(
                    name = dto.name,
                    distance = calculateDistance(location, dto.lat, dto.lng),
                    rating = String.format("%.1f", dto.rating),
                    priceRange = "$".repeat(dto.priceLevel)
                )
            } ?: emptyList()
            _uiState.value = UiState.Success(restaurants)
        }
    }
}
```

```kotlin
// Responsibilities split across layers
@HiltViewModel
class RestaurantViewModel @Inject constructor(
    private val getNearbyRestaurants: GetNearbyRestaurantsUseCase
) : ViewModel() {

    fun loadRestaurants() {
        viewModelScope.launch {
            getNearbyRestaurants()
                .onSuccess { restaurants ->
                    _uiState.value = UiState.Success(restaurants)
                }
                .onFailure { error ->
                    _uiState.value = UiState.Error(error.message.orEmpty())
                }
        }
    }
}
```

### 2. Leaky Abstractions

```kotlin
// Domain layer leaking Room entities
// domain/repository/NoteRepository.kt
interface NoteRepository {
    suspend fun getAll(): List<NoteEntity> // Room entity in domain!
}
```

```kotlin
// Domain layer uses domain models
// domain/model/Note.kt
data class Note(val id: String, val title: String, val content: String)

// domain/repository/NoteRepository.kt
interface NoteRepository {
    suspend fun getAll(): List<Note> // Domain model
}

// data/repository/NoteRepositoryImpl.kt
class NoteRepositoryImpl @Inject constructor(
    private val dao: NoteDao
) : NoteRepository {
    override suspend fun getAll(): List<Note> =
        dao.getAll().map { it.toDomain() } // Mapping at the boundary
}
```

### 3. Mutable State Exposure

```kotlin
// Exposing MutableStateFlow directly
@HiltViewModel
class CartViewModel @Inject constructor() : ViewModel() {
    val items = MutableStateFlow<List<CartItem>>(emptyList()) // External code can mutate state
}
```

```kotlin
// Expose immutable StateFlow
@HiltViewModel
class CartViewModel @Inject constructor() : ViewModel() {
    private val _items = MutableStateFlow<List<CartItem>>(emptyList())
    val items: StateFlow<List<CartItem>> = _items.asStateFlow()
}
```

### 4. Catching All Exceptions

```kotlin
// Swallowing all exceptions hides bugs
suspend fun loadData() {
    try {
        val data = repository.getData()
        _uiState.value = UiState.Success(data)
    } catch (e: Exception) { // Catches CancellationException, breaking structured concurrency
        _uiState.value = UiState.Error("Something went wrong")
    }
}
```

```kotlin
// Catch specific exceptions; rethrow CancellationException
suspend fun loadData() {
    try {
        val data = repository.getData()
        _uiState.value = UiState.Success(data)
    } catch (e: CancellationException) {
        throw e // Never swallow cancellation
    } catch (e: IOException) {
        _uiState.value = UiState.Error("Network error: ${e.message}")
    } catch (e: HttpException) {
        _uiState.value = UiState.Error("Server error: ${e.code()}")
    }
}
```

### 5. Composable With Too Many Parameters

```kotlin
// Too many parameters makes the composable fragile and hard to use
@Composable
fun RestaurantCard(
    name: String,
    address: String,
    city: String,
    state: String,
    zip: String,
    rating: Float,
    reviewCount: Int,
    priceLevel: Int,
    imageUrl: String,
    isFavorite: Boolean,
    isOpen: Boolean,
    distance: String,
    onFavoriteClick: () -> Unit,
    onClick: () -> Unit,
    modifier: Modifier = Modifier
) { /* ... */ }
```

```kotlin
// Group related data into a model; fewer, more meaningful parameters
data class RestaurantCardState(
    val name: String,
    val formattedAddress: String,
    val rating: Float,
    val reviewCount: Int,
    val priceLevel: Int,
    val imageUrl: String,
    val isFavorite: Boolean,
    val isOpen: Boolean,
    val distance: String
)

@Composable
fun RestaurantCard(
    state: RestaurantCardState,
    onFavoriteClick: () -> Unit,
    onClick: () -> Unit,
    modifier: Modifier = Modifier
) { /* ... */ }
```

### 6. Platform-Dependent Domain Layer

```kotlin
// Domain layer with Android dependencies
// domain/usecase/FormatDateUseCase.kt
import android.text.format.DateUtils // Android framework in domain!

class FormatDateUseCase {
    operator fun invoke(timestamp: Long): String {
        return DateUtils.getRelativeTimeSpanString(timestamp).toString()
    }
}
```

```kotlin
// Domain layer free of platform dependencies
// domain/usecase/FormatDateUseCase.kt
import java.time.Instant
import java.time.ZoneId
import java.time.format.DateTimeFormatter

class FormatDateUseCase @Inject constructor(
    private val dateFormatter: DateFormatter // Interface in domain
) {
    operator fun invoke(timestamp: Long): String {
        return dateFormatter.formatRelative(timestamp)
    }
}

// domain/formatter/DateFormatter.kt
interface DateFormatter {
    fun formatRelative(timestamp: Long): String
}

// data/formatter/AndroidDateFormatter.kt (in data layer)
class AndroidDateFormatter @Inject constructor(
    @ApplicationContext private val context: Context
) : DateFormatter {
    override fun formatRelative(timestamp: Long): String {
        return DateUtils.getRelativeTimeSpanString(timestamp).toString()
    }
}
```

### 7. Unkeyed `LazyColumn` Items With `MutableState`

```kotlin
// Without keys, removing an item causes state to "shift" to the wrong row
LazyColumn {
    items(tasks) { task ->
        var isChecked by remember { mutableStateOf(task.completed) }
        TaskRow(task = task, isChecked = isChecked, onCheckedChange = { isChecked = it })
    }
}
```

```kotlin
// Stable keys ensure state is associated with the correct item
LazyColumn {
    items(tasks, key = { it.id }) { task ->
        var isChecked by remember { mutableStateOf(task.completed) }
        TaskRow(task = task, isChecked = isChecked, onCheckedChange = { isChecked = it })
    }
}
```

---

## 📚 Further Reading

- [Testing Guide](./testing.md)
- [Code Style](./code-style.md)
- [Compose Performance](./compose-performance.md)
- [Documentation Standards](./documentation.md)
- [Module Structure](./module-structure.md)
- [README Standards](./readme-standards.md)
- [UI Guidelines](./ui-guidelines.md)
- [Clean Architecture](../Architecture/clean-architecture.md)
- [MVVM Pattern](../Architecture/mvvm.md)
- [SOLID Principles](../Architecture/solid-principles.md)
- [Dependency Injection](../Architecture/dependency-injection.md)
- [Domain Layer](../Layers/domain.md)
- [Presentation Layer](../Layers/presentation.md)
- [Data Layer](../Layers/data.md)
- [Git Commits](../Workflow/git-commits.md)
- [Git Branches](../Workflow/git-branches.md)

---

**Remember**: Code reviews are about making the code better, not about being right. Approach every review as a conversation between collaborators who share the same goal: shipping excellent software.
