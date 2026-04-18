---
description: "Apple iOS, iPadOSアプリの開発ガイドライン"
applyTo: []
---

# プロジェクトガイドライン（iOS / iPadOS アプリ開発用）

## 前提

- 各セクションに **【iPhone のみ】**・**【iPad のみ】**・**【ユニバーサルのみ】** のタグが付いた指示は、「対応デバイス」の設定に応じて以下のとおり扱うこと。
  - **「iPhone のみ」** の場合: 【iPad のみ】【ユニバーサルのみ】タグの指示は無視する
  - **「iPad のみ」** の場合: 【iPhone のみ】【ユニバーサルのみ】タグの指示は無視する
  - **「ユニバーサル」** の場合: タグに関わらずすべての指示を適用する

## プロジェクト概要

<!-- プロジェクトに応じて以下を記入してください -->

- **プロジェクト名**: {プロジェクト名}
- **概要**: {プロジェクトの目的・概要を簡潔に記述}
- **対象プラットフォーム**: {iOS 26.0+ / iPadOS 26.0+ / iOS・iPadOS 26.0+（ユニバーサル）}（該当するものを選択）
- **対応デバイス**: {iPhone のみ / iPad のみ / iPhone + iPad（ユニバーサル）}（該当するものを選択）
- **最低 Deployment Target**: {iOS 26.0 / iPadOS 26.0}
- **リポジトリ構成**: {シングルレポ / モノレポ、主なディレクトリ構成の説明}

## 技術スタック

### 推奨

| カテゴリ | 技術 / ツール | バージョン | 備考 |
|---------|-------------|-----------|------|
| 言語 | Swift | 6 | |
| IDE | Xcode | 26 | |
| プロジェクト管理 | XcodeGen | 最新 | project.yml で管理 |
| パッケージマネージャ | Swift Package Manager | | |
| UI フレームワーク | SwiftUI | iOS 26 SDK | UIKit との混在は最小限に |
| UI フレームワーク（補助） | UIKit | | SwiftUI で対応不可な場合のみ |
| アーキテクチャ | MVC | | |
| テスト | XCTest / Swift Testing | | 両フレームワーク併用可 |
| リンター / フォーマッター | SwiftLint | 最新 | .swiftlint.yml で設定 |
| アイコン作成 | Icon Composer | Xcode 26 内蔵 | レイヤー構造のアイコンを作成 |
| CI/CD | {例: GitHub Actions} | | |

### 今後追加予定

| カテゴリ | 技術 / ツール | バージョン | 備考 |
|---------|-------------|-----------|------|
| ウィジェット | WidgetKit | | ホーム画面・ロック画面ウィジェット |
| システム連携 | App Intents | | Siri / Shortcuts 対応 |

## 推奨 Copilot agent 構成

- 複数 agent で進める場合は `agents/orchestrator.agent.md` を中心に運用する。
- 仕様整理は `agents/product-manager.agent.md`、技術設計は `agents/architect.agent.md`、実装は `agents/developer.agent.md` を基本とする。
- UI 変更や Liquid Glass 対応を含む場合は `agents/ui-designer.agent.md` を併用し、画面状態・視覚階層・アクセシビリティを先に詰める。
- 仕上げは `agents/reviewer.agent.md` と `agents/tester.agent.md` で品質ゲートを通す。

## UI ガイドライン

iOS / iPadOS の UI 設計・実装（HIG、Liquid Glass、ナビゲーション、サイズクラス、Dynamic Type、アイコン等）については以下のスキルを参照すること。

- `skills/apple-ui-guidelines/SKILL.md` — Apple プラットフォーム UI ガイドライン（iOS / iPadOS / macOS 共通）
- `skills/ui-accessibility/SKILL.md` — アクセシビリティ共通原則
- `skills/ui-review-checklist/SKILL.md` — UI レビュー時のチェックリスト


## コーディング規約

Swift のコーディング規約については `skills/swift-coding-standards/SKILL.md` を参照すること。
