---
description: "Windows アプリの開発ガイドライン"
applyTo: []
---

# プロジェクトガイドライン（Windows アプリ開発用）

## プロジェクト概要

<!-- プロジェクトに応じて以下を記入してください -->

- **プロジェクト名**: {プロジェクト名}
- **概要**: {プロジェクトの目的・概要を簡潔に記述}
- **対象プラットフォーム**: Windows 11 以上（Windows 10 サポートは任意）
- **最低対応バージョン**: {Windows 11 22H2 / Windows 10 1903}（該当するものを選択）
- **配布方式**: {Microsoft Store / サイドローディング / MSIX / Win32インストーラー}（選択）
- **リポジトリ構成**: {シングルレポ / モノレポ、主なディレクトリ構成の説明}

## 技術スタック

| カテゴリ | 技術 / ツール | バージョン | 備考 |
|---------|-------------|-----------|------|
| 言語 | C# | 最新安定版 | nullable 有効 |
| IDE | Visual Studio / VS Code | 最新安定版 | |
| UI フレームワーク | WinUI 3 | 最新安定版 | Windows App SDK 使用 |
| SDK | Windows App SDK | 最新安定版 | |
| アーキテクチャ | MVVM | | CommunityToolkit.Mvvm 推奨 |
| 非同期処理 | async/await | | |
| テスト | MSTest v3 / xUnit | 最新安定版 | |
| パッケージ管理 | NuGet | | |
| リンター / フォーマッター | Roslyn Analyzers / EditorConfig | | |
| CI/CD | {例: GitHub Actions} | | |

## 推奨 Copilot agent 構成

- 複数 agent で進める場合は `agents/orchestrator.agent.md` を起点にする。
- 要件整理は `agents/product-manager.agent.md`、技術設計は `agents/architect.agent.md`、実装は `agents/developer.agent.md` を使い分ける。
- Fluent Design や WinUI 3 コンポーネントの検討には `agents/ui-designer.agent.md` を併用する。
- 実装後は `agents/reviewer.agent.md` と `agents/tester.agent.md` を品質ゲートとして使う。
- パッケージング・配布パイプラインは `agents/devops.agent.md` を使う。

## UI ガイドライン

Windows の UI 設計・実装（Fluent Design、WinUI 3、ナビゲーション、タイトルバー統合等）については以下のスキルを参照すること。

- `skills/windows-ui-guidelines/SKILL.md` — Windows UI ガイドライン（Fluent Design / WinUI 3）
- `skills/mfc-ui-guidelines/SKILL.md` — MFC アプリケーション UI ガイドライン
- `skills/ui-accessibility/SKILL.md` — アクセシビリティ共通原則
- `skills/ui-review-checklist/SKILL.md` — UI レビュー時のチェックリスト

## コーディング規約

C# のコーディング規約については `skills/csharp-coding-standards/SKILL.md` を参照すること。

## アーキテクチャ方針

### レイヤー構成（MVVM）

```
Views/           — XAML ページ・UserControl・ダイアログ
ViewModels/      — ViewModel（INotifyPropertyChanged）
Models/          — ドメインモデル・DTO
Services/        — ビジネスロジック・外部 API
Repository/      — データアクセス層
```

### MVVM の運用

- `CommunityToolkit.Mvvm` の `ObservableObject` / `RelayCommand` を活用する
- `[ObservableProperty]` ソースジェネレータを使い、ボイラープレートを削減する
- View は ViewModel を介してのみデータを取得し、コードビハインドにロジックを書かない
- ViewModel は View に直接参照を持たない

### 依存注入

- `Microsoft.Extensions.DependencyInjection` を使用する
- `App.xaml.cs` または `Program.cs` で DI コンテナを構成する

## ウィンドウ管理

- マルチウィンドウは `Microsoft.UI.Xaml.Window` で管理する
- ウィンドウ状態（位置・サイズ）は `ApplicationData.Current.LocalSettings` で永続化する
- タイトルバー統合には `ExtendsContentIntoTitleBar` を使い、アプリコンテンツを最大限活用する

## 入力対応

- **マウス / キーボード** を主軸に設計し、タッチ・ペンはオプションとして対応する
- キーボードショートカットは `KeyboardAccelerator` で定義し、`ToolTip` に表示する
- タッチ対応では最小タップターゲットを 44×44px とする

## テスト方針

- ViewModel・サービスは **MSTest v3** または **xUnit** で単体テスト
- UI テストは **Appium + WinAppDriver** を検討する
- テスト可能性のために、サービスはインターフェースを介して依存を持つ

## パッケージングと配布

- 配布形式は **MSIX** パッケージを推奨する
- Microsoft Store 配布の場合は Store アソシエーションを早期に設定する
- バージョンは `Package.appxmanifest` の `Version` 属性でセマンティックバージョニングに従う

## セキュリティ

- 機密データは `PasswordVault`（Windows Credential Manager）で保存する
- アプリデータは `ApplicationData.Current.LocalFolder` に格納し、適切なパーミッションを設定する
- ネットワーク通信は HTTPS のみ許可する

詳細は `skills/security-practices/SKILL.md` を参照すること。

## 国際化（i18n）

- テキストは `Resources.resw` で管理し、コード中にハードコードしない
- `ResourceLoader` を介してリソースを読み込む
- 日付・数値・通貨は `Windows.Globalization` API を使う

詳細は `skills/i18n-localization/SKILL.md` を参照すること。
