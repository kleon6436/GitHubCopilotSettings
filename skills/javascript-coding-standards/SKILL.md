---
name: javascript-coding-standards
description: 'JavaScriptのコーディング規約を参照・適用する。JavaScript コーディング規約、命名規則、スタイルガイド、変数宣言、関数、モジュール、非同期処理、エラーハンドリング、コメント規約を確認・適用したいときに使用。Use when: applying JavaScript style guide, reviewing JavaScript code conventions, naming rules, variable declarations, async/await, error handling patterns, JSDoc.'
argument-hint: '確認・適用したいコーディング規約の項目（省略可）'
---

# JavaScript コーディング規約

## 概要

このスキルは JavaScript コードのコーディング規約を定義します。
コードレビュー・新規実装の際はこの規約に従ってください。

---

## 1. 命名規則

| 種別 | 規則 | 例 |
|------|------|----|
| 変数 / 関数 / メソッド | `lowerCamelCase` | `fetchUserData()` |
| クラス / コンストラクタ | `UpperCamelCase` | `UserProfile` |
| 定数（モジュールスコープ） | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| プライベートメンバ（慣例） | 先頭 `_` | `_cache` |
| ファイル名 | `kebab-case` | `user-service.js` |

```js
// ✅ Good
const MAX_TIMEOUT = 30;

class NetworkManager {
  constructor() {
    this._session = null;
  }

  fetchData(endpoint) { }
}

// ❌ Bad
const maxTimeout = 30;
class networkmanager { }
function FetchData() { }
```

---

## 2. コードフォーマット

<!-- プロジェクトに応じて値を変更してください -->

| 項目 | 設定値 |
|------|--------|
| インデント | スペース {2} 個（タブ不可） |
| 1行の最大文字数 | {120} 文字 |
| 文字列クォート | {シングルクォート `'` を優先} |
| セミコロン | {付ける / 付けない} |
| 末尾カンマ | 複数行の場合は末尾カンマを付ける |
| フォーマッター | {例: Prettier} |
| リンター | {例: ESLint} |

```js
// ✅ Good（複数行の末尾カンマ）
const SUPPORTED_FORMATS = [
  'json',
  'csv',
  'xml',
];

// ❌ Bad
const SUPPORTED_FORMATS = ['json', 'csv',
  'xml']
```

---

## 3. 変数宣言

- `const` を優先する。再代入が必要な場合のみ `let` を使用する。
- `var` は使用しない。

```js
// ✅ Good
const userId = 'abc123';
let retryCount = 0;

// ❌ Bad
var userId = 'abc123';
let userId2 = 'abc123'; // 再代入しないのに let
```

---

## 4. 関数

- **アロー関数**をコールバック・短い関数に使用する。
- **`function` 宣言**はモジュールトップレベルの名前付き関数に使用する。
- デフォルト引数を活用する。

```js
// ✅ Good
function fetchUser(id) {
  return apiClient.get(`/users/${id}`);
}

const doubled = numbers.map((n) => n * 2);

function createUser(name, role = 'viewer') { }

// ❌ Bad
const fetchUser = function(id) { /* ... */ };  // トップレベルで無名関数
const doubled = numbers.map(function(n) { return n * 2; });
```

---

## 5. モジュール

- ES Modules（`import` / `export`）を使用する。`require` は使わない。
- `default export` は1ファイルに1つのクラス・関数の場合のみ使用する。
- 複数エクスポートは **named export** を優先する。

```js
// ✅ Good
export function fetchUser(id) { }
export const MAX_RETRY = 3;

import { fetchUser, MAX_RETRY } from './user-service.js';

// ❌ Bad
module.exports = { fetchUser };
const { fetchUser } = require('./user-service');
```

---

## 6. エラーハンドリング

- `try / catch` を使用し、エラーを握りつぶさない。
- エラーは `Error` を継承したカスタムクラスで分類する。
- `catch` ブロックでは必ず適切なログ出力または再スローを行う。

```js
// ✅ Good
class ApiError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = 'ApiError';
    this.statusCode = statusCode;
  }
}

try {
  const data = await fetchData();
} catch (error) {
  if (error instanceof ApiError) {
    console.error(`API error ${error.statusCode}: ${error.message}`);
  } else {
    throw error;
  }
}

// ❌ Bad
try {
  const data = await fetchData();
} catch (e) {
  // 何もしない
}
```

---

## 7. 非同期処理

- `async / await` を優先する。`.then()` チェーンは避ける。
- `Promise.all` / `Promise.allSettled` を使って並列処理を行う。
- `await` の使い忘れに注意する（戻り値の型を確認する）。

```js
// ✅ Good
async function loadDashboard(userId) {
  const [user, posts] = await Promise.all([
    fetchUser(userId),
    fetchPosts(userId),
  ]);
  return { user, posts };
}

// ❌ Bad
function loadDashboard(userId) {
  return fetchUser(userId).then((user) => {
    return fetchPosts(userId).then((posts) => ({ user, posts }));
  });
}
```

---

## 8. コメント規約

- コードのロジックが自明でない箇所にのみコメントを付ける。
- 公開 API には **JSDoc** 形式のドキュメントコメントを付ける。
- TODO / FIXME は `// TODO: 説明` の形式で記述し、チケット番号を添える。

```js
/**
 * 指定した ID のユーザーを取得します。
 * @param {string} id - ユーザー識別子
 * @returns {Promise<User>} ユーザーオブジェクト
 * @throws {ApiError} ユーザーが存在しない場合
 */
async function fetchUser(id) {
  // ...
}

// TODO: #456 キャッシュ戦略を実装する
```

---

## 9. プロジェクト固有のルール

<!-- プロジェクトに応じて追記してください -->

- {プロジェクト固有のルールをここに記載}
