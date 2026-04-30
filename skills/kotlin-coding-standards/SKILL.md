---
name: kotlin-coding-standards
description: 'Reference and apply Kotlin coding standards. Use when: applying Kotlin style guide, reviewing Kotlin code conventions, null safety, data class, sealed class, extension functions, coroutines, Flow, error handling, KDoc.'
argument-hint: 'Coding standard item to check or apply (optional)'
---

# Kotlin Coding Standards

## Overview

This skill defines coding standards for Kotlin code.
It is based on the [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html) with project-specific rules added.
Follow these conventions during code review and new implementation.

---

## 1. Naming Conventions

| Type | Convention | Example |
|------|------|----|
| Class / Interface / Object / Enum | `UpperCamelCase` | `UserProfile`, `UserRepository` |
| Function / Method / Variable / Parameter | `lowerCamelCase` | `fetchUser()`, `userName` |
| Constant (`companion object` / `top-level`) | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| Property (backing field) | Leading `_` + `lowerCamelCase` | `_uiState` |
| File name | `UpperCamelCase.kt` (matches class name) | `UserProfile.kt` |
| Extension function file | `TargetTypeExtensions.kt` | `StringExtensions.kt` |

```kotlin
// ✅ Good
class NetworkManager {
    companion object {
        const val MAX_RETRY_COUNT = 3
    }

    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState = _uiState.asStateFlow()

    suspend fun fetchUser(userId: String): User { TODO() }
}

// ❌ Bad
class networkmanager { }
fun FetchUser(UserId: String): User { TODO() }
```

---

## 2. Code Formatting

<!-- Change values as needed for your project -->

| Item | Setting |
|------|--------|
| Indent | {4} spaces (no tabs) |
| Max line length | {120} characters |
| Trailing comma | Add trailing comma for multi-line |
| Formatter | {e.g., ktfmt / ktlint} |

```kotlin
// ✅ Good (trailing comma for multi-line)
val supportedFormats = listOf(
    "json",
    "csv",
    "xml",
)

// ❌ Bad
val supportedFormats = listOf("json", "csv",
    "xml")
```

---

## 3. Null Safety

- `!!` (non-null assertion) is generally prohibited.
- Use `?.` (safe call) and `?:` (Elvis operator) proactively.
- Use `lateinit var` only when the lifecycle is guaranteed, and check with `::prop.isInitialized` before access.
- When a function may not return a value, consider using `sealed class` / `Result` instead of `null`.

```kotlin
// ✅ Good
val displayName = user?.profile?.displayName ?: "Anonymous"
val length = items?.size ?: 0

// ❌ Bad
val displayName = user!!.profile!!.displayName  // Force unwrap with !!
```

---

## 4. Data Classes and Sealed Classes

- Use `data class` for immutable data transfer objects (DTOs).
- Use `sealed class` / `sealed interface` to represent states (UI State, Result, etc.).
- Use `data class`'s `copy()` to perform state updates.

```kotlin
// ✅ Good
data class User(
    val id: String,
    val name: String,
    val email: String,
)

sealed interface UiState {
    data object Loading : UiState
    data class Success(val user: User) : UiState
    data class Error(val message: String) : UiState
}

// Handle exhaustively with when expression
when (uiState) {
    is UiState.Loading -> showSkeleton()
    is UiState.Success -> showUser(uiState.user)
    is UiState.Error   -> showError(uiState.message)
}

// ❌ Bad
class User(var id: String, var name: String)  // Mutable, not a data class
```

---

## 5. Extension Functions and Extension Properties

- Group extension functions in a dedicated file (`XxxExtensions.kt`).
- Only define extensions that are relevant to the responsibility of the extended type.
- Do not omit `this` if omitting it reduces readability.

```kotlin
// ✅ Good (StringExtensions.kt)
fun String.toTitleCase(): String =
    split(" ").joinToString(" ") { word ->
        word.replaceFirstChar { it.uppercaseChar() }
    }

// ❌ Bad (cramming business logic into extension functions)
fun User.sendPasswordResetEmail(service: EmailService) {
    service.send(this.email, "Password Reset")
}
```

---

## 6. Coroutines and Flow

- Design `suspend` functions to be main-safe.
- Dispatch I/O operations with `withContext(Dispatchers.IO)`.
- Manage UI state with `StateFlow`, preferring it over `LiveData`.
- Manage coroutine lifecycle with `viewModelScope` / `lifecycleScope`.
- Do not use `GlobalScope`.

```kotlin
// ✅ Good
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState = _uiState.asStateFlow()

    fun loadUser(userId: String) {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            runCatching { repository.fetchUser(userId) }
                .onSuccess { _uiState.value = UiState.Success(it) }
                .onFailure { _uiState.value = UiState.Error(it.message ?: "Unknown error") }
        }
    }
}

// ❌ Bad
GlobalScope.launch { /* ... */ }  // Using GlobalScope
```

---

## 7. Error Handling

- Prefer `runCatching` over `try / catch`.
- Classify exceptions with custom classes that extend `Exception`.
- Do not swallow exceptions in `catch` blocks.

```kotlin
// ✅ Good
class ApiException(message: String, val statusCode: Int) : Exception(message)

val result = runCatching { repository.fetchUser(userId) }
result
    .onSuccess { user -> processUser(user) }
    .onFailure { e ->
        when (e) {
            is ApiException -> logger.error("API error ${e.statusCode}: ${e.message}")
            else            -> throw e
        }
    }

// ❌ Bad
try {
    repository.fetchUser(userId)
} catch (e: Exception) {
    // Swallowed exception
}
```

---

## 8. Comment Conventions

- Add comments only where the code logic is not self-evident.
- Add **KDoc** documentation comments to public APIs.
- Write TODO / FIXME comments in the format `// TODO: description` and include a ticket number.

```kotlin
/**
 * Retrieves the user with the specified ID.
 *
 * @param userId User identifier
 * @return User object
 * @throws ApiException When the HTTP request fails
 */
suspend fun fetchUser(userId: String): User {
    // ...
}

// TODO: #654 Implement offline cache
```

---

## 9. Project-Specific Rules

<!-- Add project-specific rules here -->

- {Add project-specific rules here}
