# 🏛️ Clean Architecture

**The definitive guide to Clean Architecture for Android/Kotlin projects at ARC Labs Studio. This document establishes the architectural boundaries, dependency rules, and implementation patterns that every ARC Labs Android project must follow.**

---

## Table of Contents

- [Core Principle: The Dependency Rule](#-core-principle-the-dependency-rule)
- [ARC Labs Layer Structure](#-arc-labs-layer-structure)
- [Layer Responsibilities](#-layer-responsibilities)
  - [Presentation Layer](#1-presentation-layer)
  - [Domain Layer](#2-domain-layer)
  - [Data Layer](#3-data-layer)
- [Data Flow Example](#-data-flow-example)
- [Dependency Injection with Hilt](#-dependency-injection-with-hilt)
- [Testing Benefits](#-testing-benefits)
- [Clean Architecture Checklist](#-clean-architecture-checklist)
- [Common Mistakes](#-common-mistakes)
- [Migration Guide](#-migration-guide)
- [Further Reading](#-further-reading)

---

## 🎯 Core Principle: The Dependency Rule

The **Dependency Rule** is the single most important concept in Clean Architecture.
Source code dependencies must point **inward** — toward higher-level policies.

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│                   PRESENTATION LAYER                    │
│                                                         │
│   Composables  ·  ViewModels  ·  Navigation  ·  UI     │
│                                                         │
│         │                             │                 │
│         │  depends on                 │  depends on     │
│         ▼                             ▼                 │
│  ┌─────────────────────────────────────────────────┐    │
│  │                                                 │    │
│  │                 DOMAIN LAYER                     │    │
│  │                                                 │    │
│  │   Entities  ·  Use Cases  ·  Repository         │    │
│  │                                Interfaces       │    │
│  │                                                 │    │
│  └─────────────────────────────────────────────────┘    │
│         ▲                             ▲                 │
│         │  implements                 │  implements     │
│         │                             │                 │
│                                                         │
│                     DATA LAYER                          │
│                                                         │
│   Repositories  ·  Data Sources  ·  DTOs  ·  Mappers   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### The Flow of Dependencies

```
Presentation  ───────►  Domain  ◄───────  Data
    (UI)              (Business)        (Storage/Network)
```

**Key rules:**

1. **Domain Layer knows NOTHING about UI or data storage.** It contains pure Kotlin
   classes and interfaces with zero Android framework dependencies.

2. **Presentation Layer depends on Domain.** ViewModels call Use Cases and observe
   Domain Entities. They never touch Retrofit, Room, or any data source directly.

3. **Data Layer depends on Domain.** Repository implementations fulfill the contracts
   (interfaces) defined in the Domain Layer. The Domain never knows how data is
   fetched or stored.

4. **Dependencies flow INWARD.** Outer layers know about inner layers, but inner
   layers are completely unaware of outer layers.

5. **Cross-layer communication uses interfaces.** The Domain defines repository
   interfaces; the Data layer implements them. This is the Dependency Inversion
   Principle in action.

### Why This Matters

| Benefit              | Explanation                                                  |
|----------------------|--------------------------------------------------------------|
| **Testability**      | Domain logic can be tested without Android framework         |
| **Flexibility**      | Swap Retrofit for Ktor without touching business logic       |
| **Maintainability**  | Changes in one layer don't cascade through the entire app    |
| **Team Scalability** | Different developers can work on different layers in parallel|
| **Readability**      | Clear separation makes the codebase easier to navigate       |

---

## 📂 ARC Labs Layer Structure

Every ARC Labs Android project follows this package structure. The top-level packages
mirror the three architecture layers, with feature-based organization inside the
presentation layer.

```
com.arclabs.myapp/
│
├── presentation/                   # PRESENTATION LAYER
│   ├── features/
│   │   ├── userprofile/
│   │   │   ├── screen/
│   │   │   │   ├── UserProfileScreen.kt
│   │   │   │   └── UserProfileContent.kt
│   │   │   ├── viewmodel/
│   │   │   │   ├── UserProfileViewModel.kt
│   │   │   │   └── UserProfileUiState.kt
│   │   │   ├── navigation/
│   │   │   │   └── UserProfileNavigation.kt
│   │   │   └── components/
│   │   │       ├── UserAvatar.kt
│   │   │       └── UserStatsCard.kt
│   │   │
│   │   ├── home/
│   │   │   ├── screen/
│   │   │   │   └── HomeScreen.kt
│   │   │   ├── viewmodel/
│   │   │   │   ├── HomeViewModel.kt
│   │   │   │   └── HomeUiState.kt
│   │   │   └── navigation/
│   │   │       └── HomeNavigation.kt
│   │   │
│   │   └── settings/
│   │       ├── screen/
│   │       │   └── SettingsScreen.kt
│   │       └── viewmodel/
│   │           └── SettingsViewModel.kt
│   │
│   ├── navigation/
│   │   ├── AppNavHost.kt
│   │   └── Screen.kt
│   │
│   ├── theme/
│   │   ├── Theme.kt
│   │   ├── Color.kt
│   │   └── Type.kt
│   │
│   └── components/                 # Shared UI components
│       ├── LoadingIndicator.kt
│       ├── ErrorScreen.kt
│       └── EmptyStateView.kt
│
├── domain/                         # DOMAIN LAYER
│   ├── entities/
│   │   ├── User.kt
│   │   ├── Restaurant.kt
│   │   └── Review.kt
│   │
│   ├── usecases/
│   │   ├── user/
│   │   │   ├── GetUserProfileUseCase.kt
│   │   │   ├── UpdateUserProfileUseCase.kt
│   │   │   └── DeleteUserAccountUseCase.kt
│   │   ├── restaurant/
│   │   │   ├── SearchRestaurantsUseCase.kt
│   │   │   └── GetRestaurantDetailsUseCase.kt
│   │   └── review/
│   │       ├── SubmitReviewUseCase.kt
│   │       └── GetReviewsUseCase.kt
│   │
│   ├── repositories/               # INTERFACES ONLY
│   │   ├── UserRepository.kt
│   │   ├── RestaurantRepository.kt
│   │   └── ReviewRepository.kt
│   │
│   └── exceptions/
│       ├── DomainException.kt
│       └── UserNotFoundException.kt
│
├── data/                           # DATA LAYER
│   ├── repositories/               # IMPLEMENTATIONS
│   │   ├── UserRepositoryImpl.kt
│   │   ├── RestaurantRepositoryImpl.kt
│   │   └── ReviewRepositoryImpl.kt
│   │
│   ├── datasources/
│   │   ├── remote/
│   │   │   ├── api/
│   │   │   │   ├── UserApiService.kt
│   │   │   │   └── RestaurantApiService.kt
│   │   │   └── dto/
│   │   │       ├── UserDto.kt
│   │   │       ├── RestaurantDto.kt
│   │   │       └── ReviewDto.kt
│   │   │
│   │   └── local/
│   │       ├── database/
│   │       │   ├── AppDatabase.kt
│   │       │   ├── UserDao.kt
│   │       │   └── RestaurantDao.kt
│   │       ├── entities/
│   │       │   ├── UserEntity.kt
│   │       │   └── RestaurantEntity.kt
│   │       └── preferences/
│   │           └── UserPreferencesDataStore.kt
│   │
│   ├── mappers/
│   │   ├── UserMapper.kt
│   │   └── RestaurantMapper.kt
│   │
│   └── di/
│       ├── DataModule.kt
│       ├── NetworkModule.kt
│       └── DatabaseModule.kt
│
└── di/
    └── AppModule.kt
```

### Package Naming Conventions

| Convention                    | Example                                         |
|-------------------------------|--------------------------------------------------|
| Base package                  | `com.arclabs.<appname>`                          |
| Feature package               | `presentation.features.<featurename>`            |
| Use case grouping             | `domain.usecases.<featuregroup>`                 |
| DTO location                  | `data.datasources.remote.dto`                    |
| Room entities                 | `data.datasources.local.entities`                |
| DI modules                    | `data.di` or top-level `di`                      |

---

## 🏛️ Layer Responsibilities

### 1. Presentation Layer

**Purpose:** Transform domain data into UI state and handle user interactions.
The Presentation Layer is the outermost layer that users see and interact with.
It is responsible for rendering screens, managing UI state, and delegating user
actions to the Domain Layer through ViewModels.

**Components:**

| Component       | Responsibility                                          |
|-----------------|---------------------------------------------------------|
| **Composables** | Render UI based on state, emit user events              |
| **ViewModels**  | Manage UI state, call Use Cases, handle side effects    |
| **UiState**     | Sealed interfaces representing all possible screen states|
| **Navigation**  | Define routes, handle screen transitions                |
| **Components**  | Reusable UI building blocks shared across features      |

#### ✅ Presentation Layer Responsibilities

- ✅ Render UI based on state from ViewModels
- ✅ Capture user input and forward events to ViewModels
- ✅ Format domain data for display (date formatting, string resources)
- ✅ Handle navigation between screens
- ✅ Manage UI-specific state (scroll position, dialog visibility)
- ✅ Show loading indicators, error messages, and empty states
- ✅ Apply theming and styling

#### ❌ Presentation Layer Must NOT

- ❌ Contain business logic or validation rules
- ❌ Access data sources (Retrofit, Room, DataStore) directly
- ❌ Define or manipulate domain entities
- ❌ Perform data transformations that belong in the Domain
- ❌ Hold references to repositories or data sources
- ❌ Make network calls or database queries

#### ViewModel Example

Every ViewModel at ARC Labs uses `@HiltViewModel` for dependency injection,
`StateFlow` for reactive UI state, and a `sealed interface` for exhaustive
state modeling.

```kotlin
package com.arclabs.myapp.presentation.features.userprofile.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.arclabs.myapp.domain.entities.User
import com.arclabs.myapp.domain.usecases.user.GetUserProfileUseCase
import com.arclabs.myapp.domain.usecases.user.UpdateUserProfileUseCase
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class UserProfileViewModel @Inject constructor(
    private val getUserProfileUseCase: GetUserProfileUseCase,
    private val updateUserProfileUseCase: UpdateUserProfileUseCase,
) : ViewModel() {

    private val _uiState = MutableStateFlow<UserProfileUiState>(UserProfileUiState.Loading)
    val uiState: StateFlow<UserProfileUiState> = _uiState.asStateFlow()

    fun loadUserProfile(userId: String) {
        viewModelScope.launch {
            _uiState.value = UserProfileUiState.Loading
            try {
                val user = getUserProfileUseCase(userId)
                _uiState.value = UserProfileUiState.Success(user)
            } catch (e: Exception) {
                _uiState.value = UserProfileUiState.Error(e.message ?: "Unknown error")
            }
        }
    }

    fun updateProfile(user: User) {
        viewModelScope.launch {
            _uiState.value = UserProfileUiState.Loading
            try {
                updateUserProfileUseCase(user)
                _uiState.value = UserProfileUiState.Success(user)
            } catch (e: Exception) {
                _uiState.value = UserProfileUiState.Error(e.message ?: "Failed to update profile")
            }
        }
    }
}

sealed interface UserProfileUiState {
    data object Loading : UserProfileUiState
    data class Success(val user: User) : UserProfileUiState
    data class Error(val message: String) : UserProfileUiState
}
```

#### Composable Screen Example

Screens observe the ViewModel's `StateFlow` and render based on the current state.
They never call Use Cases or repositories directly.

```kotlin
package com.arclabs.myapp.presentation.features.userprofile.screen

import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.hilt.navigation.compose.hiltViewModel
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.arclabs.myapp.presentation.features.userprofile.viewmodel.UserProfileUiState
import com.arclabs.myapp.presentation.features.userprofile.viewmodel.UserProfileViewModel

@Composable
fun UserProfileScreen(
    userId: String,
    viewModel: UserProfileViewModel = hiltViewModel(),
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    LaunchedEffect(userId) {
        viewModel.loadUserProfile(userId)
    }

    Scaffold { paddingValues ->
        Box(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues),
            contentAlignment = Alignment.Center,
        ) {
            when (val state = uiState) {
                is UserProfileUiState.Loading -> {
                    CircularProgressIndicator()
                }
                is UserProfileUiState.Success -> {
                    UserProfileContent(user = state.user)
                }
                is UserProfileUiState.Error -> {
                    Text(
                        text = state.message,
                        color = MaterialTheme.colorScheme.error,
                    )
                }
            }
        }
    }
}

@Composable
private fun UserProfileContent(
    user: com.arclabs.myapp.domain.entities.User,
) {
    Column {
        Text(
            text = user.displayName,
            style = MaterialTheme.typography.headlineMedium,
        )
        Text(
            text = user.email,
            style = MaterialTheme.typography.bodyLarge,
        )
    }
}
```

---

### 2. Domain Layer

**Purpose:** Encapsulate all business logic and rules. The Domain Layer is the
innermost and most stable layer. It contains the core business logic of the
application, expressed through Entities, Use Cases, and Repository Interfaces.
This layer has **zero Android dependencies** — it is pure Kotlin.

**Components:**

| Component                 | Responsibility                                            |
|---------------------------|-----------------------------------------------------------|
| **Entities**              | Core business objects with behavior and validation        |
| **Use Cases**             | Single-purpose business operations                        |
| **Repository Interfaces** | Contracts for data access (no implementations here)       |
| **Domain Exceptions**     | Business-specific error types                             |

#### ✅ Domain Layer Responsibilities

- ✅ Define core business entities with validation logic
- ✅ Encapsulate business rules in Use Cases
- ✅ Define repository interfaces (contracts)
- ✅ Contain business-specific exceptions and error types
- ✅ Validate inputs and enforce business invariants
- ✅ Orchestrate multiple repository calls within Use Cases

#### ❌ Domain Layer Must NOT

- ❌ Import any Android framework classes (`android.*`, `androidx.*`)
- ❌ Know about Retrofit, Room, Ktor, or any data framework
- ❌ Know about Compose, Views, or any UI framework
- ❌ Contain DTOs, API models, or database entities
- ❌ Implement repository interfaces (that belongs in Data)
- ❌ Depend on Hilt annotations (use constructor injection only)
- ❌ Reference `Context`, `Application`, or any Android component

#### Entity Example

Entities are the heart of the Domain. They represent core business objects and
contain business logic as computed properties and methods. Entities must be
pure Kotlin data classes with no framework dependencies.

```kotlin
package com.arclabs.myapp.domain.entities

import kotlinx.datetime.Instant

data class User(
    val id: String,
    val email: String,
    val name: String,
    val avatarUrl: String?,
    val isActive: Boolean = true,
    val createdAt: Instant,
) {
    val isProfileComplete: Boolean
        get() = name.isNotBlank() && avatarUrl != null

    val displayName: String
        get() = name.ifBlank { "Anonymous User" }

    val initials: String
        get() = name
            .split(" ")
            .filter { it.isNotBlank() }
            .take(2)
            .map { it.first().uppercaseChar() }
            .joinToString("")
            .ifBlank { "?" }

    fun canPerformAction(): Boolean {
        return isActive && isProfileComplete
    }
}
```

#### Use Case Example

Use Cases encapsulate a single business operation. At ARC Labs, every Use Case:
- Has a single `suspend operator fun invoke()` method
- Receives dependencies via constructor injection
- Validates inputs before proceeding
- Contains business logic, not data-fetching logic

```kotlin
package com.arclabs.myapp.domain.usecases.user

import com.arclabs.myapp.domain.entities.User
import com.arclabs.myapp.domain.repositories.UserRepository
import javax.inject.Inject

class GetUserProfileUseCase @Inject constructor(
    private val userRepository: UserRepository,
) {

    suspend operator fun invoke(userId: String): User {
        require(userId.isNotBlank()) { "User ID must not be blank" }

        val user = userRepository.getUser(userId)

        check(user.isActive) { "User account is inactive" }

        return user
    }
}
```

#### Use Case with Multiple Repositories

Some Use Cases need to orchestrate data from multiple sources. This is where
the real power of Use Cases becomes apparent — they coordinate complex
business operations that span multiple repositories.

```kotlin
package com.arclabs.myapp.domain.usecases.restaurant

import com.arclabs.myapp.domain.entities.Restaurant
import com.arclabs.myapp.domain.entities.RestaurantWithReviews
import com.arclabs.myapp.domain.repositories.RestaurantRepository
import com.arclabs.myapp.domain.repositories.ReviewRepository
import javax.inject.Inject

class GetRestaurantDetailsUseCase @Inject constructor(
    private val restaurantRepository: RestaurantRepository,
    private val reviewRepository: ReviewRepository,
) {

    suspend operator fun invoke(restaurantId: String): RestaurantWithReviews {
        require(restaurantId.isNotBlank()) { "Restaurant ID must not be blank" }

        val restaurant = restaurantRepository.getRestaurant(restaurantId)
        val reviews = reviewRepository.getReviewsForRestaurant(restaurantId)
        val averageRating = reviews
            .map { it.rating }
            .average()
            .takeIf { !it.isNaN() }
            ?: 0.0

        return RestaurantWithReviews(
            restaurant = restaurant,
            reviews = reviews,
            averageRating = averageRating,
        )
    }
}
```

#### Repository Interface Example

Repository interfaces define the contract between the Domain and Data layers.
They specify WHAT data operations are available without dictating HOW they
are implemented.

```kotlin
package com.arclabs.myapp.domain.repositories

import com.arclabs.myapp.domain.entities.User
import kotlinx.coroutines.flow.Flow

interface UserRepository {

    suspend fun getUser(userId: String): User

    suspend fun updateUser(user: User)

    suspend fun deleteUser(userId: String)

    fun observeUsers(): Flow<List<User>>

    fun observeUser(userId: String): Flow<User>

    suspend fun searchUsers(query: String): List<User>
}
```

#### Domain Exceptions Example

Custom exceptions provide meaningful error types that the Presentation layer
can handle appropriately.

```kotlin
package com.arclabs.myapp.domain.exceptions

sealed class DomainException(
    override val message: String,
    override val cause: Throwable? = null,
) : Exception(message, cause)

class UserNotFoundException(
    userId: String,
) : DomainException("User not found: $userId")

class UserInactiveException(
    userId: String,
) : DomainException("User account is inactive: $userId")

class InvalidInputException(
    field: String,
    reason: String,
) : DomainException("Invalid $field: $reason")

class NetworkException(
    override val message: String = "Network error occurred",
    override val cause: Throwable? = null,
) : DomainException(message, cause)
```

---

### 3. Data Layer

**Purpose:** Implement data access and storage. The Data Layer is responsible
for fetching, caching, and persisting data. It implements the repository
interfaces defined in the Domain Layer and manages the coordination between
remote and local data sources.

**Components:**

| Component                     | Responsibility                                        |
|-------------------------------|-------------------------------------------------------|
| **Repository Implementations**| Fulfill Domain repository contracts                   |
| **Remote Data Sources**       | API calls (Retrofit, Ktor)                            |
| **Local Data Sources**        | Database (Room), preferences (DataStore)              |
| **DTOs**                      | Network/database models with mapping functions         |
| **Mappers**                   | Transform between DTOs and Domain Entities            |

#### ✅ Data Layer Responsibilities

- ✅ Implement repository interfaces from the Domain Layer
- ✅ Manage remote API calls (Retrofit/Ktor services)
- ✅ Manage local persistence (Room DAOs, DataStore)
- ✅ Map DTOs to Domain Entities and vice versa
- ✅ Handle caching strategies (offline-first, cache-then-network)
- ✅ Handle data synchronization between remote and local
- ✅ Translate data-level errors into Domain exceptions
- ✅ Provide Hilt modules for dependency injection bindings

#### ❌ Data Layer Must NOT

- ❌ Contain business logic or validation rules
- ❌ Reference Compose, ViewModels, or any UI component
- ❌ Expose DTOs or database entities to other layers
- ❌ Make business decisions about data
- ❌ Define new entity types that don't map to Domain entities
- ❌ Skip error handling or expose raw HTTP/database errors

#### Repository Implementation Example

Repository implementations coordinate between remote and local data sources.
They contain data-access logic such as caching strategies but never business logic.

```kotlin
package com.arclabs.myapp.data.repositories

import com.arclabs.myapp.data.datasources.local.UserLocalDataSource
import com.arclabs.myapp.data.datasources.remote.UserRemoteDataSource
import com.arclabs.myapp.domain.entities.User
import com.arclabs.myapp.domain.exceptions.NetworkException
import com.arclabs.myapp.domain.exceptions.UserNotFoundException
import com.arclabs.myapp.domain.repositories.UserRepository
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map
import javax.inject.Inject

class UserRepositoryImpl @Inject constructor(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource,
) : UserRepository {

    override suspend fun getUser(userId: String): User {
        // Offline-first strategy: try local, fall back to remote
        return localDataSource.getUser(userId)?.toDomain()
            ?: fetchAndCacheUser(userId)
    }

    override suspend fun updateUser(user: User) {
        try {
            remoteDataSource.updateUser(user.toDto())
            localDataSource.saveUser(user.toEntity())
        } catch (e: Exception) {
            throw NetworkException("Failed to update user", e)
        }
    }

    override suspend fun deleteUser(userId: String) {
        try {
            remoteDataSource.deleteUser(userId)
            localDataSource.deleteUser(userId)
        } catch (e: Exception) {
            throw NetworkException("Failed to delete user", e)
        }
    }

    override fun observeUsers(): Flow<List<User>> {
        return localDataSource.observeAllUsers().map { entities ->
            entities.map { it.toDomain() }
        }
    }

    override fun observeUser(userId: String): Flow<User> {
        return localDataSource.observeUser(userId).map { entity ->
            entity.toDomain()
        }
    }

    override suspend fun searchUsers(query: String): List<User> {
        return try {
            val remoteDtos = remoteDataSource.searchUsers(query)
            val entities = remoteDtos.map { it.toEntity() }
            localDataSource.saveUsers(entities)
            remoteDtos.map { it.toDomain() }
        } catch (e: Exception) {
            // Fall back to local search on network failure
            localDataSource.searchUsers(query).map { it.toDomain() }
        }
    }

    private suspend fun fetchAndCacheUser(userId: String): User {
        return try {
            val dto = remoteDataSource.fetchUser(userId)
            localDataSource.saveUser(dto.toEntity())
            dto.toDomain()
        } catch (e: Exception) {
            throw UserNotFoundException(userId)
        }
    }
}
```

#### DTO (Data Transfer Object) Example

DTOs represent the data format from external sources (API, database). They
include mapping functions to convert to/from Domain Entities.

```kotlin
package com.arclabs.myapp.data.datasources.remote.dto

import com.arclabs.myapp.data.datasources.local.entities.UserEntity
import com.arclabs.myapp.domain.entities.User
import kotlinx.datetime.Instant
import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class UserDto(
    val id: String,
    val email: String,
    val name: String,
    @SerialName("avatar_url") val avatarUrl: String?,
    @SerialName("is_active") val isActive: Boolean,
    @SerialName("created_at") val createdAt: String,
) {

    fun toDomain(): User = User(
        id = id,
        email = email,
        name = name,
        avatarUrl = avatarUrl,
        isActive = isActive,
        createdAt = Instant.parse(createdAt),
    )

    fun toEntity(): UserEntity = UserEntity(
        id = id,
        email = email,
        name = name,
        avatarUrl = avatarUrl,
        isActive = isActive,
        createdAt = createdAt,
    )
}

fun User.toDto(): UserDto = UserDto(
    id = id,
    email = email,
    name = name,
    avatarUrl = avatarUrl,
    isActive = isActive,
    createdAt = createdAt.toString(),
)
```

#### Room Entity Example

Local database entities are separate from DTOs and Domain Entities. They
represent the local storage schema.

```kotlin
package com.arclabs.myapp.data.datasources.local.entities

import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey
import com.arclabs.myapp.domain.entities.User
import kotlinx.datetime.Instant

@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey val id: String,
    val email: String,
    val name: String,
    @ColumnInfo(name = "avatar_url") val avatarUrl: String?,
    @ColumnInfo(name = "is_active") val isActive: Boolean,
    @ColumnInfo(name = "created_at") val createdAt: String,
) {

    fun toDomain(): User = User(
        id = id,
        email = email,
        name = name,
        avatarUrl = avatarUrl,
        isActive = isActive,
        createdAt = Instant.parse(createdAt),
    )
}

fun User.toEntity(): UserEntity = UserEntity(
    id = id,
    email = email,
    name = name,
    avatarUrl = avatarUrl,
    isActive = isActive,
    createdAt = createdAt.toString(),
)
```

#### Remote Data Source Example

```kotlin
package com.arclabs.myapp.data.datasources.remote

import com.arclabs.myapp.data.datasources.remote.dto.UserDto
import retrofit2.http.Body
import retrofit2.http.DELETE
import retrofit2.http.GET
import retrofit2.http.PUT
import retrofit2.http.Path
import retrofit2.http.Query

interface UserApiService {

    @GET("users/{userId}")
    suspend fun fetchUser(@Path("userId") userId: String): UserDto

    @PUT("users/{userId}")
    suspend fun updateUser(
        @Path("userId") userId: String,
        @Body user: UserDto,
    ): UserDto

    @DELETE("users/{userId}")
    suspend fun deleteUser(@Path("userId") userId: String)

    @GET("users/search")
    suspend fun searchUsers(@Query("q") query: String): List<UserDto>
}

class UserRemoteDataSource(
    private val apiService: UserApiService,
) {

    suspend fun fetchUser(userId: String): UserDto {
        return apiService.fetchUser(userId)
    }

    suspend fun updateUser(userDto: UserDto): UserDto {
        return apiService.updateUser(userDto.id, userDto)
    }

    suspend fun deleteUser(userId: String) {
        apiService.deleteUser(userId)
    }

    suspend fun searchUsers(query: String): List<UserDto> {
        return apiService.searchUsers(query)
    }
}
```

#### Local Data Source Example

```kotlin
package com.arclabs.myapp.data.datasources.local

import com.arclabs.myapp.data.datasources.local.entities.UserEntity
import kotlinx.coroutines.flow.Flow

interface UserLocalDataSource {

    suspend fun getUser(userId: String): UserEntity?

    suspend fun saveUser(entity: UserEntity)

    suspend fun saveUsers(entities: List<UserEntity>)

    suspend fun deleteUser(userId: String)

    fun observeAllUsers(): Flow<List<UserEntity>>

    fun observeUser(userId: String): Flow<UserEntity>

    suspend fun searchUsers(query: String): List<UserEntity>
}
```

---

## 🔄 Data Flow Example

This section traces a complete data flow from user action to screen update.
Understanding this flow is essential to working effectively within Clean
Architecture.

### Scenario: User taps "Load Profile"

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│              │    │              │    │              │    │              │    │              │
│  Composable  │───►│  ViewModel   │───►│  Use Case    │───►│  Repository  │───►│  DataSource  │
│  (Screen)    │    │              │    │              │    │  (Impl)      │    │  (Remote/    │
│              │◄───│              │◄───│              │◄───│              │◄───│   Local)     │
│              │    │              │    │              │    │              │    │              │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
     OBSERVE             EMIT              RETURN              RETURN              FETCH
     STATE              STATE              ENTITY              ENTITY              DTO
```

### Step 1: User Action (Composable)

The user taps a button. The Composable calls a method on the ViewModel.

```kotlin
// In UserProfileScreen.kt
@Composable
fun UserProfileScreen(
    userId: String,
    viewModel: UserProfileViewModel = hiltViewModel(),
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    // Step 1: User action triggers ViewModel method
    LaunchedEffect(userId) {
        viewModel.loadUserProfile(userId)
    }

    // UI renders based on state...
}
```

### Step 2: ViewModel Calls Use Case

The ViewModel updates the UI state to loading, then delegates to the Use Case.

```kotlin
// In UserProfileViewModel.kt
fun loadUserProfile(userId: String) {
    viewModelScope.launch {
        // Step 2a: Emit loading state
        _uiState.value = UserProfileUiState.Loading

        try {
            // Step 2b: Delegate to Use Case
            val user = getUserProfileUseCase(userId)

            // Step 2e: Emit success state (after steps 3 & 4 complete)
            _uiState.value = UserProfileUiState.Success(user)
        } catch (e: Exception) {
            _uiState.value = UserProfileUiState.Error(e.message ?: "Unknown error")
        }
    }
}
```

### Step 3: Use Case Executes Business Logic

The Use Case validates the input, calls the repository, and applies business rules.

```kotlin
// In GetUserProfileUseCase.kt
suspend operator fun invoke(userId: String): User {
    // Step 3a: Validate input
    require(userId.isNotBlank()) { "User ID must not be blank" }

    // Step 3b: Fetch from repository
    val user = userRepository.getUser(userId)

    // Step 3c: Apply business rules
    check(user.isActive) { "User account is inactive" }

    return user
}
```

### Step 4: Repository Fetches Data

The repository implementation coordinates between local and remote data sources.

```kotlin
// In UserRepositoryImpl.kt
override suspend fun getUser(userId: String): User {
    // Step 4a: Try local cache first
    val cachedUser = localDataSource.getUser(userId)
    if (cachedUser != null) {
        return cachedUser.toDomain()
    }

    // Step 4b: Fetch from remote
    val dto = remoteDataSource.fetchUser(userId)

    // Step 4c: Cache locally
    localDataSource.saveUser(dto.toEntity())

    // Step 4d: Map DTO to Domain Entity and return
    return dto.toDomain()
}
```

### Step 5: UI Updates

The Composable automatically recomposes when the `StateFlow` emits a new value.

```kotlin
// Back in UserProfileScreen.kt — the when block re-evaluates
when (val state = uiState) {
    is UserProfileUiState.Loading -> {
        CircularProgressIndicator()
    }
    is UserProfileUiState.Success -> {
        // Step 5: UI renders the loaded user
        UserProfileContent(user = state.user)
    }
    is UserProfileUiState.Error -> {
        ErrorScreen(message = state.message)
    }
}
```

---

## 💉 Dependency Injection with Hilt

Hilt modules wire the layers together while maintaining the Dependency Rule.
The Data layer provides the Hilt modules that bind implementations to interfaces.

```kotlin
package com.arclabs.myapp.data.di

import com.arclabs.myapp.data.datasources.local.UserLocalDataSource
import com.arclabs.myapp.data.datasources.local.UserLocalDataSourceImpl
import com.arclabs.myapp.data.datasources.remote.UserApiService
import com.arclabs.myapp.data.datasources.remote.UserRemoteDataSource
import com.arclabs.myapp.data.repositories.UserRepositoryImpl
import com.arclabs.myapp.domain.repositories.UserRepository
import dagger.Binds
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import retrofit2.Retrofit
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
abstract class DataModule {

    @Binds
    @Singleton
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl,
    ): UserRepository

    @Binds
    @Singleton
    abstract fun bindUserLocalDataSource(
        impl: UserLocalDataSourceImpl,
    ): UserLocalDataSource
}

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    @Singleton
    fun provideUserApiService(retrofit: Retrofit): UserApiService {
        return retrofit.create(UserApiService::class.java)
    }

    @Provides
    @Singleton
    fun provideUserRemoteDataSource(
        apiService: UserApiService,
    ): UserRemoteDataSource {
        return UserRemoteDataSource(apiService)
    }
}
```

---

## 🧪 Testing Benefits

Clean Architecture makes every layer independently testable. Each layer can
be tested in isolation with appropriate test doubles.

### Testing Use Cases with MockK

Use Cases are tested by mocking the repository interfaces they depend on.

```kotlin
package com.arclabs.myapp.domain.usecases.user

import com.arclabs.myapp.domain.entities.User
import com.arclabs.myapp.domain.repositories.UserRepository
import io.mockk.coEvery
import io.mockk.mockk
import kotlinx.coroutines.test.runTest
import kotlinx.datetime.Instant
import org.junit.jupiter.api.Assertions.assertEquals
import org.junit.jupiter.api.Assertions.assertThrows
import org.junit.jupiter.api.Test

class GetUserProfileUseCaseTest {

    private val userRepository: UserRepository = mockk()
    private val sut = GetUserProfileUseCase(userRepository)

    @Test
    fun `should return user when user exists and is active`() = runTest {
        // Given
        val expectedUser = makeUser()
        coEvery { userRepository.getUser("user-1") } returns expectedUser

        // When
        val result = sut("user-1")

        // Then
        assertEquals(expectedUser, result)
    }

    @Test
    fun `should throw when user ID is blank`() = runTest {
        // Given
        val blankId = ""

        // When / Then
        assertThrows(IllegalArgumentException::class.java) {
            runTest { sut(blankId) }
        }
    }

    @Test
    fun `should throw when user is inactive`() = runTest {
        // Given
        val inactiveUser = makeUser(isActive = false)
        coEvery { userRepository.getUser("user-1") } returns inactiveUser

        // When / Then
        assertThrows(IllegalStateException::class.java) {
            runTest { sut("user-1") }
        }
    }

    private fun makeUser(
        id: String = "user-1",
        isActive: Boolean = true,
    ): User = User(
        id = id,
        email = "test@arclabs.com",
        name = "Test User",
        avatarUrl = "https://example.com/avatar.jpg",
        isActive = isActive,
        createdAt = Instant.parse("2025-01-01T00:00:00Z"),
    )
}
```

### Testing ViewModels with Turbine

ViewModels are tested by verifying the sequence of UI states emitted in
response to actions. Turbine makes `StateFlow` testing concise and readable.

```kotlin
package com.arclabs.myapp.presentation.features.userprofile.viewmodel

import app.cash.turbine.test
import com.arclabs.myapp.domain.entities.User
import com.arclabs.myapp.domain.usecases.user.GetUserProfileUseCase
import com.arclabs.myapp.domain.usecases.user.UpdateUserProfileUseCase
import io.mockk.coEvery
import io.mockk.mockk
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.test.StandardTestDispatcher
import kotlinx.coroutines.test.resetMain
import kotlinx.coroutines.test.runTest
import kotlinx.coroutines.test.setMain
import kotlinx.datetime.Instant
import org.junit.jupiter.api.AfterEach
import org.junit.jupiter.api.Assertions.assertEquals
import org.junit.jupiter.api.Assertions.assertTrue
import org.junit.jupiter.api.BeforeEach
import org.junit.jupiter.api.Test

@OptIn(ExperimentalCoroutinesApi::class)
class UserProfileViewModelTest {

    private val testDispatcher = StandardTestDispatcher()
    private val getUserProfileUseCase: GetUserProfileUseCase = mockk()
    private val updateUserProfileUseCase: UpdateUserProfileUseCase = mockk()

    private lateinit var sut: UserProfileViewModel

    @BeforeEach
    fun setup() {
        Dispatchers.setMain(testDispatcher)
        sut = UserProfileViewModel(getUserProfileUseCase, updateUserProfileUseCase)
    }

    @AfterEach
    fun tearDown() {
        Dispatchers.resetMain()
    }

    @Test
    fun `should emit loading then success when profile loads`() = runTest {
        // Given
        val expectedUser = makeUser()
        coEvery { getUserProfileUseCase("user-1") } returns expectedUser

        // When / Then
        sut.uiState.test {
            assertEquals(UserProfileUiState.Loading, awaitItem())

            sut.loadUserProfile("user-1")

            assertEquals(UserProfileUiState.Loading, awaitItem())
            assertEquals(UserProfileUiState.Success(expectedUser), awaitItem())
        }
    }

    @Test
    fun `should emit error when use case throws`() = runTest {
        // Given
        coEvery { getUserProfileUseCase("bad-id") } throws Exception("Not found")

        // When / Then
        sut.uiState.test {
            assertEquals(UserProfileUiState.Loading, awaitItem())

            sut.loadUserProfile("bad-id")

            assertEquals(UserProfileUiState.Loading, awaitItem())

            val errorState = awaitItem()
            assertTrue(errorState is UserProfileUiState.Error)
            assertEquals("Not found", (errorState as UserProfileUiState.Error).message)
        }
    }

    private fun makeUser(): User = User(
        id = "user-1",
        email = "test@arclabs.com",
        name = "Test User",
        avatarUrl = "https://example.com/avatar.jpg",
        isActive = true,
        createdAt = Instant.parse("2025-01-01T00:00:00Z"),
    )
}
```

### Testing Repositories with Fake Data Sources

Repository implementations are tested using fake (in-memory) data sources
instead of mocks. This approach gives higher confidence in the caching logic.

```kotlin
package com.arclabs.myapp.data.repositories

import com.arclabs.myapp.data.datasources.local.entities.UserEntity
import com.arclabs.myapp.data.datasources.remote.dto.UserDto
import com.arclabs.myapp.domain.entities.User
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.test.runTest
import org.junit.jupiter.api.Assertions.assertEquals
import org.junit.jupiter.api.Assertions.assertNotNull
import org.junit.jupiter.api.Test

class UserRepositoryImplTest {

    private val fakeRemoteDataSource = FakeUserRemoteDataSource()
    private val fakeLocalDataSource = FakeUserLocalDataSource()
    private val sut = UserRepositoryImpl(fakeRemoteDataSource, fakeLocalDataSource)

    @Test
    fun `should return cached user when available locally`() = runTest {
        // Given
        val entity = makeUserEntity()
        fakeLocalDataSource.users[entity.id] = entity

        // When
        val result = sut.getUser("user-1")

        // Then
        assertEquals("user-1", result.id)
        assertEquals("Test User", result.name)
    }

    @Test
    fun `should fetch from remote and cache when not available locally`() = runTest {
        // Given
        val dto = makeUserDto()
        fakeRemoteDataSource.users["user-1"] = dto

        // When
        val result = sut.getUser("user-1")

        // Then
        assertEquals("user-1", result.id)
        assertNotNull(fakeLocalDataSource.users["user-1"]) // Verify caching
    }

    private fun makeUserDto(): UserDto = UserDto(
        id = "user-1",
        email = "test@arclabs.com",
        name = "Test User",
        avatarUrl = null,
        isActive = true,
        createdAt = "2025-01-01T00:00:00Z",
    )

    private fun makeUserEntity(): UserEntity = UserEntity(
        id = "user-1",
        email = "test@arclabs.com",
        name = "Test User",
        avatarUrl = null,
        isActive = true,
        createdAt = "2025-01-01T00:00:00Z",
    )
}

// Fake implementations for testing
class FakeUserRemoteDataSource {
    val users = mutableMapOf<String, UserDto>()

    suspend fun fetchUser(userId: String): UserDto {
        return users[userId] ?: throw Exception("User not found")
    }

    suspend fun searchUsers(query: String): List<UserDto> {
        return users.values.filter { it.name.contains(query, ignoreCase = true) }
    }
}

class FakeUserLocalDataSource {
    val users = mutableMapOf<String, UserEntity>()
    private val usersFlow = MutableStateFlow<List<UserEntity>>(emptyList())

    fun getUser(userId: String): UserEntity? = users[userId]

    fun saveUser(entity: UserEntity) {
        users[entity.id] = entity
        usersFlow.value = users.values.toList()
    }

    fun observeAllUsers(): Flow<List<UserEntity>> = usersFlow
}
```

---

## ✅ Clean Architecture Checklist

Use this checklist when reviewing code or creating new features.

### Dependency Direction

- [ ] Presentation depends on Domain only (never on Data)
- [ ] Domain has zero dependencies on other layers
- [ ] Data depends on Domain only (never on Presentation)
- [ ] No circular dependencies between packages
- [ ] Inner layers never import outer layer classes

### Layer Isolation

- [ ] Domain Layer contains NO Android imports (`android.*`, `androidx.*`)
- [ ] Domain Layer contains NO framework-specific annotations
- [ ] Entities are pure Kotlin data classes
- [ ] Use Cases contain only business logic, not UI or data logic
- [ ] ViewModels do NOT access data sources directly
- [ ] Composables do NOT access Use Cases or Repositories directly
- [ ] DTOs are confined to the Data Layer

### Interface Usage

- [ ] Repository interfaces are defined in Domain
- [ ] Repository implementations are in Data
- [ ] Hilt modules bind implementations to interfaces
- [ ] Use Cases receive repository interfaces, not implementations
- [ ] ViewModels receive Use Cases, not repositories

### Testing

- [ ] Use Cases are tested with mocked repositories
- [ ] ViewModels are tested with mocked Use Cases
- [ ] Repositories are tested with fake data sources
- [ ] Each test follows Given-When-Then structure
- [ ] Test classes use `makeSUT()` or similar factory patterns
- [ ] Domain layer tests have no Android dependencies

### Code Organization

- [ ] Features are organized under `presentation/features/`
- [ ] Each feature has `screen/`, `viewmodel/`, and optionally `navigation/`
- [ ] Use Cases are grouped by domain concept
- [ ] DTOs live in `data/datasources/remote/dto/`
- [ ] Room entities live in `data/datasources/local/entities/`

---

## 🚫 Common Mistakes

### Mistake 1: ViewModel Contains Business Logic

The ViewModel should delegate all business logic to Use Cases. It should only
manage UI state and call Use Cases.

#### ❌ Wrong: Business logic in ViewModel

```kotlin
@HiltViewModel
class UserProfileViewModel @Inject constructor(
    private val userRepository: UserRepository, // Direct repository access!
) : ViewModel() {

    fun loadUserProfile(userId: String) {
        viewModelScope.launch {
            val user = userRepository.getUser(userId)

            // Business logic in ViewModel — WRONG!
            if (!user.isActive) {
                _uiState.value = UserProfileUiState.Error("User inactive")
                return@launch
            }

            if (user.name.isBlank()) {
                _uiState.value = UserProfileUiState.Error("Profile incomplete")
                return@launch
            }

            _uiState.value = UserProfileUiState.Success(user)
        }
    }
}
```

#### ✅ Correct: Business logic in Use Case

```kotlin
// Use Case handles business rules
class GetUserProfileUseCase @Inject constructor(
    private val userRepository: UserRepository,
) {
    suspend operator fun invoke(userId: String): User {
        val user = userRepository.getUser(userId)
        check(user.isActive) { "User inactive" }
        check(user.name.isNotBlank()) { "Profile incomplete" }
        return user
    }
}

// ViewModel only manages UI state
@HiltViewModel
class UserProfileViewModel @Inject constructor(
    private val getUserProfileUseCase: GetUserProfileUseCase,
) : ViewModel() {

    fun loadUserProfile(userId: String) {
        viewModelScope.launch {
            _uiState.value = UserProfileUiState.Loading
            try {
                val user = getUserProfileUseCase(userId)
                _uiState.value = UserProfileUiState.Success(user)
            } catch (e: Exception) {
                _uiState.value = UserProfileUiState.Error(e.message ?: "Unknown error")
            }
        }
    }
}
```

---

### Mistake 2: Domain Depends on Data

The Domain Layer must never import anything from the Data Layer. If you find
yourself importing DTOs, Room entities, or Retrofit services in the Domain,
you have a dependency violation.

#### ❌ Wrong: Domain imports Data layer classes

```kotlin
package com.arclabs.myapp.domain.usecases

// WRONG! Domain importing Data layer class
import com.arclabs.myapp.data.datasources.remote.dto.UserDto
import com.arclabs.myapp.data.datasources.remote.UserApiService

class GetUserProfileUseCase(
    private val apiService: UserApiService, // WRONG! Data layer dependency
) {
    suspend operator fun invoke(userId: String): UserDto { // WRONG! Returns DTO
        return apiService.fetchUser(userId)
    }
}
```

#### ✅ Correct: Domain uses only its own interfaces and entities

```kotlin
package com.arclabs.myapp.domain.usecases

import com.arclabs.myapp.domain.entities.User
import com.arclabs.myapp.domain.repositories.UserRepository

class GetUserProfileUseCase(
    private val userRepository: UserRepository, // Domain interface
) {
    suspend operator fun invoke(userId: String): User { // Domain entity
        require(userId.isNotBlank()) { "User ID must not be blank" }
        return userRepository.getUser(userId)
    }
}
```

---

### Mistake 3: No Use Cases (Direct Repository Access from ViewModel)

Skipping Use Cases might seem simpler, but it leads to scattered business logic
across ViewModels, making the code harder to test, reuse, and maintain.

#### ❌ Wrong: ViewModel calls Repository directly

```kotlin
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val userRepository: UserRepository,       // Direct access!
    private val restaurantRepository: RestaurantRepository, // Direct access!
) : ViewModel() {

    fun loadDashboard(userId: String) {
        viewModelScope.launch {
            // Business logic scattered across the ViewModel
            val user = userRepository.getUser(userId)
            val nearby = restaurantRepository.getNearbyRestaurants(
                latitude = user.lastLatitude,
                longitude = user.lastLongitude,
            )
            val favorites = restaurantRepository.getFavorites(userId)
            // More orchestration logic here...
        }
    }
}
```

#### ✅ Correct: ViewModel calls Use Case, Use Case orchestrates

```kotlin
// Use Case encapsulates the orchestration logic
class GetDashboardDataUseCase @Inject constructor(
    private val userRepository: UserRepository,
    private val restaurantRepository: RestaurantRepository,
) {
    suspend operator fun invoke(userId: String): DashboardData {
        val user = userRepository.getUser(userId)
        val nearby = restaurantRepository.getNearbyRestaurants(
            latitude = user.lastLatitude,
            longitude = user.lastLongitude,
        )
        val favorites = restaurantRepository.getFavorites(userId)
        return DashboardData(user = user, nearby = nearby, favorites = favorites)
    }
}

// ViewModel is thin — only UI state management
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getDashboardDataUseCase: GetDashboardDataUseCase,
) : ViewModel() {

    fun loadDashboard(userId: String) {
        viewModelScope.launch {
            _uiState.value = HomeUiState.Loading
            try {
                val data = getDashboardDataUseCase(userId)
                _uiState.value = HomeUiState.Success(data)
            } catch (e: Exception) {
                _uiState.value = HomeUiState.Error(e.message ?: "Unknown error")
            }
        }
    }
}
```

---

## 🔀 Migration Guide

If you are migrating an existing project to Clean Architecture, follow these
steps incrementally.

### Phase 1: Create the Domain Layer

1. Identify your core business entities
2. Extract them as pure Kotlin data classes into `domain/entities/`
3. Define repository interfaces in `domain/repositories/`
4. Move business logic from ViewModels into Use Cases in `domain/usecases/`

### Phase 2: Create the Data Layer

1. Move repository implementations to `data/repositories/`
2. Ensure implementations fulfill the Domain interfaces
3. Create DTOs in `data/datasources/remote/dto/`
4. Add mapping functions (`toDomain()`, `toEntity()`)
5. Set up Hilt modules to bind implementations to interfaces

### Phase 3: Refactor the Presentation Layer

1. Update ViewModels to depend on Use Cases instead of repositories
2. Remove direct data source access from ViewModels
3. Ensure Composables only interact with ViewModels
4. Organize features under `presentation/features/`

### Phase 4: Validate

1. Run the Clean Architecture Checklist above
2. Verify no circular dependencies exist
3. Confirm Domain Layer has zero Android imports
4. Ensure all tests pass and coverage meets targets

---

## 📚 Further Reading

### Books

- **Clean Architecture** by Robert C. Martin — The foundational text
- **Dependency Injection Principles, Practices, and Patterns** by Steven van Deursen & Mark Seemann

### Internal Documentation

- [Domain Layer Guide](../Layers/domain.md) — Deep dive into entities, use cases, and interfaces
- [Data Layer Guide](../Layers/data.md) — Repository patterns, caching, and data sources
- [Presentation Layer Guide](../Layers/presentation.md) — Compose, ViewModels, and navigation
- [Testing Standards](../Quality/testing.md) — Coverage requirements and patterns
- [Dependency Injection Guide](../Architecture/dependency-injection.md) — Hilt modules and scoping
- [Module Structure](../Quality/module-structure.md) — Package organization conventions

### External Resources

- [Guide to app architecture (Android Developers)](https://developer.android.com/topic/architecture)
- [Now in Android (Google sample)](https://github.com/android/nowinandroid) — Official architecture reference
- [Kotlin Coroutines](https://kotlinlang.org/docs/coroutines-overview.html) — For async operations
- [Hilt Documentation](https://dagger.dev/hilt/) — Dependency injection for Android

---

**Remember**: Clean Architecture is about **boundaries and dependencies**. Keep the Domain pure, make everything testable, and let the architecture guide you toward maintainable code. 🏛️
