---
name: rust-coding-standards
description: 'Rustのコーディング規約を参照・適用する。Rust コーディング規約、命名規則、スタイルガイド、所有権・借用・ライフタイム、エラーハンドリング（Result/Option）、トレイト、非同期処理（async/await）、rustdocコメントを確認・適用したいときに使用。Use when: applying Rust style guide, reviewing Rust code conventions, ownership, borrowing, lifetimes, error handling with Result/Option, traits, async/await, rustdoc.'
argument-hint: '確認・適用したいコーディング規約の項目（省略可）'
---

# Rust コーディング規約

## 概要

このスキルは Rust コードのコーディング規約を定義します。
[Rust API ガイドライン](https://rust-lang.github.io/api-guidelines/) および標準的な Rust イディオムに準拠しています。
コードレビュー・新規実装の際はこの規約に従ってください。

---

## 1. 命名規則

| 種別 | 規則 | 例 |
|------|------|----|
| 関数 / メソッド / 変数 / モジュール | `snake_case` | `fetch_user()`, `user_name` |
| 型（構造体 / 列挙型 / トレイト） | `UpperCamelCase` | `UserProfile`, `ApiError` |
| 定数 / `static` | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| 型パラメータ | 単一大文字 or 短い `UpperCamelCase` | `T`, `Key`, `Val` |
| ライフタイム | 単一小文字 or 短い名前 | `'a`, `'buf` |
| クレート / パッケージ | `snake_case`（ハイフンも可） | `my_crate` |

```rust
// ✅ Good
const MAX_RETRIES: u32 = 3;

struct UserProfile {
    id: String,
    name: String,
}

fn fetch_user(user_id: &str) -> Result<UserProfile, ApiError> { todo!() }

// ❌ Bad
struct userProfile { }
fn FetchUser(userId: &str) { }
const maxRetries: u32 = 3;
```

---

## 2. コードフォーマット

<!-- プロジェクトに応じて値を変更してください -->

| 項目 | 設定値 |
|------|--------|
| インデント | スペース 4 個（`rustfmt` デフォルト） |
| 1行の最大文字数 | {100} 文字（`rustfmt` の `max_width`） |
| フォーマッター | `rustfmt`（`cargo fmt` で適用） |
| リンター | `clippy`（`cargo clippy` で適用） |

```toml
# rustfmt.toml
max_width = 100
edition = "2021"
```

---

## 3. 所有権・借用・ライフタイム

- 所有権の移動（`move`）が必要な場合を除き、参照（`&` / `&mut`）を使用する。
- 不変参照（`&T`）を優先し、可変参照（`&mut T`）は必要な場合のみ使用する。
- ライフタイムアノテーションはコンパイラが推論できない場合にのみ明示する。
- `clone()` は性能上の問題がない範囲で使用し、過度な最適化は避ける。

```rust
// ✅ Good
fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}

fn process_users(users: &[User]) {
    for user in users {
        println!("{}", user.name);
    }
}

// ❌ Bad
fn greet(name: String) -> String {  // 不必要な所有権の移動
    format!("Hello, {}!", name)
}
```

---

## 4. エラーハンドリング（Result / Option）

- エラーが発生し得る処理は `Result<T, E>` を返す。
- 値が存在しない場合は `Option<T>` を返す（`None` と `Err` を使い分ける）。
- `unwrap()` / `expect()` は本番コードでは原則禁止。テストコードのみ許可。
- `?` 演算子を積極的に使用してエラーを伝播させる。
- `thiserror` クレートでカスタムエラー型を定義する。`anyhow` はアプリケーション層で使用する。

```rust
// ✅ Good
use thiserror::Error;

#[derive(Debug, Error)]
pub enum ApiError {
    #[error("HTTP request failed: {0}")]
    Http(#[from] reqwest::Error),

    #[error("User not found: {id}")]
    NotFound { id: String },

    #[error("Failed to deserialize response")]
    Deserialization(#[from] serde_json::Error),
}

async fn fetch_user(id: &str) -> Result<User, ApiError> {
    let response = client.get(&format!("/users/{}", id)).send().await?;
    let user = response.json::<User>().await?;
    Ok(user)
}

// ❌ Bad
async fn fetch_user(id: &str) -> User {
    client.get(&format!("/users/{}", id))
        .send()
        .await
        .unwrap()
        .json::<User>()
        .await
        .unwrap()  // unwrap の連鎖
}
```

---

## 5. トレイト

- 共通の振る舞いはトレイトで抽象化する。
- `Display` / `Debug` / `Clone` / `PartialEq` 等の標準トレイトは `#[derive]` で自動実装する。
- トレイトオブジェクト（`dyn Trait`）よりジェネリクス（`impl Trait`）を優先する。
- トレイトの `default` 実装を活用する。

```rust
// ✅ Good
#[derive(Debug, Clone, PartialEq)]
pub struct User {
    pub id: String,
    pub name: String,
}

// impl Trait を優先
fn process(item: impl Displayable) { }

// ❌ Bad
fn process(item: &dyn Displayable) { }  // 動的ディスパッチが不要な場合
```

---

## 6. 非同期処理（async / await）

- 非同期ランタイムは {例: Tokio} を使用する（プロジェクトに応じて変更）。
- `async fn` を使用し、`.await` で非同期処理を待機する。
- `tokio::spawn` でタスクを並列実行し、`JoinHandle` でライフサイクルを管理する。
- `tokio::try_join!` / `tokio::join!` で並列実行を行う。

```rust
// ✅ Good
async fn load_dashboard(user_id: &str) -> Result<Dashboard, ApiError> {
    let (user, posts) = tokio::try_join!(
        fetch_user(user_id),
        fetch_posts(user_id),
    )?;
    Ok(Dashboard { user, posts })
}

// ❌ Bad
async fn load_dashboard(user_id: &str) -> Result<Dashboard, ApiError> {
    let user  = fetch_user(user_id).await?;   // 直列実行（並列にすべき）
    let posts = fetch_posts(user_id).await?;
    Ok(Dashboard { user, posts })
}
```

---

## 7. コメント規約

- コードのロジックが自明でない箇所にのみコメントを付ける。
- 公開アイテムには **rustdoc** 形式のドキュメントコメント（`///`）を付ける。
- モジュールレベルのドキュメントには `//!` を使用する。
- TODO / FIXME は `// TODO: 説明` の形式で記述し、チケット番号を添える。

```rust
/// 指定した ID のユーザーを取得します。
///
/// # Arguments
/// * `id` - ユーザー識別子
///
/// # Errors
/// * [`ApiError::Http`] - HTTP リクエストが失敗した場合
/// * [`ApiError::NotFound`] - ユーザーが存在しない場合
///
/// # Examples
/// ```
/// let user = fetch_user("user_123").await?;
/// println!("{}", user.name);
/// ```
pub async fn fetch_user(id: &str) -> Result<User, ApiError> {
    // ...
}

// TODO: #202 レート制限の実装
```

---

## 8. プロジェクト固有のルール

<!-- プロジェクトに応じて追記してください -->

- {プロジェクト固有のルールをここに記載}
