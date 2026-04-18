---
description: "Apple macOSアプリの開発ガイドライン"
applyTo: []
---

# プロジェクトガイドライン（macOS Tahoe アプリ開発用）

## プロジェクト概要

<!-- プロジェクトに応じて以下を記入してください -->

- **プロジェクト名**: {プロジェクト名}
- **概要**: {プロジェクトの目的・概要を簡潔に記述}
- **対象プラットフォーム**: macOS 26（macOS Tahoe）以上
- **最低 Deployment Target**: macOS 26.0
- **リポジトリ構成**: {シングルレポ / モノレポ、主なディレクトリ構成の説明}

## 技術スタック

| カテゴリ | 技術 / ツール | バージョン | 備考 |
|---------|-------------|-----------|------|
| 言語 | Swift | 6 | |
| IDE | Xcode | 26 | |
| プロジェクト管理 | XcodeGen | 最新 | project.yml で管理 |
| パッケージマネージャ | Swift Package Manager | | |
| UI フレームワーク | SwiftUI | macOS 26 SDK | AppKit との混在は最小限に |
| アーキテクチャ | MVC | | |
| テスト | XCTest / Swift Testing | | 両フレームワーク併用可 |
| リンター / フォーマッター | SwiftLint | 最新 | .swiftlint.yml で設定 |
| アイコン作成 | Icon Composer | Xcode 26 内蔵 | レイヤー構造のアイコンを作成 |
| CI/CD | {例: GitHub Actions} | | |

## 推奨 Copilot agent 構成

- 複数 agent で進める場合は `agents/orchestrator.agent.md` を起点にする。
- 要件整理は `agents/product-manager.agent.md`、技術設計は `agents/architect.agent.md`、実装は `agents/developer.agent.md` を使い分ける。
- UI や HIG / Liquid Glass の検討を伴う場合は `agents/ui-designer.agent.md` を併用し、情報設計とアクセシビリティを早めに固める。
- 実装後は `agents/reviewer.agent.md` と `agents/tester.agent.md` を品質ゲートとして使う。

## UI ガイドライン

macOS の UI 設計・実装（HIG、Liquid Glass、ウィンドウ、ナビゲーション、キーボードショートカット、アイコン等）については以下のスキルを参照すること。

- `skills/apple-ui-guidelines/SKILL.md` — Apple プラットフォーム UI ガイドライン（iOS / iPadOS / macOS 共通）
- `skills/ui-accessibility/SKILL.md` — アクセシビリティ共通原則
- `skills/ui-review-checklist/SKILL.md` — UI レビュー時のチェックリスト


## コーディング規約

Swift のコーディング規約については `skills/swift-coding-standards/SKILL.md` を参照すること。
