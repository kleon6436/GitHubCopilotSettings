---
name: css-coding-standards
description: 'CSS・Tailwind CSSのコーディング規約を参照・適用する。CSS・Tailwind コーディング規約、命名規則（BEM）、セレクター規約、プロパティ記述順序、Tailwind使用方針、レスポンシブデザイン、CSS Variables（カスタムプロパティ）、コメント規約を確認・適用したいときに使用。Use when: applying CSS or Tailwind CSS style guide, BEM naming, selector conventions, property ordering, responsive design, CSS custom properties, utility-first styling.'
argument-hint: '確認・適用したいコーディング規約の項目（省略可）'
---

# CSS / Tailwind CSS コーディング規約

## 概要

このスキルは CSS および Tailwind CSS のコーディング規約を定義します。
Tailwind CSS を使用するプロジェクトでは Tailwind の方針を優先し、素の CSS は補助的に使用します。
コードレビュー・新規実装の際はこの規約に従ってください。

---

## 1. 命名規則

### 素の CSS を書く場合：BEM（Block Element Modifier）

- **Block**：独立したコンポーネント → `block`
- **Element**：Block の構成要素 → `block__element`
- **Modifier**：状態や見た目のバリエーション → `block--modifier` / `block__element--modifier`
- クラス名はすべて `kebab-case` を使用する。

```css
/* ✅ Good */
.user-card { }
.user-card__avatar { }
.user-card__name { }
.user-card--featured { }
.user-card__name--truncated { }

/* ❌ Bad */
.UserCard { }          /* UpperCamelCase */
.userCard_avatar { }   /* BEM ではない */
.card.featured { }     /* Modifier を別クラスで表現 */
```

### Tailwind CSS を使用する場合：Utility-First

- ユーティリティクラスを直接 HTML / JSX に記述する。
- カスタムクラスは `@apply` を多用せず、コンポーネント分割で対応する。
- 繰り返しが多いパターンのみ `@apply` でまとめる。

```html
<!-- ✅ Good -->
<div class="flex items-center gap-4 rounded-lg border border-gray-200 p-4 shadow-sm">
  <img class="h-12 w-12 rounded-full object-cover" src="..." alt="...">
  <p class="text-sm font-medium text-gray-900">ユーザー名</p>
</div>

<!-- ❌ Bad -->
<div class="custom-card">  <!-- カスタムクラスに @apply で全部書く -->
```

```css
/* ❌ Bad（@apply の乱用） */
.custom-card {
  @apply flex items-center gap-4 rounded-lg border border-gray-200 p-4 shadow-sm;
}
```

---

## 2. セレクター規約

- ID セレクター（`#id`）はスタイリングに使用しない（JavaScript フックには使用可）。
- タグセレクター（`div`, `p` 等）の単独使用を避け、クラスと組み合わせる。
- セレクターの詳細度（specificity）はできる限り低く保つ。
- `!important` は原則使用しない。使用する場合は理由をコメントで明示する。

```css
/* ✅ Good */
.user-card { }
.nav-link:hover { }
.user-card > .user-card__avatar { }

/* ❌ Bad */
#user-card { }          /* ID セレクター */
div.user-card { }       /* 不必要なタグ修飾 */
.user-card { color: red !important; }  /* !important */
```

---

## 3. プロパティ記述順序

プロパティは以下のカテゴリ順に記述する。

1. **配置・表示**：`display`, `position`, `top/right/bottom/left`, `z-index`, `float`, `clear`
2. **ボックスモデル**：`width`, `height`, `margin`, `padding`, `border`, `border-radius`
3. **テキスト・フォント**：`font-family`, `font-size`, `font-weight`, `line-height`, `color`, `text-align`
4. **背景・装飾**：`background`, `box-shadow`, `opacity`
5. **トランジション・アニメーション**：`transition`, `animation`, `transform`
6. **その他**：`cursor`, `overflow`, `pointer-events`, `content`

```css
/* ✅ Good */
.button {
  /* 配置 */
  display: inline-flex;
  position: relative;
  /* ボックスモデル */
  padding: 8px 16px;
  border: 1px solid transparent;
  border-radius: 4px;
  /* テキスト */
  font-size: 14px;
  font-weight: 500;
  color: #fff;
  /* 背景 */
  background-color: #3b82f6;
  /* トランジション */
  transition: background-color 150ms ease;
  /* その他 */
  cursor: pointer;
}
```

---

## 4. Tailwind CSS 使用方針

- `tailwind.config` でデザイントークン（色・フォント・スペーシング等）を定義し、任意の値（`text-[14px]` 等）は最小限にする。
- クラスの記述順序は **レイアウト → ボックス → テキスト → 色・背景 → 状態** の順にする。
- `dark:` / `hover:` / `focus:` 等のバリアントは対象クラスに隣接させる。
- レスポンシブプレフィックスは `sm:` → `md:` → `lg:` → `xl:` の順に記述する。

```html
<!-- ✅ Good（記述順序） -->
<button class="
  flex items-center justify-center
  h-10 w-full px-4 rounded-md
  text-sm font-medium text-white
  bg-blue-600 hover:bg-blue-700
  focus:outline-none focus:ring-2 focus:ring-blue-500
  disabled:cursor-not-allowed disabled:opacity-50
  transition-colors
">
  送信
</button>
```

---

## 5. レスポンシブデザイン

- **モバイルファースト**で設計する（ベーススタイルはモバイル向け、ブレークポイントで拡張）。
- Tailwind の場合：プレフィックスなし → `sm:` → `md:` → `lg:` → `xl:` の順に記述する。
- 素の CSS の場合：`min-width` メディアクエリを使用する。

```html
<!-- ✅ Good（モバイルファースト） -->
<div class="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
```

```css
/* ✅ Good（モバイルファースト） */
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

/* ❌ Bad（デスクトップファースト） */
@media (max-width: 1024px) { .grid { grid-template-columns: repeat(2, 1fr); } }
@media (max-width: 640px)  { .grid { grid-template-columns: 1fr; } }
```

---

## 6. カスタムプロパティ（CSS Variables）

- デザイントークン（色・スペーシング・フォント等）は CSS カスタムプロパティで定義する。
- カスタムプロパティは `:root` に定義し、`--` プレフィックスで命名する。
- Tailwind を使う場合は `tailwind.config` でトークンを管理し、カスタムプロパティとの二重管理を避ける。

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

/* ❌ Bad（ハードコードされた値） */
.button {
  background-color: #3b82f6;
  border-radius: 6px;
  padding: 8px 16px;
}
```

---

## 7. コメント規約

- ファイルの先頭に目的・スコープを記述する。
- セクションの区切りにはコメントブロックを使用する。
- コードが自明でない箇所（ハック、ブラウザ対応等）にのみコメントを付ける。
- TODO / FIXME は `/* TODO: 説明 */` の形式で記述する。

```css
/* ==========================================================================
   Button コンポーネント
   ========================================================================== */

/* プライマリボタン */
.button--primary { }

/* セカンダリボタン */
.button--secondary { }

/* Safari 16 以下の gap バグに対応するためのハック */
.flex-container > * + * {
  margin-left: var(--spacing-sm);
}

/* TODO: ダークモード対応を追加する */
```

---

## 8. プロジェクト固有のルール

<!-- プロジェクトに応じて追記してください -->

- {プロジェクト固有のルールをここに記載}
