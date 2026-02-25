# 📋 Plan Mode

**Plan Mode is a structured approach for handling complex tasks. Before implementing, think deeply, ask questions, and draft a plan.**

---

## Purpose

Plan Mode exists to:

- **Prevent costly rework** by planning before coding
- **Align on requirements** before implementation begins
- **Break complex tasks** into manageable, reviewable steps
- **Identify risks and trade-offs** early in the process
- **Create shared understanding** between developer and AI agent

---

## When to Enter Plan Mode

### Automatic Triggers

Enter Plan Mode automatically when any of these conditions are met:

1. **Complex feature** spanning multiple layers (Domain + Data + Presentation)
2. **Ambiguous requirements** that need clarification before implementation
3. **Multiple valid approaches** with different trade-offs
4. **Architectural decisions** that affect multiple modules or features
5. **User explicitly requests** "plan first", "think about this", or "plan mode"
6. **Migration or refactoring** that touches many existing files
7. **New module creation** requiring build configuration and structure decisions

### Decision Tree

```
Is the task clear and simple (<100 lines, 1-2 files)?
|
+-- YES --> Is it a well-known pattern (e.g., add a field, fix a typo)?
|           |
|           +-- YES --> Implement directly
|           +-- NO  --> Implement with brief explanation
|
+-- NO  --> Is there only one obvious approach?
            |
            +-- YES --> Implement with brief explanation of approach
            +-- NO  --> ENTER PLAN MODE
```

### Examples by Complexity

| Task | Plan Mode? | Reason |
|------|-----------|--------|
| Fix a typo in a string resource | No | Trivial, one file |
| Add a field to an existing entity | No | Simple, well-known pattern |
| Add a new Use Case with tests | Maybe | Depends on domain complexity |
| Add a new screen with ViewModel | Yes | Multiple layers, UI decisions |
| Implement offline-first caching | Yes | Architecture decision, multiple layers |
| Migrate from Gson to Kotlin Serialization | Yes | Wide-reaching change, many files |
| Add Room database to a module | Yes | New dependency, schema design |
| Refactor navigation structure | Yes | Affects multiple screens, coordination |

---

## Plan Mode Process

### Phase 1: Deep Reflection

Before responding to the user, silently analyze:

**Architecture questions:**
- What layers are affected (Domain, Data, Presentation)?
- Which modules need changes?
- Are there existing patterns to follow?
- Does this require new Hilt modules or bindings?

**Implementation questions:**
- What existing code will change?
- What new files are needed?
- What dependencies are required?
- Are there Compose-specific considerations?

**Testing questions:**
- What tests need to be written?
- What mocks or fakes are needed?
- Are there edge cases to consider?
- Do we need instrumented tests?

**Risk questions:**
- Are there performance implications?
- Could this break existing functionality?
- Are there backward compatibility concerns?
- What could go wrong?

### Phase 2: Ask Clarifying Questions

Ask 4-6 specific, actionable questions. Avoid generic questions.

**Architecture questions:**
- "Should this be a new Gradle module `:feature:search` or part of `:feature:home`?"
- "Should the navigation use a nested NavGraph or top-level routes?"
- "Do we need a new Hilt component scope for this feature?"

**Scope questions:**
- "Should the search filter by name only, or also by cuisine type and price range?"
- "Is this feature for authenticated users only or also for guests?"
- "Should we support pagination or load all results at once?"

**Data questions:**
- "Should search results be cached in Room or only kept in memory?"
- "Should we use a local-first approach or always fetch from the API?"
- "What's the expected data size? (affects pagination strategy)"

**UI/UX questions:**
- "Should the search trigger on every keystroke (with debounce) or on submit?"
- "Should we show a full-screen search or an expanding search bar?"
- "Do we need custom animations for the search results?"

**Testing questions:**
- "Should we add Compose UI tests or only ViewModel unit tests?"
- "Should we write integration tests with a fake API?"
- "Are there specific edge cases you want covered?"

**Trade-off questions:**
- "Do you prefer offline-first (more complex, better UX) or online-only (simpler)?"
- "Should we optimize for development speed or long-term maintainability?"
- "Is it acceptable to add a new library dependency for this?"

### Phase 3: Draft Implementation Plan

Present a structured plan covering all layers:

```markdown
## Implementation Plan: Restaurant Search

### Overview
Add search functionality to the home screen, allowing users to filter
restaurants by name with debounced input and local caching.

### 1. Domain Layer
- [ ] Create `SearchRestaurantsUseCase`
  - `operator fun invoke(query: String): Flow<List<Restaurant>>`
  - Empty query returns all restaurants
  - Case-insensitive matching
- [ ] Add `searchRestaurants(query: String): Flow<List<Restaurant>>`
  to `RestaurantRepository` interface

### 2. Data Layer
- [ ] Add Room DAO query:
  ```kotlin
  @Query("SELECT * FROM restaurants WHERE name LIKE '%' || :query || '%'")
  fun searchByName(query: String): Flow<List<RestaurantEntity>>
  ```
- [ ] Implement `searchRestaurants` in `RestaurantRepositoryImpl`
  - Local-first: query Room, then refresh from API
  - Map entities to domain models

### 3. Presentation Layer
- [ ] Add to `HomeUiState`:
  ```kotlin
  data class HomeUiState(
      val searchQuery: String = "",
      val restaurants: List<Restaurant> = emptyList(),
      val isSearching: Boolean = false,
  )
  ```
- [ ] Add to `HomeViewModel`:
  - `onSearchQueryChanged(query: String)` with 300ms debounce
  - Collect search results Flow
- [ ] Create `SearchBar` composable:
  - Material3 `SearchBar` component
  - Clear button when query is not empty
  - Loading indicator during search

### 4. Dependency Injection
- [ ] No new Hilt modules needed (existing `HomeModule` sufficient)
- [ ] Bind new use case in existing module

### 5. Testing
- [ ] `SearchRestaurantsUseCaseTest` (5 tests):
  - Empty query returns all restaurants
  - Partial match returns filtered results
  - No results returns empty list
  - Special characters are handled
  - Case-insensitive matching works
- [ ] `HomeViewModelTest` search tests (4 tests):
  - Search query updates state
  - Debounce waits 300ms
  - Search results update restaurant list
  - Clear search resets to all restaurants
- [ ] `RestaurantDaoTest` search query (3 tests):
  - Basic search works
  - Empty query returns all
  - No match returns empty

### 6. Files Changed/Created
| Action | File | Lines |
|--------|------|-------|
| Create | `SearchRestaurantsUseCase.kt` | ~25 |
| Modify | `RestaurantRepository.kt` | +3 |
| Modify | `RestaurantDao.kt` | +5 |
| Modify | `RestaurantRepositoryImpl.kt` | +15 |
| Modify | `HomeUiState.kt` | +5 |
| Modify | `HomeViewModel.kt` | +20 |
| Create | `SearchBar.kt` | ~60 |
| Create | `SearchRestaurantsUseCaseTest.kt` | ~80 |
| Modify | `HomeViewModelTest.kt` | +60 |
| Create | `RestaurantDaoTest.kt` | ~50 |

### 7. Estimated Complexity
- **Level**: Medium
- **Files changed**: 10
- **New files**: 4
- **Lines**: ~300
- **Risk**: Low-Medium (new DAO queries need testing)
- **Time estimate**: 60-90 minutes

### 8. Risks and Mitigations
| Risk | Impact | Mitigation |
|------|--------|------------|
| Room FTS performance on large datasets | Slow search | Add index, consider FTS4 if needed |
| Debounce timing feels sluggish | Poor UX | Make debounce configurable, test with users |
| Search conflicts with existing filters | Logic bugs | Clear search when filters change |

### 9. Implementation Order
1. Domain layer (UseCase + Repository interface)
2. Data layer (DAO + Repository implementation)
3. ViewModel (state + logic)
4. UI (SearchBar composable)
5. Tests (all layers)
6. Integration test (full flow)
```

### Phase 4: Get Approval

After presenting the plan:

- Ask "Does this plan look good? Any changes before I start?"
- Wait for explicit approval before implementing
- If user requests changes, update the plan and re-present
- Don't start implementing until the user says "go" or "approved" or similar

### Phase 5: Implementation with Progress Updates

During implementation, provide updates:

```
Step 1/5: Domain Layer
- Created SearchRestaurantsUseCase
- Added searchRestaurants to RestaurantRepository interface
Done.

Step 2/5: Data Layer
- Added search query to RestaurantDao
- Implemented searchRestaurants in RestaurantRepositoryImpl
Done.

Step 3/5: Presentation Layer
- Updated HomeUiState with search fields
- Added search logic to HomeViewModel with debounce
- Created SearchBar composable
Done.

Step 4/5: Testing
- Added SearchRestaurantsUseCaseTest (5 tests passing)
- Added HomeViewModel search tests (4 tests passing)
- Added RestaurantDaoTest (3 tests passing)
Done.

Step 5/5: Integration
- Verified search works end-to-end
- Checked dark theme support
- Verified accessibility labels
Done.

All steps complete. 12 tests passing, 10 files changed.
```

---

## Example Plan Mode Flows

### Example 1: Add Offline Support

**User request**: "Add offline support to the restaurant list."

**Questions:**
1. Should offline data be served immediately while refreshing from the API (stale-while-revalidate)?
2. How long should cached data be considered fresh? (e.g., 1 hour, 1 day)
3. Should we show a banner when data is from cache vs. live?
4. Should offline support extend to restaurant details or just the list?
5. Are we already using Room, or do we need to add it?

**Plan outline:**
1. Add Room database with RestaurantEntity
2. Create RestaurantDao with CRUD operations
3. Implement cache-first strategy in repository
4. Add connectivity check with ConnectivityManager
5. Show offline banner in UI
6. Add tests for offline scenarios

### Example 2: Implement Pagination

**User request**: "Add pagination to the restaurant list."

**Questions:**
1. Should we use Paging 3 library or manual pagination?
2. What's the page size? (e.g., 20 items)
3. Should we show a loading indicator at the bottom while loading next page?
4. Should we prefetch the next page before the user reaches the end?
5. Does the API support cursor-based or offset-based pagination?

**Plan outline:**
1. Add Paging 3 dependency
2. Create RestaurantPagingSource
3. Update repository to return PagingData
4. Update ViewModel to use Pager
5. Update UI to use LazyPagingItems
6. Add loading and error states for pagination
7. Test PagingSource with fake API responses

### Example 3: Add Dark Theme Support

**User request**: "Add dark theme support to the app."

**Questions:**
1. Should we follow system theme or add an in-app toggle?
2. Are there any custom colors beyond Material3 dynamic color?
3. Do we need to handle images differently in dark mode?
4. Should the theme preference persist (DataStore)?
5. Are there any existing composables with hardcoded colors?

**Plan outline:**
1. Define dark color scheme in Theme.kt
2. Add theme preference to DataStore
3. Create ThemeViewModel for theme state
4. Update AppTheme to switch based on preference
5. Add theme toggle in Settings screen
6. Audit all composables for hardcoded colors
7. Test both themes on all screens

---

## Plan Mode Best Practices

### DO

- Plan for tasks that touch 3 or more files
- Include testing in every plan
- Estimate complexity (Simple / Medium / Complex)
- Break large tasks into implementation phases
- Get explicit approval before implementing
- Provide progress updates during implementation
- Consider edge cases and error scenarios
- Think about accessibility implications
- Consider performance implications
- Reference existing patterns in the codebase

### DON'T

- Don't over-plan simple tasks (fixing a typo doesn't need a plan)
- Don't implement before approval
- Don't skip the questions phase
- Don't ignore user preferences or constraints
- Don't present a plan that's too vague to be actionable
- Don't forget to include testing in the plan
- Don't plan without looking at existing code first
- Don't create plans that will take more than a day to implement

---

## Complexity Assessment

| Level | Lines | Files | Time | Example |
|-------|-------|-------|------|---------|
| Simple | <100 | 1-2 | <30min | Add a computed property to an entity |
| Medium | 100-300 | 3-8 | 30-90min | New Use Case + ViewModel + tests |
| Complex | 300-600 | 8-15 | 90min-3hr | New feature module with all layers |
| Epic | >600 | >15 | >3hr | **Must break into multiple tasks** |

### Complexity Factors

| Factor | Adds Complexity |
|--------|----------------|
| Multiple layers affected | +1 level |
| New Gradle module needed | +1 level |
| Database schema changes | +1 level |
| New navigation routes | +1 level |
| API integration | +1 level |
| Existing test changes | +1 level |
| Accessibility requirements | +0.5 level |
| Animation requirements | +0.5 level |

---

## Plan Mode Templates

### Feature Template

```markdown
## Feature: [Feature Name]

### Requirements
- [ ] Requirement 1
- [ ] Requirement 2
- [ ] Requirement 3

### Architecture Decisions
- Decision 1: [chosen approach] because [reason]
- Decision 2: [chosen approach] because [reason]

### Implementation Steps
1. **Domain Layer**
   - [ ] Task 1
   - [ ] Task 2
2. **Data Layer**
   - [ ] Task 1
   - [ ] Task 2
3. **Presentation Layer**
   - [ ] Task 1
   - [ ] Task 2
4. **Dependency Injection**
   - [ ] Task 1
5. **Testing**
   - [ ] Task 1
   - [ ] Task 2
   - [ ] Task 3

### Risks
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Risk 1 | Low/Med/High | Low/Med/High | Approach |

### Estimated Complexity
- Level: Simple/Medium/Complex
- Files: X
- Lines: ~Y
- Time: Z minutes
```

### Bug Fix Template

```markdown
## Bug Fix: [Issue ID] - [Description]

### Problem
- What is happening
- What should happen
- Steps to reproduce

### Root Cause Analysis
- Where the bug originates
- Why it happens

### Fix Approach
- [ ] Step 1: Write failing test
- [ ] Step 2: Implement fix
- [ ] Step 3: Verify test passes
- [ ] Step 4: Check for similar issues elsewhere

### Regression Risk
- What else could this fix affect
- How to verify no regressions
```

### Refactoring Template

```markdown
## Refactoring: [Description]

### Current State
- What exists today
- What's wrong with it

### Desired State
- What it should look like after
- Why this is better

### Migration Steps
1. [ ] Step 1 (backward compatible)
2. [ ] Step 2 (backward compatible)
3. [ ] Step 3 (remove old code)

### Safety Checks
- [ ] All existing tests still pass
- [ ] No public API changes (or documented in CHANGELOG)
- [ ] Performance is equal or better
```

---

## Plan Mode Checklist

Before starting implementation, verify:

- [ ] Task complexity has been assessed
- [ ] Plan Mode is appropriate (not over-planning a simple task)
- [ ] Clarifying questions have been asked and answered
- [ ] Implementation plan has been drafted
- [ ] All architecture layers are covered
- [ ] Testing is included in the plan
- [ ] Risks have been identified
- [ ] Complexity has been estimated
- [ ] User has explicitly approved the plan
- [ ] Implementation order is clear

During implementation, verify:

- [ ] Progress updates are provided after each major step
- [ ] Deviations from the plan are flagged immediately
- [ ] Tests are written alongside implementation (not after)
- [ ] Each step leaves the codebase in a buildable state

After implementation, verify:

- [ ] All planned steps are complete
- [ ] All tests pass
- [ ] The implementation matches the approved plan
- [ ] Any deviations have been explained
- [ ] A summary of changes has been provided

---

## Further Reading

- [Git Commits](./git-commits.md)
- [Git Branches](./git-branches.md)
- [Clean Architecture](../Architecture/clean-architecture.md)
- [Testing Standards](../Quality/testing.md)

---

**Remember**: 5 minutes of planning saves 50 minutes of rework. Plan first, implement second.
