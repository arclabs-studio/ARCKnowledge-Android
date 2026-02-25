# 📄 README Standards

**A good README is the front door to your project. It should answer: What is this? How do I use it? How do I contribute? A well-crafted README reduces onboarding time, prevents repetitive questions, and signals professional quality.**

---

## 🎯 README Philosophy

### First Impressions Matter
Your README is the single most-read document in any repository. When a developer lands on your project, they decide within 30 seconds whether to invest more time. A clear, well-structured README earns that investment. A messy or incomplete one drives people away, no matter how good the underlying code is.

### Progressive Disclosure
Structure information from high-level to detailed:

1. **What** — Title, badges, one-line description (instant understanding)
2. **Why** — Features, screenshots, use cases (motivation to try it)
3. **How** — Installation, quick start (getting started fast)
4. **Deep** — Configuration, API reference, architecture (advanced usage)
5. **Meta** — Contributing, license, changelog (project governance)

This pattern respects the reader's time. Most visitors need the first three levels. Only contributors and power users need levels four and five.

### Keep It Current
Outdated documentation is worse than no documentation. It actively misleads developers and erodes trust. Every pull request that changes public API, build configuration, or project structure should include a README update. Treat README changes as part of the definition of done.

### Write for the Newcomer
Assume the reader has never seen your project before. Avoid jargon specific to your codebase. Spell out acronyms on first use. If a step seems obvious to you, include it anyway — what is obvious to the author is rarely obvious to the reader.

---

## 📐 Library README Template

Use this template for all Android library packages published by ARC Labs Studio.

````markdown
# LibraryName

[![Build Status](https://github.com/ARC-Labs-Studio/LibraryName/actions/workflows/ci.yml/badge.svg)](https://github.com/ARC-Labs-Studio/LibraryName/actions/workflows/ci.yml)
[![Version](https://img.shields.io/maven-central/v/com.arclabs/libraryname)](https://central.sonatype.com/artifact/com.arclabs/libraryname)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Min SDK](https://img.shields.io/badge/minSdk-26-green.svg)](https://developer.android.com/about/versions/oreo)
[![Kotlin](https://img.shields.io/badge/Kotlin-2.0-purple.svg)](https://kotlinlang.org)

**Brief one-line description of what this library does and the problem it solves.**

LibraryName provides a longer paragraph explanation with more context. Describe the
core value proposition, the approach taken, and why someone would choose this over
alternatives. Keep it to 2-3 sentences maximum.

## Features

- **Feature One** — Short description of what it does
- **Feature Two** — Short description of what it does
- **Feature Three** — Short description of what it does
- **Feature Four** — Short description of what it does
- **Jetpack Compose** — Built with Compose-first APIs
- **Material Design 3** — Full MD3 theming support

## Screenshots

| Light Mode | Dark Mode |
|:---:|:---:|
| ![Light](docs/images/screenshot-light.png) | ![Dark](docs/images/screenshot-dark.png) |

> **Note:** Place screenshots in `docs/images/`. Use PNG format. Keep file sizes under 500 KB.
> For animated demos, use GIF or link to a short MP4 video.

## Requirements

| Requirement | Version |
|---|---|
| Android Min SDK | 26 (Android 8.0) |
| Android Target SDK | 35 |
| Kotlin | 2.0+ |
| Jetpack Compose BOM | 2024.12.01 |
| Android Gradle Plugin | 8.7+ |
| Java | 17 |

## Installation

### Gradle Version Catalog (Recommended)

Add to your `gradle/libs.versions.toml`:

```toml
[versions]
arclabs-libraryname = "1.0.0"

[libraries]
arclabs-libraryname = { group = "com.arclabs", name = "libraryname", version.ref = "arclabs-libraryname" }
```

Then in your module's `build.gradle.kts`:

```kotlin
dependencies {
    implementation(libs.arclabs.libraryname)
}
```

### Direct Dependency

If you are not using a version catalog, add directly to `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.arclabs:libraryname:1.0.0")
}
```

### Snapshots

For unreleased development builds, add the snapshots repository:

```kotlin
repositories {
    maven { url = uri("https://s01.oss.sonatype.org/content/repositories/snapshots/") }
}

dependencies {
    implementation("com.arclabs:libraryname:1.1.0-SNAPSHOT")
}
```

## Quick Start

A complete, minimal working example that compiles and runs:

```kotlin
import com.arclabs.libraryname.FeatureComponent
import com.arclabs.libraryname.LibraryNameConfig

// 1. Configure the library (optional — sensible defaults are provided)
val config = LibraryNameConfig.Builder()
    .setOption("value")
    .setDebugLogging(BuildConfig.DEBUG)
    .build()

// 2. Initialize in your Application or Hilt module
LibraryName.initialize(context, config)

// 3. Use in a Composable
@Composable
fun MyScreen() {
    FeatureComponent(
        data = sampleData,
        onAction = { action ->
            // Handle action
        },
        modifier = Modifier.fillMaxWidth()
    )
}
```

## API Reference

### Core Classes

| Class | Description |
|---|---|
| `LibraryName` | Main entry point and initialization |
| `FeatureComponent` | Primary Composable UI component |
| `LibraryNameConfig` | Configuration builder |

### Configuration Options

| Option | Type | Default | Description |
|---|---|---|---|
| `debugLogging` | `Boolean` | `false` | Enable debug log output |
| `cacheEnabled` | `Boolean` | `true` | Enable in-memory caching |
| `timeout` | `Duration` | `30.seconds` | Network request timeout |

For full API documentation, see the [generated KDoc](docs/api/).

## Configuration

### Basic Setup

```kotlin
val config = LibraryNameConfig.Builder()
    .setDebugLogging(true)
    .build()
```

### Advanced Setup with Hilt

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object LibraryNameModule {

    @Provides
    @Singleton
    fun provideLibraryNameConfig(): LibraryNameConfig {
        return LibraryNameConfig.Builder()
            .setOption("value")
            .build()
    }

    @Provides
    @Singleton
    fun provideLibraryName(
        @ApplicationContext context: Context,
        config: LibraryNameConfig
    ): LibraryName {
        return LibraryName.create(context, config)
    }
}
```

## Architecture

This library follows **Clean Architecture** principles:

```
libraryname/
├── api/              # Public API surface
├── domain/
│   ├── model/        # Domain entities
│   └── usecase/      # Business logic
├── data/
│   ├── repository/   # Repository implementations
│   └── source/       # Local and remote data sources
└── ui/
    ├── component/    # Composable components
    └── theme/        # Library theming
```

## Contributing

We welcome contributions. Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/ISSUE-ID-description`)
3. Write tests for your changes
4. Ensure all tests pass (`./gradlew test`)
5. Ensure lint passes (`./gradlew detekt`)
6. Commit using [Conventional Commits](https://www.conventionalcommits.org/)
7. Open a Pull Request

## License

MIT License - see [LICENSE](LICENSE) for details.

```
Copyright (c) 2025 ARC Labs Studio
```
````

---

## 📐 App README Template

Use this template for all Android application projects at ARC Labs Studio.

````markdown
# AppName

Brief description of the app, its purpose, and target users.

AppName is an Android application that does X for Y users. Built with Jetpack Compose
and Clean Architecture, it provides Z capability.

## Architecture

The project follows **Clean Architecture + MVVM** with unidirectional data flow:

```
┌──────────────────────────────────────┐
│           Presentation               │
│   (Compose UI + ViewModels)          │
├──────────────────────────────────────┤
│             Domain                   │
│   (Use Cases + Entities + Repos)     │
├──────────────────────────────────────┤
│              Data                    │
│   (Repositories + DataSources)       │
└──────────────────────────────────────┘
```

- **Presentation:** Jetpack Compose views, ViewModels with `@HiltViewModel`
- **Domain:** Pure Kotlin use cases and entity models, repository interfaces
- **Data:** Repository implementations, Retrofit services, Room DAOs, DataStore

## Tech Stack

| Category | Technology |
|---|---|
| Language | Kotlin 2.0 |
| UI Framework | Jetpack Compose + Material Design 3 |
| Dependency Injection | Hilt |
| Networking | Retrofit + OkHttp + Kotlinx Serialization |
| Local Storage | Room + DataStore |
| Navigation | Compose Navigation (type-safe) |
| Async | Kotlin Coroutines + Flow |
| Image Loading | Coil |
| Testing | JUnit 5, Turbine, Mockk |
| CI/CD | GitHub Actions |
| Code Quality | Detekt, ktlint |

## Setup

### Prerequisites

- [Android Studio](https://developer.android.com/studio) Ladybug (2024.2.1) or later
- JDK 17
- Android SDK 35
- Android Emulator or physical device (API 26+)

### Steps

1. **Clone the repository:**
   ```bash
   git clone https://github.com/ARC-Labs-Studio/AppName.git
   cd AppName
   ```

2. **Create local configuration:**
   ```bash
   cp local.properties.example local.properties
   ```

3. **Add required API keys** to `local.properties`:
   ```properties
   MAPS_API_KEY=your_google_maps_key
   API_BASE_URL=https://api.example.com
   ```

4. **Open in Android Studio** and let Gradle sync complete.

5. **Run the app:**
   ```bash
   ./gradlew installDebug
   ```
   Or press Run in Android Studio.

## Project Structure

```
app/
├── src/main/
│   ├── kotlin/com/arclabs/appname/
│   │   ├── AppNameApplication.kt
│   │   ├── MainActivity.kt
│   │   ├── navigation/
│   │   │   └── AppNavGraph.kt
│   │   ├── di/
│   │   │   └── AppModule.kt
│   │   └── feature/
│   │       ├── home/
│   │       │   ├── HomeScreen.kt
│   │       │   └── HomeViewModel.kt
│   │       └── detail/
│   │           ├── DetailScreen.kt
│   │           └── DetailViewModel.kt
│   └── res/
│       └── ...
├── core/
│   ├── domain/
│   │   ├── model/
│   │   ├── repository/
│   │   └── usecase/
│   ├── data/
│   │   ├── repository/
│   │   ├── remote/
│   │   ├── local/
│   │   └── dto/
│   └── ui/
│       ├── component/
│       └── theme/
└── build.gradle.kts
```

## Build Commands

```bash
# Debug build
./gradlew assembleDebug

# Release build
./gradlew assembleRelease

# Run unit tests
./gradlew test

# Run instrumented tests
./gradlew connectedAndroidTest

# Run lint checks
./gradlew detekt

# Format code
./gradlew ktlintFormat

# Generate test coverage report
./gradlew koverHtmlReport
```

## Environment Variables

Configuration is managed through `local.properties` (git-ignored) and `BuildConfig`:

| Variable | Required | Description |
|---|---|---|
| `MAPS_API_KEY` | Yes | Google Maps API key |
| `API_BASE_URL` | Yes | Backend API base URL |
| `ANALYTICS_KEY` | No | Analytics tracking key |

A `local.properties.example` file is provided as a template. Copy it and fill in your values.

## Contributing

1. Pick an issue from the [project board](link)
2. Create a branch: `feature/ISSUE-ID-description`
3. Follow the [code style guide](link)
4. Write tests (minimum 80% coverage)
5. Submit a PR with the provided template

## License

Private - ARC Labs Studio. All rights reserved.
````

---

## 🏷️ Badge Examples

Badges provide at-a-glance project status. Place them immediately below the title.

### Standard Badge Set for Android Libraries

```markdown
[![Build Status](https://github.com/ARC-Labs-Studio/LibraryName/actions/workflows/ci.yml/badge.svg)](https://github.com/ARC-Labs-Studio/LibraryName/actions/workflows/ci.yml)
[![Version](https://img.shields.io/maven-central/v/com.arclabs/libraryname)](https://central.sonatype.com/artifact/com.arclabs/libraryname)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Min SDK](https://img.shields.io/badge/minSdk-26-green.svg)](https://developer.android.com/about/versions/oreo)
[![Kotlin](https://img.shields.io/badge/Kotlin-2.0-purple.svg)](https://kotlinlang.org)
```

### Badge Descriptions

| Badge | Purpose | When to Include |
|---|---|---|
| Build Status | Shows CI pipeline health | Always for libraries |
| Version | Current published version | Always for libraries |
| License | Project license at a glance | Always |
| Min SDK | Minimum Android SDK level | Always for Android projects |
| Kotlin | Kotlin version used | Recommended |
| Coverage | Test coverage percentage | Optional, when tracking coverage |
| Compose BOM | Compose BOM version | Optional |

### Custom Badges

Use [shields.io](https://shields.io) for custom badges:

```markdown
[![Min SDK](https://img.shields.io/badge/minSdk-26-green.svg)](link)
[![Compose BOM](https://img.shields.io/badge/Compose%20BOM-2024.12.01-blue.svg)](link)
[![Coverage](https://img.shields.io/badge/coverage-95%25-brightgreen.svg)](link)
```

---

## 📋 Required vs Optional Sections

| Section | Libraries | Apps | Notes |
|---|---|---|---|
| Title | ✅ Required | ✅ Required | Must match the project/module name |
| Badges | ✅ Required | Optional | Build, version, license at minimum |
| One-line Description | ✅ Required | ✅ Required | What this project does in one sentence |
| Extended Description | ✅ Required | ✅ Required | 2-3 sentence expanded explanation |
| Features | ✅ Required | ✅ Required | Bulleted list, bolded names |
| Screenshots | Recommended | ✅ Required | Light and dark mode when applicable |
| Requirements Table | ✅ Required | ✅ Required | SDK, Kotlin, Compose versions |
| Installation | ✅ Required | ❌ N/A | Version catalog + direct dependency |
| Quick Start | ✅ Required | ❌ N/A | Complete compilable code example |
| Setup Instructions | ❌ N/A | ✅ Required | Step-by-step with prerequisites |
| API Reference | Recommended | ❌ N/A | Table of core classes + link to KDoc |
| Configuration | Recommended | Optional | When the project has configurable options |
| Architecture | Optional | ✅ Required | Diagram + brief description |
| Tech Stack Table | Optional | ✅ Required | Category + technology pairs |
| Project Structure | Optional | ✅ Required | Directory tree with descriptions |
| Build Commands | Optional | ✅ Required | Common Gradle tasks |
| Environment Variables | ❌ N/A | ✅ Required | Table with required/optional flags |
| Contributing | ✅ Required | ✅ Required | Link to CONTRIBUTING.md or inline |
| License | ✅ Required | ✅ Required | MIT for libraries, Private for apps |
| Changelog | Recommended | Optional | Link to CHANGELOG.md |

---

## ✍️ Writing Style Guide

### Tone
- Professional but approachable
- Direct and concise — do not pad with filler words
- Action-oriented — tell the reader what to do, not what they could do

### Tense and Voice
- Use **present tense**: "This library provides..." not "This library will provide..."
- Use **active voice**: "Run the command" not "The command should be run"
- Use **second person** for instructions: "Add the dependency to your build file"

### Formatting Rules

| Rule | ✅ Correct | ❌ Incorrect |
|---|---|---|
| Code references | Use backticks: `ClassName` | Plain text: ClassName |
| File paths | Use backticks: `build.gradle.kts` | Plain text: build.gradle.kts |
| Commands | Code block with language hint | Inline or no language hint |
| Emphasis | **Bold** for UI elements, key terms | ALL CAPS or _italics_ for emphasis |
| Lists | Consistent punctuation and structure | Mixed styles in the same list |
| Headings | Sentence case: "Quick start" | Title Case: "Quick Start" (exception: proper nouns) |

### Content Rules
- One idea per paragraph
- Maximum 3-4 sentences per paragraph
- Prefer bullet lists over long paragraphs for feature descriptions
- Every code example must compile — verify before committing
- Avoid relative time references ("recently", "soon") — use version numbers or dates
- Spell out acronyms on first use: "Dependency Injection (DI)"

---

## 🖼️ Screenshots Section

### Guidelines
- Place images in `docs/images/` within the repository
- Use PNG for static screenshots, GIF for short animations (under 10 seconds)
- For longer demos, record a video, upload it, and link to it
- Keep image file sizes under 500 KB — compress with a tool like `pngquant` or `tinypng`
- Always show both light and dark mode when the app supports theming
- Capture at a standard resolution (1080x1920 for phone, 1200x800 for tablet)

### Table Layout for Screenshots

```markdown
## Screenshots

| Light Mode | Dark Mode |
|:---:|:---:|
| ![Light](docs/images/screenshot-light.png) | ![Dark](docs/images/screenshot-dark.png) |
```

### GIF Demo

```markdown
## Demo

![Demo](docs/images/demo.gif)

> Showing feature X in action: tap the button, view the result, swipe to dismiss.
```

### Keeping Screenshots Current
- Update screenshots whenever the UI changes
- Add screenshot updates to your PR checklist
- Name files descriptively: `home-screen-light.png`, not `screenshot1.png`
- Include a short caption or alt text for accessibility

---

## 🔢 Versioning in READMEs

### Referencing Versions
Always show the specific version number in installation instructions. Never use placeholder text like "latest" without also showing the actual version number.

✅ **Correct:**
```markdown
implementation("com.arclabs:libraryname:2.1.0")
```

❌ **Incorrect:**
```markdown
implementation("com.arclabs:libraryname:latest")
```

### Version Catalog References
When using a version catalog, define the version in the `[versions]` section so it is easy to update in one place:

```toml
[versions]
arclabs-libraryname = "2.1.0"

[libraries]
arclabs-libraryname = { group = "com.arclabs", name = "libraryname", version.ref = "arclabs-libraryname" }
```

### Changelog References
Link to a changelog so readers can understand what changed between versions:

```markdown
See [CHANGELOG.md](CHANGELOG.md) for a detailed history of changes.
```

### Version Update Workflow
When releasing a new version:

1. Update the version number in the README installation section
2. Update any version-specific badge URLs
3. Add the release notes to `CHANGELOG.md`
4. Commit the README and changelog changes as part of the release PR

---

## ✅ README Checklist

Use this checklist before merging any PR that introduces or modifies a README.

### Structure
- [ ] Title matches the project or module name exactly
- [ ] One-line description is present and accurate
- [ ] Extended description provides enough context for a newcomer
- [ ] Sections follow the standard order from the template
- [ ] Horizontal rules separate major sections cleanly

### Content
- [ ] Features list is current and complete
- [ ] Requirements table lists correct version numbers
- [ ] Installation instructions are copy-pasteable and use current version
- [ ] Quick start example compiles without modification
- [ ] API reference covers all public classes
- [ ] Configuration options are documented with defaults

### Quality
- [ ] All links are valid (no 404s)
- [ ] All badges resolve and show correct status
- [ ] Screenshots are current and display correctly
- [ ] Code blocks have language hints (`kotlin`, `toml`, `bash`)
- [ ] No spelling or grammar errors
- [ ] No relative time references ("recently", "soon")

### Compliance
- [ ] License section is present and correct
- [ ] Contributing section links to CONTRIBUTING.md or provides inline instructions
- [ ] Version numbers in installation instructions match the latest release
- [ ] Sensitive information (API keys, tokens) is not included

---

## ❌ Common Mistakes

### 1. Outdated Installation Instructions

The version number in the README does not match the latest published release.

❌ **Incorrect:**
```markdown
## Installation
Add to your build.gradle.kts:
implementation("com.arclabs:libraryname:0.9.0")  // Published version is 2.1.0
```

✅ **Correct:**
```markdown
## Installation
Add to your build.gradle.kts:
implementation("com.arclabs:libraryname:2.1.0")  // Matches latest release
```

**Fix:** Include README version updates in your release checklist.

### 2. Missing Requirements Section

The reader has no idea what SDK, Kotlin, or Compose version they need.

❌ **Incorrect:**
```markdown
# MyLibrary
A great library.

## Installation
...
```

✅ **Correct:**
```markdown
# MyLibrary
A great library.

## Requirements
| Requirement | Version |
|---|---|
| Android Min SDK | 26 |
| Kotlin | 2.0+ |
| Compose BOM | 2024.12.01 |

## Installation
...
```

**Fix:** Always include a requirements table before installation instructions.

### 3. Broken or Missing Links

Links to documentation, contributing guides, or external resources return 404.

❌ **Incorrect:**
```markdown
See [Architecture Guide](docs/architecture.md) for details.
<!-- File does not exist -->
```

✅ **Correct:**
```markdown
See [Architecture Guide](docs/architecture.md) for details.
<!-- File exists and is up to date -->
```

**Fix:** Verify every link before merging. Use a link checker tool or manually click through each one.

### 4. No Quick Start Example

The reader can install the library but has no idea how to use it. They have to dig through source code to figure out the first call.

❌ **Incorrect:**
```markdown
## Installation
Add the dependency and you're good to go!
```

✅ **Correct:**
```markdown
## Quick Start

```kotlin
// 1. Initialize
val client = LibraryName.create(context)

// 2. Use the main feature
val result = client.doSomething(input)

// 3. Display in Compose
@Composable
fun MyScreen() {
    FeatureComponent(data = result)
}
```
```

**Fix:** Every library README must include a complete, compilable quick start example that covers initialization and basic usage.

### 5. Overly Verbose Description

Walls of text in the description that bury the key information.

❌ **Incorrect:**
```markdown
# MyLibrary

MyLibrary is a library that was created by ARC Labs Studio in 2024 as part of our
ongoing effort to build high-quality Android components for the modern developer
ecosystem. It leverages the latest features of Kotlin 2.0 and Jetpack Compose to
provide a seamless development experience. The library was inspired by several open
source projects and aims to fill a gap in the ecosystem by providing a solution that
is both powerful and easy to use. We believe that developers deserve better tools and
this library is our contribution to making Android development more enjoyable...
```

✅ **Correct:**
```markdown
# MyLibrary

**Type-safe navigation components for Jetpack Compose with built-in deep link support.**

MyLibrary simplifies Compose navigation by providing type-safe route definitions,
automatic argument parsing, and deep link handling out of the box.
```

**Fix:** Lead with a bold one-liner. Follow with 2-3 sentences maximum. Save the story for a blog post.

### 6. Placeholder Content Left In

Template sections that were never filled in ship to production.

❌ **Incorrect:**
```markdown
## Features

- Feature 1
- Feature 2
- TODO: Add more features
```

✅ **Correct:**
```markdown
## Features

- **Type-safe routes** — Define navigation routes as Kotlin data classes
- **Deep link support** — Automatic deep link registration and handling
- **Compose integration** — Seamless integration with NavHost and NavController
```

**Fix:** Search for "TODO", "TBD", "Feature 1", and other placeholder text before merging.

### 7. Missing Environment Setup for Apps

App README assumes the reader has all API keys and configuration already set up.

❌ **Incorrect:**
```markdown
## Setup
1. Clone the repo
2. Open in Android Studio
3. Run the app
```

✅ **Correct:**
```markdown
## Setup
1. Clone the repo
2. Copy `local.properties.example` to `local.properties`
3. Add your API keys (see Environment Variables section)
4. Open in Android Studio
5. Sync Gradle and run on an emulator or device
```

**Fix:** Document every prerequisite. Provide a `local.properties.example` template. List all required environment variables in a table.

---

## 📚 Further Reading

- [Documentation Guide](./documentation.md)
- [Module Structure](./module-structure.md)
- [Code Review Standards](./code-review.md)
- [Testing Standards](./testing.md)

---

**Remember**: Your README is often the first and only documentation people read. Make it count. 📄
