---
name: apple-app-store-submission
description: 'Apple App Store（iOS / macOS）へのアプリ提出・審査・公開プロセスのガイドライン。App Store Connect 設定、メタデータ・スクリーンショット準備、App Review Guidelines 対応、プライバシー・Export Compliance、TestFlight ベータテスト、バージョン管理・リリース戦略を確認・適用したいときに使用。Use when: submitting apps to the Apple App Store, preparing App Store metadata and screenshots, responding to App Review rejections, configuring App Store Connect, managing TestFlight beta testing, handling privacy compliance and export regulations.'
argument-hint: '確認・適用したい提出プロセスの項目（省略可）'
---

# Apple App Store 提出ガイドライン（iOS / macOS）

## 概要

このスキルは iOS / macOS アプリを Apple App Store に公開するための提出・審査・公開プロセスの規約を定義します。
App Store Connect の設定からメタデータ準備、App Review Guidelines への対応、リリース管理までの実践的なガイドラインを提供します。

> **CI/CD との棲み分け**: ビルド自動化・Fastlane・Xcode Cloud によるパイプライン構築は `cicd-deployment` スキルを参照してください。本スキルは提出準備・審査対応・公開管理に特化します。

---

## 1. 提出前準備

### 1.1 App Store Connect アカウント

| 項目 | 要件 |
|------|------|
| Apple Developer Program | 年間 $99（個人 / 組織） |
| 役割 | App Manager 以上の権限が必要 |
| チーム設定 | 組織アカウントの場合、D-U-N-S ナンバーが必要 |
| 契約 | 有料アプリ契約・税務情報・銀行口座情報の登録（有料アプリ / IAP がある場合） |

### 1.2 Bundle ID・App ID

- Bundle ID は逆ドメイン形式を使用: `com.example.appname`
- Apple Developer Portal で App ID を登録
- Explicit App ID を使用（Wildcard は App Store 提出に使用不可）
- 一度 App Store に提出した Bundle ID は変更不可

### 1.3 証明書・Provisioning Profile

| 種類 | 用途 |
|------|------|
| Apple Distribution 証明書 | App Store / TestFlight 用のコード署名 |
| Provisioning Profile (App Store) | App Store 配布用プロファイル |

- 証明書の有効期限を定期的に確認する
- 自動署名（Xcode Automatically manage signing）の使用を推奨
- CI/CD 環境では手動署名 + Keychain 管理が必要（詳細は `cicd-deployment` スキル参照）

### 1.4 Capabilities・Entitlements

- 使用する Capability を Apple Developer Portal と Xcode の両方で有効化
- 未使用の Capability を含めない（審査でリジェクトの原因になる）
- 主要な Capabilities:

| Capability | 注意事項 |
|-----------|---------|
| Push Notifications | APNs 証明書 or Key の設定が必要 |
| Sign in with Apple | サードパーティログインがある場合は必須 |
| App Groups | アプリ間データ共有・ウィジェット連携時 |
| Associated Domains | Universal Links・Handoff 利用時 |
| HealthKit | 健康データアクセスには追加の審査基準あり |
| BackgroundModes | 使用するモードの正当性を説明できること |

---

## 2. App Store Connect 設定

### 2.1 アプリ基本情報

| 項目 | ガイドライン |
|------|------------|
| アプリ名 | 最大 30 文字。商標侵害・汎用的すぎる名前を避ける |
| サブタイトル | 最大 30 文字。アプリの簡潔な説明 |
| プライマリカテゴリ | アプリの主要な機能に最も合致するカテゴリを選択 |
| セカンダリカテゴリ | 任意。補助的なカテゴリ |
| コンテンツレーティング | App Rating の質問に正確に回答する |

### 2.2 価格・配信

- 価格帯（Price Tier）を設定。無料アプリでも設定が必要
- 配信する国・地域を選択（デフォルトは全地域）
- 先行予約（Pre-Order）を利用する場合は最大 180 日前から設定可能
- Universal Purchase を有効にすると iOS / macOS で 1 回の購入で両方利用可能

### 2.3 アプリ内課金（IAP）・サブスクリプション

| 種類 | 説明 |
|------|------|
| Consumable | 消費型（例: ゲーム内通貨） |
| Non-Consumable | 非消費型（例: 追加機能の永久解除） |
| Auto-Renewable Subscription | 自動更新サブスクリプション |
| Non-Renewing Subscription | 非自動更新サブスクリプション |

- IAP を使用する場合、App Store Connect で商品を事前登録
- サブスクリプションの場合、サブスクリプショングループを適切に設定
- Restore Purchases 機能の実装は必須（Guideline 3.1.1）
- StoreKit 2 の使用を推奨

### 2.4 追加ターゲット

| ターゲット | 設定のポイント |
|-----------|--------------|
| App Clip | App Clip カードのメタデータを App Store Connect で設定 |
| ウィジェット | メインアプリに含めて提出。個別の提出は不要 |
| iMessage Extension | メインアプリに含めて提出 |
| watchOS App | メインアプリのコンパニオンまたは独立アプリとして提出 |

---

## 3. メタデータ・アセット準備

### 3.1 説明文・キーワード

| 項目 | 要件 | ガイドライン |
|------|------|------------|
| 説明文 | 最大 4,000 文字 | 最初の 1〜3 行が最重要（折りたたみ前に表示）。主要機能を冒頭に記載 |
| キーワード | 最大 100 文字（カンマ区切り） | 競合名・無関係な語を含めない。単数形のみ使用しスペースを節約 |
| What's New | 最大 4,000 文字 | バージョンごとの主要な変更点を簡潔に記載 |
| プロモーションテキスト | 最大 170 文字 | 審査不要で随時更新可能。キャンペーン告知等に活用 |
| サポート URL | 必須 | 有効なサポートページの URL |
| マーケティング URL | 任意 | アプリの公式サイト |

### 3.2 スクリーンショット要件

#### 【iOS】

| デバイス | サイズ（px） | 必須 |
|---------|------------|------|
| iPhone 6.9" (Pro Max) | 1320 × 2868 | ✅ |
| iPhone 6.7" | 1290 × 2796 | ✅ |
| iPhone 6.5" | 1284 × 2778 または 1242 × 2688 | ✅ |
| iPhone 5.5" | 1242 × 2208 | ✅ |
| iPad Pro 13" (M4) | 2064 × 2752 | iPad 対応アプリの場合 ✅ |
| iPad Pro 12.9" (6th gen) | 2048 × 2732 | iPad 対応アプリの場合 ✅ |

#### 【macOS】

| サイズ（px） | 備考 |
|------------|------|
| 1280 × 800 以上（最大 2560 × 1600） | 最低 1 枚、最大 10 枚 |

#### 共通ルール

- 各デバイスサイズにつき最低 1 枚、最大 10 枚
- PNG または JPEG 形式（透過なし）
- ステータスバーに個人情報を含めない
- ローカライズ対象の言語ごとにスクリーンショットを用意することを推奨

### 3.3 App プレビュー動画

| 項目 | 要件 |
|------|------|
| 長さ | 15〜30 秒 |
| 形式 | H.264 / HEVC、MOV |
| 解像度 | 各デバイスのスクリーンショットサイズに合わせる |
| 音声 | 任意（デフォルトはミュート再生） |
| 枚数 | 各デバイスサイズにつき最大 3 本 |

### 3.4 アプリアイコン

| 項目 | 要件 |
|------|------|
| サイズ | 1024 × 1024 px |
| 形式 | PNG（透過なし、角丸なし） |
| 注意 | iOS はシステムが角丸マスクを適用するため、正方形のまま提出 |

---

## 4. プライバシー・コンプライアンス

### 4.1 App Privacy Details（プライバシーラベル）

App Store Connect で以下を申告:

1. **収集するデータの種類**: 連絡先情報、位置情報、識別子、使用状況データ等
2. **データの使用目的**: アプリ機能、分析、サードパーティ広告等
3. **データのリンク**: ユーザーの身元に紐づくかどうか
4. **トラッキング**: App Tracking Transparency (ATT) が必要かどうか

- サードパーティ SDK のデータ収集も含めて申告が必要
- 申告内容とアプリの実際の動作が一致していないとリジェクトされる

### 4.2 プライバシーポリシー

- **すべてのアプリで必須**
- 有効な URL を App Store Connect に登録
- アプリが収集するデータとその使用方法を明記
- 対象地域の法律に準拠（GDPR、CCPA 等）

### 4.3 Privacy Manifest（PrivacyInfo.xcprivacy）

Xcode プロジェクトに `PrivacyInfo.xcprivacy` ファイルを追加:

```xml
<!-- PrivacyInfo.xcprivacy の構成例 -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSPrivacyTracking</key>
    <false/>
    <key>NSPrivacyTrackingDomains</key>
    <array/>
    <key>NSPrivacyCollectedDataTypes</key>
    <array>
        <!-- 収集するデータ型ごとにエントリを追加 -->
    </array>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <!-- Required Reason API ごとにエントリを追加 -->
    </array>
</dict>
</plist>
```

### 4.4 Required Reason API

以下の API を使用する場合、正当な理由の申告が必要:

| API カテゴリ | 例 |
|-------------|-----|
| File timestamp APIs | `NSFileCreationDate`, `NSFileModificationDate` |
| System boot time APIs | `systemUptime`, `ProcessInfo` |
| Disk space APIs | `volumeAvailableCapacity` |
| User defaults APIs | `UserDefaults`（サードパーティ SDK 経由含む） |
| Active keyboard APIs | `UITextInputMode` |

- サードパーティ SDK が使用する Required Reason API も把握すること
- Xcode の Privacy Report で使用状況を確認可能

### 4.5 Export Compliance（暗号化の使用申告）

`Info.plist` に以下を設定:

```xml
<key>ITSAppUsesNonExemptEncryption</key>
<false/>
```

| 状況 | 設定値 |
|------|-------|
| HTTPS のみ使用（ATS 標準） | `false` |
| 独自の暗号化アルゴリズムを実装 | `true`（暗号化文書の提出が必要） |
| 標準的な暗号化ライブラリのみ使用 | 多くの場合 `false`（免除対象） |

- `true` の場合、フランスの暗号化規制対応書類が必要な場合あり
- 不明な場合は法務・コンプライアンスチームに確認

### 4.6 年齢制限・COPPA 対応

- 子供向けアプリ（Kids カテゴリ）は追加の審査基準あり
- サードパーティの分析・広告 SDK の使用に制限
- COPPA（Children's Online Privacy Protection Act）準拠が必要
- App Store Connect のコンテンツレーティングを正確に設定

---

## 5. App Review Guidelines 対応

### 5.1 よくあるリジェクト理由と回避策

| Guideline | 理由 | 回避策 |
|-----------|------|--------|
| **2.1** パフォーマンス | クラッシュ・バグが多い | 十分なテストを実施。TestFlight で事前検証 |
| **2.1** 不完全なアプリ | プレースホルダーコンテンツ・未実装機能がある | すべての機能が動作する状態で提出 |
| **2.3.7** メタデータの正確性 | スクリーンショットが実際のアプリと異なる | 最新のビルドからスクリーンショットを取得 |
| **2.3.10** アプリ内広告 | テスト広告が表示される | 本番用広告 ID に切り替え |
| **3.1.1** IAP の要件 | デジタルコンテンツの購入に外部決済を使用 | デジタルコンテンツは IAP を使用。物理的商品は外部決済可 |
| **3.1.2** サブスクリプション | Restore Purchases がない | Restore 機能を必ず実装 |
| **4.0** デザイン | HIG に準拠していない | `apple-ui-guidelines` スキルを参照 |
| **4.2** 最低限の機能 | Web サイトのラッパーにすぎない | ネイティブ機能を活用し付加価値を提供 |
| **5.1.1** データ収集 | 不要なデータを収集している | 必要最小限のデータのみ収集。理由を明示 |
| **5.1.2** データの使用と共有 | プライバシーラベルと実装が不一致 | Privacy Manifest と App Privacy Details を正確に記入 |
| **5.2.1** 法的要件 | プライバシーポリシーが不備 | 有効なプライバシーポリシー URL を設定 |

### 5.2 審査メモ・デモアカウント

- **審査メモ（App Review Notes）**: 審査員への補足説明を記載
  - ログインが必要なアプリではデモアカウントの認証情報を提供
  - 特殊なハードウェアが必要な機能の説明
  - サーバーサイドの設定が必要な場合の手順

```
✅ Good: 審査メモの例
---
デモアカウント:
  メールアドレス: demo@example.com
  パスワード: Review2026!

Bluetooth 機能について:
  本アプリの Bluetooth 機能はデモモードで動作確認可能です。
  設定 > デモモードを ON にしてください。
```

### 5.3 異議申し立て（Appeal）

1. App Store Connect の「Resolution Center」で審査チームとコミュニケーション
2. リジェクト理由に対する具体的な反論・修正内容を記載
3. 解決しない場合は App Review Board に正式な異議申し立てが可能
4. 丁寧かつ事実ベースで対応する

---

## 6. バージョン管理・リリース戦略

### 6.1 バージョニング規約

| キー | 説明 | 例 |
|------|------|-----|
| `CFBundleShortVersionString` | ユーザーに表示されるバージョン番号 | `2.1.0` |
| `CFBundleVersion` | ビルド番号（同一バージョン内で一意） | `2024041801` |

- バージョン番号はセマンティックバージョニング（`MAJOR.MINOR.PATCH`）を推奨
- ビルド番号は単調増加であること（App Store Connect が拒否する）
- ビルド番号の形式例: `YYYYMMDDNN` または連番

### 6.2 リリースオプション

| オプション | 説明 | 推奨ケース |
|-----------|------|-----------|
| 自動リリース | 審査通過後すぐに公開 | 通常のアップデート |
| 手動リリース | 審査通過後、手動で公開 | マーケティングと連動するリリース |
| 段階的リリース（Phased Release） | 7 日間で段階的に配信 | リスクを抑えたいメジャーアップデート |
| 特定日リリース | 指定した日時に公開 | イベント・キャンペーン連動 |

### 6.3 段階的リリース（Phased Release）

| 日 | 配信割合 |
|----|---------|
| 1 日目 | 1% |
| 2 日目 | 2% |
| 3 日目 | 5% |
| 4 日目 | 10% |
| 5 日目 | 20% |
| 6 日目 | 50% |
| 7 日目 | 100% |

- 段階的リリース中でも、ユーザーが手動でアップデートすれば取得可能
- 問題が見つかった場合、段階的リリースを一時停止可能
- 深刻な問題の場合はリリースを取り消してホットフィックスを提出

### 6.4 複数プラットフォーム同時リリース

- **Universal Purchase**: iOS と macOS で同じ購入権を共有
  - 同一の Bundle ID グループに属させる
  - App Store Connect でリンクを設定
- iOS と macOS のバージョン番号を揃えることを推奨
- 片方のプラットフォームだけ先にリリースする場合は、リリースノートで告知

---

## 7. TestFlight ベータテスト

### 7.1 内部テスト vs 外部テスト

| 項目 | 内部テスト | 外部テスト |
|------|-----------|-----------|
| テスター数 | 最大 100 人 | 最大 10,000 人 |
| 対象者 | チームメンバー（App Store Connect ユーザー） | メールアドレスまたは公開リンクで招待 |
| Beta App Review | 不要 | 初回および大幅な変更時に必要 |
| ビルド反映 | 即座に利用可能 | Beta App Review 後に利用可能 |

### 7.2 テストグループ管理

- 目的別にグループを作成（例: 社内 QA、外部ベータ、VIP ユーザー）
- グループごとに異なるビルドを配信可能
- テスターへのリリースノート（What to Test）を記載

### 7.3 Beta App Review の注意事項

- 初回の外部テストビルドは審査が必要（通常 24〜48 時間）
- 大幅な変更がない後続ビルドは自動承認されることが多い
- 本番 App Review と同様のガイドラインが適用される
- テスト用のデモアカウント情報を審査メモに記載

### 7.4 テスト期間

- TestFlight ビルドの有効期限は **90 日間**
- 期限切れ前に新しいビルドをアップロード
- テスターにフィードバック送信を促す（TestFlight アプリ内のスクリーンショット付きフィードバック機能）

---

## 8. macOS 固有の考慮事項【macOS】

### 8.1 Mac Catalyst vs ネイティブ macOS

| 項目 | Mac Catalyst | ネイティブ macOS |
|------|-------------|----------------|
| 提出方法 | iOS アプリの「Mac (Designed for iPad)」または Catalyst 対応 | 独立した macOS アプリとして提出 |
| Bundle ID | iOS アプリと同一の場合あり（`maccatalyst.*`） | 独自の Bundle ID |
| 審査 | iOS とは別に macOS 向けの審査が行われる | macOS 独自の審査基準 |

### 8.2 Sandbox 要件

- **Mac App Store で配布するアプリは App Sandbox が必須**
- Entitlements ファイルで必要な権限を宣言:

| Entitlement | 用途 |
|------------|------|
| `com.apple.security.network.client` | ネットワーク接続（アウトバウンド） |
| `com.apple.security.network.server` | ネットワーク接続（インバウンド） |
| `com.apple.security.files.user-selected.read-write` | ユーザーが選択したファイルへのアクセス |
| `com.apple.security.files.bookmarks.app-scope` | セキュリティスコープブックマーク |

- Sandbox 制限により一部の機能が制約される場合がある
- Temporary Exception Entitlements は審査で厳しくチェックされる

### 8.3 Hardened Runtime・Notarization

| 項目 | 要件 |
|------|------|
| Hardened Runtime | Mac App Store 提出に必須。コード署名時に有効化 |
| Notarization | Mac App Store 外で配布する場合に必須。App Store 提出の場合は Apple が自動処理 |

- Hardened Runtime の例外は最小限に抑える:
  - `com.apple.security.cs.disable-library-validation` — 外部ライブラリ読み込み時のみ
  - `com.apple.security.cs.allow-jit` — JIT コンパイルが必要な場合のみ

### 8.4 ヘルパーツール・拡張機能の署名

- ヘルパーツール（LaunchAgent / LaunchDaemon）も署名が必要
- System Extension は別途の審査要件あり（Endpoint Security 等）
- Finder Sync Extension、Share Extension 等はメインアプリに含めて提出

---

## 9. 提出後の運用

### 9.1 App Analytics

| 指標 | 活用方法 |
|------|---------|
| インプレッション数 | App Store ページの表示回数。メタデータ最適化の効果測定 |
| コンバージョン率 | 閲覧 → ダウンロード率。スクリーンショット・説明文の改善指標 |
| リテンション率 | 継続利用率。アプリの品質指標 |
| クラッシュ率 | 安定性の指標。優先度の高い修正対象の特定 |

### 9.2 カスタマーレビュー対応

- App Store Connect でレビューに返信可能
- ネガティブなレビューには丁寧に対応し、改善予定を伝える
- `SKStoreReviewController.requestReview()` で適切なタイミングでレビューを依頼
  - ユーザーがポジティブな体験をした直後が効果的
  - 過度な表示はユーザー体験を損なう（システムが表示頻度を制御）

### 9.3 クラッシュレポート監視

| ツール | 特徴 |
|--------|------|
| Xcode Organizer | Xcode 内蔵。クラッシュログ・エネルギーレポート・ディスク書き込みレポート |
| App Store Connect | Web ベース。Metrics タブでクラッシュ率の推移を確認 |
| MetricKit | アプリ内でパフォーマンスデータを収集する API |

- クラッシュ率が高い場合は速やかにホットフィックスを提出
- Expedited Review（迅速審査）をリクエスト可能（重大なバグ修正時）

### 9.4 アプリの削除・非公開

| 操作 | 説明 |
|------|------|
| 販売停止 | アプリを App Store から非表示にする（既存ユーザーは再ダウンロード可能） |
| アプリの削除 | App Store Connect からアプリを完全に削除（180 日間は復元可能） |
| 特定地域での非公開 | 配信地域から除外 |

- 販売停止後も既存ユーザーのアプリは引き続き動作する
- アプリ名と Bundle ID は削除後も一定期間は再利用不可

---

## 10. 提出チェックリスト

### 初回提出

- [ ] Apple Developer Program に登録済み
- [ ] App Store Connect でアプリを作成済み
- [ ] Bundle ID が正しく設定されている
- [ ] Distribution 証明書と Provisioning Profile が有効
- [ ] 必要な Capabilities / Entitlements が設定されている

### メタデータ

- [ ] アプリ名・サブタイトルが設定されている
- [ ] 説明文・キーワードが適切に記載されている
- [ ] すべての必須デバイスサイズのスクリーンショットがアップロードされている
- [ ] アプリアイコン（1024 × 1024 px）がアップロードされている
- [ ] サポート URL が有効
- [ ] カテゴリ・コンテンツレーティングが設定されている

### プライバシー・コンプライアンス

- [ ] プライバシーポリシー URL が登録されている
- [ ] App Privacy Details（プライバシーラベル）が正確に記入されている
- [ ] Privacy Manifest（PrivacyInfo.xcprivacy）がプロジェクトに含まれている
- [ ] Required Reason API の使用理由が申告されている
- [ ] Export Compliance（`ITSAppUsesNonExemptEncryption`）が設定されている
- [ ] 子供向けアプリの場合、COPPA 準拠を確認

### ビルド・署名

- [ ] Release ビルドで Archive を作成
- [ ] テスト広告・デバッグフラグが無効化されている
- [ ] クラッシュ・重大なバグがないことを TestFlight で確認済み
- [ ] dSYM ファイルがアップロードされている（クラッシュレポート用）

### App Review 対応

- [ ] デモアカウント情報を審査メモに記載（ログインが必要な場合）
- [ ] 特殊な機能の使い方を審査メモに記載
- [ ] スクリーンショットが実際のアプリと一致している
- [ ] すべてのリンク・ボタンが正常に動作する

### macOS 固有 【macOS】

- [ ] App Sandbox が有効化されている
- [ ] Hardened Runtime が有効化されている
- [ ] ヘルパーツール・拡張機能が署名されている
- [ ] Temporary Exception Entitlements は必要最小限

### リリース設定

- [ ] リリースオプション（自動 / 手動 / 段階的）を選択済み
- [ ] バージョン番号・ビルド番号が正しい（単調増加）
- [ ] What's New（リリースノート）を記載済み
