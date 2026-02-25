# 🛠️ Android Studio

**The official Integrated Development Environment (IDE) for Android app development, built on JetBrains IntelliJ IDEA. This guide covers project configuration, build settings, debugging, profiling, testing, and best practices for Android Studio in the context of modern Android development with Kotlin and Jetpack Compose.**

---

## 🎯 Overview

Android Studio is the official IDE for Android development, provided and maintained by Google. It is built on top of JetBrains IntelliJ IDEA and offers a comprehensive set of tools specifically designed for building Android applications.

### Why Android Studio?

- **Official IDE:** Google's recommended and supported environment for Android development
- **IntelliJ IDEA Foundation:** Inherits a powerful code editor, refactoring tools, and plugin ecosystem
- **Compose Tooling:** Built-in support for Jetpack Compose previews, Layout Inspector, and recomposition debugging
- **Gradle Integration:** Deep integration with the Gradle build system for building, testing, and deploying
- **Emulator:** Bundled Android Emulator with hardware acceleration for fast device simulation
- **Profiling Tools:** CPU, Memory, Network, and Energy profilers built directly into the IDE
- **Version Control:** Native Git integration with visual diff, merge conflict resolution, and branch management

### Version Recommendations

| Component | Recommended Version |
|-----------|-------------------|
| Android Studio | Latest Stable (Meerkat 2024.3.1+) |
| Gradle | 8.9+ |
| Android Gradle Plugin (AGP) | 8.8+ |
| JDK | 17 (bundled with Android Studio) |
| Kotlin | 2.1+ |
| Compose Compiler | Kotlin compiler plugin (integrated since Kotlin 2.0) |

### Installation

✅ **DO:**
- Download from the official site: https://developer.android.com/studio
- Use the bundled JDK (JBR 17) for Gradle builds
- Keep Android Studio updated to the latest stable release
- Install required SDK platforms and build tools via SDK Manager

❌ **DON'T:**
- Use beta or canary channels for production projects
- Mix JDK versions between Android Studio and Gradle
- Skip SDK license agreements (causes build failures in CI)

---

## 📂 Project Structure

### Android View vs. Project View

Android Studio provides two primary ways to view your project files:

**Android View (Default):**
- Groups files by module and type
- Simplifies navigation for Android-specific resources
- Combines build variants and source sets visually
- Hides Gradle and build files for a cleaner experience

**Project View:**
- Shows the actual file system structure on disk
- Useful for understanding the true directory layout
- Required when working with multi-module projects or custom configurations
- Shows all files including hidden ones

✅ **DO:**
- Use Android View for day-to-day development within a module
- Switch to Project View when configuring Gradle, multi-module dependencies, or CI

❌ **DON'T:**
- Rely solely on Android View for understanding the project layout
- Manually move files in the file system without understanding module boundaries

### Standard Directory Layout

```
project-root/
├── app/                              # Main application module
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/                 # Kotlin/Java source files
│   │   │   │   └── com/
│   │   │   │       └── example/
│   │   │   │           └── myapp/
│   │   │   │               ├── MainActivity.kt
│   │   │   │               ├── MyApplication.kt
│   │   │   │               ├── di/              # Dependency injection
│   │   │   │               ├── ui/              # Presentation layer
│   │   │   │               │   ├── theme/
│   │   │   │               │   ├── navigation/
│   │   │   │               │   └── screens/
│   │   │   │               ├── domain/          # Domain layer
│   │   │   │               │   ├── model/
│   │   │   │               │   ├── repository/
│   │   │   │               │   └── usecase/
│   │   │   │               └── data/            # Data layer
│   │   │   │                   ├── repository/
│   │   │   │                   ├── remote/
│   │   │   │                   └── local/
│   │   │   ├── res/
│   │   │   │   ├── drawable/
│   │   │   │   ├── mipmap-hdpi/
│   │   │   │   ├── mipmap-mdpi/
│   │   │   │   ├── mipmap-xhdpi/
│   │   │   │   ├── mipmap-xxhdpi/
│   │   │   │   ├── mipmap-xxxhdpi/
│   │   │   │   ├── values/
│   │   │   │   │   ├── strings.xml
│   │   │   │   │   ├── colors.xml
│   │   │   │   │   └── themes.xml
│   │   │   │   └── values-es/        # Spanish localization
│   │   │   │       └── strings.xml
│   │   │   └── AndroidManifest.xml
│   │   ├── debug/                     # Debug-only sources/resources
│   │   ├── release/                   # Release-only sources/resources
│   │   └── test/                      # Unit tests
│   │       └── java/
│   │           └── com/example/myapp/
│   └── build.gradle.kts
├── core/                              # Core library module
│   ├── src/
│   │   ├── main/
│   │   └── test/
│   └── build.gradle.kts
├── feature-home/                      # Feature module
│   ├── src/
│   │   ├── main/
│   │   └── test/
│   └── build.gradle.kts
├── build.gradle.kts                   # Root build file
├── settings.gradle.kts                # Module declarations
├── gradle.properties                  # Gradle properties
├── gradle/
│   ├── wrapper/
│   │   ├── gradle-wrapper.jar
│   │   └── gradle-wrapper.properties
│   └── libs.versions.toml             # Version catalog
└── local.properties                   # Local SDK path (DO NOT commit)
```

### Gradle Module Structure

In a multi-module project, each module has its own `build.gradle.kts` file and source sets.

**settings.gradle.kts:**

```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "MyApp"

include(":app")
include(":core")
include(":feature-home")
include(":feature-profile")
```

✅ **DO:**
- Use `settings.gradle.kts` (Kotlin DSL) over `settings.gradle` (Groovy)
- Use a version catalog (`libs.versions.toml`) for dependency management
- Organize code into feature modules for large projects
- Keep `local.properties` in `.gitignore`

❌ **DON'T:**
- Commit `local.properties` to version control
- Define dependency versions inline in multiple `build.gradle.kts` files
- Create circular module dependencies

---

## ⚙️ Build Settings

### SDK Version Recommendations

| Setting | Value | Notes |
|---------|-------|-------|
| `compileSdk` | 35 | Android 15 (VanillaIceCream) |
| `minSdk` | 26 | Android 8.0 - covers 97%+ of active devices |
| `targetSdk` | 35 | Must match compileSdk for Play Store compliance |

### Kotlin Version

```kotlin
// gradle/libs.versions.toml
[versions]
kotlin = "2.1.0"
agp = "8.8.0"
composeBom = "2024.12.01"

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
```

### Compose Compiler Configuration

Since Kotlin 2.0, the Compose compiler is integrated as a Kotlin compiler plugin. No separate Compose compiler version is needed.

```kotlin
// app/build.gradle.kts
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
}

android {
    namespace = "com.example.myapp"
    compileSdk = 35

    defaultConfig {
        applicationId = "com.example.myapp"
        minSdk = 26
        targetSdk = 35
        versionCode = 1
        versionName = "1.0.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
    }

    buildFeatures {
        compose = true
        buildConfig = true
    }
}

dependencies {
    implementation(platform(libs.compose.bom))
    implementation(libs.compose.ui)
    implementation(libs.compose.material3)
    implementation(libs.compose.ui.tooling.preview)
    debugImplementation(libs.compose.ui.tooling)
    debugImplementation(libs.compose.ui.test.manifest)
}
```

### Compose Compiler Reports (Optional)

To enable Compose compiler stability reports for performance debugging:

```kotlin
// app/build.gradle.kts
composeCompiler {
    reportsDestination = layout.buildDirectory.dir("compose_compiler")
    stabilityConfigurationFile = rootProject.layout.projectDirectory.file("stability_config.conf")
}
```

✅ **DO:**
- Use version catalog (`libs.versions.toml`) for all dependency versions
- Set `jvmTarget = "17"` to match the bundled JDK
- Enable `buildConfig = true` only if you need `BuildConfig` fields
- Use the Compose BOM for consistent Compose library versions

❌ **DON'T:**
- Set `minSdk` below 26 unless there is a clear business requirement
- Use a `jvmTarget` different from the project's JDK version
- Mix Compose BOM versions with manually specified Compose library versions

---

## 🏗️ Build Configurations

### Debug, Release, and Staging Build Types

```kotlin
// app/build.gradle.kts
android {
    buildTypes {
        debug {
            isDebuggable = true
            isMinifyEnabled = false
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-debug"

            buildConfigField("String", "BASE_URL", "\"https://api-dev.example.com\"")
            buildConfigField("Boolean", "ENABLE_LOGGING", "true")
        }

        release {
            isDebuggable = false
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )

            buildConfigField("String", "BASE_URL", "\"https://api.example.com\"")
            buildConfigField("Boolean", "ENABLE_LOGGING", "false")
        }

        create("staging") {
            initWith(getByName("debug"))
            applicationIdSuffix = ".staging"
            versionNameSuffix = "-staging"

            buildConfigField("String", "BASE_URL", "\"https://api-staging.example.com\"")
            buildConfigField("Boolean", "ENABLE_LOGGING", "true")
        }
    }
}
```

### BuildConfig Fields

Access build configuration values in Kotlin code:

```kotlin
// Automatically generated by the build system
val baseUrl = BuildConfig.BASE_URL
val isLoggingEnabled = BuildConfig.ENABLE_LOGGING
val isDebug = BuildConfig.DEBUG
val appVersion = BuildConfig.VERSION_NAME
val buildNumber = BuildConfig.VERSION_CODE
```

### Signing Configurations

```kotlin
// app/build.gradle.kts
android {
    signingConfigs {
        create("release") {
            storeFile = file(System.getenv("KEYSTORE_PATH") ?: "keystore/release.jks")
            storePassword = System.getenv("KEYSTORE_PASSWORD") ?: ""
            keyAlias = System.getenv("KEY_ALIAS") ?: ""
            keyPassword = System.getenv("KEY_PASSWORD") ?: ""
        }
    }

    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
            // ... other release config
        }
    }
}
```

✅ **DO:**
- Store signing credentials in environment variables or a secrets manager
- Use `isMinifyEnabled = true` and `isShrinkResources = true` for release builds
- Use `applicationIdSuffix` so debug/staging can be installed alongside release
- Use ProGuard/R8 rules to keep necessary classes from being stripped

❌ **DON'T:**
- Hardcode keystore passwords in `build.gradle.kts`
- Commit keystore files to version control
- Disable minification in release builds
- Skip ProGuard testing before release

### Product Flavors

Use product flavors when you need to build different variants of your app (e.g., free vs. paid, different backend environments):

```kotlin
// app/build.gradle.kts
android {
    flavorDimensions += "tier"

    productFlavors {
        create("free") {
            dimension = "tier"
            applicationIdSuffix = ".free"
            buildConfigField("Boolean", "IS_PREMIUM", "false")
        }

        create("premium") {
            dimension = "tier"
            applicationIdSuffix = ".premium"
            buildConfigField("Boolean", "IS_PREMIUM", "true")
        }
    }
}
```

---

## 📋 Manifest Configuration (AndroidManifest.xml)

### Application Attributes

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Permissions -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <application
        android:name=".MyApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyApp">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:theme="@style/Theme.MyApp">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

### Activity Configuration for Compose

For a single-activity Compose application, the `MainActivity` is typically minimal:

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            MyAppTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    AppNavigation()
                }
            }
        }
    }
}
```

### Custom Application Class

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        // Initialize dependency injection (Hilt, Koin, etc.)
        // Initialize crash reporting
        // Initialize analytics
        if (BuildConfig.ENABLE_LOGGING) {
            Timber.plant(Timber.DebugTree())
        }
    }
}
```

### Required Permissions by Feature

| Feature | Permission |
|---------|-----------|
| Network calls | `android.permission.INTERNET` |
| Network state | `android.permission.ACCESS_NETWORK_STATE` |
| Camera | `android.permission.CAMERA` |
| Fine location | `android.permission.ACCESS_FINE_LOCATION` |
| Coarse location | `android.permission.ACCESS_COARSE_LOCATION` |
| Read storage (API < 33) | `android.permission.READ_EXTERNAL_STORAGE` |
| Read media images (API 33+) | `android.permission.READ_MEDIA_IMAGES` |
| Notifications (API 33+) | `android.permission.POST_NOTIFICATIONS` |

✅ **DO:**
- Only declare permissions that your app actually uses
- Use runtime permission requests for dangerous permissions
- Add `android:exported="true"` for the launcher activity
- Use `enableEdgeToEdge()` for modern edge-to-edge display

❌ **DON'T:**
- Request unnecessary permissions (impacts Play Store review)
- Forget to handle permission denial gracefully
- Use `android:exported` without a clear intent (required for API 31+)
- Omit `android:name` on the `<application>` tag if you have a custom Application class

---

## 🎨 Resources

### Drawable Resources

Vector drawables are preferred over raster images for scalability and smaller APK size:

```xml
<!-- res/drawable/ic_search.xml -->
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#FF000000"
        android:pathData="M15.5,14h-0.79l-0.28,-0.27C15.41,12.59 16,11.11 16,9.5 16,5.91 13.09,3 9.5,3S3,5.91 3,9.5 5.91,16 9.5,16c1.61,0 3.09,-0.59 4.23,-1.57l0.27,0.28v0.79l5,4.99L20.49,19l-4.99,-5z"/>
</vector>
```

### Values Resources

**strings.xml:**

```xml
<!-- res/values/strings.xml -->
<resources>
    <string name="app_name">My App</string>
    <string name="home_search_hint">Search restaurants...</string>
    <string name="profile_edit_button_label">Edit Profile</string>
    <string name="error_network_message">No internet connection. Please try again.</string>
    <string name="common_cancel_button">Cancel</string>
    <string name="common_ok_button">OK</string>
    <string name="settings_notification_title">Notifications</string>
    <string name="home_welcome_message">Welcome, %1$s!</string>
    <plurals name="home_item_count">
        <item quantity="one">%d item</item>
        <item quantity="other">%d items</item>
    </plurals>
</resources>
```

**colors.xml:**

```xml
<!-- res/values/colors.xml -->
<resources>
    <color name="primary">#FF6200EE</color>
    <color name="primary_variant">#FF3700B3</color>
    <color name="secondary">#FF03DAC5</color>
    <color name="background">#FFFFFFFF</color>
    <color name="surface">#FFFFFFFF</color>
    <color name="error">#FFB00020</color>
    <color name="on_primary">#FFFFFFFF</color>
    <color name="on_secondary">#FF000000</color>
    <color name="on_background">#FF000000</color>
    <color name="on_surface">#FF000000</color>
    <color name="on_error">#FFFFFFFF</color>
</resources>
```

**themes.xml:**

```xml
<!-- res/values/themes.xml -->
<resources>
    <style name="Theme.MyApp" parent="android:Theme.Material.Light.NoActionBar">
        <!-- Use Material 3 theming via Compose; this is a fallback for system UI -->
    </style>
</resources>
```

### Mipmap Resources (App Icons)

Provide adaptive icons for Android 8.0+:

```
res/
├── mipmap-hdpi/
│   └── ic_launcher.webp
├── mipmap-mdpi/
│   └── ic_launcher.webp
├── mipmap-xhdpi/
│   └── ic_launcher.webp
├── mipmap-xxhdpi/
│   └── ic_launcher.webp
├── mipmap-xxxhdpi/
│   └── ic_launcher.webp
└── mipmap-anydpi-v26/
    ├── ic_launcher.xml          <!-- Adaptive icon -->
    └── ic_launcher_round.xml
```

**Adaptive icon configuration:**

```xml
<!-- res/mipmap-anydpi-v26/ic_launcher.xml -->
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background"/>
    <foreground android:drawable="@drawable/ic_launcher_foreground"/>
    <monochrome android:drawable="@drawable/ic_launcher_monochrome"/>
</adaptive-icon>
```

### Naming Conventions

All Android resources use **snake_case**:

| Resource Type | Convention | Example |
|--------------|-----------|---------|
| Layouts | `{type}_{name}` | `activity_main.xml` |
| Drawables | `ic_{name}` or `bg_{name}` | `ic_search.xml`, `bg_card.xml` |
| Strings | `{module}_{screen}_{element}_{type}` | `home_search_hint` |
| Colors | `{semantic_name}` | `primary`, `on_surface` |
| Dimensions | `{type}_{size}` | `margin_medium`, `text_body` |
| IDs | `{type}_{name}` | `btn_submit`, `tv_title` |

### String Resource Key Convention

Format: `{module}_{screen}_{element}_{type}`

Examples:

```
home_search_hint              # Home screen, search field, hint text
profile_edit_button_label     # Profile screen, edit button, label
error_network_message         # Error category, network error, message
common_cancel_button          # Common/shared, cancel button
settings_notification_title   # Settings screen, notification section, title
auth_login_email_label        # Auth module, login screen, email field, label
auth_login_password_hint      # Auth module, login screen, password field, hint
onboarding_step1_title        # Onboarding flow, step 1, title
```

### Localization Setup

1. Create locale-specific `values` folders:

```
res/
├── values/              # Default (English)
│   └── strings.xml
├── values-es/           # Spanish
│   └── strings.xml
├── values-fr/           # French
│   └── strings.xml
└── values-pt-rBR/       # Brazilian Portuguese
    └── strings.xml
```

2. Configure supported locales in `build.gradle.kts`:

```kotlin
android {
    defaultConfig {
        resourceConfigurations += listOf("en", "es", "fr", "pt-rBR")
    }
}
```

✅ **DO:**
- Use vector drawables over PNGs wherever possible
- Extract all user-facing strings to `strings.xml`
- Use string format parameters (`%1$s`, `%d`) for dynamic content
- Provide adaptive icons with monochrome layer for Android 13+

❌ **DON'T:**
- Hardcode strings in Kotlin or XML layout files
- Use pixel values for dimensions (use `dp` and `sp`)
- Forget to provide localized strings for all supported languages
- Use raster images when vectors would suffice

---

## 🔍 Debugging

### Logcat Filtering

Logcat is the primary tool for viewing system and application logs.

**Setting Up Logcat Filters:**

1. Open the **Logcat** tool window (View > Tool Windows > Logcat)
2. Use the filter bar with query syntax:

```
# Filter by tag
tag:MyViewModel

# Filter by log level
level:ERROR

# Filter by package
package:com.example.myapp

# Combine filters
package:com.example.myapp level:WARN tag:Network

# Exclude noise
package:com.example.myapp -tag:chatty

# Filter by message content
message:onClick
```

**Using Timber for Structured Logging:**

```kotlin
// Add dependency
// implementation("com.jakewharton.timber:timber:5.0.1")

// Initialize in Application class
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        if (BuildConfig.DEBUG) {
            Timber.plant(Timber.DebugTree())
        }
    }
}

// Usage in code
Timber.d("User clicked on item: %s", itemId)
Timber.e(exception, "Failed to load data")
Timber.w("Network request took %dms", elapsedTime)
```

### Breakpoints and Conditional Breakpoints

**Setting a Breakpoint:**
- Click in the gutter next to the line number, or press `Cmd + F8`

**Conditional Breakpoints:**
- Right-click on a breakpoint to add a condition:

```kotlin
// Break only when userId equals a specific value
userId == "user_123"

// Break only after N iterations
count > 100

// Break on a specific exception type
exception is IOException
```

**Log Breakpoints (Non-Suspending):**
- Right-click breakpoint > uncheck "Suspend" > add a log expression
- Useful for adding temporary logging without modifying code

### Evaluate Expression

During a debug session:
1. Pause execution at a breakpoint
2. Open **Evaluate Expression** (`Alt + F8`)
3. Type any valid Kotlin expression:

```kotlin
// Check a variable's state
viewModel.uiState.value

// Call a function
repository.getItemById("123")

// Modify a variable (use with caution)
counter = 0
```

### Layout Inspector (Compose)

The Layout Inspector allows you to inspect the Compose UI tree at runtime:

1. Run your app in debug mode
2. Open **Layout Inspector** (View > Tool Windows > Layout Inspector)
3. Select the running process
4. Features:
   - View the Compose node tree
   - Inspect modifier chains
   - See recomposition counts
   - View component parameters and state
   - 3D view of the layout hierarchy

### Network Inspector

1. Open **App Inspection** (View > Tool Windows > App Inspection)
2. Select the **Network Inspector** tab
3. Features:
   - View all HTTP requests and responses
   - Inspect headers, body, and timing
   - Filter by URL, method, or status code
   - Works with OkHttp and HttpURLConnection

### Database Inspector (Room)

1. Open **App Inspection** (View > Tool Windows > App Inspection)
2. Select the **Database Inspector** tab
3. Features:
   - Browse Room database tables
   - Execute SQL queries in real time
   - Modify data directly for testing
   - Observe live query results

✅ **DO:**
- Use Logcat query syntax for efficient filtering
- Use conditional breakpoints to avoid breaking on every iteration
- Use Layout Inspector to debug Compose recomposition issues
- Use Timber for structured logging in debug builds

❌ **DON'T:**
- Leave `Log.d()` calls in production code
- Use `println()` for debugging (use Timber or Logcat tags)
- Forget to remove debug breakpoints before committing
- Log sensitive data (tokens, passwords, PII)

---

## 📊 Profiler

### Accessing the Profiler

Open the **Profiler** tool window (View > Tool Windows > Profiler) or click the **Profile** button in the toolbar.

### CPU Profiler

Identifies performance bottlenecks, expensive function calls, and thread contention.

**Usage:**
1. Start a profiling session
2. Select **CPU** from the profiler dashboard
3. Choose a recording type:
   - **Callstack Sample Recording:** Low overhead, good for general profiling
   - **System Trace Recording:** Detailed system-level events (recommended for Compose)
   - **Java/Kotlin Method Trace:** Every method entry/exit (high overhead)

**What to Look For:**
- Functions with high "self time" (time spent in the function itself)
- Long-running operations on the main thread
- Excessive garbage collection pauses
- Thread contention and lock waits

### Memory Profiler

Identifies memory leaks, excessive allocations, and memory pressure.

**Usage:**
1. Select **Memory** from the profiler dashboard
2. Monitor the memory graph for:
   - Steady growth (potential leak)
   - Frequent GC events (excessive allocation)
   - Large allocations

**Heap Dump Analysis:**
- Capture a heap dump to inspect all allocated objects
- Look for Activity or Fragment instances that should have been garbage collected
- Check for large bitmap allocations

```kotlin
// Common memory leak patterns to avoid:

// BAD: Static reference to Activity context
companion object {
    var activityContext: Context? = null  // MEMORY LEAK!
}

// GOOD: Use Application context for long-lived references
companion object {
    lateinit var appContext: Context  // Application context is safe
}

// BAD: Anonymous inner class holding Activity reference
handler.postDelayed(object : Runnable {
    override fun run() {
        // This holds a reference to the enclosing Activity
        textView.text = "Updated"
    }
}, 10000)

// GOOD: Use lifecycle-aware components
lifecycleScope.launch {
    delay(10000)
    textView.text = "Updated"
}
```

### Energy Profiler

Monitors battery usage patterns:
- CPU wake locks
- Network activity
- GPS usage
- Alarm scheduling

### Network Profiler

Tracks all network requests with timing information:
- Request/response timeline
- Payload sizes
- Connection reuse
- SSL handshake time

### Profiling Compose Recomposition

**Method 1: Layout Inspector Recomposition Counts**
1. Open Layout Inspector with a debug build
2. Enable "Show Recomposition Counts" in the toolbar
3. Interact with the app and observe which composables recompose frequently

**Method 2: Compose Compiler Metrics**

```kotlin
// build.gradle.kts
composeCompiler {
    reportsDestination = layout.buildDirectory.dir("compose_compiler")
}
```

Run the build, then inspect the generated reports:
- `*-composables.txt`: Stability of each composable
- `*-classes.txt`: Stability of data classes
- `*-composables.csv`: Machine-readable composable data

**Method 3: System Trace with Compose Markers**

```kotlin
// Add compose tracing dependency
// debugImplementation("androidx.compose.runtime:runtime-tracing:1.0.0-beta01")

// Compose automatically adds trace markers for composable functions
// View them in a System Trace recording in the CPU Profiler
```

✅ **DO:**
- Profile on a real device (emulator performance differs significantly)
- Use System Trace for Compose performance analysis
- Check for memory leaks after screen navigation
- Profile release builds (with debuggable flag) for accurate results

❌ **DON'T:**
- Profile only on the emulator (not representative of real devices)
- Ignore GC pressure in memory profiling
- Use Java/Kotlin Method Trace for general profiling (too much overhead)
- Optimize without profiling first (measure, then optimize)

---

## 🧪 Testing in Android Studio

### Run Configurations for Unit Tests

**Running a Single Test:**
1. Click the green play icon in the gutter next to a test function
2. Or right-click the test class/function > Run

**Running All Tests in a Module:**
1. Right-click the `test` directory > Run Tests
2. Or use the Gradle task: `./gradlew :app:test`

**Example Unit Test:**

```kotlin
class RestaurantRepositoryTest {

    private lateinit var sut: RestaurantRepository
    private val mockApi: RestaurantApi = mockk()
    private val mockDao: RestaurantDao = mockk()

    @BeforeTest
    fun setUp() {
        sut = RestaurantRepositoryImpl(
            api = mockApi,
            dao = mockDao
        )
    }

    @Test
    fun `should return restaurants from API when cache is empty`() = runTest {
        // Given
        val expectedRestaurants = listOf(
            Restaurant(id = "1", name = "Test Restaurant")
        )
        coEvery { mockDao.getAll() } returns emptyList()
        coEvery { mockApi.getRestaurants() } returns expectedRestaurants

        // When
        val result = sut.getRestaurants()

        // Then
        assertEquals(expectedRestaurants, result)
        coVerify { mockApi.getRestaurants() }
    }

    @Test
    fun `should return cached restaurants when available`() = runTest {
        // Given
        val cachedRestaurants = listOf(
            Restaurant(id = "1", name = "Cached Restaurant")
        )
        coEvery { mockDao.getAll() } returns cachedRestaurants

        // When
        val result = sut.getRestaurants()

        // Then
        assertEquals(cachedRestaurants, result)
        coVerify(exactly = 0) { mockApi.getRestaurants() }
    }
}
```

### Run Configurations for Instrumented Tests

Instrumented tests run on a device or emulator and have access to the Android framework.

**Running Instrumented Tests:**
1. Right-click the `androidTest` directory > Run Tests
2. Or use: `./gradlew :app:connectedAndroidTest`

**Example Compose UI Test:**

```kotlin
@RunWith(AndroidJUnit4::class)
class HomeScreenTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun homeScreen_displaysSearchBar() {
        // Given
        composeTestRule.setContent {
            MyAppTheme {
                HomeScreen()
            }
        }

        // Then
        composeTestRule
            .onNodeWithText("Search restaurants...")
            .assertIsDisplayed()
    }

    @Test
    fun homeScreen_clickingItem_navigatesToDetail() {
        // Given
        var navigatedToId: String? = null
        composeTestRule.setContent {
            MyAppTheme {
                HomeScreen(
                    onItemClick = { id -> navigatedToId = id }
                )
            }
        }

        // When
        composeTestRule
            .onNodeWithTag("restaurant_item_1")
            .performClick()

        // Then
        assertEquals("1", navigatedToId)
    }
}
```

### Coverage Visualization

**Running Tests with Coverage:**
1. Right-click a test class or directory
2. Select **Run with Coverage**
3. Or use: `./gradlew :app:testDebugUnitTestCoverage`

**Viewing Coverage:**
- Green: Line is covered by tests
- Red: Line is not covered
- Yellow: Branch partially covered

**Configuring JaCoCo for Coverage Reports:**

```kotlin
// app/build.gradle.kts
plugins {
    jacoco
}

tasks.withType<Test> {
    extensions.configure<JacocoTaskExtension> {
        isIncludeNoLocationClasses = true
        excludes = listOf("jdk.internal.*")
    }
}
```

✅ **DO:**
- Write unit tests for ViewModels, UseCases, and Repositories
- Write Compose UI tests for critical user flows
- Use test doubles (mocks, fakes) for external dependencies
- Run tests with coverage to identify untested code paths

❌ **DON'T:**
- Skip testing because "it works on my device"
- Write tests that depend on network or database state
- Test private implementation details (test behavior, not implementation)
- Ignore flaky tests (fix them or mark them appropriately)

---

## ⌨️ Keyboard Shortcuts (macOS)

### Essential Shortcuts

| Action | Shortcut |
|--------|----------|
| Run | `Ctrl + R` |
| Debug | `Ctrl + D` |
| Stop | `Cmd + F2` |
| Find in Files | `Cmd + Shift + F` |
| Replace in Files | `Cmd + Shift + R` |
| Go to File | `Cmd + Shift + O` |
| Go to Symbol | `Cmd + Alt + O` |
| Reformat Code | `Cmd + Alt + L` |
| Generate | `Cmd + N` |
| Quick Fix | `Alt + Enter` |
| Refactor This | `Ctrl + T` |
| Build Project | `Cmd + F9` |
| Find Usages | `Alt + F7` |
| Navigate Back | `Cmd + [` |
| Navigate Forward | `Cmd + ]` |
| Toggle Line Comment | `Cmd + /` |
| Duplicate Line | `Cmd + D` |

### Navigation

| Action | Shortcut |
|--------|----------|
| Go to Class | `Cmd + O` |
| Go to File | `Cmd + Shift + O` |
| Go to Symbol | `Cmd + Alt + O` |
| Go to Declaration | `Cmd + B` |
| Go to Implementation | `Cmd + Alt + B` |
| Go to Super | `Cmd + U` |
| Recent Files | `Cmd + E` |
| File Structure | `Cmd + F12` |
| Navigate to Test | `Cmd + Shift + T` |
| Search Everywhere | `Double Shift` |

### Editing

| Action | Shortcut |
|--------|----------|
| Complete Statement | `Cmd + Shift + Enter` |
| Smart Completion | `Ctrl + Shift + Space` |
| Override Methods | `Ctrl + O` |
| Implement Methods | `Ctrl + I` |
| Surround With | `Cmd + Alt + T` |
| Unwrap | `Cmd + Shift + Delete` |
| Extend Selection | `Alt + Up` |
| Shrink Selection | `Alt + Down` |
| Move Line Up | `Alt + Shift + Up` |
| Move Line Down | `Alt + Shift + Down` |
| Delete Line | `Cmd + Backspace` |
| Join Lines | `Ctrl + Shift + J` |

### Debugging

| Action | Shortcut |
|--------|----------|
| Toggle Breakpoint | `Cmd + F8` |
| Step Over | `F8` |
| Step Into | `F7` |
| Step Out | `Shift + F8` |
| Resume | `Cmd + Alt + R` |
| Evaluate Expression | `Alt + F8` |
| View Breakpoints | `Cmd + Shift + F8` |
| Run to Cursor | `Alt + F9` |

---

## 🔧 Useful Plugins

### Recommended Plugins

| Plugin | Purpose |
|--------|---------|
| **JSON to Kotlin Class** | Generate Kotlin data classes from JSON |
| **Database Navigator** | Browse and query databases directly in the IDE |
| **ADB Idea** | Quick ADB commands (clear data, revoke permissions, uninstall) |
| **Detekt** | Static code analysis for Kotlin |
| **Rainbow Brackets** | Color-coded bracket pairs for readability |
| **Key Promoter X** | Learn keyboard shortcuts by showing them when you use the mouse |
| **String Manipulation** | Case conversion, encoding/decoding, and text transformations |
| **GitToolBox** | Enhanced Git integration (inline blame, status bar, auto-fetch) |

### Installing Plugins

1. Open **Settings** (Cmd + ,)
2. Navigate to **Plugins**
3. Search the **Marketplace** tab
4. Click **Install** and restart if prompted

### Plugin Configuration Tips

**ADB Idea:**
- Access via Tools > ADB Idea, or assign a custom shortcut
- Most useful commands:
  - Clear App Data and Restart
  - Revoke Permissions
  - Kill App
  - Uninstall App

**Detekt:**

```kotlin
// build.gradle.kts
plugins {
    id("io.gitlab.arturbosch.detekt") version "1.23.7"
}

detekt {
    config.setFrom("$rootDir/config/detekt/detekt.yml")
    buildUponDefaultConfig = true
    allRules = false
}
```

✅ **DO:**
- Install only plugins you actively use (too many plugins slow down the IDE)
- Keep plugins updated
- Use Key Promoter X to learn shortcuts faster

❌ **DON'T:**
- Install plugins from untrusted sources
- Use plugins that duplicate built-in functionality
- Keep outdated or abandoned plugins installed

---

## 📋 Android Studio Checklist

Use this checklist when setting up a new project or onboarding to an existing one.

### Initial Setup

- [ ] Android Studio is updated to the latest stable version
- [ ] Required SDK platforms are installed (API 35)
- [ ] Build tools are installed (35.0.0+)
- [ ] JDK 17 is configured (use bundled JBR)
- [ ] Android Emulator is installed with a target device image
- [ ] Hardware acceleration (HAXM or Hypervisor) is enabled

### Project Configuration

- [ ] `compileSdk` is set to 35
- [ ] `minSdk` is set to 26
- [ ] `targetSdk` is set to 35
- [ ] Kotlin version is 2.1+
- [ ] Compose compiler plugin is applied
- [ ] Version catalog (`libs.versions.toml`) is configured
- [ ] `.gitignore` excludes `local.properties`, `.idea/`, `build/`, `*.iml`

### Build Variants

- [ ] Debug build type has `applicationIdSuffix`
- [ ] Release build type has minification and resource shrinking enabled
- [ ] Signing configuration uses environment variables (not hardcoded)
- [ ] ProGuard/R8 rules are tested with a release build
- [ ] BuildConfig fields are defined for each variant

### Code Quality

- [ ] Detekt or ktlint is configured for static analysis
- [ ] Code formatting rules are shared via `.editorconfig` or IDE settings
- [ ] Pre-commit hooks are set up for linting
- [ ] Test coverage thresholds are configured

### Debugging & Profiling

- [ ] Timber is configured for debug logging
- [ ] Network Inspector is verified to work with your HTTP client
- [ ] Layout Inspector is verified to work with Compose
- [ ] Profiler is tested on a real device

---

## 🚫 Common Issues and Fixes

### 1. Gradle Sync Failed

**Symptoms:** Red error banner, unresolved references, build errors.

**Common Causes and Fixes:**

```
# Issue: Incompatible Gradle and AGP versions
# Fix: Check compatibility matrix at developer.android.com/build/releases/gradle-plugin

# Issue: Missing SDK components
# Fix: Open SDK Manager and install required platforms/build tools

# Issue: Network timeout
# Fix: Check proxy settings in gradle.properties
systemProp.http.proxyHost=proxy.example.com
systemProp.http.proxyPort=8080

# Issue: Corrupted Gradle cache
# Fix: Invalidate caches and restart
# File > Invalidate Caches > Invalidate and Restart

# Or manually clear the Gradle cache:
rm -rf ~/.gradle/caches/
```

**Gradle Wrapper Fix:**

```bash
# Regenerate the Gradle wrapper
./gradlew wrapper --gradle-version=8.9
```

### 2. Emulator Won't Start

**Symptoms:** Emulator hangs on boot, black screen, or crashes.

**Fixes:**

```bash
# Check if hardware acceleration is available
emulator -accel-check

# Cold boot the emulator (bypasses snapshot issues)
# AVD Manager > ▼ dropdown on device > Cold Boot Now

# Wipe emulator data
# AVD Manager > ▼ dropdown on device > Wipe Data

# Increase emulator RAM
# AVD Manager > Edit > Show Advanced Settings > RAM: 2048 MB

# On macOS with Apple Silicon, use ARM64 system images
# (x86 images won't work on Apple Silicon without translation)
```

**If Emulator Still Fails:**
1. Delete the AVD and create a new one
2. Ensure Hypervisor Framework is enabled (macOS: `sysctl kern.hv_support`)
3. Try a different API level system image
4. Use a physical device as a fallback

### 3. Slow Builds

**Symptoms:** Builds taking several minutes, IDE freezing during build.

**Gradle Performance Tuning:**

```properties
# gradle.properties

# Increase JVM heap size
org.gradle.jvmargs=-Xmx4096m -XX:+UseParallelGC

# Enable Gradle daemon
org.gradle.daemon=true

# Enable parallel execution
org.gradle.parallel=true

# Enable build caching
org.gradle.caching=true

# Enable configuration cache (Gradle 8+)
org.gradle.configuration-cache=true

# Use non-transitive R classes
android.nonTransitiveRClass=true
```

**Additional Build Speed Tips:**
- Use modularization to limit incremental build scope
- Avoid using `kapt`; migrate to KSP where possible
- Keep annotation processors up to date
- Use `--build-cache` and remote build caching for CI
- Avoid dynamic dependency versions (`implementation("lib:+")`)

### 4. Compose Preview Not Rendering

**Symptoms:** Preview shows "Render problem" or "Build & Refresh" message.

**Fixes:**

```kotlin
// Ensure @Preview annotation is correct
@Preview(showBackground = true)
@Composable
fun MyComposablePreview() {
    MyAppTheme {
        MyComposable()
    }
}

// Previews cannot access ViewModel or navigation directly
// Use preview parameter providers for dynamic data

@Preview
@Composable
fun RestaurantCardPreview(
    @PreviewParameter(RestaurantPreviewProvider::class) restaurant: Restaurant
) {
    RestaurantCard(restaurant = restaurant)
}

class RestaurantPreviewProvider : PreviewParameterProvider<Restaurant> {
    override val values = sequenceOf(
        Restaurant(id = "1", name = "Pizza Place", rating = 4.5),
        Restaurant(id = "2", name = "Sushi Bar", rating = 4.8)
    )
}
```

**Common Preview Issues:**
1. **Build project first** - Previews require a successful build
2. **Check for runtime dependencies** - Previews cannot access network or database
3. **Avoid side effects** - Previews should be pure composable functions
4. **Memory limit** - Large previews may need increased IDE memory:
   - Help > Edit Custom VM Options > `-Xmx4096m`

### 5. Kotlin Unresolved Reference After Updating

**Fixes:**

```bash
# Clean and rebuild
./gradlew clean
./gradlew assembleDebug

# Invalidate caches
# File > Invalidate Caches > Invalidate and Restart

# Sync Gradle
# File > Sync Project with Gradle Files
```

---

## 📚 Further Reading

### Official Documentation

- [Android Studio User Guide](https://developer.android.com/studio/intro)
- [Android Developer Guides](https://developer.android.com/guide)
- [Jetpack Compose Documentation](https://developer.android.com/compose)
- [Gradle Build Configuration](https://developer.android.com/build)
- [Testing in Android](https://developer.android.com/training/testing)

### Jetpack Compose Resources

- [Compose Pathway](https://developer.android.com/courses/pathways/compose)
- [Compose Samples (GitHub)](https://github.com/android/compose-samples)
- [Compose Performance Guide](https://developer.android.com/develop/ui/compose/performance)
- [Compose Stability Explained](https://developer.android.com/develop/ui/compose/performance/stability)

### Architecture

- [Guide to App Architecture](https://developer.android.com/topic/architecture)
- [Now in Android (Reference App)](https://github.com/android/nowinandroid)
- [Modularization Guide](https://developer.android.com/topic/modularization)

### Build & Tooling

- [Gradle Version Catalog](https://docs.gradle.org/current/userguide/platforms.html)
- [R8 Shrinking & Obfuscation](https://developer.android.com/build/shrink-code)
- [Build Performance](https://developer.android.com/build/optimize-your-build)

### Testing

- [Testing Compose Layouts](https://developer.android.com/develop/ui/compose/testing)
- [MockK Library](https://mockk.io/)
- [Turbine (Flow Testing)](https://github.com/cashapp/turbine)
- [Robolectric](http://robolectric.org/)

### Community

- [Kotlin Slack](https://kotlinlang.slack.com)
- [Android Dev Subreddit](https://www.reddit.com/r/androiddev/)
- [Google Issue Tracker](https://issuetracker.google.com/issues?q=componentid:192708)
