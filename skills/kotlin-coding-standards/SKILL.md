---
name: kotlin-coding-standards
description: 'Kotlinのコーディング規約を参照・適用する。Kotlin コーディング規約、命名規則、スタイルガイド、null安全性、データクラス、sealed class、拡張関数、コルーチン、Flow、エラーハンドリング、KDocコメントを確認・適用したいときに使用。Use when: applying Kotlin style guide, reviewing Kotlin code conventions, null safety, data class, sealed class, extension functions, coroutines, Flow, error handling, KDoc.'
argument-hint: '確認・適用したいコーディング規約の項目（省略可）'
---

# Kotlin コーディング規約

## 概要

このスキルは Kotlin コードのコーディング規約を定義します。
基本的に [Kotlin コーディング規約](https://kotlinlang.org/docs/coding-conventions.html) に準拠しつつ、プロジェクト固有のルールを追記しています。
コードレビュー・新規実装の際はこの規約に従ってください。

---

## 1. 命名規則

| 種別 | 規則 | 例 |
|------|------|----|
| クラス / インターフェース / オブジェクト / 列挙型 | `UpperCamelCase` | `UserProfile`, `UserRepository` |
| 関数 / メソッド / 変数 / 引数 | `lowerCamelCase` | `fetchUser()`, `userName` |
| 定数（`companion object` / `top-level`） | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| プロパティ（バッキングフィールド） | 先頭 `_` + `lowerCamelCase` | `_uiState` |
| ファイル名 | `UpperCamelCase.kt`（クラスと一致） | `UserProfile.kt` |
| 拡張関数ファイル | `対象型Extensions.kt` | `StringExtensions.kt` |

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

## 2. コードフォーマット

<!-- プロジェクトに応じて値を変更してください -->

| 項目 | 設定値 |
|------|--------|
| インデント | スペース {4} 個（タブ不可） |
| 1行の最大文字数 | {120} 文字 |
| 末尾カンマ | 複数行の場合は末尾カンマを付ける |
| フォーマッター | {例: ktfmt / ktlint} |

```kotlin
// ✅ Good（複数行の末尾カンマ）
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

## 3. null 安全性

- `!!`（非 null アサーション）は原則禁止。
- `?.`（safe call）と `?:`（Elvis 演算子）を積極的に使用する。
- `lateinit var` はライフサイクルが保証されている場合のみ使用し、アクセス前に `::prop.isInitialized` でチェックする。
- 関数の戻り値が存在しない場合は `null` ではなく `sealed class` / `Result` で表現することを検討する。

```kotlin
// ✅ Good
val displayName = user?.profile?.displayName ?: "Anonymous"
val length = items?.size ?: 0

// ❌ Bad
val displayName = user!!.profile!!.displayName  // !! による強制アンラップ
```

---

## 4. データクラス・sealed class

- 不変のデータ転送オブジェクト（DTO）には `data class` を使用する。
- 状態（UI State、Result 等）の表現には `sealed class` / `sealed interface` を使用する。
- `data class` の `copy()` を活用して状態更新を行う。

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

// when 式で網羅的に処理
when (uiState) {
    is UiState.Loading -> showSkeleton()
    is UiState.Success -> showUser(uiState.user)
    is UiState.Error   -> showError(uiState.message)
}

// ❌ Bad
class User(var id: String, var name: String)  // 可変・data class でない
```

---

## 5. 拡張関数・拡張プロパティ

- 拡張関数はファイル単位でまとめる（`XxxExtensions.kt`）。
- 拡張先の型の責務に関係する処理のみを拡張として定義する。
- `this` の省略が可読性を下げる場合は省略しない。

```kotlin
// ✅ Good（StringExtensions.kt）
fun String.toTitleCase(): String =
    split(" ").joinToString(" ") { word ->
        word.replaceFirstChar { it.uppercaseChar() }
    }

// ❌ Bad（ビジネスロジックを拡張関数に詰め込む）
fun User.sendPasswordResetEmail(service: EmailService) {
    service.send(this.email, "パスワードリセット")
}
```

---

## 6. コルーチン（Coroutines / Flow）

- `suspend` 関数はメインセーフになるよう設計する。
- `withContext(Dispatchers.IO)` で I/O 処理をディスパッチする。
- UI の状態は `StateFlow` で管理し、`LiveData` よりも優先する。
- `viewModelScope` / `lifecycleScope` でコルーチンのライフサイクルを管理する。
- `GlobalScope` は使用しない。

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
GlobalScope.launch { /* ... */ }  // GlobalScope の使用
```

---

## 7. エラーハンドリング

- `try / catch` より `runCatching` を積極的に使用する。
- 例外は `Exception` を継承したカスタムクラスで分類する。
- `catch` ブロックで例外を握りつぶさない。

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
    // 握りつぶし
}
```

---

## 8. コメント規約

- コードのロジックが自明でない箇所にのみコメントを付ける。
- 公開 API には **KDoc** 形式のドキュメントコメントを付ける。
- TODO / FIXME は `// TODO: 説明` の形式で記述し、チケット番号を添える。

```kotlin
/**
 * 指定した ID のユーザーを取得します。
 *
 * @param userId ユーザー識別子
 * @return ユーザーオブジェクト
 * @throws ApiException HTTP リクエストが失敗した場合
 */
suspend fun fetchUser(userId: String): User {
    // ...
}

// TODO: #654 オフラインキャッシュを実装する
```

---

## 9. プロジェクト固有のルール

<!-- プロジェクトに応じて追記してください -->

- {プロジェクト固有のルールをここに記載}
