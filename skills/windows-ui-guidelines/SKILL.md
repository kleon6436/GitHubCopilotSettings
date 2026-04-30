---
name: windows-ui-guidelines
description: 'Windows UI guidelines. Use to review and apply the Fluent Design System, WinUI 3 / WPF, Mica / Acrylic materials, title bar integration, Narrator accessibility, touch/mouse/pen input, and Windows 11 design principles. Use when: designing or implementing Windows desktop UI with WinUI 3, WPF, or MAUI; applying Fluent Design; reviewing Windows app UI.'
argument-hint: 'Item to review (Fluent / Mica / accessibility, etc. — optional)'
---

# Windows UI Guidelines

## Overview

This skill defines UI conventions for Windows 11 and later desktop apps.
It summarizes the rules for delivering a consistent experience with WinUI 3 / WPF / .NET MAUI, following Microsoft’s **Fluent Design System**.

References:
- [Fluent Design System](https://learn.microsoft.com/windows/apps/design/)
- [Windows 11 Design principles](https://learn.microsoft.com/windows/apps/design/signature-experiences/design-principles)
- [WinUI 3 documentation](https://learn.microsoft.com/windows/apps/winui/winui3/)

---

## 1. Fluent Design Core Principles

- **Effortless** — Reduce friction in achieving goals. Use standard controls and minimize custom implementations.
- **Calm** — Suppress visual noise and let content take center stage. Avoid excessive animations and decorations.
- **Personal** — Respect the user’s theme (Light / Dark / accent color) and system settings.
- **Familiar** — Follow Windows 11 standard patterns (NavigationView, CommandBar, context menus).
- **Complete** — Support all input methods: touch, mouse, keyboard, pen, and gamepad.

---

## 2. Recommended Technology Stack

| Use Case | Recommended | Notes |
|---|---|---|
| New desktop app | **WinUI 3** + Windows App SDK | Latest Fluent, Mica, Windows 11 integration |
| Existing WPF continuation | **WPF** + ModernWpf / WPF UI | Apply Fluent theme via external libraries |
| Cross-platform | **.NET MAUI** | Windows / macOS / iOS / Android |
| Web-based | **WebView2** host + Fluent UI Web | For hybrid apps |

---

## 3. Materials (Mica / Acrylic)

Windows 11 uses background materials to express hierarchy and depth.

| Material | Usage | Implementation |
|---|---|---|
| **Mica** | Window background (long-lived areas) | `SystemBackdrop="Mica"` |
| **Mica Alt** | Tabbed windows | `SystemBackdrop="MicaAlt"` |
| **Acrylic** | Transient UI (flyouts, menus, dialogs) | `AcrylicBrush` |
| **Smoke** | Modal background | `SystemBackdrop="Smoke"` |

```xml
<!-- WinUI 3: Apply Mica to a window -->
<Window x:Class="MyApp.MainWindow"
        xmlns="...">
    <Window.SystemBackdrop>
        <MicaBackdrop Kind="Base" />
    </Window.SystemBackdrop>
</Window>
```

- **Do not override Mica's opacity or color.** Respect the user's desktop wallpaper and accent color.
- Do not use Acrylic as the background for content areas (affects performance and readability). Limit it to flyouts and task panes.

---

## 4. Title Bar Integration

In Windows 11, integrate the title bar with app content (`ExtendsContentIntoTitleBar`).

```csharp
// WinUI 3
this.ExtendsContentIntoTitleBar = true;
this.SetTitleBar(AppTitleBar);
```

- The title bar can contain an app icon, title, search box, profile, etc.
- Clearly separate the drag region (`IsHitTestVisible="False"`) from interactive regions.
- Leave the Caption Buttons (minimize/maximize/close) to the OS — do not reimplement their position or size.

---

## 5. Navigation Structure

### NavigationView (Recommended)

Use **`NavigationView`** for top-level navigation.

```xml
<NavigationView PaneDisplayMode="Auto"
                IsBackButtonVisible="Auto">
    <NavigationView.MenuItems>
        <NavigationViewItem Content="Home" Icon="Home" Tag="home" />
        <NavigationViewItem Content="Library" Icon="Library" Tag="library" />
    </NavigationView.MenuItems>
    <Frame x:Name="ContentFrame" />
</NavigationView>
```

- `PaneDisplayMode="Auto"` automatically switches between Left / Top / LeftCompact / LeftMinimal based on window width.
- Recommend Top for 5 items or fewer, Left for 6 or more.
- Pin Settings to the footer with **`IsSettingsVisible="True"`**.

### TabView (Document-style Apps)

Use **`TabView`** (browser/editor style) when working with multiple documents.

---

## 6. Command Surfaces (Command Bar)

| Surface | Usage |
|---|---|
| **CommandBar** | Primary actions at the top or bottom of a page |
| **CommandBarFlyout** | Context menus and selection-triggered actions |
| **MenuFlyout** | Right-click and overflow menus |

```xml
<CommandBar DefaultLabelPosition="Right">
    <AppBarButton Icon="Add" Label="New" />
    <AppBarButton Icon="Save" Label="Save" />
    <AppBarSeparator />
    <AppBarButton Icon="Share" Label="Share" />
</CommandBar>
```

- Order by importance: **Primary Commands** (left or top), secondary in Secondary Commands (overflow).
- Use [Segoe Fluent Icons](https://learn.microsoft.com/windows/apps/design/style/segoe-fluent-icons-font) for icons (Windows 11).

---

## 7. Controls

- Prefer standard WinUI controls (`Button` / `ToggleSwitch` / `Slider` / `ComboBox` / `CalendarDatePicker`, etc.).
- Button styles:
  - **AccentButtonStyle**: Only one primary action per screen
  - **DefaultButton**: Normal operations
  - **SubtleButtonStyle**: Supplementary operations
- Destructive actions use `Foreground="{ThemeResource SystemFillColorCriticalBrush}"` (red tones).

```xml
<Button Content="Save"
        Style="{ThemeResource AccentButtonStyle}" />
```

---

## 8. Typography

Use Windows 11’s **Segoe UI Variable** as the base font.

| Style | Size | Usage |
|---|---|---|
| Caption | 12 | Supplementary information |
| Body | 14 | Body text |
| Body Strong | 14 Semibold | Emphasized body text |
| Body Large | 18 | Body text (emphasized) |
| Subtitle | 20 Semibold | Sub-heading |
| Title | 28 Semibold | Section heading |
| Title Large | 40 Semibold | Page heading |
| Display | 68 Semibold | Hero heading |

```xml
<TextBlock Text="Heading" Style="{ThemeResource TitleTextBlockStyle}" />
<TextBlock Text="Body" Style="{ThemeResource BodyTextBlockStyle}" />
```

- Prioritize Segoe UI Variable for custom fonts; fall back to Yu Gothic UI for Japanese.
- Verify that the layout does not break at the user’s “Text Size” setting (100%–225%).

---

## 9. Color & Themes

- Use **theme resources** (`{ThemeResource ...}`) and avoid hardcoded colors.
  - `SystemAccentColor` — system accent
  - `TextFillColorPrimaryBrush` / `TextFillColorSecondaryBrush` — text
  - `SolidBackgroundFillColorBaseBrush` — base background
- Test against all three themes: Light, Dark, and High Contrast.
- Respect the accent color from system settings (do not override the user’s selection).

---

## 10. Spacing & Grid

- Base unit is a **4px grid** (4, 8, 12, 16, 20, 24, 32, 40).
- Page edge padding: **24px or more** on each side.
- Control spacing: **12px** vertical, **8px** horizontal as a baseline.

```xml
<Thickness x:Key="PageMargin">24,24,24,24</Thickness>
<StackPanel Spacing="12" />
```

---

## 11. Input Support

Windows natively supports multiple input methods.

| Input | Considerations |
|---|---|
| **Mouse** | Hover states, right-click support |
| **Keyboard** | Tab order, accelerators (Alt+Key), shortcuts (Ctrl+S, etc.) |
| **Touch** | Minimum target **40×40 pixels** (48×48 recommended) |
| **Pen** | Ink input support (`InkCanvas`), hover (Windows Ink) |
| **Gamepad** | For Xbox Game Bar / Xbox app. XY focus navigation |

```xml
<!-- Keyboard accelerator -->
<Button Content="Save">
    <Button.KeyboardAccelerators>
        <KeyboardAccelerator Key="S" Modifiers="Control" />
    </Button.KeyboardAccelerators>
</Button>
```

- Use `AccessKey="S"` to support Alt navigation.

---

## 12. Animations & Motion

- Use **Connected Animation** to smoothly transition elements.

```csharp
ConnectedAnimationService.GetForCurrentView()
    .PrepareToAnimate("forwardAnimation", sourceElement);
```

- Use the standard easing `FluentAnimations.Standard`.
- Motion duration is **150–300ms** as a baseline.
- Skip animations when the system setting "Show animations" is OFF.

```csharp
if (UISettings.AnimationsEnabled) { /* animate */ }
```

---

## 13. Accessibility

- Make every screen operable with **Narrator**.
- Set `AutomationProperties.Name` / `AutomationProperties.HelpText` on all interactive elements.

```xml
<Button Icon="Delete"
        AutomationProperties.Name="Delete"
        ToolTipService.ToolTip="Delete" />
```

- Contrast ratio must meet **WCAG 2.2 AA or above** (4.5:1 for normal text, 3:1 for large text).
- All UI must be visible in the **High Contrast theme**. Use `SystemColor*Brush`.
- Tab focus follows logical order; `TabIndex` can be used for explicit control.
- Do not hide the automatic focus ring (XAML default).
- For details, see `skills/ui-accessibility/SKILL.md`.

---

## 14. Responsive Layout

- Use **Adaptive Triggers** to switch layouts based on window width.

```xml
<VisualStateManager.VisualStateGroups>
    <VisualStateGroup>
        <VisualState x:Name="Narrow">
            <VisualState.StateTriggers>
                <AdaptiveTrigger MinWindowWidth="0" />
            </VisualState.StateTriggers>
        </VisualState>
        <VisualState x:Name="Wide">
            <VisualState.StateTriggers>
                <AdaptiveTrigger MinWindowWidth="720" />
            </VisualState.StateTriggers>
        </VisualState>
    </VisualStateGroup>
</VisualStateManager.VisualStateGroups>
```

- Breakpoint guidelines: **640 / 1008 / 1366 / 1920 px**.
- Minimum window size: at least **500px** wide and **320px** tall.
- Also consider tablet mode and portrait orientation.

---

## 15. Dialogs & Teaching Tip

- Use **`ContentDialog`** for modal confirmations.

```csharp
var dialog = new ContentDialog
{
    Title = "Delete?",
    Content = "This action cannot be undone.",
    PrimaryButtonText = "Delete",
    CloseButtonText = "Cancel",
    DefaultButton = ContentDialogButton.Close,
    XamlRoot = this.XamlRoot
};
```

- Use **`TeachingTip`** for new feature announcements.
- Use **Windows App Notification** (formerly Toast Notification) for toast notifications.

---

## 16. Icons

- Use **Segoe Fluent Icons** for system icons (Windows 11).
- Prefer SVG for custom icons (`PathIcon`).
- Base sizes on the **16 / 20 / 24 / 32 / 48 px** grid.
- Register app icons in Package.appxmanifest: `.ico` (16 / 24 / 32 / 48 / 64 / 256) plus `StoreLogo` / `Square44x44Logo` / `Square150x150Logo` / `Wide310x150Logo`.

---

## 17. Windows 11 Integration Features

- **Snap Layouts** — Show layout candidates on maximize button hover. Note window size constraints.
- **Widgets** — Adaptive Cards-based. Supported via `WindowsAppSDK`.
- **Share Charm** — Register as a share source via `DataTransferManager`.
- **File Picker** — Use `FileOpenPicker` / `FileSavePicker`; do not implement a custom replacement.

---

## Related Skills

- Accessibility in general: `skills/ui-accessibility/SKILL.md`
- UI Review: `skills/ui-review-checklist/SKILL.md`
