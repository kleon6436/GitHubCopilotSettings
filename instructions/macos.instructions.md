---
description: "Development guidelines for Apple macOS apps"
applyTo: []
---

# Project Guidelines (macOS Tahoe App Development)

## Project Overview

<!-- Fill in the following according to your project -->

- **Project Name**: {Project Name}
- **Overview**: {Brief description of the project's purpose and overview}
- **Target Platform**: macOS 26 (macOS Tahoe) or later
- **Minimum Deployment Target**: macOS 26.0
- **Repository Structure**: {Single repo / Monorepo, description of main directory structure}

## Tech Stack

| Category | Technology / Tool | Version | Notes |
|---------|-------------|-----------|------|
| Language | Swift | 6 | |
| IDE | Xcode | 26 | |
| Project Management | XcodeGen | Latest | Managed via project.yml |
| Package Manager | Swift Package Manager | | |
| UI Framework | SwiftUI | macOS 26 SDK | Minimize mixing with AppKit |
| Architecture | MVC | | |
| Testing | XCTest / Swift Testing | | Both frameworks can be used together |
| Linter / Formatter | SwiftLint | Latest | Configured via .swiftlint.yml |
| Icon Creation | Icon Composer | Built into Xcode 26 | Create layered icons |
| CI/CD | {e.g., GitHub Actions} | | |

## Recommended Copilot Agent Configuration

- When working with multiple agents, use `agents/orchestrator.agent.md` as the starting point.
- Use `agents/product-manager.agent.md` for requirements clarification, `agents/architect.agent.md` for technical design, and `agents/developer.agent.md` for implementation.
- For UI work involving HIG / Liquid Glass considerations, use `agents/ui-designer.agent.md` in conjunction to finalize information architecture and accessibility early.
- After implementation, use `agents/reviewer.agent.md` and `agents/tester.agent.md` as quality gates.

## UI Guidelines

For UI design and implementation on macOS (HIG, Liquid Glass, windows, navigation, keyboard shortcuts, icons, etc.), refer to the following skills:

- `skills/apple-ui-guidelines/SKILL.md` — Apple Platform UI Guidelines (iOS / iPadOS / macOS common)
- `skills/ui-accessibility/SKILL.md` — Common accessibility principles
- `skills/ui-review-checklist/SKILL.md` — Checklist for UI review


## Coding Standards

For Swift coding standards, refer to `skills/swift-coding-standards/SKILL.md`.
