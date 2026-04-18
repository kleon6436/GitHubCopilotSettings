---
description: "Android アプリの開発ガイドライン"
applyTo: []
---

# プロジェクトガイドライン（Android アプリ開発用）

## プロジェクト概要

<!-- プロジェクトに応じて以下を記入してください -->

- **プロジェクト名**: {プロジェクト名}
- **概要**: {プロジェクトの目的・概要を簡潔に記述}
- **対象プラットフォーム**: Android 10（API 29）以上
- **最低 minSdk**: {29}
- **ターゲット SDK**: {最新安定版}
- **リポジトリ構成**: {シングルレポ / モノレポ、主なディレクトリ構成の説明}

## 技術スタック

| カテゴリ | 技術 / ツール | バージョン | 備考 |
|---------|-------------|-----------|------|
| 言語 | Kotlin | 最新安定版 | |
| IDE | Android Studio | 最新安定版 | |
| ビルドシステム | Gradle（KTS） | | build.gradle.kts を使用 |
| UI フレームワーク | Jetpack Compose | 最新安定版 | View システムとの混在は最小限に |
| DI | Hilt | 最新安定版 | |
| ナビゲーション | Navigation Compose | 最新安定版 | |
| 非同期処理 | Kotlin Coroutines / Flow | 最新安定版 | |
| ネットワーク | Retrofit + OkHttp | 最新安定版 | |
| アーキテクチャ | MVVM + Clean Architecture | | |
| テスト | JUnit 5 / Espresso / Compose Testing | 最新安定版 | |
| リンター | ktlint / Detekt | 最新安定版 | |
| CI/CD | {例: GitHub Actions} | | |

## 推奨 Copilot agent 構成

- 複数 agent で進める場合は `agents/orchestrator.agent.md` を起点にする。
- 要件整理は `agents/product-manager.agent.md`、技術設計は `agents/architect.agent.md`、実装は `agents/developer.agent.md` を使い分ける。
- UI や Material Design 3 の検討を伴う場合は `agents/ui-designer.agent.md` を併用し、情報設計とアクセシビリティを早めに固める。
- 実装後は `agents/reviewer.agent.md` と `agents/tester.agent.md` を品質ゲートとして使う。
- CI/CD 設定や配布パイプラインは `agents/devops.agent.md` を使う。

## UI ガイドライン

Android の UI 設計・実装（Material Design 3、Jetpack Compose、ナビゲーション、Large Screen 対応等）については以下のスキルを参照すること。

- `skills/android-ui-guidelines/SKILL.md` — Android UI ガイドライン（Material Design 3 / Jetpack Compose）
- `skills/ui-accessibility/SKILL.md` — アクセシビリティ共通原則
- `skills/ui-review-checklist/SKILL.md` — UI レビュー時のチェックリスト

## コーディング規約

Kotlin のコーディング規約については `skills/kotlin-coding-standards/SKILL.md` を参照すること。

## アーキテクチャ方針

### レイヤー構成（Clean Architecture）

```
presentation/   — ViewModel・UI State・Compose Screen
domain/         — UseCase・Repository Interface・Entity
data/           — Repository 実装・DataSource・API Client
```

### 状態管理

- `ViewModel` に UI State を集約し、`StateFlow` で Compose に流す
- 副作用（ナビゲーション・スナックバー等）は `SharedFlow` または `Channel` で通知する
- `MutableStateFlow` は private とし、外部には読み取り専用の `StateFlow` のみ公開する

### 依存関係

- DI には必ず Hilt を使用する
- `@Singleton` は本当に共有すべきスコープにのみ付与する
- `@HiltViewModel` で ViewModel に依存注入する

## テスト方針

- ViewModel・UseCase は **JUnit 5 + MockK** で単体テスト
- Compose UI は **Compose Testing** で結合テスト
- E2E は **Espresso** または **UI Automator** を用いる
- テスト可能性のために、ViewModel・UseCase はインターフェースを介して依存を持つ

## Large Screen / フォルダブル対応

- `WindowSizeClass` を使ってレイアウトを切り替える
- コンパクト：シングルパネル、ミディアム以上：ツーパネル構成を基本とする
- フォルダブル端末では `FoldingFeature` を考慮したレイアウト分岐を行う
- `Modifier.fillMaxSize()` を安易に使わず、Adaptive レイアウトを優先する

## セキュリティ

- 機密データは `EncryptedSharedPreferences` または `DataStore` + AES 暗号化で保存する
- API キー・シークレットは `local.properties` で管理し、リポジトリにコミットしない
- ネットワーク通信は HTTPS のみ許可し、証明書ピンニングを検討する
- `ProGuard / R8` の難読化ルールを設定する

詳細は `skills/security-practices/SKILL.md` を参照すること。

## 国際化（i18n）

- テキストは必ず `strings.xml` で管理し、コード中にハードコードしない
- RTL 言語対応のため、`start/end` を `left/right` より優先して使用する
- 複数形は `plurals` を使う

詳細は `skills/i18n-localization/SKILL.md` を参照すること。

## パフォーマンス

- `remember` / `derivedStateOf` / `key` を適切に使い、不要な Recomposition を避ける
- 画像の読み込みには **Coil** を使い、`AsyncImage` でキャッシュを活用する
- リストには `LazyColumn` / `LazyRow` を使用し、`items(key = ...)` を指定する
- ANR 防止のため、ブロッキング処理は必ず `Dispatchers.IO` または `Dispatchers.Default` で実行する

詳細は `skills/performance-optimization/SKILL.md` を参照すること。
