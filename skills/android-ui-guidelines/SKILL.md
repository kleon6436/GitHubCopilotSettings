---
name: android-ui-guidelines
description: 'Android UI guidelines. Use to review and apply Material Design 3 (Expressive), Jetpack Compose, Dynamic Color, Edge-to-Edge, WindowSizeClass, Predictive Back, Large Screen support, TalkBack accessibility, and touch/gesture handling. Use when: designing or implementing Android app UI with Jetpack Compose or View system; applying Material 3; reviewing Android UI code.'
argument-hint: 'Topic to check (e.g. Material / Compose / Large Screen — optional)'
---

# Android UI Guidelines

## Overview

This skill defines UI conventions for Android apps.
It covers rules for building experiences that span Phone / Foldable / Tablet / ChromeOS, centered on **Material Design 3 (Material You / Expressive)** and **Jetpack Compose**.

References:
- [Material Design 3](https://m3.material.io/)
- [Android Developers - UI](https://developer.android.com/develop/ui)
- [Jetpack Compose](https://developer.android.com/jetpack/compose)
- [Large Screen guidelines](https://developer.android.com/guide/topics/large-screens)

---

## 1. Core Principles

- **Prefer Jetpack Compose.** New screens use Compose; migrate existing View-based code incrementally.
- **Follow Material Design 3 (M3).** Customize `MaterialTheme` rather than building a proprietary theme.
- **Respect Dynamic Color.** On Android 12+, use colors derived from the user's wallpaper.
- **Default to Edge-to-Edge.** Android 15 (API 35)+ enforces Edge-to-Edge.
- **Support all screen sizes.** Phone / Foldable (folded & unfolded) / Tablet / ChromeOS.
- **Ensure TalkBack operability.** Provide content descriptions for all interactive elements.

---

## 2. Recommended Tech Stack

| Category | Recommendation |
|---|---|
| UI Framework | **Jetpack Compose** (`androidx.compose.*`) |
| Design System | **Material 3** (`androidx.compose.material3`) |
| Navigation | **Navigation Compose** (Type-safe Navigation) |
| Image Loading | **Coil** |
| Async | **Coroutines** + **Flow** |
| Adaptive | **Material 3 Adaptive** (`androidx.compose.material3.adaptive`) |
| Animation | **Compose Animation** + **Motion** |

---

## 3. Theme (Material 3)

### Basic Theme

```kotlin
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit,
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        darkTheme -> darkColorScheme(/* brand colors */)
        else -> lightColorScheme(/* brand colors */)
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = AppTypography,
        shapes = AppShapes,
        content = content,
    )
}
```

### Color Roles (M3)

| Role | Usage |
|---|---|
| `primary` / `onPrimary` | Primary action |
| `secondary` / `onSecondary` | Secondary action |
| `tertiary` / `onTertiary` | Contrast accent |
| `surface` / `onSurface` | Card & sheet background |
| `surfaceContainer*` | Layered container background |
| `error` / `onError` | Error & destructive action |

- Avoid hardcoded colors (`Color(0xFF...)`) and use `MaterialTheme.colorScheme.primary` instead.

---

## 4. Typography (Type Scale)

Material 3 type scale:

| Style | Usage |
|---|---|
| `displayLarge` / `displayMedium` / `displaySmall` | Hero headings |
| `headlineLarge` / `headlineMedium` / `headlineSmall` | Screen headings |
| `titleLarge` / `titleMedium` / `titleSmall` | Section titles, dialog headings |
| `bodyLarge` / `bodyMedium` / `bodySmall` | Body text |
| `labelLarge` / `labelMedium` / `labelSmall` | Button & tab labels |

```kotlin
Text(
    text = "Title",
    style = MaterialTheme.typography.headlineMedium,
)
```

- Prefer variable fonts (Roboto Flex / Google Sans). For Japanese, use Noto Sans CJK JP.
- Follow the user's font scale setting (Settings → Display → Font size). Use `sp` units; do not use `dp` for text.

---

## 5. Layout & Spacing

- Use an **8dp grid** as the baseline. Stick to `4, 8, 12, 16, 24, 32, 40, 48` dp.
- Screen edge margins: **16dp** (Compact) / **24dp** (Medium and above).
- Tokenize spacing:

```kotlin
object Spacing {
    val xs = 4.dp
    val sm = 8.dp
    val md = 16.dp
    val lg = 24.dp
    val xl = 32.dp
}
```

- Minimum touch target: **48×48dp** (Material recommendation, WCAG 2.5.5 compliant).

---

## 6. Components (M3)

### Key Components

| Usage | Component |
|---|---|
| Main action | `Button` (Filled) / `FilledTonalButton` / `OutlinedButton` / `TextButton` |
| FAB | `FloatingActionButton` / `ExtendedFloatingActionButton` |
| Top bar | `TopAppBar` / `CenterAlignedTopAppBar` / `MediumTopAppBar` / `LargeTopAppBar` |
| Bottom bar | `BottomAppBar` / `NavigationBar` |
| Navigation | `NavigationBar` (≤ 5 items) / `NavigationRail` (Medium+) / `NavigationDrawer` (Expanded) |
| Card | `Card` / `ElevatedCard` / `OutlinedCard` |
| List | `ListItem` |
| Input | `TextField` (Filled) / `OutlinedTextField` |
| Selection | `Checkbox` / `RadioButton` / `Switch` / `Slider` |
| Feedback | `Snackbar` / `LinearProgressIndicator` / `CircularProgressIndicator` |
| Dialog | `AlertDialog` / `DatePickerDialog` / `TimePickerDialog` / `ModalBottomSheet` |

### Button Hierarchy

- **FAB**: For a single primary action per screen only.
- **Filled Button**: Important actions that don't warrant a FAB.
- **Tonal Button**: Medium emphasis.
- **Outlined Button**: Alternative / secondary actions.
- **Text Button**: Non-destructive supplementary actions.

```kotlin
Button(onClick = { /* ... */ }) { Text("Save") }
OutlinedButton(onClick = { /* ... */ }) { Text("Cancel") }
```

---

## 7. Navigation

### Adaptive Navigation

Switch navigation based on `WindowSizeClass`:

| WindowSizeClass | Navigation |
|---|---|
| **Compact** (width < 600dp) | `NavigationBar` (bottom) |
| **Medium** (600 ≤ width < 840dp) | `NavigationRail` (left) |
| **Expanded** (width ≥ 840dp) | `PermanentNavigationDrawer` or `NavigationRail` |

```kotlin
val windowSizeClass = currentWindowAdaptiveInfo().windowSizeClass

NavigationSuiteScaffold(
    navigationSuiteItems = {
        items.forEach { item ->
            item(
                icon = { Icon(item.icon, null) },
                label = { Text(item.label) },
                selected = item.selected,
                onClick = item.onClick,
            )
        }
    },
) {
    // content
}
```

- Use **Type-safe Navigation** in **Navigation Compose** (kotlinx-serialization based).

```kotlin
@Serializable data object Home
@Serializable data class Detail(val id: Long)

NavHost(navController, startDestination = Home) {
    composable<Home> { HomeScreen(onOpenDetail = { navController.navigate(Detail(it)) }) }
    composable<Detail> { backStack ->
        val detail: Detail = backStack.toRoute()
        DetailScreen(id = detail.id)
    }
}
```

### Predictive Back

Support **Predictive Back** on Android 14+:

```kotlin
// AndroidManifest
// android:enableOnBackInvokedCallback="true"

PredictiveBackHandler(enabled = hasUnsavedChanges) { progress ->
    try {
        progress.collect { /* animation progress */ }
        // gesture complete → handle back
    } catch (e: CancellationException) {
        // cancelled
    }
}
```

---

## 8. Edge-to-Edge / Insets

Android 15 (API 35)+ **enforces Edge-to-Edge by default**.

```kotlin
// Activity
override fun onCreate(savedInstanceState: Bundle?) {
    enableEdgeToEdge()
    super.onCreate(savedInstanceState)
    setContent { AppTheme { App() } }
}
```

- Consume **WindowInsets** appropriately:

```kotlin
Scaffold(
    topBar = { TopAppBar(...) }, // insets applied automatically
    contentWindowInsets = WindowInsets.safeDrawing,
) { padding ->
    Column(Modifier.padding(padding)) { ... }
}

// custom element
Box(Modifier.windowInsetsPadding(WindowInsets.systemBars)) { ... }
```

- Draw content behind the status bar and navigation bar, using appropriate insets to keep controls clear.

---

## 9. Large Screen / Foldable Support

- Use **WindowSizeClass** to apply **List-Detail** / **Supporting Pane** / **Feed** patterns throughout the app.

```kotlin
val navigator = rememberListDetailPaneScaffoldNavigator<Item>()

ListDetailPaneScaffold(
    directive = navigator.scaffoldDirective,
    value = navigator.scaffoldValue,
    listPane = { AnimatedPane { ItemList(onSelect = { navigator.navigateTo(ListDetailPaneScaffoldRole.Detail, it) }) } },
    detailPane = {
        AnimatedPane {
            navigator.currentDestination?.content?.let { DetailScreen(it) }
        }
    },
)
```

- Use **`WindowInfoTracker`** to obtain hinge and fold information to handle fold/unfold transitions.
- Test that the layout does not break in landscape, split-screen, or multi-window modes.
- **ChromeOS / DeX** support: account for mouse, keyboard, and resizable windows.

---

## 10. Animation & Motion

- Basic transitions:
  - `AnimatedVisibility` — element enter/exit
  - `AnimatedContent` — content switching
  - `Crossfade` — simple switching
  - `animate*AsState` — value interpolation
- Use **Shared Element Transition** (Compose 1.7+) for hero transitions.

```kotlin
SharedTransitionLayout {
    AnimatedContent(targetState = state) { target ->
        if (target) {
            Image(
                modifier = Modifier.sharedElement(
                    rememberSharedContentState("image"),
                    animatedVisibilityScope = this@AnimatedContent,
                )
            )
        }
    }
}
```

- Use `FastOutSlowInEasing` / `EmphasizedEasing` from Material Motion.
- Duration: **100–400ms**.
- Respect the **Remove Animations** accessibility setting:

```kotlin
val reduceMotion = LocalAccessibilityManager.current?.let { /* check */ } ?: false
```

---

## 11. Accessibility (TalkBack)

- Set `contentDescription` on images and icon buttons. Use `null` for decorative images.

```kotlin
Icon(
    imageVector = Icons.Default.Delete,
    contentDescription = "Delete",
)

IconButton(onClick = { /* ... */ }) {
    Icon(Icons.Default.Share, contentDescription = "Share")
}
```

- Explicit semantics:

```kotlin
Modifier.semantics {
    contentDescription = "Add to favorites"
    role = Role.Button
    stateDescription = if (isFavorite) "Added" else "Not added"
}
```

- Contrast ratio **WCAG AA** (4.5:1 / 3:1).
- Touch targets **48×48dp** (enforced with `Modifier.minimumInteractiveComponentSize()`).
- Layout must not break at font scale **200%**.
- Do not convey information by color alone.
- See `skills/ui-accessibility/SKILL.md` for details.

---

## 12. Lists & Performance

- Use **`LazyColumn` / `LazyRow` / `LazyVerticalGrid`**. Do not render lists with `Column` + `verticalScroll`.
- Specify stable IDs with the `key` parameter to avoid unnecessary recompositions.

```kotlin
LazyColumn {
    items(items, key = { it.id }) { item ->
        ItemRow(item)
    }
}
```

- Load images lazily with **Coil** + `crossfade`.
- Optimize recomposition: use `remember` / `derivedStateOf` appropriately.
- Use `@Stable` / `@Immutable` annotations to give Compose hints for optimization.

---

## 13. Input & Gestures

- **IME (keyboard) handling**: Use `Modifier.imePadding()` to avoid keyboard overlap. Use `WindowInsets.ime`.
- Text input: configure `KeyboardOptions` (`keyboardType` / `imeAction` / `autoCorrect`) appropriately.
- Gestures: use `Modifier.pointerInput` with `detectTapGestures` / `detectDragGestures`.
- Account for both touch and mouse (ChromeOS). Use `Modifier.hoverable` for hover support.

---

## 14. Icons

- Use **Material Symbols** (`material-icons-extended`).
- Provide custom icons as Vector Drawables (converted from SVG).
- Base icon sizes: **20 / 24 / 32 / 40dp**.
- Provide app icons as **Adaptive Icons** (foreground / background layers) + **Monochrome** (for Themed Icons).

```xml
<!-- res/mipmap-anydpi-v26/ic_launcher.xml -->
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
    <monochrome android:drawable="@drawable/ic_launcher_monochrome" />
</adaptive-icon>
```

---

## 15. Notifications & System Integration

- Categorize notifications into **Notification Channels** (required for Android 8+).
- Request the **POST_NOTIFICATIONS** permission (Android 13+) appropriately.
- Verify rich notification behavior (images, actions, replies) and expansion in the notification tray.
- Expose frequently used features via **App Shortcuts** / **Widgets** (Glance API).

---

## 16. Testing

- Validate screen logic with **Compose UI Test** (`composeTestRule`).
- Run automated accessibility checks with **Accessibility Scanner**.
- Use **Espresso** + **UIAutomator** for E2E testing.
- Measure launch and scroll performance with **Baseline Profile** / **Macrobenchmark**.

---

## 17. Design Previews

Cover various configurations with `@Preview`:

```kotlin
@Preview(name = "Light", showBackground = true)
@Preview(name = "Dark", uiMode = UI_MODE_NIGHT_YES, showBackground = true)
@Preview(name = "Large Font", fontScale = 2.0f)
@Preview(name = "Tablet", device = "spec:width=1280dp,height=800dp,dpi=240")
@Composable
fun ScreenPreview() {
    AppTheme { HomeScreen() }
}
```

---

## Related Skills

- General accessibility: `skills/ui-accessibility/SKILL.md`
- UI review: `skills/ui-review-checklist/SKILL.md`
