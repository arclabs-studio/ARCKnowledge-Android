# 📱 Android Apps

**ARC Labs Android apps follow Clean Architecture with MVVM, Jetpack Compose, Material Design 3, and Hilt dependency injection.**

---

## 🎯 App Philosophy

### Core Principles

1. **User First** - Every feature serves the user, not the developer
2. **Offline Capable** - Core features work without a network connection
3. **Performant** - 60fps scrolling, fast startup, smooth animations
4. **Accessible** - TalkBack support, content descriptions, dynamic text sizing
5. **Testable** - 80%+ code coverage for apps

### App Goals

- Simple, intuitive interfaces that feel native to Android
- Material Design 3 with dynamic color (Material You)
- Full dark and light theme support
- Localized for multiple languages
- Edge-to-edge design with proper inset handling
- Predictive back gesture support

---

## 📂 Current ARC Labs Android Projects

| App | Status | Description |
|-----|--------|-------------|
| FavRes-Android | Planned | Favorite restaurants tracker |
| FavBook-Android | Planned | Book collection manager |
| TicketMind-Android | Planned | Event ticket organizer |

All apps share common libraries (ARCDesignSystem, ARCLogger-Android, ARCNetworking-Android, ARCStorage-Android) and follow identical architecture patterns.

---

## 📁 App Structure

### Standard Directory Layout

```
app/
├── src/
│   ├── main/
│   │   ├── java/com/arclabs/myapp/
│   │   │   ├── MyApplication.kt
│   │   │   ├── MainActivity.kt
│   │   │   ├── navigation/
│   │   │   │   ├── AppNavHost.kt
│   │   │   │   ├── Routes.kt
│   │   │   │   └── TopLevelDestination.kt
│   │   │   ├── di/
│   │   │   │   ├── AppModule.kt
│   │   │   │   ├── NetworkModule.kt
│   │   │   │   ├── DatabaseModule.kt
│   │   │   │   └── RepositoryModule.kt
│   │   │   ├── theme/
│   │   │   │   ├── Color.kt
│   │   │   │   ├── Theme.kt
│   │   │   │   ├── Type.kt
│   │   │   │   └── Shape.kt
│   │   │   └── feature/
│   │   │       ├── home/
│   │   │       │   ├── presentation/
│   │   │       │   ├── domain/
│   │   │       │   └── data/
│   │   │       ├── profile/
│   │   │       │   ├── presentation/
│   │   │       │   ├── domain/
│   │   │       │   └── data/
│   │   │       └── settings/
│   │   │           ├── presentation/
│   │   │           ├── domain/
│   │   │           └── data/
│   │   ├── res/
│   │   │   ├── values/
│   │   │   │   ├── strings.xml
│   │   │   │   ├── colors.xml
│   │   │   │   └── themes.xml
│   │   │   ├── values-night/
│   │   │   │   └── themes.xml
│   │   │   ├── drawable/
│   │   │   ├── mipmap-xxxhdpi/
│   │   │   └── xml/
│   │   │       └── backup_rules.xml
│   │   └── AndroidManifest.xml
│   ├── test/
│   │   └── java/com/arclabs/myapp/
│   │       └── feature/
│   │           ├── home/
│   │           │   ├── domain/
│   │           │   └── presentation/
│   │           └── profile/
│   └── androidTest/
│       └── java/com/arclabs/myapp/
│           └── feature/
│               └── home/
├── build.gradle.kts
├── proguard-rules.pro
└── consumer-rules.pro
```

### Feature Organization

Each feature follows Clean Architecture within its package:

```
feature/home/
├── presentation/
│   ├── HomeScreen.kt          # Composable screen
│   ├── HomeViewModel.kt       # ViewModel with StateFlow
│   ├── HomeUiState.kt         # UI state sealed interface
│   ├── HomeUiEvent.kt         # One-shot UI events
│   └── components/
│       ├── RestaurantCard.kt  # Reusable composable
│       ├── SearchBar.kt       # Search composable
│       └── FilterChips.kt     # Filter composable
├── domain/
│   ├── entities/
│   │   └── Restaurant.kt      # Domain model
│   ├── usecases/
│   │   ├── GetRestaurantsUseCase.kt
│   │   └── SearchRestaurantsUseCase.kt
│   └── repositories/
│       └── RestaurantRepository.kt  # Interface only
└── data/
    ├── repositories/
    │   └── RestaurantRepositoryImpl.kt
    ├── datasources/
    │   ├── local/
    │   │   ├── RestaurantDao.kt
    │   │   └── RestaurantEntity.kt
    │   └── remote/
    │       ├── RestaurantApiService.kt
    │       └── RestaurantDto.kt
    └── mappers/
        ├── RestaurantEntityMapper.kt
        └── RestaurantDtoMapper.kt
```

---

## 📦 Library Dependencies

### Required Dependencies

| Library | Purpose | Version |
|---------|---------|---------|
| Jetpack Compose | Declarative UI framework | BOM 2024.12.01 |
| Material3 | Material Design 3 components | via Compose BOM |
| Hilt | Dependency injection | 2.51.1 |
| Navigation Compose | Type-safe navigation | 2.8.5 |
| Lifecycle Runtime Compose | Lifecycle-aware Compose state | 2.8.7 |
| Lifecycle ViewModel Compose | ViewModel integration | 2.8.7 |
| Kotlinx Coroutines | Asynchronous programming | 1.9.0 |
| Kotlinx Serialization | JSON and route serialization | 1.7.3 |

### Optional Dependencies

| Library | Purpose | When to Use |
|---------|---------|-------------|
| Retrofit + OkHttp | HTTP networking | API calls |
| Room | SQLite database | Local persistence |
| DataStore | Preferences storage | Settings, simple key-value state |
| Coil | Image loading and caching | Remote or local images |
| Paging 3 | Pagination | Large lists from API or DB |
| WorkManager | Background work | Sync, uploads, scheduled tasks |
| Accompanist | Compose utilities | Permissions, system UI |

### build.gradle.kts Dependencies Block

```kotlin
dependencies {
    // Compose BOM
    val composeBom = platform("androidx.compose:compose-bom:2024.12.01")
    implementation(composeBom)
    androidTestImplementation(composeBom)

    // Compose
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    debugImplementation("androidx.compose.ui:ui-tooling")
    debugImplementation("androidx.compose.ui:ui-test-manifest")

    // Navigation
    implementation("androidx.navigation:navigation-compose:2.8.5")

    // Lifecycle
    implementation("androidx.lifecycle:lifecycle-runtime-compose:2.8.7")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.7")

    // Hilt
    implementation("com.google.dagger:hilt-android:2.51.1")
    ksp("com.google.dagger:hilt-android-compiler:2.51.1")
    implementation("androidx.hilt:hilt-navigation-compose:1.2.0")

    // Kotlinx Serialization
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")

    // Testing
    testImplementation("org.junit.jupiter:junit-jupiter:5.10.3") // # verify latest
    testImplementation("io.mockk:mockk:1.13.13")
    testImplementation("app.cash.turbine:turbine:1.2.0")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.9.0")
    androidTestImplementation("androidx.compose.ui:ui-test-junit4")
}
```

---

## 🏗️ App Architecture

### Clean Architecture Layers

```
┌──────────────────────────────────────────────────────┐
│                   Presentation Layer                  │
│  ┌────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  Screens   │  │  ViewModels  │  │  UI State    │ │
│  │ (Compose)  │  │ (Hilt)       │  │  (data class)│ │
│  └────────────┘  └──────────────┘  └──────────────┘ │
│                         │                             │
│                         ▼                             │
├──────────────────────────────────────────────────────┤
│                    Domain Layer                       │
│  ┌────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  Entities  │  │  Use Cases   │  │  Repository  │ │
│  │(data class)│  │ (invoke())   │  │ (interface)  │ │
│  └────────────┘  └──────────────┘  └──────────────┘ │
│                         ▲                             │
│                         │                             │
├──────────────────────────────────────────────────────┤
│                     Data Layer                        │
│  ┌────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  Repo Impl │  │ Data Sources │  │    DTOs /    │ │
│  │            │  │ (local+remote│  │   Entities   │ │
│  └────────────┘  └──────────────┘  └──────────────┘ │
└──────────────────────────────────────────────────────┘
```

### Dependency Rule

Dependencies point INWARD only:

- **Presentation** depends on **Domain** (never Data)
- **Data** depends on **Domain** (implements repository interfaces)
- **Domain** depends on **nothing** (pure Kotlin, no Android imports)

### Dependency Flow with Hilt

```kotlin
// Domain layer: interface
interface RestaurantRepository {
    fun getRestaurants(): Flow<List<Restaurant>>
}

// Data layer: implementation
class RestaurantRepositoryImpl @Inject constructor(
    private val dao: RestaurantDao,
    private val api: RestaurantApiService,
) : RestaurantRepository {
    override fun getRestaurants(): Flow<List<Restaurant>> = // ...
}

// DI module: binding
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Binds
    abstract fun bindRestaurantRepository(
        impl: RestaurantRepositoryImpl
    ): RestaurantRepository
}

// Presentation: consumption
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getRestaurants: GetRestaurantsUseCase,
) : ViewModel() {
    // ...
}
```

---

## 🚀 App Initialization

### Application Class

```kotlin
@HiltAndroidApp
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        // Initialize libraries that need Application context
        // e.g., Timber, Coil defaults, etc.
    }
}
```

### Main Activity

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Enable edge-to-edge display
        enableEdgeToEdge()

        setContent {
            AppTheme {
                AppNavHost()
            }
        }
    }
}
```

### Navigation Setup

```kotlin
@Composable
fun AppNavHost(
    navController: NavHostController = rememberNavController(),
) {
    NavHost(
        navController = navController,
        startDestination = Routes.Home,
    ) {
        composable<Routes.Home> {
            HomeScreen(
                onRestaurantClick = { id ->
                    navController.navigate(Routes.RestaurantDetail(id))
                },
            )
        }
        composable<Routes.RestaurantDetail> {
            RestaurantDetailScreen(
                onBackClick = { navController.popBackStack() },
            )
        }
        composable<Routes.Settings> {
            SettingsScreen()
        }
    }
}
```

### Route Definitions

```kotlin
@Serializable
sealed interface Routes {
    @Serializable
    data object Home : Routes

    @Serializable
    data class RestaurantDetail(val id: String) : Routes

    @Serializable
    data object Settings : Routes
}
```

---

## ⚙️ Feature Implementation Pattern

### Step 1: Domain Entity

```kotlin
// domain/entities/Restaurant.kt
data class Restaurant(
    val id: String,
    val name: String,
    val cuisine: String,
    val rating: Double,
    val priceRange: PriceRange,
    val imageUrl: String?,
    val isFavorite: Boolean,
)

enum class PriceRange {
    Budget,
    Moderate,
    Expensive,
    Luxury,
}
```

### Step 2: Repository Interface

```kotlin
// domain/repositories/RestaurantRepository.kt
interface RestaurantRepository {
    fun getRestaurants(): Flow<List<Restaurant>>
    fun getRestaurant(id: String): Flow<Restaurant>
    suspend fun toggleFavorite(id: String)
    fun searchRestaurants(query: String): Flow<List<Restaurant>>
}
```

### Step 3: Use Case

```kotlin
// domain/usecases/GetRestaurantsUseCase.kt
class GetRestaurantsUseCase @Inject constructor(
    private val repository: RestaurantRepository,
) {
    operator fun invoke(): Flow<List<Restaurant>> {
        return repository.getRestaurants()
    }
}
```

### Step 4: Repository Implementation

```kotlin
// data/repositories/RestaurantRepositoryImpl.kt
class RestaurantRepositoryImpl @Inject constructor(
    private val dao: RestaurantDao,
    private val api: RestaurantApiService,
    private val entityMapper: RestaurantEntityMapper,
    private val dtoMapper: RestaurantDtoMapper,
) : RestaurantRepository {

    override fun getRestaurants(): Flow<List<Restaurant>> {
        return dao.getAll().map { entities ->
            entities.map { entityMapper.toDomain(it) }
        }.onStart {
            refreshFromRemote()
        }
    }

    private suspend fun refreshFromRemote() {
        try {
            val dtos = api.getRestaurants()
            val entities = dtos.map { dtoMapper.toEntity(it) }
            dao.insertAll(entities)
        } catch (e: Exception) {
            // Log error, data will be served from cache
        }
    }

    override fun getRestaurant(id: String): Flow<Restaurant> {
        return dao.getById(id).map { entityMapper.toDomain(it) }
    }

    override suspend fun toggleFavorite(id: String) {
        dao.toggleFavorite(id)
    }

    override fun searchRestaurants(query: String): Flow<List<Restaurant>> {
        return dao.searchByName(query).map { entities ->
            entities.map { entityMapper.toDomain(it) }
        }
    }
}
```

### Step 5: UI State

```kotlin
// presentation/HomeUiState.kt
sealed interface HomeUiState {
    data object Loading : HomeUiState
    data class Success(
        val restaurants: List<Restaurant>,
        val searchQuery: String = "",
        val isSearching: Boolean = false,
    ) : HomeUiState
    data class Error(val message: String) : HomeUiState
}
```

### Step 6: ViewModel

```kotlin
// presentation/HomeViewModel.kt
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getRestaurants: GetRestaurantsUseCase,
    private val searchRestaurants: SearchRestaurantsUseCase,
) : ViewModel() {

    private val _uiState = MutableStateFlow<HomeUiState>(HomeUiState.Loading)
    val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()

    private val searchQuery = MutableStateFlow("")

    init {
        loadRestaurants()
        observeSearch()
    }

    private fun loadRestaurants() {
        viewModelScope.launch {
            getRestaurants()
                .catch { e ->
                    _uiState.value = HomeUiState.Error(
                        e.message ?: "Unknown error"
                    )
                }
                .collect { restaurants ->
                    _uiState.value = HomeUiState.Success(
                        restaurants = restaurants,
                    )
                }
        }
    }

    private fun observeSearch() {
        viewModelScope.launch {
            searchQuery
                .debounce(300)
                .distinctUntilChanged()
                .flatMapLatest { query ->
                    if (query.isBlank()) {
                        getRestaurants()
                    } else {
                        searchRestaurants(query)
                    }
                }
                .collect { restaurants ->
                    _uiState.update { current ->
                        when (current) {
                            is HomeUiState.Success -> current.copy(
                                restaurants = restaurants,
                                isSearching = false,
                            )
                            else -> HomeUiState.Success(
                                restaurants = restaurants,
                            )
                        }
                    }
                }
        }
    }

    fun onSearchQueryChanged(query: String) {
        searchQuery.value = query
        _uiState.update { current ->
            when (current) {
                is HomeUiState.Success -> current.copy(
                    searchQuery = query,
                    isSearching = true,
                )
                else -> current
            }
        }
    }
}
```

### Step 7: Screen Composable

```kotlin
// presentation/HomeScreen.kt
@Composable
fun HomeScreen(
    onRestaurantClick: (String) -> Unit,
    viewModel: HomeViewModel = hiltViewModel(),
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    HomeScreenContent(
        uiState = uiState,
        onSearchQueryChanged = viewModel::onSearchQueryChanged,
        onRestaurantClick = onRestaurantClick,
    )
}

@Composable
private fun HomeScreenContent(
    uiState: HomeUiState,
    onSearchQueryChanged: (String) -> Unit,
    onRestaurantClick: (String) -> Unit,
) {
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Restaurants") },
            )
        },
    ) { padding ->
        when (uiState) {
            is HomeUiState.Loading -> {
                Box(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(padding),
                    contentAlignment = Alignment.Center,
                ) {
                    CircularProgressIndicator()
                }
            }
            is HomeUiState.Success -> {
                Column(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(padding),
                ) {
                    SearchBar(
                        query = uiState.searchQuery,
                        onQueryChange = onSearchQueryChanged,
                        isSearching = uiState.isSearching,
                    )
                    LazyColumn(
                        contentPadding = PaddingValues(16.dp),
                        verticalArrangement = Arrangement.spacedBy(12.dp),
                    ) {
                        items(
                            items = uiState.restaurants,
                            key = { it.id },
                        ) { restaurant ->
                            RestaurantCard(
                                restaurant = restaurant,
                                onClick = { onRestaurantClick(restaurant.id) },
                            )
                        }
                    }
                }
            }
            is HomeUiState.Error -> {
                Box(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(padding),
                    contentAlignment = Alignment.Center,
                ) {
                    Text(
                        text = uiState.message,
                        style = MaterialTheme.typography.bodyLarge,
                        color = MaterialTheme.colorScheme.error,
                    )
                }
            }
        }
    }
}
```

---

## 🎨 Theme Configuration

### Color Scheme

```kotlin
// theme/Color.kt
val LightColorScheme = lightColorScheme(
    primary = Color(0xFF6750A4),
    onPrimary = Color(0xFFFFFFFF),
    primaryContainer = Color(0xFFEADDFF),
    onPrimaryContainer = Color(0xFF21005E),
    secondary = Color(0xFF625B71),
    onSecondary = Color(0xFFFFFFFF),
    background = Color(0xFFFFFBFE),
    onBackground = Color(0xFF1C1B1F),
    surface = Color(0xFFFFFBFE),
    onSurface = Color(0xFF1C1B1F),
    error = Color(0xFFB3261E),
    onError = Color(0xFFFFFFFF),
)

val DarkColorScheme = darkColorScheme(
    primary = Color(0xFFD0BCFF),
    onPrimary = Color(0xFF381E72),
    primaryContainer = Color(0xFF4F378B),
    onPrimaryContainer = Color(0xFFEADDFF),
    secondary = Color(0xFFCCC2DC),
    onSecondary = Color(0xFF332D41),
    background = Color(0xFF1C1B1F),
    onBackground = Color(0xFFE6E1E5),
    surface = Color(0xFF1C1B1F),
    onSurface = Color(0xFFE6E1E5),
    error = Color(0xFFF2B8B5),
    onError = Color(0xFF601410),
)
```

### Theme Composable

```kotlin
// theme/Theme.kt
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit,
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context)
            else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = AppTypography,
        shapes = AppShapes,
        content = content,
    )
}
```

---

## 🧪 Testing Strategy

### Coverage Requirements

- **App modules**: 80%+ code coverage
- **Library modules**: 100% code coverage
- Focus on ViewModel and Use Case testing

### What to Test

| Layer | What | How | Priority |
|-------|------|-----|----------|
| Domain | Use Cases | Unit test with MockK | High |
| Domain | Entities | Unit test (if logic exists) | Medium |
| Presentation | ViewModels | Unit test with Turbine | High |
| Presentation | UI State mapping | Unit test | Medium |
| Presentation | Composables | Compose UI test | Low-Medium |
| Data | Repositories | Integration test with fakes | Medium |
| Data | DAOs | Instrumented test with Room | Medium |
| Data | Mappers | Unit test | High |

### Example ViewModel Test

```kotlin
class HomeViewModelTest {

    private val getRestaurants = mockk<GetRestaurantsUseCase>()
    private val searchRestaurants = mockk<SearchRestaurantsUseCase>()

    private fun makeSUT(): HomeViewModel {
        return HomeViewModel(
            getRestaurants = getRestaurants,
            searchRestaurants = searchRestaurants,
        )
    }

    @Test
    fun `initial state is loading`() = runTest {
        // Given
        every { getRestaurants() } returns flowOf(emptyList())
        every { searchRestaurants(any()) } returns flowOf(emptyList())

        // When
        val sut = makeSUT()

        // Then
        sut.uiState.test {
            assertThat(awaitItem()).isInstanceOf(HomeUiState.Loading::class.java)
            cancelAndIgnoreRemainingEvents()
        }
    }

    @Test
    fun `loads restaurants successfully`() = runTest {
        // Given
        val restaurants = listOf(
            Restaurant(
                id = "1",
                name = "Test Restaurant",
                cuisine = "Italian",
                rating = 4.5,
                priceRange = PriceRange.Moderate,
                imageUrl = null,
                isFavorite = false,
            ),
        )
        every { getRestaurants() } returns flowOf(restaurants)
        every { searchRestaurants(any()) } returns flowOf(emptyList())

        // When
        val sut = makeSUT()

        // Then
        sut.uiState.test {
            skipItems(1) // Skip Loading
            val state = awaitItem()
            assertThat(state).isInstanceOf(HomeUiState.Success::class.java)
            assertThat((state as HomeUiState.Success).restaurants).hasSize(1)
            cancelAndIgnoreRemainingEvents()
        }
    }
}
```

### Example Use Case Test

```kotlin
class GetRestaurantsUseCaseTest {

    private val repository = mockk<RestaurantRepository>()

    private fun makeSUT(): GetRestaurantsUseCase {
        return GetRestaurantsUseCase(repository = repository)
    }

    @Test
    fun `returns restaurants from repository`() = runTest {
        // Given
        val expected = listOf(
            Restaurant(
                id = "1", name = "Test", cuisine = "Italian",
                rating = 4.0, priceRange = PriceRange.Moderate,
                imageUrl = null, isFavorite = false,
            ),
        )
        every { repository.getRestaurants() } returns flowOf(expected)
        val sut = makeSUT()

        // When
        val result = sut().first()

        // Then
        assertThat(result).isEqualTo(expected)
    }
}
```

---

## 💉 Dependency Injection with Hilt

### Module Organization

```kotlin
// di/AppModule.kt - Application-level singletons
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides
    @Singleton
    fun provideDatabase(
        @ApplicationContext context: Context,
    ): AppDatabase {
        return Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "app-database",
        ).build()
    }
}

// di/NetworkModule.kt - Network layer
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
    fun provideRetrofit(client: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.arclabs.com/")
            .client(client)
            .addConverterFactory(Json.asConverterFactory("application/json".toMediaType()))
            .build()
    }
}

// di/RepositoryModule.kt - Repository bindings
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    abstract fun bindRestaurantRepository(
        impl: RestaurantRepositoryImpl,
    ): RestaurantRepository
}

// di/DaoModule.kt - DAO providers
@Module
@InstallIn(SingletonComponent::class)
object DaoModule {

    @Provides
    fun provideRestaurantDao(database: AppDatabase): RestaurantDao {
        return database.restaurantDao()
    }
}
```

---

## ✅ App Checklist

### Code Quality

- [ ] No force casts (`as` without `?`)
- [ ] No hardcoded strings (use `strings.xml`)
- [ ] No hardcoded dimensions (use `Dp` values or dimension resources)
- [ ] No hardcoded colors (use `MaterialTheme.colorScheme`)
- [ ] No `!!` (non-null assertion operator)
- [ ] No `var` where `val` suffices
- [ ] No unused imports or variables
- [ ] ktlint and detekt pass with zero warnings

### Architecture

- [ ] Clean Architecture layers respected
- [ ] Domain layer has no Android imports
- [ ] All dependencies injected via Hilt
- [ ] Repository pattern used for data access
- [ ] Use Cases encapsulate business logic
- [ ] ViewModels use StateFlow (not LiveData)
- [ ] UI state modeled with sealed interface

### User Experience

- [ ] Dark theme tested and working
- [ ] Light theme tested and working
- [ ] Dynamic color supported (API 31+)
- [ ] TalkBack accessible
- [ ] Content descriptions on all interactive elements
- [ ] Loading states shown for async operations
- [ ] Error states with retry actions
- [ ] Empty states with helpful messages
- [ ] Edge-to-edge display with proper insets
- [ ] Predictive back gesture supported

### Performance

- [ ] LazyColumn/LazyRow used for lists (not Column/Row with scroll)
- [ ] `key` parameter provided for LazyColumn items
- [ ] Images loaded with Coil (not blocking main thread)
- [ ] Heavy operations on IO dispatcher
- [ ] No unnecessary recompositions (use `remember`, `derivedStateOf`)
- [ ] Startup time under 1 second

### Testing

- [ ] 80%+ code coverage
- [ ] ViewModel tests with Turbine
- [ ] Use Case tests with MockK
- [ ] Given-When-Then structure in all tests
- [ ] `makeSUT()` factory method used
- [ ] Edge cases covered

### Integration

- [ ] ARCDevTools lint rules pass
- [ ] CI pipeline passes
- [ ] ProGuard rules configured for release
- [ ] Version catalog used for dependency management

---

## 🚫 Common Mistakes

### 1. Business Logic in Composables

```kotlin
// ❌ BAD: Business logic in composable
@Composable
fun HomeScreen(viewModel: HomeViewModel = hiltViewModel()) {
    val restaurants = viewModel.restaurants
    val filtered = restaurants.filter { it.rating > 4.0 } // Business logic!
    // ...
}

// ✅ GOOD: Business logic in ViewModel or Use Case
@Composable
fun HomeScreen(viewModel: HomeViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    HomeScreenContent(uiState = uiState)
}
```

### 2. Not Using Sealed Interface for UI State

```kotlin
// ❌ BAD: Multiple state variables
@HiltViewModel
class HomeViewModel : ViewModel() {
    val isLoading = MutableStateFlow(false)
    val restaurants = MutableStateFlow<List<Restaurant>>(emptyList())
    val error = MutableStateFlow<String?>(null)
}

// ✅ GOOD: Sealed interface
sealed interface HomeUiState {
    data object Loading : HomeUiState
    data class Success(val restaurants: List<Restaurant>) : HomeUiState
    data class Error(val message: String) : HomeUiState
}
```

### 3. Missing Offline Support

```kotlin
// ❌ BAD: Only fetching from network
override fun getRestaurants(): Flow<List<Restaurant>> {
    return flow { emit(api.getRestaurants()) }
}

// ✅ GOOD: Cache-first with network refresh
override fun getRestaurants(): Flow<List<Restaurant>> {
    return dao.getAll()
        .map { it.map(entityMapper::toDomain) }
        .onStart { refreshFromRemote() }
}
```

### 4. Skipping Accessibility

```kotlin
// ❌ BAD: No content description
Icon(imageVector = Icons.Default.Favorite, contentDescription = null)

// ✅ GOOD: Meaningful content description
Icon(
    imageVector = Icons.Default.Favorite,
    contentDescription = stringResource(R.string.add_to_favorites),
)
```

### 5. Using LiveData Instead of StateFlow

```kotlin
// ❌ BAD: LiveData (old pattern)
val uiState: LiveData<HomeUiState> = _uiState

// ✅ GOOD: StateFlow (modern pattern)
val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()
```

### 6. Not Handling Configuration Changes

```kotlin
// ❌ BAD: Fetching data in LaunchedEffect
@Composable
fun HomeScreen() {
    var restaurants by remember { mutableStateOf(emptyList<Restaurant>()) }
    LaunchedEffect(Unit) {
        restaurants = api.getRestaurants() // Re-fetches on rotation!
    }
}

// ✅ GOOD: Data in ViewModel survives configuration changes
@HiltViewModel
class HomeViewModel @Inject constructor(
    getRestaurants: GetRestaurantsUseCase,
) : ViewModel() {
    val uiState = getRestaurants()
        .map<List<Restaurant>, HomeUiState> { HomeUiState.Success(it) }
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), HomeUiState.Loading)
}
```

---

## 📚 Further Reading

- [Clean Architecture](../Architecture/clean-architecture.md)
- [MVVM Pattern](../Architecture/mvvm.md)
- [Testing Standards](../Quality/testing.md)
- [UI Guidelines](../Quality/ui-guidelines.md)
- [Libraries](./libraries.md)
- [Hilt Dependency Injection](../Architecture/dependency-injection.md)

---

**Remember**: An app is the sum of its architecture, design, and attention to detail. Build it right.
