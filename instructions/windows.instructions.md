---
description: "Development guidelines for Windows apps"
applyTo: []
---

# Project Guidelines (Windows App Development)

## Project Overview

<!-- Fill in the following according to your project -->

- **Project Name**: {Project Name}
- **Overview**: {Brief description of the project's purpose and overview}
- **Target Platform**: Windows 11 or later (Windows 10 support is optional)
- **Minimum Supported Version**: {Windows 11 22H2 / Windows 10 1903} (select as applicable)
- **Distribution Method**: {Microsoft Store / Sideloading / MSIX / Win32 Installer} (select)
- **Repository Structure**: {Single repo / Monorepo, description of main directory structure}

## Tech Stack

| Category | Technology / Tool | Version | Notes |
|---------|-------------|-----------|------|
| Language | C# | Latest stable | nullable enabled |
| IDE | Visual Studio / VS Code | Latest stable | |
| UI Framework | WinUI 3 | Latest stable | Uses Windows App SDK |
| SDK | Windows App SDK | Latest stable | |
| Architecture | MVVM | | CommunityToolkit.Mvvm recommended |
| Async | async/await | | |
| Testing | MSTest v3 / xUnit | Latest stable | |
| Package Management | NuGet | | |
| Linter / Formatter | Roslyn Analyzers / EditorConfig | | |
| CI/CD | {e.g., GitHub Actions} | | |

## Recommended Copilot Agent Configuration

- When working with multiple agents, use `agents/orchestrator.agent.md` as the starting point.
- Use `agents/product-manager.agent.md` for requirements clarification, `agents/architect.agent.md` for technical design, and `agents/developer.agent.md` for implementation.
- Use `agents/ui-designer.agent.md` in conjunction for Fluent Design and WinUI 3 component considerations.
- After implementation, use `agents/reviewer.agent.md` and `agents/tester.agent.md` as quality gates.
- Use `agents/devops.agent.md` for packaging and distribution pipelines.

## UI Guidelines

For UI design and implementation on Windows (Fluent Design, WinUI 3, navigation, title bar integration, etc.), refer to the following skills:

- `skills/windows-ui-guidelines/SKILL.md` — Windows UI Guidelines (Fluent Design / WinUI 3)
- `skills/mfc-ui-guidelines/SKILL.md` — MFC Application UI Guidelines
- `skills/ui-accessibility/SKILL.md` — Common accessibility principles
- `skills/ui-review-checklist/SKILL.md` — Checklist for UI review

## Coding Standards

For C# coding standards, refer to `skills/csharp-coding-standards/SKILL.md`.

## Architecture Policy

### Layer Structure (MVVM)

```
Views/           — XAML pages · UserControls · dialogs
ViewModels/      — ViewModel (INotifyPropertyChanged)
Models/          — Domain models · DTOs
Services/        — Business logic · external APIs
Repository/      — Data access layer
```

### MVVM Practice

- Leverage `ObservableObject` / `RelayCommand` from `CommunityToolkit.Mvvm`
- Use `[ObservableProperty]` source generators to reduce boilerplate
- Views access data only through ViewModels; do not write logic in code-behind
- ViewModels do not hold direct references to Views

### Dependency Injection

- Use `Microsoft.Extensions.DependencyInjection`
- Configure the DI container in `App.xaml.cs` or `Program.cs`

## Window Management

- Manage multi-windows with `Microsoft.UI.Xaml.Window`
- Persist window state (position, size) with `ApplicationData.Current.LocalSettings`
- Use `ExtendsContentIntoTitleBar` for title bar integration to maximize app content area

## Input Handling

- Design primarily for **mouse / keyboard**, with touch and pen as optional
- Define keyboard shortcuts with `KeyboardAccelerator` and display in `ToolTip`
- Minimum tap target size for touch support is 44×44px

## Testing Policy

- Unit test ViewModels and services with **MSTest v3** or **xUnit**
- Consider **Appium + WinAppDriver** for UI tests
- Services should depend on interfaces for testability

## Packaging and Distribution

- **MSIX** package is the recommended distribution format
- Set up Store association early when distributing via Microsoft Store
- Follow semantic versioning with the `Version` attribute in `Package.appxmanifest`

## Security

- Store sensitive data in `PasswordVault` (Windows Credential Manager)
- Store app data in `ApplicationData.Current.LocalFolder` with appropriate permissions
- Allow HTTPS only for network traffic

For details, refer to `skills/security-practices/SKILL.md`.

## Internationalization (i18n)

- Manage text with `Resources.resw`; do not hardcode in code
- Load resources via `ResourceLoader`
- Use `Windows.Globalization` APIs for dates, numbers, and currencies

For details, refer to `skills/i18n-localization/SKILL.md`.
