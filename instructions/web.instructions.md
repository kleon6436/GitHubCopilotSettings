---
description: "Development guidelines for web apps"
applyTo: []
---

# Project Guidelines (Web App Development)

## Project Overview

<!-- Fill in the following according to your project -->

- **Project Name**: {Project Name}
- **Overview**: {Brief description of the project's purpose and overview}
- **Target Browsers**: {e.g., Chrome 120+ / Firefox 120+ / Safari 17+ / Edge 120+}
- **Rendering Method**: {CSR / SSR / SSG / ISR} (select as applicable)
- **Repository Structure**: {Single repo / Monorepo, description of main directory structure}

## Tech Stack

| Category | Technology / Tool | Version | Notes |
|---------|-------------|-----------|------|
| Language | TypeScript | Latest stable 5.x | strict mode enabled |
| UI Framework | {e.g., React / Vue 3 / Svelte} | Latest stable | |
| Meta-framework | {e.g., Next.js / Nuxt / SvelteKit} | Latest stable | |
| Styling | {e.g., Tailwind CSS / CSS Modules} | Latest stable | |
| State Management | {e.g., Zustand / Pinia / Jotai} | Latest stable | |
| Data Fetching | {e.g., TanStack Query / SWR} | Latest stable | |
| Testing | Vitest + Testing Library + Playwright | Latest stable | |
| Formatter | Prettier | Latest stable | |
| Linter | ESLint + typescript-eslint | Latest stable | |
| Package Manager | {e.g., pnpm / npm / yarn} | Latest stable | |
| CI/CD | {e.g., GitHub Actions + Vercel / Cloudflare Pages} | | |

## Recommended Copilot Agent Configuration

- When working with multiple agents, use `agents/orchestrator.agent.md` as the starting point.
- Use `agents/product-manager.agent.md` for requirements clarification, `agents/architect.agent.md` for technical design, and `agents/developer.agent.md` for implementation.
- For UI design involving WCAG 2.2 AA compliance, use `agents/ui-designer.agent.md` in conjunction.
- After implementation, use `agents/reviewer.agent.md` and `agents/tester.agent.md` as quality gates.
- Use `agents/devops.agent.md` for deployment and infrastructure configuration.
- Route security reviews through `agents/security-reviewer.agent.md`.

## UI Guidelines

For Web UI design and implementation, refer to the following skills:

- `skills/web-ui-guidelines/SKILL.md` — Web UI Guidelines (Semantic HTML, Responsive Design, Core Web Vitals)
- `skills/ui-accessibility/SKILL.md` — Common accessibility principles
- `skills/ui-review-checklist/SKILL.md` — Checklist for UI review

## Coding Standards

- TypeScript: `skills/typescript-coding-standards/SKILL.md`
- JavaScript: `skills/javascript-coding-standards/SKILL.md`
- CSS: `skills/css-coding-standards/SKILL.md`
- React (if used): `skills/react-coding-standards/SKILL.md`

## Architecture Policy

### Directory Structure (e.g., Next.js App Router)

```
src/
  app/           — Routing · pages · layouts
  components/    — Reusable UI components
  features/      — Feature domain modules
  hooks/         — Custom Hooks
  lib/           — External library wrappers and configuration
  services/      — API clients · external services
  stores/        — Global state management
  types/         — Type definitions
  utils/         — General utilities
```

### Component Design

- Clearly separate **Server Components** and **Client Components**
- Apply `"use client"` only where truly necessary
- Design components with single responsibility; consider splitting when exceeding 100 lines
- Define props types with `interface` and explicitly mark optional properties with `?`

### Data Fetching

- Prefer server-side fetching; use client-side fetching only when necessary for UX
- Always implement `loading.tsx` / `error.tsx` / `not-found.tsx` as a set
- Validate API responses with `zod`

## Testing Policy

- Write component tests with **Vitest + Testing Library** based on user interactions
- Use **Playwright** for E2E tests
- Use `data-testid` exclusively for testing; confirm it is not visible to users
- Use snapshot tests for UI change detection, but do not over-rely on them

## Core Web Vitals Targets

| Metric | Target |
|------|------|
| LCP (Largest Contentful Paint) | ≤ 2.5s |
| INP (Interaction to Next Paint) | ≤ 200ms |
| CLS (Cumulative Layout Shift) | ≤ 0.1 |

For details, refer to `skills/performance-optimization/SKILL.md`.

## Security

- XSS prevention: Do not insert user input directly into the DOM; avoid `dangerouslySetInnerHTML`
- CSRF prevention: Use CSRF tokens or SameSite cookies for state-changing requests
- Environment variables: Manage client-exposed variables with explicit prefixes like `NEXT_PUBLIC_`
- Configure Content Security Policy (CSP) appropriately

For details, refer to `skills/security-practices/SKILL.md`.

## Internationalization (i18n)

- Manage text with libraries like `next-intl` / `i18next`; do not hardcode in code
- Organize translation files in `messages/{locale}.json` or similar
- Use `Intl` API or i18n libraries for date, number, and currency formatting
- Use `dir` attribute and `logical properties` for RTL language support

For details, refer to `skills/i18n-localization/SKILL.md`.
