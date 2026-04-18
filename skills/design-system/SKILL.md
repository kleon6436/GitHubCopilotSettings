---
name: design-system
description: 'デザインシステムとデザイントークン管理のガイドラインを参照・適用する。カラーシステム・タイポグラフィ・スペーシング・コンポーネント仕様・プラットフォーム間のブランド統一を確認・適用したいときに使用。Use when: establishing design tokens, color system, typography scale, spacing system, component specifications, cross-platform brand consistency.'
argument-hint: '確認・適用したいデザインシステムの項目（省略可）'
---

# デザインシステム・トークン管理 ガイドライン

## 概要

このスキルはデザインシステムとデザイントークン管理の規約を定義します。
プラットフォーム間でブランドの一貫性を保ちながら、各プラットフォームのデザイン言語にも適応するための実践的なガイドラインを提供します。

---

## 1. デザイントークンとは

デザイントークンは、カラー・タイポグラフィ・スペーシング等のデザイン上の決定事項を**言語に依存しない形式で管理する仕組み**です。
Single Source of Truth として Figma や JSON で定義し、各プラットフォームの形式に変換して使用します。

### トークンの階層

```
Foundation（基礎）
  └── Semantic（意味付き）
        └── Component（コンポーネント固有）
```

| 階層 | 例 | 用途 |
|------|----|----|
| **Foundation** | `color.blue.500 = #2563EB` | 原色パレット。直接使用しない |
| **Semantic** | `color.primary = color.blue.500` | 意味で参照。コンポーネントはここを使う |
| **Component** | `button.primary.background = color.primary` | コンポーネント固有の設定 |

---

## 2. カラーシステム

### ライト / ダークモード対応

```json
// tokens.json（Style Dictionary 形式）
{
  "color": {
    "background": {
      "primary": {
        "$value": { "light": "#FFFFFF", "dark": "#1C1C1E" },
        "$type": "color"
      },
      "secondary": {
        "$value": { "light": "#F2F2F7", "dark": "#2C2C2E" },
        "$type": "color"
      }
    },
    "text": {
      "primary": {
        "$value": { "light": "#000000", "dark": "#FFFFFF" },
        "$type": "color"
      },
      "secondary": {
        "$value": { "light": "#6B7280", "dark": "#9CA3AF" },
        "$type": "color"
      }
    },
    "interactive": {
      "primary": {
        "$value": { "light": "#2563EB", "dark": "#3B82F6" },
        "$type": "color"
      }
    }
  }
}
```

### アクセシビリティ要件（WCAG 2.2 AA）

| コントラスト比 | 用途 |
|--------------|------|
| 4.5 : 1 以上 | 通常テキスト（18px 未満） |
| 3 : 1 以上 | 大きなテキスト（18px 以上 / Bold 14px 以上）・UI コンポーネント |

詳細は `skills/ui-accessibility/SKILL.md` を参照すること。

---

## 3. タイポグラフィスケール

### スケール定義

| トークン | サイズ | Weight | 用途 |
|---------|-------|--------|------|
| `text.display` | 34px / 36px | Bold | ヒーロー見出し |
| `text.title1` | 28px / 34px | Regular / Bold | ページタイトル |
| `text.title2` | 22px / 28px | Regular / Bold | セクションタイトル |
| `text.title3` | 20px / 24px | Regular / Semibold | サブセクション |
| `text.headline` | 17px / 20px | Semibold | カード見出し |
| `text.body` | 17px / 16px | Regular | 本文 |
| `text.callout` | 16px / 15px | Regular | 補足情報 |
| `text.subheadline` | 15px / 14px | Regular | ラベル |
| `text.footnote` | 13px / 12px | Regular | 注釈 |
| `text.caption` | 12px / 11px | Regular | キャプション |

### プラットフォーム適応

| プラットフォーム | 推奨フォント | スケーリング |
|--------------|-----------|------------|
| iOS / macOS | SF Pro / SF Pro Rounded | Dynamic Type 対応 |
| Android | Roboto / Google Sans | sp 単位で定義 |
| Web | system-ui / Inter / Noto Sans | rem 単位で定義 |
| Windows | Segoe UI Variable | WinUI 3 TextBlock |

---

## 4. スペーシングシステム

### ベースグリッド: 4px

```
space-1  =  4px  — 極小（アイコン内余白）
space-2  =  8px  — 小（関連要素間）
space-3  = 12px  — 小中（コンパクトな余白）
space-4  = 16px  — 中（標準余白・パディング）
space-5  = 20px  — 中大
space-6  = 24px  — 大（セクション内余白）
space-8  = 32px  — 特大（セクション間余白）
space-10 = 40px  — 超大
space-12 = 48px  — ページレベルの余白
space-16 = 64px  — 最大余白
```

### 原則

- 隣接する **関連要素は小さい余白**、関連しない要素は**大きい余白**で分ける
- コンテンツとエッジの余白は `space-4`（16px）を基準にする
- プラットフォームのシステムの余白（Safe Area・Status Bar 等）を考慮する

---

## 5. コンポーネント仕様

### ボタン

| バリアント | 用途 | 見た目 |
|---------|------|-------|
| `Primary` | 主要アクション（1画面に1つを原則） | Filled・ブランドカラー |
| `Secondary` | 副次アクション | Outlined または Tonal |
| `Tertiary` | 補助的なアクション | Text only |
| `Destructive` | 削除・キャンセル等 | 赤色系 |
| `Ghost` | 最低優先度 | 背景なし・文字のみ |

```
最小タップターゲット: 44×44px（iOS HIG / WCAG 準拠）
```

### 状態設計（すべてのインタラクティブ要素に必須）

| 状態 | 定義 |
|------|------|
| **Default** | 通常表示 |
| **Hover** | マウスカーソルが上にある（Desktop） |
| **Pressed** | タップ / クリック中 |
| **Focused** | キーボードフォーカス（アクセシビリティ必須） |
| **Disabled** | 操作不可 |
| **Loading** | 処理中 |

---

## 6. トークン変換（Style Dictionary）

```js
// style-dictionary.config.js
module.exports = {
  source: ['tokens/**/*.json'],
  platforms: {
    // CSS Custom Properties (Web)
    css: {
      transformGroup: 'css',
      buildPath: 'src/styles/',
      files: [{ destination: 'tokens.css', format: 'css/variables' }],
    },
    // Swift (iOS / macOS)
    ios_swift: {
      transformGroup: 'ios-swift',
      buildPath: 'Sources/DesignSystem/',
      files: [{ destination: 'DesignTokens.swift', format: 'ios-swift/enum.swift' }],
    },
    // Android (Compose)
    android: {
      transformGroup: 'android',
      buildPath: 'app/src/main/res/values/',
      files: [{ destination: 'tokens.xml', format: 'android/resources' }],
    },
  },
};
```

---

## 7. Figma との連携

- Figma の **Variables** / **Styles** でトークンを管理し、開発と同期する
- **Tokens Studio**（Figma プラグイン）で JSON エクスポートを自動化する
- デザイナーと開発者が同じトークン名を使うことで、実装のズレを防ぐ

---

## 8. チェックリスト

- [ ] Foundation / Semantic / Component の 3 階層でトークンを定義している
- [ ] カラーにライト / ダークモード両方のバリューが定義されている
- [ ] テキストコントラストが WCAG 2.2 AA を満たしている
- [ ] スペーシングが 4px グリッドに沿っている
- [ ] すべてのインタラクティブコンポーネントに 6 つの状態が定義されている
- [ ] トークンが各プラットフォームの形式（CSS / Swift / XML）に自動変換されている
- [ ] Figma のトークン定義とコードが同期されている
