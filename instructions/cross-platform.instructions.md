---
description: "Development guidelines for cross-platform projects spanning multiple platforms"
applyTo: []
---

# Project Guidelines (Cross-Platform Development)

## Assumptions

These guidelines apply to projects developing concurrently across multiple platforms: iOS / iPadOS, macOS, Android, Web, and Windows.
Use in conjunction with platform-specific guidelines; this file defines **common policies**.

## Platform-Specific Guideline References

For details on each platform, follow the instruction files below:

| Platform | Instruction File |
|--------------|-------------|
| iOS / iPadOS | `instructions/ios.instructions.md` |
| macOS | `instructions/macos.instructions.md` |
| Android | `instructions/android.instructions.md` |
| Web | `instructions/web.instructions.md` |
| Windows | `instructions/windows.instructions.md` |

## Recommended Copilot Agent Configuration

Agent coordination is especially important in cross-platform projects.

- Use `sisyphus` as the main orchestrator. All tasks start here.
- Use `prometheus` for requirements gathering, platform difference definitions, and acceptance criteria.
- Use `oracle` for cross-platform architecture decisions and shared API design.
- Use `metis` / `metis-deep` for gap analysis on plans spanning multiple platforms.
- For visual-engineering tasks on each platform, pass to `atlas` (using Gemini 3.1 Pro) with explicit platform context.
- For security-sensitive changes (auth, cross-platform data handling), route reviews through `momus-deep`.
- **UI is platform-native** by principle; even when functionality is shared, UI is not unified.

## Common Architecture Policy

### What to Share

| Type | Description |
|------|------|
| **API Specifications** | OpenAPI / GraphQL schemas · endpoint definitions |
| **Domain Models** | Entities · business rules (align logic across languages) |
| **Error Codes** | Error code definitions returned from the server |
| **Feature Flags** | ON/OFF conditions for feature toggles |
| **Analytics Events** | Tracking event names · parameter specifications |

### What NOT to Share (Platform-Specific)

| Type | Reason |
|------|------|
| **UI Components** | Must comply with HIG, Material Design, Fluent Design respectively |
| **Navigation** | Platform navigation conventions differ |
| **Storage APIs** | KeyChain / SharedPreferences / localStorage specs differ |
| **Push Notifications** | APNs / FCM implementations differ |

## API Design Policy

### Backend API

- Use **REST** or **GraphQL** with a design accessible from all platforms
- Manage OpenAPI 3.x or GraphQL schema as Single Source of Truth
- Prefer **auto-generating** client code for each platform from the schema
- API versioning via URL path (`/v1/`) or HTTP headers

### Handling Platform Differences

- Abstract platform-specific features (In-App Purchase, push notifications, etc.) in the backend
- Include platform conditions in feature flags to enable staged rollouts

## Feature Parity Management

### Parity Matrix (example)

| Feature | iOS | Android | Web | Windows | macOS |
|------|-----|---------|-----|---------|-------|
| Authentication | ✅ | ✅ | ✅ | {planned} | {planned} |
| Offline Support | ✅ | ✅ | {planned} | — | — |
| Push Notifications | ✅ | ✅ | ✅ (Web Push) | {planned} | — |

- `✅` = implemented, `{planned}` = planned, `—` = not supported / out of scope

### Principles for Difference Management

- Product Manager manages feature differences per platform
- Where differences exist, provide platform-appropriate alternative experiences in the UI
- Feature priorities are determined based on user count and revenue contribution

## Design System and Brand Consistency

- Unify brand elements such as color palettes, typography, and icon sets
- However, **adapt the expression** to match each platform's design language
- For design tokens, refer to `skills/design-system/SKILL.md`

## Testing Policy

### Shared Logic Testing

- Write unit tests for business logic in a form independent of language and platform
- Share API schema validation tests across all platforms

### E2E Tests

- Cover primary user flows with E2E tests on each platform
- Define test scenarios commonly and execute with platform-specific implementations

## CI/CD

- Each platform has an independent pipeline while sharing common quality gates
- For details, refer to `skills/cicd-deployment/SKILL.md`

## Security

- Unify authentication and authorization policies across all platforms
- For details, refer to `skills/security-practices/SKILL.md`

## Internationalization (i18n)

- Centralize translation string source management and convert to per-platform formats
- For details, refer to `skills/i18n-localization/SKILL.md`
