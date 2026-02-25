# 🔧 ARCDevTools-Android

**Centralized development tooling, linting configurations, CI/CD workflows, and Claude Code skills for all ARC Labs Studio Android projects. One submodule to enforce consistency across every Kotlin/Android repository.**

---

## 🎯 What It Provides

ARCDevTools-Android is the **single source of truth** for development tooling across all ARC Labs Android projects. By integrating it as a Git submodule, every project automatically inherits:

- **Centralized tooling configuration** for ARC Labs Android projects
- **ktlint configurations** with ARC Labs custom rules and `.editorconfig`
- **detekt configurations** with complexity, style, and custom rule sets
- **GitHub Actions workflows** for testing, linting, and release automation
- **Git hooks** (pre-commit, pre-push) to catch issues before they reach CI
- **Claude Code skills** installation for AI-assisted development
- **Makefile** with standardized commands across all projects
- **Branch protection** and GitHub label setup scripts

---

## 📋 Requirements

Before integrating ARCDevTools-Android, ensure your environment meets the following:

| Requirement | Minimum Version | Notes |
|-------------|----------------|-------|
| **Kotlin** | 2.0+ | Required for K2 compiler support |
| **Android Studio** | Latest stable | Hedgehog or newer recommended |
| **JDK** | 17 | Zulu distribution preferred |
| **Gradle** | 8.5+ | With Kotlin DSL |
| **ktlint** | 1.1.0+ | CLI or Gradle plugin |
| **detekt** | 1.23.0+ | CLI or Gradle plugin |
| **Git** | 2.30+ | Submodule support required |

### Recommended Gradle Plugins

```kotlin
// build.gradle.kts (project-level)
plugins {
    id("org.jlleitschuh.gradle.ktlint") version "12.1.0"
    id("io.gitlab.arturbosch.detekt") version "1.23.4"
}
```

---

## 🚀 Installation

### Step 1: Add as Submodule

```bash
# Navigate to your Android project root
cd /path/to/your-android-project

# Add ARCDevTools-Android as a submodule
git submodule add https://github.com/arclabs-studio/ARCDevTools-Android.git ARCDevTools

# Initialize and fetch submodule contents
git submodule update --init --recursive
```

### Step 2: Run Setup Script

```bash
# Make setup script executable (if needed)
chmod +x ./ARCDevTools/arcdevtools-setup

# Run the automated setup
./ARCDevTools/arcdevtools-setup
```

The setup script will:
- ✅ Symlink ktlint configuration (`.editorconfig`)
- ✅ Symlink detekt configuration (`detekt.yml`)
- ✅ Install Git hooks (pre-commit, pre-push)
- ✅ Copy Makefile to project root (if not present)
- ✅ Install Claude Code skills
- ✅ Verify tool versions

### Step 3: Commit Integration

```bash
# Stage all changes
git add .gitmodules ARCDevTools Makefile .editorconfig

# Commit with conventional commit message
git commit -m "chore: integrate ARCDevTools-Android"
```

### Step 4: Verify Installation

```bash
# Run the full check suite to verify everything works
make check
```

Expected output:

```
✅ ktlint configuration found
✅ detekt configuration found
✅ Git hooks installed
✅ Makefile commands available
✅ Claude Code skills installed
✅ All checks passed
```

---

## 📁 Project Structure

```
ARCDevTools/
├── configs/
│   ├── ktlint/
│   │   ├── .editorconfig            # ktlint rules via editorconfig
│   │   └── ktlint-baseline.xml      # Baseline for legacy projects
│   └── detekt/
│       ├── detekt.yml               # Main detekt configuration
│       ├── detekt-baseline.xml      # Baseline for legacy projects
│       └── custom-rules/
│           └── arclabs-rules.jar    # ARC Labs custom detekt rules
├── workflows/
│   ├── android-test.yml             # Unit & instrumented test workflow
│   ├── android-lint.yml             # ktlint + detekt + Android Lint
│   └── android-release.yml          # Release build & distribution
├── hooks/
│   ├── pre-commit                   # Lint staged files before commit
│   └── pre-push                     # Run tests before push
├── scripts/
│   ├── setup-skills.sh              # Install Claude Code skills
│   ├── setup-github-labels.sh       # Configure GitHub issue labels
│   ├── setup-branch-protection.sh   # Set branch protection rules
│   └── verify-installation.sh       # Verify ARCDevTools setup
├── skills/
│   ├── lint-fix.md                  # Skill: auto-fix lint issues
│   ├── test-generate.md             # Skill: generate test scaffolds
│   ├── review-pr.md                 # Skill: review pull requests
│   └── architecture-check.md        # Skill: verify Clean Architecture
├── Makefile                         # Standardized make commands
└── arcdevtools-setup                # Main setup script
```

### Key Files Explained

| File | Purpose |
|------|---------|
| `configs/ktlint/.editorconfig` | All ktlint formatting rules |
| `configs/detekt/detekt.yml` | Static analysis rules and thresholds |
| `hooks/pre-commit` | Runs ktlint + detekt on staged `.kt` files |
| `hooks/pre-push` | Runs full test suite before push |
| `arcdevtools-setup` | One-command project setup |

---

## 📖 Usage (Makefile Commands)

ARCDevTools-Android provides a standardized Makefile with commands that work identically across all ARC Labs Android projects.

| Command | Description |
|---------|-------------|
| `make lint` | Run ktlint + detekt on entire project |
| `make format` | Auto-fix ktlint formatting issues |
| `make test` | Run all unit tests |
| `make test-coverage` | Run tests with JaCoCo coverage report |
| `make build` | Build debug APK |
| `make build-release` | Build release APK (requires signing config) |
| `make clean` | Clean build directories |
| `make check` | Run lint + test (full validation) |
| `make detekt` | Run detekt only |
| `make ktlint` | Run ktlint only |
| `make hooks` | Re-install Git hooks |

### Makefile Contents

```makefile
.PHONY: lint format test test-coverage build build-release clean check detekt ktlint hooks

# Lint: Run all static analysis tools
lint: ktlint detekt
	@echo "✅ All lint checks passed"

# Format: Auto-fix ktlint issues
format:
	@echo "🧹 Running ktlint format..."
	./gradlew ktlintFormat
	@echo "✅ Formatting complete"

# Test: Run all unit tests
test:
	@echo "🧪 Running unit tests..."
	./gradlew test
	@echo "✅ All tests passed"

# Test with coverage: Run tests and generate JaCoCo report
test-coverage:
	@echo "🧪 Running tests with coverage..."
	./gradlew test jacocoTestReport
	@echo "📊 Coverage report: build/reports/jacoco/index.html"

# Build: Assemble debug APK
build:
	@echo "🔨 Building debug APK..."
	./gradlew assembleDebug
	@echo "✅ Build complete"

# Build Release: Assemble release APK
build-release:
	@echo "🔨 Building release APK..."
	./gradlew assembleRelease
	@echo "✅ Release build complete"

# Clean: Remove build artifacts
clean:
	@echo "🗑️  Cleaning build directories..."
	./gradlew clean
	@echo "✅ Clean complete"

# Check: Full validation (lint + test)
check: lint test
	@echo "✅ All checks passed"

# Detekt: Run detekt static analysis
detekt:
	@echo "🔍 Running detekt..."
	./gradlew detekt
	@echo "✅ Detekt passed"

# ktlint: Run ktlint check
ktlint:
	@echo "🧹 Running ktlint..."
	./gradlew ktlintCheck
	@echo "✅ ktlint passed"

# Hooks: Re-install Git hooks
hooks:
	@echo "🪝 Installing Git hooks..."
	cp ARCDevTools/hooks/pre-commit .git/hooks/pre-commit
	cp ARCDevTools/hooks/pre-push .git/hooks/pre-push
	chmod +x .git/hooks/pre-commit .git/hooks/pre-push
	@echo "✅ Git hooks installed"
```

---

## 🪝 Git Hooks

ARCDevTools-Android installs two Git hooks to catch issues before they reach CI.

### Pre-commit

The pre-commit hook runs on every `git commit` and validates only staged `.kt` files for fast feedback.

**What it does:**
- ✅ Runs ktlint on staged `.kt` files only (fast)
- ✅ Runs detekt on staged `.kt` files only (fast)
- ❌ Blocks commit if any ktlint violations are found
- ❌ Blocks commit if any detekt issues exceed threshold

```bash
#!/usr/bin/env bash
# ARCDevTools-Android pre-commit hook
# Runs ktlint and detekt on staged Kotlin files

set -e

echo "🪝 Pre-commit: Running lint checks on staged files..."

# Get list of staged .kt files
STAGED_KT_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep '\.kt$' || true)

if [ -z "$STAGED_KT_FILES" ]; then
    echo "✅ No Kotlin files staged. Skipping lint."
    exit 0
fi

echo "📋 Checking ${#STAGED_KT_FILES[@]} Kotlin file(s)..."

# Run ktlint on staged files
echo "🧹 Running ktlint..."
echo "$STAGED_KT_FILES" | xargs ktlint --editorconfig=.editorconfig
if [ $? -ne 0 ]; then
    echo ""
    echo "❌ ktlint found issues. Fix them or run 'make format' to auto-fix."
    echo "   To bypass (not recommended): git commit --no-verify"
    exit 1
fi
echo "✅ ktlint passed"

# Run detekt on staged files
echo "🔍 Running detekt..."
echo "$STAGED_KT_FILES" | xargs detekt --config ARCDevTools/configs/detekt/detekt.yml --input
if [ $? -ne 0 ]; then
    echo ""
    echo "❌ detekt found issues. Please fix them before committing."
    echo "   To bypass (not recommended): git commit --no-verify"
    exit 1
fi
echo "✅ detekt passed"

echo "✅ Pre-commit checks passed!"
```

### Pre-push

The pre-push hook runs on every `git push` and ensures the full test suite passes.

**What it does:**
- ✅ Runs the full unit test suite via Gradle
- ❌ Blocks push if any tests fail

```bash
#!/usr/bin/env bash
# ARCDevTools-Android pre-push hook
# Runs full test suite before pushing

set -e

echo "🪝 Pre-push: Running full test suite..."

# Run all unit tests
./gradlew test --quiet

if [ $? -ne 0 ]; then
    echo ""
    echo "❌ Tests failed. Fix failing tests before pushing."
    echo "   To bypass (not recommended): git push --no-verify"
    exit 1
fi

echo "✅ All tests passed. Pushing..."
```

---

## 🧹 ktlint Configuration

ARCDevTools-Android configures ktlint via `.editorconfig`, which is symlinked to the project root during setup.

### .editorconfig Rules

```editorconfig
# ARCDevTools-Android ktlint configuration
# Symlinked from ARCDevTools/configs/ktlint/.editorconfig

root = true

[*]
charset = utf-8
end_of_line = lf
indent_size = 4
indent_style = space
insert_final_newline = true
max_line_length = 120
trim_trailing_whitespace = true

[*.{kt,kts}]
# Indentation
ktlint_standard_indent = enabled

# Imports
ktlint_standard_no-wildcard-imports = enabled
ij_kotlin_imports_layout = *,java.**,javax.**,kotlin.**,^

# Spacing
ktlint_standard_no-empty-first-line-in-class-body = enabled
ktlint_standard_no-consecutive-blank-lines = enabled
ktlint_standard_no-trailing-spaces = enabled
ktlint_standard_no-blank-line-before-rbrace = enabled

# Naming
ktlint_standard_package-name = enabled
ktlint_standard_class-naming = enabled
ktlint_standard_function-naming = enabled

# Wrapping
ktlint_standard_argument-list-wrapping = enabled
ktlint_standard_parameter-list-wrapping = enabled
ktlint_standard_wrapping = enabled

# Function signatures
ktlint_standard_function-signature = enabled
ktlint_function_signature_body_expression_wrapping = multiline
ktlint_function_signature_force_multiline_when_parameter_count_greater_or_equal_than = 3

# Annotations
ktlint_standard_annotation = enabled
ktlint_standard_annotation-spacing = enabled

# Disabled rules (project preference)
ktlint_standard_filename = disabled
ktlint_standard_no-blank-line-in-list = disabled

[*.gradle.kts]
# Relaxed rules for Gradle build scripts
ktlint_standard_no-wildcard-imports = disabled
```

### Enabled Rules Summary

| Rule | Status | Description |
|------|--------|-------------|
| `no-wildcard-imports` | ✅ Enabled | Prohibits `import foo.*` |
| `indent` | ✅ Enabled | 4-space indentation |
| `max-line-length` | ✅ Enabled | 120 character limit |
| `no-consecutive-blank-lines` | ✅ Enabled | Max 1 blank line |
| `no-trailing-spaces` | ✅ Enabled | No trailing whitespace |
| `argument-list-wrapping` | ✅ Enabled | Wrap long argument lists |
| `parameter-list-wrapping` | ✅ Enabled | Wrap long parameter lists |
| `class-naming` | ✅ Enabled | PascalCase for classes |
| `function-naming` | ✅ Enabled | camelCase for functions |
| `package-name` | ✅ Enabled | Lowercase package names |
| `annotation` | ✅ Enabled | Annotation formatting |
| `filename` | ❌ Disabled | Not enforced (flexibility) |

### Custom ARC Labs ktlint Rules

ARCDevTools-Android includes custom ktlint rules specific to ARC Labs conventions:

```kotlin
// ❌ VIOLATION: Wildcard imports are never allowed
import com.example.feature.*

// ✅ CORRECT: Explicit imports only
import com.example.feature.FeatureViewModel
import com.example.feature.FeatureScreen
```

```kotlin
// ❌ VIOLATION: Line exceeds 120 characters
fun processRestaurantSearchResultsWithFilteringAndSortingAndPaginationSupport(query: String, filters: List<Filter>, sortOrder: SortOrder, page: Int): Result<List<Restaurant>> { }

// ✅ CORRECT: Properly wrapped
fun processRestaurantSearchResults(
    query: String,
    filters: List<Filter>,
    sortOrder: SortOrder,
    page: Int,
): Result<List<Restaurant>> {
}
```

---

## 🔍 detekt Configuration

ARCDevTools-Android provides a comprehensive `detekt.yml` with rules tuned for ARC Labs Android projects.

### detekt.yml Contents

```yaml
# ARCDevTools-Android detekt configuration
# Located at: ARCDevTools/configs/detekt/detekt.yml

build:
  maxIssues: 0
  excludeCorrectable: false

config:
  validation: true
  warningsAsErrors: true

complexity:
  active: true
  LongMethod:
    active: true
    threshold: 30
  LargeClass:
    active: true
    threshold: 200
  CyclomaticComplexMethod:
    active: true
    threshold: 10
  LongParameterList:
    active: true
    functionThreshold: 5
    constructorThreshold: 8
    ignoreDefaultParameters: true
    ignoreAnnotated:
      - "Composable"
  NestedBlockDepth:
    active: true
    threshold: 4
  TooManyFunctions:
    active: true
    thresholdInFiles: 15
    thresholdInClasses: 12
    thresholdInInterfaces: 8
    thresholdInObjects: 10
    thresholdInEnums: 8
  ComplexCondition:
    active: true
    threshold: 4

style:
  active: true
  ForbiddenComment:
    active: true
    values:
      - "TODO:"
      - "FIXME:"
      - "HACK:"
    allowedPatterns: "TODO\\(\\w+\\):"
  MagicNumber:
    active: true
    ignoreNumbers:
      - "-1"
      - "0"
      - "1"
      - "2"
    ignoreHashCodeFunction: true
    ignorePropertyDeclaration: true
    ignoreAnnotation: true
    ignoreEnums: true
    ignoreRanges: true
    ignoreCompanionObjectPropertyDeclaration: true
  MaxLineLength:
    active: true
    maxLineLength: 120
    excludeCommentStatements: true
  ReturnCount:
    active: true
    max: 3
    excludeLabeled: true
    excludeReturnFromLambda: true
    excludeGuardClauses: true
  WildcardImport:
    active: true
  UnnecessaryAbstractClass:
    active: true
  UnusedPrivateMember:
    active: true
    allowedNames: "(_|ignored|expected|serialVersionUID)"
  UseDataClass:
    active: true

exceptions:
  active: true
  SwallowedException:
    active: true
    ignoredExceptionTypes:
      - "InterruptedException"
      - "CancellationException"
  TooGenericExceptionCaught:
    active: true
    exceptionNames:
      - "Exception"
      - "RuntimeException"
      - "Throwable"
  TooGenericExceptionThrown:
    active: true
    exceptionNames:
      - "Exception"
      - "RuntimeException"
      - "Throwable"

naming:
  active: true
  ClassNaming:
    active: true
    classPattern: "[A-Z][a-zA-Z0-9]*"
  FunctionNaming:
    active: true
    functionPattern: "[a-z][a-zA-Z0-9]*"
    ignoreAnnotated:
      - "Composable"
  PackageNaming:
    active: true
    packagePattern: "[a-z]+(\\.[a-z][A-Za-z0-9]*)*"
  VariableNaming:
    active: true
    variablePattern: "[a-z][A-Za-z0-9]*"
    privateVariablePattern: "_?[a-z][A-Za-z0-9]*"

performance:
  active: true
  SpreadOperator:
    active: true
  ForEachOnRange:
    active: true
  UnnecessaryTemporaryInstantiation:
    active: true
```

### ARC Labs Custom detekt Rules

Beyond the standard detekt rule set, ARCDevTools-Android enforces ARC Labs-specific rules:

#### Rule: No `!!` Operator

```kotlin
// ❌ VIOLATION: Not-null assertion operator is forbidden
val name = user!!.name

// ✅ CORRECT: Use safe calls or require
val name = user?.name ?: throw IllegalStateException("User must not be null")
val name = requireNotNull(user).name
```

#### Rule: No Mutable State Exposure from ViewModel

```kotlin
// ❌ VIOLATION: Exposing MutableStateFlow from ViewModel
class FeatureViewModel : ViewModel() {
    val state = MutableStateFlow<UiState>(UiState.Idle)  // BAD: mutable exposed
}

// ✅ CORRECT: Expose immutable StateFlow
class FeatureViewModel : ViewModel() {
    private val _state = MutableStateFlow<UiState>(UiState.Idle)
    val state: StateFlow<UiState> = _state.asStateFlow()
}
```

#### Rule: No Business Logic in Composables

```kotlin
// ❌ VIOLATION: Business logic inside a Composable function
@Composable
fun RestaurantListScreen(restaurants: List<Restaurant>) {
    val filtered = restaurants.filter { it.rating > 4.0 }  // BAD: logic in Composable
    val sorted = filtered.sortedByDescending { it.rating }  // BAD: logic in Composable
    LazyColumn {
        items(sorted) { RestaurantCard(it) }
    }
}

// ✅ CORRECT: Business logic in ViewModel, Composable only renders
@Composable
fun RestaurantListScreen(viewModel: RestaurantListViewModel) {
    val uiState by viewModel.state.collectAsStateWithLifecycle()
    LazyColumn {
        items(uiState.restaurants) { RestaurantCard(it) }
    }
}
```

---

## 🔄 GitHub Actions Workflows

ARCDevTools-Android includes three production-ready GitHub Actions workflows.

### Test Workflow (`android-test.yml`)

Runs on every push and pull request to `main` and `develop`.

```yaml
name: Android Tests
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4

      - name: Cache Gradle packages
        uses: actions/cache@v4
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: gradle-${{ runner.os }}-${{ hashFiles('**/*.gradle.kts') }}
          restore-keys: gradle-${{ runner.os }}-

      - name: Run Unit Tests
        run: ./gradlew test

      - name: Generate Coverage Report
        run: ./gradlew jacocoTestReport

      - name: Upload Test Results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: '**/build/reports/tests/'

      - name: Upload Coverage Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage-report
          path: '**/build/reports/jacoco/'
```

### Lint Workflow (`android-lint.yml`)

Runs on every pull request.

```yaml
name: Android Lint
on:
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4

      - name: Run ktlint
        run: ./gradlew ktlintCheck

      - name: Run detekt
        run: ./gradlew detekt

      - name: Run Android Lint
        run: ./gradlew lint

      - name: Upload Lint Results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: lint-results
          path: |
            **/build/reports/ktlint/
            **/build/reports/detekt/
            **/build/reports/lint-results*.html
```

### Release Workflow (`android-release.yml`)

Triggered manually or on version tags.

```yaml
name: Android Release
on:
  push:
    tags:
      - 'v*'
  workflow_dispatch:
    inputs:
      build_type:
        description: 'Build type (release/debug)'
        required: true
        default: 'release'

jobs:
  release:
    runs-on: ubuntu-latest
    timeout-minutes: 45

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4

      - name: Run all checks
        run: |
          ./gradlew ktlintCheck
          ./gradlew detekt
          ./gradlew test

      - name: Build Release APK
        run: ./gradlew assembleRelease

      - name: Build Release Bundle (AAB)
        run: ./gradlew bundleRelease

      - name: Upload Release Artifacts
        uses: actions/upload-artifact@v4
        with:
          name: release-artifacts
          path: |
            app/build/outputs/apk/release/
            app/build/outputs/bundle/release/
```

---

## 🤖 Claude Code Skills

ARCDevTools-Android includes Claude Code skills that provide AI-assisted development workflows.

### Installation

Skills are automatically installed during `./ARCDevTools/arcdevtools-setup`. To manually re-install:

```bash
# Run skills setup script
./ARCDevTools/scripts/setup-skills.sh
```

### Available Skills

| Skill | Command | Description |
|-------|---------|-------------|
| **Lint Fix** | `/lint-fix` | Analyze and auto-fix lint violations |
| **Test Generate** | `/test-generate` | Generate test scaffolds for a class |
| **PR Review** | `/review-pr` | Review a pull request for ARC Labs standards |
| **Architecture Check** | `/architecture-check` | Verify Clean Architecture compliance |

### Manual Re-installation

If skills are not working correctly:

```bash
# Remove existing skills
rm -rf .claude/skills/

# Re-run setup
./ARCDevTools/scripts/setup-skills.sh

# Verify installation
ls -la .claude/skills/
```

---

## 🔄 Updating ARCDevTools

When ARCDevTools-Android receives updates, pull the latest version into your project:

```bash
# Navigate to the submodule
cd ARCDevTools

# Fetch and pull latest changes
git fetch origin
git pull origin main

# Return to project root
cd ..

# Re-run setup to apply any new configurations
./ARCDevTools/arcdevtools-setup

# Stage and commit the submodule update
git add ARCDevTools
git commit -m "chore: update ARCDevTools-Android"
```

### Checking for Updates

```bash
# Check if ARCDevTools has new commits
cd ARCDevTools
git log HEAD..origin/main --oneline
cd ..
```

If output is empty, you are up to date. Otherwise, follow the update steps above.

---

## ⚙️ Customization

While ARCDevTools-Android provides sensible defaults, projects may need to customize certain rules.

### Project-Specific detekt Rules

Create a `detekt-overrides.yml` in your project root to override specific rules:

```yaml
# detekt-overrides.yml
# Project-specific overrides (merged with ARCDevTools config)

complexity:
  LongParameterList:
    constructorThreshold: 10  # Override: allow more constructor params

style:
  MagicNumber:
    ignoreNumbers:
      - "-1"
      - "0"
      - "1"
      - "2"
      - "100"  # Added: common percentage value
```

Then configure Gradle to use both:

```kotlin
// build.gradle.kts
detekt {
    config.setFrom(
        files(
            "ARCDevTools/configs/detekt/detekt.yml",
            "detekt-overrides.yml"
        )
    )
}
```

### Disabling Rules Inline

For specific cases where a rule must be suppressed:

```kotlin
// Suppress a single detekt rule
@Suppress("MagicNumber")
val maxRetries = 3

// Suppress multiple rules
@Suppress("MagicNumber", "LongMethod")
fun complexInitialization() {
    // ...
}

// Suppress for an entire file (place at top of file)
@file:Suppress("TooManyFunctions")
package com.arclabs.feature.utils
```

### Disabling ktlint Rules Inline

```kotlin
// Disable ktlint for a specific block
/* ktlint-disable no-wildcard-imports */
import android.widget.*
/* ktlint-enable no-wildcard-imports */
```

### Custom ktlint .editorconfig Overrides

Add project-specific overrides in your root `.editorconfig`:

```editorconfig
# Project-level overrides (after ARCDevTools defaults)
[*.{kt,kts}]
ktlint_standard_filename = enabled  # Re-enable filename rule
```

---

## 🔍 Troubleshooting

### ktlint Not Found

**Symptom:** `make lint` fails with "ktlint: command not found"

**Solution:**
```bash
# Ensure ktlint Gradle plugin is applied
# build.gradle.kts (project-level)
plugins {
    id("org.jlleitschuh.gradle.ktlint") version "12.1.0"
}

# Or install CLI
brew install ktlint
```

### detekt Config Not Loading

**Symptom:** detekt runs with default rules instead of ARC Labs config

**Solution:**
```bash
# Verify symlink exists
ls -la detekt.yml

# If missing, re-run setup
./ARCDevTools/arcdevtools-setup

# Or manually configure in build.gradle.kts
detekt {
    config.setFrom(files("ARCDevTools/configs/detekt/detekt.yml"))
}
```

### Git Hooks Not Running

**Symptom:** Commits proceed without lint checks

**Solution:**
```bash
# Verify hooks are installed
ls -la .git/hooks/pre-commit
ls -la .git/hooks/pre-push

# Re-install hooks
make hooks

# Verify hooks are executable
chmod +x .git/hooks/pre-commit .git/hooks/pre-push
```

### Submodule Issues

**Symptom:** `ARCDevTools/` directory is empty after clone

**Solution:**
```bash
# Initialize and update submodules
git submodule update --init --recursive

# If submodule is in a detached HEAD state
cd ARCDevTools
git checkout main
cd ..
```

**Symptom:** Submodule shows as modified but no changes were made

**Solution:**
```bash
# Reset submodule to expected commit
git submodule update --force
```

### Gradle Build Cache Conflicts

**Symptom:** Lint results are stale or inconsistent after updating ARCDevTools

**Solution:**
```bash
# Clear Gradle build cache
./gradlew clean
rm -rf .gradle/

# Re-run checks
make check
```

### CI Workflow Failures

**Symptom:** GitHub Actions fail with submodule errors

**Solution:** Ensure your checkout step includes `submodules: recursive`:

```yaml
- uses: actions/checkout@v4
  with:
    submodules: recursive  # Required for ARCDevTools
```

---

## ✅ Best Practices

Follow these guidelines to get the most out of ARCDevTools-Android:

1. **Never bypass Git hooks** -- If the hooks catch an issue, fix it rather than using `--no-verify`
2. **Run `make check` before opening a PR** -- Catch issues locally before CI
3. **Keep ARCDevTools updated** -- Pull the latest submodule monthly at minimum
4. **Use inline suppression sparingly** -- Every `@Suppress` should have a justifying comment
5. **Do not modify ARCDevTools files directly** -- Use project-level overrides instead
6. **Commit `.editorconfig` to your repo** -- Ensures IDE and CI use the same rules
7. **Review detekt reports regularly** -- Use `build/reports/detekt/` to track code quality trends
8. **Configure IDE to use project ktlint** -- Ensures real-time feedback while coding
9. **Add `ARCDevTools` to your CI cache key** -- Invalidate cache when tooling config changes
10. **Use baselines for legacy projects** -- Adopt incrementally rather than fixing everything at once

### IDE Integration

#### Android Studio ktlint Plugin

1. Install the **ktlint** plugin from JetBrains Marketplace
2. Configure it to use the project `.editorconfig`
3. Enable "Format on save" for consistent formatting

#### Android Studio detekt Plugin

1. Install the **Detekt** plugin from JetBrains Marketplace
2. Point configuration to `ARCDevTools/configs/detekt/detekt.yml`
3. Enable "Run detekt on the fly" for real-time feedback

---

## 📚 Further Reading

- **ARCKnowledge-Android:** [https://github.com/arclabs-studio/ARCKnowledge-Android](https://github.com/arclabs-studio/ARCKnowledge-Android) -- Full Android coding standards and architecture guides
- **ARCDevTools-Android Repository:** [https://github.com/arclabs-studio/ARCDevTools-Android](https://github.com/arclabs-studio/ARCDevTools-Android) -- Source code and release notes
- **ktlint Documentation:** [https://pinterest.github.io/ktlint/](https://pinterest.github.io/ktlint/) -- Official ktlint rules reference
- **detekt Documentation:** [https://detekt.dev/](https://detekt.dev/) -- Official detekt rules and configuration
- **Conventional Commits:** [https://www.conventionalcommits.org/](https://www.conventionalcommits.org/) -- Commit message standard used by ARC Labs
- **GitHub Actions for Android:** [https://docs.github.com/en/actions](https://docs.github.com/en/actions) -- CI/CD workflow documentation
