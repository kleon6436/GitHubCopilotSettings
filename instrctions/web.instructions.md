---
description: "Web アプリの開発ガイドライン"
applyTo: []
---

# プロジェクトガイドライン（Web アプリ開発用）

## プロジェクト概要

<!-- プロジェクトに応じて以下を記入してください -->

- **プロジェクト名**: {プロジェクト名}
- **概要**: {プロジェクトの目的・概要を簡潔に記述}
- **対象ブラウザ**: {例: Chrome 120+ / Firefox 120+ / Safari 17+ / Edge 120+}
- **レンダリング方式**: {CSR / SSR / SSG / ISR}（該当するものを選択）
- **リポジトリ構成**: {シングルレポ / モノレポ、主なディレクトリ構成の説明}

## 技術スタック

| カテゴリ | 技術 / ツール | バージョン | 備考 |
|---------|-------------|-----------|------|
| 言語 | TypeScript | 5 系 最新安定版 | strict モード有効 |
| UI フレームワーク | {例: React / Vue 3 / Svelte} | 最新安定版 | |
| メタフレームワーク | {例: Next.js / Nuxt / SvelteKit} | 最新安定版 | |
| スタイリング | {例: Tailwind CSS / CSS Modules} | 最新安定版 | |
| 状態管理 | {例: Zustand / Pinia / Jotai} | 最新安定版 | |
| データフェッチ | {例: TanStack Query / SWR} | 最新安定版 | |
| テスト | Vitest + Testing Library + Playwright | 最新安定版 | |
| フォーマッター | Prettier | 最新安定版 | |
| リンター | ESLint + typescript-eslint | 最新安定版 | |
| パッケージマネージャ | {例: pnpm / npm / yarn} | 最新安定版 | |
| CI/CD | {例: GitHub Actions + Vercel / Cloudflare Pages} | | |

## 推奨 Copilot agent 構成

- 複数 agent で進める場合は `agents/orchestrator.agent.md` を起点にする。
- 要件整理は `agents/product-manager.agent.md`、技術設計は `agents/architect.agent.md`、実装は `agents/developer.agent.md` を使い分ける。
- UI 設計や WCAG 2.2 AA 対応を伴う場合は `agents/ui-designer.agent.md` を併用する。
- 実装後は `agents/reviewer.agent.md` と `agents/tester.agent.md` を品質ゲートとして使う。
- デプロイ・インフラ設定は `agents/devops.agent.md` を使う。
- セキュリティレビューは `agents/security-reviewer.agent.md` を通す。

## UI ガイドライン

Web の UI 設計・実装については以下のスキルを参照すること。

- `skills/web-ui-guidelines/SKILL.md` — Web UI ガイドライン（セマンティックHTML・レスポンシブ・Core Web Vitals）
- `skills/ui-accessibility/SKILL.md` — アクセシビリティ共通原則
- `skills/ui-review-checklist/SKILL.md` — UI レビュー時のチェックリスト

## コーディング規約

- TypeScript: `skills/typescript-coding-standards/SKILL.md`
- JavaScript: `skills/javascript-coding-standards/SKILL.md`
- CSS: `skills/css-coding-standards/SKILL.md`
- React（使用する場合）: `skills/react-coding-standards/SKILL.md`

## アーキテクチャ方針

### ディレクトリ構成（例: Next.js App Router）

```
src/
  app/           — ルーティング・ページ・レイアウト
  components/    — 再利用可能な UI コンポーネント
  features/      — 機能ドメイン単位のモジュール
  hooks/         — カスタム Hooks
  lib/           — 外部ライブラリのラッパー・設定
  services/      — API クライアント・外部サービス
  stores/        — グローバル状態管理
  types/         — 型定義
  utils/         — 汎用ユーティリティ
```

### コンポーネント設計

- **Server Components** と **Client Components** を明確に分離する
- `"use client"` は本当に必要な箇所にのみ付与する
- コンポーネントは単一責任で設計し、100 行を超えたら分割を検討する
- Props の型は `interface` で定義し、任意プロパティには `?` を明示する

### データフェッチ

- サーバーサイドフェッチを優先し、クライアントサイドフェッチは UX 上必要な場合のみ使う
- `loading.tsx` / `error.tsx` / `not-found.tsx` を必ずセットで実装する
- API レスポンスは `zod` でバリデーションを行う

## テスト方針

- コンポーネントテストは **Vitest + Testing Library** でユーザー操作ベースで書く
- E2E テストは **Playwright** を使用する
- `data-testid` はテスト専用とし、ユーザーに見えないことを確認する
- スナップショットテストは UI の変更検知に使うが、過信しない

## Core Web Vitals 目標値

| 指標 | 目標 |
|------|------|
| LCP（Largest Contentful Paint） | 2.5 秒以下 |
| INP（Interaction to Next Paint） | 200 ms 以下 |
| CLS（Cumulative Layout Shift） | 0.1 以下 |

詳細は `skills/performance-optimization/SKILL.md` を参照すること。

## セキュリティ

- XSS 対策：ユーザー入力を DOM に直接挿入しない、`dangerouslySetInnerHTML` を避ける
- CSRF 対策：State 変更を伴うリクエストには CSRF トークンまたは SameSite Cookie を使う
- 環境変数：クライアントに公開する変数は `NEXT_PUBLIC_` のような明示的なプレフィックスで管理する
- Content Security Policy（CSP）を適切に設定する

詳細は `skills/security-practices/SKILL.md` を参照すること。

## 国際化（i18n）

- テキストは `next-intl` / `i18next` 等のライブラリで管理し、コード中にハードコードしない
- 翻訳ファイルは `messages/{locale}.json` 等で整理する
- 日付・数値・通貨フォーマットは `Intl` API または i18n ライブラリを使う
- RTL 言語対応のため、`dir` 属性と `logical properties` を使う

詳細は `skills/i18n-localization/SKILL.md` を参照すること。
