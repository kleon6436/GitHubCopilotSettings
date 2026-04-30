---
name: web-ui-guidelines
description: 'Web UI guidelines. Use to review and apply responsive design (mobile-first), semantic HTML, CSS architecture (Flexbox / Grid / logical properties / Container Queries), WCAG 2.2 AA, ARIA, focus management, and Core Web Vitals (LCP / CLS / INP). Use when: designing or implementing web UI; reviewing HTML/CSS/React/Vue components; applying accessibility and performance best practices.'
argument-hint: 'Topic to check (e.g. CSS / accessibility / performance — optional)'
---

# Web UI Guidelines

## Overview

This skill defines UI conventions for web apps and websites.
It is grounded in mobile-first design, semantic HTML, WCAG 2.2 AA compliance, and Core Web Vitals optimization.

References:
- [MDN Web Docs](https://developer.mozilla.org/)
- [WCAG 2.2](https://www.w3.org/TR/WCAG22/)
- [Web.dev](https://web.dev/)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)

---

## 1. Core Principles

- **Semantic HTML first.** Before reaching for `<div>` / `<span>`, use the right element (`<button>` / `<nav>` / `<article>` / `<section>` / `<header>` / `<main>` / `<footer>` / `<dialog>`).
- **Progressive enhancement.** Layer features HTML → CSS → JS, ensuring core functionality works without JS.
- **Mobile-first.** Design from the smallest width and expand with `min-width` media queries.
- **Accessibility is not an afterthought.** Build on WCAG 2.2 AA from the design stage.
- **Performance is a feature.** Treat Core Web Vitals as a design quality metric.

---

## 2. Semantic HTML

### Recommended Elements

| Usage | Element |
|---|---|
| Page primary navigation | `<nav>` |
| Page main content | `<main>` (one per page) |
| Self-contained content | `<article>` |
| Thematic grouping | `<section>` (with a heading) |
| Supplementary information | `<aside>` |
| Page/section introduction | `<header>` |
| Copyright, links, etc. | `<footer>` |
| Interactive action | `<button>` (do not use `<div onclick>`) |
| Navigation link | `<a href>` |
| Modal | `<dialog>` |
| Disclosure | `<details>` / `<summary>` |

### Heading Hierarchy

- One `<h1>` per page. Step down logically: `<h2>` → `<h3>`.
- Do not skip levels (e.g. `<h2>` → `<h4>`).
- Do not choose a heading level for styling; adjust styles with CSS.

### Forms

```html
<label for="email">Email address</label>
<input id="email" name="email" type="email" autocomplete="email" required
       aria-describedby="email-help" />
<p id="email-help">e.g. user@example.com</p>
```

- `<label>` is required. Link `for` to `id`.
- Set `autocomplete` appropriately (`email` / `current-password` / `one-time-code`, etc.).
- Use the correct `type` (`email` / `tel` / `url` / `number` / `date` / `search`). Displays the optimal mobile keyboard.
- For errors, associate the error message with `aria-invalid="true"` + `aria-describedby`.

---

## 3. CSS Architecture

### Layout Methods

| Usage | First choice |
|---|---|
| One-dimensional (row or column) | **Flexbox** |
| Two-dimensional (grid) | **CSS Grid** |
| Container-dependent switching | **Container Queries** |
| Parent-child interaction | `:has()` |

### Logical Properties

Use **logical properties** (`margin-inline-start` / `padding-inline-end`) instead of physical properties (`margin-left` / `padding-right`) for multi-language and RTL support.

```css
/* ❌ Bad */
.card { margin-left: 16px; padding-right: 24px; }

/* ✅ Good */
.card { margin-inline-start: 16px; padding-inline-end: 24px; }
```

### Modern Units & Functions

- **`clamp()`** for fluid values.

```css
h1 { font-size: clamp(1.5rem, 2.5vw + 1rem, 3rem); }
```

- **`dvh` / `svh` / `lvh`** — handle mobile browser UI show/hide (use instead of `100vh`).
- **`aspect-ratio`** to lock aspect ratio.
- **Color functions** `color-mix()` / `oklch()` for a consistent palette.

### Container Queries

```css
.card-container { container-type: inline-size; }

@container (min-width: 400px) {
    .card { display: grid; grid-template-columns: auto 1fr; }
}
```

### Cascade Layers

Use `@layer` to resolve specificity conflicts:

```css
@layer reset, base, components, utilities;
```

### Custom Properties (Tokens)

Tokenize instead of hardcoding:

```css
:root {
    --space-1: 4px;
    --space-2: 8px;
    --space-3: 16px;
    --space-4: 24px;
    --color-text: light-dark(#1a1a1a, #f5f5f5);
    --color-accent: oklch(60% 0.2 250);
    --radius-sm: 4px;
    --radius-md: 8px;
}

:root { color-scheme: light dark; }
```

---

## 4. Responsive Design

### Breakpoints (guidelines)

| Name | Min-width | Target |
|---|---|---|
| Mobile | 0 | Smartphones (portrait) |
| Tablet | 600px | Tablets, large phones (landscape) |
| Laptop | 1024px | Laptops, desktops |
| Wide | 1440px | Wide monitors |

```css
/* Mobile-first */
.grid { display: grid; grid-template-columns: 1fr; gap: 16px; }

@media (min-width: 600px) {
    .grid { grid-template-columns: repeat(2, 1fr); }
}

@media (min-width: 1024px) {
    .grid { grid-template-columns: repeat(3, 1fr); }
}
```

- **Prefer Container Queries** and limit media queries to layout shells (direct children of `<body>`) — the modern approach.
- Minimum touch target: **44×44 CSS pixels** (WCAG 2.5.5; same as Apple HIG / Material).

---

## 5. Typography

- Base font size: **16px** (`html { font-size: 100% }`). Avoid fixed px; use `rem`.
- Line length: **45–75 characters** (research-backed optimal range). Apply `max-width: 65ch` to paragraphs.
- Line height: **1.5–1.7** (body text), **1.2–1.3** for headings.
- Load fonts in **WOFF2** + `font-display: swap`, and use `size-adjust` / `ascent-override` to tune fallback metrics and prevent CLS.

```css
@font-face {
    font-family: "AppFont";
    src: url("/fonts/app.woff2") format("woff2");
    font-display: swap;
}
```

- Prefer variable fonts to reduce the number of requests.

---

## 6. Color

- Contrast ratio **WCAG AA**: 4.5:1 for normal text, 3:1 for large text (18pt / 14pt bold and above).
- Use `color-scheme: light dark` to follow the OS theme.
- Use the `light-dark()` function for theme branching.
- **Do not convey information by color alone** (use icons, text, and patterns as well).
- Identify links with underlines or clear visual distinction (do not rely on color alone).

---

## 7. Accessibility (ARIA / Focus)

### ARIA Rules

1. **Semantic HTML before ARIA.** If `<button>` suffices, `role="button"` is unnecessary.
2. Do not break native semantics.
3. All interactive ARIA widgets must be keyboard-operable.
4. Do not hide focusable elements with `role="presentation"` / `aria-hidden="true"`.

### Common Patterns

```html
<!-- Toggle open/close -->
<button aria-expanded="false" aria-controls="menu">Menu</button>
<ul id="menu" hidden>...</ul>

<!-- Modal -->
<dialog aria-labelledby="dlg-title">
    <h2 id="dlg-title">Confirmation</h2>
    ...
</dialog>

<!-- Live region -->
<div aria-live="polite" aria-atomic="true">Saved</div>

<!-- Label association -->
<button aria-label="Close">
    <svg aria-hidden="true">...</svg>
</button>
```

### Focus Management

- **Do not remove** the focus ring (`:focus { outline: none }` is forbidden). Use **`:focus-visible`** for styling instead.

```css
:focus-visible {
    outline: 2px solid var(--color-accent);
    outline-offset: 2px;
    border-radius: 4px;
}
```

- Move focus into the modal when it opens, and close it with Esc.
- Use `tabindex="0"` only as an exception; `tabindex="-1"` for programmatic focus only. Never use positive `tabindex` values.
- Place a skip link (`<a href="#main">Skip to main content</a>`) as the first focus target.

See `skills/ui-accessibility/SKILL.md` for details.

---

## 8. Motion

- Always respect **`prefers-reduced-motion`**.

```css
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        transition-duration: 0.01ms !important;
    }
}
```

- Transition duration: **150–300ms** is the baseline.
- Prefer `transform` / `opacity` (GPU-accelerated, no layout shift). Avoid animating `top` / `left` / `width`.
- Use `will-change` only when necessary; do not set it permanently.

---

## 9. Core Web Vitals

| Metric | Target | Optimization |
|---|---|---|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | `fetchpriority="high"` on hero images, font preload, CDN |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | `width` / `height` on images & iframes, font fallback tuning, ad slot reservation |
| **INP** (Interaction to Next Paint) | ≤ 200ms | Split long tasks, `requestIdleCallback`, reduce unnecessary JS |

### Image Optimization

```html
<img src="hero.avif"
     srcset="hero-400.avif 400w, hero-800.avif 800w, hero-1600.avif 1600w"
     sizes="(max-width: 600px) 100vw, 50vw"
     width="1600" height="900"
     alt="..."
     loading="lazy"
     decoding="async" />
```

- Format priority: **AVIF > WebP > JPEG/PNG**. Use `<picture>` for progressive delivery.
- Always specify `width` / `height` on images (prevents CLS).
- Use `loading="lazy"` for below-the-fold images; `fetchpriority="high"` for above-the-fold.

### JS Optimization

- Use `defer` / `async` for non-critical JS.
- Module splitting + dynamic `import()`.
- Audit third-party scripts (delay tag managers).

---

## 10. Dark Mode

```css
:root {
    color-scheme: light dark;
    --color-bg: light-dark(#ffffff, #1a1a1a);
    --color-fg: light-dark(#1a1a1a, #f5f5f5);
}

body {
    background: var(--color-bg);
    color: var(--color-fg);
}
```

- Specify `<meta name="theme-color">` per theme (syncs with the mobile URL bar).

```html
<meta name="theme-color" content="#ffffff" media="(prefers-color-scheme: light)">
<meta name="theme-color" content="#1a1a1a" media="(prefers-color-scheme: dark)">
```

- Adjust images with `filter: brightness(0.9)` in dark mode, or serve a dark variant via `<picture>`.

---

## 11. Internationalization (i18n)

- Set `<html lang="ja">` correctly.
- Use the **`Intl` API** for dates, numbers, and currencies.
- RTL support: `dir="auto"` + logical properties.
- Language-specific font fallbacks (pay attention to CJK fallback ordering).
- Allow text to expand (German / Finnish can be 1.3× longer than English).

---

## 12. Component Design

### Class Naming

- Choose **BEM** or **Utility-First** (Tailwind, etc.) and apply it consistently.
- Limit scope with CSS Modules / Scoped CSS (Vue SFC) / styled-components.

### Design System

- Define **design tokens** (colors, spacing, font sizes, shadows) as CSS custom properties.
- Two-layer structure: primitive components (Button / Input / Stack) → composite components (Card / Dialog / Form).
- Recommended strategy: use **Headless UI** (Radix / React Aria / Headless UI) for accessible behavior, and customize appearance only.

---

## 13. Images & Media

- Inline SVG (icons) or use `<img src=".svg">`.
- Use icon sets with clear licensing: **Lucide / Phosphor / Material Symbols**, etc.
- Video: `<video>` + `poster` + `preload="metadata"`. Autoplay only with muted + `playsinline`.
- `<iframe>` must have a `title` attribute.

---

## 14. Security (UI Perspective)

- External links: add `rel="noopener noreferrer"` (when using `target="_blank"`).
- Always escape user-generated content for display (rely on framework defaults; minimize `dangerouslySetInnerHTML`).
- Form submissions need CSRF tokens; use `autocomplete="off"` carefully (it hinders password managers).

---

## 15. Recommended Tools

- **Lighthouse** / **PageSpeed Insights** — performance & accessibility audits
- **axe DevTools** — accessibility inspection
- **Stylelint** — CSS linting
- **ESLint** + **eslint-plugin-jsx-a11y** — JSX accessibility
- **Storybook** — component catalogue
- **Playwright** — E2E / accessibility testing

---

## Related Skills

- General accessibility: `skills/ui-accessibility/SKILL.md`
- UI review: `skills/ui-review-checklist/SKILL.md`
