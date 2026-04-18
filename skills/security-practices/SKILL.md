---
name: security-practices
description: 'セキュリティベストプラクティスを参照・適用する。認証・認可・入力バリデーション・暗号化・APIキー管理・OWASP Top 10・プラットフォーム別セキュリティ対策を確認・適用したいときに使用。Use when: implementing authentication, securing API keys, input validation, encryption, OWASP Top 10 compliance, platform-specific security.'
argument-hint: '確認・適用したいセキュリティ項目（省略可）'
---

# セキュリティベストプラクティス

## 概要

このスキルはアプリケーションのセキュリティ実装ガイドラインを定義します。
OWASP Top 10 を基準に、プラットフォーム横断のセキュリティ対策を網羅します。

---

## 1. 認証・認可

### 原則

- **最小権限の原則** — ユーザーが必要とする最低限の権限のみ付与する
- **認証と認可を分離** — 「誰か」の確認と「何ができるか」の確認を別々に実装する
- **セッション管理** — セッション ID は十分なランダム性を持ち、ログアウト後は無効化する

### トークン管理

```ts
// ✅ Good: HttpOnly Cookie にトークンを保存（Web）
// → JavaScript からアクセス不可 → XSS 耐性
res.cookie('accessToken', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  maxAge: 15 * 60 * 1000, // 15分
});

// ❌ Bad: localStorage にトークンを保存（XSS で盗まれる）
localStorage.setItem('accessToken', token);
```

### JWT の扱い

- ペイロードに機密情報（パスワード・クレジットカード番号等）を含めない
- 有効期限（`exp`）を短く設定し、リフレッシュトークンで更新する
- 署名アルゴリズムは `RS256`（RSA）または `ES256`（ECDSA）を使う。`HS256` は共有シークレット管理が必要なため注意

---

## 2. 入力バリデーション

### 原則

- **サーバーサイドで必ず検証** — クライアントサイドのバリデーションは UX 補助にすぎない
- **ホワイトリスト方式** — 許可する形式を定義し、それ以外を拒否する
- **バリデーションライブラリを使う** — 自前パース実装はバグの温床になる

```ts
// ✅ Good: Zod でスキーマ検証（TypeScript / Web）
import { z } from 'zod';

const UserSchema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150),
  name: z.string().min(1).max(100),
});

const parsed = UserSchema.safeParse(req.body);
if (!parsed.success) {
  return res.status(400).json({ errors: parsed.error.flatten() });
}
```

### SQL インジェクション対策

```ts
// ✅ Good: プリペアドステートメント
const user = await db.query(
  'SELECT * FROM users WHERE email = $1',
  [email]
);

// ❌ Bad: 文字列結合
const user = await db.query(
  `SELECT * FROM users WHERE email = '${email}'`
);
```

### XSS 対策

```ts
// ✅ Good: React は自動エスケープ
<p>{userInput}</p>

// ❌ Bad: dangerouslySetInnerHTML（やむを得ない場合は DOMPurify でサニタイズ）
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// DOMPurify を使う場合
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

---

## 3. シークレット・API キー管理

### 原則

- **コードにシークレットをハードコードしない**
- **リポジトリにシークレットをコミットしない**（`.env` は `.gitignore` に追加）
- **最小スコープのシークレット** — 必要なアクセス権のみ付与する

### Web / サーバーサイド

```bash
# .env ファイル（リポジトリには含めない）
DATABASE_URL=postgresql://...
STRIPE_SECRET_KEY=sk_live_...
JWT_SECRET=...
```

```ts
// ✅ Good: 環境変数から読み込む
const apiKey = process.env.STRIPE_SECRET_KEY;
if (!apiKey) throw new Error('STRIPE_SECRET_KEY is not set');
```

### iOS / macOS（Swift）

```swift
// ✅ Good: Keychain に保存
import Security

func saveToKeychain(key: String, value: String) {
    let data = value.data(using: .utf8)!
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecValueData as String: data,
    ]
    SecItemAdd(query as CFDictionary, nil)
}

// ❌ Bad: UserDefaults に機密情報を保存
UserDefaults.standard.set(apiKey, forKey: "apiKey")
```

### Android（Kotlin）

```kotlin
// ✅ Good: EncryptedSharedPreferences に保存
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val prefs = EncryptedSharedPreferences.create(
    context, "secret_prefs", masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)
```

---

## 4. 暗号化

### データ保存時の暗号化

| プラットフォーム | 推奨方式 |
|--------------|---------|
| iOS / macOS | Keychain（シークレット）・`CryptoKit`（データ暗号化） |
| Android | `EncryptedSharedPreferences` / `EncryptedFile` / Keystore |
| Web（ブラウザ） | Web Crypto API（機密データはサーバー側で管理を推奨） |
| サーバー | AES-256-GCM、argon2id（パスワードハッシュ） |

### 通信の暗号化

- すべての通信は **HTTPS（TLS 1.2 以上）** を使う
- HTTP へのフォールバックを禁止する（HSTS 設定を推奨）
- 証明書の有効性を検証し、自己署名証明書を本番環境で使わない

---

## 5. OWASP Top 10 対応チェックリスト

| # | リスク | 対策 |
|---|--------|------|
| A01 | アクセス制御の不備 | 最小権限の原則・サーバーサイド認可チェック |
| A02 | 暗号化の失敗 | TLS・AES-256・argon2id・Keychain/Keystore |
| A03 | インジェクション | プリペアドステートメント・入力バリデーション |
| A04 | 安全でない設計 | 脅威モデリング・セキュリティ要件の定義 |
| A05 | セキュリティの設定ミス | デフォルト設定の見直し・不要なポートを閉じる |
| A06 | 脆弱なコンポーネント | 定期的な依存ライブラリ更新・Dependabot 設定 |
| A07 | 認証の失敗 | MFA・ブルートフォース対策・セッション管理 |
| A08 | ソフトウェア整合性の失敗 | CI/CD パイプラインの署名検証 |
| A09 | ログとモニタリングの不備 | 認証失敗ログ・異常検知アラート |
| A10 | SSRF | 外部リクエスト URL のホワイトリスト検証 |

---

## 6. プラットフォーム別注意事項

### iOS / macOS

- App Transport Security（ATS）は無効化しない
- ディープリンク URL の処理では必ずホワイトリスト検証を行う
- iCloud / バックアップ対象から機密データを除外する（`isExcludedFromBackup`）

### Android

- `android:exported="true"` は必要な Activity のみに設定する
- `WebView.setJavaScriptEnabled(true)` を使う場合は信頼できる URL のみ許可する
- `StrictMode` を開発環境で有効にしてネットワーク・ディスクアクセスを検査する

### Web

- `Content-Security-Policy`（CSP）ヘッダを設定する
- `X-Frame-Options: DENY` でクリックジャッキングを防ぐ
- Cookie には `HttpOnly`・`Secure`・`SameSite=Strict` を設定する

---

## 7. セキュリティレビュー手順

実装後は `agents/security-reviewer.agent.md` を使ってセキュリティレビューを実施すること。

### レビューポイント

- [ ] シークレット・API キーがコードにハードコードされていない
- [ ] すべての入力がサーバーサイドでバリデーションされている
- [ ] 認証・認可のチェックがサーバーサイドで行われている
- [ ] データ保存・通信が適切に暗号化されている
- [ ] 依存ライブラリに既知の脆弱性がないか確認した
- [ ] エラーメッセージに内部情報（スタックトレース等）を含めていない
- [ ] ログに機密情報（パスワード・トークン等）を出力していない
