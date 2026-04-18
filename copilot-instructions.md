# プロジェクトガイドライン

## 前提

- **回答は必ず日本語で行うこと。**
- コードの変更をする際、変更量が200行を超える可能性が高い場合は、事前に「この指示では変更量が200行を超える可能性がありますが、実行しますか？」とユーザーに確認をとること。
- 何か大きい変更を加える場合、まず何をするのか計画を立てた上で、ユーザーに「このような計画で進めようと思います。」と提案すること。
- 考えてから書くこと
- シンプル優先
- 必要な部分だけ触ること
- ゴール基準で動くこと
- 変更後、必ずコードレビューとテストを実施すること

## プロジェクト概要

<!-- プロジェクトに応じて以下を記入してください -->

- **プロジェクト名**: {プロジェクト名}
- **概要**: {プロジェクトの目的・概要を簡潔に記述}
- **対象プラットフォーム**: {iOS / Android / Web / Server 等}
- **リポジトリ構成**: {モノレポ / シングルレポ、主なディレクトリ構成の説明}

## 技術スタック

<!-- プロジェクトで使用する技術をプラットフォームごとに記入してください -->
<!-- 不要な行は削除し、必要に応じて行を追加してください -->

| カテゴリ | 技術 / ツール | バージョン | 備考 |
|---------|-------------|-----------|------|
| 言語 | {例: Swift} | {例: 5.9} | |
| IDE / エディタ | {例: Xcode} | {例: 15.0} | |
| プロジェクト管理 | {例: XcodeGen} | {例: 2.38} | |
| パッケージマネージャ | {例: Swift Package Manager} | | |
| UIフレームワーク | {例: SwiftUI} | | |
| アーキテクチャ | {例: MVVM} | | |
| テスト | {例: XCTest} | | |
| CI/CD | {例: GitHub Actions} | | |
| リンター / フォーマッター | {例: SwiftLint} | | |
| その他 | | | |

## 推奨 Copilot agent 構成

- オーケストレーションパターンで進める場合は `agents/` 配下のテンプレートを利用する。
- 中核は `agents/orchestrator.agent.md` とし、要件整理は `product-manager`、技術設計は `architect`、実装は `developer`、レビューは `reviewer`、テストは `tester` を使い分ける。
- UI を伴う作業では `agents/ui-designer.agent.md` も組み合わせ、情報設計・状態設計・アクセシビリティ観点を早い段階で入れる。
- CI/CD・インフラ・デプロイ設定には `agents/devops.agent.md` を使う。
- セキュリティリスクの評価・脆弱性レビューには `agents/security-reviewer.agent.md` を通す。

### agent 一覧（9種類）

| agent | 主な責務 |
|-------|---------|
| `orchestrator` | 要件分解・委譲・品質ゲート管理・統合 |
| `product-manager` | 要件整理・受け入れ条件・優先順位 |
| `architect` | 技術設計・責務分割・移行戦略 |
| `developer` | 実装・修正・リファクタリング |
| `ui-designer` | 情報設計・状態設計・アクセシビリティ |
| `reviewer` | コードレビュー・リスク指摘・保守性評価 |
| `tester` | テスト計画・ケース設計・品質リスク評価 |
| `devops` | CI/CD・インフラ・デプロイ・監視 |
| `security-reviewer` | セキュリティ脅威分析・脆弱性評価・コンプライアンス |

## プラットフォーム別ガイドライン

各プラットフォームの詳細な開発ガイドラインは以下の指示ファイルを使用すること。

| プラットフォーム | 指示ファイル |
|--------------|-------------|
| iOS / iPadOS | `instrctions/ios.instructions.md` |
| macOS | `instrctions/macos.instructions.md` |
| Android | `instrctions/android.instructions.md` |
| Web | `instrctions/web.instructions.md` |
| Windows | `instrctions/windows.instructions.md` |
| クロスプラットフォーム | `instrctions/cross-platform.instructions.md` |

## スキル一覧

| カテゴリ | スキル |
|---------|-------|
| **コーディング規約** | `swift-coding-standards` / `kotlin-coding-standards` / `typescript-coding-standards` / `javascript-coding-standards` / `python-coding-standards` / `cpp-coding-standards` / `csharp-coding-standards` / `rust-coding-standards` / `css-coding-standards` / `react-coding-standards` |
| **UI / UX** | `apple-ui-guidelines` / `android-ui-guidelines` / `web-ui-guidelines` / `windows-ui-guidelines` / `ui-accessibility` / `ui-review-checklist` / `design-system` |
| **品質・セキュリティ** | `security-practices` / `cicd-deployment` / `performance-optimization` / `apple-app-store-submission` |
| **国際化** | `i18n-localization` |