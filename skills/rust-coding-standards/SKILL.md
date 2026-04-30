---
name: rust-coding-standards
description: 'Reference and apply Rust coding standards. Use when: applying Rust style guide, reviewing Rust code conventions, ownership, borrowing, lifetimes, error handling with Result/Option, traits, async/await, rustdoc.'
argument-hint: 'The coding standard item to review or apply (optional)'
---

# Rust Coding Standards

## Overview

This skill defines coding standards for Rust code.
Follows the [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/) and standard Rust idioms.
Adhere to these standards during code reviews and new implementations.

---

## 1. Naming Conventions

| Kind | Rule | Example |
|------|------|----|
| Functions / Methods / Variables / Modules | `snake_case` | `fetch_user()`, `user_name` |
| Types (structs / enums / traits) | `UpperCamelCase` | `UserProfile`, `ApiError` |
| Constants / `static` | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| Type parameters | Single uppercase or short `UpperCamelCase` | `T`, `Key`, `Val` |
| Lifetimes | Single lowercase or short name | `'a`, `'buf` |
| Crates / Packages | `snake_case` (hyphens also allowed) | `my_crate` |

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

## 2. Code Formatting

<!-- Adjust values according to your project -->

| Item | Setting |
|------|--------|
| Indentation | 4 spaces (`rustfmt` default) |
| Max line length | {100} characters (`rustfmt` `max_width`) |
| Formatter | `rustfmt` (apply with `cargo fmt`) |
| Linter | `clippy` (apply with `cargo clippy`) |

```toml
# rustfmt.toml
max_width = 100
edition = "2021"
```

---

## 3. Ownership, Borrowing, and Lifetimes

- Use references (`&` / `&mut`) unless ownership transfer (`move`) is required.
- Prefer immutable references (`&T`); use mutable references (`&mut T`) only when necessary.
- Only annotate lifetimes explicitly when the compiler cannot infer them.
- Use `clone()` where performance is not an issue; avoid premature optimization.

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
fn greet(name: String) -> String {  // Unnecessary ownership transfer
    format!("Hello, {}!", name)
}
```

---

## 4. Error Handling (Result / Option)

- Return `Result<T, E>` for operations that can fail.
- Return `Option<T>` for values that may not exist (distinguish between `None` and `Err`).
- `unwrap()` / `expect()` are banned in production code. Allowed only in test code.
- Use the `?` operator actively to propagate errors.
- Define custom error types with the `thiserror` crate. Use `anyhow` in application layers.

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
        .unwrap()  // Chained unwraps
}
```

---

## 5. Traits

- Abstract common behavior using traits.
- Auto-implement standard traits like `Display` / `Debug` / `Clone` / `PartialEq` with `#[derive]`.
- Prefer generics (`impl Trait`) over trait objects (`dyn Trait`).
- Leverage `default` implementations in traits.

```rust
// ✅ Good
#[derive(Debug, Clone, PartialEq)]
pub struct User {
    pub id: String,
    pub name: String,
}

// Prefer impl Trait
fn process(item: impl Displayable) { }

// ❌ Bad
fn process(item: &dyn Displayable) { }  // When dynamic dispatch is unnecessary
```

---

## 6. Async Processing (async / await)

- Use {e.g., Tokio} as the async runtime (adjust per project).
- Use `async fn` and `.await` to wait for async operations.
- Use `tokio::spawn` for parallel task execution; manage lifecycle with `JoinHandle`.
- Use `tokio::try_join!` / `tokio::join!` for parallel execution.

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
    let user  = fetch_user(user_id).await?;   // Sequential (should be parallel)
    let posts = fetch_posts(user_id).await?;
    Ok(Dashboard { user, posts })
}
```

---

## 7. Comment Conventions

- Add comments only where the code logic is not self-evident.
- Add **rustdoc** format documentation comments (`///`) to public items.
- Use `//!` for module-level documentation.
- Write TODO / FIXME as `// TODO: description` and include a ticket number.

```rust
/// Retrieves the user with the specified ID.
///
/// # Arguments
/// * `id` - User identifier
///
/// # Errors
/// * [`ApiError::Http`] - If the HTTP request fails
/// * [`ApiError::NotFound`] - If the user does not exist
///
/// # Examples
/// ```
/// let user = fetch_user("user_123").await?;
/// println!("{}", user.name);
/// ```
pub async fn fetch_user(id: &str) -> Result<User, ApiError> {
    // ...
}

// TODO: #202 Implement rate limiting
```

---

## 8. Project-Specific Rules

<!-- Add project-specific rules here -->

- {Add project-specific rules here}
