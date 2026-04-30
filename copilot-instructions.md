# Project Guidelines

## Assumptions

- When making code changes that are likely to exceed 200 lines, first ask the user: "This instruction may result in changes exceeding 200 lines. Do you want to proceed?"
- Before making any large changes, plan what to do first, then propose to the user: "Here is the plan I'd like to follow."
- Think before you write
- Prefer simplicity
- Only touch what is necessary
- Work toward goals
- After changes, always perform code review and testing

## Project Overview

<!-- Fill in the following as appropriate for your project -->

- **Project Name**: {Project Name}
- **Overview**: {Brief description of the project's purpose and overview}
- **Target Platform**: {iOS / Android / Web / Server, etc.}
- **Repository Structure**: {Monorepo / Single repo, description of main directory structure}

## Tech Stack

<!-- Fill in technologies used in the project per platform -->
<!-- Delete unnecessary rows and add rows as needed -->

| Category | Technology / Tool | Version | Notes |
|---------|-------------|-----------|------|
| Language | {e.g. Swift} | {e.g. 5.9} | |
| IDE / Editor | {e.g. Xcode} | {e.g. 15.0} | |
| Project Management | {e.g. XcodeGen} | {e.g. 2.38} | |
| Package Manager | {e.g. Swift Package Manager} | | |
| UI Framework | {e.g. SwiftUI} | | |
| Architecture | {e.g. MVVM} | | |
| Testing | {e.g. XCTest} | | |
| CI/CD | {e.g. GitHub Actions} | | |
| Linter / Formatter | {e.g. SwiftLint} | | |
| Other | | | |

## Recommended Copilot Agent Configuration

- When using the orchestration pattern, use the templates under `agents/`.
- Use `agents/sisyphus.agent.md` as the central entry point and delegate to specialist agents based on task type.
- Handle small tasks and typo fixes with `sisyphus-junior` to save high-cost model usage.
- Always plan with `prometheus` and run `metis` gap analysis before implementing.
- Always have important changes reviewed by `momus`.

### Agent List (10 agents)

**Discipline Layer**

| Agent | Model | Primary Responsibilities |
|-------|--------|------|
| `sisyphus` | Claude Sonnet 4.6 | Main orchestrator. Intent analysis, delegation, verification, integration, BOULDER.md management |
| `sisyphus-junior` | GPT-5 mini | Lightweight orchestrator. Dedicated to typos, single-line changes, and small tasks |
| `prometheus` | Claude Sonnet 4.6 | Strategic planner. Requirements gathering, acceptance criteria, plan creation. Does not write code |
| `hephaestus` | GPT-5.3-Codex | Autonomous deep worker. Self-contained explore→plan→execute→verify cycle. Explicit activation only |

**Specialized Layer**

| Agent | Model | Primary Responsibilities |
|-------|--------|------|
| `oracle` | GPT-5.4 | Top-level consultant. Complex debugging, architecture decisions. Explicit activation only when the path forward is unclear |
| `librarian` | GPT-5 mini | Evidence-based researcher. Official docs, GitHub examples. URL/permalink required |
| `explore` | Grok Code Fast 1 | Fast codebase scanner. Parallel activation allowed. Read-only |
| `metis` | GPT-5.4 mini | Plan consultant. Catches ambiguity, gaps, and incorrect assumptions in the planning phase |
| `momus` | GPT-5.4 | Relentless verifier. Comprehensive code review, test quality, security (OWASP Top 10) |
| `atlas` | GPT-5.4 mini | Implementer. Executes verified plans. Also handles CI/CD and deployment |

### Category Quick Reference

| Category | Example Tasks | Recommended Agent | Recommended Model |
|---------|---------|-----------|----------|
| quick | typo, single-line fix, config value change | `sisyphus-junior` | GPT-5 mini |
| plan | requirements, planning, acceptance criteria | `prometheus` | Claude Sonnet 4.6 |
| deep | autonomous large-scale implementation | `hephaestus` | GPT-5.3-Codex |
| ultrabrain | architecture decisions, complex debugging | `oracle` | GPT-5.4 |
| writing | documentation, research, cited answers | `librarian` | GPT-5 mini |
| search | codebase grep, dependency analysis | `explore` | Grok Code Fast 1 |
| review | code quality, testing, security | `momus` | GPT-5.4 |
| implement | implementation, fixes, CI/CD | `atlas` | GPT-5.4 mini |
| visual-engineering | UI/UX, accessibility | `atlas` (using Gemini 3.1 Pro) | Gemini 3.1 Pro |

### Model Cost Policy

- **High cost (evaluate each time)**: Claude Sonnet 4.6 / GPT-5.4 / GPT-5.3-Codex — limit to complex reasoning, critical design decisions, and large-scale implementation
- **Medium cost (use actively)**: GPT-5.4 mini / Gemini 3.1 Pro — planning assistance, implementation, visual tasks
- **Low cost (use freely)**: GPT-5 mini / Grok Code Fast 1 — small tasks, search, research

> `atlas` uses GPT-5.4 mini for lighter cases; consider switching to Claude Sonnet 4.6 for large-scale refactoring or implementations that must closely follow existing conventions.

### BOULDER.md Protocol

For session continuity, `sisyphus` manages `BOULDER.md` in the project root.

```markdown
# Boulder - Session State
Last Updated: {datetime}
Task: {task summary}

## Completed ✅
- [x] ...

## In Progress 🔄
- [ ] ...

## On Hold / Blockers
- ...

## Handoff Notes
{Important information and decision rationale for the next session}
```

- **Session start**: `sisyphus` reads `BOULDER.md` to understand incomplete tasks before starting work
- **After each major step**: Update Completed / In Progress / On Hold
- **Session end**: Record remaining tasks and handoff notes before closing

## Platform-Specific Guidelines

For detailed development guidelines per platform, refer to the following instruction files.

| Platform | Instruction File |
|--------------|-------------|
| iOS / iPadOS | `instrctions/ios.instructions.md` |
| macOS | `instrctions/macos.instructions.md` |
| Android | `instrctions/android.instructions.md` |
| Web | `instrctions/web.instructions.md` |
| Windows | `instrctions/windows.instructions.md` |
| Cross-Platform | `instrctions/cross-platform.instructions.md` |

## Skills List

| Category | Skills |
|---------|-------|
| **Coding Standards** | `swift-coding-standards` / `kotlin-coding-standards` / `typescript-coding-standards` / `javascript-coding-standards` / `python-coding-standards` / `cpp-coding-standards` / `csharp-coding-standards` / `rust-coding-standards` / `css-coding-standards` / `react-coding-standards` |
| **UI / UX** | `apple-ui-guidelines` / `android-ui-guidelines` / `web-ui-guidelines` / `windows-ui-guidelines` / `mfc-ui-guidelines` / `ui-accessibility` / `ui-review-checklist` / `design-system` |
| **Quality & Security** | `security-practices` / `cicd-deployment` / `performance-optimization` / `apple-app-store-submission` |
| **Internationalization** | `i18n-localization` |