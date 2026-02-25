# 🧩 Skills Index

**Skills are AI agent context files that provide specialized knowledge for specific tasks. This index helps you find the right skill.**

---

## Skill Sources Overview

| Source | Type | Focus | Count |
|--------|------|-------|-------|
| **ARC Labs** | Internal Standards | Architecture, quality, workflow | 9 |
| **External** | Community | Android-specific tools | Varies |
| **MCP Servers** | Documentation | Official Android docs | 1 |

---

## Quick Decision Guide

```
Need to...                                 Use
-------------------------------------------------------------------
Set up a new project/module                /arc-project-setup
Design architecture for a feature          /arc-android-architecture
Write Composables or ViewModels            /arc-presentation-layer
Implement API calls, Room, DataStore       /arc-data-layer
Write or review tests                      /arc-tdd-patterns
Review code quality, lint, docs            /arc-quality-standards
Final review before merge                  /arc-final-review
Audit entire project quality               /arc-audit
Make commits, create PRs                   /arc-workflow
```

---

## ARC Labs Skills (Internal Standards)

These are the primary skills maintained by ARC Labs Studio. They encode our standards and patterns for Android development.

### /arc-android-architecture

**Purpose**: Clean Architecture, MVVM, Hilt DI, Navigation Compose

**When to use**:
- Designing a new feature or module
- Deciding how to structure layers
- Setting up Hilt modules and bindings
- Configuring navigation graphs

**Key knowledge**:
- Clean Architecture layer rules (Presentation -> Domain <- Data)
- ViewModel patterns with StateFlow and sealed interface UiState
- Hilt module organization (AppModule, NetworkModule, RepositoryModule)
- Type-safe Navigation Compose with Kotlin Serialization routes

### /arc-presentation-layer

**Purpose**: Composables, @HiltViewModel, StateFlow, Material3

**When to use**:
- Building new screens or components
- Implementing ViewModel state management
- Working with Material Design 3
- Handling UI events and navigation

**Key knowledge**:
- Composable function conventions and patterns
- ViewModel with StateFlow + UiState sealed interface
- Material3 theming (dynamic color, dark theme)
- Screen vs. content pattern (testability)
- Lifecycle-aware state collection

### /arc-data-layer

**Purpose**: Repositories, Retrofit, Room, DataStore, DTOs

**When to use**:
- Implementing API calls with Retrofit
- Setting up Room database and DAOs
- Implementing repository pattern
- Working with DataStore preferences
- Mapping between DTOs, entities, and domain models

**Key knowledge**:
- Repository implementation patterns (cache-first, network-first)
- Room database setup, DAOs, entities, migrations
- Retrofit service interfaces and interceptors
- DataStore for preferences and typed data
- Mapper patterns for layer boundaries

### /arc-tdd-patterns

**Purpose**: JUnit 5, MockK, Turbine, TDD workflow

**When to use**:
- Writing unit tests for ViewModels, Use Cases, Repositories
- Testing Kotlin Flows and coroutines
- Setting up test fixtures and fakes
- Following TDD (Red-Green-Refactor)

**Key knowledge**:
- Given-When-Then test structure
- `makeSUT()` factory method pattern
- MockK mocking and verification
- Turbine for Flow testing
- Coroutines test dispatcher setup
- Compose UI testing

### /arc-quality-standards

**Purpose**: Code review, ktlint/detekt, documentation, accessibility

**When to use**:
- Reviewing code for quality
- Configuring linting rules
- Writing KDoc documentation
- Checking accessibility compliance

**Key knowledge**:
- ktlint and detekt rule configuration
- KDoc standards for public APIs
- Accessibility checklist (TalkBack, content descriptions)
- Code review checklist
- Performance review checklist

### /arc-workflow

**Purpose**: Git commits, branches, PRs, Plan Mode

**When to use**:
- Making commits (Conventional Commits format)
- Creating branches (naming conventions)
- Opening pull requests (PR template)
- Planning complex tasks (Plan Mode)

**Key knowledge**:
- Conventional Commits types and scopes
- Branch naming: `<type>/<issue-id>-<description>`
- PR template with summary, changes, testing sections
- Plan Mode process (reflect, ask, plan, approve, implement)

### /arc-project-setup

**Purpose**: New modules/apps, ARCDevTools, CI/CD

**When to use**:
- Creating a new Android project from scratch
- Adding a new feature module to an existing project
- Configuring CI/CD with GitHub Actions
- Setting up ARCDevTools integration

**Key knowledge**:
- Standard project directory layout
- build.gradle.kts templates
- Version catalog setup
- GitHub Actions workflow templates
- ARCDevTools configuration

### /arc-final-review

**Purpose**: Comprehensive quality check before merge

**When to use**:
- Before merging a PR to develop
- Before a release branch merge to main
- When you want a thorough review of all changes

**Key knowledge**:
- Architecture compliance checklist
- Code quality checklist
- Testing verification
- Documentation completeness
- Performance considerations
- Accessibility verification

### /arc-audit

**Purpose**: Full project audit by domain

**When to use**:
- Periodic project health checks
- Onboarding to a new project
- Preparing for a major release
- After significant refactoring

**Key knowledge**:
- Architecture audit (layer violations, dependency direction)
- Code quality audit (lint, formatting, naming)
- Testing audit (coverage, test quality)
- Documentation audit (KDoc, README, CHANGELOG)
- Dependency audit (outdated, unused, vulnerable)

---

## External Skills (Community)

### Android-Specific Skills

| Skill | Purpose | Source |
|-------|---------|--------|
| `compose-ui` | Advanced Compose UI patterns, custom layouts, animations | Community |
| `kotlin-coroutines` | Coroutines and Flow advanced patterns, structured concurrency | Community |
| `hilt-advanced` | Advanced DI patterns, custom scopes, assisted inject | Community |
| `room-migrations` | Database migration strategies, auto-migration, testing | Community |
| `gradle-optimization` | Build speed optimization, configuration cache, build scans | Community |
| `compose-testing` | Advanced Compose UI testing, screenshot testing | Community |

### General Development Skills

| Skill | Purpose | Source |
|-------|---------|--------|
| `git-advanced` | Rebase strategies, cherry-pick, bisect | Community |
| `ci-cd` | GitHub Actions advanced patterns, matrix builds | Community |
| `code-review` | Effective code review practices | Community |
| `performance` | Android app performance profiling and optimization | Community |

### MCP Servers

| Server | Purpose | Status |
|--------|---------|--------|
| `android-mcp-server` | Android developer documentation | Available |
| `kotlin-mcp-server` | Kotlin language documentation | Available |

**Gap Note**: There is no equivalent to MCP Cupertino (Apple docs MCP server) for Android. For official Android documentation, use web search as a fallback, or check the android-mcp-server for available documentation.

---

## Coverage Matrix

| Area | ARC Skills | External Skills | Gap |
|------|-----------|-----------------|-----|
| Architecture (Clean, MVVM) | Full | Full | None |
| UI (Jetpack Compose) | Full | Full | None |
| Data (Room, Retrofit, DataStore) | Full | Full | None |
| Dependency Injection (Hilt) | Full | Full | None |
| Testing (JUnit 5, MockK, Turbine) | Full | Partial | Advanced UI testing |
| CI/CD (GitHub Actions) | Full | Full | None |
| Performance Profiling | Partial | Full | Systrace, Macrobenchmark |
| Accessibility | Partial | Full | Automated accessibility scanning |
| Security | Minimal | Partial | ProGuard rules, network security |
| Gradle Build System | Partial | Full | Convention plugins, composite builds |
| Kotlin Multiplatform | None | Partial | Full KMP support |

---

## Common Scenarios

### 1. "I need to build a new feature"

Follow this sequence of skills:

```
/arc-android-architecture   --> Design the architecture and layer structure
/arc-data-layer             --> Implement data access (API, DB, cache)
/arc-presentation-layer     --> Implement UI (ViewModel, Screen, components)
/arc-tdd-patterns           --> Write tests for all layers
/arc-quality-standards      --> Review quality before PR
/arc-workflow               --> Create branch, commits, PR
```

### 2. "I need to fix a bug"

```
/arc-tdd-patterns           --> Write a failing test that reproduces the bug
/arc-data-layer or
  /arc-presentation-layer   --> Fix the code in the appropriate layer
/arc-quality-standards      --> Verify fix quality
/arc-workflow               --> Commit with conventional message (fix type)
```

### 3. "I need to review code"

```
/arc-quality-standards      --> Review checklist (lint, format, docs)
/arc-android-architecture   --> Verify architecture compliance
/arc-tdd-patterns           --> Verify test quality and coverage
/arc-final-review           --> Comprehensive final review
```

### 4. "I need to set up a new project"

```
/arc-project-setup          --> Create project structure and configuration
/arc-android-architecture   --> Set up architecture layers
/arc-workflow               --> Set up Git workflow (branches, hooks)
/arc-quality-standards      --> Configure linting and documentation
```

### 5. "I need to add a new library"

```
/arc-project-setup          --> Create library module structure
/arc-quality-standards      --> Set up documentation requirements
/arc-tdd-patterns           --> Set up 100% test coverage
/arc-workflow               --> Create branch and initial commits
```

### 6. "I need to refactor existing code"

```
/arc-tdd-patterns           --> Ensure tests exist before refactoring
/arc-android-architecture   --> Verify target architecture
/arc-quality-standards      --> Review refactored code quality
/arc-final-review           --> Comprehensive review before merge
```

---

## Priority Order (When Multiple Skills Apply)

When a task requires multiple skills, apply them in this order:

1. **Architecture first** - Get the structure right before writing code
2. **Implementation second** - Build on a solid architectural foundation
3. **Testing third** - Verify behavior with comprehensive tests
4. **Quality last** - Polish, document, and review

This order prevents rework. Architecture mistakes are the most expensive to fix, so get them right first.

---

## Skill Combination Patterns

### Full Feature Development
```
Architecture --> Data Layer --> Presentation Layer --> Testing --> Quality
```

### Bug Fix
```
Testing (reproduce) --> Implementation (fix) --> Quality (verify)
```

### Refactoring
```
Testing (ensure coverage) --> Architecture (design target) --> Implementation --> Quality
```

### Code Review
```
Quality --> Architecture --> Testing --> Final Review
```

---

## Creating Custom Skills

ARC Labs skills are stored as Markdown instruction files inside the `.claude/commands/` directory at the project or organization level.

### Directory Structure

```
.claude/
  commands/
    arc-android-architecture.md
    arc-presentation-layer.md
    arc-data-layer.md
    arc-tdd-patterns.md
    arc-quality-standards.md
    arc-workflow.md
    arc-project-setup.md
    arc-final-review.md
    arc-audit.md
```

### Skill File Template

```markdown
# Skill: /arc-<name>

## Description
Brief description of what this skill does.

## Preconditions
- [ ] List of conditions that must be true before running

## Steps
1. Step one with detailed instructions
2. Step two referencing ARC Labs conventions
3. Step three with validation checks

## Inputs
- `$ARGUMENTS` -- Description of expected arguments

## Outputs
- List of files created or modified
- Expected state after completion

## Example
Show a concrete example of running the skill and its output.
```

### Skill Resolution Order

When a skill is invoked, Claude Code resolves it in the following order:

1. **Project-level** -- `.claude/commands/arc-<name>.md` in the current repo
2. **Organization-level** -- Shared commands from ARCDevTools-Android
3. **Built-in** -- Default Claude Code slash commands

---

## Coverage Gaps

The following areas do not yet have dedicated skills:

| Gap | Description | Priority |
|-----|-------------|----------|
| `arc-migrate` | Migrate between Gradle versions or library major versions | High |
| `arc-benchmark` | Set up and run performance benchmarks (Macrobenchmark) | Medium |
| `arc-accessibility` | Audit Compose UI for accessibility compliance | Medium |
| `arc-security` | Scan for common security vulnerabilities | High |
| `arc-localization` | Manage string resources and translation workflows | Low |
| `arc-compose-preview` | Generate Compose Preview functions for all screens | Low |
| `arc-api-client` | Generate API client code from OpenAPI/Swagger specs | Medium |
| `arc-modularize` | Break a monolithic module into Clean Architecture layers | High |

If you identify a repeated workflow that is not covered by an existing skill, open a Linear issue with the label `skill-request`.

---

## Version Information

| Component | Version | Last Updated |
|-----------|---------|-------------|
| ARC Labs Skills | 1.0.0 | 2026-02-25 |
| ARCKnowledge-Android | 1.0.0 | 2026-02-25 |
| Skills Index | 1.0.0 | 2026-02-25 |

---

## Further Reading

- [Clean Architecture](../Architecture/clean-architecture.md)
- [Testing Standards](../Quality/testing.md)
- [README Standards](../Quality/readme-standards.md)
- [Git Commits](../Workflow/git-commits.md)
- [Git Branches](../Workflow/git-branches.md)
- [Plan Mode](../Workflow/plan-mode.md)

---

**Note**: Skills are living documents. When you find gaps or outdated information, flag it for updates. The goal is comprehensive, accurate coverage of all Android development patterns used at ARC Labs.
