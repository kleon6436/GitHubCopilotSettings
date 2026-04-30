---
name: csharp-coding-standards
description: 'Reference and apply C# coding standards. Use when: applying C# style guide, reviewing C# code conventions, naming rules, access modifiers, null handling, LINQ, async/await, exception handling, XML documentation comments.'
argument-hint: 'Coding standard item to check or apply (optional)'
---

# C# Coding Standards

## Overview

This skill defines the coding standards for C# code.
It follows the [Microsoft C# Coding Conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions) as a baseline, with project-specific rules added.
Follow these standards during code reviews and new implementations.

---

## 1. Naming Conventions

| Category | Rule | Example |
|----------|------|---------|

| Class / Struct / Interface / Enum | `UpperCamelCase` | `UserProfile`, `IRepository` |
| Method / Property / Event / Constant | `UpperCamelCase` | `FetchUserAsync()`, `MaxRetryCount` |
| Local variable / Parameter | `lowerCamelCase` | `userName`, `retryCount` |
| Private field | Leading `_` + `lowerCamelCase` | `_cache`, `_logger` |
| Interface | Leading `I` + `UpperCamelCase` | `IUserRepository` |
| Type parameter | `T` prefix | `TEntity`, `TKey` |
| Async method | Trailing `Async` | `FetchUserAsync()` |

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
public async Task<UserProfile?> fetchUser(string userId) { }  // missing Async suffix
```

---

## 2. Code Formatting

<!-- Change values as needed for your project -->

| Item | Value |
|------|-------|
| Indentation | {4} spaces (no tabs) |
| Max line length | {120} characters |
| Brace `{` position | Next line (Allman style) |
| `using` declarations | Group at file top; remove unused usings |
| Formatter | {e.g., EditorConfig / dotnet-format} |

```csharp
// ✅ Good (Allman style)
public void ProcessOrder(Order order)
{
    if (order.IsValid)
    {
        Submit(order);
    }
}

// ❌ Bad (K&R style)
public void ProcessOrder(Order order) {
    if (order.IsValid) {
        Submit(order);
    }
}
```

---

## 3. Access Modifiers

- Use the most restrictive access modifier possible (principle of least privilege).
- Always declare access modifiers explicitly (do not omit even if the default is `private`).
- Use `internal` for intra-assembly sharing; use `public` only for intentional public APIs.

```csharp
// ✅ Good
private int _retryCount;
private void Validate(Order order) { }
public Task<Order> CreateOrderAsync(OrderRequest request) { }

// ❌ Bad
int _retryCount;          // access modifier omitted
void Validate(Order o) { }
```

---

## 4. Null Handling

- Enable nullable reference types (`#nullable enable`).
- Use the null-coalescing operator (`??`) and null-conditional operator (`?.`).
- Validate method arguments with `ArgumentNullException.ThrowIfNull`.

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

- Actively use LINQ method chaining.
- Prefer method syntax over query syntax.
- Materialize query results used multiple times with `.ToList()` / `.ToArray()`.
- Prefer `FirstOrDefault()` over `First()` and handle via null check instead of catching exceptions.

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
                  select new { u.Id, u.Name };  // not materialized

var firstAdmin = users.First(u => u.Role == Role.Admin);  // throws if not found
```

---

## 6. Asynchronous Processing (async / await)

- Use `async / await` for all I/O-bound operations.
- Do not use `async void` (except for event handlers).
- Do not use `Task.Result` / `.Wait()` (causes deadlocks).
- Accept and propagate `CancellationToken`.

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
    return _httpClient.GetAsync($"/users/{id}").Result;  // blocking call
}

public async void FetchUserAsync(string id) { }  // async void
```

---

## 7. Error Handling

- Do not throw `Exception` directly; use a purpose-specific custom exception class.
- Do not swallow exceptions with empty `catch` blocks like `catch (Exception e) { }`.
- Release resources reliably with `finally` / `using`.

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
catch { }  // swallowed exception

throw new Exception("User not found");  // generic Exception
```

---

## 8. Comment Conventions

- Add comments only where the code logic is not self-evident.
- Add **XML Doc** comments (`///`) to all public APIs.
- Write TODO / FIXME in the format `// TODO: description` and include a ticket number.

```csharp
/// <summary>
/// Retrieves the user with the specified ID.
/// </summary>
/// <param name="id">User identifier.</param>
/// <param name="ct">Cancellation token.</param>
/// <returns>The user object, or <see langword="null"/> if not found.</returns>
/// <exception cref="HttpRequestException">Thrown when the HTTP request fails.</exception>
public async Task<User?> GetByIdAsync(string id, CancellationToken ct = default)
{
    // ...
}

// TODO: #321 Implement rate limiting
```

---

## 9. Project-Specific Rules

<!-- Add project-specific rules as needed -->

- {Add project-specific rules here}
