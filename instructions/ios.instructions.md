---
description: "Development guidelines for Apple iOS and iPadOS apps"
applyTo: []
---

# Project Guidelines (iOS / iPadOS App Development)

## Assumptions

- Instructions tagged with **[iPhone only]**, **[iPad only]**, or **[Universal only]** in each section should be handled according to the "Supported Devices" setting as follows:
  - **"iPhone only"**: Ignore instructions tagged [iPad only] and [Universal only]
  - **"iPad only"**: Ignore instructions tagged [iPhone only] and [Universal only]
  - **"Universal"**: Apply all instructions regardless of tags

## Project Overview

<!-- Fill in the following according to your project -->

- **Project Name**: {Project Name}
- **Overview**: {Brief description of the project's purpose and overview}
- **Target Platform**: {iOS 26.0+ / iPadOS 26.0+ / iOS & iPadOS 26.0+ (Universal)} (select as applicable)
- **Supported Devices**: {iPhone only / iPad only / iPhone + iPad (Universal)} (select as applicable)
- **Minimum Deployment Target**: {iOS 26.0 / iPadOS 26.0}
- **Repository Structure**: {Single repo / Monorepo, description of main directory structure}

## Tech Stack

### Recommended

| Category | Technology / Tool | Version | Notes |
|---------|-------------|-----------|------|
| Language | Swift | 6 | |
| IDE | Xcode | 26 | |
| Project Management | XcodeGen | Latest | Managed via project.yml |
| Package Manager | Swift Package Manager | | |
| UI Framework | SwiftUI | iOS 26 SDK | Minimize mixing with UIKit |
| UI Framework (Supplementary) | UIKit | | Only when SwiftUI is insufficient |
| Architecture | MVC | | |
| Testing | XCTest / Swift Testing | | Both frameworks can be used together |
| Linter / Formatter | SwiftLint | Latest | Configured via .swiftlint.yml |
| Icon Creation | Icon Composer | Built into Xcode 26 | Create layered icons |
| CI/CD | {e.g., GitHub Actions} | | |

### Planned Additions

| Category | Technology / Tool | Version | Notes |
|---------|-------------|-----------|------|
| Widgets | WidgetKit | | Home screen / lock screen widgets |
| System Integration | App Intents | | Siri / Shortcuts support |

## Recommended Copilot Agent Configuration

- When working with multiple agents, use `agents/orchestrator.agent.md` as the central coordinator.
- Use `agents/product-manager.agent.md` for requirements clarification, `agents/architect.agent.md` for technical design, and `agents/developer.agent.md` for implementation.
- For UI changes including Liquid Glass support, use `agents/ui-designer.agent.md` in conjunction to finalize screen states, visual hierarchy, and accessibility first.
- Run final quality gates through `agents/reviewer.agent.md` and `agents/tester.agent.md`.

## UI Guidelines

For UI design and implementation on iOS / iPadOS (HIG, Liquid Glass, navigation, size classes, Dynamic Type, icons, etc.), refer to the following skills:

- `skills/apple-ui-guidelines/SKILL.md` — Apple Platform UI Guidelines (iOS / iPadOS / macOS common)
- `skills/ui-accessibility/SKILL.md` — Common accessibility principles
- `skills/ui-review-checklist/SKILL.md` — Checklist for UI review


## Coding Standards

For Swift coding standards, refer to `skills/swift-coding-standards/SKILL.md`.
