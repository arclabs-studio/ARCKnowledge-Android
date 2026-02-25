# 🎯 Domain Layer

**The heart of your application — pure business logic with zero framework dependencies.**

---

The Domain Layer is the innermost layer in Clean Architecture. It encapsulates all business rules,
entities, and use cases that define what the application *does* regardless of how it is presented
or where the data comes from. This layer has **no dependency** on Android, Hilt modules, Room,
Retrofit, or any other framework. It is pure Kotlin.

> **Golden Rule:** If you deleted every Android framework import and the Domain layer still
> compiled, you are doing it right.

---

## 🎯 Purpose

The Domain Layer serves as the **single source of truth** for business logic across the entire
application. Every piece of meaningful behavior flows through this layer.

### Core Responsibilities

1. **Contains ALL business logic** — validation, transformation, computation, rules
2. **Defines Entities** — the core data structures of the application
3. **Defines Use Cases** — the operations the application can perform
4. **Defines Repository Interfaces** — contracts that the Data layer must fulfill
5. **Defines Domain Exceptions** — structured error handling for business rule violations

### Dependency Rule

```
Presentation Layer  -->  Domain Layer  <--  Data Layer
     (depends on)         (depends on       (depends on)
                           NOTHING)
```

The Domain Layer sits at the center and **depends on nothing**. Both the Presentation and Data
layers depend on it — never the other way around. This is the fundamental rule of
[Clean Architecture](../Architecture/clean-architecture.md).

### Key Principles

- **Zero framework dependencies** — no Android imports, no Hilt `@Module` or `@Component`, no Room,
  no Retrofit, no Gson/Moshi annotations
- **Use Cases are the ONLY entry point** for business logic — ViewModels never call repositories
  directly
- **Repository interfaces** are defined here but **implemented** in the Data layer
- **Pure Kotlin** — the Domain layer can be a pure Kotlin module (no Android Gradle plugin)
- **Testable in isolation** — no emulator, no instrumented tests, just fast unit tests

---

## 📂 Directory Structure

A well-organized Domain layer follows a feature-based or type-based directory structure. We
recommend grouping by type at the top level, then by feature within each type:

```
domain/
├── entities/
│   ├── User.kt
│   ├── Restaurant.kt
│   ├── Order.kt
│   ├── Review.kt
│   ├── Location.kt
│   ├── CuisineType.kt
│   ├── PriceRange.kt
│   └── RatingCategory.kt
├── usecases/
│   ├── user/
│   │   ├── GetUserUseCase.kt
│   │   ├── UpdateUserUseCase.kt
│   │   ├── DeleteUserUseCase.kt
│   │   ├── ObserveUserUseCase.kt
│   │   └── ValidateUserEmailUseCase.kt
│   ├── restaurant/
│   │   ├── GetRestaurantsUseCase.kt
│   │   ├── GetRestaurantDetailUseCase.kt
│   │   ├── SearchRestaurantsUseCase.kt
│   │   ├── FilterRestaurantsUseCase.kt
│   │   ├── ObserveRestaurantsUseCase.kt
│   │   └── ToggleFavoriteUseCase.kt
│   ├── review/
│   │   ├── GetReviewsUseCase.kt
│   │   ├── SubmitReviewUseCase.kt
│   │   └── DeleteReviewUseCase.kt
│   └── order/
│       ├── PlaceOrderUseCase.kt
│       ├── CancelOrderUseCase.kt
│       └── ObserveOrderStatusUseCase.kt
├── repositories/
│   ├── UserRepository.kt
│   ├── RestaurantRepository.kt
│   ├── ReviewRepository.kt
│   └── OrderRepository.kt
└── exceptions/
    └── DomainException.kt
```

### Gradle Module Setup

The Domain layer should be a **pure Kotlin** module — not an Android library module:

```kotlin
// domain/build.gradle.kts
plugins {
    id("org.jetbrains.kotlin.jvm")
    id("org.jetbrains.kotlin.kapt")
}

dependencies {
    // Pure Kotlin dependencies only
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
    implementation("org.jetbrains.kotlinx:kotlinx-datetime:0.5.0")
    implementation("javax.inject:javax.inject:1")

    // Testing
    testImplementation("org.junit.jupiter:junit-jupiter:5.10.1")
    testImplementation("io.mockk:mockk:1.13.8")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.7.3")
    testImplementation("app.cash.turbine:turbine:1.0.0")
}
```

> **Note:** We use `javax.inject:javax.inject` for the `@Inject` annotation instead of
> `dagger.hilt.android`. This keeps the domain module free of Android and Hilt framework code
> while still enabling constructor injection.

---

## 📦 Entities

Entities are the **core business objects** of your application. They represent the fundamental
concepts that your business operates on. In Kotlin, we use **data classes** for entities because
they provide automatic `equals()`, `hashCode()`, `toString()`, `copy()`, and destructuring.

### Entity Guidelines

- ✅ Use `data class` with `val` properties (immutable by default)
- ✅ Add computed properties for derived business rules
- ✅ Include validation logic inside the entity when appropriate
- ✅ Use meaningful default values where they make sense
- ✅ Use enums or sealed classes for finite sets of values
- ✅ Use `kotlinx-datetime` for time types (pure Kotlin, no `java.time` dependency needed)
- ❌ No Room annotations (`@Entity`, `@PrimaryKey`, `@ColumnInfo` belong in the Data layer)
- ❌ No serialization annotations (`@Serializable`, `@SerialName`, `@Json` belong in the Data layer)
- ❌ No Android imports (`android.*`, `androidx.*`)
- ❌ No mutable state (`var` properties)
- ❌ No framework-specific types (`Uri`, `Bundle`, `Parcelable`)

---

### Complete Entity Examples

#### User Entity

```kotlin
package com.arclabs.domain.entities

import kotlinx.datetime.Clock
import kotlinx.datetime.Instant
import kotlin.time.Duration.Companion.days

/**
 * Represents a user in the system.
 *
 * This entity contains all user-related business rules as computed properties.
 * It has no framework dependencies and can be tested in pure unit tests.
 */
data class User(
    val id: String,
    val email: String,
    val name: String,
    val avatarUrl: String?,
    val isActive: Boolean,
    val membershipTier: MembershipTier = MembershipTier.FREE,
    val createdAt: Instant,
) {
    /**
     * A profile is considered complete when the user has provided a name
     * and uploaded an avatar image.
     */
    val isProfileComplete: Boolean
        get() = name.isNotBlank() && avatarUrl != null

    /**
     * Returns the user's display name, falling back to "Anonymous User"
     * when the name field is blank.
     */
    val displayName: String
        get() = name.ifBlank { "Anonymous User" }

    /**
     * Returns the initials derived from the user's name.
     * Example: "John Doe" -> "JD", "Alice" -> "A"
     */
    val initials: String
        get() = name
            .split(" ")
            .filter { it.isNotBlank() }
            .take(2)
            .joinToString("") { it.first().uppercase() }
            .ifEmpty { "?" }

    /**
     * Whether the user qualifies for premium features based on their membership tier.
     */
    val hasPremiumAccess: Boolean
        get() = membershipTier == MembershipTier.PREMIUM ||
            membershipTier == MembershipTier.ENTERPRISE

    /**
     * Checks if the user account is older than the specified number of days.
     */
    fun isOlderThan(days: Int): Boolean {
        val threshold = Clock.System.now().minus(days.days)
        return createdAt < threshold
    }

    /**
     * Validates the email format using a basic check.
     */
    fun hasValidEmail(): Boolean {
        return email.contains("@") && email.contains(".") && email.length >= 5
    }

    companion object
}

/**
 * Membership tiers available in the system.
 */
enum class MembershipTier {
    FREE,
    PREMIUM,
    ENTERPRISE,
}
```

#### Restaurant Entity

```kotlin
package com.arclabs.domain.entities

/**
 * Represents a restaurant listing in the application.
 *
 * Contains business logic for rating categorization, price formatting,
 * and filtering rules.
 */
data class Restaurant(
    val id: String,
    val name: String,
    val description: String,
    val cuisineType: CuisineType,
    val location: Location,
    val averageRating: Double,
    val totalReviews: Int,
    val priceRange: PriceRange,
    val isOpen: Boolean,
    val isFavorite: Boolean,
    val imageUrl: String?,
) {
    /**
     * A restaurant is considered highly rated when its average
     * rating is 4.0 or above.
     */
    val isHighlyRated: Boolean
        get() = averageRating >= 4.0

    /**
     * A restaurant is considered popular when it has both a high
     * rating and a significant number of reviews.
     */
    val isPopular: Boolean
        get() = averageRating >= 4.0 && totalReviews >= 50

    /**
     * Categorizes the restaurant's rating into human-readable tiers.
     */
    val ratingCategory: RatingCategory
        get() = when {
            averageRating >= 4.5 -> RatingCategory.EXCELLENT
            averageRating >= 4.0 -> RatingCategory.VERY_GOOD
            averageRating >= 3.0 -> RatingCategory.GOOD
            averageRating >= 2.0 -> RatingCategory.FAIR
            else -> RatingCategory.AVERAGE
        }

    /**
     * A formatted string combining cuisine type and price range
     * for display purposes.
     */
    val formattedCategory: String
        get() = "${cuisineType.displayName} ${priceRange.symbol}"

    /**
     * Whether this restaurant has enough reviews to be considered
     * trustworthy for recommendation.
     */
    val hasReliableRating: Boolean
        get() = totalReviews >= 10

    /**
     * Calculates the distance from this restaurant to a given location.
     */
    fun distanceTo(userLocation: Location): Double {
        return location.distanceTo(userLocation)
    }

    /**
     * Returns true if the restaurant matches the given search query
     * against its name, description, or cuisine type.
     */
    fun matchesSearch(query: String): Boolean {
        val lowerQuery = query.lowercase()
        return name.lowercase().contains(lowerQuery) ||
            description.lowercase().contains(lowerQuery) ||
            cuisineType.displayName.lowercase().contains(lowerQuery)
    }

    companion object
}
```

#### Supporting Enums and Value Objects

```kotlin
package com.arclabs.domain.entities

/**
 * Types of cuisine offered by restaurants.
 */
enum class CuisineType(val displayName: String) {
    ITALIAN("Italian"),
    JAPANESE("Japanese"),
    MEXICAN("Mexican"),
    INDIAN("Indian"),
    AMERICAN("American"),
    CHINESE("Chinese"),
    THAI("Thai"),
    FRENCH("French"),
    MEDITERRANEAN("Mediterranean"),
    OTHER("Other"),
}

/**
 * Price range tiers for restaurants.
 */
enum class PriceRange(val symbol: String, val description: String) {
    BUDGET("$", "Budget-friendly"),
    MODERATE("$$", "Moderate"),
    EXPENSIVE("$$$", "Expensive"),
    LUXURY("$$$$", "Fine dining"),
}

/**
 * Rating categories derived from numeric average ratings.
 */
enum class RatingCategory(val label: String) {
    EXCELLENT("Excellent"),
    VERY_GOOD("Very Good"),
    GOOD("Good"),
    FAIR("Fair"),
    AVERAGE("Average"),
}
```

#### Location Value Object

```kotlin
package com.arclabs.domain.entities

import kotlin.math.atan2
import kotlin.math.cos
import kotlin.math.pow
import kotlin.math.sin
import kotlin.math.sqrt

/**
 * Represents a geographic location with coordinates and human-readable address info.
 *
 * This value object contains the Haversine formula for calculating the distance
 * between two points on Earth's surface.
 */
data class Location(
    val latitude: Double,
    val longitude: Double,
    val address: String = "",
    val city: String = "",
    val country: String = "",
) {
    /**
     * A formatted string for display, showing city and country.
     */
    val displayAddress: String
        get() = when {
            city.isNotBlank() && country.isNotBlank() -> "$city, $country"
            city.isNotBlank() -> city
            address.isNotBlank() -> address
            else -> "Unknown location"
        }

    /**
     * Calculates the distance to another location in kilometers
     * using the Haversine formula.
     *
     * @param other The target location.
     * @return Distance in kilometers.
     */
    fun distanceTo(other: Location): Double {
        val earthRadiusKm = 6371.0

        val dLat = Math.toRadians(other.latitude - latitude)
        val dLon = Math.toRadians(other.longitude - longitude)

        val a = sin(dLat / 2).pow(2) +
            cos(Math.toRadians(latitude)) *
            cos(Math.toRadians(other.latitude)) *
            sin(dLon / 2).pow(2)

        val c = 2 * atan2(sqrt(a), sqrt(1 - a))

        return earthRadiusKm * c
    }

    /**
     * Returns true if this location is within the specified radius (in km)
     * of the given center location.
     */
    fun isWithinRadius(center: Location, radiusKm: Double): Boolean {
        return distanceTo(center) <= radiusKm
    }

    companion object
}
```

#### Order Entity

```kotlin
package com.arclabs.domain.entities

import kotlinx.datetime.Instant

/**
 * Represents a food order placed by a user at a restaurant.
 */
data class Order(
    val id: String,
    val userId: String,
    val restaurantId: String,
    val items: List<OrderItem>,
    val status: OrderStatus,
    val createdAt: Instant,
    val estimatedDeliveryAt: Instant?,
) {
    /**
     * The total price of all items in the order.
     */
    val totalPrice: Double
        get() = items.sumOf { it.totalPrice }

    /**
     * The total number of individual items in the order.
     */
    val totalItemCount: Int
        get() = items.sumOf { it.quantity }

    /**
     * Whether the order can still be cancelled.
     * Orders can only be cancelled if they haven't been prepared yet.
     */
    val isCancellable: Boolean
        get() = status == OrderStatus.PENDING || status == OrderStatus.CONFIRMED

    /**
     * Whether the order is in a terminal state (completed or cancelled).
     */
    val isFinalized: Boolean
        get() = status == OrderStatus.DELIVERED || status == OrderStatus.CANCELLED

    companion object
}

/**
 * A single item within an order.
 */
data class OrderItem(
    val id: String,
    val name: String,
    val unitPrice: Double,
    val quantity: Int,
    val specialInstructions: String?,
) {
    val totalPrice: Double
        get() = unitPrice * quantity
}

/**
 * The lifecycle states of an order.
 */
enum class OrderStatus {
    PENDING,
    CONFIRMED,
    PREPARING,
    READY_FOR_PICKUP,
    OUT_FOR_DELIVERY,
    DELIVERED,
    CANCELLED,
}
```

#### Review Entity

```kotlin
package com.arclabs.domain.entities

import kotlinx.datetime.Instant

/**
 * Represents a user review for a restaurant.
 */
data class Review(
    val id: String,
    val userId: String,
    val restaurantId: String,
    val rating: Int,
    val comment: String,
    val createdAt: Instant,
) {
    init {
        require(rating in 1..5) { "Rating must be between 1 and 5, got $rating" }
    }

    /**
     * Whether the review contains a meaningful written comment.
     */
    val hasComment: Boolean
        get() = comment.isNotBlank()

    /**
     * Whether this is a positive review (4 or 5 stars).
     */
    val isPositive: Boolean
        get() = rating >= 4

    companion object
}
```

---

## 🔧 Use Cases

Use Cases (also known as Interactors) are the **application-specific business rules**. Each Use
Case encapsulates **exactly one** business operation. They orchestrate the flow of data between
repositories, apply business rules, and return results.

### Use Case Rules

- ✅ **Single responsibility** — ONE business operation per Use Case
- ✅ **`operator fun invoke()`** — enables clean call-site syntax: `useCase(params)`
- ✅ **`@Inject constructor`** — the only annotation allowed in the Domain layer for DI
- ✅ **`suspend`** for one-shot async operations
- ✅ **`Flow`** for observable/streaming data
- ✅ Business validation and transformation logic
- ✅ Combining data from multiple repositories
- ❌ No UI logic (formatting for display belongs in Presentation)
- ❌ No direct data source access (always go through repository interfaces)
- ❌ No Android framework calls
- ❌ No multiple public methods — one Use Case, one `invoke()`

### Naming Convention

Use Cases follow the pattern: **`<Verb><Noun>UseCase`**

| Pattern | Example |
|---------|---------|
| Get (single item) | `GetUserUseCase` |
| Get (list) | `GetRestaurantsUseCase` |
| Search | `SearchRestaurantsUseCase` |
| Observe (stream) | `ObserveOrderStatusUseCase` |
| Create | `PlaceOrderUseCase` |
| Update | `UpdateUserUseCase` |
| Delete | `DeleteReviewUseCase` |
| Toggle | `ToggleFavoriteUseCase` |
| Validate | `ValidateUserEmailUseCase` |

---

### Use Case Pattern: Simple Retrieval

```kotlin
package com.arclabs.domain.usecases.user

import com.arclabs.domain.entities.User
import com.arclabs.domain.exceptions.DomainException
import com.arclabs.domain.repositories.UserRepository
import javax.inject.Inject

/**
 * Retrieves a user by their ID with business validation.
 *
 * Validates that the user ID is not blank and that the returned
 * user is active. Throws appropriate domain exceptions otherwise.
 */
class GetUserUseCase @Inject constructor(
    private val userRepository: UserRepository,
) {
    /**
     * @param userId The unique identifier of the user to retrieve.
     * @return The active [User] matching the given ID.
     * @throws IllegalArgumentException if [userId] is blank.
     * @throws DomainException.UserInactive if the user is not active.
     * @throws DomainException.UserNotFound if no user matches the ID.
     */
    suspend operator fun invoke(userId: String): User {
        require(userId.isNotBlank()) { "User ID must not be blank" }

        val user = userRepository.getUser(userId)

        if (!user.isActive) {
            throw DomainException.UserInactive(userId)
        }

        return user
    }
}
```

### Use Case Pattern: Returning a Flow

```kotlin
package com.arclabs.domain.usecases.restaurant

import com.arclabs.domain.entities.CuisineType
import com.arclabs.domain.entities.Restaurant
import com.arclabs.domain.repositories.RestaurantRepository
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map
import javax.inject.Inject

/**
 * Observes restaurants with optional cuisine type filtering.
 *
 * Returns a Flow that emits an updated list of restaurants whenever
 * the underlying data changes. Results are sorted by rating (highest first).
 */
class ObserveRestaurantsUseCase @Inject constructor(
    private val restaurantRepository: RestaurantRepository,
) {
    /**
     * @param cuisineType Optional filter. When non-null, only restaurants
     *   matching this cuisine type are emitted.
     * @return A [Flow] of filtered and sorted restaurant lists.
     */
    operator fun invoke(cuisineType: CuisineType? = null): Flow<List<Restaurant>> {
        return restaurantRepository.observeRestaurants()
            .map { restaurants ->
                restaurants
                    .filter { cuisineType == null || it.cuisineType == cuisineType }
                    .sortedByDescending { it.averageRating }
            }
    }
}
```

### Use Case Pattern: Combining Multiple Repositories

```kotlin
package com.arclabs.domain.usecases.restaurant

import com.arclabs.domain.entities.Restaurant
import com.arclabs.domain.entities.Review
import com.arclabs.domain.repositories.RestaurantRepository
import com.arclabs.domain.repositories.ReviewRepository
import javax.inject.Inject

/**
 * Retrieves a restaurant along with its reviews, combining data
 * from two different repositories.
 */
class GetRestaurantWithReviewsUseCase @Inject constructor(
    private val restaurantRepository: RestaurantRepository,
    private val reviewRepository: ReviewRepository,
) {
    suspend operator fun invoke(restaurantId: String): RestaurantWithReviews {
        require(restaurantId.isNotBlank()) { "Restaurant ID must not be blank" }

        val restaurant = restaurantRepository.getRestaurant(restaurantId)
        val reviews = reviewRepository.getReviewsForRestaurant(restaurantId)

        val sortedReviews = reviews.sortedByDescending { it.createdAt }
        val calculatedAverage = if (reviews.isNotEmpty()) {
            reviews.map { it.rating.toDouble() }.average()
        } else {
            0.0
        }

        return RestaurantWithReviews(
            restaurant = restaurant,
            reviews = sortedReviews,
            averageRating = calculatedAverage,
            totalReviews = reviews.size,
        )
    }
}

/**
 * Combined model containing a restaurant and its associated reviews.
 */
data class RestaurantWithReviews(
    val restaurant: Restaurant,
    val reviews: List<Review>,
    val averageRating: Double,
    val totalReviews: Int,
) {
    val hasReviews: Boolean
        get() = reviews.isNotEmpty()

    val positiveReviewPercentage: Double
        get() = if (reviews.isNotEmpty()) {
            reviews.count { it.isPositive }.toDouble() / reviews.size * 100
        } else {
            0.0
        }
}
```

### Use Case Pattern: Parameterized with Data Class

```kotlin
package com.arclabs.domain.usecases.restaurant

import com.arclabs.domain.entities.CuisineType
import com.arclabs.domain.entities.Location
import com.arclabs.domain.entities.PriceRange
import com.arclabs.domain.entities.Restaurant
import com.arclabs.domain.repositories.RestaurantRepository
import javax.inject.Inject

/**
 * Searches restaurants with multiple filter criteria.
 *
 * When the search query and all filters produce no results, an empty
 * list is returned rather than throwing an exception.
 */
class SearchRestaurantsUseCase @Inject constructor(
    private val restaurantRepository: RestaurantRepository,
) {
    /**
     * @param params The search and filter parameters.
     * @return A list of restaurants matching the criteria, sorted by relevance.
     */
    suspend operator fun invoke(params: Params): List<Restaurant> {
        require(params.query.length >= 2) { "Search query must be at least 2 characters" }

        val results = restaurantRepository.searchRestaurants(params.query)

        return results
            .filter { restaurant ->
                val matchesCuisine = params.cuisineType == null ||
                    restaurant.cuisineType == params.cuisineType
                val matchesRating = params.minRating == null ||
                    restaurant.averageRating >= params.minRating
                val matchesPrice = params.priceRange == null ||
                    restaurant.priceRange == params.priceRange
                val matchesDistance = params.userLocation == null ||
                    params.maxDistanceKm == null ||
                    restaurant.location.isWithinRadius(
                        params.userLocation,
                        params.maxDistanceKm,
                    )

                matchesCuisine && matchesRating && matchesPrice && matchesDistance
            }
            .sortedByDescending { it.averageRating }
    }

    /**
     * Search parameters with sensible defaults.
     */
    data class Params(
        val query: String,
        val cuisineType: CuisineType? = null,
        val minRating: Double? = null,
        val priceRange: PriceRange? = null,
        val userLocation: Location? = null,
        val maxDistanceKm: Double? = null,
    )
}
```

### Use Case Pattern: Write Operation with Validation

```kotlin
package com.arclabs.domain.usecases.review

import com.arclabs.domain.entities.Review
import com.arclabs.domain.exceptions.DomainException
import com.arclabs.domain.repositories.ReviewRepository
import javax.inject.Inject

/**
 * Submits a new review for a restaurant.
 *
 * Validates that the rating is within bounds and the comment meets
 * minimum length requirements for meaningful reviews.
 */
class SubmitReviewUseCase @Inject constructor(
    private val reviewRepository: ReviewRepository,
) {
    suspend operator fun invoke(params: Params): Review {
        if (params.rating !in 1..5) {
            throw DomainException.InvalidInput("rating", "must be between 1 and 5")
        }

        if (params.comment.isNotBlank() && params.comment.length < 10) {
            throw DomainException.InvalidInput("comment", "must be at least 10 characters")
        }

        return reviewRepository.submitReview(
            restaurantId = params.restaurantId,
            userId = params.userId,
            rating = params.rating,
            comment = params.comment.trim(),
        )
    }

    data class Params(
        val restaurantId: String,
        val userId: String,
        val rating: Int,
        val comment: String = "",
    )
}
```

### Use Case Pattern: Toggle/State Change

```kotlin
package com.arclabs.domain.usecases.restaurant

import com.arclabs.domain.repositories.RestaurantRepository
import javax.inject.Inject

/**
 * Toggles the favorite status of a restaurant for the current user.
 */
class ToggleFavoriteUseCase @Inject constructor(
    private val restaurantRepository: RestaurantRepository,
) {
    suspend operator fun invoke(restaurantId: String) {
        require(restaurantId.isNotBlank()) { "Restaurant ID must not be blank" }
        restaurantRepository.toggleFavorite(restaurantId)
    }
}
```

---

## 📋 Repository Interfaces

Repository interfaces define the **contract** between the Domain layer and the Data layer. They
are declared in the Domain layer and implemented in the Data layer. This follows the
[Dependency Inversion Principle](../Architecture/solid-principles.md) — high-level modules
(Domain) should not depend on low-level modules (Data); both should depend on abstractions.

### Repository Rules

- ✅ Defined in the Domain layer as `interface`
- ✅ Return only Domain entities (never DTOs, never Room entities)
- ✅ Use `suspend` for one-shot operations (get, create, update, delete)
- ✅ Use `Flow` for observable/streaming data
- ✅ Keep method signatures minimal and focused
- ❌ Never expose implementation details (no SQL queries, no API endpoints)
- ❌ Never return `Response<T>`, `Result<T>` wrappers from networking libraries
- ❌ Never accept DTOs or framework types as parameters

### Complete Repository Interface Examples

```kotlin
package com.arclabs.domain.repositories

import com.arclabs.domain.entities.User
import kotlinx.coroutines.flow.Flow

/**
 * Contract for user data operations.
 *
 * Implementations are responsible for determining the data source
 * (remote API, local database, cache, or a combination).
 */
interface UserRepository {
    /**
     * Retrieves a single user by their unique identifier.
     * @throws DomainException.UserNotFound if no user is found.
     */
    suspend fun getUser(userId: String): User

    /**
     * Persists changes to an existing user.
     */
    suspend fun updateUser(user: User)

    /**
     * Deletes a user by their unique identifier.
     * @throws DomainException.UserNotFound if no user is found.
     */
    suspend fun deleteUser(userId: String)

    /**
     * Observes a single user, emitting updates whenever the user data changes.
     */
    fun observeUser(userId: String): Flow<User>

    /**
     * Observes the complete list of users.
     */
    fun observeUsers(): Flow<List<User>>
}
```

```kotlin
package com.arclabs.domain.repositories

import com.arclabs.domain.entities.Restaurant
import kotlinx.coroutines.flow.Flow

/**
 * Contract for restaurant data operations.
 */
interface RestaurantRepository {
    suspend fun getRestaurant(id: String): Restaurant
    suspend fun searchRestaurants(query: String): List<Restaurant>
    fun observeRestaurants(): Flow<List<Restaurant>>
    fun observeFavorites(): Flow<List<Restaurant>>
    suspend fun toggleFavorite(restaurantId: String)
}
```

```kotlin
package com.arclabs.domain.repositories

import com.arclabs.domain.entities.Review

/**
 * Contract for review data operations.
 */
interface ReviewRepository {
    suspend fun getReviewsForRestaurant(restaurantId: String): List<Review>
    suspend fun submitReview(
        restaurantId: String,
        userId: String,
        rating: Int,
        comment: String,
    ): Review
    suspend fun deleteReview(reviewId: String)
}
```

```kotlin
package com.arclabs.domain.repositories

import com.arclabs.domain.entities.Order
import com.arclabs.domain.entities.OrderItem
import com.arclabs.domain.entities.OrderStatus
import kotlinx.coroutines.flow.Flow

/**
 * Contract for order data operations.
 */
interface OrderRepository {
    suspend fun placeOrder(
        userId: String,
        restaurantId: String,
        items: List<OrderItem>,
    ): Order
    suspend fun cancelOrder(orderId: String)
    suspend fun getOrder(orderId: String): Order
    fun observeOrderStatus(orderId: String): Flow<OrderStatus>
    fun observeUserOrders(userId: String): Flow<List<Order>>
}
```

---

## ⚠️ Domain Exceptions

Domain exceptions represent **business rule violations** and error conditions specific to the
application's domain. They are structured as a sealed class hierarchy so that consumers can
exhaustively handle all known error types.

```kotlin
package com.arclabs.domain.exceptions

/**
 * Base class for all domain-specific exceptions.
 *
 * Using a sealed class allows exhaustive `when` matching and ensures
 * that all error types are explicitly defined in the Domain layer.
 */
sealed class DomainException(message: String) : Exception(message) {

    // -- User Errors --

    class UserNotFound(userId: String) :
        DomainException("User not found: $userId")

    class UserInactive(userId: String) :
        DomainException("User is inactive: $userId")

    class UserAlreadyExists(email: String) :
        DomainException("User already exists with email: $email")

    // -- Restaurant Errors --

    class RestaurantNotFound(id: String) :
        DomainException("Restaurant not found: $id")

    class RestaurantClosed(id: String) :
        DomainException("Restaurant is currently closed: $id")

    // -- Order Errors --

    class OrderNotFound(orderId: String) :
        DomainException("Order not found: $orderId")

    class OrderNotCancellable(orderId: String) :
        DomainException("Order cannot be cancelled: $orderId")

    class EmptyOrder :
        DomainException("Order must contain at least one item")

    // -- Review Errors --

    class DuplicateReview(userId: String, restaurantId: String) :
        DomainException("User $userId has already reviewed restaurant $restaurantId")

    // -- Generic Errors --

    class InvalidInput(field: String, reason: String) :
        DomainException("Invalid $field: $reason")

    class NetworkUnavailable :
        DomainException("Network is unavailable")

    class Unauthorized :
        DomainException("User is not authorized to perform this action")

    class RateLimited :
        DomainException("Too many requests. Please try again later.")
}
```

### Handling Domain Exceptions in Presentation

While this is technically a Presentation concern, understanding how exceptions are consumed
helps write better Domain code:

```kotlin
// In ViewModel (Presentation layer)
viewModelScope.launch {
    try {
        val user = getUserUseCase(userId)
        _state.value = UserState.Success(user)
    } catch (e: DomainException.UserNotFound) {
        _state.value = UserState.Error("User not found")
    } catch (e: DomainException.UserInactive) {
        _state.value = UserState.Error("This account has been deactivated")
    } catch (e: DomainException.NetworkUnavailable) {
        _state.value = UserState.Error("Please check your internet connection")
    } catch (e: DomainException) {
        _state.value = UserState.Error(e.message ?: "An unexpected error occurred")
    }
}
```

---

## 🧪 Testing Domain Layer

The Domain layer is the **easiest layer to test** because it has no framework dependencies. Tests
run on the JVM without an emulator or Android device. The primary test targets are **Use Cases**
and **Entity business rules**.

### Testing Strategy

| Target | What to Test | Mocking |
|--------|-------------|---------|
| Entities | Computed properties, validation, business rules | None needed |
| Use Cases | Input validation, orchestration, error handling | Mock repositories |
| Repository Interfaces | N/A (interfaces, not implementations) | N/A |
| Domain Exceptions | Construction, message formatting | None needed |

### Testing Tools

- **JUnit 5** — test framework
- **MockK** — mocking library (Kotlin-first)
- **kotlinx-coroutines-test** — `runTest` for coroutine testing
- **Turbine** — `Flow` testing

---

### Testing Use Cases

```kotlin
package com.arclabs.domain.usecases.user

import com.arclabs.domain.entities.User
import com.arclabs.domain.exceptions.DomainException
import com.arclabs.domain.repositories.UserRepository
import io.mockk.coEvery
import io.mockk.coVerify
import io.mockk.mockk
import kotlinx.coroutines.test.runTest
import org.junit.jupiter.api.Assertions.assertEquals
import org.junit.jupiter.api.Assertions.assertThrows
import org.junit.jupiter.api.BeforeEach
import org.junit.jupiter.api.DisplayName
import org.junit.jupiter.api.Nested
import org.junit.jupiter.api.Test

class GetUserUseCaseTest {

    private val userRepository: UserRepository = mockk()
    private lateinit var sut: GetUserUseCase

    @BeforeEach
    fun setup() {
        sut = GetUserUseCase(userRepository)
    }

    @Nested
    @DisplayName("Given a valid user ID")
    inner class ValidUserId {

        @Test
        @DisplayName("Returns user when found and active")
        fun `returns user when found and active`() = runTest {
            // Given
            val expectedUser = User.fake(isActive = true)
            coEvery { userRepository.getUser("123") } returns expectedUser

            // When
            val result = sut("123")

            // Then
            assertEquals(expectedUser, result)
            coVerify(exactly = 1) { userRepository.getUser("123") }
        }

        @Test
        @DisplayName("Throws UserInactive when user is not active")
        fun `throws UserInactive when user is not active`() = runTest {
            // Given
            val inactiveUser = User.fake(isActive = false)
            coEvery { userRepository.getUser("123") } returns inactiveUser

            // When / Then
            assertThrows(DomainException.UserInactive::class.java) {
                runTest { sut("123") }
            }
        }
    }

    @Nested
    @DisplayName("Given an invalid user ID")
    inner class InvalidUserId {

        @Test
        @DisplayName("Throws IllegalArgumentException for blank user ID")
        fun `throws for blank user ID`() = runTest {
            // When / Then
            assertThrows(IllegalArgumentException::class.java) {
                runTest { sut("") }
            }
        }

        @Test
        @DisplayName("Throws IllegalArgumentException for whitespace-only user ID")
        fun `throws for whitespace user ID`() = runTest {
            // When / Then
            assertThrows(IllegalArgumentException::class.java) {
                runTest { sut("   ") }
            }
        }
    }
}
```

### Testing Flow-Based Use Cases

```kotlin
package com.arclabs.domain.usecases.restaurant

import app.cash.turbine.test
import com.arclabs.domain.entities.CuisineType
import com.arclabs.domain.entities.Restaurant
import com.arclabs.domain.repositories.RestaurantRepository
import io.mockk.every
import io.mockk.mockk
import kotlinx.coroutines.flow.flowOf
import kotlinx.coroutines.test.runTest
import org.junit.jupiter.api.Assertions.assertEquals
import org.junit.jupiter.api.BeforeEach
import org.junit.jupiter.api.DisplayName
import org.junit.jupiter.api.Test

class ObserveRestaurantsUseCaseTest {

    private val restaurantRepository: RestaurantRepository = mockk()
    private lateinit var sut: ObserveRestaurantsUseCase

    @BeforeEach
    fun setup() {
        sut = ObserveRestaurantsUseCase(restaurantRepository)
    }

    @Test
    @DisplayName("Emits all restaurants sorted by rating when no filter applied")
    fun `emits all restaurants sorted by rating`() = runTest {
        // Given
        val restaurants = listOf(
            Restaurant.fake(name = "Low", averageRating = 3.0),
            Restaurant.fake(name = "High", averageRating = 4.8),
            Restaurant.fake(name = "Mid", averageRating = 4.2),
        )
        every { restaurantRepository.observeRestaurants() } returns flowOf(restaurants)

        // When / Then
        sut().test {
            val result = awaitItem()
            assertEquals("High", result[0].name)
            assertEquals("Mid", result[1].name)
            assertEquals("Low", result[2].name)
            awaitComplete()
        }
    }

    @Test
    @DisplayName("Filters by cuisine type when filter is provided")
    fun `filters by cuisine type`() = runTest {
        // Given
        val restaurants = listOf(
            Restaurant.fake(name = "Sushi Place", cuisineType = CuisineType.JAPANESE),
            Restaurant.fake(name = "Pizza Shop", cuisineType = CuisineType.ITALIAN),
            Restaurant.fake(name = "Ramen Bar", cuisineType = CuisineType.JAPANESE),
        )
        every { restaurantRepository.observeRestaurants() } returns flowOf(restaurants)

        // When / Then
        sut(cuisineType = CuisineType.JAPANESE).test {
            val result = awaitItem()
            assertEquals(2, result.size)
            result.forEach { assertEquals(CuisineType.JAPANESE, it.cuisineType) }
            awaitComplete()
        }
    }

    @Test
    @DisplayName("Emits empty list when no restaurants match the filter")
    fun `emits empty list for no matches`() = runTest {
        // Given
        val restaurants = listOf(
            Restaurant.fake(cuisineType = CuisineType.ITALIAN),
        )
        every { restaurantRepository.observeRestaurants() } returns flowOf(restaurants)

        // When / Then
        sut(cuisineType = CuisineType.THAI).test {
            val result = awaitItem()
            assertEquals(emptyList<Restaurant>(), result)
            awaitComplete()
        }
    }
}
```

### Testing Entities

```kotlin
package com.arclabs.domain.entities

import org.junit.jupiter.api.Assertions.assertEquals
import org.junit.jupiter.api.Assertions.assertFalse
import org.junit.jupiter.api.Assertions.assertTrue
import org.junit.jupiter.api.DisplayName
import org.junit.jupiter.api.Nested
import org.junit.jupiter.api.Test

class UserTest {

    @Nested
    @DisplayName("isProfileComplete")
    inner class IsProfileComplete {

        @Test
        fun `returns true when name and avatar are present`() {
            // Given
            val user = User.fake(
                name = "John Doe",
                avatarUrl = "https://example.com/avatar.jpg",
            )

            // Then
            assertTrue(user.isProfileComplete)
        }

        @Test
        fun `returns false when avatar is null`() {
            // Given
            val user = User.fake(name = "John Doe", avatarUrl = null)

            // Then
            assertFalse(user.isProfileComplete)
        }

        @Test
        fun `returns false when name is blank`() {
            // Given
            val user = User.fake(name = "", avatarUrl = "https://example.com/avatar.jpg")

            // Then
            assertFalse(user.isProfileComplete)
        }
    }

    @Nested
    @DisplayName("displayName")
    inner class DisplayName {

        @Test
        fun `returns name when present`() {
            val user = User.fake(name = "John Doe")
            assertEquals("John Doe", user.displayName)
        }

        @Test
        fun `returns Anonymous User when name is blank`() {
            val user = User.fake(name = "")
            assertEquals("Anonymous User", user.displayName)
        }
    }

    @Nested
    @DisplayName("initials")
    inner class Initials {

        @Test
        fun `returns two-letter initials for full name`() {
            val user = User.fake(name = "John Doe")
            assertEquals("JD", user.initials)
        }

        @Test
        fun `returns single letter for single name`() {
            val user = User.fake(name = "Alice")
            assertEquals("A", user.initials)
        }

        @Test
        fun `returns question mark for blank name`() {
            val user = User.fake(name = "")
            assertEquals("?", user.initials)
        }
    }
}
```

```kotlin
package com.arclabs.domain.entities

import org.junit.jupiter.api.Assertions.assertEquals
import org.junit.jupiter.api.Assertions.assertFalse
import org.junit.jupiter.api.Assertions.assertTrue
import org.junit.jupiter.api.DisplayName
import org.junit.jupiter.api.Nested
import org.junit.jupiter.api.Test

class RestaurantTest {

    @Nested
    @DisplayName("ratingCategory")
    inner class RatingCategoryTests {

        @Test
        fun `returns EXCELLENT for 4_5 and above`() {
            val restaurant = Restaurant.fake(averageRating = 4.8)
            assertEquals(RatingCategory.EXCELLENT, restaurant.ratingCategory)
        }

        @Test
        fun `returns VERY_GOOD for 4_0 to 4_4`() {
            val restaurant = Restaurant.fake(averageRating = 4.2)
            assertEquals(RatingCategory.VERY_GOOD, restaurant.ratingCategory)
        }

        @Test
        fun `returns GOOD for 3_0 to 3_9`() {
            val restaurant = Restaurant.fake(averageRating = 3.5)
            assertEquals(RatingCategory.GOOD, restaurant.ratingCategory)
        }

        @Test
        fun `returns AVERAGE for below 2_0`() {
            val restaurant = Restaurant.fake(averageRating = 1.5)
            assertEquals(RatingCategory.AVERAGE, restaurant.ratingCategory)
        }
    }

    @Nested
    @DisplayName("isPopular")
    inner class IsPopular {

        @Test
        fun `returns true for high rating and many reviews`() {
            val restaurant = Restaurant.fake(averageRating = 4.5, totalReviews = 100)
            assertTrue(restaurant.isPopular)
        }

        @Test
        fun `returns false for high rating but few reviews`() {
            val restaurant = Restaurant.fake(averageRating = 4.5, totalReviews = 10)
            assertFalse(restaurant.isPopular)
        }
    }
}
```

### Test Fakes (Test Fixtures)

Place fake factory methods in the test source set as companion object extensions. This keeps
production code clean while providing convenient test data creation.

```kotlin
// File: domain/src/test/kotlin/com/arclabs/domain/entities/Fakes.kt
package com.arclabs.domain.entities

import kotlinx.datetime.Clock
import kotlinx.datetime.Instant

fun User.Companion.fake(
    id: String = "user-123",
    email: String = "test@example.com",
    name: String = "Test User",
    avatarUrl: String? = null,
    isActive: Boolean = true,
    membershipTier: MembershipTier = MembershipTier.FREE,
    createdAt: Instant = Clock.System.now(),
) = User(
    id = id,
    email = email,
    name = name,
    avatarUrl = avatarUrl,
    isActive = isActive,
    membershipTier = membershipTier,
    createdAt = createdAt,
)

fun Restaurant.Companion.fake(
    id: String = "restaurant-123",
    name: String = "Test Restaurant",
    description: String = "A great test restaurant",
    cuisineType: CuisineType = CuisineType.ITALIAN,
    location: Location = Location.fake(),
    averageRating: Double = 4.0,
    totalReviews: Int = 25,
    priceRange: PriceRange = PriceRange.MODERATE,
    isOpen: Boolean = true,
    isFavorite: Boolean = false,
    imageUrl: String? = null,
) = Restaurant(
    id = id,
    name = name,
    description = description,
    cuisineType = cuisineType,
    location = location,
    averageRating = averageRating,
    totalReviews = totalReviews,
    priceRange = priceRange,
    isOpen = isOpen,
    isFavorite = isFavorite,
    imageUrl = imageUrl,
)

fun Location.Companion.fake(
    latitude: Double = 40.7128,
    longitude: Double = -74.0060,
    address: String = "123 Test Street",
    city: String = "New York",
    country: String = "US",
) = Location(
    latitude = latitude,
    longitude = longitude,
    address = address,
    city = city,
    country = country,
)

fun Review.Companion.fake(
    id: String = "review-123",
    userId: String = "user-123",
    restaurantId: String = "restaurant-123",
    rating: Int = 4,
    comment: String = "Great food and service!",
    createdAt: Instant = Clock.System.now(),
) = Review(
    id = id,
    userId = userId,
    restaurantId = restaurantId,
    rating = rating,
    comment = comment,
    createdAt = createdAt,
)

fun Order.Companion.fake(
    id: String = "order-123",
    userId: String = "user-123",
    restaurantId: String = "restaurant-123",
    items: List<OrderItem> = listOf(
        OrderItem(
            id = "item-1",
            name = "Margherita Pizza",
            unitPrice = 12.99,
            quantity = 1,
            specialInstructions = null,
        ),
    ),
    status: OrderStatus = OrderStatus.PENDING,
    createdAt: Instant = Clock.System.now(),
    estimatedDeliveryAt: Instant? = null,
) = Order(
    id = id,
    userId = userId,
    restaurantId = restaurantId,
    items = items,
    status = status,
    createdAt = createdAt,
    estimatedDeliveryAt = estimatedDeliveryAt,
)
```

---

## ✅ Domain Layer Checklist

Use this checklist to verify your Domain layer follows all conventions:

### Entities
- [ ] Entities are pure Kotlin `data class` with `val` properties
- [ ] No Room annotations (`@Entity`, `@PrimaryKey`, `@ColumnInfo`)
- [ ] No serialization annotations (`@Serializable`, `@SerialName`, `@Json`)
- [ ] No Android imports (`android.*`, `androidx.*`)
- [ ] Business rules expressed as computed properties or methods
- [ ] Validation logic embedded where appropriate
- [ ] `companion object` declared for test fake extensions

### Use Cases
- [ ] Single responsibility — one operation per Use Case
- [ ] Named with `<Verb><Noun>UseCase` pattern
- [ ] Uses `operator fun invoke()` for clean call-site syntax
- [ ] `@Inject constructor` for Hilt dependency injection
- [ ] `suspend` for one-shot operations, `Flow` for streams
- [ ] Input validation with `require()` or domain exceptions
- [ ] No UI/formatting logic

### Repository Interfaces
- [ ] Defined as `interface` in the Domain layer
- [ ] Return only Domain entities (never DTOs or framework types)
- [ ] `suspend` for one-shot, `Flow` for observable operations
- [ ] Clear KDoc documenting expected behavior and exceptions

### Exceptions
- [ ] `sealed class` hierarchy extending `Exception`
- [ ] Descriptive error messages
- [ ] Covers all known business rule violations

### Testing
- [ ] All Use Cases have tests (100% coverage target)
- [ ] All Entity business rules have tests
- [ ] Given-When-Then pattern used consistently
- [ ] MockK used for repository mocking
- [ ] Turbine used for Flow testing
- [ ] Test fakes provided as companion object extensions
- [ ] Edge cases and error paths tested

---

## 🚫 Common Mistakes

### ❌ Mistake 1: Android Framework in Domain

```kotlin
// WRONG: Android import in Domain layer
import android.content.Context
import android.location.LocationManager

class GetNearbyRestaurantsUseCase(
    private val context: Context, // Framework dependency!
    private val restaurantRepository: RestaurantRepository,
) {
    suspend operator fun invoke(): List<Restaurant> {
        val locationManager = context.getSystemService(Context.LOCATION_SERVICE) as LocationManager
        // ...
    }
}
```

✅ **Correct approach:** Define a `LocationProvider` interface in the Domain layer and inject it.

```kotlin
// Domain layer — interface only
interface LocationProvider {
    suspend fun getCurrentLocation(): Location
}

// Domain layer — Use Case depends on abstraction
class GetNearbyRestaurantsUseCase @Inject constructor(
    private val locationProvider: LocationProvider,
    private val restaurantRepository: RestaurantRepository,
) {
    suspend operator fun invoke(radiusKm: Double = 5.0): List<Restaurant> {
        val currentLocation = locationProvider.getCurrentLocation()
        return restaurantRepository.observeRestaurants()
            // ...
    }
}

// Data layer — implementation uses Android framework
class AndroidLocationProvider @Inject constructor(
    @ApplicationContext private val context: Context,
) : LocationProvider {
    override suspend fun getCurrentLocation(): Location { /* ... */ }
}
```

---

### ❌ Mistake 2: Using DTOs as Domain Entities

```kotlin
// WRONG: Serialization annotations on domain entity
@Serializable
data class User(
    @SerialName("user_id") val id: String,
    @SerialName("full_name") val name: String,
    @SerialName("avatar_url") val avatarUrl: String?,
)
```

✅ **Correct approach:** Create separate DTO and domain entity with a mapper.

```kotlin
// Data layer — DTO with serialization
@Serializable
data class UserDto(
    @SerialName("user_id") val id: String,
    @SerialName("full_name") val name: String,
    @SerialName("avatar_url") val avatarUrl: String?,
)

// Domain layer — pure entity
data class User(
    val id: String,
    val name: String,
    val avatarUrl: String?,
)

// Data layer — mapper
fun UserDto.toDomain(): User = User(
    id = id,
    name = name,
    avatarUrl = avatarUrl,
)
```

---

### ❌ Mistake 3: Use Case with Multiple Responsibilities

```kotlin
// WRONG: God Use Case that does everything
class UserUseCase @Inject constructor(
    private val userRepository: UserRepository,
) {
    suspend fun getUser(id: String): User { /* ... */ }
    suspend fun updateUser(user: User) { /* ... */ }
    suspend fun deleteUser(id: String) { /* ... */ }
    suspend fun searchUsers(query: String): List<User> { /* ... */ }
    suspend fun validateEmail(email: String): Boolean { /* ... */ }
}
```

✅ **Correct approach:** One Use Case per business operation.

```kotlin
class GetUserUseCase @Inject constructor(private val repo: UserRepository) {
    suspend operator fun invoke(id: String): User { /* ... */ }
}

class UpdateUserUseCase @Inject constructor(private val repo: UserRepository) {
    suspend operator fun invoke(user: User) { /* ... */ }
}

class DeleteUserUseCase @Inject constructor(private val repo: UserRepository) {
    suspend operator fun invoke(id: String) { /* ... */ }
}

class SearchUsersUseCase @Inject constructor(private val repo: UserRepository) {
    suspend operator fun invoke(query: String): List<User> { /* ... */ }
}

class ValidateUserEmailUseCase @Inject constructor() {
    operator fun invoke(email: String): Boolean { /* ... */ }
}
```

---

### ❌ Mistake 4: ViewModel Calling Repository Directly

```kotlin
// WRONG: Bypassing Use Cases
class UserViewModel @Inject constructor(
    private val userRepository: UserRepository, // Direct repository access!
) {
    fun loadUser(id: String) {
        viewModelScope.launch {
            val user = userRepository.getUser(id) // No business validation!
            _state.value = UserState.Success(user)
        }
    }
}
```

✅ **Correct approach:** Always go through a Use Case.

```kotlin
class UserViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase,
) {
    fun loadUser(id: String) {
        viewModelScope.launch {
            try {
                val user = getUserUseCase(id) // Business rules applied
                _state.value = UserState.Success(user)
            } catch (e: DomainException) {
                _state.value = UserState.Error(e.message)
            }
        }
    }
}
```

---

### ❌ Mistake 5: Returning Framework Types from Repository Interfaces

```kotlin
// WRONG: Retrofit Response in domain interface
interface UserRepository {
    suspend fun getUser(id: String): Response<UserDto> // Framework type!
    suspend fun getUsers(): NetworkResult<List<UserDto>> // Custom wrapper from data layer!
}
```

✅ **Correct approach:** Return plain domain entities and throw exceptions for errors.

```kotlin
interface UserRepository {
    suspend fun getUser(id: String): User
    suspend fun getUsers(): List<User>
}
```

---

### ❌ Mistake 6: Hilt Modules in Domain

```kotlin
// WRONG: Hilt module in domain layer
@Module
@InstallIn(SingletonComponent::class)
abstract class DomainModule {
    @Binds
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}
```

✅ **Correct approach:** Hilt modules that bind repository implementations belong in the **Data
layer** or a dedicated **DI module**. The Domain layer only uses `@Inject` on Use Case
constructors.

---

## 📚 Further Reading

- [Clean Architecture](../Architecture/clean-architecture.md) — The overall architecture pattern
- [SOLID Principles](../Architecture/solid-principles.md) — Foundational design principles
- [Testing Guide](../Quality/testing.md) — Comprehensive testing strategies
- [Data Layer](./data.md) — Repository implementations, DTOs, data sources
- [Presentation Layer](./presentation.md) — ViewModels, UI state, navigation
