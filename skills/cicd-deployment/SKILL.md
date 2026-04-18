---
name: cicd-deployment
description: 'CI/CD パイプラインとデプロイメント戦略のガイドラインを参照・適用する。GitHub Actions・ビルド自動化・テスト自動化・ステージング環境・カナリアリリース・ロールバック・プラットフォーム別配布を確認・適用したいときに使用。Use when: setting up CI/CD pipelines, automated testing, deployment strategies, release management, rollback procedures.'
argument-hint: '確認・適用したい CI/CD / デプロイメントの項目（省略可）'
---

# CI/CD・デプロイメント ガイドライン

## 概要

このスキルは CI/CD パイプラインとデプロイメント戦略の規約を定義します。
自動化・品質ゲート・安全なリリースを実現するための実践的なガイドラインを提供します。

---

## 1. ブランチ戦略

### 推奨: GitHub Flow（シンプル）

```
main           — 本番環境に直結。常にデプロイ可能な状態を保つ
feature/*      — 機能開発ブランチ。main からフォークし、PR でマージ
hotfix/*       — 本番障害対応。main から直接フォーク
```

### 大規模プロジェクト: Git Flow

```
main           — 本番リリース済みのコード
develop        — 次リリースの統合ブランチ
feature/*      — 機能開発（develop からフォーク）
release/*      — リリース準備（develop からフォーク → main にマージ）
hotfix/*       — 本番障害対応（main からフォーク）
```

### ブランチ保護ルール（main）

- **直接プッシュを禁止** する
- **PR + レビュー承認**（最低 1 名）を必須にする
- **CI 通過を必須**にする
- **マージ前に最新化**（Require branches to be up to date）

---

## 2. CI パイプライン構成

### 基本ステージ

```yaml
# .github/workflows/ci.yml（例）
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      - name: Lint
        run: pnpm lint
      - name: Type check
        run: pnpm typecheck
      - name: Unit tests
        run: pnpm test --coverage
      - name: Build
        run: pnpm build
```

### 品質ゲート（必須）

| チェック | 説明 |
|---------|------|
| **Lint** | コードスタイル・静的解析 |
| **型チェック** | コンパイルエラーがないこと |
| **単体テスト** | カバレッジ閾値を下回らないこと |
| **ビルド** | エラーなくビルドが完了すること |
| **セキュリティスキャン** | 依存ライブラリの既知脆弱性チェック |

### テストカバレッジ目標

| 種別 | 目標 |
|------|------|
| Statements | 80% 以上 |
| Branches | 70% 以上 |
| Functions | 80% 以上 |
| Lines | 80% 以上 |

---

## 3. プラットフォーム別 CI/CD

### iOS / macOS（Xcode Cloud または GitHub Actions）

```yaml
# GitHub Actions の例
jobs:
  ios-build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install dependencies
        run: bundle install  # Fastlane
      - name: Run tests
        run: bundle exec fastlane test
      - name: Build & Archive
        run: bundle exec fastlane build
      - name: Upload to TestFlight
        run: bundle exec fastlane beta
        env:
          APP_STORE_CONNECT_API_KEY: ${{ secrets.ASC_API_KEY }}
```

**Fastlane 推奨レーン:**
- `test` — 単体テスト・UI テスト実行
- `beta` — TestFlight へアップロード
- `release` — App Store へ提出

### Android（GitHub Actions）

```yaml
jobs:
  android-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup JDK
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run tests
        run: ./gradlew test
      - name: Build release APK/AAB
        run: ./gradlew bundleRelease
        env:
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
      - name: Upload to Google Play
        uses: r0adkll/upload-google-play@v1
```

### Web（Vercel / Cloudflare Pages / AWS）

```yaml
jobs:
  web-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install & Build
        run: pnpm install && pnpm build
      - name: E2E tests
        run: pnpm playwright test
      - name: Deploy to production
        run: vercel --prod
        env:
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
```

---

## 4. 環境管理

### 環境の定義

| 環境 | 用途 | ブランチ | 自動デプロイ |
|------|------|---------|------------|
| **development** | ローカル開発 | feature/* | — |
| **staging** | 品質確認・レビュー | develop / PR | ✅ |
| **production** | 本番 | main | ✅（承認後） |

### 環境変数の管理

- シークレットは GitHub Actions Secrets または専用の Secrets Manager（AWS Secrets Manager / GCP Secret Manager）で管理する
- 環境ごとに異なる値は `Environment` を使って管理する
- `.env.example` をリポジトリに含め、必要な変数を文書化する

---

## 5. デプロイメント戦略

### カナリアリリース（Web）

新機能を全ユーザーに一気に展開せず、段階的に展開する。

```
5% のユーザーにリリース → メトリクス確認 → 25% → 50% → 100%
問題が発生したら即座にロールバック
```

### フィーチャーフラグ

- 機能の ON/OFF を**コードではなく設定**で制御する
- 段階的ロールアウト・A/B テスト・緊急停止に活用する
- ツール例: LaunchDarkly / Unleash / 自前実装

### ブルーグリーンデプロイ

```
現在の本番（Blue）を維持しながら新バージョン（Green）をデプロイ
↓
Green が正常なことを確認
↓
トラフィックを Green に切り替え
↓
問題があれば Blue に即座に戻す
```

---

## 6. ロールバック手順

### 準備

- **タグを必ず打つ** — `v1.2.3` 形式で本番リリースごとにタグを付ける
- **データベースマイグレーション** は前方互換にする（古いコードでも動作するようにする）

### ロールバック実行

```bash
# Git でタグに戻す
git checkout v1.2.2

# または GitHub Releases から前バージョンのアーティファクトを再デプロイ
```

### iOS / Android

- TestFlight / Firebase App Distribution から前バージョンを再配布
- App Store / Google Play のロールアウトを停止または戻す

---

## 7. モニタリングとアラート

### 必須メトリクス

| メトリクス | ツール例 |
|---------|---------|
| エラー率・クラッシュレート | Sentry / Crashlytics / Firebase Crashlytics |
| レスポンスタイム / レイテンシ | Datadog / New Relic / CloudWatch |
| Core Web Vitals | Google Search Console / Vercel Analytics |
| アプリ評価・レビュー | App Store Connect / Google Play Console |

### アラート設定

- エラー率が平常の **2 倍以上**になったらアラートを発火する
- P95 レイテンシが閾値を超えたらアラートを発火する
- クリティカルなアラートは **Slack / PagerDuty** に通知する

---

## 8. セキュリティスキャン

### 依存ライブラリの脆弱性チェック

```yaml
# GitHub Dependabot を有効にする（.github/dependabot.yml）
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    reviewers:
      - "security-team"
```

### シークレットスキャン

- **GitHub Secret Scanning** を有効にする（無料）
- CI で `truffleHog` または `gitleaks` を実行する

---

## 9. チェックリスト

- [ ] main ブランチに直接プッシュ禁止のルールが設定されている
- [ ] PR には CI パス + レビュー承認が必須になっている
- [ ] 本番デプロイには承認フロー（manual approval）が設定されている
- [ ] シークレットは Secrets Manager で管理されており、コードにない
- [ ] テストカバレッジの閾値が設定されている
- [ ] ロールバック手順が文書化されている
- [ ] Dependabot（または同等ツール）が有効になっている
- [ ] エラー監視・アラートが設定されている
