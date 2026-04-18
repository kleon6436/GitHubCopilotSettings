---
name: web-ui-guidelines
description: 'Web 向け UI ガイドライン。レスポンシブデザイン（モバイルファースト）、セマンティック HTML、CSS 設計（Flexbox / Grid / 論理プロパティ / Container Queries）、WCAG 2.2 AA、ARIA、フォーカス管理、Core Web Vitals（LCP / CLS / INP）を確認・適用したいときに使用。Use when: designing or implementing web UI; reviewing HTML/CSS/React/Vue components; applying accessibility and performance best practices.'
argument-hint: '確認したい項目（CSS / アクセシビリティ / パフォーマンス など、省略可）'
---

# Web UI ガイドライン

## 概要

このスキルは Web アプリ・Web サイト向け UI 規約を定義します。
モバイルファースト、セマンティック HTML、WCAG 2.2 AA 準拠、Core Web Vitals 最適化を軸にしています。

参考:
- [MDN Web Docs](https://developer.mozilla.org/)
- [WCAG 2.2](https://www.w3.org/TR/WCAG22/)
- [Web.dev](https://web.dev/)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)

---

## 1. 基本原則

- **セマンティック HTML ファースト。** `<div>` / `<span>` に頼る前に適切な要素（`<button>` / `<nav>` / `<article>` / `<section>` / `<header>` / `<main>` / `<footer>` / `<dialog>`）を使う。
- **プログレッシブエンハンスメント。** HTML → CSS → JS の順で機能を積み上げ、JS 無効でも主要機能が動く設計を心がける。
- **モバイルファースト。** 最小幅からデザインし、`min-width` メディアクエリで拡張する。
- **アクセシビリティは後付けしない。** 設計段階から WCAG 2.2 AA を前提にする。
- **パフォーマンスは機能。** Core Web Vitals を設計品質指標とする。

---

## 2. セマンティック HTML

### 推奨要素

| 用途 | 要素 |
|---|---|
| ページの主要ナビゲーション | `<nav>` |
| 本文の主要コンテンツ | `<main>`（1 ページに 1 つ） |
| 自己完結したコンテンツ | `<article>` |
| テーマでグループ化 | `<section>`（見出しを持つ） |
| 補助情報 | `<aside>` |
| ページ/セクションの導入 | `<header>` |
| 著作権・リンク等 | `<footer>` |
| インタラクティブアクション | `<button>`（`<div onclick>` は使わない） |
| 遷移 | `<a href>` |
| モーダル | `<dialog>` |
| 折りたたみ | `<details>` / `<summary>` |

### 見出し階層

- ページに `<h1>` は 1 つ。`<h2>`→`<h3>` と論理的に下る。
- スキップ（`<h2>` → `<h4>`）しない。
- スタイルのために見出しレベルを選ばない（スタイルは CSS で調整）。

### フォーム

```html
<label for="email">メールアドレス</label>
<input id="email" name="email" type="email" autocomplete="email" required
       aria-describedby="email-help" />
<p id="email-help">例: user@example.com</p>
```

- `<label>` は必須。`for` と `id` を紐付ける。
- `autocomplete` 属性を適切に設定（`email` / `current-password` / `one-time-code` 等）。
- `type` を正しく使う（`email` / `tel` / `url` / `number` / `date` / `search`）。モバイル最適キーボードが表示される。
- エラーは `aria-invalid="true"` + `aria-describedby` でエラーメッセージを関連付ける。

---

## 3. CSS 設計

### レイアウト手法

| 用途 | 第一選択 |
|---|---|
| 1 次元（横 or 縦） | **Flexbox** |
| 2 次元（グリッド） | **CSS Grid** |
| コンテナ依存の切替 | **Container Queries** |
| 親子インタラクション | `:has()` |

### 論理プロパティ

物理プロパティ（`margin-left` / `padding-right`）ではなく **論理プロパティ**（`margin-inline-start` / `padding-inline-end`）を使用し、多言語・RTL に対応。

```css
/* ❌ Bad */
.card { margin-left: 16px; padding-right: 24px; }

/* ✅ Good */
.card { margin-inline-start: 16px; padding-inline-end: 24px; }
```

### 最新単位・関数

- **`clamp()`** で流動的な値。

```css
h1 { font-size: clamp(1.5rem, 2.5vw + 1rem, 3rem); }
```

- **`dvh` / `svh` / `lvh`** — モバイルブラウザの UI 出入りに対応（`100vh` の代わり）。
- **`aspect-ratio`** で縦横比固定。
- **カラー関数** `color-mix()` / `oklch()` で一貫したパレット生成。

### Container Queries

```css
.card-container { container-type: inline-size; }

@container (min-width: 400px) {
    .card { display: grid; grid-template-columns: auto 1fr; }
}
```

### カスケードレイヤー

優先順位の競合を解消する `@layer`:

```css
@layer reset, base, components, utilities;
```

### カスタムプロパティ（トークン）

ハードコードではなくトークン化:

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

## 4. レスポンシブデザイン

### ブレークポイント（目安）

| 名称 | 最小幅 | 想定 |
|---|---|---|
| Mobile | 0 | 〜スマートフォン縦 |
| Tablet | 600px | タブレット・大型スマホ横 |
| Laptop | 1024px | ラップトップ・デスクトップ |
| Wide | 1440px | ワイドモニター |

```css
/* モバイルファースト */
.grid { display: grid; grid-template-columns: 1fr; gap: 16px; }

@media (min-width: 600px) {
    .grid { grid-template-columns: repeat(2, 1fr); }
}

@media (min-width: 1024px) {
    .grid { grid-template-columns: repeat(3, 1fr); }
}
```

- **Container Queries を優先**し、メディアクエリはレイアウトシェル（`<body>` 直下）に限定するのが現代的。
- タッチ最小ターゲット: **44×44 CSS ピクセル**（WCAG 2.5.5、Apple HIG / Material も同様）。

---

## 5. タイポグラフィ

- ベースフォントサイズ: **16px**（`html { font-size: 100% }`）。固定 px を避け `rem` で指定。
- 行の長さ: **45〜75 文字**（読みやすさの研究値）。`max-width: 65ch` を段落に適用。
- 行間: **1.5〜1.7**（本文）、見出しは **1.2〜1.3**。
- フォントは **WOFF2** + `font-display: swap` で読み込み、CLS を回避するため `size-adjust` / `ascent-override` でフォールバック調整。

```css
@font-face {
    font-family: "AppFont";
    src: url("/fonts/app.woff2") format("woff2");
    font-display: swap;
}
```

- フォントウェイトは可変フォント（Variable Font）を優先し、リクエスト数を減らす。

---

## 6. カラー

- コントラスト比 **WCAG AA**: 通常テキスト 4.5:1、大テキスト（18pt / 14pt bold 以上）3:1。
- `color-scheme: light dark` で OS テーマに追従。
- `light-dark()` 関数でテーマ分岐。
- 情報を **色のみで伝えない**（アイコン・テキスト・パターンを併用）。
- リンクは下線または明確な視覚差異で識別（色だけに頼らない）。

---

## 7. アクセシビリティ（ARIA / フォーカス）

### ARIA ルール

1. **ARIA より先にセマンティック HTML**。`<button>` で済むなら `role="button"` は不要。
2. ネイティブセマンティクスを壊さない。
3. すべてのインタラクティブ ARIA ウィジェットはキーボード操作可能。
4. `role="presentation"` / `aria-hidden="true"` でフォーカス可能要素を隠さない。

### よく使うパターン

```html
<!-- 開閉トグル -->
<button aria-expanded="false" aria-controls="menu">メニュー</button>
<ul id="menu" hidden>...</ul>

<!-- モーダル -->
<dialog aria-labelledby="dlg-title">
    <h2 id="dlg-title">確認</h2>
    ...
</dialog>

<!-- ライブリージョン -->
<div aria-live="polite" aria-atomic="true">保存しました</div>

<!-- ラベル関連付け -->
<button aria-label="閉じる">
    <svg aria-hidden="true">...</svg>
</button>
```

### フォーカス管理

- フォーカスリングを **消さない**（`:focus { outline: none }` 禁止）。代わりに **`:focus-visible`** でスタイリング。

```css
:focus-visible {
    outline: 2px solid var(--color-accent);
    outline-offset: 2px;
    border-radius: 4px;
}
```

- モーダル開放時はフォーカスをモーダル内へ移動し、Esc で閉じる。
- `tabindex="0"` は例外的に、`tabindex="-1"` はプログラム的フォーカス用のみ。`tabindex` に正の値は使わない。
- Skip Link（`<a href="#main">メインコンテンツへスキップ</a>`）を最初のフォーカスターゲットに。

詳細は `skills/ui-accessibility/SKILL.md` 参照。

---

## 8. モーション

- **`prefers-reduced-motion`** に必ず対応。

```css
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        transition-duration: 0.01ms !important;
    }
}
```

- トランジション持続時間: **150〜300ms** が基本。
- `transform` / `opacity` を優先（レイアウトシフトを起こさない GPU アクセラレーション）。`top` / `left` / `width` のアニメーションは避ける。
- `will-change` は必要時のみ。常時指定しない。

---

## 9. Core Web Vitals

| 指標 | 目標 | 対策 |
|---|---|---|
| **LCP**（Largest Contentful Paint） | ≤ 2.5s | ヒーロー画像に `fetchpriority="high"`、フォントプリロード、CDN |
| **CLS**（Cumulative Layout Shift） | ≤ 0.1 | 画像・iframe に `width` / `height`、フォントフォールバック調整、広告枠予約 |
| **INP**（Interaction to Next Paint） | ≤ 200ms | 長時間タスク分割、`requestIdleCallback`、不要な JS 削減 |

### 画像最適化

```html
<img src="hero.avif"
     srcset="hero-400.avif 400w, hero-800.avif 800w, hero-1600.avif 1600w"
     sizes="(max-width: 600px) 100vw, 50vw"
     width="1600" height="900"
     alt="..."
     loading="lazy"
     decoding="async" />
```

- フォーマット優先順: **AVIF > WebP > JPEG/PNG**。`<picture>` で段階的提供。
- 画像には必ず `width` / `height` を指定（CLS 防止）。
- ファーストビュー以外は `loading="lazy"`、ファーストビューは `fetchpriority="high"`。

### JS 最適化

- クリティカルでない JS は `defer` / `async`。
- モジュール分割 + 動的 `import()`。
- サードパーティスクリプトを監査（タグマネージャーは遅延）。

---

## 10. ダークモード

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

- `<meta name="theme-color">` をテーマごとに指定（モバイル URL バー連動）。

```html
<meta name="theme-color" content="#ffffff" media="(prefers-color-scheme: light)">
<meta name="theme-color" content="#1a1a1a" media="(prefers-color-scheme: dark)">
```

- 画像はダーク時に `filter: brightness(0.9)` などで調整、またはダーク版を `<picture>` で出し分け。

---

## 11. 国際化（i18n）

- `<html lang="ja">` を正しく設定。
- 日付・数値・通貨は **`Intl` API** を使用。
- RTL サポート: `dir="auto"` + 論理プロパティ。
- フォントは言語別フォールバック（CJK フォールバック順序に注意）。
- テキストは伸縮を許容（ドイツ語・フィンランド語は英語の 1.3 倍）。

---

## 12. コンポーネント設計

### クラス命名

- **BEM** または **Utility-First**（Tailwind 等）を選び統一。
- CSS Modules / Scoped CSS（Vue SFC）/ styled-components でスコープを限定。

### デザインシステム

- **デザイントークン**（色・間隔・フォントサイズ・影）を CSS カスタムプロパティで定義。
- プリミティブコンポーネント（Button / Input / Stack）→ 複合コンポーネント（Card / Dialog / Form）の 2 層構成。
- **Headless UI**（Radix / React Aria / Headless UI）でアクセシブルな動作を担保し、見た目のみカスタマイズする戦略を推奨。

---

## 13. 画像・メディア

- SVG はインライン化（アイコン）または `<img src=".svg">`。
- アイコンセットは **Lucide / Phosphor / Material Symbols** 等ライセンス明確なものを使用。
- 動画は `<video>` + `poster` + `preload="metadata"`。自動再生はミュート + `playsinline`。
- `<iframe>` には `title` 属性を必須設定。

---

## 14. セキュリティ関連（UI 視点）

- 外部リンク: `rel="noopener noreferrer"` を付与（`target="_blank"` 時）。
- ユーザー入力表示は必ずエスケープ（フレームワークの既定に任せ、`dangerouslySetInnerHTML` は最小限）。
- フォーム送信は CSRF トークン、`autocomplete="off"` は慎重に（パスワードマネージャ阻害）。

---

## 15. 推奨ツール

- **Lighthouse** / **PageSpeed Insights** — パフォーマンス・アクセシビリティ監査
- **axe DevTools** — アクセシビリティ検査
- **Stylelint** — CSS リント
- **ESLint** + **eslint-plugin-jsx-a11y** — JSX アクセシビリティ
- **Storybook** — コンポーネントカタログ
- **Playwright** — E2E / アクセシビリティテスト

---

## 関連スキル

- アクセシビリティ全般: `skills/ui-accessibility/SKILL.md`
- UI レビュー: `skills/ui-review-checklist/SKILL.md`
