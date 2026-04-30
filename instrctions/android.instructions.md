---
description: "Development guidelines for Android apps"
applyTo: []
---

# Project Guidelines (Android App Development)

## Project Overview

<!-- Fill in the following according to your project -->

- **Project Name**: {Project Name}
- **Overview**: {Brief description of the project's purpose and overview}
- **Target Platform**: Android 10 (API 29) or later
- **Minimum minSdk**: {29}
- **Target SDK**: {Latest stable}
- **Repository Structure**: {Single repo / Monorepo, description of main directory structure}

## Tech Stack

| Category | Technology / Tool | Version | Notes |
|---------|-------------|-----------|------|
| Language | Kotlin | Latest stable | |
| IDE | Android Studio | Latest stable | |
| Build System | Gradle (KTS) | | Use build.gradle.kts |
| UI Framework | Jetpack Compose | Latest stable | Minimize mixing with View system |
| DI | Hilt | Latest stable | |
| Navigation | Navigation Compose | Latest stable | |
| Async | Kotlin Coroutines / Flow | Latest stable | |
| Networking | Retrofit + OkHttp | Latest stable | |
| Architecture | MVVM + Clean Architecture | | |
| Testing | JUnit 5 / Espresso / Compose Testing | Latest stable | |
| Linter | ktlint / Detekt | Latest stable | |
| CI/CD | {e.g., GitHub Actions} | | |

## Recommended Copilot Agent Configuration

- When working with multiple agents, use `agents/orchestrator.agent.md` as the starting point.
- Use `agents/product-manager.agent.md` for requirements clarification, `agents/architect.agent.md` for technical design, and `agents/developer.agent.md` for implementation.
- For UI work involving Material Design 3 considerations, use `agents/ui-designer.agent.md` in conjunction to finalize information architecture and accessibility early.
- After implementation, use `agents/reviewer.agent.md` and `agents/tester.agent.md` as quality gates.
- Use `agents/devops.agent.md` for CI/CD configuration and distribution pipelines.

## UI Guidelines

For UI design and implementation on Android (Material Design 3, Jetpack Compose, navigation, Large Screen support, etc.), refer to the following skills:

- `skills/android-ui-guidelines/SKILL.md` — Android UI Guidelines (Material Design 3 / Jetpack Compose)
- `skills/ui-accessibility/SKILL.md` — Common accessibility principles
- `skills/ui-review-checklist/SKILL.md` — Checklist for UI review

## Coding Standards

For Kotlin coding standards, refer to `skills/kotlin-coding-standards/SKILL.md`.

## Architecture Policy

### Layer Structure (Clean Architecture)

```
presentation/   — ViewModel · UI State · Compose Screen
domain/         — UseCase · Repository Interface · Entity
data/           — Repository Implementation · DataSource · API Client
```

### State Management

- Aggregate UI State in `ViewModel` and push to Compose via `StateFlow`
- Notify side effects (navigation, snackbars, etc.) via `SharedFlow` or `Channel`
- Keep `MutableStateFlow` private; expose only read-only `StateFlow` externally

### Dependencies

- Always use Hilt for DI
- Apply `@Singleton` only to scopes that truly need to be shared
- Inject dependencies into ViewModels via `@HiltViewModel`

## Testing Policy

- Unit test ViewModels and UseCases with **JUnit 5 + MockK**
- Integration test Compose UI with **Compose Testing**
- Use **Espresso** or **UI Automator** for E2E tests
- ViewModels and UseCases should depend on interfaces for testability

## Large Screen / Foldable Support

- Use `WindowSizeClass` to switch layouts
- Compact: single panel; Medium and above: two-panel layout as baseline
- Apply layout branching considering `FoldingFeature` for foldable devices
- Avoid overusing `Modifier.fillMaxSize()`; prefer adaptive layouts

## Security

- Store sensitive data with `EncryptedSharedPreferences` or `DataStore` + AES encryption
- Manage API keys and secrets in `local.properties`; do not commit to repository
- Allow HTTPS only for network traffic; consider certificate pinning
- Configure obfuscation rules for `ProGuard / R8`

For details, refer to `skills/security-practices/SKILL.md`.

## Internationalization (i18n)

- Always manage text via `strings.xml`; do not hardcode in code
- Prefer `start/end` over `left/right` for RTL language support
- Use `plurals` for plural forms

For details, refer to `skills/i18n-localization/SKILL.md`.

## Performance

- Use `remember` / `derivedStateOf` / `key` appropriately to avoid unnecessary recomposition
- Use **Coil** for image loading and leverage caching with `AsyncImage`
- Use `LazyColumn` / `LazyRow` for lists and specify `items(key = ...)`
- Always run blocking operations on `Dispatchers.IO` or `Dispatchers.Default` to prevent ANR

For details, refer to `skills/performance-optimization/SKILL.md`.
