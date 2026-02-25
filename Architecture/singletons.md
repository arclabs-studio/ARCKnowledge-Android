# ⚠️ Singletons

**Singletons are one of the most overused and misunderstood patterns in Android development. While they seem convenient, they introduce tight coupling, hidden dependencies, and make testing significantly harder. This guide defines when singletons are acceptable at ARC Labs Studio and how to implement them correctly using Hilt dependency injection.**

---

## 🎯 The Singleton Problem

Singletons violate several core principles of clean architecture. Before reaching for a singleton, understand the problems they introduce.

### Tight Coupling

When a class directly references a singleton, it becomes impossible to swap that dependency. Every consumer is permanently bound to one concrete implementation.

```kotlin
// Tight coupling: UserRepository DEPENDS on a concrete object
class UserRepository {
    fun getUser(id: String): User {
        val cached = CacheManager.get(id)  // Direct singleton reference
        NetworkManager.log("Fetching user $id")  // Another direct reference
        return cached ?: fetchFromNetwork(id)
    }
}
```

This class cannot function without `CacheManager` and `NetworkManager` existing as global objects. You cannot substitute them, mock them, or remove them.

### Hidden Dependencies

Singletons hide what a class truly depends on. The constructor tells you nothing about what the class actually needs to function.

```kotlin
// What does this class depend on? The constructor says "nothing."
class OrderService {
    fun placeOrder(order: Order) {
        val user = UserManager.currentUser          // Hidden dependency 1
        val config = AppConfig.apiBaseUrl            // Hidden dependency 2
        val tracker = AnalyticsTracker.instance      // Hidden dependency 3
        val logger = LogManager.shared               // Hidden dependency 4
        // ...
    }
}
```

A developer reading only the constructor signature would have no idea that `OrderService` depends on four different global objects. This makes the code fragile and difficult to reason about.

### Untestable Code

Singletons make unit testing extremely difficult because you cannot isolate the system under test.

```kotlin
// How do you test this without a real database and network?
class SyncService {
    fun sync() {
        val data = DatabaseManager.getAllPending()   // Real database
        NetworkManager.upload(data)                  // Real network
        AnalyticsTracker.instance.track("sync")      // Real analytics
    }
}

// This test is IMPOSSIBLE to write as a true unit test
// You need real infrastructure or complex reflection hacks
```

### Android-Specific Issues

Android introduces additional problems that make singletons particularly dangerous.

#### Memory Leaks

Singletons live for the entire application lifetime. If they hold references to Activities, Fragments, or Views, those objects can never be garbage collected.

```kotlin
// DANGEROUS: This WILL cause memory leaks
object ViewCache {
    // Activity references held forever, surviving configuration changes
    private val cache = mutableMapOf<String, View>()

    fun store(key: String, view: View) {
        cache[key] = view  // View holds reference to Activity context
    }
}
```

#### Lifecycle Mismatches

Android components have complex lifecycles. Singletons exist outside of these lifecycles, creating mismatches that lead to crashes and unexpected behavior.

```kotlin
// DANGEROUS: Singleton outlives the Activity
object NavigationManager {
    private var currentActivity: Activity? = null

    fun setActivity(activity: Activity) {
        currentActivity = activity
    }

    fun navigate(destination: String) {
        // currentActivity may be destroyed, causing a crash
        currentActivity?.startActivity(Intent(currentActivity, /* ... */))
    }
}
```

#### Process Death

Android can kill your app's process at any time. Singletons lose their state when this happens, but Activities are restored from saved state, leading to inconsistencies.

```kotlin
// State lost on process death, but UI is restored
object CartManager {
    var items = mutableListOf<CartItem>()  // Gone after process death

    fun addItem(item: CartItem) {
        items.add(item)  // User sees restored UI but empty cart
    }
}
```

---

## 🔒 When Singletons Are Acceptable

Singletons are acceptable **only** for truly application-scoped services that meet ALL of the following criteria:

1. **Stateless or read-only state** - No mutable shared state
2. **Infrastructure concern** - Not business logic
3. **Single instance is a requirement** - Multiple instances would cause problems
4. **Injected via Hilt** - NEVER via `object` or `companion object`

### Acceptable Use Cases

| Use Case | Why It Is Acceptable |
|---|---|
| **Application-level configuration** | Read-only, app-scoped, single source of truth |
| **Logging systems** | Stateless infrastructure, single output stream |
| **Analytics tracking** | Stateless infrastructure, single event pipeline |
| **HTTP client (OkHttp/Retrofit)** | Connection pooling requires single instance |
| **Database instance (Room)** | Multiple instances cause file locking issues |

### Never Acceptable

| Use Case | Why It Is NOT Acceptable |
|---|---|
| **User session / auth state** | Mutable business state, must be scoped properly |
| **Navigation state** | Lifecycle-dependent, tied to Activity/Fragment |
| **UI state or cache** | Lifecycle-dependent, causes memory leaks |
| **Business logic services** | Violates Single Responsibility and testability |
| **Mutable shared state** | Race conditions, unpredictable behavior |

---

## 📐 Hilt @Singleton Pattern

At ARC Labs Studio, the **only** acceptable way to create a singleton in Android is through Hilt's `@Singleton` scope. This ensures proper dependency injection, testability, and lifecycle management.

### ✅ Correct: Hilt @Singleton with Interface

```kotlin
// 1. Define the interface (Domain layer)
interface Logger {
    fun debug(tag: String, message: String)
    fun info(tag: String, message: String)
    fun error(tag: String, message: String, throwable: Throwable? = null)
}
```

```kotlin
// 2. Implement the interface (Data layer)
class LoggerImpl @Inject constructor(
    private val config: AppConfig,
) : Logger {

    override fun debug(tag: String, message: String) {
        if (config.isDebugLoggingEnabled) {
            Log.d(tag, message)
        }
    }

    override fun info(tag: String, message: String) {
        Log.i(tag, message)
    }

    override fun error(tag: String, message: String, throwable: Throwable?) {
        Log.e(tag, message, throwable)
    }
}
```

```kotlin
// 3. Provide via Hilt module with @Singleton scope
@Module
@InstallIn(SingletonComponent::class)
abstract class LoggerModule {

    @Singleton
    @Binds
    abstract fun bindLogger(impl: LoggerImpl): Logger
}
```

```kotlin
// 4. Inject via constructor - clean, testable, explicit
class UserRepository @Inject constructor(
    private val logger: Logger,
    private val api: UserApi,
    private val dao: UserDao,
) {
    suspend fun getUser(id: String): User {
        logger.debug("UserRepository", "Fetching user $id")
        return dao.getUser(id) ?: api.fetchUser(id).also { dao.insert(it) }
    }
}
```

### ✅ Correct: Hilt @Provides for Third-Party Libraries

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Singleton
    @Provides
    fun provideOkHttpClient(
        interceptor: AuthInterceptor,
    ): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(interceptor)
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build()
    }

    @Singleton
    @Provides
    fun provideRetrofit(
        okHttpClient: OkHttpClient,
        config: AppConfig,
    ): Retrofit {
        return Retrofit.Builder()
            .baseUrl(config.apiBaseUrl)
            .client(okHttpClient)
            .addConverterFactory(Json.asConverterFactory("application/json".toMediaType()))
            .build()
    }

    @Provides
    fun provideUserApi(retrofit: Retrofit): UserApi {
        return retrofit.create(UserApi::class.java)
    }
}
```

### ❌ Incorrect: Kotlin object for Shared State

```kotlin
// NEVER use Kotlin object for shared state
object UserManager {
    var currentUser: User? = null  // Global mutable state!

    fun isLoggedIn(): Boolean = currentUser != null

    fun login(user: User) {
        currentUser = user  // No thread safety, no lifecycle awareness
    }

    fun logout() {
        currentUser = null  // State lost on process death
    }
}
```

### ❌ Incorrect: Companion Object Instance

```kotlin
// NEVER use companion object singleton pattern
class ApiClient private constructor() {
    companion object {
        @Volatile
        private var instance: ApiClient? = null

        // This is Java-era singleton thinking. Do NOT do this in Kotlin.
        fun getInstance(): ApiClient {
            return instance ?: synchronized(this) {
                instance ?: ApiClient().also { instance = it }
            }
        }
    }

    fun fetch(url: String): Response { /* ... */ }
}
```

### ❌ Incorrect: Direct Object Reference in Business Logic

```kotlin
// NEVER reference singletons directly in business logic
class CheckoutUseCase {
    fun execute(cart: Cart): Order {
        val user = UserManager.currentUser ?: throw NotLoggedInException()
        val config = AppConfig.shippingRates  // Direct singleton reference
        AnalyticsTracker.track("checkout_started")  // Hidden dependency
        return Order(user, cart, config)
    }
}
```

---

## 🔄 Wrapping with Interfaces

Every singleton **must** be wrapped behind an interface. This is non-negotiable at ARC Labs Studio.

### The Pattern

```
Interface (Domain layer)
    ↑
Implementation (Data layer) ← Hilt @Singleton binding
    ↑
Consumer (Any layer) ← Constructor injection of interface
```

### Full Example: Analytics Tracker

```kotlin
// ── Domain Layer ──────────────────────────────────────────

interface AnalyticsTracker {
    fun trackEvent(name: String, properties: Map<String, Any> = emptyMap())
    fun trackScreen(screenName: String)
    fun setUserProperty(key: String, value: String)
}
```

```kotlin
// ── Data Layer ────────────────────────────────────────────

class FirebaseAnalyticsTracker @Inject constructor(
    @ApplicationContext private val context: Context,
) : AnalyticsTracker {

    private val firebaseAnalytics: FirebaseAnalytics by lazy {
        FirebaseAnalytics.getInstance(context)
    }

    override fun trackEvent(name: String, properties: Map<String, Any>) {
        val bundle = Bundle().apply {
            properties.forEach { (key, value) ->
                when (value) {
                    is String -> putString(key, value)
                    is Int -> putInt(key, value)
                    is Long -> putLong(key, value)
                    is Double -> putDouble(key, value)
                    is Boolean -> putBoolean(key, value)
                    else -> putString(key, value.toString())
                }
            }
        }
        firebaseAnalytics.logEvent(name, bundle)
    }

    override fun trackScreen(screenName: String) {
        trackEvent(FirebaseAnalytics.Event.SCREEN_VIEW, mapOf("screen_name" to screenName))
    }

    override fun setUserProperty(key: String, value: String) {
        firebaseAnalytics.setUserProperty(key, value)
    }
}
```

```kotlin
// ── Hilt Module ───────────────────────────────────────────

@Module
@InstallIn(SingletonComponent::class)
abstract class AnalyticsModule {

    @Singleton
    @Binds
    abstract fun bindAnalyticsTracker(
        impl: FirebaseAnalyticsTracker,
    ): AnalyticsTracker
}
```

```kotlin
// ── Consumer ──────────────────────────────────────────────

class PlaceOrderUseCase @Inject constructor(
    private val orderRepository: OrderRepository,
    private val analyticsTracker: AnalyticsTracker,
) {
    suspend fun execute(order: Order): Result<OrderConfirmation> {
        return orderRepository.placeOrder(order).also { result ->
            result.onSuccess {
                analyticsTracker.trackEvent(
                    "order_placed",
                    mapOf("order_id" to it.id, "total" to it.total),
                )
            }
        }
    }
}
```

---

## 🧪 Testing Singletons

Because all singletons are injected via Hilt and accessed through interfaces, testing is straightforward.

### Unit Tests with Fakes

```kotlin
// ── Fake Implementation ───────────────────────────────────

class FakeLogger : Logger {
    val debugMessages = mutableListOf<Pair<String, String>>()
    val infoMessages = mutableListOf<Pair<String, String>>()
    val errorMessages = mutableListOf<Triple<String, String, Throwable?>>()

    override fun debug(tag: String, message: String) {
        debugMessages.add(tag to message)
    }

    override fun info(tag: String, message: String) {
        infoMessages.add(tag to message)
    }

    override fun error(tag: String, message: String, throwable: Throwable?) {
        errorMessages.add(Triple(tag, message, throwable))
    }
}
```

```kotlin
// ── Unit Test ─────────────────────────────────────────────

class UserRepositoryTest {

    private fun makeSUT(
        logger: Logger = FakeLogger(),
        api: UserApi = FakeUserApi(),
        dao: UserDao = FakeUserDao(),
    ): UserRepository {
        return UserRepository(logger, api, dao)
    }

    @Test
    fun `should log debug message when fetching user`() = runTest {
        // Given
        val fakeLogger = FakeLogger()
        val sut = makeSUT(logger = fakeLogger)

        // When
        sut.getUser("user-123")

        // Then
        assertThat(fakeLogger.debugMessages).containsExactly(
            "UserRepository" to "Fetching user user-123"
        )
    }
}
```

### Instrumented Tests with Hilt

```kotlin
// ── Test Module Replacement ───────────────────────────────

@Module
@InstallIn(SingletonComponent::class)
abstract class FakeAnalyticsModule {

    @Singleton
    @Binds
    abstract fun bindAnalyticsTracker(
        impl: FakeAnalyticsTracker,
    ): AnalyticsTracker
}

class FakeAnalyticsTracker @Inject constructor() : AnalyticsTracker {
    val trackedEvents = mutableListOf<Pair<String, Map<String, Any>>>()
    val trackedScreens = mutableListOf<String>()

    override fun trackEvent(name: String, properties: Map<String, Any>) {
        trackedEvents.add(name to properties)
    }

    override fun trackScreen(screenName: String) {
        trackedScreens.add(screenName)
    }

    override fun setUserProperty(key: String, value: String) { /* no-op */ }
}
```

```kotlin
// ── Hilt Instrumented Test ────────────────────────────────

@HiltAndroidTest
@UninstallModules(AnalyticsModule::class)
@RunWith(AndroidJUnit4::class)
class PlaceOrderUseCaseTest {

    @get:Rule
    val hiltRule = HiltAndroidRule(this)

    @Inject
    lateinit var analyticsTracker: AnalyticsTracker

    @Inject
    lateinit var placeOrderUseCase: PlaceOrderUseCase

    @Before
    fun setUp() {
        hiltRule.inject()
    }

    @Test
    fun shouldTrackEventWhenOrderIsPlaced() = runTest {
        // Given
        val order = Order(items = listOf(OrderItem("item-1", 2)))
        val fakeTracker = analyticsTracker as FakeAnalyticsTracker

        // When
        placeOrderUseCase.execute(order)

        // Then
        assertThat(fakeTracker.trackedEvents).hasSize(1)
        assertThat(fakeTracker.trackedEvents.first().first).isEqualTo("order_placed")
    }
}
```

---

## ✅ Singleton Checklist

Before making anything a singleton, verify **every** item on this checklist:

- [ ] **Is this truly application-scoped?** Does it need to live for the entire app lifetime?
- [ ] **Is the state immutable or read-only?** Mutable shared state must NOT be a singleton.
- [ ] **Is this an infrastructure concern?** Business logic must NEVER be a singleton.
- [ ] **Is it provided via Hilt `@Singleton`?** NEVER use `object` or `companion object`.
- [ ] **Is it behind an interface?** Every singleton must be accessed through an interface.
- [ ] **Does it avoid holding Android context references?** Use `@ApplicationContext` only when absolutely necessary.
- [ ] **Is it testable?** Can you substitute it with a fake in unit tests?
- [ ] **Is it thread-safe?** Singletons are accessed from multiple threads.
- [ ] **Does it handle process death correctly?** State is lost when the process is killed.
- [ ] **Have you considered a narrower scope?** `@ActivityScoped`, `@ViewModelScoped`, or `@FragmentScoped` may be more appropriate.

If any answer is **no**, do not make it a singleton. Use a more appropriate scope.

---

## 🚫 Common Mistakes

### Mistake 1: Using `object` for Shared Mutable State

❌ **Wrong:**

```kotlin
object SessionManager {
    var accessToken: String? = null
    var refreshToken: String? = null
    var currentUser: User? = null

    fun isAuthenticated(): Boolean = accessToken != null
}

// Usage scattered across the codebase
class ProfileViewModel : ViewModel() {
    fun loadProfile() {
        val user = SessionManager.currentUser  // Hidden dependency
            ?: return
        // ...
    }
}
```

✅ **Correct:**

```kotlin
// Interface
interface SessionManager {
    val currentUser: Flow<User?>
    val isAuthenticated: Flow<Boolean>
    suspend fun setSession(accessToken: String, refreshToken: String, user: User)
    suspend fun clearSession()
}

// Implementation with proper storage
class SessionManagerImpl @Inject constructor(
    private val dataStore: DataStore<Preferences>,
    private val encryptedPrefs: SharedPreferences,
) : SessionManager {

    override val currentUser: Flow<User?> = dataStore.data.map { prefs ->
        prefs[USER_JSON_KEY]?.let { json -> User.fromJson(json) }
    }

    override val isAuthenticated: Flow<Boolean> = currentUser.map { it != null }

    override suspend fun setSession(
        accessToken: String,
        refreshToken: String,
        user: User,
    ) {
        encryptedPrefs.edit().putString("access_token", accessToken).apply()
        encryptedPrefs.edit().putString("refresh_token", refreshToken).apply()
        dataStore.edit { prefs -> prefs[USER_JSON_KEY] = user.toJson() }
    }

    override suspend fun clearSession() {
        encryptedPrefs.edit().clear().apply()
        dataStore.edit { prefs -> prefs.clear() }
    }
}

// Hilt module
@Module
@InstallIn(SingletonComponent::class)
abstract class SessionModule {
    @Singleton
    @Binds
    abstract fun bindSessionManager(impl: SessionManagerImpl): SessionManager
}

// Clean usage with explicit dependency
class ProfileViewModel @Inject constructor(
    private val sessionManager: SessionManager,
) : ViewModel() {
    val user = sessionManager.currentUser.stateIn(
        viewModelScope,
        SharingStarted.WhileSubscribed(5000),
        null,
    )
}
```

### Mistake 2: Singleton Holding Activity Context

❌ **Wrong:**

```kotlin
// This WILL leak the Activity
object ImageLoader {
    private lateinit var context: Context

    fun init(context: Context) {
        this.context = context  // If this is an Activity, it leaks forever
    }

    fun load(url: String): Bitmap {
        // Uses leaked context
        return BitmapFactory.decodeStream(
            context.contentResolver.openInputStream(Uri.parse(url))
        )
    }
}

// Called from Activity.onCreate()
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ImageLoader.init(this)  // Activity reference held forever
    }
}
```

✅ **Correct:**

```kotlin
interface ImageLoader {
    suspend fun load(url: String): Bitmap
}

class CoilImageLoader @Inject constructor(
    @ApplicationContext private val context: Context,
) : ImageLoader {

    private val imageLoader = context.imageLoader

    override suspend fun load(url: String): Bitmap {
        val request = ImageRequest.Builder(context)
            .data(url)
            .build()
        return (imageLoader.execute(request).drawable as BitmapDrawable).bitmap
    }
}

@Module
@InstallIn(SingletonComponent::class)
abstract class ImageLoaderModule {
    @Singleton
    @Binds
    abstract fun bindImageLoader(impl: CoilImageLoader): ImageLoader
}
```

### Mistake 3: Using Singleton Where ViewModel Scope Suffices

❌ **Wrong:**

```kotlin
// This does NOT need to be a singleton
@Singleton
class SearchStateHolder @Inject constructor() {
    var query: String = ""
    var results: List<SearchResult> = emptyList()
    var isLoading: Boolean = false
}
```

✅ **Correct:**

```kotlin
// UI state belongs in a ViewModel, scoped to the screen lifecycle
@HiltViewModel
class SearchViewModel @Inject constructor(
    private val searchUseCase: SearchUseCase,
) : ViewModel() {

    var query by mutableStateOf("")
        private set

    var results by mutableStateOf<List<SearchResult>>(emptyList())
        private set

    var isLoading by mutableStateOf(false)
        private set

    fun onQueryChanged(newQuery: String) {
        query = newQuery
    }

    fun search() {
        viewModelScope.launch {
            isLoading = true
            results = searchUseCase.execute(query)
            isLoading = false
        }
    }
}
```

---

## 📚 Further Reading

- [Hilt Dependency Injection - Android Developers](https://developer.android.com/training/dependency-injection/hilt-android)
- [Scoping in Hilt - Dagger Documentation](https://dagger.dev/hilt/components.html)
- [Guide to App Architecture - Android Developers](https://developer.android.com/topic/architecture)
- [Dependency Injection Best Practices - Android Developers](https://developer.android.com/training/dependency-injection)
- [Testing with Hilt - Android Developers](https://developer.android.com/training/dependency-injection/hilt-testing)
- [Clean Architecture by Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [SOLID Principles in Kotlin](https://medium.com/android-news/solid-principles-in-kotlin-79100c670df2)
- [Kotlin Coroutines and Flow for State Management](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow)
