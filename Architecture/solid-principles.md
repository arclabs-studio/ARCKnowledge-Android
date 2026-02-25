# 🧱 SOLID Principles

**A comprehensive guide to applying SOLID principles in Android development with Kotlin, Clean Architecture, and modern Jetpack libraries.**

---

> SOLID is not about writing more code. It is about writing code that is easier to understand,
> easier to change, and easier to test. In Android development, where requirements shift rapidly
> and codebases grow quickly, SOLID principles are the foundation that keeps your architecture
> from collapsing under its own weight.

---

## 🎯 Why SOLID Matters

Modern Android applications are complex systems composed of UI layers, business logic, networking,
local persistence, background processing, and platform integrations. Without a disciplined approach
to design, these systems become brittle, difficult to test, and expensive to maintain.

SOLID principles provide five foundational guidelines that, when applied consistently, produce
codebases with the following characteristics:

- **Low coupling** — Components can be changed independently without cascading failures.
- **High cohesion** — Each class has a clear, focused purpose.
- **Testability** — Business logic can be verified in isolation from the Android framework.
- **Extensibility** — New features can be added without modifying existing, stable code.
- **Readability** — Intent is clear because responsibilities are well-defined.

### Relevance to Android Clean Architecture

Clean Architecture organizes code into three layers — **Presentation**, **Domain**, and **Data**.
Each SOLID principle reinforces the boundaries between these layers:

| Principle | Clean Architecture Benefit |
|-----------|--------------------------|
| **S** — Single Responsibility | Each layer has one reason to change |
| **O** — Open/Closed | New features extend the system without modifying stable layers |
| **L** — Liskov Substitution | Repository implementations are interchangeable |
| **I** — Interface Segregation | Layers expose only the contracts their consumers need |
| **D** — Dependency Inversion | Domain never depends on Data or Presentation |

When SOLID principles are applied within Clean Architecture, the result is a codebase where the
Domain layer is a pure Kotlin module with zero Android dependencies, the Data layer can swap
between Room, Retrofit, or DataStore without touching business logic, and the Presentation layer
can migrate from XML Views to Jetpack Compose without rewriting a single UseCase.

---

## 📐 The Five Principles

---

### S — Single Responsibility Principle

> **A class should have one, and only one, reason to change.**

The Single Responsibility Principle (SRP) states that every class should encapsulate a single
piece of functionality. When a class has more than one responsibility, changes to one responsibility
risk breaking the other. In Android, this is the most commonly violated principle.

#### Why It Matters on Android

Android encourages violation of SRP by design. Activities, Fragments, and even ViewModels become
magnets for unrelated logic: navigation, data fetching, validation, formatting, analytics tracking,
and permission handling all tend to accumulate in the same class.

#### ❌ Violation: The God ViewModel

This ViewModel does far too many things. It fetches data from the network, validates user input,
formats display strings, tracks analytics events, and manages navigation. Any change to the date
formatting logic, for example, risks breaking the network call or the validation rules.

```kotlin
// ❌ BAD — This ViewModel has at least five reasons to change
class UserProfileViewModel(
    private val retrofit: Retrofit,
    private val analyticsTracker: AnalyticsTracker,
    private val context: Context,
) : ViewModel() {

    var userName by mutableStateOf("")
    var userEmail by mutableStateOf("")
    var errorMessage by mutableStateOf<String?>(null)
    var isLoading by mutableStateOf(false)
    var formattedJoinDate by mutableStateOf("")

    // Responsibility 1: Network call
    fun loadProfile(userId: String) {
        viewModelScope.launch {
            isLoading = true
            try {
                val api = retrofit.create(UserApi::class.java)
                val response = api.getUser(userId)
                userName = response.name
                userEmail = response.email
                formattedJoinDate = formatDate(response.joinedAt)
                analyticsTracker.track("profile_loaded", mapOf("userId" to userId))
            } catch (e: Exception) {
                errorMessage = e.message
            } finally {
                isLoading = false
            }
        }
    }

    // Responsibility 2: Input validation
    fun validateEmail(email: String): Boolean {
        val pattern = Patterns.EMAIL_ADDRESS
        return pattern.matcher(email).matches()
    }

    // Responsibility 3: Date formatting
    private fun formatDate(timestamp: Long): String {
        val sdf = SimpleDateFormat("MMM dd, yyyy", Locale.getDefault())
        return sdf.format(Date(timestamp))
    }

    // Responsibility 4: Analytics
    fun trackScreenView() {
        analyticsTracker.track("profile_screen_viewed")
    }

    // Responsibility 5: Navigation
    fun onEditProfileClicked() {
        // navigate somewhere...
    }
}
```

#### ✅ Correct: Separated Responsibilities

Each responsibility is extracted into its own class. The ViewModel orchestrates them but does not
implement any of the details. Each class has exactly one reason to change.

```kotlin
// ✅ GOOD — UseCase handles the business logic of fetching a profile
class GetUserProfileUseCase(
    private val userRepository: UserRepository,
) {
    suspend operator fun invoke(userId: String): Result<UserProfile> {
        return userRepository.getUserProfile(userId)
    }
}
```

```kotlin
// ✅ GOOD — Formatter handles only display formatting
class DateFormatter {
    fun format(timestamp: Long): String {
        val sdf = SimpleDateFormat("MMM dd, yyyy", Locale.getDefault())
        return sdf.format(Date(timestamp))
    }
}
```

```kotlin
// ✅ GOOD — Validator handles only input validation
class EmailValidator {
    fun isValid(email: String): Boolean {
        return Patterns.EMAIL_ADDRESS.matcher(email).matches()
    }
}
```

```kotlin
// ✅ GOOD — Analytics tracker is its own concern
interface AnalyticsTracker {
    fun track(event: String, properties: Map<String, Any> = emptyMap())
}
```

```kotlin
// ✅ GOOD — ViewModel only orchestrates, does not implement details
class UserProfileViewModel(
    private val getUserProfile: GetUserProfileUseCase,
    private val dateFormatter: DateFormatter,
    private val emailValidator: EmailValidator,
    private val analyticsTracker: AnalyticsTracker,
) : ViewModel() {

    var uiState by mutableStateOf<UserProfileUiState>(UserProfileUiState.Idle)
        private set

    fun loadProfile(userId: String) {
        viewModelScope.launch {
            uiState = UserProfileUiState.Loading
            getUserProfile(userId)
                .onSuccess { profile ->
                    uiState = UserProfileUiState.Success(
                        name = profile.name,
                        email = profile.email,
                        formattedJoinDate = dateFormatter.format(profile.joinedAt),
                    )
                    analyticsTracker.track("profile_loaded")
                }
                .onFailure { error ->
                    uiState = UserProfileUiState.Error(error.message.orEmpty())
                }
        }
    }

    fun isEmailValid(email: String): Boolean {
        return emailValidator.isValid(email)
    }
}
```

```kotlin
// ✅ GOOD — UI state is a sealed interface with clear states
sealed interface UserProfileUiState {
    data object Idle : UserProfileUiState
    data object Loading : UserProfileUiState
    data class Success(
        val name: String,
        val email: String,
        val formattedJoinDate: String,
    ) : UserProfileUiState
    data class Error(val message: String) : UserProfileUiState
}
```

#### SRP Checklist

- [ ] Can you describe the class's purpose in one sentence without using "and"?
- [ ] Does the class have only one reason to change?
- [ ] If you renamed the class to describe exactly what it does, would the name still be short?

---

### O — Open/Closed Principle

> **Software entities should be open for extension, but closed for modification.**

The Open/Closed Principle (OCP) states that you should be able to add new behavior to a system
without changing existing, tested code. In Kotlin, sealed interfaces and polymorphism are the
primary tools for achieving this.

#### Why It Matters on Android

Android apps frequently need to support new types: new payment methods, new notification channels,
new analytics providers, new list item types. Without OCP, adding a new type means modifying a
`when` block or `if-else` chain that is already long and fragile.

#### ❌ Violation: Giant When Block

Every time a new payment method is added, the `processPayment` function must be modified. This
violates OCP because the existing, tested function is changed every time a new requirement appears.

```kotlin
// ❌ BAD — Must modify this function every time a new payment method is added
class PaymentProcessor {

    fun processPayment(type: String, amount: Double): PaymentResult {
        return when (type) {
            "credit_card" -> {
                // 30 lines of credit card logic
                val gateway = CreditCardGateway()
                gateway.charge(amount)
                PaymentResult.Success("Credit card charged")
            }
            "paypal" -> {
                // 25 lines of PayPal logic
                val paypal = PayPalClient()
                paypal.executePayment(amount)
                PaymentResult.Success("PayPal payment executed")
            }
            "crypto" -> {
                // 20 lines of crypto logic
                val wallet = CryptoWallet()
                wallet.transfer(amount)
                PaymentResult.Success("Crypto transfer completed")
            }
            // Every new payment method requires modifying this function
            else -> PaymentResult.Failure("Unknown payment type: $type")
        }
    }
}
```

#### ✅ Correct: Sealed Interface + Polymorphism

Each payment method is a self-contained implementation of a common interface. Adding a new payment
method means creating a new class — the existing code is never modified.

```kotlin
// ✅ GOOD — Define the contract
sealed interface PaymentMethod {
    val name: String
    suspend fun process(amount: Double): PaymentResult
}
```

```kotlin
// ✅ GOOD — Each payment method is a separate, self-contained class
class CreditCardPayment(
    private val gateway: CreditCardGateway,
) : PaymentMethod {

    override val name: String = "Credit Card"

    override suspend fun process(amount: Double): PaymentResult {
        return try {
            gateway.charge(amount)
            PaymentResult.Success("Credit card charged: $$amount")
        } catch (e: Exception) {
            PaymentResult.Failure("Credit card failed: ${e.message}")
        }
    }
}

class PayPalPayment(
    private val client: PayPalClient,
) : PaymentMethod {

    override val name: String = "PayPal"

    override suspend fun process(amount: Double): PaymentResult {
        return try {
            client.executePayment(amount)
            PaymentResult.Success("PayPal payment: $$amount")
        } catch (e: Exception) {
            PaymentResult.Failure("PayPal failed: ${e.message}")
        }
    }
}

class CryptoPayment(
    private val wallet: CryptoWallet,
) : PaymentMethod {

    override val name: String = "Cryptocurrency"

    override suspend fun process(amount: Double): PaymentResult {
        return try {
            wallet.transfer(amount)
            PaymentResult.Success("Crypto transfer: $$amount")
        } catch (e: Exception) {
            PaymentResult.Failure("Crypto failed: ${e.message}")
        }
    }
}
```

```kotlin
// ✅ GOOD — Processor is closed for modification, open for extension
class PaymentProcessor(
    private val methods: Map<String, PaymentMethod>,
) {
    suspend fun process(methodKey: String, amount: Double): PaymentResult {
        val method = methods[methodKey]
            ?: return PaymentResult.Failure("Unknown payment method: $methodKey")
        return method.process(amount)
    }
}
```

```kotlin
// ✅ GOOD — Adding a new payment method requires zero changes to existing code
class GooglePayPayment(
    private val googlePayClient: GooglePayClient,
) : PaymentMethod {

    override val name: String = "Google Pay"

    override suspend fun process(amount: Double): PaymentResult {
        return try {
            googlePayClient.pay(amount)
            PaymentResult.Success("Google Pay: $$amount")
        } catch (e: Exception) {
            PaymentResult.Failure("Google Pay failed: ${e.message}")
        }
    }
}
```

```kotlin
// ✅ GOOD — Hilt wiring adds the new method without modifying any existing module
@Module
@InstallIn(SingletonComponent::class)
object PaymentModule {

    @Provides
    fun providePaymentMethods(
        creditCard: CreditCardPayment,
        payPal: PayPalPayment,
        crypto: CryptoPayment,
        googlePay: GooglePayPayment,
    ): Map<String, PaymentMethod> = mapOf(
        "credit_card" to creditCard,
        "paypal" to payPal,
        "crypto" to crypto,
        "google_pay" to googlePay,
    )
}
```

```kotlin
// ✅ GOOD — Result sealed interface
sealed interface PaymentResult {
    data class Success(val message: String) : PaymentResult
    data class Failure(val reason: String) : PaymentResult
}
```

#### OCP Checklist

- [ ] Can you add a new variant without modifying existing classes?
- [ ] Are `when` blocks exhaustive over sealed types (compiler-enforced)?
- [ ] Is polymorphism preferred over conditional branching?

---

### L — Liskov Substitution Principle

> **Subtypes must be substitutable for their base types without altering the correctness of the program.**

The Liskov Substitution Principle (LSP) states that if class `B` is a subtype of class `A`, then
objects of type `A` can be replaced with objects of type `B` without breaking the program. In
Kotlin, this applies to interface implementations and class inheritance.

#### Why It Matters on Android

Android developers frequently define interfaces for repositories, data sources, and services. If
an implementation of one of these interfaces behaves differently than what the contract promises —
for example, by throwing an unexpected exception or returning an invalid state — the calling code
breaks in subtle, hard-to-debug ways.

#### ❌ Violation: Implementation That Breaks the Contract

The `ReadOnlyCache` implementation violates LSP because it throws an exception for a method that
the interface promises will work. Any code that uses `Cache` and calls `put()` will crash when
given a `ReadOnlyCache`.

```kotlin
// ❌ BAD — Interface promises both read and write
interface Cache<K, V> {
    fun get(key: K): V?
    fun put(key: K, value: V)
    fun remove(key: K)
    fun clear()
}

// ❌ BAD — This implementation violates the contract by throwing on put()
class ReadOnlyCache<K, V>(
    private val data: Map<K, V>,
) : Cache<K, V> {

    override fun get(key: K): V? = data[key]

    override fun put(key: K, value: V) {
        // Violates LSP: callers expect put() to work
        throw UnsupportedOperationException("This cache is read-only")
    }

    override fun remove(key: K) {
        throw UnsupportedOperationException("This cache is read-only")
    }

    override fun clear() {
        throw UnsupportedOperationException("This cache is read-only")
    }
}
```

```kotlin
// ❌ BAD — This function assumes Cache contract is honored
fun warmUpCache(cache: Cache<String, User>, users: List<User>) {
    users.forEach { user ->
        cache.put(user.id, user) // CRASH if ReadOnlyCache is passed
    }
}
```

#### ✅ Correct: Proper Interface Segregation Preserves LSP

Split the interface so that read-only consumers get a read-only contract, and read-write consumers
get a read-write contract. No implementation needs to throw `UnsupportedOperationException`.

```kotlin
// ✅ GOOD — Read-only contract
interface ReadableCache<K, V> {
    fun get(key: K): V?
    fun contains(key: K): Boolean
}

// ✅ GOOD — Write contract extends read contract
interface WritableCache<K, V> : ReadableCache<K, V> {
    fun put(key: K, value: V)
    fun remove(key: K)
    fun clear()
}
```

```kotlin
// ✅ GOOD — Full cache honors the complete contract
class InMemoryCache<K, V> : WritableCache<K, V> {

    private val store = mutableMapOf<K, V>()

    override fun get(key: K): V? = store[key]
    override fun contains(key: K): Boolean = store.containsKey(key)
    override fun put(key: K, value: V) { store[key] = value }
    override fun remove(key: K) { store.remove(key) }
    override fun clear() { store.clear() }
}
```

```kotlin
// ✅ GOOD — Snapshot cache honors the read-only contract perfectly
class SnapshotCache<K, V>(
    private val data: Map<K, V>,
) : ReadableCache<K, V> {

    override fun get(key: K): V? = data[key]
    override fun contains(key: K): Boolean = data.containsKey(key)
}
```

```kotlin
// ✅ GOOD — Consumer asks for exactly what it needs
fun displayCachedUsers(cache: ReadableCache<String, User>) {
    // This works with both InMemoryCache and SnapshotCache
    val user = cache.get("user-123")
    user?.let { println(it.name) }
}

fun warmUpCache(cache: WritableCache<String, User>, users: List<User>) {
    // This only accepts writable caches — SnapshotCache cannot be passed here
    users.forEach { user ->
        cache.put(user.id, user)
    }
}
```

#### LSP Checklist

- [ ] Can every implementation be used wherever the interface is expected?
- [ ] Does any implementation throw `UnsupportedOperationException`? (If yes, LSP is violated.)
- [ ] Do all implementations honor the same postconditions and invariants?

---

### I — Interface Segregation Principle

> **No client should be forced to depend on methods it does not use.**

The Interface Segregation Principle (ISP) states that large, monolithic interfaces should be split
into smaller, more focused ones. Clients should only know about the methods that are relevant to
them.

#### Why It Matters on Android

In Android, repositories are the most common victim of ISP violations. A single `UserRepository`
interface often grows to include methods for fetching, creating, updating, deleting, searching,
and batch operations. A ViewModel that only needs to display a user profile should not depend on
an interface that also includes `deleteUser()` and `batchImport()`.

#### ❌ Violation: Fat Interface

This single interface forces every consumer to depend on all 10 methods, even if they only need
one or two.

```kotlin
// ❌ BAD — Fat interface with too many responsibilities
interface UserRepository {
    suspend fun getUserById(id: String): User?
    suspend fun getUsersByName(name: String): List<User>
    suspend fun getAllUsers(): List<User>
    suspend fun searchUsers(query: String, filters: UserFilters): List<User>
    suspend fun createUser(user: User): User
    suspend fun updateUser(user: User): User
    suspend fun deleteUser(id: String)
    suspend fun batchImportUsers(users: List<User>): BatchResult
    suspend fun exportUsers(format: ExportFormat): ByteArray
    suspend fun getUserStatistics(): UserStatistics
}
```

```kotlin
// ❌ BAD — This ViewModel only needs getUserById but depends on ALL 10 methods
class UserDetailViewModel(
    private val userRepository: UserRepository, // depends on delete, export, batch, etc.
) : ViewModel() {

    fun loadUser(userId: String) {
        viewModelScope.launch {
            val user = userRepository.getUserById(userId)
            // ...
        }
    }
}
```

```kotlin
// ❌ BAD — Test must mock all 10 methods even though only 1 is used
@Test
fun `should load user by id`() = runTest {
    val mockRepo = mockk<UserRepository>()
    // Must set up stubs for every method to avoid crashes
    coEvery { mockRepo.getUserById("123") } returns testUser
    coEvery { mockRepo.getUsersByName(any()) } returns emptyList()
    coEvery { mockRepo.getAllUsers() } returns emptyList()
    coEvery { mockRepo.searchUsers(any(), any()) } returns emptyList()
    coEvery { mockRepo.createUser(any()) } returns testUser
    coEvery { mockRepo.updateUser(any()) } returns testUser
    coEvery { mockRepo.deleteUser(any()) } just Runs
    coEvery { mockRepo.batchImportUsers(any()) } returns BatchResult.empty()
    coEvery { mockRepo.exportUsers(any()) } returns byteArrayOf()
    coEvery { mockRepo.getUserStatistics() } returns UserStatistics.empty()
    // ... actual test
}
```

#### ✅ Correct: Focused Interfaces

Split the fat interface into small, cohesive contracts. Each consumer depends only on the methods
it actually uses.

```kotlin
// ✅ GOOD — Read operations
interface ReadableUserRepository {
    suspend fun getUserById(id: String): User?
    suspend fun getUsersByName(name: String): List<User>
    suspend fun getAllUsers(): List<User>
}
```

```kotlin
// ✅ GOOD — Search operations
interface SearchableUserRepository {
    suspend fun searchUsers(query: String, filters: UserFilters): List<User>
}
```

```kotlin
// ✅ GOOD — Write operations
interface WritableUserRepository {
    suspend fun createUser(user: User): User
    suspend fun updateUser(user: User): User
    suspend fun deleteUser(id: String)
}
```

```kotlin
// ✅ GOOD — Batch and export operations (admin features)
interface AdminUserRepository {
    suspend fun batchImportUsers(users: List<User>): BatchResult
    suspend fun exportUsers(format: ExportFormat): ByteArray
    suspend fun getUserStatistics(): UserStatistics
}
```

```kotlin
// ✅ GOOD — Implementation can implement all interfaces
class UserRepositoryImpl(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource,
) : ReadableUserRepository,
    SearchableUserRepository,
    WritableUserRepository,
    AdminUserRepository {

    override suspend fun getUserById(id: String): User? {
        return localDataSource.getUserById(id)
            ?: remoteDataSource.getUserById(id)?.also {
                localDataSource.insertUser(it)
            }
    }

    override suspend fun getUsersByName(name: String): List<User> {
        return localDataSource.getUsersByName(name)
    }

    override suspend fun getAllUsers(): List<User> {
        return localDataSource.getAllUsers()
    }

    override suspend fun searchUsers(query: String, filters: UserFilters): List<User> {
        return remoteDataSource.searchUsers(query, filters)
    }

    override suspend fun createUser(user: User): User {
        val created = remoteDataSource.createUser(user)
        localDataSource.insertUser(created)
        return created
    }

    override suspend fun updateUser(user: User): User {
        val updated = remoteDataSource.updateUser(user)
        localDataSource.updateUser(updated)
        return updated
    }

    override suspend fun deleteUser(id: String) {
        remoteDataSource.deleteUser(id)
        localDataSource.deleteUser(id)
    }

    override suspend fun batchImportUsers(users: List<User>): BatchResult {
        return remoteDataSource.batchImport(users)
    }

    override suspend fun exportUsers(format: ExportFormat): ByteArray {
        return remoteDataSource.export(format)
    }

    override suspend fun getUserStatistics(): UserStatistics {
        return remoteDataSource.getStatistics()
    }
}
```

```kotlin
// ✅ GOOD — ViewModel depends only on what it needs
class UserDetailViewModel(
    private val userRepository: ReadableUserRepository,
) : ViewModel() {

    var uiState by mutableStateOf<UserDetailUiState>(UserDetailUiState.Idle)
        private set

    fun loadUser(userId: String) {
        viewModelScope.launch {
            uiState = UserDetailUiState.Loading
            val user = userRepository.getUserById(userId)
            uiState = if (user != null) {
                UserDetailUiState.Success(user)
            } else {
                UserDetailUiState.Error("User not found")
            }
        }
    }
}
```

```kotlin
// ✅ GOOD — Test only mocks what is needed
@Test
fun `should load user by id`() = runTest {
    val mockRepo = mockk<ReadableUserRepository>()
    coEvery { mockRepo.getUserById("123") } returns testUser

    val viewModel = UserDetailViewModel(mockRepo)
    viewModel.loadUser("123")

    assertThat(viewModel.uiState).isEqualTo(
        UserDetailUiState.Success(testUser)
    )
}
```

#### ISP Checklist

- [ ] Does any consumer depend on methods it never calls?
- [ ] Can you describe each interface's purpose in one phrase?
- [ ] Are interfaces named by their capability (e.g., `Readable`, `Writable`, `Searchable`)?

---

### D — Dependency Inversion Principle

> **High-level modules should not depend on low-level modules. Both should depend on abstractions.**

The Dependency Inversion Principle (DIP) states that the direction of dependency should be toward
abstractions, not implementations. In Clean Architecture, this means the Domain layer defines
interfaces (abstractions), and the Data layer provides implementations.

#### Why It Matters on Android

Without DIP, ViewModels depend directly on Retrofit services, Room DAOs, and SharedPreferences.
This makes the presentation layer impossible to test without the Android framework and impossible
to change without ripple effects throughout the codebase.

#### ❌ Violation: Direct Dependency on Implementation

The ViewModel creates and depends on concrete classes. Testing requires a real database and a real
network connection.

```kotlin
// ❌ BAD — ViewModel depends directly on concrete implementations
class RestaurantListViewModel : ViewModel() {

    // Direct dependency on Retrofit — cannot test without network
    private val api = Retrofit.Builder()
        .baseUrl("https://api.example.com/")
        .addConverterFactory(GsonConverterFactory.create())
        .build()
        .create(RestaurantApi::class.java)

    // Direct dependency on Room — cannot test without Android context
    private val database = Room.databaseBuilder(
        MyApp.instance,
        AppDatabase::class.java,
        "restaurants-db",
    ).build()

    var restaurants by mutableStateOf<List<Restaurant>>(emptyList())
        private set

    fun loadRestaurants() {
        viewModelScope.launch {
            try {
                val response = api.getRestaurants()
                database.restaurantDao().insertAll(response)
                restaurants = database.restaurantDao().getAll()
            } catch (e: Exception) {
                restaurants = database.restaurantDao().getAll()
            }
        }
    }
}
```

#### ✅ Correct: Depend on Abstractions, Inject Implementations

The Domain layer defines the interface. The Data layer implements it. The Presentation layer
depends on the abstraction. Hilt wires everything together.

**Step 1: Domain layer defines the abstraction**

```kotlin
// ✅ GOOD — Domain layer: pure Kotlin, no Android dependencies
// domain/repository/RestaurantRepository.kt
interface RestaurantRepository {
    suspend fun getRestaurants(): Result<List<Restaurant>>
    suspend fun getRestaurantById(id: String): Result<Restaurant>
    fun observeRestaurants(): Flow<List<Restaurant>>
}
```

```kotlin
// ✅ GOOD — Domain layer: entity
// domain/model/Restaurant.kt
data class Restaurant(
    val id: String,
    val name: String,
    val cuisine: String,
    val rating: Double,
    val address: String,
)
```

```kotlin
// ✅ GOOD — Domain layer: use case
// domain/usecase/GetRestaurantsUseCase.kt
class GetRestaurantsUseCase(
    private val repository: RestaurantRepository,
) {
    suspend operator fun invoke(): Result<List<Restaurant>> {
        return repository.getRestaurants()
    }
}
```

**Step 2: Data layer implements the abstraction**

```kotlin
// ✅ GOOD — Data layer: implementation with caching strategy
// data/repository/RestaurantRepositoryImpl.kt
class RestaurantRepositoryImpl @Inject constructor(
    private val remoteDataSource: RestaurantRemoteDataSource,
    private val localDataSource: RestaurantLocalDataSource,
) : RestaurantRepository {

    override suspend fun getRestaurants(): Result<List<Restaurant>> {
        return try {
            val remote = remoteDataSource.fetchRestaurants()
            localDataSource.saveRestaurants(remote)
            Result.success(remote)
        } catch (e: Exception) {
            val cached = localDataSource.getRestaurants()
            if (cached.isNotEmpty()) {
                Result.success(cached)
            } else {
                Result.failure(e)
            }
        }
    }

    override suspend fun getRestaurantById(id: String): Result<Restaurant> {
        return try {
            val restaurant = remoteDataSource.fetchRestaurantById(id)
            localDataSource.saveRestaurant(restaurant)
            Result.success(restaurant)
        } catch (e: Exception) {
            localDataSource.getRestaurantById(id)
                ?.let { Result.success(it) }
                ?: Result.failure(e)
        }
    }

    override fun observeRestaurants(): Flow<List<Restaurant>> {
        return localDataSource.observeRestaurants()
    }
}
```

**Step 3: Hilt module binds implementation to abstraction**

```kotlin
// ✅ GOOD — Hilt module: the ONLY place where concrete meets abstract
// di/RepositoryModule.kt
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    abstract fun bindRestaurantRepository(
        impl: RestaurantRepositoryImpl,
    ): RestaurantRepository
}
```

**Step 4: Presentation layer depends on abstraction via UseCase**

```kotlin
// ✅ GOOD — ViewModel depends on UseCase (which depends on abstraction)
// presentation/viewmodel/RestaurantListViewModel.kt
@HiltViewModel
class RestaurantListViewModel @Inject constructor(
    private val getRestaurants: GetRestaurantsUseCase,
) : ViewModel() {

    var uiState by mutableStateOf<RestaurantListUiState>(RestaurantListUiState.Idle)
        private set

    fun loadRestaurants() {
        viewModelScope.launch {
            uiState = RestaurantListUiState.Loading
            getRestaurants()
                .onSuccess { restaurants ->
                    uiState = RestaurantListUiState.Success(restaurants)
                }
                .onFailure { error ->
                    uiState = RestaurantListUiState.Error(error.message.orEmpty())
                }
        }
    }
}
```

```kotlin
// ✅ GOOD — UI state sealed interface
sealed interface RestaurantListUiState {
    data object Idle : RestaurantListUiState
    data object Loading : RestaurantListUiState
    data class Success(val restaurants: List<Restaurant>) : RestaurantListUiState
    data class Error(val message: String) : RestaurantListUiState
}
```

#### DIP Checklist

- [ ] Does the Domain layer have zero dependencies on Data or Presentation?
- [ ] Are all repository contracts defined as interfaces in the Domain layer?
- [ ] Does Hilt (or your DI framework) bind implementations to abstractions?
- [ ] Can you swap implementations (e.g., mock, fake, in-memory) without changing any business logic?

---

## 🏗️ SOLID in Clean Architecture

Each SOLID principle maps naturally to the layered structure of Clean Architecture. The table below
shows where each principle has its greatest impact and provides a concrete example.

| Principle | Primary Layer | How It Manifests | Example |
|-----------|--------------|------------------|---------|
| **SRP** | Presentation | ViewModels orchestrate, do not implement | `ViewModel` calls `UseCase`, not `Retrofit` |
| **SRP** | Domain | Each UseCase has one job | `GetUserUseCase`, `UpdateUserUseCase` |
| **SRP** | Data | Data sources separated from repositories | `RemoteDataSource` vs `LocalDataSource` |
| **OCP** | Domain | New use cases do not modify existing ones | Add `SearchUsersUseCase` without touching `GetUsersUseCase` |
| **OCP** | Data | New data sources extend, not modify | Add `FirebaseDataSource` alongside `RetrofitDataSource` |
| **LSP** | Data | All repository implementations honor the contract | `FakeRepository` in tests behaves like `RealRepository` |
| **ISP** | Domain | Interfaces are focused on consumer needs | `ReadableRepo` vs `WritableRepo` |
| **DIP** | All | Domain defines interfaces, Data implements them | `RestaurantRepository` interface in domain, `RestaurantRepositoryImpl` in data |

### Dependency Flow

```
┌─────────────────────────────────────────────────────┐
│                  Presentation Layer                  │
│                                                     │
│   ViewModel ──depends on──► UseCase (interface)     │
│                                                     │
└────────────────────────┬────────────────────────────┘
                         │ depends on
                         ▼
┌─────────────────────────────────────────────────────┐
│                    Domain Layer                      │
│                                                     │
│   UseCase ──depends on──► Repository (interface)    │
│   Entity (pure data)                                │
│                                                     │
└────────────────────────┬────────────────────────────┘
                         │ implements
                         ▼
┌─────────────────────────────────────────────────────┐
│                     Data Layer                       │
│                                                     │
│   RepositoryImpl ──uses──► DataSource               │
│   DTO ──maps to──► Entity                           │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 🧪 SOLID Enables Testing

SOLID principles are not academic exercises. Their most practical benefit is **testability**.
When code follows SOLID, writing tests becomes straightforward and even enjoyable.

### How ISP Makes Mocking Easier

When interfaces are small and focused, mock setup is minimal. You only stub the methods your
test actually exercises.

```kotlin
// ✅ GOOD — Small interface = small mock
interface ReadableUserRepository {
    suspend fun getUserById(id: String): User?
}

@Test
fun `should display user name when user is found`() = runTest {
    // Arrange — only one method to mock
    val repo = mockk<ReadableUserRepository>()
    coEvery { repo.getUserById("123") } returns User(id = "123", name = "Alice")

    val viewModel = UserDetailViewModel(repo)

    // Act
    viewModel.loadUser("123")

    // Assert
    val state = viewModel.uiState as UserDetailUiState.Success
    assertThat(state.user.name).isEqualTo("Alice")
}
```

### How DIP Enables Test Doubles

Because the Domain layer defines interfaces, tests can use fakes, mocks, or stubs without any
knowledge of the real implementation.

```kotlin
// ✅ GOOD — Fake implementation for testing
class FakeRestaurantRepository : RestaurantRepository {

    private val restaurants = mutableListOf<Restaurant>()
    var shouldFail = false

    fun addRestaurant(restaurant: Restaurant) {
        restaurants.add(restaurant)
    }

    override suspend fun getRestaurants(): Result<List<Restaurant>> {
        return if (shouldFail) {
            Result.failure(IOException("Network error"))
        } else {
            Result.success(restaurants.toList())
        }
    }

    override suspend fun getRestaurantById(id: String): Result<Restaurant> {
        val found = restaurants.find { it.id == id }
        return if (found != null) {
            Result.success(found)
        } else {
            Result.failure(NoSuchElementException("Not found"))
        }
    }

    override fun observeRestaurants(): Flow<List<Restaurant>> {
        return flowOf(restaurants.toList())
    }
}
```

```kotlin
// ✅ GOOD — Test using the fake
class GetRestaurantsUseCaseTest {

    private val fakeRepository = FakeRestaurantRepository()
    private val useCase = GetRestaurantsUseCase(fakeRepository)

    @Test
    fun `should return restaurants when repository succeeds`() = runTest {
        // Given
        val sushi = Restaurant("1", "Sushi Place", "Japanese", 4.5, "123 Main St")
        val pizza = Restaurant("2", "Pizza World", "Italian", 4.2, "456 Oak Ave")
        fakeRepository.addRestaurant(sushi)
        fakeRepository.addRestaurant(pizza)

        // When
        val result = useCase()

        // Then
        assertThat(result.isSuccess).isTrue()
        assertThat(result.getOrThrow()).containsExactly(sushi, pizza)
    }

    @Test
    fun `should return failure when repository fails`() = runTest {
        // Given
        fakeRepository.shouldFail = true

        // When
        val result = useCase()

        // Then
        assertThat(result.isFailure).isTrue()
        assertThat(result.exceptionOrNull()).isInstanceOf(IOException::class.java)
    }
}
```

### How SRP Makes Tests Focused

When each class has one responsibility, each test class has one focus. Tests are short, readable,
and easy to maintain.

```kotlin
// ✅ GOOD — Testing the DateFormatter in isolation
class DateFormatterTest {

    private val formatter = DateFormatter()

    @Test
    fun `should format timestamp as readable date`() {
        // Given
        val timestamp = 1672531200000L // Jan 1, 2023

        // When
        val result = formatter.format(timestamp)

        // Then
        assertThat(result).isEqualTo("Jan 01, 2023")
    }

    @Test
    fun `should handle zero timestamp`() {
        // Given
        val timestamp = 0L

        // When
        val result = formatter.format(timestamp)

        // Then
        assertThat(result).isEqualTo("Jan 01, 1970")
    }
}
```

```kotlin
// ✅ GOOD — Testing the EmailValidator in isolation
class EmailValidatorTest {

    private val validator = EmailValidator()

    @Test
    fun `should accept valid email`() {
        assertThat(validator.isValid("alice@example.com")).isTrue()
    }

    @Test
    fun `should reject email without at symbol`() {
        assertThat(validator.isValid("alice-example.com")).isFalse()
    }

    @Test
    fun `should reject empty string`() {
        assertThat(validator.isValid("")).isFalse()
    }
}
```

---

## ✅ SOLID Checklist

Use this checklist during code reviews and when designing new features.

### Single Responsibility Principle

- [ ] Can the class be described in one sentence without "and"?
- [ ] Does the class have only one reason to change?
- [ ] Is the class under 200 lines?
- [ ] Are formatting, validation, and networking in separate classes?

### Open/Closed Principle

- [ ] Can a new variant be added without modifying existing code?
- [ ] Are `when` expressions exhaustive over sealed types?
- [ ] Is behavior selected by polymorphism, not conditionals?

### Liskov Substitution Principle

- [ ] Can every implementation replace the interface without errors?
- [ ] Does no implementation throw `UnsupportedOperationException`?
- [ ] Do all implementations honor the same preconditions and postconditions?

### Interface Segregation Principle

- [ ] Does any consumer depend on methods it never calls?
- [ ] Can each interface be named with a single adjective (Readable, Writable, Searchable)?
- [ ] Are test mocks easy to set up (few methods to stub)?

### Dependency Inversion Principle

- [ ] Does the Domain layer have zero Android or framework imports?
- [ ] Are all cross-layer dependencies expressed as interfaces?
- [ ] Does the DI framework (Hilt) bind all implementations to abstractions?
- [ ] Can implementations be swapped for testing without changing business logic?

---

## 🚫 Common Violations

### Violation 1: ViewModel That Formats UI Strings

```kotlin
// ❌ BAD — ViewModel formats strings (SRP violation)
class OrderViewModel(
    private val getOrder: GetOrderUseCase,
) : ViewModel() {

    var formattedTotal by mutableStateOf("")
        private set

    fun loadOrder(orderId: String) {
        viewModelScope.launch {
            val order = getOrder(orderId).getOrNull() ?: return@launch
            formattedTotal = "$${String.format("%.2f", order.totalCents / 100.0)}"
        }
    }
}
```

```kotlin
// ✅ GOOD — Formatter is a separate class
class CurrencyFormatter {
    fun formatCents(cents: Long): String {
        return "$${String.format("%.2f", cents / 100.0)}"
    }
}

class OrderViewModel(
    private val getOrder: GetOrderUseCase,
    private val currencyFormatter: CurrencyFormatter,
) : ViewModel() {

    var formattedTotal by mutableStateOf("")
        private set

    fun loadOrder(orderId: String) {
        viewModelScope.launch {
            val order = getOrder(orderId).getOrNull() ?: return@launch
            formattedTotal = currencyFormatter.formatCents(order.totalCents)
        }
    }
}
```

---

### Violation 2: Adding Behavior With Type Checks Instead of Polymorphism

```kotlin
// ❌ BAD — Type checking instead of polymorphism (OCP violation)
fun getNotificationIcon(notification: Notification): Int {
    return when (notification.type) {
        "message" -> R.drawable.ic_message
        "alert" -> R.drawable.ic_alert
        "promotion" -> R.drawable.ic_promo
        // Must modify this function for every new type
        else -> R.drawable.ic_default
    }
}
```

```kotlin
// ✅ GOOD — Each notification type provides its own icon
sealed interface NotificationType {
    val iconRes: Int

    data object Message : NotificationType {
        override val iconRes: Int = R.drawable.ic_message
    }

    data object Alert : NotificationType {
        override val iconRes: Int = R.drawable.ic_alert
    }

    data object Promotion : NotificationType {
        override val iconRes: Int = R.drawable.ic_promo
    }
}
```

---

### Violation 3: Repository That Depends on a Concrete Database

```kotlin
// ❌ BAD — Repository depends directly on Room DAO (DIP violation)
class UserRepositoryImpl(
    private val userDao: UserDao, // concrete Room class
) : UserRepository {

    override suspend fun getUser(id: String): User? {
        return userDao.getUserById(id)?.toUser()
    }
}
```

```kotlin
// ✅ GOOD — Repository depends on a data source abstraction
interface UserLocalDataSource {
    suspend fun getUserById(id: String): User?
    suspend fun saveUser(user: User)
}

class RoomUserLocalDataSource(
    private val userDao: UserDao,
) : UserLocalDataSource {

    override suspend fun getUserById(id: String): User? {
        return userDao.getUserById(id)?.toUser()
    }

    override suspend fun saveUser(user: User) {
        userDao.insert(user.toEntity())
    }
}

class UserRepositoryImpl(
    private val localDataSource: UserLocalDataSource, // abstraction
    private val remoteDataSource: UserRemoteDataSource, // abstraction
) : UserRepository {

    override suspend fun getUser(id: String): User? {
        return localDataSource.getUserById(id)
            ?: remoteDataSource.fetchUser(id)?.also {
                localDataSource.saveUser(it)
            }
    }
}
```

---

### Violation 4: Test That Requires Real Dependencies

```kotlin
// ❌ BAD — Test requires real Retrofit and Room (DIP violation in test)
@Test
fun `should load restaurants`() = runTest {
    val retrofit = Retrofit.Builder()
        .baseUrl("https://api.example.com/")
        .build()
    val db = Room.inMemoryDatabaseBuilder(context, AppDatabase::class.java).build()

    val viewModel = RestaurantListViewModel(retrofit, db)
    viewModel.loadRestaurants()

    // Flaky: depends on network, Android context, timing
}
```

```kotlin
// ✅ GOOD — Test uses a fake repository
@Test
fun `should load restaurants`() = runTest {
    // Given
    val fakeRepo = FakeRestaurantRepository()
    fakeRepo.addRestaurant(testRestaurant)
    val useCase = GetRestaurantsUseCase(fakeRepo)
    val viewModel = RestaurantListViewModel(useCase)

    // When
    viewModel.loadRestaurants()

    // Then
    val state = viewModel.uiState as RestaurantListUiState.Success
    assertThat(state.restaurants).containsExactly(testRestaurant)
}
```

---

### Violation 5: One Interface for Read and Write Consumers

```kotlin
// ❌ BAD — Settings screen only reads but depends on write methods (ISP violation)
interface AppSettings {
    fun getTheme(): Theme
    fun getLanguage(): String
    fun setTheme(theme: Theme)
    fun setLanguage(language: String)
    fun clearAll()
    fun exportSettings(): ByteArray
    fun importSettings(data: ByteArray)
}

class ThemePickerViewModel(
    private val settings: AppSettings, // depends on clearAll, export, import
) : ViewModel() {
    fun getCurrentTheme(): Theme = settings.getTheme()
}
```

```kotlin
// ✅ GOOD — Separate read and write interfaces
interface ReadableAppSettings {
    fun getTheme(): Theme
    fun getLanguage(): String
}

interface WritableAppSettings : ReadableAppSettings {
    fun setTheme(theme: Theme)
    fun setLanguage(language: String)
}

interface AdminAppSettings : WritableAppSettings {
    fun clearAll()
    fun exportSettings(): ByteArray
    fun importSettings(data: ByteArray)
}

class ThemePickerViewModel(
    private val settings: ReadableAppSettings, // only what it needs
) : ViewModel() {
    fun getCurrentTheme(): Theme = settings.getTheme()
}
```

---

## 📚 Further Reading

### Official Resources

- [Kotlin Official Documentation](https://kotlinlang.org/docs/home.html)
- [Android Architecture Guide](https://developer.android.com/topic/architecture)
- [Guide to App Architecture](https://developer.android.com/topic/architecture/intro)
- [Hilt Dependency Injection](https://developer.android.com/training/dependency-injection/hilt-android)

### Books

- **Clean Architecture** by Robert C. Martin — The definitive reference on layered architecture and SOLID.
- **Agile Software Development, Principles, Patterns, and Practices** by Robert C. Martin — Original formulation of SOLID.
- **Head First Design Patterns** by Eric Freeman and Elisabeth Robson — Accessible introduction to OCP and polymorphism.
- **Kotlin in Action** by Dmitry Jemerov and Svetlana Isakova — Kotlin-specific patterns that support SOLID.

### Related ARC Labs Documents

- `Architecture/clean-architecture.md` — How SOLID principles integrate into our layered architecture.
- `Architecture/dependency-injection.md` — Hilt configuration and module organization.
- `Architecture/testing-strategy.md` — Test doubles, fakes, and the testing pyramid.
- `Quality/code-review-checklist.md` — SOLID violations to check during reviews.

---

> **Remember:** SOLID principles are guidelines, not dogma. Apply them where they reduce complexity
> and improve maintainability. If splitting an interface creates more confusion than it eliminates,
> keep it together. The goal is always a codebase that is **simple, lovable, and complete**.
