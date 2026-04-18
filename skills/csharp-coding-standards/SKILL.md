---
name: csharp-coding-standards
description: 'C#のコーディング規約を参照・適用する。C# コーディング規約、命名規則、スタイルガイド、アクセス修飾子、null処理、LINQ、非同期処理、エラーハンドリング、XML Docコメントを確認・適用したいときに使用。Use when: applying C# style guide, reviewing C# code conventions, naming rules, access modifiers, null handling, LINQ, async/await, exception handling, XML documentation comments.'
argument-hint: '確認・適用したいコーディング規約の項目（省略可）'
---

# C# コーディング規約

## 概要

このスキルは C# コードのコーディング規約を定義します。
基本的に [Microsoft C# コーディング規則](https://learn.microsoft.com/ja-jp/dotnet/csharp/fundamentals/coding-style/coding-conventions) に準拠しつつ、プロジェクト固有のルールを追記しています。
コードレビュー・新規実装の際はこの規約に従ってください。

---

## 1. 命名規則

| 種別 | 規則 | 例 |
|------|------|----|
| クラス / 構造体 / インターフェース / 列挙型 | `UpperCamelCase` | `UserProfile`, `IRepository` |
| メソッド / プロパティ / イベント / 定数 | `UpperCamelCase` | `FetchUserAsync()`, `MaxRetryCount` |
| ローカル変数 / 引数 | `lowerCamelCase` | `userName`, `retryCount` |
| プライベートフィールド | 先頭 `_` + `lowerCamelCase` | `_cache`, `_logger` |
| インターフェース | 先頭 `I` + `UpperCamelCase` | `IUserRepository` |
| 型パラメータ | `T` プレフィックス | `TEntity`, `TKey` |
| 非同期メソッド | 末尾 `Async` | `FetchUserAsync()` |

```csharp
// ✅ Good
public interface IUserRepository
{
    Task<UserProfile?> GetByIdAsync(string id, CancellationToken ct = default);
}

public class UserService
{
    private readonly IUserRepository _repository;
    private const int MaxRetryCount = 3;

    public async Task<UserProfile?> FetchUserAsync(string userId) { }
}

// ❌ Bad
public class userService { }
public async Task<UserProfile?> fetchUser(string userId) { }  // Async サフィックスなし
```

---

## 2. コードフォーマット

<!-- プロジェクトに応じて値を変更してください -->

| 項目 | 設定値 |
|------|--------|
| インデント | スペース {4} 個（タブ不可） |
| 1行の最大文字数 | {120} 文字 |
| 中括弧 `{` の位置 | 次の行（Allman スタイル） |
| `using` 宣言 | ファイル先頭にまとめる・不要な using は削除 |
| フォーマッター | {例: EditorConfig / dotnet-format} |

```csharp
// ✅ Good（Allman スタイル）
public void ProcessOrder(Order order)
{
    if (order.IsValid)
    {
        Submit(order);
    }
}

// ❌ Bad（K&R スタイル）
public void ProcessOrder(Order order) {
    if (order.IsValid) {
        Submit(order);
    }
}
```

---

## 3. アクセス修飾子

- 最小限のアクセス修飾子を使用する（デフォルト非公開の原則）。
- アクセス修飾子は必ず明示する（デフォルト `private` でも省略しない）。
- `internal` はアセンブリ内の共有に使用し、`public` は意図的な外部公開にのみ使用する。

```csharp
// ✅ Good
private int _retryCount;
private void Validate(Order order) { }
public Task<Order> CreateOrderAsync(OrderRequest request) { }

// ❌ Bad
int _retryCount;          // アクセス修飾子省略
void Validate(Order o) { }
```

---

## 4. null 処理

- nullable reference types（`#nullable enable`）を有効にする。
- `null` 合体演算子（`??`）と null 条件演算子（`?.`）を活用する。
- `ArgumentNullException.ThrowIfNull` でメソッド引数を検証する。

```csharp
// ✅ Good
#nullable enable

public string GetDisplayName(User? user)
{
    return user?.DisplayName ?? "Anonymous";
}

public void ProcessUser(User user)
{
    ArgumentNullException.ThrowIfNull(user);
    // ...
}

// ❌ Bad
public string GetDisplayName(User user)
{
    if (user != null && user.DisplayName != null)
        return user.DisplayName;
    return "Anonymous";
}
```

---

## 5. LINQ

- LINQ メソッドチェーンを積極的に使用する。
- クエリ構文よりメソッド構文を優先する。
- 複数回使用するクエリ結果は `.ToList()` / `.ToArray()` で実体化する。
- `First()` より `FirstOrDefault()` を使用し、例外より null チェックで処理する。

```csharp
// ✅ Good
var activeUsers = users
    .Where(u => u.IsActive)
    .OrderBy(u => u.Name)
    .Select(u => new { u.Id, u.Name })
    .ToList();

var firstAdmin = users.FirstOrDefault(u => u.Role == Role.Admin);
if (firstAdmin is null) return;

// ❌ Bad
var activeUsers = from u in users
                  where u.IsActive
                  orderby u.Name
                  select new { u.Id, u.Name };  // 実体化されていない

var firstAdmin = users.First(u => u.Role == Role.Admin);  // 見つからない場合に例外
```

---

## 6. 非同期処理（async / await）

- I/O バウンドの処理はすべて `async / await` を使用する。
- `async void` は使用しない（イベントハンドラを除く）。
- `Task.Result` / `.Wait()` は使用しない（デッドロックの原因）。
- `CancellationToken` を受け取って伝播させる。

```csharp
// ✅ Good
public async Task<User?> FetchUserAsync(string id, CancellationToken ct = default)
{
    var response = await _httpClient.GetAsync($"/users/{id}", ct);
    response.EnsureSuccessStatusCode();
    return await response.Content.ReadFromJsonAsync<User>(cancellationToken: ct);
}

// ❌ Bad
public User? FetchUser(string id)
{
    return _httpClient.GetAsync($"/users/{id}").Result;  // ブロッキング呼び出し
}

public async void FetchUserAsync(string id) { }  // async void
```

---

## 7. エラーハンドリング

- `Exception` を直接スローせず、目的に合ったカスタム例外クラスを使用する。
- `catch (Exception e) { }` のような握りつぶしは禁止。
- `finally` / `using` でリソースを確実に解放する。

```csharp
// ✅ Good
public sealed class NotFoundException : Exception
{
    public NotFoundException(string resourceName, object key)
        : base($"'{resourceName}' with key '{key}' was not found.") { }
}

try
{
    var user = await _repository.GetByIdAsync(userId, ct)
        ?? throw new NotFoundException(nameof(User), userId);
}
catch (NotFoundException)
{
    return Results.NotFound();
}
catch (Exception ex)
{
    _logger.LogError(ex, "Unexpected error fetching user {UserId}", userId);
    throw;
}

// ❌ Bad
try { }
catch { }  // 握りつぶし

throw new Exception("User not found");  // 汎用 Exception
```

---

## 8. コメント規約

- コードのロジックが自明でない箇所にのみコメントを付ける。
- 公開 API には **XML Doc** コメント（`///`）を付ける。
- TODO / FIXME は `// TODO: 説明` の形式で記述し、チケット番号を添える。

```csharp
/// <summary>
/// 指定した ID のユーザーを取得します。
/// </summary>
/// <param name="id">ユーザー識別子。</param>
/// <param name="ct">キャンセレーショントークン。</param>
/// <returns>ユーザーオブジェクト。見つからない場合は <see langword="null"/>。</returns>
/// <exception cref="HttpRequestException">HTTP リクエストが失敗した場合。</exception>
public async Task<User?> GetByIdAsync(string id, CancellationToken ct = default)
{
    // ...
}

// TODO: #321 レート制限の実装
```

---

## 9. プロジェクト固有のルール

<!-- プロジェクトに応じて追記してください -->

- {プロジェクト固有のルールをここに記載}
