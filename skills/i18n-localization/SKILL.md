---
name: i18n-localization
description: '国際化（i18n）とローカライゼーション（l10n）のガイドラインを参照・適用する。多言語対応、翻訳ファイル管理、日付・数値・通貨フォーマット、RTL 対応、複数形処理、プラットフォーム別実装を確認・適用したいときに使用。Use when: implementing internationalization, managing translation files, locale formatting, RTL layout support, pluralization.'
argument-hint: '確認・適用したい i18n / l10n の項目（省略可）'
---

# 国際化（i18n）/ ローカライゼーション（l10n）ガイドライン

## 概要

このスキルはアプリケーションの国際化・ローカライゼーション実装の規約を定義します。
多言語・多地域に対応したプロダクトを開発する際はこのガイドラインに従ってください。

---

## 1. 基本原則

- **コード中にテキストをハードコードしない** — すべての表示テキストはリソースファイルで管理する
- **デフォルトロケールは `en`（英語）を基準** にし、各言語への翻訳を派生させる
- **文化的中立性** — アイコン・色・ジェスチャーが文化的に問題ないか確認する
- **テキスト膨張を考慮** — 英語の 1.3〜1.5 倍の長さを想定してレイアウトを設計する

---

## 2. 対応ロケールの管理

### ロケールコード規則

- IETF BCP 47 に従う（例: `en`, `ja`, `zh-Hans`, `zh-Hant`, `ar`, `he`）
- 地域バリアント（例: `en-US`, `en-GB`）は必要な場合のみ分離する

### 言語ファイルの構造

```
messages/
  en.json         — デフォルト（必須）
  ja.json         — 日本語
  zh-Hans.json    — 中国語（簡体字）
  zh-Hant.json    — 中国語（繁体字）
  ar.json         — アラビア語（RTL）
  ko.json         — 韓国語
  fr.json         — フランス語
  de.json         — ドイツ語
  es.json         — スペイン語
  pt-BR.json      — ポルトガル語（ブラジル）
```

---

## 3. 翻訳キーの設計

### キー命名規則

- `スコープ.コンポーネント.意味` の形式を使う（ドット区切りネスト）
- **動詞で始めない** — 意味の主体を先に書く（`button.submit` ではなく `form.submitButton`）
- **内容ではなく意味で命名** — `redText` ではなく `errorMessage`

```json
// ✅ Good
{
  "common": {
    "button": {
      "submit": "Submit",
      "cancel": "Cancel",
      "save": "Save"
    },
    "error": {
      "required": "This field is required.",
      "networkError": "Network error. Please try again."
    }
  },
  "auth": {
    "login": {
      "title": "Sign In",
      "emailLabel": "Email Address",
      "passwordLabel": "Password",
      "submitButton": "Sign In",
      "forgotPassword": "Forgot your password?"
    }
  }
}

// ❌ Bad
{
  "loginTitle": "Sign In",        // スコープなし
  "redText": "Error",             // 見た目で命名
  "button1": "Submit"             // 意味不明
}
```

### 変数の埋め込み

- プレースホルダーは `{変数名}` 形式を使う（ライブラリに応じて変わる場合あり）

```json
{
  "greeting": "Hello, {name}!",
  "itemCount": "You have {count} items in your cart."
}
```

---

## 4. 複数形（Plural）処理

- 必ず複数形対応のロジックを使用し、`if count === 1` のようなハードコードをしない
- `zero` / `one` / `two` / `few` / `many` / `other` の各カテゴリを言語ごとに定義する

### Web（next-intl の例）

```json
{
  "items": {
    "one": "{count} item",
    "other": "{count} items"
  }
}
```

### iOS / macOS（Localizable.stringsdict）

```xml
<key>%d items</key>
<dict>
  <key>NSStringLocalizedFormatKey</key>
  <string>%#@items@</string>
  <key>items</key>
  <dict>
    <key>NSStringFormatSpecTypeKey</key>
    <string>NSStringPluralRuleType</string>
    <key>one</key>
    <string>%d item</string>
    <key>other</key>
    <string>%d items</string>
  </dict>
</dict>
```

### Android（strings.xml）

```xml
<plurals name="items_count">
  <item quantity="one">%d item</item>
  <item quantity="other">%d items</item>
</plurals>
```

---

## 5. 日付・時刻・数値・通貨のフォーマット

- **ハードコードしない** — `Intl` API / ロケール対応ライブラリを使う
- 日付は ISO 8601 でサーバーとやりとりし、表示時にローカル形式に変換する

### Web（Intl API）

```ts
// 日付
new Intl.DateTimeFormat('ja-JP', { dateStyle: 'long' }).format(date);
// → 2026年4月18日

// 数値
new Intl.NumberFormat('de-DE').format(1234567.89);
// → 1.234.567,89

// 通貨
new Intl.NumberFormat('ja-JP', { style: 'currency', currency: 'JPY' }).format(1500);
// → ¥1,500
```

### iOS / macOS（Swift）

```swift
let formatter = DateFormatter()
formatter.locale = Locale.current
formatter.dateStyle = .long
formatter.string(from: Date())

// 通貨
let nf = NumberFormatter()
nf.numberStyle = .currency
nf.locale = Locale.current
nf.string(from: 1500)
```

---

## 6. RTL（Right-to-Left）対応

RTL 言語（アラビア語 `ar`、ヘブライ語 `he`、ペルシャ語 `fa` 等）に対応するための原則。

### Web

```css
/* logical properties を使う */
margin-inline-start: 1rem;    /* ✅ left/right に依存しない */
padding-inline-end: 0.5rem;

/* ❌ 避ける */
margin-left: 1rem;
padding-right: 0.5rem;
```

```html
<!-- html タグに dir 属性を設定 -->
<html lang="ar" dir="rtl">
```

### iOS / macOS（Swift）

```swift
// UIKit: semanticContentAttribute を使う
view.semanticContentAttribute = .forceRightToLeft

// SwiftUI: environment で自動対応される
// HStack は RTL で自動反転する
```

### Android（Kotlin）

```xml
<!-- start/end を使う（left/right ではなく） -->
<TextView
    android:paddingStart="16dp"
    android:paddingEnd="16dp"
    android:layout_marginStart="8dp" />
```

---

## 7. プラットフォーム別実装

### iOS / macOS

- テキストは `String(localized:)` または `LocalizedStringKey` を使う
- ファイル: `Localizable.strings`（一般テキスト）+ `Localizable.stringsdict`（複数形）
- Xcode の **String Catalog（.xcstrings）** を最優先で使用する

```swift
// ✅ String Catalog（推奨）
Text("auth.login.title")  // SwiftUI

// ✅ コードから
String(localized: "greeting", defaultValue: "Hello, \(name)!")
```

### Android

- テキストは `strings.xml`、複数形は `plurals` を使う
- `getString(R.string.key, arg1, arg2)` でプレースホルダーを埋める
- `Context` なしで文字列を取得しない

```kotlin
// ✅ Good
getString(R.string.greeting, userName)
resources.getQuantityString(R.plurals.items_count, count, count)
```

### Web（next-intl の例）

```tsx
import { useTranslations } from 'next-intl';

function LoginPage() {
  const t = useTranslations('auth.login');
  return <h1>{t('title')}</h1>;
}
```

### Windows（C#）

- テキストは `Resources.resw` で管理する
- `ResourceLoader` を使って実行時に取得する

```csharp
var resourceLoader = ResourceLoader.GetForCurrentView();
string text = resourceLoader.GetString("Auth/Login/Title");
```

---

## 8. 翻訳ワークフロー

### 翻訳ファイルの管理

1. **開発者** がデフォルト言語（`en`）のキーを追加する
2. **CI** で未翻訳キーを自動検出してアラートを出す
3. **翻訳担当 / 翻訳ツール**（Crowdin / Phrase / Lokalise 等）が各言語に翻訳する
4. **PR** で翻訳ファイルをレビューしてマージする

### 未翻訳キーのハンドリング

- デフォルト言語（`en`）にフォールバックする
- フォールバックが発生した場合はログに記録する（本番環境では過検知に注意）

---

## 9. チェックリスト

- [ ] すべての表示テキストがリソースファイルで管理されている
- [ ] コード中にテキストがハードコードされていない
- [ ] 複数形処理が適切に実装されている
- [ ] 日付・時刻・数値・通貨がロケール対応フォーマットを使用している
- [ ] RTL 言語で表示崩れが起きないことを確認した
- [ ] テキスト膨張でレイアウトが崩れないことを確認した
- [ ] プレースホルダーが変数名で命名されている
- [ ] CI で未翻訳キー検出が動作している
