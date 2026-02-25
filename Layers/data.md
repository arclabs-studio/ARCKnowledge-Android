# 💾 Data Layer

**The Data Layer is the outermost layer of Clean Architecture. It provides concrete implementations of the repository interfaces defined in the Domain Layer, manages all data access (network, database, preferences), and handles the mapping between external data formats (DTOs, Room Entities) and Domain Entities. The Domain Layer never knows how data is fetched or stored -- that is entirely the Data Layer's responsibility.**

---

## 🎯 Purpose

The Data Layer exists to:

- **Implement data access and persistence** -- provide the concrete logic for fetching, storing, updating, and deleting data across all sources (remote APIs, local databases, preference stores).
- **Fulfill Domain repository contracts** -- implement the repository interfaces declared in the Domain Layer, ensuring the Domain never depends on infrastructure details.
- **Handle API communication** -- manage all networking concerns including Retrofit service definitions, OkHttp client configuration, request/response serialization, interceptors, and authentication headers.
- **Manage local database operations** -- configure Room databases, define DAOs for structured queries, handle migrations, and provide reactive data observation via Kotlin Flows.
- **Implement caching strategies** -- coordinate between remote and local data sources to provide offline-first behavior, cache invalidation, and optimistic updates.
- **Map DTOs to Domain entities** -- translate between the external data representations (JSON DTOs, Room Entities) and the clean Domain models, ensuring no serialization annotations or database annotations leak into the Domain.

---

## 📂 Directory Structure

```
data/
├── repositories/
│   ├── UserRepositoryImpl.kt
│   ├── RestaurantRepositoryImpl.kt
│   └── SettingsRepositoryImpl.kt
├── datasources/
│   ├── remote/
│   │   ├── UserRemoteDataSource.kt
│   │   ├── RestaurantRemoteDataSource.kt
│   │   └── api/
│   │       ├── UserApiService.kt
│   │       ├── RestaurantApiService.kt
│   │       └── interceptors/
│   │           ├── AuthInterceptor.kt
│   │           └── ErrorInterceptor.kt
│   └── local/
│       ├── UserLocalDataSource.kt
│       ├── RestaurantLocalDataSource.kt
│       ├── UserPreferencesDataSource.kt
│       ├── dao/
│       │   ├── UserDao.kt
│       │   └── RestaurantDao.kt
│       └── database/
│           ├── AppDatabase.kt
│           └── migrations/
│               └── Migration1To2.kt
├── models/
│   ├── dto/
│   │   ├── UserDto.kt
│   │   ├── RestaurantDto.kt
│   │   ├── SearchQueryDto.kt
│   │   └── PaginatedResponseDto.kt
│   └── entity/
│       ├── UserEntity.kt
│       └── RestaurantEntity.kt
├── mappers/
│   ├── UserMapper.kt
│   └── RestaurantMapper.kt
└── di/
    ├── NetworkModule.kt
    ├── DatabaseModule.kt
    ├── DataStoreModule.kt
    └── RepositoryModule.kt
```

### Key Conventions

- **`repositories/`** -- one implementation file per Domain repository interface.
- **`datasources/remote/`** -- Retrofit API service interfaces and their wrapper data source classes.
- **`datasources/local/`** -- Room DAOs, the database class, and preference data sources.
- **`models/dto/`** -- Kotlinx Serialization data classes representing JSON payloads from the API.
- **`models/entity/`** -- Room `@Entity` data classes representing database tables.
- **`mappers/`** -- extension functions for converting between DTOs, Entities, and Domain models.
- **`di/`** -- Hilt modules providing singletons and bindings for the Data Layer.

---

## 🌐 Remote Data Source (Retrofit + OkHttp)

The remote data source manages all communication with backend APIs. We use **Retrofit** for type-safe HTTP client generation and **OkHttp** for the underlying HTTP transport with interceptors.

### API Service Interface

Retrofit interfaces define the contract for each API endpoint. These are pure interfaces with suspend functions for coroutine support.

```kotlin
package com.arclabs.app.data.datasources.remote.api

import com.arclabs.app.data.models.dto.PaginatedResponseDto
import com.arclabs.app.data.models.dto.SearchQueryDto
import com.arclabs.app.data.models.dto.UserDto
import retrofit2.http.Body
import retrofit2.http.DELETE
import retrofit2.http.GET
import retrofit2.http.POST
import retrofit2.http.PUT
import retrofit2.http.Path
import retrofit2.http.Query

interface UserApiService {

    @GET("users/{id}")
    suspend fun getUser(@Path("id") userId: String): UserDto

    @GET("users")
    suspend fun getUsers(
        @Query("page") page: Int = 1,
        @Query("limit") limit: Int = 20,
    ): PaginatedResponseDto<UserDto>

    @POST("users")
    suspend fun createUser(@Body user: UserDto): UserDto

    @PUT("users/{id}")
    suspend fun updateUser(
        @Path("id") userId: String,
        @Body user: UserDto,
    ): UserDto

    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") userId: String)

    @POST("users/search")
    suspend fun searchUsers(@Body query: SearchQueryDto): PaginatedResponseDto<UserDto>

    @POST("users/{id}/avatar")
    @Multipart
    suspend fun uploadAvatar(
        @Path("id") userId: String,
        @Part avatar: MultipartBody.Part,
    ): UserDto
}
```

### Paginated Response DTO

```kotlin
package com.arclabs.app.data.models.dto

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class PaginatedResponseDto<T>(
    val data: List<T>,
    val page: Int,
    @SerialName("total_pages") val totalPages: Int,
    @SerialName("total_count") val totalCount: Int,
    @SerialName("has_next") val hasNext: Boolean,
)
```

### Search Query DTO

```kotlin
package com.arclabs.app.data.models.dto

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class SearchQueryDto(
    val query: String,
    val page: Int = 1,
    val limit: Int = 20,
    @SerialName("sort_by") val sortBy: String? = null,
    @SerialName("sort_order") val sortOrder: String? = null,
    val filters: Map<String, String>? = null,
)
```

### Remote Data Source Implementation

The Remote Data Source wraps the Retrofit API service and acts as a thin abstraction layer. This makes it easy to swap out the network implementation or add cross-cutting concerns like caching headers.

```kotlin
package com.arclabs.app.data.datasources.remote

import com.arclabs.app.data.datasources.remote.api.UserApiService
import com.arclabs.app.data.models.dto.PaginatedResponseDto
import com.arclabs.app.data.models.dto.SearchQueryDto
import com.arclabs.app.data.models.dto.UserDto
import javax.inject.Inject

class UserRemoteDataSource @Inject constructor(
    private val apiService: UserApiService,
) {

    suspend fun fetchUser(userId: String): UserDto =
        apiService.getUser(userId)

    suspend fun fetchUsers(page: Int = 1, limit: Int = 20): PaginatedResponseDto<UserDto> =
        apiService.getUsers(page = page, limit = limit)

    suspend fun createUser(userDto: UserDto): UserDto =
        apiService.createUser(userDto)

    suspend fun updateUser(userId: String, userDto: UserDto): UserDto =
        apiService.updateUser(userId, userDto)

    suspend fun deleteUser(userId: String) =
        apiService.deleteUser(userId)

    suspend fun searchUsers(query: String, page: Int = 1): PaginatedResponseDto<UserDto> =
        apiService.searchUsers(SearchQueryDto(query = query, page = page))
}
```

### Auth Interceptor

```kotlin
package com.arclabs.app.data.datasources.remote.api.interceptors

import com.arclabs.app.data.datasources.local.TokenProvider
import okhttp3.Interceptor
import okhttp3.Response
import javax.inject.Inject

class AuthInterceptor @Inject constructor(
    private val tokenProvider: TokenProvider,
) : Interceptor {

    override fun intercept(chain: Interceptor.Chain): Response {
        val originalRequest = chain.request()
        val token = tokenProvider.getAccessToken() ?: return chain.proceed(originalRequest)

        val authenticatedRequest = originalRequest.newBuilder()
            .header("Authorization", "Bearer $token")
            .build()

        return chain.proceed(authenticatedRequest)
    }
}
```

### Error Interceptor

```kotlin
package com.arclabs.app.data.datasources.remote.api.interceptors

import okhttp3.Interceptor
import okhttp3.Response
import java.io.IOException

class ErrorInterceptor : Interceptor {

    override fun intercept(chain: Interceptor.Chain): Response {
        val response = chain.proceed(chain.request())

        if (!response.isSuccessful) {
            val errorBody = response.body?.string()
            throw when (response.code) {
                401 -> UnauthorizedException(errorBody)
                403 -> ForbiddenException(errorBody)
                404 -> NotFoundException(errorBody)
                422 -> ValidationException(errorBody)
                in 500..599 -> ServerException(response.code, errorBody)
                else -> HttpException(response.code, errorBody)
            }
        }

        return response
    }
}

open class HttpException(val code: Int, message: String?) : IOException(message)
class UnauthorizedException(message: String?) : HttpException(401, message)
class ForbiddenException(message: String?) : HttpException(403, message)
class NotFoundException(message: String?) : HttpException(404, message)
class ValidationException(message: String?) : HttpException(422, message)
class ServerException(code: Int, message: String?) : HttpException(code, message)
```

### Network Module (Hilt)

```kotlin
package com.arclabs.app.data.di

import com.arclabs.app.data.datasources.remote.api.UserApiService
import com.arclabs.app.data.datasources.remote.api.interceptors.AuthInterceptor
import com.arclabs.app.data.datasources.remote.api.interceptors.ErrorInterceptor
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import kotlinx.serialization.json.Json
import okhttp3.MediaType.Companion.toMediaType
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.kotlinx.serialization.asConverterFactory
import java.util.concurrent.TimeUnit
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Singleton
    @Provides
    fun provideJson(): Json = Json {
        ignoreUnknownKeys = true
        isLenient = true
        encodeDefaults = true
        prettyPrint = false
        coerceInputValues = true
    }

    @Singleton
    @Provides
    fun provideLoggingInterceptor(): HttpLoggingInterceptor =
        HttpLoggingInterceptor().apply {
            level = if (BuildConfig.DEBUG) {
                HttpLoggingInterceptor.Level.BODY
            } else {
                HttpLoggingInterceptor.Level.NONE
            }
        }

    @Singleton
    @Provides
    fun provideOkHttpClient(
        loggingInterceptor: HttpLoggingInterceptor,
        authInterceptor: AuthInterceptor,
    ): OkHttpClient = OkHttpClient.Builder()
        .addInterceptor(authInterceptor)
        .addInterceptor(ErrorInterceptor())
        .addInterceptor(loggingInterceptor)
        .connectTimeout(30, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .writeTimeout(30, TimeUnit.SECONDS)
        .retryOnConnectionFailure(true)
        .build()

    @Singleton
    @Provides
    fun provideRetrofit(
        okHttpClient: OkHttpClient,
        json: Json,
    ): Retrofit = Retrofit.Builder()
        .baseUrl(BuildConfig.BASE_URL)
        .client(okHttpClient)
        .addConverterFactory(json.asConverterFactory("application/json".toMediaType()))
        .build()

    @Provides
    fun provideUserApiService(retrofit: Retrofit): UserApiService =
        retrofit.create(UserApiService::class.java)
}
```

---

## 💿 Local Data Source (Room)

Room provides a compile-time verified abstraction over SQLite. We use it for structured data persistence with reactive observation through Kotlin Flows.

### Room Entity

Room entities define database tables. Each entity maps to a single table with columns derived from its properties.

```kotlin
package com.arclabs.app.data.models.entity

import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.Index
import androidx.room.PrimaryKey

@Entity(
    tableName = "users",
    indices = [
        Index(value = ["email"], unique = true),
        Index(value = ["name"]),
    ],
)
data class UserEntity(
    @PrimaryKey
    val id: String,

    val email: String,

    val name: String,

    @ColumnInfo(name = "avatar_url")
    val avatarUrl: String?,

    @ColumnInfo(name = "is_active")
    val isActive: Boolean,

    @ColumnInfo(name = "created_at")
    val createdAt: Long,

    @ColumnInfo(name = "updated_at")
    val updatedAt: Long,

    @ColumnInfo(name = "cached_at", defaultValue = "0")
    val cachedAt: Long = System.currentTimeMillis(),
)
```

### DAO (Data Access Object)

DAOs define the interface for database operations. Room generates the implementation at compile time.

```kotlin
package com.arclabs.app.data.datasources.local.dao

import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import androidx.room.Transaction
import androidx.room.Update
import com.arclabs.app.data.models.entity.UserEntity
import kotlinx.coroutines.flow.Flow

@Dao
interface UserDao {

    // -- Single Item Queries --

    @Query("SELECT * FROM users WHERE id = :userId")
    suspend fun getUser(userId: String): UserEntity?

    @Query("SELECT * FROM users WHERE id = :userId")
    fun observeUser(userId: String): Flow<UserEntity?>

    @Query("SELECT * FROM users WHERE email = :email")
    suspend fun getUserByEmail(email: String): UserEntity?

    // -- List Queries --

    @Query("SELECT * FROM users ORDER BY name ASC")
    fun observeUsers(): Flow<List<UserEntity>>

    @Query("SELECT * FROM users WHERE is_active = 1 ORDER BY name ASC")
    fun observeActiveUsers(): Flow<List<UserEntity>>

    @Query("SELECT * FROM users WHERE name LIKE '%' || :query || '%' OR email LIKE '%' || :query || '%'")
    suspend fun searchUsers(query: String): List<UserEntity>

    @Query("SELECT * FROM users ORDER BY name ASC LIMIT :limit OFFSET :offset")
    suspend fun getUsersPaginated(limit: Int, offset: Int): List<UserEntity>

    // -- Insert Operations --

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: UserEntity)

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUsers(users: List<UserEntity>)

    // -- Update Operations --

    @Update
    suspend fun updateUser(user: UserEntity)

    @Query("UPDATE users SET is_active = :isActive WHERE id = :userId")
    suspend fun setUserActive(userId: String, isActive: Boolean)

    // -- Delete Operations --

    @Delete
    suspend fun deleteUser(user: UserEntity)

    @Query("DELETE FROM users WHERE id = :userId")
    suspend fun deleteUserById(userId: String)

    @Query("DELETE FROM users")
    suspend fun deleteAll()

    // -- Aggregation Queries --

    @Query("SELECT COUNT(*) FROM users")
    suspend fun getUserCount(): Int

    @Query("SELECT COUNT(*) FROM users WHERE is_active = 1")
    suspend fun getActiveUserCount(): Int

    // -- Transaction Example --

    @Transaction
    suspend fun replaceAllUsers(users: List<UserEntity>) {
        deleteAll()
        insertUsers(users)
    }

    // -- Cache Staleness Query --

    @Query("SELECT * FROM users WHERE cached_at < :threshold")
    suspend fun getStaleUsers(threshold: Long): List<UserEntity>
}
```

### App Database

```kotlin
package com.arclabs.app.data.datasources.local.database

import androidx.room.Database
import androidx.room.RoomDatabase
import com.arclabs.app.data.datasources.local.dao.UserDao
import com.arclabs.app.data.datasources.local.dao.RestaurantDao
import com.arclabs.app.data.models.entity.UserEntity
import com.arclabs.app.data.models.entity.RestaurantEntity

@Database(
    entities = [
        UserEntity::class,
        RestaurantEntity::class,
    ],
    version = 2,
    exportSchema = true,
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
    abstract fun restaurantDao(): RestaurantDao
}
```

### Room Migrations

```kotlin
package com.arclabs.app.data.datasources.local.database.migrations

import androidx.room.migration.Migration
import androidx.sqlite.db.SupportSQLiteDatabase

val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(db: SupportSQLiteDatabase) {
        db.execSQL("ALTER TABLE users ADD COLUMN updated_at INTEGER NOT NULL DEFAULT 0")
        db.execSQL("ALTER TABLE users ADD COLUMN cached_at INTEGER NOT NULL DEFAULT 0")
    }
}
```

### Database Module (Hilt)

```kotlin
package com.arclabs.app.data.di

import android.content.Context
import androidx.room.Room
import com.arclabs.app.data.datasources.local.dao.UserDao
import com.arclabs.app.data.datasources.local.dao.RestaurantDao
import com.arclabs.app.data.datasources.local.database.AppDatabase
import com.arclabs.app.data.datasources.local.database.migrations.MIGRATION_1_2
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Singleton
    @Provides
    fun provideDatabase(
        @ApplicationContext context: Context,
    ): AppDatabase = Room.databaseBuilder(
        context,
        AppDatabase::class.java,
        "app_database",
    )
        .addMigrations(MIGRATION_1_2)
        .build()

    @Provides
    fun provideUserDao(database: AppDatabase): UserDao =
        database.userDao()

    @Provides
    fun provideRestaurantDao(database: AppDatabase): RestaurantDao =
        database.restaurantDao()
}
```

### Local Data Source Implementation

```kotlin
package com.arclabs.app.data.datasources.local

import com.arclabs.app.data.datasources.local.dao.UserDao
import com.arclabs.app.data.models.entity.UserEntity
import kotlinx.coroutines.flow.Flow
import javax.inject.Inject

class UserLocalDataSource @Inject constructor(
    private val userDao: UserDao,
) {

    suspend fun getUser(userId: String): UserEntity? =
        userDao.getUser(userId)

    fun observeUser(userId: String): Flow<UserEntity?> =
        userDao.observeUser(userId)

    fun observeUsers(): Flow<List<UserEntity>> =
        userDao.observeUsers()

    suspend fun saveUser(userEntity: UserEntity) =
        userDao.insertUser(userEntity)

    suspend fun saveUsers(users: List<UserEntity>) =
        userDao.insertUsers(users)

    suspend fun deleteUser(userId: String) =
        userDao.deleteUserById(userId)

    suspend fun deleteAllUsers() =
        userDao.deleteAll()

    suspend fun replaceAllUsers(users: List<UserEntity>) =
        userDao.replaceAllUsers(users)

    suspend fun searchUsers(query: String): List<UserEntity> =
        userDao.searchUsers(query)

    suspend fun getStaleUsers(maxAgeMillis: Long): List<UserEntity> {
        val threshold = System.currentTimeMillis() - maxAgeMillis
        return userDao.getStaleUsers(threshold)
    }
}
```

---

## 🔧 DataStore (Preferences)

Jetpack DataStore replaces SharedPreferences for simple key-value storage. It provides a coroutine-based and Flow-based API with type safety.

### Preferences DataSource

```kotlin
package com.arclabs.app.data.datasources.local

import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.intPreferencesKey
import androidx.datastore.preferences.core.longPreferencesKey
import androidx.datastore.preferences.core.stringPreferencesKey
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.flow.map
import java.io.IOException
import javax.inject.Inject

class UserPreferencesDataSource @Inject constructor(
    private val dataStore: DataStore<Preferences>,
) {

    // -- Theme --

    val isDarkTheme: Flow<Boolean> = dataStore.data
        .catch { exception ->
            if (exception is IOException) emit(androidx.datastore.preferences.core.emptyPreferences())
            else throw exception
        }
        .map { preferences ->
            preferences[DARK_THEME_KEY] ?: false
        }

    suspend fun setDarkTheme(enabled: Boolean) {
        dataStore.edit { preferences ->
            preferences[DARK_THEME_KEY] = enabled
        }
    }

    // -- Language --

    val languageCode: Flow<String> = dataStore.data
        .catch { exception ->
            if (exception is IOException) emit(androidx.datastore.preferences.core.emptyPreferences())
            else throw exception
        }
        .map { preferences ->
            preferences[LANGUAGE_KEY] ?: "en"
        }

    suspend fun setLanguageCode(code: String) {
        dataStore.edit { preferences ->
            preferences[LANGUAGE_KEY] = code
        }
    }

    // -- Onboarding --

    val hasCompletedOnboarding: Flow<Boolean> = dataStore.data
        .catch { exception ->
            if (exception is IOException) emit(androidx.datastore.preferences.core.emptyPreferences())
            else throw exception
        }
        .map { preferences ->
            preferences[ONBOARDING_COMPLETED_KEY] ?: false
        }

    suspend fun setOnboardingCompleted(completed: Boolean) {
        dataStore.edit { preferences ->
            preferences[ONBOARDING_COMPLETED_KEY] = completed
        }
    }

    // -- Last Sync Timestamp --

    val lastSyncTimestamp: Flow<Long> = dataStore.data
        .catch { exception ->
            if (exception is IOException) emit(androidx.datastore.preferences.core.emptyPreferences())
            else throw exception
        }
        .map { preferences ->
            preferences[LAST_SYNC_KEY] ?: 0L
        }

    suspend fun setLastSyncTimestamp(timestamp: Long) {
        dataStore.edit { preferences ->
            preferences[LAST_SYNC_KEY] = timestamp
        }
    }

    // -- Items Per Page --

    val itemsPerPage: Flow<Int> = dataStore.data
        .catch { exception ->
            if (exception is IOException) emit(androidx.datastore.preferences.core.emptyPreferences())
            else throw exception
        }
        .map { preferences ->
            preferences[ITEMS_PER_PAGE_KEY] ?: 20
        }

    suspend fun setItemsPerPage(count: Int) {
        dataStore.edit { preferences ->
            preferences[ITEMS_PER_PAGE_KEY] = count
        }
    }

    // -- Clear All --

    suspend fun clearAll() {
        dataStore.edit { preferences ->
            preferences.clear()
        }
    }

    private companion object {
        val DARK_THEME_KEY = booleanPreferencesKey("dark_theme")
        val LANGUAGE_KEY = stringPreferencesKey("language_code")
        val ONBOARDING_COMPLETED_KEY = booleanPreferencesKey("onboarding_completed")
        val LAST_SYNC_KEY = longPreferencesKey("last_sync_timestamp")
        val ITEMS_PER_PAGE_KEY = intPreferencesKey("items_per_page")
    }
}
```

### DataStore Module (Hilt)

```kotlin
package com.arclabs.app.data.di

import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.preferencesDataStore
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

private val Context.dataStore: DataStore<Preferences> by preferencesDataStore(
    name = "user_preferences",
)

@Module
@InstallIn(SingletonComponent::class)
object DataStoreModule {

    @Singleton
    @Provides
    fun provideDataStore(
        @ApplicationContext context: Context,
    ): DataStore<Preferences> = context.dataStore
}
```

---

## 🔄 DTOs and Mappers

DTOs (Data Transfer Objects) represent the raw data format from external sources. Mappers convert between DTOs, Room Entities, and Domain models. This separation ensures that serialization details never leak into the Domain Layer.

### DTO (Kotlinx Serialization)

```kotlin
package com.arclabs.app.data.models.dto

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class UserDto(
    val id: String,
    val email: String,
    val name: String,
    @SerialName("avatar_url") val avatarUrl: String? = null,
    @SerialName("is_active") val isActive: Boolean = true,
    @SerialName("created_at") val createdAt: String,
    @SerialName("updated_at") val updatedAt: String? = null,
)
```

### Mapper Extension Functions

Mappers are implemented as extension functions for maximum clarity and composability. Each direction of conversion is an explicit, testable function.

```kotlin
package com.arclabs.app.data.mappers

import com.arclabs.app.data.models.dto.UserDto
import com.arclabs.app.data.models.entity.UserEntity
import com.arclabs.app.domain.models.User
import kotlinx.datetime.Instant

// -- DTO -> Domain --

fun UserDto.toDomain(): User = User(
    id = id,
    email = email,
    name = name,
    avatarUrl = avatarUrl,
    isActive = isActive,
    createdAt = Instant.parse(createdAt),
    updatedAt = updatedAt?.let { Instant.parse(it) },
)

fun List<UserDto>.toDomain(): List<User> = map { it.toDomain() }

// -- Domain -> DTO --

fun User.toDto(): UserDto = UserDto(
    id = id,
    email = email,
    name = name,
    avatarUrl = avatarUrl,
    isActive = isActive,
    createdAt = createdAt.toString(),
    updatedAt = updatedAt?.toString(),
)

// -- Room Entity -> Domain --

fun UserEntity.toDomain(): User = User(
    id = id,
    email = email,
    name = name,
    avatarUrl = avatarUrl,
    isActive = isActive,
    createdAt = Instant.fromEpochMilliseconds(createdAt),
    updatedAt = Instant.fromEpochMilliseconds(updatedAt),
)

fun List<UserEntity>.toDomain(): List<User> = map { it.toDomain() }

// -- Domain -> Room Entity --

fun User.toEntity(): UserEntity = UserEntity(
    id = id,
    email = email,
    name = name,
    avatarUrl = avatarUrl,
    isActive = isActive,
    createdAt = createdAt.toEpochMilliseconds(),
    updatedAt = updatedAt?.toEpochMilliseconds() ?: System.currentTimeMillis(),
    cachedAt = System.currentTimeMillis(),
)

fun List<User>.toEntity(): List<UserEntity> = map { it.toEntity() }

// -- DTO -> Room Entity --

fun UserDto.toEntity(): UserEntity = UserEntity(
    id = id,
    email = email,
    name = name,
    avatarUrl = avatarUrl,
    isActive = isActive,
    createdAt = Instant.parse(createdAt).toEpochMilliseconds(),
    updatedAt = updatedAt?.let { Instant.parse(it).toEpochMilliseconds() } ?: 0L,
    cachedAt = System.currentTimeMillis(),
)

fun List<UserDto>.toEntity(): List<UserEntity> = map { it.toEntity() }
```

### Mapper Pattern: Why Extension Functions

```
✅ DO: Use extension functions for mappers

    fun UserDto.toDomain(): User = ...
    fun User.toDto(): UserDto = ...

    Advantages:
    - Readable call site: userDto.toDomain()
    - Discoverable via autocomplete
    - Easily testable as standalone functions
    - No class instantiation needed
```

```
❌ DON'T: Use mapper classes with abstract methods

    class UserMapper {
        fun mapToDomain(dto: UserDto): User = ...
        fun mapToDto(user: User): UserDto = ...
    }

    Problems:
    - Unnecessary class allocation
    - Harder to discover
    - Must be injected or instantiated
```

---

## 🏗️ Repository Implementation

The repository is where all data sources come together. It decides which source to read from, how to cache data, and how to handle errors. The repository implements the Domain Layer interface and returns Domain models.

### Offline-First Repository Pattern

```kotlin
package com.arclabs.app.data.repositories

import com.arclabs.app.data.datasources.local.UserLocalDataSource
import com.arclabs.app.data.datasources.remote.UserRemoteDataSource
import com.arclabs.app.data.mappers.toDomain
import com.arclabs.app.data.mappers.toEntity
import com.arclabs.app.domain.models.User
import com.arclabs.app.domain.repositories.UserRepository
import kotlinx.coroutines.CoroutineDispatcher
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map
import kotlinx.coroutines.withContext
import javax.inject.Inject

class UserRepositoryImpl @Inject constructor(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource,
    @IoDispatcher private val ioDispatcher: CoroutineDispatcher,
) : UserRepository {

    override suspend fun getUser(userId: String): User = withContext(ioDispatcher) {
        // Try local first
        val cachedUser = localDataSource.getUser(userId)
        if (cachedUser != null) {
            return@withContext cachedUser.toDomain()
        }

        // Fetch from remote and cache
        val remoteUser = remoteDataSource.fetchUser(userId)
        localDataSource.saveUser(remoteUser.toEntity())
        remoteUser.toDomain()
    }

    override fun observeUsers(): Flow<List<User>> =
        localDataSource.observeUsers().map { entities ->
            entities.map { it.toDomain() }
        }

    override suspend fun refreshUsers(): Unit = withContext(ioDispatcher) {
        val response = remoteDataSource.fetchUsers()
        val entities = response.data.map { it.toEntity() }
        localDataSource.replaceAllUsers(entities)
    }

    override suspend fun createUser(user: User): User = withContext(ioDispatcher) {
        val dto = user.toDto()
        val createdDto = remoteDataSource.createUser(dto)
        localDataSource.saveUser(createdDto.toEntity())
        createdDto.toDomain()
    }

    override suspend fun updateUser(user: User): Unit = withContext(ioDispatcher) {
        val dto = user.toDto()
        val updatedDto = remoteDataSource.updateUser(user.id, dto)
        localDataSource.saveUser(updatedDto.toEntity())
    }

    override suspend fun deleteUser(userId: String): Unit = withContext(ioDispatcher) {
        remoteDataSource.deleteUser(userId)
        localDataSource.deleteUser(userId)
    }

    override suspend fun searchUsers(query: String): List<User> = withContext(ioDispatcher) {
        // Search remote, cache results, return domain models
        val response = remoteDataSource.searchUsers(query)
        val entities = response.data.map { it.toEntity() }
        localDataSource.saveUsers(entities)
        response.data.map { it.toDomain() }
    }
}
```

### Repository with Error Handling

```kotlin
package com.arclabs.app.data.repositories

import com.arclabs.app.data.datasources.local.UserLocalDataSource
import com.arclabs.app.data.datasources.remote.UserRemoteDataSource
import com.arclabs.app.data.datasources.remote.api.interceptors.HttpException
import com.arclabs.app.data.mappers.toDomain
import com.arclabs.app.data.mappers.toEntity
import com.arclabs.app.domain.models.User
import com.arclabs.app.domain.repositories.UserRepository
import com.arclabs.app.domain.util.Result
import kotlinx.coroutines.CoroutineDispatcher
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map
import kotlinx.coroutines.withContext
import java.io.IOException
import javax.inject.Inject

class UserRepositoryImpl @Inject constructor(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource,
    @IoDispatcher private val ioDispatcher: CoroutineDispatcher,
) : UserRepository {

    override suspend fun getUser(userId: String): Result<User> = withContext(ioDispatcher) {
        try {
            // Try local cache first
            val cached = localDataSource.getUser(userId)
            if (cached != null) {
                return@withContext Result.Success(cached.toDomain())
            }

            // Fetch from network
            val dto = remoteDataSource.fetchUser(userId)
            localDataSource.saveUser(dto.toEntity())
            Result.Success(dto.toDomain())
        } catch (e: IOException) {
            // Network error -- try cache as fallback
            val cached = localDataSource.getUser(userId)
            if (cached != null) {
                Result.Success(cached.toDomain())
            } else {
                Result.Error(e)
            }
        } catch (e: HttpException) {
            Result.Error(e)
        }
    }

    override fun observeUsers(): Flow<List<User>> =
        localDataSource.observeUsers().map { entities ->
            entities.map { it.toDomain() }
        }

    override suspend fun refreshUsers(): Result<Unit> = withContext(ioDispatcher) {
        try {
            val response = remoteDataSource.fetchUsers()
            val entities = response.data.map { it.toEntity() }
            localDataSource.replaceAllUsers(entities)
            Result.Success(Unit)
        } catch (e: IOException) {
            Result.Error(e)
        } catch (e: HttpException) {
            Result.Error(e)
        }
    }
}
```

### Repository Module (Hilt)

```kotlin
package com.arclabs.app.data.di

import com.arclabs.app.data.repositories.UserRepositoryImpl
import com.arclabs.app.data.repositories.RestaurantRepositoryImpl
import com.arclabs.app.data.repositories.SettingsRepositoryImpl
import com.arclabs.app.domain.repositories.UserRepository
import com.arclabs.app.domain.repositories.RestaurantRepository
import com.arclabs.app.domain.repositories.SettingsRepository
import dagger.Binds
import dagger.Module
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent

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
    abstract fun bindSettingsRepository(
        impl: SettingsRepositoryImpl,
    ): SettingsRepository
}
```

### Dispatcher Module

```kotlin
package com.arclabs.app.data.di

import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import kotlinx.coroutines.CoroutineDispatcher
import kotlinx.coroutines.Dispatchers
import javax.inject.Qualifier

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class IoDispatcher

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class DefaultDispatcher

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class MainDispatcher

@Module
@InstallIn(SingletonComponent::class)
object DispatcherModule {

    @IoDispatcher
    @Provides
    fun provideIoDispatcher(): CoroutineDispatcher = Dispatchers.IO

    @DefaultDispatcher
    @Provides
    fun provideDefaultDispatcher(): CoroutineDispatcher = Dispatchers.Default

    @MainDispatcher
    @Provides
    fun provideMainDispatcher(): CoroutineDispatcher = Dispatchers.Main
}
```

---

## 🧪 Testing Data Layer

### Testing Mappers

Mappers are pure functions and the easiest Data Layer components to test. Every mapper direction should be tested independently.

```kotlin
package com.arclabs.app.data.mappers

import com.arclabs.app.data.models.dto.UserDto
import com.arclabs.app.data.models.entity.UserEntity
import com.arclabs.app.domain.models.User
import kotlinx.datetime.Instant
import org.junit.Assert.assertEquals
import org.junit.Test

class UserMapperTest {

    @Test
    fun `UserDto toDomain maps all fields correctly`() {
        // Given
        val dto = UserDto(
            id = "user-1",
            email = "test@example.com",
            name = "Test User",
            avatarUrl = "https://example.com/avatar.png",
            isActive = true,
            createdAt = "2025-01-15T10:30:00Z",
            updatedAt = "2025-06-20T14:00:00Z",
        )

        // When
        val domain = dto.toDomain()

        // Then
        assertEquals("user-1", domain.id)
        assertEquals("test@example.com", domain.email)
        assertEquals("Test User", domain.name)
        assertEquals("https://example.com/avatar.png", domain.avatarUrl)
        assertEquals(true, domain.isActive)
        assertEquals(Instant.parse("2025-01-15T10:30:00Z"), domain.createdAt)
        assertEquals(Instant.parse("2025-06-20T14:00:00Z"), domain.updatedAt)
    }

    @Test
    fun `UserDto toDomain handles null optional fields`() {
        // Given
        val dto = UserDto(
            id = "user-2",
            email = "minimal@example.com",
            name = "Minimal",
            avatarUrl = null,
            isActive = false,
            createdAt = "2025-01-01T00:00:00Z",
            updatedAt = null,
        )

        // When
        val domain = dto.toDomain()

        // Then
        assertEquals(null, domain.avatarUrl)
        assertEquals(null, domain.updatedAt)
    }

    @Test
    fun `User toEntity maps timestamps to epoch milliseconds`() {
        // Given
        val user = User(
            id = "user-1",
            email = "test@example.com",
            name = "Test User",
            avatarUrl = null,
            isActive = true,
            createdAt = Instant.parse("2025-01-15T10:30:00Z"),
            updatedAt = Instant.parse("2025-06-20T14:00:00Z"),
        )

        // When
        val entity = user.toEntity()

        // Then
        assertEquals(Instant.parse("2025-01-15T10:30:00Z").toEpochMilliseconds(), entity.createdAt)
        assertEquals(Instant.parse("2025-06-20T14:00:00Z").toEpochMilliseconds(), entity.updatedAt)
    }

    @Test
    fun `UserDto toEntity converts ISO timestamp to epoch`() {
        // Given
        val dto = UserDto(
            id = "user-3",
            email = "dto@example.com",
            name = "DTO User",
            avatarUrl = null,
            isActive = true,
            createdAt = "2025-01-15T10:30:00Z",
            updatedAt = null,
        )

        // When
        val entity = dto.toEntity()

        // Then
        assertEquals(Instant.parse("2025-01-15T10:30:00Z").toEpochMilliseconds(), entity.createdAt)
        assertEquals(0L, entity.updatedAt)
    }

    @Test
    fun `round trip DTO to Domain to DTO preserves data`() {
        // Given
        val original = UserDto(
            id = "user-rt",
            email = "roundtrip@example.com",
            name = "Round Trip",
            avatarUrl = "https://example.com/pic.png",
            isActive = true,
            createdAt = "2025-01-15T10:30:00Z",
            updatedAt = "2025-06-20T14:00:00Z",
        )

        // When
        val result = original.toDomain().toDto()

        // Then
        assertEquals(original.id, result.id)
        assertEquals(original.email, result.email)
        assertEquals(original.name, result.name)
        assertEquals(original.avatarUrl, result.avatarUrl)
        assertEquals(original.isActive, result.isActive)
    }
}
```

### Testing DAOs with In-Memory Room Database

```kotlin
package com.arclabs.app.data.datasources.local.dao

import androidx.room.Room
import androidx.test.core.app.ApplicationProvider
import com.arclabs.app.data.datasources.local.database.AppDatabase
import com.arclabs.app.data.models.entity.UserEntity
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.test.runTest
import org.junit.After
import org.junit.Assert.assertEquals
import org.junit.Assert.assertNull
import org.junit.Before
import org.junit.Test

class UserDaoTest {

    private lateinit var database: AppDatabase
    private lateinit var sut: UserDao

    @Before
    fun setup() {
        database = Room.inMemoryDatabaseBuilder(
            ApplicationProvider.getApplicationContext(),
            AppDatabase::class.java,
        )
            .allowMainThreadQueries()
            .build()
        sut = database.userDao()
    }

    @After
    fun tearDown() {
        database.close()
    }

    @Test
    fun insertUser_and_getUser_returnsInsertedUser() = runTest {
        // Given
        val user = makeUserEntity(id = "user-1", name = "Alice")

        // When
        sut.insertUser(user)
        val result = sut.getUser("user-1")

        // Then
        assertEquals(user, result)
    }

    @Test
    fun getUser_withNonexistentId_returnsNull() = runTest {
        // When
        val result = sut.getUser("nonexistent")

        // Then
        assertNull(result)
    }

    @Test
    fun insertUser_withConflict_replacesExisting() = runTest {
        // Given
        val original = makeUserEntity(id = "user-1", name = "Alice")
        val updated = makeUserEntity(id = "user-1", name = "Alice Updated")

        // When
        sut.insertUser(original)
        sut.insertUser(updated)
        val result = sut.getUser("user-1")

        // Then
        assertEquals("Alice Updated", result?.name)
    }

    @Test
    fun observeUsers_emitsUpdates() = runTest {
        // Given
        val user1 = makeUserEntity(id = "user-1", name = "Alice")
        val user2 = makeUserEntity(id = "user-2", name = "Bob")

        // When
        sut.insertUsers(listOf(user1, user2))
        val result = sut.observeUsers().first()

        // Then
        assertEquals(2, result.size)
        assertEquals("Alice", result[0].name)  // Ordered by name ASC
        assertEquals("Bob", result[1].name)
    }

    @Test
    fun deleteAll_removesAllUsers() = runTest {
        // Given
        sut.insertUsers(listOf(
            makeUserEntity(id = "user-1"),
            makeUserEntity(id = "user-2"),
        ))

        // When
        sut.deleteAll()
        val count = sut.getUserCount()

        // Then
        assertEquals(0, count)
    }

    @Test
    fun replaceAllUsers_deletesAndInsertsInTransaction() = runTest {
        // Given
        sut.insertUser(makeUserEntity(id = "old-user"))
        val newUsers = listOf(
            makeUserEntity(id = "new-1"),
            makeUserEntity(id = "new-2"),
        )

        // When
        sut.replaceAllUsers(newUsers)
        val result = sut.getUserCount()

        // Then
        assertEquals(2, result)
        assertNull(sut.getUser("old-user"))
    }

    // -- Test Helpers --

    private fun makeUserEntity(
        id: String = "user-1",
        name: String = "Test User",
        email: String = "$id@example.com",
    ) = UserEntity(
        id = id,
        email = email,
        name = name,
        avatarUrl = null,
        isActive = true,
        createdAt = System.currentTimeMillis(),
        updatedAt = System.currentTimeMillis(),
        cachedAt = System.currentTimeMillis(),
    )
}
```

### Testing Repository with Fake Data Sources

```kotlin
package com.arclabs.app.data.repositories

import com.arclabs.app.data.models.dto.PaginatedResponseDto
import com.arclabs.app.data.models.dto.UserDto
import com.arclabs.app.data.models.entity.UserEntity
import com.arclabs.app.domain.models.User
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.test.StandardTestDispatcher
import kotlinx.coroutines.test.runTest
import org.junit.Assert.assertEquals
import org.junit.Assert.assertNotNull
import org.junit.Before
import org.junit.Test

class UserRepositoryImplTest {

    private lateinit var fakeRemoteDataSource: FakeUserRemoteDataSource
    private lateinit var fakeLocalDataSource: FakeUserLocalDataSource
    private val testDispatcher = StandardTestDispatcher()
    private lateinit var sut: UserRepositoryImpl

    @Before
    fun setup() {
        fakeRemoteDataSource = FakeUserRemoteDataSource()
        fakeLocalDataSource = FakeUserLocalDataSource()
        sut = UserRepositoryImpl(
            remoteDataSource = fakeRemoteDataSource,
            localDataSource = fakeLocalDataSource,
            ioDispatcher = testDispatcher,
        )
    }

    @Test
    fun getUser_whenCached_returnsLocalData() = runTest(testDispatcher) {
        // Given
        fakeLocalDataSource.users["user-1"] = makeUserEntity(id = "user-1", name = "Cached")

        // When
        val result = sut.getUser("user-1")

        // Then
        assertEquals("Cached", result.name)
        assertEquals(0, fakeRemoteDataSource.fetchCount)
    }

    @Test
    fun getUser_whenNotCached_fetchesFromRemoteAndCaches() = runTest(testDispatcher) {
        // Given
        fakeRemoteDataSource.usersToReturn["user-1"] = makeUserDto(id = "user-1", name = "Remote")

        // When
        val result = sut.getUser("user-1")

        // Then
        assertEquals("Remote", result.name)
        assertEquals(1, fakeRemoteDataSource.fetchCount)
        assertNotNull(fakeLocalDataSource.users["user-1"])
    }

    @Test
    fun observeUsers_returnsMappedDomainModels() = runTest(testDispatcher) {
        // Given
        fakeLocalDataSource.usersFlow.value = listOf(
            makeUserEntity(id = "user-1", name = "Alice"),
            makeUserEntity(id = "user-2", name = "Bob"),
        )

        // When
        val result = sut.observeUsers().first()

        // Then
        assertEquals(2, result.size)
        assertEquals("Alice", result[0].name)
    }

    // -- Test Helpers --

    private fun makeUserDto(id: String, name: String = "Test") = UserDto(
        id = id, email = "$id@test.com", name = name,
        avatarUrl = null, isActive = true,
        createdAt = "2025-01-01T00:00:00Z", updatedAt = null,
    )

    private fun makeUserEntity(id: String, name: String = "Test") = UserEntity(
        id = id, email = "$id@test.com", name = name,
        avatarUrl = null, isActive = true,
        createdAt = 0L, updatedAt = 0L, cachedAt = 0L,
    )
}

// -- Fake Implementations --

class FakeUserRemoteDataSource {
    val usersToReturn = mutableMapOf<String, UserDto>()
    var fetchCount = 0

    suspend fun fetchUser(userId: String): UserDto {
        fetchCount++
        return usersToReturn[userId] ?: throw RuntimeException("Not found")
    }

    suspend fun fetchUsers(page: Int = 1): PaginatedResponseDto<UserDto> =
        PaginatedResponseDto(
            data = usersToReturn.values.toList(),
            page = page, totalPages = 1, totalCount = usersToReturn.size, hasNext = false,
        )
}

class FakeUserLocalDataSource {
    val users = mutableMapOf<String, UserEntity>()
    val usersFlow = MutableStateFlow<List<UserEntity>>(emptyList())

    suspend fun getUser(userId: String): UserEntity? = users[userId]
    fun observeUsers(): Flow<List<UserEntity>> = usersFlow
    suspend fun saveUser(entity: UserEntity) { users[entity.id] = entity }
    suspend fun saveUsers(entities: List<UserEntity>) { entities.forEach { users[it.id] = it } }
    suspend fun deleteUser(userId: String) { users.remove(userId) }
    suspend fun replaceAllUsers(entities: List<UserEntity>) {
        users.clear()
        entities.forEach { users[it.id] = it }
        usersFlow.value = entities
    }
}
```

---

## ✅ Data Layer Checklist

Use this checklist before submitting a pull request that touches the Data Layer:

- [ ] **Repository implements Domain interface** -- the repository class implements the interface defined in the Domain Layer, not a Data Layer interface.
- [ ] **DTOs use `@Serializable`** -- all network DTOs are annotated with Kotlinx Serialization, not Gson or Moshi.
- [ ] **Room Entities use `@Entity`** -- all database models are properly annotated with Room annotations.
- [ ] **Mappers cover all directions** -- DTO to Domain, Domain to DTO, Entity to Domain, Domain to Entity, DTO to Entity.
- [ ] **`withContext(ioDispatcher)`** -- all blocking I/O operations (network, database writes) are wrapped in `withContext` with an injected dispatcher.
- [ ] **`OnConflictStrategy` is specified** -- all `@Insert` operations define a conflict resolution strategy (typically `REPLACE`).
- [ ] **Flows are used for observation** -- reactive data from Room DAOs returns `Flow`, not `LiveData`.
- [ ] **Error handling is present** -- repositories handle `IOException` and `HttpException` gracefully with fallback to cache.
- [ ] **No Domain models in DAOs or API services** -- DAOs work with `Entity` types, API services work with `DTO` types.
- [ ] **Hilt modules are configured** -- `NetworkModule`, `DatabaseModule`, `DataStoreModule`, and `RepositoryModule` are all present and correct.
- [ ] **Database migrations exist** -- any schema change includes a `Migration` object, not `fallbackToDestructiveMigration()` in production.
- [ ] **Mapper tests are written** -- every mapper function has at least one test covering the happy path and null/optional fields.
- [ ] **DAO tests use in-memory database** -- DAO tests create an in-memory Room database and run on `allowMainThreadQueries()`.
- [ ] **Repository tests use fakes** -- repository tests inject fake data sources, not mocks, for deterministic behavior.
- [ ] **No `suspend` on Flow-returning functions** -- functions that return `Flow` should NOT be `suspend` functions.
- [ ] **DataStore reads handle `IOException`** -- all `dataStore.data` reads include a `.catch` block for `IOException`.

---

## 🚫 Common Mistakes

### 1. Returning DTOs to the Domain Layer

```
❌ WRONG: Exposing DTOs beyond the Data Layer
```

```kotlin
// DON'T: Repository returns a DTO
class UserRepositoryImpl @Inject constructor(
    private val apiService: UserApiService,
) : UserRepository {

    override suspend fun getUser(userId: String): UserDto =  // ❌ DTO leaks to Domain
        apiService.getUser(userId)
}
```

```
✅ CORRECT: Map DTOs to Domain models before returning
```

```kotlin
// DO: Repository returns a Domain model
class UserRepositoryImpl @Inject constructor(
    private val remoteDataSource: UserRemoteDataSource,
) : UserRepository {

    override suspend fun getUser(userId: String): User =  // ✅ Domain model
        remoteDataSource.fetchUser(userId).toDomain()
}
```

**Why:** The Domain Layer must never depend on serialization libraries, API response formats, or database schemas. DTOs contain `@Serializable`, `@SerialName`, and other infrastructure annotations that do not belong in clean Domain models.

---

### 2. Business Logic in the Repository

```
❌ WRONG: Putting business rules in the repository
```

```kotlin
class UserRepositoryImpl @Inject constructor(
    private val remoteDataSource: UserRemoteDataSource,
) : UserRepository {

    override suspend fun getUser(userId: String): User {
        val user = remoteDataSource.fetchUser(userId).toDomain()

        // ❌ Business logic does not belong here
        if (!user.isActive) {
            throw UserDeactivatedException("User $userId is not active")
        }

        if (user.name.isBlank()) {
            throw ValidationException("User name cannot be blank")
        }

        return user
    }
}
```

```
✅ CORRECT: Keep business logic in Domain Use Cases
```

```kotlin
// Repository -- only data access
class UserRepositoryImpl @Inject constructor(
    private val remoteDataSource: UserRemoteDataSource,
) : UserRepository {

    override suspend fun getUser(userId: String): User =  // ✅ Pure data access
        remoteDataSource.fetchUser(userId).toDomain()
}

// Use Case -- business logic lives here
class GetActiveUserUseCase @Inject constructor(
    private val repository: UserRepository,
) {
    suspend operator fun invoke(userId: String): User {
        val user = repository.getUser(userId)
        require(user.isActive) { "User $userId is not active" }  // ✅ Business rule
        return user
    }
}
```

**Why:** Repositories are responsible for data access only. Business rules, validation, and orchestration belong in Use Cases in the Domain Layer. This keeps the Data Layer replaceable without affecting business logic.

---

### 3. Not Using `withContext` for I/O Operations

```
❌ WRONG: Performing I/O on the caller's dispatcher
```

```kotlin
class UserRepositoryImpl @Inject constructor(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource,
) : UserRepository {

    override suspend fun getUser(userId: String): User {
        // ❌ Runs on whatever dispatcher the caller uses (possibly Main)
        val dto = remoteDataSource.fetchUser(userId)
        localDataSource.saveUser(dto.toEntity())
        return dto.toDomain()
    }
}
```

```
✅ CORRECT: Explicitly switch to IO dispatcher
```

```kotlin
class UserRepositoryImpl @Inject constructor(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource,
    @IoDispatcher private val ioDispatcher: CoroutineDispatcher,
) : UserRepository {

    override suspend fun getUser(userId: String): User = withContext(ioDispatcher) {
        // ✅ Always runs on IO dispatcher regardless of caller
        val dto = remoteDataSource.fetchUser(userId)
        localDataSource.saveUser(dto.toEntity())
        dto.toDomain()
    }
}
```

**Why:** If the ViewModel calls `getUser()` from the main thread, network and database operations would block the UI without `withContext`. By injecting the dispatcher, you also make the repository testable with `StandardTestDispatcher`.

---

### 4. Missing `OnConflictStrategy` in Room Inserts

```
❌ WRONG: No conflict strategy
```

```kotlin
@Dao
interface UserDao {

    @Insert  // ❌ Defaults to OnConflictStrategy.ABORT -- crashes on duplicate
    suspend fun insertUser(user: UserEntity)

    @Insert  // ❌ Same problem for batch inserts
    suspend fun insertUsers(users: List<UserEntity>)
}
```

```
✅ CORRECT: Always specify a conflict strategy
```

```kotlin
@Dao
interface UserDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)  // ✅ Upsert behavior
    suspend fun insertUser(user: UserEntity)

    @Insert(onConflict = OnConflictStrategy.REPLACE)  // ✅ Safe for batch operations
    suspend fun insertUsers(users: List<UserEntity>)

    @Insert(onConflict = OnConflictStrategy.IGNORE)  // ✅ Skip duplicates silently
    suspend fun insertUsersIfNotExists(users: List<UserEntity>)
}
```

**Why:** Without a conflict strategy, Room defaults to `ABORT`, which throws an `SQLiteConstraintException` if a row with the same primary key already exists. In an offline-first app where you cache API responses, duplicate inserts are expected. Use `REPLACE` for upsert behavior or `IGNORE` to skip duplicates.

---

### 5. Not Handling Network Errors Properly

```
❌ WRONG: Letting exceptions propagate unhandled
```

```kotlin
class UserRepositoryImpl @Inject constructor(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource,
    @IoDispatcher private val ioDispatcher: CoroutineDispatcher,
) : UserRepository {

    override suspend fun getUser(userId: String): User = withContext(ioDispatcher) {
        // ❌ If network fails, user sees a crash or generic error
        val dto = remoteDataSource.fetchUser(userId)
        localDataSource.saveUser(dto.toEntity())
        dto.toDomain()
    }
}
```

```
✅ CORRECT: Handle errors and fall back to cache
```

```kotlin
class UserRepositoryImpl @Inject constructor(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource,
    @IoDispatcher private val ioDispatcher: CoroutineDispatcher,
) : UserRepository {

    override suspend fun getUser(userId: String): Result<User> = withContext(ioDispatcher) {
        try {
            // Try network first
            val dto = remoteDataSource.fetchUser(userId)
            localDataSource.saveUser(dto.toEntity())
            Result.Success(dto.toDomain())
        } catch (e: IOException) {
            // ✅ Network unavailable -- fall back to cache
            val cached = localDataSource.getUser(userId)
            if (cached != null) {
                Result.Success(cached.toDomain())
            } else {
                Result.Error(e)
            }
        } catch (e: HttpException) {
            // ✅ Server error -- return structured error
            Result.Error(e)
        }
    }
}
```

**Why:** Users expect apps to work offline. A repository that crashes on network failure provides a poor user experience. By catching `IOException` (network issues) and `HttpException` (server errors) separately, the repository can fall back to cached data when the network is unavailable and report structured errors for server-side problems.

---

## 📚 Further Reading

- **Android Developer Guides**
    - [Guide to app architecture - Data Layer](https://developer.android.com/topic/architecture/data-layer)
    - [Room Persistence Library](https://developer.android.com/training/data-storage/room)
    - [Retrofit](https://square.github.io/retrofit/)
    - [Jetpack DataStore](https://developer.android.com/topic/libraries/architecture/datastore)
    - [Kotlinx Serialization](https://github.com/Kotlin/kotlinx.serialization)

- **Dependency Injection**
    - [Hilt - Dependency Injection](https://developer.android.com/training/dependency-injection/hilt-android)
    - [Hilt Modules](https://developer.android.com/training/dependency-injection/hilt-android#hilt-modules)

- **Testing**
    - [Testing Room DAOs](https://developer.android.com/training/data-storage/room/testing-db)
    - [Testing Coroutines](https://developer.android.com/kotlin/coroutines/test)
    - [Testing best practices](https://developer.android.com/training/testing)

- **Architecture References**
    - [Clean Architecture by Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
    - [Now in Android - Data Layer](https://github.com/android/nowinandroid/tree/main/core/data)
    - [Offline-first apps](https://developer.android.com/topic/architecture/data-layer/offline-first)
