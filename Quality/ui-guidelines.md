# 🎨 UI Guidelines

**ARC Labs apps follow Material 3 Expressive (material3 1.5.0-beta01) with a brand color scheme (dynamic color off), consistent theming, accessibility, and localization across all screens.**

---

## 🎯 UI Philosophy

- **Material 3 Expressive** as the foundation for all visual, motion and interaction patterns
- **Native first**: use Jetpack Compose components before reaching for custom solutions
- **Consistent** across light and dark themes, all screen sizes, and device types
- **Accessible** to all users, including those using assistive technologies
- **Simple, Lovable, Complete**: every screen should feel polished and intentional
- **Performance-conscious**: smooth 60fps interactions with no visible jank

Material Design 3 provides a comprehensive design system that handles color, typography, shape, motion, and interaction patterns. ARC Labs builds on top of M3 rather than fighting against it. When M3 provides a component or pattern, prefer it over custom implementations.

---

## 🎨 Material Design 3

### Material 3 Expressive

ARC Labs apps use **Material 3 Expressive** — the evolution of Material 3 with research-backed updates to components, motion, shape and typography — via `androidx.compose.material3:material3` **1.5.0-beta01**, pinned explicitly in the version catalog on top of Compose BOM 2026.09.00 (the BOM alone resolves the stable line).

Expressive changes what "native" means: prefer the Expressive variants and defaults the library ships (expressive motion scheme, shape morphing, updated components such as button groups, toolbars and loading indicators) instead of re-creating them by hand. API surface in a beta moves: **verify every Expressive API** (name, parameters, `@ExperimentalMaterial3ExpressiveApi` opt-in status) with Context7 or `android docs` before using it.

### Theme Setup

Every ARC Labs app has one theme composable, `AppTheme`, in `:core:designsystem`. **Dynamic color is off**: ARC Labs apps carry their brand identity, so the color scheme always comes from the brand light/dark schemes.

```kotlin
// Verify MaterialExpressiveTheme's parameters and opt-in requirement against material3 1.5.0-beta01.
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit,
) {
    MaterialExpressiveTheme(
        colorScheme = if (darkTheme) DarkColorScheme else LightColorScheme,
        motionScheme = MotionScheme.expressive(),
        typography = AppTypography,
        shapes = AppShapes,
        content = content,
    )
}
```

- ✅ Brand `lightColorScheme` / `darkColorScheme` generated from the brand colors (Material Theme Builder) and kept in `:core:designsystem` — the only place color literals may appear.
- ✅ Light and dark themes are both first-class; every screen is previewed and screenshot-tested in both.
- ❌ No `dynamicLightColorScheme` / `dynamicDarkColorScheme` in ARC Labs apps (wallpaper colors would replace the brand).
- ❌ No custom re-implementations of M3 components; customise through component parameters, `*Defaults`, theme tokens or the Compose Styles API (official `styles` skill).

### Custom Color Scheme Definition

Define both light and dark color schemes explicitly. Use the Material Theme Builder tool to generate these values from your brand colors.

```kotlin
// ✅ Correct: Define complete color schemes
private val LightColorScheme = lightColorScheme(
    primary = Color(0xFF6750A4),
    onPrimary = Color(0xFFFFFFFF),
    primaryContainer = Color(0xFFEADDFF),
    onPrimaryContainer = Color(0xFF21005D),
    secondary = Color(0xFF625B71),
    onSecondary = Color(0xFFFFFFFF),
    secondaryContainer = Color(0xFFE8DEF8),
    onSecondaryContainer = Color(0xFF1D192B),
    tertiary = Color(0xFF7D5260),
    onTertiary = Color(0xFFFFFFFF),
    tertiaryContainer = Color(0xFFFFD8E4),
    onTertiaryContainer = Color(0xFF31111D),
    surface = Color(0xFFFFFBFE),
    onSurface = Color(0xFF1C1B1F),
    surfaceVariant = Color(0xFFE7E0EC),
    onSurfaceVariant = Color(0xFF49454F),
    outline = Color(0xFF79747E),
    outlineVariant = Color(0xFFCAC4D0),
    error = Color(0xFFB3261E),
    onError = Color(0xFFFFFFFF),
    errorContainer = Color(0xFFF9DEDC),
    onErrorContainer = Color(0xFF410E0B),
)

private val DarkColorScheme = darkColorScheme(
    primary = Color(0xFFD0BCFF),
    onPrimary = Color(0xFF381E72),
    primaryContainer = Color(0xFF4F378B),
    onPrimaryContainer = Color(0xFFEADDFF),
    secondary = Color(0xFFCCC2DC),
    onSecondary = Color(0xFF332D41),
    secondaryContainer = Color(0xFF4A4458),
    onSecondaryContainer = Color(0xFFE8DEF8),
    tertiary = Color(0xFFEFB8C8),
    onTertiary = Color(0xFF492532),
    tertiaryContainer = Color(0xFF633B48),
    onTertiaryContainer = Color(0xFFFFD8E4),
    surface = Color(0xFF1C1B1F),
    onSurface = Color(0xFFE6E1E5),
    surfaceVariant = Color(0xFF49454F),
    onSurfaceVariant = Color(0xFFCAC4D0),
    outline = Color(0xFF938F99),
    outlineVariant = Color(0xFF49454F),
    error = Color(0xFFF2B8B5),
    onError = Color(0xFF601410),
    errorContainer = Color(0xFF8C1D18),
    onErrorContainer = Color(0xFFF9DEDC),
)
```

### Color Roles

| Role | Usage |
|------|-------|
| primary | Main action buttons, active states |
| onPrimary | Text/icons on primary color |
| primaryContainer | Filled tonal buttons, selected chips |
| onPrimaryContainer | Text/icons on primary container |
| secondary | Less prominent components, filters |
| onSecondary | Text/icons on secondary color |
| secondaryContainer | Tonal buttons, chips |
| tertiary | Contrasting accents, complementary elements |
| surface | Backgrounds, cards, sheets |
| onSurface | Text/icons on surface |
| surfaceVariant | Alternate surface for differentiation (e.g., search bars) |
| onSurfaceVariant | Secondary text, icons on surface variant |
| outline | Borders, dividers, outlines |
| outlineVariant | Subtle borders (e.g., between cards) |
| inverseSurface | Snackbar backgrounds, inverted UI elements |
| inverseOnSurface | Text/icons on inverse surface |
| error | Error states, destructive actions |
| onError | Text/icons on error color |
| errorContainer | Error banners, validation backgrounds |
| onErrorContainer | Text/icons on error container |

### Typography Scale

| Style | Usage |
|-------|-------|
| displayLarge/Medium/Small | Hero text, large numbers |
| headlineLarge/Medium/Small | Screen titles, section headers |
| titleLarge/Medium/Small | Card titles, list item primary text |
| bodyLarge/Medium/Small | Body text, descriptions |
| labelLarge/Medium/Small | Buttons, tabs, chips |

### Custom Font Setup

```kotlin
// ✅ Correct: Define typography with custom fonts
val AppFontFamily = FontFamily(
    Font(R.font.inter_regular, FontWeight.Normal),
    Font(R.font.inter_medium, FontWeight.Medium),
    Font(R.font.inter_semibold, FontWeight.SemiBold),
    Font(R.font.inter_bold, FontWeight.Bold),
)

val AppTypography = Typography(
    displayLarge = TextStyle(
        fontFamily = AppFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 57.sp,
        lineHeight = 64.sp,
        letterSpacing = (-0.25).sp,
    ),
    headlineMedium = TextStyle(
        fontFamily = AppFontFamily,
        fontWeight = FontWeight.SemiBold,
        fontSize = 28.sp,
        lineHeight = 36.sp,
    ),
    titleLarge = TextStyle(
        fontFamily = AppFontFamily,
        fontWeight = FontWeight.Medium,
        fontSize = 22.sp,
        lineHeight = 28.sp,
    ),
    bodyLarge = TextStyle(
        fontFamily = AppFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp,
    ),
    bodyMedium = TextStyle(
        fontFamily = AppFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.25.sp,
    ),
    labelLarge = TextStyle(
        fontFamily = AppFontFamily,
        fontWeight = FontWeight.Medium,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.1.sp,
    ),
)
```

### Shapes

Material Design 3 uses rounded corner shapes at three sizes. Define these consistently across your app.

```kotlin
val AppShapes = Shapes(
    small = RoundedCornerShape(4.dp),   // Chips, small buttons
    medium = RoundedCornerShape(12.dp), // Cards, dialogs
    large = RoundedCornerShape(16.dp),  // Bottom sheets, large cards
    extraLarge = RoundedCornerShape(28.dp), // FAB, large containers
)
```

### Elevation and Tonal Elevation

M3 uses tonal elevation instead of shadow elevation. Tonal elevation overlays the surface color with the primary color at varying opacities.

```kotlin
// ✅ Correct: Use tonalElevation for M3 surfaces
Surface(
    tonalElevation = 2.dp,
    modifier = Modifier.fillMaxWidth(),
) {
    Text("This surface has a subtle tonal tint")
}

// ❌ Incorrect: Using shadow elevation as the primary visual indicator
Surface(
    shadowElevation = 8.dp, // Shadow-only elevation, not M3 style
    modifier = Modifier.fillMaxWidth(),
) {
    Text("This relies on shadows instead of tonal elevation")
}
```

| Level | Tonal Elevation | Usage |
|-------|----------------|-------|
| Level 0 | 0.dp | Default surface |
| Level 1 | 1.dp | Cards, navigation rail |
| Level 2 | 3.dp | Bottom app bar, elevated buttons |
| Level 3 | 6.dp | FAB, navigation drawer |
| Level 4 | 8.dp | Dialogs, menus |
| Level 5 | 12.dp | Top of stack elements |

---

## 🧩 Component Usage Patterns

### Buttons

M3 provides several button types. Choose the correct one based on emphasis level.

| Button Type | Emphasis | When to Use |
|-------------|----------|-------------|
| `Button` (Filled) | Highest | Primary actions (Save, Submit, Confirm) |
| `FilledTonalButton` | Medium-High | Important but not primary (Filter, Sort) |
| `OutlinedButton` | Medium | Secondary actions (Cancel, Back) |
| `TextButton` | Low | Tertiary actions (Learn more, Skip) |
| `FloatingActionButton` | Highest | Primary screen action (Create, Compose) |

```kotlin
// ✅ Correct: Use the right button for the action's emphasis level
@Composable
fun ActionButtons(
    onConfirm: () -> Unit,
    onCancel: () -> Unit,
    onLearnMore: () -> Unit,
) {
    // Primary action — filled button
    Button(onClick = onConfirm) {
        Text("Confirm Reservation")
    }

    // Secondary action — outlined button
    OutlinedButton(onClick = onCancel) {
        Text("Cancel")
    }

    // Tertiary action — text button
    TextButton(onClick = onLearnMore) {
        Text("Learn More")
    }
}

// ✅ Correct: FAB for the primary screen action
FloatingActionButton(
    onClick = onCreateReview,
    containerColor = MaterialTheme.colorScheme.primaryContainer,
    contentColor = MaterialTheme.colorScheme.onPrimaryContainer,
) {
    Icon(Icons.Default.Add, contentDescription = stringResource(R.string.create_review))
}

// ✅ Correct: Extended FAB with label
ExtendedFloatingActionButton(
    onClick = onCreateReview,
    icon = { Icon(Icons.Default.Edit, contentDescription = null) },
    text = { Text("Write Review") },
)
```

### Cards

```kotlin
// ✅ Elevated card — for content that needs visual prominence
ElevatedCard(
    modifier = Modifier.fillMaxWidth(),
    colors = CardDefaults.elevatedCardColors(),
) {
    Column(modifier = Modifier.padding(16.dp)) {
        Text("Restaurant Name", style = MaterialTheme.typography.titleMedium)
        Text("Description", style = MaterialTheme.typography.bodyMedium)
    }
}

// ✅ Filled card — for grouped content on a surface
Card(
    modifier = Modifier.fillMaxWidth(),
    colors = CardDefaults.cardColors(
        containerColor = MaterialTheme.colorScheme.surfaceVariant,
    ),
) {
    Column(modifier = Modifier.padding(16.dp)) {
        Text("Menu Item", style = MaterialTheme.typography.titleSmall)
        Text("$12.99", style = MaterialTheme.typography.bodyLarge)
    }
}

// ✅ Outlined card — for content at the same elevation as its container
OutlinedCard(
    modifier = Modifier.fillMaxWidth(),
    border = BorderStroke(1.dp, MaterialTheme.colorScheme.outlineVariant),
) {
    Column(modifier = Modifier.padding(16.dp)) {
        Text("Note", style = MaterialTheme.typography.titleSmall)
        Text("Additional information", style = MaterialTheme.typography.bodyMedium)
    }
}
```

### Top App Bars

```kotlin
// ✅ Center-aligned — for screens with a single prominent title
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun HomeTopBar(scrollBehavior: TopAppBarScrollBehavior) {
    CenterAlignedTopAppBar(
        title = { Text("Favorites") },
        navigationIcon = {
            IconButton(onClick = { /* open drawer */ }) {
                Icon(Icons.Default.Menu, contentDescription = stringResource(R.string.open_menu))
            }
        },
        actions = {
            IconButton(onClick = { /* search */ }) {
                Icon(Icons.Default.Search, contentDescription = stringResource(R.string.search))
            }
        },
        scrollBehavior = scrollBehavior,
    )
}

// ✅ Large — for screens that benefit from a prominent title that collapses on scroll
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun DetailTopBar(
    title: String,
    onBack: () -> Unit,
    scrollBehavior: TopAppBarScrollBehavior,
) {
    LargeTopAppBar(
        title = { Text(title) },
        navigationIcon = {
            IconButton(onClick = onBack) {
                Icon(Icons.AutoMirrored.Filled.ArrowBack, contentDescription = stringResource(R.string.go_back))
            }
        },
        scrollBehavior = scrollBehavior,
    )
}

// ✅ Medium — a balanced option for lists and detail screens
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun SettingsTopBar(onBack: () -> Unit, scrollBehavior: TopAppBarScrollBehavior) {
    MediumTopAppBar(
        title = { Text("Settings") },
        navigationIcon = {
            IconButton(onClick = onBack) {
                Icon(Icons.AutoMirrored.Filled.ArrowBack, contentDescription = stringResource(R.string.go_back))
            }
        },
        scrollBehavior = scrollBehavior,
    )
}
```

### Bottom Sheets

```kotlin
// ✅ Modal bottom sheet for contextual actions
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun FilterBottomSheet(
    onDismiss: () -> Unit,
    sheetState: SheetState = rememberModalBottomSheetState(),
) {
    ModalBottomSheet(
        onDismissRequest = onDismiss,
        sheetState = sheetState,
        dragHandle = { BottomSheetDefaults.DragHandle() },
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(
                text = "Filter Options",
                style = MaterialTheme.typography.titleLarge,
            )
            Spacer(modifier = Modifier.height(16.dp))
            // Filter content here
            Spacer(modifier = Modifier.height(32.dp)) // Bottom padding for navigation bar
        }
    }
}
```

### Dialogs

```kotlin
// ✅ AlertDialog for confirmations and decisions
@Composable
fun ConfirmDeleteDialog(
    onConfirm: () -> Unit,
    onDismiss: () -> Unit,
) {
    AlertDialog(
        onDismissRequest = onDismiss,
        icon = { Icon(Icons.Default.Delete, contentDescription = null) },
        title = { Text("Delete Review?") },
        text = { Text("This action cannot be undone. Your review will be permanently removed.") },
        confirmButton = {
            TextButton(onClick = onConfirm) {
                Text("Delete", color = MaterialTheme.colorScheme.error)
            }
        },
        dismissButton = {
            TextButton(onClick = onDismiss) {
                Text("Cancel")
            }
        },
    )
}
```

### Snackbars

```kotlin
// ✅ Snackbar with SnackbarHost in Scaffold
@Composable
fun MyScreen() {
    val snackbarHostState = remember { SnackbarHostState() }
    val scope = rememberCoroutineScope()

    Scaffold(
        snackbarHost = { SnackbarHost(snackbarHostState) },
    ) { padding ->
        // Screen content
        Button(
            onClick = {
                scope.launch {
                    val result = snackbarHostState.showSnackbar(
                        message = "Review deleted",
                        actionLabel = "Undo",
                        duration = SnackbarDuration.Short,
                    )
                    if (result == SnackbarResult.ActionPerformed) {
                        // Handle undo
                    }
                }
            },
            modifier = Modifier.padding(padding),
        ) {
            Text("Delete")
        }
    }
}
```

---

## 🌙 Dark Theme

### Core Principles

- ✅ Use `MaterialTheme.colorScheme` (adapts automatically to light/dark)
- ✅ Test all screens in both themes
- ✅ Use surface colors for backgrounds
- ✅ Use `onSurface` and `onSurfaceVariant` for text
- ❌ Don't hardcode `Color(0xFF...)` values
- ❌ Don't use pure black (`#000000`) for dark backgrounds
- ❌ Don't assume white text in dark theme — use semantic colors

### Preview Both Themes

```kotlin
@Preview(name = "Light", showBackground = true)
@Preview(name = "Dark", showBackground = true, uiMode = UI_MODE_NIGHT_YES)
@Composable
private fun MyScreenPreview() {
    AppTheme { MyScreen() }
}
```

### Testing with PreviewParameterProvider

```kotlin
class ThemePreviewProvider : PreviewParameterProvider<Boolean> {
    override val values = sequenceOf(false, true)
}

@Preview(showBackground = true)
@Composable
private fun CardPreview(
    @PreviewParameter(ThemePreviewProvider::class) darkTheme: Boolean,
) {
    AppTheme(darkTheme = darkTheme) {
        RestaurantCard(sampleRestaurant)
    }
}
```

### Surface Color Roles for Dark Theme

In dark theme, surfaces use tonal elevation to create a visual hierarchy. Higher elevation surfaces are lighter, not darker.

```kotlin
// ✅ Correct: Use surface colors that adapt to theme
Surface(
    color = MaterialTheme.colorScheme.surface,
    tonalElevation = 3.dp,
) {
    Text(
        text = "Elevated content",
        color = MaterialTheme.colorScheme.onSurface,
    )
}

// ❌ Incorrect: Hardcoding dark theme colors
Surface(
    color = Color(0xFF121212), // Hardcoded dark color
) {
    Text(
        text = "This won't adapt to light theme",
        color = Color.White, // Hardcoded text color
    )
}
```

### Images and Icons in Dark Mode

```kotlin
// ✅ Correct: Use tinted icons from MaterialTheme
Icon(
    imageVector = Icons.Default.Star,
    contentDescription = null,
    tint = MaterialTheme.colorScheme.primary, // Adapts to theme
)

// ❌ Incorrect: Hardcoded tint color
Icon(
    imageVector = Icons.Default.Star,
    contentDescription = null,
    tint = Color(0xFFFFD700), // Gold — won't adapt, may clash in dark theme
)

// ✅ Correct: Adjust image alpha for dark theme
val alpha = if (isSystemInDarkTheme()) 0.8f else 1.0f
Image(
    painter = painterResource(R.drawable.hero_image),
    contentDescription = stringResource(R.string.hero_image_desc),
    alpha = alpha,
)
```

---

## ♿ Accessibility

### Content Descriptions

```kotlin
// ✅ Meaningful descriptions for interactive elements
Icon(
    imageVector = Icons.Default.Favorite,
    contentDescription = stringResource(R.string.add_to_favorites),
)

// ✅ Decorative elements — set contentDescription to null
Icon(
    imageVector = Icons.Default.ChevronRight,
    contentDescription = null, // Decorative, not announced by TalkBack
)

// ✅ Images with meaningful content
Image(
    painter = painterResource(R.drawable.restaurant_photo),
    contentDescription = stringResource(R.string.restaurant_photo_desc, restaurantName),
)

// ✅ Clickable rows with complete descriptions
Row(
    modifier = Modifier
        .clickable(onClick = onNavigate)
        .semantics {
            contentDescription = "Navigate to $restaurantName, rated $rating stars"
        },
) {
    // Row content
}
```

### Touch Targets

```kotlin
// ✅ Minimum 48dp touch target — IconButton handles this automatically
IconButton(onClick = { /* ... */ }) {
    Icon(Icons.Default.Close, contentDescription = stringResource(R.string.close))
}

// ✅ For custom clickable elements, ensure minimum size
Box(
    modifier = Modifier
        .sizeIn(minWidth = 48.dp, minHeight = 48.dp)
        .clickable(onClick = onTap),
    contentAlignment = Alignment.Center,
) {
    Icon(
        imageVector = Icons.Default.Info,
        contentDescription = stringResource(R.string.info),
        modifier = Modifier.size(24.dp), // Visual size can be smaller
    )
}
```

### Semantics

```kotlin
// ✅ Custom semantics for complex components
Box(
    modifier = Modifier.semantics {
        contentDescription = "User rating: 4.5 out of 5 stars"
        stateDescription = "Favorited"
    }
)

// ✅ Merge semantics for composite elements
Row(modifier = Modifier.semantics(mergeDescendants = true) { }) {
    Icon(Icons.Default.Star, contentDescription = null)
    Text("4.5")
}

// ✅ Custom actions for complex interactions
Box(
    modifier = Modifier.semantics {
        contentDescription = "Restaurant: $name"
        customActions = listOf(
            CustomAccessibilityAction("Add to favorites") { onFavorite(); true },
            CustomAccessibilityAction("Share") { onShare(); true },
            CustomAccessibilityAction("Get directions") { onDirections(); true },
        )
    }
)
```

### Heading Semantics

```kotlin
// ✅ Mark section titles as headings for screen reader navigation
Text(
    text = "Section Title",
    modifier = Modifier.semantics { heading() },
    style = MaterialTheme.typography.headlineMedium,
)
```

### Traversal Order

```kotlin
// ✅ Control focus traversal order for logical reading flow
Column {
    Text(
        text = "Important announcement",
        modifier = Modifier.semantics { traversalIndex = 0f },
    )
    Text(
        text = "Supporting details",
        modifier = Modifier.semantics { traversalIndex = 1f },
    )
    Button(
        onClick = onAction,
        modifier = Modifier.semantics { traversalIndex = 2f },
    ) {
        Text("Take Action")
    }
}
```

### Focus Management

```kotlin
// ✅ Request focus programmatically (e.g., after navigation or error)
val focusRequester = remember { FocusRequester() }

LaunchedEffect(errorMessage) {
    if (errorMessage != null) {
        focusRequester.requestFocus()
    }
}

Text(
    text = errorMessage ?: "",
    modifier = Modifier
        .focusRequester(focusRequester)
        .semantics { liveRegion = LiveRegionMode.Polite },
    color = MaterialTheme.colorScheme.error,
)
```

### TalkBack Testing Guide

1. Enable TalkBack: Settings > Accessibility > TalkBack
2. Navigate by swiping right (next element) and left (previous)
3. Verify every interactive element is announced with a meaningful description
4. Verify headings are announced as headings (swipe up/down to navigate by headings)
5. Verify custom actions are available (swipe up then right to access actions menu)
6. Verify no information is conveyed by color alone
7. Verify focus order is logical (left-to-right, top-to-bottom)

### Contrast Ratios

Follow WCAG 2.1 AA minimum contrast ratios:

| Element | Minimum Ratio |
|---------|--------------|
| Normal text (< 18sp) | 4.5:1 |
| Large text (>= 18sp or >= 14sp bold) | 3:1 |
| UI components (icons, borders) | 3:1 |

Material Design 3 color roles are designed to meet these ratios when used correctly. Always use `on*` colors on their corresponding surface (e.g., `onPrimary` on `primary`).

### Dynamic Text Sizing

```kotlin
// ✅ Correct: Use sp for text sizes — respects user font size preferences
Text(
    text = "Readable text",
    fontSize = 16.sp, // Scales with user preferences
)

// ❌ Incorrect: Using dp for text sizes — ignores user preferences
Text(
    text = "Fixed text",
    fontSize = 16.dp.value.sp, // Don't convert dp to sp
)
```

---

## 🌐 Localization

### String Resources

```xml
<!-- res/values/strings.xml -->
<resources>
    <string name="home_title">Home</string>
    <string name="user_profile_title">Profile</string>
    <string name="error_generic">Something went wrong. Please try again.</string>
    <string name="restaurant_rating">Rating: %1$.1f</string>
    <plurals name="review_count">
        <item quantity="one">%d review</item>
        <item quantity="other">%d reviews</item>
    </plurals>
</resources>
```

### Usage in Compose

```kotlin
Text(stringResource(R.string.home_title))
Text(stringResource(R.string.restaurant_rating, 4.5f))
Text(pluralStringResource(R.plurals.review_count, count, count))
```

### Key Naming Convention

Format: `{module}_{screen}_{element}_{type}`
- `home_search_hint` -- Search bar hint on home screen
- `profile_edit_button` -- Edit button on profile screen
- `error_network_message` -- Network error message
- `common_cancel` -- Cancel button (shared across screens)

### RTL Support

Support right-to-left languages (Arabic, Hebrew, etc.) by using directional-aware modifiers and layout properties.

```kotlin
// ✅ Correct: Use start/end instead of left/right
Row(
    modifier = Modifier.padding(start = 16.dp, end = 8.dp),
) {
    Text("This respects RTL layout direction")
}

// ❌ Incorrect: Using absolute left/right
Row(
    modifier = Modifier.padding(left = 16.dp, right = 8.dp), // Won't mirror in RTL
) {
    Text("This breaks in RTL languages")
}

// ✅ Correct: Use Arrangement.Start/End
Row(
    horizontalArrangement = Arrangement.Start, // Mirrors correctly in RTL
) {
    // Content
}

// ✅ Correct: Use AutoMirrored icons for directional icons
Icon(
    imageVector = Icons.AutoMirrored.Filled.ArrowBack, // Mirrors in RTL
    contentDescription = stringResource(R.string.go_back),
)

// ❌ Incorrect: Non-mirrored directional icons
Icon(
    imageVector = Icons.Default.ArrowBack, // Won't mirror in RTL
    contentDescription = stringResource(R.string.go_back),
)
```

### Date, Time, and Number Formatting

```kotlin
// ✅ Correct: Use locale-aware formatting
import java.time.format.DateTimeFormatter
import java.time.format.FormatStyle

val dateFormatter = DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM)
val timeFormatter = DateTimeFormatter.ofLocalizedTime(FormatStyle.SHORT)

Text(localDate.format(dateFormatter)) // "Feb 25, 2026" (en-US) or "25 feb 2026" (es)
Text(localTime.format(timeFormatter)) // "3:30 PM" (en-US) or "15:30" (de)

// ✅ Correct: Locale-aware number formatting
import java.text.NumberFormat
import java.util.Locale

val currencyFormatter = NumberFormat.getCurrencyInstance(Locale.getDefault())
Text(currencyFormatter.format(12.99)) // "$12.99" (en-US) or "12,99 EUR" (de)

val percentFormatter = NumberFormat.getPercentInstance(Locale.getDefault())
Text(percentFormatter.format(0.85)) // "85%" (en-US) or "85 %" (fr)

// ❌ Incorrect: Hardcoded format strings
Text("$${price}") // Assumes USD and English formatting
Text("${date.dayOfMonth}/${date.monthValue}/${date.year}") // Assumes US date format
```

### Testing with Pseudo-Locales

Enable pseudo-locales in developer options to catch localization issues early:

1. Go to Settings > System > Developer Options > Languages
2. Enable "Pseudo-Locale [en-XA]" for accented English (tests string boundaries)
3. Enable "Pseudo-Locale [ar-XB]" for RTL testing (tests layout mirroring)

Things to verify:
- No truncated text (pseudo-locale adds ~30% length)
- All strings are localized (unlocalizable strings appear without accents)
- RTL layout mirrors correctly
- No hardcoded directional values

---

## 📱 Responsive Layout

### Adaptive Layouts (Window Size)

ARC Labs follows the official `adaptive` skill (android/skills plugin):

- **Navigation area**: `NavigationSuiteScaffold` switches between bottom bar and navigation rail by window size — never branch on size classes by hand for this. See [Navigation 3](../Architecture/navigation-3.md#-top-level-tabs-with-navigationsuitescaffold).
- **Multi-pane**: list-detail and supporting-pane layouts are Navigation 3 **scene strategies** (`ListDetailSceneStrategy`, `SupportingPaneSceneStrategy` from `adaptive-navigation3`), not hand-written `Row`s and not `ListDetailPaneScaffold` directly. See [Navigation 3 → Adaptive Layouts](../Architecture/navigation-3.md#-adaptive-layouts-list-detail).
- **Lists**: grids use adaptive columns (`GridCells.Adaptive(minSize)` / `StaggeredGridCells.Adaptive(minSize)`).
- **Window info**: when a screen genuinely needs the window size, read it from Material 3 adaptive (the official recipes use `currentWindowAdaptiveInfoV2()`; verify the current entry point for material3-adaptive 1.3.0). The legacy `calculateWindowSizeClass(activity)` from `material3-window-size-class` is not used in new code.
- **Verify** with screenshot tests at phone, foldable, tablet and desktop sizes (`@PreviewTest` + multi-device `@Preview`s).

### Foldable Device Support

> **Verify before use:** `LocalFoldingFeatures` below is illustrative, not a Jetpack API. Read posture/hinge information from Material 3 adaptive window info (`windowPosture`) or Jetpack WindowManager, confirming the API with the official `adaptive` skill.

```kotlin
@Composable
fun FoldAwareLayout() {
    val foldingFeatures = LocalFoldingFeatures.current

    val foldState = foldingFeatures.firstOrNull()

    when {
        foldState != null && foldState.state == FoldingFeature.State.HALF_OPENED -> {
            // Table-top mode: content above fold, controls below
            TwoHingeLayout(foldState)
        }
        else -> {
            // Normal layout
            StandardLayout()
        }
    }
}
```

---

## ✨ Animations

### Material Motion Patterns

M3 defines motion patterns for transitions. Use `AnimatedVisibility` and `animateContentSize` for most cases.

```kotlin
// ✅ AnimatedVisibility for enter/exit transitions
AnimatedVisibility(
    visible = isVisible,
    enter = fadeIn() + slideInVertically(),
    exit = fadeOut() + slideOutVertically(),
) {
    Card {
        Text("Animated content")
    }
}

// ✅ animateContentSize for layout changes
Column(
    modifier = Modifier.animateContentSize(
        animationSpec = spring(
            dampingRatio = Spring.DampingRatioMediumBouncy,
            stiffness = Spring.StiffnessLow,
        ),
    ),
) {
    Text("Title")
    if (isExpanded) {
        Text("Expanded details that animate in smoothly")
    }
}
```

### Shared Element Transitions

```kotlin
// ✅ Shared element transitions between list and detail
SharedTransitionLayout {
    AnimatedContent(targetState = selectedItem) { item ->
        if (item == null) {
            LazyColumn {
                items(restaurants) { restaurant ->
                    Row(
                        modifier = Modifier
                            .sharedElement(
                                state = rememberSharedContentState(key = "restaurant-${restaurant.id}"),
                                animatedVisibilityScope = this@AnimatedContent,
                            )
                            .clickable { onSelect(restaurant) },
                    ) {
                        Text(restaurant.name)
                    }
                }
            }
        } else {
            RestaurantDetail(
                restaurant = item,
                modifier = Modifier.sharedElement(
                    state = rememberSharedContentState(key = "restaurant-${item.id}"),
                    animatedVisibilityScope = this@AnimatedContent,
                ),
            )
        }
    }
}
```

### When to Use Animations

| Scenario | Use Animation | Type |
|----------|:------------:|------|
| Item appearing/disappearing | Yes | `AnimatedVisibility` |
| Content size changing | Yes | `animateContentSize` |
| State change (color, size) | Yes | `animate*AsState` |
| Screen transitions | Yes | Navigation animations |
| Loading indicators | Yes | Indeterminate progress |
| Decorative/gratuitous motion | No | None |
| User has reduced motion enabled | Respect | Check `LocalReducedMotion` |

```kotlin
// ✅ Respect reduced motion preferences
val reducedMotion = LocalReducedMotion.current
val animationSpec = if (reducedMotion) {
    snap<Float>()
} else {
    tween<Float>(durationMillis = 300)
}
```

---

## 📐 Layout Guidelines

### Scaffold Patterns

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ScreenScaffold(
    title: String,
    onBack: (() -> Unit)? = null,
    floatingActionButton: @Composable () -> Unit = {},
    content: @Composable (PaddingValues) -> Unit,
) {
    val scrollBehavior = TopAppBarDefaults.pinnedScrollBehavior()

    Scaffold(
        modifier = Modifier.nestedScroll(scrollBehavior.nestedScrollConnection),
        topBar = {
            CenterAlignedTopAppBar(
                title = { Text(title) },
                navigationIcon = {
                    if (onBack != null) {
                        IconButton(onClick = onBack) {
                            Icon(
                                Icons.AutoMirrored.Filled.ArrowBack,
                                contentDescription = stringResource(R.string.go_back),
                            )
                        }
                    }
                },
                scrollBehavior = scrollBehavior,
            )
        },
        floatingActionButton = floatingActionButton,
        content = content,
    )
}
```

### Spacing System (4dp Grid)

All spacing should follow the 4dp grid system for visual consistency.

| Token | Value | Usage |
|-------|-------|-------|
| `xxs` | 2.dp | Tight spacing between related inline elements |
| `xs` | 4.dp | Minimum spacing, icon gaps |
| `sm` | 8.dp | Between list items, small component padding |
| `md` | 12.dp | Between related groups |
| `lg` | 16.dp | Screen horizontal padding, section spacing |
| `xl` | 24.dp | Between major sections |
| `xxl` | 32.dp | Large separations, dialog padding |

```kotlin
// ✅ Correct: Define spacing tokens
object Spacing {
    val xxs = 2.dp
    val xs = 4.dp
    val sm = 8.dp
    val md = 12.dp
    val lg = 16.dp
    val xl = 24.dp
    val xxl = 32.dp
}

// ✅ Correct: Use consistent spacing
LazyColumn(
    contentPadding = PaddingValues(horizontal = Spacing.lg, vertical = Spacing.sm),
    verticalArrangement = Arrangement.spacedBy(Spacing.sm),
) {
    items(restaurants) { restaurant ->
        RestaurantCard(restaurant)
    }
}
```

### Edge-to-Edge (Safe Area Handling)

```kotlin
// ✅ Correct: Enable edge-to-edge in Activity
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            AppTheme {
                AppRoot()
            }
        }
    }
}

// ✅ Correct: Scaffold handles insets automatically
Scaffold(
    topBar = { /* TopAppBar handles status bar insets */ },
    bottomBar = { /* BottomBar handles navigation bar insets */ },
) { innerPadding ->
    // innerPadding includes system bar insets
    LazyColumn(
        contentPadding = innerPadding,
    ) {
        // Content
    }
}

// ✅ Correct: Manual inset handling when not using Scaffold
Box(
    modifier = Modifier
        .fillMaxSize()
        .windowInsetsPadding(WindowInsets.systemBars),
) {
    // Content respects system bars
}
```

---

## ❌ Common Mistakes

### 1. Hardcoded Colors

```kotlin
// ❌ Incorrect: Hardcoded color values
Text(
    text = "Hello",
    color = Color(0xFF1976D2), // Won't adapt to dark theme
)

Surface(color = Color.White) { // Invisible in light theme, wrong in dark
    Text("Content", color = Color.Black)
}

// ✅ Correct: Use MaterialTheme color roles
Text(
    text = "Hello",
    color = MaterialTheme.colorScheme.primary,
)

Surface(color = MaterialTheme.colorScheme.surface) {
    Text("Content", color = MaterialTheme.colorScheme.onSurface)
}
```

### 2. Missing Content Descriptions

```kotlin
// ❌ Incorrect: No content description on interactive element
IconButton(onClick = onDelete) {
    Icon(Icons.Default.Delete, contentDescription = null) // TalkBack cannot announce this
}

Image(
    painter = painterResource(R.drawable.profile),
    contentDescription = null, // Meaningful image without description
)

// ✅ Correct: Provide meaningful descriptions
IconButton(onClick = onDelete) {
    Icon(Icons.Default.Delete, contentDescription = stringResource(R.string.delete_item))
}

Image(
    painter = painterResource(R.drawable.profile),
    contentDescription = stringResource(R.string.profile_photo, userName),
)
```

### 3. Touch Targets Too Small

```kotlin
// ❌ Incorrect: Small custom button without minimum size
Box(
    modifier = Modifier
        .size(24.dp) // Too small for comfortable touch
        .clickable(onClick = onTap),
) {
    Icon(Icons.Default.Close, contentDescription = stringResource(R.string.close))
}

// ✅ Correct: Ensure 48dp minimum touch target
Box(
    modifier = Modifier
        .sizeIn(minWidth = 48.dp, minHeight = 48.dp)
        .clickable(onClick = onTap),
    contentAlignment = Alignment.Center,
) {
    Icon(
        Icons.Default.Close,
        contentDescription = stringResource(R.string.close),
        modifier = Modifier.size(24.dp),
    )
}
```

### 4. Ignoring RTL Layout

```kotlin
// ❌ Incorrect: Absolute positioning
Row(modifier = Modifier.padding(left = 16.dp, right = 8.dp)) {
    Icon(
        imageVector = Icons.Default.ArrowBack, // Non-mirroring icon
        contentDescription = "Back",
    )
    Text("Details")
}

// ✅ Correct: Directional-aware layout
Row(modifier = Modifier.padding(start = 16.dp, end = 8.dp)) {
    Icon(
        imageVector = Icons.AutoMirrored.Filled.ArrowBack, // Mirrors in RTL
        contentDescription = stringResource(R.string.go_back),
    )
    Text(stringResource(R.string.details))
}
```

### 5. Using Fixed dp for Text Sizes

```kotlin
// ❌ Incorrect: Text size in dp ignores accessibility settings
Text(
    text = "Important info",
    style = TextStyle(fontSize = 14.dp.value.sp), // Workaround that defeats the purpose
)

// Also incorrect: not using sp at all
Canvas(modifier = Modifier.size(200.dp)) {
    drawText(
        textMeasurer = textMeasurer,
        text = "Label",
        style = TextStyle(fontSize = 14.dp.toSp()), // dp-based text
    )
}

// ✅ Correct: Use sp for all text sizes
Text(
    text = "Important info",
    style = MaterialTheme.typography.bodyMedium, // Uses sp internally
)

// ✅ Correct: Use Material typography styles consistently
Text(
    text = "Custom sized text",
    fontSize = 14.sp, // sp respects user text size preferences
)
```

### 6. Hardcoded Strings

```kotlin
// ❌ Incorrect: Hardcoded user-facing strings
Text("No results found")
Button(onClick = onRetry) { Text("Retry") }

// ✅ Correct: String resources for all user-facing text
Text(stringResource(R.string.no_results))
Button(onClick = onRetry) { Text(stringResource(R.string.common_retry)) }
```

---

## ✅ UI Checklist

- [ ] `MaterialTheme` used consistently (no hardcoded colors)
- [ ] Both light and dark themes tested
- [ ] Brand color scheme used (dynamic color off); Expressive components preferred over custom ones
- [ ] All user-facing strings from `R.string` resources
- [ ] Content descriptions on all interactive and meaningful elements
- [ ] Touch targets >= 48dp on all interactive elements
- [ ] `@Preview` for every composable (light and dark), fed from centralized sample data, covered by screenshot tests
- [ ] Responsive to different window sizes (`NavigationSuiteScaffold`, Navigation 3 list-detail scenes)
- [ ] RTL layout tested (start/end instead of left/right)
- [ ] Typography uses `sp` units (respects user font size)
- [ ] Heading semantics applied to section titles
- [ ] Animations respect reduced motion preferences
- [ ] Edge-to-edge enabled with proper inset handling
- [ ] Spacing follows the 4dp grid system
- [ ] Snackbars use `SnackbarHost` in `Scaffold`

---

## 📚 Further Reading

- [Material Design 3](https://m3.material.io/)
- [Material Design 3 for Compose](https://developer.android.com/develop/ui/compose/designsystems/material3)
- [Compose Accessibility](https://developer.android.com/develop/ui/compose/accessibility)
- [Compose Layouts](https://developer.android.com/develop/ui/compose/layouts)
- [Window Size Classes](https://developer.android.com/develop/ui/views/layout/responsive/use-window-size-classes)
- [Compose Performance](./compose-performance.md)
- [Code Style](./code-style.md)
- [Testing](./testing.md)

---

**Remember**: Great UI is invisible -- users notice when it's bad, not when it's good. Design with care.
