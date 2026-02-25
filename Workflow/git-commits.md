# 📝 Git Commit Standards

**Consistent commit messages improve readability, automate changelogs, and enable semantic versioning. ARC Labs follows Conventional Commits.**

---

## Commit Message Format

### Structure

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Components

#### Type (Required)

The type describes the category of change:

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | `feat(home): add restaurant search` |
| `fix` | Bug fix | `fix(profile): resolve crash on empty avatar` |
| `docs` | Documentation | `docs(readme): update installation guide` |
| `style` | Formatting (no logic change) | `style(app): apply ktlint formatting` |
| `refactor` | Code restructuring | `refactor(data): extract UserMapper` |
| `perf` | Performance improvement | `perf(list): add LazyColumn keys` |
| `test` | Adding/fixing tests | `test(usecase): add GetUser edge cases` |
| `chore` | Maintenance | `chore(deps): update Compose BOM` |
| `build` | Build system changes | `build(gradle): enable configuration cache` |
| `ci` | CI/CD changes | `ci(actions): add instrumented test job` |
| `revert` | Revert previous commit | `revert: revert "feat(home): add search"` |

#### Scope (Optional)

The scope describes the module or feature affected by the change.

**Module scopes:**
- `app` - Main application module
- `home` - Home feature module
- `profile` - Profile feature module
- `settings` - Settings feature module
- `core` - Core/shared module
- `network` - Networking module
- `database` - Database/persistence module
- `design` - Design system module

**Layer scopes:**
- `domain` - Domain layer (use cases, entities, repository interfaces)
- `data` - Data layer (repository implementations, data sources, DTOs)
- `presentation` - Presentation layer (ViewModels, Composables, UI state)

**Tool scopes:**
- `gradle` - Gradle build scripts
- `ci` - CI/CD configuration
- `deps` - Dependency updates
- `lint` - Linting configuration (ktlint, detekt)

**Convention**: Use the most specific scope available. For changes that span multiple modules, use the primary module affected.

#### Subject (Required)

The subject is a short description of the change:

- Use imperative mood ("add" not "added" or "adds")
- Lowercase first letter
- No period at the end
- Maximum 50 characters
- Complete the sentence: "This commit will..."

**Good subjects:**
```
add restaurant search bar
resolve crash on empty list
update Compose BOM to 2024.12.01
extract user mapping to extension
```

**Bad subjects:**
```
Added restaurant search bar          # Past tense
Adds restaurant search bar           # Third person
add restaurant search bar.           # Period at end
Add Restaurant Search Bar            # Capitalized
update stuff                         # Vague
```

#### Body (Optional)

The body provides additional context about the change:

- Blank line between subject and body (required)
- Explain WHAT changed and WHY (not HOW)
- Wrap at 72 characters per line
- Use bullet points for multiple items

**Example with body:**
```
feat(home): add restaurant search

Add a search bar to the home screen that filters restaurants
by name. Uses a debounced Flow to avoid excessive recompositions
during typing.

- SearchBar composable with Material3 styling
- 300ms debounce on query changes
- Case-insensitive matching
```

#### Footer (Optional)

The footer contains metadata about the commit:

**Breaking changes:**
```
BREAKING CHANGE: RestaurantRepository now returns Flow<List<Restaurant>>
instead of suspend fun. All callers must be updated to collect the flow.
```

**Issue references:**
```
Closes ARC-123
Fixes ARC-456
Relates-to ARC-789
```

**Co-authored commits:**
```
Co-authored-by: Jane Doe <jane@arclabs.com>
Co-authored-by: John Smith <john@arclabs.com>
```

**Multiple footers:**
```
feat(home): add restaurant search

Implement full-text search for restaurants using Room FTS4.

Closes ARC-123
Co-authored-by: Jane Doe <jane@arclabs.com>
```

---

## Complete Examples

### Simple Feature
```
feat(home): add restaurant card component
```

### Feature with Body
```
feat(home): add restaurant search

Implement search functionality with debounced input.
Results update as the user types with a 300ms delay
to prevent excessive API calls.
```

### Feature with Full Message
```
feat(home): add restaurant search with filters

Implement restaurant search with support for filtering
by cuisine type and price range. The search uses a
combination of local Room FTS and remote API fallback.

- Add SearchRestaurantsUseCase with filter parameters
- Create SearchBar composable with filter chips
- Implement FTS4 virtual table for local search
- Add API endpoint for remote search fallback

Closes ARC-123
Co-authored-by: Jane Doe <jane@arclabs.com>
```

### Bug Fix
```
fix(profile): resolve crash when avatar URL is null

The ProfileScreen crashed with a NullPointerException when
the user had no avatar set. The Coil AsyncImage now uses
a placeholder and error drawable.

Fixes ARC-456
```

### Refactoring
```
refactor(data): extract user mapping to extension functions

Move UserDTO-to-User mapping from the repository implementation
to extension functions for better testability and reuse.

No functional changes. All existing tests pass.
```

### Documentation
```
docs(readme): update installation instructions

Add Gradle dependency snippet and update minimum SDK
requirements to reflect the new API 26 minimum.
```

### Build Changes
```
build(gradle): enable Gradle configuration cache

Enable configuration cache to improve build times.
Resolves incompatibility with Hilt Gradle plugin
by updating to version 2.51.1.
```

### Breaking Change
```
feat(network): migrate to Kotlin Serialization

Replace Gson with Kotlin Serialization for JSON parsing.
This provides better Kotlin support and compile-time safety.

BREAKING CHANGE: All DTO classes must now use @Serializable
annotation instead of Gson @SerializedName. Custom TypeAdapters
must be replaced with custom KSerializers.

Closes ARC-789
```

### Revert
```
revert: revert "feat(home): add restaurant search"

This reverts commit abc1234def5678.

Reason: Search API is not ready for production. The backend
team needs two more sprints to finalize the search endpoint.
Will re-implement when ARC-900 is complete.
```

---

## Commit Frequency

### When to Commit

Commit at these natural breakpoints:

1. **After completing a logical unit of work**
   - Finished implementing a composable
   - Completed a use case with tests
   - Fixed a specific bug

2. **Before switching context**
   - Moving to a different feature
   - Pausing work for the day
   - Starting a refactor after adding a feature

3. **After getting tests to pass**
   - Green tests are a natural commit point
   - Ensures the commit is in a working state

4. **Before and after refactoring**
   - Commit working code before refactoring
   - Commit the refactored version separately
   - Makes it easy to revert if the refactor goes wrong

### Atomic Commits

Each commit should represent ONE logical change:

**Good: Atomic commits**
```
feat(domain): add SearchRestaurantsUseCase
feat(data): implement search in RestaurantRepositoryImpl
feat(home): add SearchBar composable
feat(home): connect search to HomeViewModel
test(home): add SearchRestaurantsUseCase tests
test(home): add HomeViewModel search tests
```

**Bad: Non-atomic commits**
```
feat(home): add search feature with tests and refactor data layer
```

Each atomic commit should:
- Pass all tests independently
- Be revertable without side effects
- Have a clear, single purpose
- Build successfully on its own

### Commit Size Guidelines

| Size | Lines Changed | Files | Ideal For |
|------|--------------|-------|-----------|
| Small | < 50 | 1-3 | Bug fixes, style changes, small features |
| Medium | 50-200 | 3-8 | Feature additions, refactors |
| Large | 200-500 | 8-15 | Major features (consider splitting) |
| Too Large | > 500 | > 15 | **Must split into smaller commits** |

**Rule of thumb**: If you can't describe the commit in a single subject line under 50 characters, the commit is probably too large.

---

## Branch-Specific Conventions

### Feature Branches

Feature branches typically have multiple commits following a logical progression:

```
feat(domain): add Restaurant entity
feat(domain): add GetRestaurantsUseCase
feat(data): implement RestaurantRepositoryImpl
feat(home): add RestaurantCard composable
feat(home): add RestaurantListScreen
feat(home): connect screen to ViewModel
test(domain): add GetRestaurantsUseCase tests
test(home): add HomeViewModel tests
docs(home): add KDoc to public APIs
```

### Bugfix Branches

Bugfix branches are typically smaller and more focused:

```
test(profile): add failing test for null avatar crash
fix(profile): handle null avatar URL with placeholder
```

**Best practice**: Write the failing test first, then the fix. This follows TDD and ensures the bug is covered.

### Hotfix Branches

Hotfix branches are minimal and surgical:

```
fix(auth): patch token refresh race condition
test(auth): add concurrent token refresh test
```

### Release Branches

Release branches contain only preparation commits:

```
chore(release): bump version to 1.2.0
docs(changelog): update CHANGELOG for 1.2.0
fix(proguard): add missing keep rules for release
```

---

## Special Cases

### WIP Commits (Local Only)

Work-in-progress commits are acceptable locally but must never be pushed:

```
WIP: restaurant detail layout in progress
WIP: experimenting with animation approach
```

**Rules for WIP commits:**
- Use `WIP:` prefix (all caps)
- Only exist on local branches
- Squash or amend before pushing
- Never merge a WIP commit

**Cleaning up WIP commits before push:**
```bash
# Interactive rebase to squash WIP commits
git rebase -i HEAD~5
# Change 'pick' to 'squash' or 'fixup' for WIP commits
```

### Merge Commits

When merging branches, use the default merge commit format:

```
Merge branch 'feature/ARC-123-restaurant-search' into develop
```

**For squash merges (preferred for feature branches):**
```
feat(home): add restaurant search (#42)

* feat(domain): add SearchRestaurantsUseCase
* feat(data): implement search in repository
* feat(home): add SearchBar composable
* test(home): add search tests

Closes ARC-123
```

### Revert Commits

Always explain WHY the commit is being reverted:

```
revert: revert "feat(home): add restaurant search"

This reverts commit abc1234def5678.

Reason: The search API endpoint is returning incorrect
results for queries with special characters. Reverting
until the backend team resolves ARC-500.
```

### Dependency Updates

Group related dependency updates:

```
chore(deps): update Compose BOM to 2024.12.01

Updates all Compose libraries to the December 2024 release:
- compose-ui: 1.7.6
- compose-material3: 1.3.1
- compose-foundation: 1.7.6
```

### CI/CD Changes

```
ci(actions): add instrumented test job

Add a GitHub Actions job that runs instrumented tests
on an API 34 emulator. Tests run on every PR to develop.
```

---

## Commit Message Validation

### Pre-Commit Hook

Install a Git hook to validate commit messages automatically:

```bash
#!/bin/bash
# .git/hooks/commit-msg
# Make executable: chmod +x .git/hooks/commit-msg

commit_msg_file="$1"
commit_msg=$(cat "$commit_msg_file")

# Extract first line
first_line=$(echo "$commit_msg" | head -1)

# Conventional Commits pattern
pattern="^(feat|fix|docs|style|refactor|perf|test|chore|build|ci|revert)(\([a-z][a-z0-9-]*\))?: .{1,50}$"

# Allow WIP commits locally
wip_pattern="^WIP: .+"

# Allow merge commits
merge_pattern="^Merge "

# Allow revert commits
revert_pattern="^revert: "

if echo "$first_line" | grep -qE "$pattern"; then
    exit 0
elif echo "$first_line" | grep -qE "$wip_pattern"; then
    exit 0
elif echo "$first_line" | grep -qE "$merge_pattern"; then
    exit 0
elif echo "$first_line" | grep -qE "$revert_pattern"; then
    exit 0
else
    echo ""
    echo "ERROR: Invalid commit message format."
    echo ""
    echo "First line: $first_line"
    echo ""
    echo "Expected format: <type>(<scope>): <subject>"
    echo ""
    echo "Valid types: feat, fix, docs, style, refactor, perf, test, chore, build, ci, revert"
    echo ""
    echo "Examples:"
    echo "  feat(home): add restaurant search"
    echo "  fix(profile): resolve crash on empty avatar"
    echo "  chore(deps): update Compose BOM"
    echo ""
    exit 1
fi
```

### GitHub Actions Validation

```yaml
# .github/workflows/commit-lint.yml
name: Commit Lint
on:
  pull_request:
    branches: [develop, main]

jobs:
  commitlint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Validate commit messages
        run: |
          pattern="^(feat|fix|docs|style|refactor|perf|test|chore|build|ci|revert)(\([a-z][a-z0-9-]*\))?: .{1,50}$"
          invalid=0
          while IFS= read -r msg; do
            if ! echo "$msg" | grep -qE "$pattern"; then
              echo "Invalid: $msg"
              invalid=$((invalid + 1))
            fi
          done < <(git log --format=%s origin/develop..HEAD)
          if [ $invalid -gt 0 ]; then
            echo "Found $invalid invalid commit message(s)"
            exit 1
          fi
```

---

## Changelog Generation

### Automated Changelog from Commits

Well-formatted commits enable automated changelog generation:

```bash
# Generate changelog entries from commits
git log --format="- %s" v1.1.0..v1.2.0 | grep -E "^- (feat|fix)" | sort
```

**Output:**
```
- feat(home): add restaurant search with filters
- feat(profile): add avatar upload
- fix(home): resolve crash on empty restaurant list
- fix(network): handle timeout errors gracefully
```

### Mapping Types to Changelog Sections

| Commit Type | Changelog Section |
|-------------|------------------|
| `feat` | Added |
| `fix` | Fixed |
| `perf` | Performance |
| `refactor` | Changed |
| `docs` | Documentation |
| `BREAKING CHANGE` | Breaking Changes |

---

## Best Practices Summary

### DO

- Use imperative mood ("add", "fix", "update")
- Keep subject under 50 characters
- Reference issue IDs in the footer
- Write atomic commits (one logical change)
- Explain WHY in the body, not HOW
- Separate subject from body with a blank line
- Wrap body at 72 characters
- Test before committing
- Review your diff before committing

### DON'T

- Don't use past tense ("added", "fixed")
- Don't end the subject with a period
- Don't combine unrelated changes in one commit
- Don't commit generated files (build/, .gradle/)
- Don't use vague messages ("fix stuff", "update code", "WIP")
- Don't commit secrets or API keys
- Don't commit commented-out code
- Don't push WIP commits
- Don't force push to shared branches

---

## Quick Reference Card

### Commit Types
```
feat(scope):     New feature
fix(scope):      Bug fix
docs(scope):     Documentation only
style(scope):    Formatting, no logic change
refactor(scope): Code restructuring, no behavior change
perf(scope):     Performance improvement
test(scope):     Adding or fixing tests
chore(scope):    Maintenance, tooling
build(scope):    Build system, dependencies
ci(scope):       CI/CD configuration
revert:          Revert a previous commit
```

### Subject Rules
```
- Imperative mood: "add" not "added"
- Lowercase first letter
- No period at end
- Max 50 characters
```

### Body Rules
```
- Blank line after subject
- Wrap at 72 characters
- Explain WHAT and WHY
```

### Footer Rules
```
- Breaking changes: BREAKING CHANGE: description
- Issue references: Closes ARC-123
- Co-authors: Co-authored-by: Name <email>
```

---

## Further Reading

- [Conventional Commits Specification](https://www.conventionalcommits.org/)
- [How to Write a Good Commit Message](https://chris.beams.io/posts/git-commit/)
- [Git Branches](./git-branches.md)
- [Plan Mode](./plan-mode.md)

---

**Remember**: A good commit message tells a story. Future developers (and AI agents) will thank you.
