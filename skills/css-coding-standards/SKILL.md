---
name: css-coding-standards
description: 'Reference and apply CSS and Tailwind CSS coding standards. Use when: applying CSS or Tailwind CSS style guide, BEM naming, selector conventions, property ordering, responsive design, CSS custom properties, utility-first styling.'
argument-hint: 'Coding standard item to check or apply (optional)'
---

# CSS / Tailwind CSS Coding Standards

## Overview

This skill defines the coding standards for CSS and Tailwind CSS.
For projects using Tailwind CSS, the Tailwind approach takes priority; plain CSS is used only as a supplement.
Follow these standards during code reviews and new implementations.

---

## 1. Naming Conventions

### When Writing Plain CSS: BEM (Block Element Modifier)

- **Block**: independent component → `block`
- **Element**: part of a Block → `block__element`
- **Modifier**: state or visual variation → `block--modifier` / `block__element--modifier`
- Use `kebab-case` for all class names.

```css
/* ✅ Good */
.user-card { }
.user-card__avatar { }
.user-card__name { }
.user-card--featured { }
.user-card__name--truncated { }

/* ❌ Bad */
.UserCard { }          /* UpperCamelCase */
.userCard_avatar { }   /* not BEM */
.card.featured { }     /* Modifier expressed as a separate class */
```

### When Using Tailwind CSS: Utility-First

- Apply utility classes directly in HTML / JSX.
- Avoid overusing `@apply` for custom classes; prefer component decomposition instead.
- Use `@apply` only for frequently repeated patterns.

```html
<!-- ✅ Good -->
<div class="flex items-center gap-4 rounded-lg border border-gray-200 p-4 shadow-sm">
  <img class="h-12 w-12 rounded-full object-cover" src="..." alt="...">
  <p class="text-sm font-medium text-gray-900">Username</p>
</div>

<!-- ❌ Bad -->
<div class="custom-card">  <!-- write everything with @apply in a custom class -->
```

```css
/* ❌ Bad (@apply overuse) */
.custom-card {
  @apply flex items-center gap-4 rounded-lg border border-gray-200 p-4 shadow-sm;
}
```

---

## 2. Selector Conventions

- Do not use ID selectors (`#id`) for styling (they may be used as JavaScript hooks).
- Avoid using tag selectors (`div`, `p`, etc.) alone; combine them with classes.
- Keep selector specificity as low as possible.
- Do not use `!important` as a rule. If used, document the reason in a comment.

```css
/* ✅ Good */
.user-card { }
.nav-link:hover { }
.user-card > .user-card__avatar { }

/* ❌ Bad */
#user-card { }          /* ID selector */
div.user-card { }       /* unnecessary tag qualifier */
.user-card { color: red !important; }  /* !important */
```

---

## 3. Property Declaration Order

Declare properties in the following category order.

1. **Layout & Display**: `display`, `position`, `top/right/bottom/left`, `z-index`, `float`, `clear`
2. **Box Model**: `width`, `height`, `margin`, `padding`, `border`, `border-radius`
3. **Text & Font**: `font-family`, `font-size`, `font-weight`, `line-height`, `color`, `text-align`
4. **Background & Decoration**: `background`, `box-shadow`, `opacity`
5. **Transition & Animation**: `transition`, `animation`, `transform`
6. **Other**: `cursor`, `overflow`, `pointer-events`, `content`

```css
/* ✅ Good */
.button {
  /* Layout */
  display: inline-flex;
  position: relative;
  /* Box Model */
  padding: 8px 16px;
  border: 1px solid transparent;
  border-radius: 4px;
  /* Text */
  font-size: 14px;
  font-weight: 500;
  color: #fff;
  /* Background */
  background-color: #3b82f6;
  /* Transition */
  transition: background-color 150ms ease;
  /* Other */
  cursor: pointer;
}
```

---

## 4. Tailwind CSS Usage Guidelines

- Define design tokens (colors, fonts, spacing, etc.) in `tailwind.config`; minimize arbitrary values (e.g., `text-[14px]`).
- Write classes in the order: **layout → box → text → color/background → state**.
- Place variants such as `dark:` / `hover:` / `focus:` adjacent to the target class.
- Write responsive prefixes in the order `sm:` → `md:` → `lg:` → `xl:`.

```html
<!-- ✅ Good (class ordering) -->
<button class="
  flex items-center justify-center
  h-10 w-full px-4 rounded-md
  text-sm font-medium text-white
  bg-blue-600 hover:bg-blue-700
  focus:outline-none focus:ring-2 focus:ring-blue-500
  disabled:cursor-not-allowed disabled:opacity-50
  transition-colors
">
  Submit
</button>
```

---

## 5. Responsive Design

- Design **mobile-first** (base styles target mobile; expand with breakpoints).
- With Tailwind: write in the order no prefix → `sm:` → `md:` → `lg:` → `xl:`.
- With plain CSS: use `min-width` media queries.

```html
<!-- ✅ Good (mobile-first) -->
<div class="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
```

```css
/* ✅ Good (mobile-first) */
.grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 16px;
}

@media (min-width: 640px) {
  .grid { grid-template-columns: repeat(2, 1fr); }
}

@media (min-width: 1024px) {
  .grid { grid-template-columns: repeat(3, 1fr); }
}

/* ❌ Bad (desktop-first) */
@media (max-width: 1024px) { .grid { grid-template-columns: repeat(2, 1fr); } }
@media (max-width: 640px)  { .grid { grid-template-columns: 1fr; } }
```

---

## 6. Custom Properties (CSS Variables)

- Define design tokens (colors, spacing, fonts, etc.) as CSS custom properties.
- Define custom properties on `:root` and name them with the `--` prefix.
- When using Tailwind, manage tokens in `tailwind.config` and avoid duplicating them as custom properties.

```css
/* ✅ Good */
:root {
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;
  --color-text-primary: #111827;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --radius-md: 6px;
  --font-size-sm: 14px;
}

.button {
  background-color: var(--color-primary-500);
  border-radius: var(--radius-md);
  padding: var(--spacing-sm) var(--spacing-md);
}

/* ❌ Bad (hardcoded values) */
.button {
  background-color: #3b82f6;
  border-radius: 6px;
  padding: 8px 16px;
}
```

---

## 7. Comment Conventions

- Describe the purpose and scope at the top of the file.
- Use comment blocks to separate sections.
- Add comments only for non-obvious code (hacks, browser compatibility, etc.).
- Write TODO / FIXME in the format `/* TODO: description */`.

```css
/* ==========================================================================
   Button Component
   ========================================================================== */

/* Primary button */
.button--primary { }

/* Secondary button */
.button--secondary { }

/* Hack to work around the gap bug in Safari 16 and below */
.flex-container > * + * {
  margin-left: var(--spacing-sm);
}

/* TODO: Add dark mode support */
```

---

## 8. Project-Specific Rules

<!-- Add project-specific rules as needed -->

- {Add project-specific rules here}
