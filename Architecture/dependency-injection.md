# 💉 Dependency Injection with Hilt

**The definitive guide to Dependency Injection in ARC Labs Android projects using Hilt. This document covers setup, core annotations, module patterns, scoping strategies, Clean Architecture integration, testing, and common pitfalls. All ARC Labs Android projects MUST use Hilt as their DI framework.**

---

## 🎯 Why Hilt?

**Hilt is the standard dependency injection framework for all ARC Labs Android projects. It builds on top of Dagger 2, providing a simplified API tailored to Android development with compile-time safety and lifecycle awareness.**

---

### The DI Hierarchy of Preference

When providing dependencies in Android, follow this order of preference:

1. **Constructor Injection** (ALWAYS preferred)
2. **Field Injection** (ONLY for Android framework classes)
3. **Service Locator** (NEVER use in ARC Labs projects)

---

### Why Constructor Injection Wins

```kotlin
// ✅ Constructor Injection — Dependencies are explicit, immutable, and testable
class UserRepository @Inject constructor(
    private val api: UserApi,
    private val dao: UserDao,
    private val mapper: UserMapper,
) {
    suspend fun getUser(id: String): User {
        return mapper.toDomain(api.fetchUser(id))
    }
}
```

```kotlin
// ❌ Service Locator — Dependencies are hidden, mutable, and hard to test
class UserRepository {
    private val api = ServiceLocator.get<UserApi>()
    private val dao = ServiceLocator.get<UserDao>()
    private val mapper = ServiceLocator.get<UserMapper>()

    suspend fun getUser(id: String): User {
        return mapper.toDomain(api.fetchUser(id))
    }
}
```

---

### Compile-Time Verification

Hilt validates the entire dependency graph at compile time. If a dependency is missing or misconfigured, you get a clear compiler error instead of a runtime crash.

```
error: [Dagger/MissingBinding] com.arclabs.domain.repository.UserRepository cannot be
provided without an @Provides-annotated method.
```

This is a massive advantage over runtime DI frameworks like Koin, where missing bindings only surface when the code path is executed.

---

### Android Lifecycle Awareness

Hilt understands Android component lifecycles and automatically manages the creation and destruction of scoped dependencies. Dependencies scoped to a ViewModel survive configuration changes. Dependencies scoped to an Activity are destroyed when the Activity is destroyed.

---

### Benefits Summary

| Feature                    | Hilt | Koin | Manual DI |
|----------------------------|------|------|-----------|
| Compile-time safety        | ✅   | ❌   | ✅        |
| Android lifecycle aware    | ✅   | ⚠️   | ❌        |
| Code generation            | ✅   | ❌   | ❌        |
| Boilerplate                | Low  | Low  | High      |
| Testing support            | ✅   | ✅   | ⚠️        |
| Jetpack integration        | ✅   | ⚠️   | ❌        |
| ARC Labs approved          | ✅   | ❌   | ❌        |

---

## 🚀 Setup

**Complete Hilt setup for a new ARC Labs Android project. Follow every step carefully — missing any configuration will result in cryptic build errors.**

---

### Step 1: Root `build.gradle.kts`

Add the Hilt Gradle plugin to your project-level build file:

```kotlin
// build.gradle.kts (project root)
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.hilt.android) apply false
    alias(libs.plugins.ksp) apply false
}
```

---

### Step 2: Version Catalog (`libs.versions.toml`)

Define Hilt versions in the version catalog:

```toml
[versions]
hilt = "2.51.1"
hiltNavigationCompose = "1.2.0"
ksp = "2.0.21-1.0.27"

[libraries]
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
hilt-android-testing = { group = "com.google.dagger", name = "hilt-android-testing", version.ref = "hilt" }
hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version.ref = "hiltNavigationCompose" }

[plugins]
hilt-android = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

---

### Step 3: App-Level `build.gradle.kts`

Apply the plugins and add dependencies in your app module:

```kotlin
// build.gradle.kts (app module)
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.hilt.android)
    alias(libs.plugins.ksp)
}

android {
    // ... standard Android configuration
}

dependencies {
    // Hilt
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)

    // Hilt + Jetpack Compose Navigation
    implementation(libs.hilt.navigation.compose)

    // Hilt Testing
    androidTestImplementation(libs.hilt.android.testing)
    kspAndroidTest(libs.hilt.compiler)

    // Hilt Unit Testing
    testImplementation(libs.hilt.android.testing)
    kspTest(libs.hilt.compiler)
}
```

---

### Step 4: KSP Configuration

Ensure KSP is correctly configured. Hilt uses KSP (Kotlin Symbol Processing) for annotation processing, which is faster than KAPT:

```kotlin
// build.gradle.kts (app module)
ksp {
    arg("dagger.fastInit", "enabled")
    arg("dagger.formatGeneratedSource", "disabled")
}
```

> **Note:** ARC Labs projects MUST use KSP instead of KAPT. KAPT is deprecated and slower. If migrating from KAPT, remove all `kapt` configurations and replace with `ksp`.

---

### Step 5: `@HiltAndroidApp` on Application Class

```kotlin
package com.arclabs.myapp

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class MyApplication : Application() {

    override fun onCreate() {
        super.onCreate()
        // Application-level initialization here
    }
}
```

Register the Application class in `AndroidManifest.xml`:

```xml
<application
    android:name=".MyApplication"
    android:label="@string/app_name"
    ... >
</application>
```

---

### Step 6: Verify Setup

Build the project. If Hilt is configured correctly, you should see no errors. If you encounter issues:

1. Verify all plugin versions are compatible
2. Ensure KSP version matches your Kotlin version
3. Clean and rebuild: `./gradlew clean build`
4. Check that `@HiltAndroidApp` is applied to exactly one Application class

---

## 📐 Core Annotations

**Hilt provides a focused set of annotations that handle the vast majority of DI scenarios in Android. Understanding each annotation and when to use it is critical.**

---

### @HiltAndroidApp

**Marks the Application class as the root of the Hilt dependency graph. Every Hilt-enabled project MUST have exactly one class annotated with `@HiltAndroidApp`.**

```kotlin
@HiltAndroidApp
class MyApplication : Application() {

    override fun onCreate() {
        super.onCreate()
        // Timber, analytics, crash reporting initialization
    }
}
```

What this does:
- Triggers Hilt's code generation
- Creates a `SingletonComponent` that lives for the entire application lifecycle
- Sets up the component hierarchy for all Android entry points

---

### @AndroidEntryPoint

**Marks Android framework classes (Activity, Fragment, Service, BroadcastReceiver, View) as injection targets. Hilt will generate the necessary code to provide dependencies to these classes.**

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyAppTheme {
                AppNavigation()
            }
        }
    }
}
```

```kotlin
@AndroidEntryPoint
class UserFragment : Fragment() {

    private val viewModel: UserViewModel by viewModels()

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?,
    ): View {
        return ComposeView(requireContext()).apply {
            setContent {
                UserScreen(viewModel = viewModel)
            }
        }
    }
}
```

```kotlin
@AndroidEntryPoint
class SyncService : Service() {

    @Inject
    lateinit var syncManager: SyncManager

    override fun onBind(intent: Intent?): IBinder? = null

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        syncManager.startSync()
        return START_NOT_STICKY
    }
}
```

> **Rule:** If a class extends an Android framework class and needs injected dependencies, it MUST be annotated with `@AndroidEntryPoint`. The parent Activity must also be annotated if a Fragment uses `@AndroidEntryPoint`.

---

### @HiltViewModel

**Marks a ViewModel for Hilt injection. The ViewModel's dependencies are resolved from the `ViewModelComponent` and the ViewModel itself is lifecycle-aware.**

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase,
    private val updateUserUseCase: UpdateUserUseCase,
    private val savedStateHandle: SavedStateHandle,
) : ViewModel() {

    private val userId: String = checkNotNull(savedStateHandle["userId"])

    private val _uiState = MutableStateFlow<UserUiState>(UserUiState.Loading)
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()

    init {
        loadUser()
    }

    private fun loadUser() {
        viewModelScope.launch {
            getUserUseCase(userId)
                .onSuccess { user ->
                    _uiState.value = UserUiState.Success(user)
                }
                .onFailure { error ->
                    _uiState.value = UserUiState.Error(error.message.orEmpty())
                }
        }
    }

    fun updateUser(name: String) {
        viewModelScope.launch {
            _uiState.value = UserUiState.Loading
            updateUserUseCase(userId, name)
                .onSuccess { user ->
                    _uiState.value = UserUiState.Success(user)
                }
                .onFailure { error ->
                    _uiState.value = UserUiState.Error(error.message.orEmpty())
                }
        }
    }
}

sealed interface UserUiState {
    data object Loading : UserUiState
    data class Success(val user: User) : UserUiState
    data class Error(val message: String) : UserUiState
}
```

Using the ViewModel in Compose with Hilt Navigation:

```kotlin
@Composable
fun UserScreen(
    viewModel: UserViewModel = hiltViewModel(),
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    when (val state = uiState) {
        is UserUiState.Loading -> LoadingIndicator()
        is UserUiState.Success -> UserContent(user = state.user)
        is UserUiState.Error -> ErrorMessage(message = state.message)
    }
}
```

---

### @Inject

**The most fundamental annotation. Tells Hilt how to provide instances of a type.**

---

#### Constructor Injection (Preferred)

Use `@Inject` on a constructor to tell Hilt how to create instances of that class. This is the preferred method for all types you own.

```kotlin
// ✅ Constructor injection — clean, explicit, testable
class UserRepositoryImpl @Inject constructor(
    private val api: UserApi,
    private val dao: UserDao,
    private val mapper: UserMapper,
) : UserRepository {

    override suspend fun getUser(id: String): Result<User> {
        return runCatching {
            val response = api.fetchUser(id)
            val entity = mapper.toEntity(response)
            dao.insertUser(entity)
            mapper.toDomain(entity)
        }
    }
}
```

```kotlin
// ✅ Use cases with constructor injection
class GetUserUseCase @Inject constructor(
    private val userRepository: UserRepository,
) {
    suspend operator fun invoke(userId: String): Result<User> {
        return userRepository.getUser(userId)
    }
}
```

```kotlin
// ✅ Mappers with constructor injection
class UserMapper @Inject constructor() {

    fun toDomain(entity: UserEntity): User {
        return User(
            id = entity.id,
            name = entity.name,
            email = entity.email,
        )
    }

    fun toEntity(response: UserResponse): UserEntity {
        return UserEntity(
            id = response.id,
            name = response.name,
            email = response.email,
        )
    }
}
```

---

#### Field Injection (Android Framework Classes Only)

Field injection uses `@Inject` on a `lateinit var` property. This should ONLY be used when constructor injection is not possible, specifically for Android framework classes.

```kotlin
// ✅ Field injection — only for Android framework classes
@AndroidEntryPoint
class SyncWorker @AssistedInject constructor(
    @Assisted context: Context,
    @Assisted workerParams: WorkerParameters,
) : CoroutineWorker(context, workerParams) {

    @Inject
    lateinit var syncRepository: SyncRepository

    override suspend fun doWork(): Result {
        return try {
            syncRepository.performSync()
            Result.success()
        } catch (e: Exception) {
            Result.retry()
        }
    }
}
```

```kotlin
// ❌ NEVER use field injection when constructor injection is possible
class UserRepository {
    @Inject
    lateinit var api: UserApi  // WRONG! Use constructor injection instead

    @Inject
    lateinit var dao: UserDao  // WRONG! Use constructor injection instead
}
```

---

## 📦 Modules

**Hilt Modules are classes where you define how to provide types that cannot use constructor injection — either because you do not own the type or because the type is an interface that needs to be bound to an implementation.**

---

### @Module + @InstallIn

Every Hilt module must be annotated with `@Module` and `@InstallIn`. The `@InstallIn` annotation specifies which Hilt component the module is installed in, which determines the module's lifetime and visibility.

---

#### Component Hierarchy

Hilt provides a predefined set of components that correspond to Android lifecycle scopes:

| Component               | Scope              | Created At               | Destroyed At             |
|--------------------------|--------------------|--------------------------|--------------------------|
| `SingletonComponent`     | `@Singleton`       | `Application.onCreate()` | Application destroyed    |
| `ActivityRetainedComponent` | `@ActivityRetainedScoped` | `Activity.onCreate()` first time | `Activity.onDestroy()` final |
| `ViewModelComponent`     | `@ViewModelScoped` | `ViewModel` created      | `ViewModel` cleared      |
| `ActivityComponent`      | `@ActivityScoped`  | `Activity.onCreate()`    | `Activity.onDestroy()`   |
| `FragmentComponent`      | `@FragmentScoped`  | `Fragment.onAttach()`    | `Fragment.onDestroy()`   |
| `ViewComponent`          | `@ViewScoped`      | `View.super()`           | `View` destroyed         |
| `ServiceComponent`       | `@ServiceScoped`   | `Service.onCreate()`     | `Service.onDestroy()`    |

**Parent-child relationships:**

```
SingletonComponent
├── ActivityRetainedComponent
│   ├── ViewModelComponent
│   └── ActivityComponent
│       ├── FragmentComponent
│       │   └── ViewWithFragmentComponent
│       └── ViewComponent
└── ServiceComponent
```

A component can access bindings from its parent components. For example, `ViewModelComponent` can access bindings from `ActivityRetainedComponent` and `SingletonComponent`.

---

### @Provides

**Use `@Provides` for types you do not own or that require custom construction logic.** This is common for third-party libraries like Retrofit, OkHttp, Room, DataStore, etc.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Singleton
    @Provides
    fun provideOkHttpClient(
        authInterceptor: AuthInterceptor,
    ): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(authInterceptor)
            .addInterceptor(
                HttpLoggingInterceptor().apply {
                    level = if (BuildConfig.DEBUG) {
                        HttpLoggingInterceptor.Level.BODY
                    } else {
                        HttpLoggingInterceptor.Level.NONE
                    }
                }
            )
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .writeTimeout(30, TimeUnit.SECONDS)
            .build()
    }

    @Singleton
    @Provides
    fun provideRetrofit(
        okHttpClient: OkHttpClient,
    ): Retrofit {
        val json = Json {
            ignoreUnknownKeys = true
            coerceInputValues = true
            encodeDefaults = true
        }

        return Retrofit.Builder()
            .baseUrl(BuildConfig.BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(json.asConverterFactory("application/json".toMediaType()))
            .build()
    }

    @Singleton
    @Provides
    fun provideUserApi(retrofit: Retrofit): UserApi {
        return retrofit.create(UserApi::class.java)
    }

    @Singleton
    @Provides
    fun provideRestaurantApi(retrofit: Retrofit): RestaurantApi {
        return retrofit.create(RestaurantApi::class.java)
    }
}
```

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Singleton
    @Provides
    fun provideDatabase(
        @ApplicationContext context: Context,
    ): AppDatabase {
        return Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "arclabs_database",
        )
            .fallbackToDestructiveMigration()
            .build()
    }

    @Provides
    fun provideUserDao(database: AppDatabase): UserDao {
        return database.userDao()
    }

    @Provides
    fun provideRestaurantDao(database: AppDatabase): RestaurantDao {
        return database.restaurantDao()
    }
}
```

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DataStoreModule {

    @Singleton
    @Provides
    fun provideDataStore(
        @ApplicationContext context: Context,
    ): DataStore<Preferences> {
        return PreferenceDataStoreFactory.create(
            produceFile = {
                context.preferencesDataStoreFile("user_preferences")
            }
        )
    }
}
```

---

### @Binds

**Use `@Binds` to tell Hilt which implementation to use for an interface.** This is more efficient than `@Provides` because it does not generate a separate factory class.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl,
    ): UserRepository

    @Binds
    abstract fun bindRestaurantRepository(
        impl: RestaurantRepositoryImpl,
    ): RestaurantRepository

    @Binds
    abstract fun bindAuthRepository(
        impl: AuthRepositoryImpl,
    ): AuthRepository

    @Binds
    abstract fun bindSettingsRepository(
        impl: SettingsRepositoryImpl,
    ): SettingsRepository
}
```

> **Rule:** The module class MUST be `abstract` when using `@Binds`. Use `object` only for `@Provides` methods. If you need both `@Binds` and `@Provides` in the same component, use a companion object:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class DataModule {

    @Binds
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl,
    ): UserRepository

    companion object {

        @Singleton
        @Provides
        fun provideDatabase(
            @ApplicationContext context: Context,
        ): AppDatabase {
            return Room.databaseBuilder(
                context,
                AppDatabase::class.java,
                "arclabs_database",
            ).build()
        }
    }
}
```

---

### @Provides vs @Binds Decision Table

| Scenario                                  | Use        |
|-------------------------------------------|------------|
| Binding interface to implementation       | `@Binds`   |
| Third-party library instance              | `@Provides`|
| Builder pattern construction              | `@Provides`|
| Type you own with simple constructor      | `@Inject` on constructor |
| Type requiring conditional logic          | `@Provides`|
| Abstract class binding                    | `@Binds`   |

---

## 🔒 Scopes

**Scoping controls how many instances of a dependency are created and how long they live. Incorrect scoping is one of the most common sources of bugs in Hilt projects.**

---

### Scope Reference Table

| Annotation               | Component                   | Lifetime                        | Use When                                |
|---------------------------|-----------------------------|---------------------------------|------------------------------------------|
| `@Singleton`              | `SingletonComponent`        | Entire app lifetime             | Shared state, expensive-to-create objects |
| `@ActivityRetainedScoped` | `ActivityRetainedComponent` | Survives configuration changes  | Shared across ViewModel instances        |
| `@ViewModelScoped`        | `ViewModelComponent`        | ViewModel lifetime              | Shared within a single ViewModel         |
| `@ActivityScoped`         | `ActivityComponent`         | Activity lifetime               | Activity-specific state                  |
| `@FragmentScoped`         | `FragmentComponent`         | Fragment lifetime               | Fragment-specific state                  |
| (unscoped)                | Any component               | New instance every injection    | Stateless objects, mappers, use cases    |

---

### When to Scope

✅ **DO scope when:**
- The object holds mutable shared state
- The object is expensive to create (database, HTTP client)
- Multiple consumers need the exact same instance
- The object manages a resource that should not be duplicated

❌ **DO NOT scope when:**
- The object is stateless (mappers, use cases, validators)
- The object is cheap to create
- Each consumer should get its own instance
- You are unsure — default to unscoped

---

### Scoping Examples

```kotlin
// ✅ Scoped — Database is expensive and should be shared
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Singleton
    @Provides
    fun provideDatabase(
        @ApplicationContext context: Context,
    ): AppDatabase {
        return Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "app_database",
        ).build()
    }
}
```

```kotlin
// ✅ Unscoped — Use case is stateless, cheap to create
class GetUserUseCase @Inject constructor(
    private val userRepository: UserRepository,
) {
    suspend operator fun invoke(userId: String): Result<User> {
        return userRepository.getUser(userId)
    }
}
// No scope annotation needed! A new instance is fine each time.
```

```kotlin
// ❌ Over-scoped — Mapper is stateless, should NOT be singleton
@Singleton  // WRONG! This wastes memory and adds unnecessary scope
class UserMapper @Inject constructor() {
    fun toDomain(entity: UserEntity): User { ... }
}
```

```kotlin
// ✅ Correctly unscoped mapper
class UserMapper @Inject constructor() {
    fun toDomain(entity: UserEntity): User { ... }
}
```

---

### The Overscoping Anti-Pattern

❌ **A common mistake is making everything `@Singleton`:**

```kotlin
// ❌ DO NOT DO THIS — everything is singleton for no reason
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Singleton  // WRONG — repository may not need to be singleton
    @Binds
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository

    @Singleton  // WRONG — repository may not need to be singleton
    @Binds
    abstract fun bindSettingsRepository(impl: SettingsRepositoryImpl): SettingsRepository
}
```

✅ **Only scope what truly needs to be shared:**

```kotlin
// ✅ Repositories are unscoped unless they hold shared state
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository

    @Binds
    abstract fun bindSettingsRepository(impl: SettingsRepositoryImpl): SettingsRepository
}
```

---

## 🏷️ Qualifiers

**Qualifiers disambiguate when you have multiple bindings for the same type. They are essential when providing multiple instances of the same class (e.g., different `CoroutineDispatcher` instances).**

---

### Custom Qualifier Annotations

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class IoDispatcher

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class MainDispatcher

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class DefaultDispatcher

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class UnconfinedDispatcher
```

---

### Providing Qualified Dependencies

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DispatcherModule {

    @IoDispatcher
    @Provides
    fun provideIoDispatcher(): CoroutineDispatcher = Dispatchers.IO

    @MainDispatcher
    @Provides
    fun provideMainDispatcher(): CoroutineDispatcher = Dispatchers.Main

    @DefaultDispatcher
    @Provides
    fun provideDefaultDispatcher(): CoroutineDispatcher = Dispatchers.Default

    @UnconfinedDispatcher
    @Provides
    fun provideUnconfinedDispatcher(): CoroutineDispatcher = Dispatchers.Unconfined
}
```

---

### Injecting Qualified Dependencies

```kotlin
class UserRepositoryImpl @Inject constructor(
    private val api: UserApi,
    private val dao: UserDao,
    @IoDispatcher private val ioDispatcher: CoroutineDispatcher,
) : UserRepository {

    override suspend fun getUser(id: String): Result<User> {
        return withContext(ioDispatcher) {
            runCatching {
                val response = api.fetchUser(id)
                dao.insertUser(response.toEntity())
                response.toDomain()
            }
        }
    }
}
```

---

### Qualifier for Multiple API Base URLs

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthApi

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class ContentApi

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @AuthApi
    @Singleton
    @Provides
    fun provideAuthRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BuildConfig.AUTH_BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(Json.asConverterFactory("application/json".toMediaType()))
            .build()
    }

    @ContentApi
    @Singleton
    @Provides
    fun provideContentRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BuildConfig.CONTENT_BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(Json.asConverterFactory("application/json".toMediaType()))
            .build()
    }

    @Provides
    fun provideAuthService(@AuthApi retrofit: Retrofit): AuthService {
        return retrofit.create(AuthService::class.java)
    }

    @Provides
    fun provideContentService(@ContentApi retrofit: Retrofit): ContentService {
        return retrofit.create(ContentService::class.java)
    }
}
```

---

### Built-In Qualifiers

Hilt provides two built-in qualifiers for Android Context:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides
    fun provideSharedPreferences(
        @ApplicationContext context: Context,  // Application context
    ): SharedPreferences {
        return context.getSharedPreferences("app_prefs", Context.MODE_PRIVATE)
    }
}

@Module
@InstallIn(ActivityComponent::class)
object ActivityModule {

    @Provides
    fun provideLayoutInflater(
        @ActivityContext context: Context,  // Activity context
    ): LayoutInflater {
        return LayoutInflater.from(context)
    }
}
```

---

## 🏗️ Hilt + Clean Architecture

**This section demonstrates how Hilt wires together all layers of Clean Architecture in an ARC Labs Android project. Each layer has clear responsibilities and dependencies flow inward.**

---

### Architecture Layers with Hilt

```
┌──────────────────────────────────────────────────┐
│                 Presentation Layer                │
│  ┌──────────────────┐  ┌──────────────────────┐  │
│  │   Composables    │──│     ViewModels        │  │
│  │  (UI + State)    │  │  (@HiltViewModel)     │  │
│  └──────────────────┘  └──────────────────────┘  │
│                              │                    │
│                              │ depends on         │
├──────────────────────────────┼────────────────────┤
│                 Domain Layer │                    │
│  ┌──────────────────┐  ┌────┴─────────────────┐  │
│  │    Entities       │  │     Use Cases        │  │
│  │  (Data Models)    │  │   (@Inject)          │  │
│  └──────────────────┘  └──────────────────────┘  │
│                              │                    │
│  ┌──────────────────────────┐│                    │
│  │  Repository Protocols    ││                    │
│  │    (Interfaces)          ││                    │
│  └──────────────────────────┘│                    │
├──────────────────────────────┼────────────────────┤
│                 Data Layer   │                    │
│  ┌──────────────────┐  ┌────┴─────────────────┐  │
│  │   Data Sources    │  │ Repository Impls     │  │
│  │  (API, DB, etc.)  │  │ (@Inject constructor)│  │
│  └──────────────────┘  └──────────────────────┘  │
│  ┌──────────────────────────────────────────────┐│
│  │  Hilt Modules (@Module + @InstallIn)         ││
│  └──────────────────────────────────────────────┘│
└──────────────────────────────────────────────────┘
```

---

### Full Wiring Example: User Feature

#### 1. Domain Layer — Entity

```kotlin
// domain/model/User.kt
data class User(
    val id: String,
    val name: String,
    val email: String,
    val avatarUrl: String?,
)
```

#### 2. Domain Layer — Repository Interface

```kotlin
// domain/repository/UserRepository.kt
interface UserRepository {
    suspend fun getUser(id: String): Result<User>
    suspend fun updateUser(id: String, name: String): Result<User>
    fun observeUser(id: String): Flow<User>
}
```

#### 3. Domain Layer — Use Case

```kotlin
// domain/usecase/GetUserUseCase.kt
class GetUserUseCase @Inject constructor(
    private val userRepository: UserRepository,
) {
    suspend operator fun invoke(userId: String): Result<User> {
        return userRepository.getUser(userId)
    }
}
```

```kotlin
// domain/usecase/UpdateUserUseCase.kt
class UpdateUserUseCase @Inject constructor(
    private val userRepository: UserRepository,
) {
    suspend operator fun invoke(userId: String, name: String): Result<User> {
        require(name.isNotBlank()) { "Name must not be blank" }
        return userRepository.updateUser(userId, name)
    }
}
```

#### 4. Data Layer — API Service

```kotlin
// data/remote/UserApi.kt
interface UserApi {

    @GET("users/{id}")
    suspend fun fetchUser(@Path("id") id: String): UserResponse

    @PUT("users/{id}")
    suspend fun updateUser(
        @Path("id") id: String,
        @Body request: UpdateUserRequest,
    ): UserResponse
}
```

#### 5. Data Layer — DTO / Response

```kotlin
// data/remote/dto/UserResponse.kt
@Serializable
data class UserResponse(
    val id: String,
    val name: String,
    val email: String,
    @SerialName("avatar_url") val avatarUrl: String?,
)
```

#### 6. Data Layer — Local Database Entity

```kotlin
// data/local/entity/UserEntity.kt
@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey val id: String,
    val name: String,
    val email: String,
    val avatarUrl: String?,
    val lastUpdated: Long = System.currentTimeMillis(),
)
```

#### 7. Data Layer — DAO

```kotlin
// data/local/dao/UserDao.kt
@Dao
interface UserDao {

    @Query("SELECT * FROM users WHERE id = :id")
    suspend fun getUserById(id: String): UserEntity?

    @Query("SELECT * FROM users WHERE id = :id")
    fun observeUserById(id: String): Flow<UserEntity>

    @Upsert
    suspend fun upsertUser(user: UserEntity)
}
```

#### 8. Data Layer — Mapper

```kotlin
// data/mapper/UserMapper.kt
class UserMapper @Inject constructor() {

    fun toDomain(entity: UserEntity): User {
        return User(
            id = entity.id,
            name = entity.name,
            email = entity.email,
            avatarUrl = entity.avatarUrl,
        )
    }

    fun toEntity(response: UserResponse): UserEntity {
        return UserEntity(
            id = response.id,
            name = response.name,
            email = response.email,
            avatarUrl = response.avatarUrl,
        )
    }

    fun toDomain(response: UserResponse): User {
        return User(
            id = response.id,
            name = response.name,
            email = response.email,
            avatarUrl = response.avatarUrl,
        )
    }
}
```

#### 9. Data Layer — Repository Implementation

```kotlin
// data/repository/UserRepositoryImpl.kt
class UserRepositoryImpl @Inject constructor(
    private val api: UserApi,
    private val dao: UserDao,
    private val mapper: UserMapper,
    @IoDispatcher private val ioDispatcher: CoroutineDispatcher,
) : UserRepository {

    override suspend fun getUser(id: String): Result<User> {
        return withContext(ioDispatcher) {
            runCatching {
                val response = api.fetchUser(id)
                val entity = mapper.toEntity(response)
                dao.upsertUser(entity)
                mapper.toDomain(response)
            }
        }
    }

    override suspend fun updateUser(id: String, name: String): Result<User> {
        return withContext(ioDispatcher) {
            runCatching {
                val response = api.updateUser(id, UpdateUserRequest(name))
                val entity = mapper.toEntity(response)
                dao.upsertUser(entity)
                mapper.toDomain(response)
            }
        }
    }

    override fun observeUser(id: String): Flow<User> {
        return dao.observeUserById(id).map { mapper.toDomain(it) }
    }
}
```

#### 10. DI — Hilt Modules

```kotlin
// di/NetworkModule.kt
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Singleton
    @Provides
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(HttpLoggingInterceptor().apply {
                level = HttpLoggingInterceptor.Level.BODY
            })
            .build()
    }

    @Singleton
    @Provides
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BuildConfig.BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(Json.asConverterFactory("application/json".toMediaType()))
            .build()
    }

    @Provides
    fun provideUserApi(retrofit: Retrofit): UserApi {
        return retrofit.create(UserApi::class.java)
    }
}

// di/DatabaseModule.kt
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Singleton
    @Provides
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase {
        return Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "arclabs_database",
        ).build()
    }

    @Provides
    fun provideUserDao(database: AppDatabase): UserDao {
        return database.userDao()
    }
}

// di/RepositoryModule.kt
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}
```

#### 11. Presentation Layer — ViewModel

```kotlin
// presentation/user/UserViewModel.kt
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase,
    private val updateUserUseCase: UpdateUserUseCase,
    savedStateHandle: SavedStateHandle,
) : ViewModel() {

    private val userId: String = checkNotNull(savedStateHandle["userId"])

    private val _uiState = MutableStateFlow<UserUiState>(UserUiState.Loading)
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()

    init {
        loadUser()
    }

    fun loadUser() {
        viewModelScope.launch {
            _uiState.value = UserUiState.Loading
            getUserUseCase(userId)
                .onSuccess { _uiState.value = UserUiState.Success(it) }
                .onFailure { _uiState.value = UserUiState.Error(it.message.orEmpty()) }
        }
    }

    fun updateUserName(newName: String) {
        viewModelScope.launch {
            _uiState.value = UserUiState.Loading
            updateUserUseCase(userId, newName)
                .onSuccess { _uiState.value = UserUiState.Success(it) }
                .onFailure { _uiState.value = UserUiState.Error(it.message.orEmpty()) }
        }
    }
}
```

#### 12. Presentation Layer — Composable Screen

```kotlin
// presentation/user/UserScreen.kt
@Composable
fun UserScreen(
    viewModel: UserViewModel = hiltViewModel(),
    onNavigateBack: () -> Unit,
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("User Profile") },
                navigationIcon = {
                    IconButton(onClick = onNavigateBack) {
                        Icon(Icons.AutoMirrored.Filled.ArrowBack, contentDescription = "Back")
                    }
                },
            )
        },
    ) { padding ->
        Box(
            modifier = Modifier
                .fillMaxSize()
                .padding(padding),
            contentAlignment = Alignment.Center,
        ) {
            when (val state = uiState) {
                is UserUiState.Loading -> CircularProgressIndicator()
                is UserUiState.Success -> UserContent(user = state.user)
                is UserUiState.Error -> ErrorContent(
                    message = state.message,
                    onRetry = viewModel::loadUser,
                )
            }
        }
    }
}
```

---

## 🧪 Testing with Hilt

**Hilt provides first-class testing support. You can replace real modules with test modules that provide fakes or mocks, ensuring isolated and deterministic tests.**

---

### Unit Testing ViewModels (No Hilt Needed)

For pure unit tests, you do not need Hilt at all. Simply construct the ViewModel with fake dependencies:

```kotlin
class UserViewModelTest {

    private lateinit var fakeUserRepository: FakeUserRepository
    private lateinit var getUserUseCase: GetUserUseCase
    private lateinit var updateUserUseCase: UpdateUserUseCase
    private lateinit var viewModel: UserViewModel

    @BeforeTest
    fun setup() {
        fakeUserRepository = FakeUserRepository()
        getUserUseCase = GetUserUseCase(fakeUserRepository)
        updateUserUseCase = UpdateUserUseCase(fakeUserRepository)
        viewModel = UserViewModel(
            getUserUseCase = getUserUseCase,
            updateUserUseCase = updateUserUseCase,
            savedStateHandle = SavedStateHandle(mapOf("userId" to "user-1")),
        )
    }

    @Test
    fun `loadUser emits success state when repository returns user`() = runTest {
        // Given
        val expectedUser = User(
            id = "user-1",
            name = "John Doe",
            email = "john@example.com",
            avatarUrl = null,
        )
        fakeUserRepository.setUser(expectedUser)

        // When
        viewModel.loadUser()

        // Then
        val state = viewModel.uiState.value
        assertIs<UserUiState.Success>(state)
        assertEquals(expectedUser, state.user)
    }

    @Test
    fun `loadUser emits error state when repository fails`() = runTest {
        // Given
        fakeUserRepository.setShouldFail(true)

        // When
        viewModel.loadUser()

        // Then
        val state = viewModel.uiState.value
        assertIs<UserUiState.Error>(state)
    }
}
```

---

### Fake Repository for Testing

```kotlin
class FakeUserRepository : UserRepository {

    private var user: User? = null
    private var shouldFail: Boolean = false

    fun setUser(user: User) {
        this.user = user
    }

    fun setShouldFail(shouldFail: Boolean) {
        this.shouldFail = shouldFail
    }

    override suspend fun getUser(id: String): Result<User> {
        if (shouldFail) return Result.failure(RuntimeException("Test error"))
        return user?.let { Result.success(it) }
            ?: Result.failure(NoSuchElementException("User not found"))
    }

    override suspend fun updateUser(id: String, name: String): Result<User> {
        if (shouldFail) return Result.failure(RuntimeException("Test error"))
        val updated = user?.copy(name = name) ?: return Result.failure(
            NoSuchElementException("User not found")
        )
        this.user = updated
        return Result.success(updated)
    }

    override fun observeUser(id: String): Flow<User> {
        return flow {
            user?.let { emit(it) }
        }
    }
}
```

---

### Instrumented Testing with Hilt

For Android instrumented tests that require the full DI graph (e.g., integration tests):

```kotlin
@HiltAndroidTest
@UninstallModules(RepositoryModule::class)
class UserFeatureIntegrationTest {

    @get:Rule
    val hiltRule = HiltAndroidRule(this)

    @get:Rule
    val composeTestRule = createAndroidComposeRule<MainActivity>()

    @Inject
    lateinit var fakeUserRepository: FakeUserRepository

    @Module
    @InstallIn(SingletonComponent::class)
    abstract class TestRepositoryModule {

        @Binds
        @Singleton
        abstract fun bindUserRepository(
            impl: FakeUserRepository,
        ): UserRepository
    }

    @Before
    fun setup() {
        hiltRule.inject()
    }

    @Test
    fun userScreen_displaysUserName_whenLoadSucceeds() {
        // Given
        val testUser = User(
            id = "user-1",
            name = "Jane Doe",
            email = "jane@example.com",
            avatarUrl = null,
        )
        fakeUserRepository.setUser(testUser)

        // When
        composeTestRule.onNodeWithText("Jane Doe").assertIsDisplayed()
    }

    @Test
    fun userScreen_displaysError_whenLoadFails() {
        // Given
        fakeUserRepository.setShouldFail(true)

        // Then
        composeTestRule.onNodeWithText("Error").assertIsDisplayed()
    }
}
```

---

### Test Runner Configuration

Add the Hilt test runner to your `build.gradle.kts`:

```kotlin
android {
    defaultConfig {
        testInstrumentationRunner = "com.arclabs.myapp.HiltTestRunner"
    }
}
```

```kotlin
// HiltTestRunner.kt
class HiltTestRunner : AndroidJUnitRunner() {

    override fun newApplication(
        cl: ClassLoader?,
        className: String?,
        context: Context?,
    ): Application {
        return super.newApplication(cl, HiltTestApplication::class.java.name, context)
    }
}
```

---

## ✅ DI Checklist

**Use this checklist before submitting any PR that modifies DI configuration.**

---

- [ ] **Application class** is annotated with `@HiltAndroidApp`
- [ ] **Activities and Fragments** that need injection use `@AndroidEntryPoint`
- [ ] **ViewModels** are annotated with `@HiltViewModel` and use `@Inject constructor`
- [ ] **Constructor injection** is used everywhere possible (no unnecessary field injection)
- [ ] **Modules** are installed in the correct component (`@InstallIn`)
- [ ] **`@Binds`** is used for interface-to-implementation bindings
- [ ] **`@Provides`** is used only for types you do not own
- [ ] **Scopes** are applied only when necessary (not everything is `@Singleton`)
- [ ] **Qualifiers** are used when multiple bindings exist for the same type
- [ ] **No service locator patterns** exist anywhere in the codebase
- [ ] **Test modules** exist for all repository and data source bindings
- [ ] **Fake implementations** are provided for all interfaces used in tests
- [ ] **No circular dependencies** exist in the dependency graph
- [ ] **KSP** is used instead of KAPT for annotation processing
- [ ] **Build compiles successfully** with no Hilt/Dagger errors

---

## 🚫 Common Mistakes

**These are the five most frequent DI mistakes in Android projects. Learn to recognize and avoid them.**

---

### Mistake 1: Field Injection Where Constructor Injection Is Possible

❌ **Wrong:**

```kotlin
class UserRepository {
    @Inject
    lateinit var api: UserApi

    @Inject
    lateinit var dao: UserDao

    suspend fun getUser(id: String): User {
        return api.fetchUser(id).toDomain()
    }
}
```

✅ **Correct:**

```kotlin
class UserRepository @Inject constructor(
    private val api: UserApi,
    private val dao: UserDao,
) {
    suspend fun getUser(id: String): User {
        return api.fetchUser(id).toDomain()
    }
}
```

**Why:** Constructor injection makes dependencies explicit, immutable (`val` instead of `lateinit var`), and easier to test. Field injection hides dependencies and allows partially initialized objects.

---

### Mistake 2: Installing Module in the Wrong Component

❌ **Wrong:**

```kotlin
// ViewModel-scoped repository installed in SingletonComponent — scope mismatch!
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @ViewModelScoped  // ERROR: ViewModelScoped cannot be used in SingletonComponent
    @Binds
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}
```

✅ **Correct:**

```kotlin
// Scope and component must match
@Module
@InstallIn(ViewModelComponent::class)
abstract class RepositoryModule {

    @ViewModelScoped
    @Binds
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}

// OR — unscoped in SingletonComponent (usually the right choice for repositories)
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}
```

**Why:** Each scope annotation is only valid for its corresponding component. `@ViewModelScoped` can only be used in `ViewModelComponent`. `@Singleton` can only be used in `SingletonComponent`. Mismatches cause compile errors.

---

### Mistake 3: Missing @Inject on Constructor

❌ **Wrong:**

```kotlin
// Hilt does not know how to create this class!
class UserMapper {
    fun toDomain(entity: UserEntity): User {
        return User(id = entity.id, name = entity.name, email = entity.email)
    }
}
```

Build error:
```
error: [Dagger/MissingBinding] com.arclabs.data.mapper.UserMapper cannot be provided
without an @Inject constructor or an @Provides-annotated method.
```

✅ **Correct:**

```kotlin
class UserMapper @Inject constructor() {
    fun toDomain(entity: UserEntity): User {
        return User(id = entity.id, name = entity.name, email = entity.email)
    }
}
```

**Why:** Hilt needs either an `@Inject` constructor or a `@Provides` method to know how to create instances. Even classes with no-argument constructors need the annotation.

---

### Mistake 4: Scoping Everything as Singleton

❌ **Wrong:**

```kotlin
@Singleton
class UserMapper @Inject constructor() { ... }

@Singleton
class GetUserUseCase @Inject constructor(private val repo: UserRepository) { ... }

@Singleton
class UpdateUserUseCase @Inject constructor(private val repo: UserRepository) { ... }

@Singleton
class ValidateEmailUseCase @Inject constructor() { ... }

@Singleton
class FormatDateUseCase @Inject constructor() { ... }
```

✅ **Correct:**

```kotlin
// These are all stateless — no scope needed
class UserMapper @Inject constructor() { ... }

class GetUserUseCase @Inject constructor(private val repo: UserRepository) { ... }

class UpdateUserUseCase @Inject constructor(private val repo: UserRepository) { ... }

class ValidateEmailUseCase @Inject constructor() { ... }

class FormatDateUseCase @Inject constructor() { ... }
```

**Why:** Scoping every class as `@Singleton` means those objects live for the entire application lifetime, consuming memory unnecessarily. Stateless classes should be unscoped so they are garbage collected when no longer needed. Only scope objects that hold shared mutable state or are expensive to create.

---

### Mistake 5: Circular Dependencies

❌ **Wrong:**

```kotlin
class UserRepository @Inject constructor(
    private val authRepository: AuthRepository,  // depends on AuthRepository
) : UserRepository { ... }

class AuthRepository @Inject constructor(
    private val userRepository: UserRepository,  // depends on UserRepository — CIRCULAR!
) : AuthRepository { ... }
```

Build error:
```
error: [Dagger/DependencyCycle] Found a dependency cycle:
    com.arclabs.data.UserRepository is injected at
    com.arclabs.data.AuthRepository(userRepository)
    com.arclabs.data.AuthRepository is injected at
    com.arclabs.data.UserRepository(authRepository)
```

✅ **Correct — break the cycle with an intermediary:**

```kotlin
class UserRepository @Inject constructor(
    private val userApi: UserApi,
    private val authTokenProvider: AuthTokenProvider,  // narrow dependency
) : UserRepository { ... }

class AuthRepository @Inject constructor(
    private val authApi: AuthApi,
) : AuthRepository { ... }

// Extract shared logic into a focused class
class AuthTokenProvider @Inject constructor(
    private val tokenStore: TokenStore,
) {
    suspend fun getToken(): String = tokenStore.getCurrentToken()
}
```

**Why:** Circular dependencies indicate a design problem. Break the cycle by extracting the shared concern into a separate class, applying the Interface Segregation Principle, or restructuring responsibilities so the dependency flows in one direction.

---

## 📚 Further Reading

**Official documentation and references for deeper understanding of Hilt and Dependency Injection on Android.**

---

### Official Documentation

- [Hilt Official Guide](https://developer.android.com/training/dependency-injection/hilt-android) — Android developer documentation for Hilt
- [Dependency Injection on Android](https://developer.android.com/training/dependency-injection) — Overview of DI concepts on Android
- [Hilt and Jetpack Integration](https://developer.android.com/training/dependency-injection/hilt-jetpack) — Using Hilt with ViewModel, WorkManager, Navigation
- [Hilt Testing Guide](https://developer.android.com/training/dependency-injection/hilt-testing) — Official testing guide for Hilt

### Dagger Documentation

- [Dagger 2 User's Guide](https://dagger.dev/dev-guide/) — Underlying Dagger concepts that power Hilt
- [Dagger Component Hierarchy](https://dagger.dev/hilt/components.html) — Detailed component hierarchy reference

### Architecture References

- [Guide to App Architecture](https://developer.android.com/topic/architecture) — Google's recommended app architecture
- [Now in Android](https://github.com/android/nowinandroid) — Google's sample app demonstrating Hilt with Clean Architecture

### ARC Labs Internal References

- [ARC Labs Clean Architecture Guide](../Architecture/clean-architecture.md) — ARC Labs Clean Architecture standards
- [ARC Labs Testing Guide](../Quality/testing.md) — ARC Labs testing standards for Android
- [ARC Labs Android Standards](../README.md) — Root documentation for ARC Labs Android standards

---

> **Last updated:** February 2026
>
> **Maintained by:** ARC Labs Studio
>
> **Applies to:** All ARC Labs Android projects using Hilt
