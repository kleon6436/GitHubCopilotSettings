---
name: typescript-coding-standards
description: 'TypeScriptのコーディング規約を参照・適用する。TypeScript コーディング規約、命名規則、スタイルガイド、型定義、interface vs type alias、ジェネリクス、型ガード、null/undefined処理、非同期処理を確認・適用したいときに使用。Use when: applying TypeScript style guide, reviewing TypeScript code conventions, type definitions, generics, type guards, null handling, async/await, TSDoc.'
argument-hint: '確認・適用したいコーディング規約の項目（省略可）'
---

# TypeScript コーディング規約

## 概要

このスキルは TypeScript コードのコーディング規約を定義します。
コードレビュー・新規実装の際はこの規約に従ってください。

---

## 1. 命名規則

| 種別 | 規則 | 例 |
|------|------|----|
| 変数 / 関数 / メソッド | `lowerCamelCase` | `fetchUserData()` |
| クラス / インターフェース / 型 / 列挙型 | `UpperCamelCase` | `UserProfile`, `ApiError` |
| 定数（モジュールスコープ） | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| プライベートメンバ | 先頭 `#`（クラスフィールド）または `_`（慣例） | `#cache`, `_session` |
| ファイル名 | `kebab-case` | `user-service.ts` |
| 型パラメータ | 単一大文字 or 意味のある名前 | `T`, `TKey`, `TValue` |

```ts
// ✅ Good
const MAX_TIMEOUT = 30;

class NetworkManager {
  #cache = new Map<string, unknown>();

  fetchData(endpoint: string): Promise<unknown> { return Promise.resolve(); }
}

// ❌ Bad
class networkmanager { }
const maxTimeout = 30;
```

---

## 2. コードフォーマット

<!-- プロジェクトに応じて値を変更してください -->

| 項目 | 設定値 |
|------|--------|
| インデント | スペース {2} 個（タブ不可） |
| 1行の最大文字数 | {120} 文字 |
| 文字列クォート | {シングルクォート `'` を優先} |
| セミコロン | {付ける} |
| 末尾カンマ | 複数行の場合は末尾カンマを付ける |
| フォーマッター | {例: Prettier} |
| リンター | {例: ESLint + typescript-eslint} |

---

## 3. 型定義（interface vs type alias）

- **`interface`** はオブジェクト型・クラスの契約定義に使用する。宣言マージが必要な場合も `interface`。
- **`type`** はユニオン型・交差型・プリミティブ型エイリアス・タプルに使用する。
- `any` は使用しない。やむを得ない場合は `unknown` を使用し、型ガードで絞り込む。

```ts
// ✅ Good
interface User {
  id: string;
  name: string;
  email: string;
}

type UserId = string;
type Status = 'active' | 'inactive' | 'pending';
type ApiResponse<T> = { data: T; error: null } | { data: null; error: string };

// ❌ Bad
type User = {  // オブジェクト型に type を使う（拡張性が下がる）
  id: string;
};
const response: any = await fetch('/api');  // any を使う
```

---

## 4. ジェネリクス

- 意味のある型パラメータ名を付ける（単一文字は汎用コンテナのみ）。
- 不必要なジェネリクスは避ける。
- 型パラメータに制約（`extends`）を積極的に使用する。

```ts
// ✅ Good
function getProperty<TObject, TKey extends keyof TObject>(
  obj: TObject,
  key: TKey,
): TObject[TKey] {
  return obj[key];
}

// ❌ Bad
function getProperty<T, K>(obj: T, key: K): any {
  return (obj as any)[key];
}
```

---

## 5. 型アサーション・型ガード

- 型アサーション（`as`）はやむを得ない場合のみ使用する。
- ユーザー定義型ガード（`is` キーワード）を積極的に使用する。
- `as unknown as T` のような二段階アサーションは禁止。

```ts
// ✅ Good
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value
  );
}

if (isUser(response)) {
  console.log(response.name);
}

// ❌ Bad
const user = response as User;  // 検証なしのアサーション
const user2 = response as unknown as User;  // 二段階アサーション
```

---

## 6. null / undefined の扱い

- `strict` モードを有効にし、`strictNullChecks` を必ず有効にする。
- `null` と `undefined` は明示的に使い分ける（存在しない値は `undefined`、意図的な空値は `null`）。
- オプショナルチェーン（`?.`）と null 合体演算子（`??`）を活用する。

```ts
// ✅ Good
const name = user?.profile?.displayName ?? 'Anonymous';
const length = items?.length ?? 0;

// ❌ Bad
const name = user && user.profile && user.profile.displayName
  ? user.profile.displayName
  : 'Anonymous';
```

---

## 7. エラーハンドリング

- `Error` を継承したカスタムエラークラスを使用する。
- `catch` の引数は `unknown` 型として受け取り、型ガードで絞り込む。
- エラーを握りつぶさない。

```ts
// ✅ Good
class ApiError extends Error {
  constructor(
    message: string,
    public readonly statusCode: number,
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

try {
  const data = await fetchData();
} catch (error: unknown) {
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
  console.log(e);  // 型不明のまま使用
}
```

---

## 8. 非同期処理

- `async / await` を優先する。`.then()` チェーンは避ける。
- `Promise.all` / `Promise.allSettled` で並列処理を行う。
- 戻り値の型は `Promise<T>` を明示する。

```ts
// ✅ Good
async function loadDashboard(userId: string): Promise<Dashboard> {
  const [user, posts] = await Promise.all([
    fetchUser(userId),
    fetchPosts(userId),
  ]);
  return { user, posts };
}

// ❌ Bad
function loadDashboard(userId: string) {
  return fetchUser(userId).then((user) =>
    fetchPosts(userId).then((posts) => ({ user, posts })),
  );
}
```

---

## 9. コメント規約

- コードのロジックが自明でない箇所にのみコメントを付ける。
- 公開 API には **TSDoc** 形式のドキュメントコメントを付ける。
- TODO / FIXME は `// TODO: 説明` の形式で記述し、チケット番号を添える。

```ts
/**
 * 指定した ID のユーザーを取得します。
 * @param id - ユーザー識別子
 * @returns ユーザーオブジェクト
 * @throws {@link ApiError} ユーザーが存在しない場合
 */
async function fetchUser(id: string): Promise<User> {
  // ...
}

// TODO: #456 キャッシュ戦略を実装する
```

---

## 10. プロジェクト固有のルール

<!-- プロジェクトに応じて追記してください -->

- {プロジェクト固有のルールをここに記載}
