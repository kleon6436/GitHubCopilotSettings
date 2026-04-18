---
name: cpp-coding-standards
description: 'C++のコーディング規約を参照・適用する。C++ コーディング規約、命名規則、スタイルガイド、メモリ管理、スマートポインタ、const/constexpr、ヘッダーファイル規約、モダンC++（C++17/20）、エラーハンドリングを確認・適用したいときに使用。Use when: applying C++ style guide, reviewing C++ code conventions, smart pointers, RAII, const correctness, header file conventions, modern C++ features, exception handling, Doxygen.'
argument-hint: '確認・適用したいコーディング規約の項目（省略可）'
---

# C++ コーディング規約

## 概要

このスキルは C++ コードのコーディング規約を定義します。
モダン C++（C++17 以降）を前提としています。
コードレビュー・新規実装の際はこの規約に従ってください。

---

## 1. 命名規則

| 種別 | 規則 | 例 |
|------|------|----|
| クラス / 構造体 / 列挙型 | `UpperCamelCase` | `UserProfile`, `NetworkManager` |
| 関数 / メソッド | `lowerCamelCase` | `fetchUser()`, `parseResponse()` |
| 変数（ローカル / 引数） | `snake_case` | `user_name`, `retry_count` |
| メンバー変数（プライベート） | 末尾 `_` | `cache_`, `logger_` |
| 定数 / `constexpr` | `kUpperCamelCase` | `kMaxRetryCount` |
| マクロ（最小限） | `UPPER_SNAKE_CASE` | `NDEBUG` |
| 名前空間 | `snake_case`（短く） | `network`, `app::core` |

```cpp
// ✅ Good
namespace app::network {

class NetworkManager {
 public:
  void fetchUser(std::string_view user_id);

 private:
  static constexpr int kMaxRetries = 3;
  std::unique_ptr<HttpClient> client_;
};

}  // namespace app::network

// ❌ Bad
class networkmanager {
 public:
  void FetchUser(std::string UserId);
  std::unique_ptr<HttpClient> Client;  // パブリックメンバー
};
```

---

## 2. コードフォーマット

<!-- プロジェクトに応じて値を変更してください -->

| 項目 | 設定値 |
|------|--------|
| インデント | スペース {2} 個（タブ不可） |
| 1行の最大文字数 | {100} 文字 |
| 中括弧 `{` の位置 | {例: K&R スタイル（行末）/ Allman（次行）} |
| フォーマッター | {例: clang-format} |
| リンター | {例: clang-tidy} |

```cpp
// ✅ Good（K&R スタイルの例）
void NetworkManager::fetchUser(std::string_view user_id) {
  if (user_id.empty()) {
    throw std::invalid_argument("user_id must not be empty");
  }
  // ...
}
```

---

## 3. メモリ管理（RAII・スマートポインタ）

- 生ポインタによる動的メモリ管理は禁止。スマートポインタを使用する。
- 所有権が明確な場合は `std::unique_ptr`、共有が必要な場合は `std::shared_ptr` を使用する。
- `new` / `delete` は直接使用しない。`std::make_unique` / `std::make_shared` を使う。
- リソース管理は RAII イディオムに従う。

```cpp
// ✅ Good
auto client = std::make_unique<HttpClient>(base_url);
auto cache  = std::make_shared<Cache>();

// ❌ Bad
HttpClient* client = new HttpClient(base_url);  // 生ポインタ
// delete 忘れのリスク
```

---

## 4. const / constexpr

- 変更しない変数・引数には必ず `const` を付ける（const 正確性）。
- コンパイル時定数には `constexpr` を使用する（`#define` マクロは使わない）。
- メンバー関数がオブジェクトを変更しない場合は `const` を付ける。
- 文字列引数は `const std::string&` より `std::string_view` を優先する。

```cpp
// ✅ Good
constexpr int kBufferSize = 1024;

class Parser {
 public:
  std::string parse(std::string_view input) const;
};

void log(std::string_view message);

// ❌ Bad
#define BUFFER_SIZE 1024  // マクロ定数
void log(std::string message);  // 不必要なコピー
```

---

## 5. ヘッダーファイル規約

- すべてのヘッダーファイルに **インクルードガード**（`#pragma once`）を使用する。
- 前方宣言（forward declaration）を積極的に使用してインクルード依存を減らす。
- ヘッダーファイルにはインターフェースのみ記述し、実装は `.cpp` ファイルに書く。
- インクルード順序: 対応する `.h`、標準ライブラリ、サードパーティ、プロジェクト内。

```cpp
// ✅ Good（user_service.h）
#pragma once

#include <memory>
#include <string>

// 前方宣言（インクルード不要）
class HttpClient;
class User;

class UserService {
 public:
  explicit UserService(std::shared_ptr<HttpClient> client);
  User fetchUser(std::string_view id) const;

 private:
  std::shared_ptr<HttpClient> client_;
};

// ❌ Bad
#ifndef USER_SERVICE_H  // 古いインクルードガード（#pragma once を使う）
#define USER_SERVICE_H
#include "http_client.h"  // 前方宣言で十分なのにインクルード
```

---

## 6. モダン C++（C++17 / C++20 機能の活用）

- 範囲 `for` ループを使用する（インデックスループより優先）。
- 構造化束縛（`auto [a, b] = ...`）を活用する。
- `std::optional` で値の存在を表現する（ポインタや番兵値は使わない）。
- ラムダ式でキャプチャは最小限にする（`[&]` の乱用禁止）。

```cpp
// ✅ Good
for (const auto& [key, value] : config_map) {
  process(key, value);
}

std::optional<User> findUser(std::string_view id) {
  auto it = cache_.find(id);
  if (it == cache_.end()) return std::nullopt;
  return it->second;
}

auto transform = [](const std::string& s) { return s.size(); };  // 必要なもののみキャプチャ

// ❌ Bad
for (int i = 0; i < config_map.size(); ++i) { /* ... */ }

User* findUser(std::string_view id) {
  return nullptr;  // ポインタで存在を表現
}

auto transform = [&]() { return data_.size(); };  // [&] の乱用
```

---

## 7. エラーハンドリング

- エラーは例外（`std::exception` 派生クラス）で伝播させる。
- 戻り値でエラーを表現する場合は `std::expected`（C++23）または `std::optional` を使用する。
- エラーコードの整数値をそのまま戻り値にしない。
- デストラクタで例外をスローしない（`noexcept` を付ける）。

```cpp
// ✅ Good
class NetworkException : public std::runtime_error {
 public:
  explicit NetworkException(std::string_view message, int status_code)
      : std::runtime_error(std::string(message)), status_code_(status_code) {}
  int statusCode() const noexcept { return status_code_; }
 private:
  int status_code_;
};

try {
  auto user = service.fetchUser(user_id);
} catch (const NetworkException& e) {
  logger.error("Network error {}: {}", e.statusCode(), e.what());
}

// ❌ Bad
int fetchUser(const std::string& id);  // 戻り値がエラーコード
```

---

## 8. コメント規約

- コードのロジックが自明でない箇所にのみコメントを付ける。
- 公開 API には **Doxygen** 形式のドキュメントコメントを付ける。
- TODO / FIXME は `// TODO: 説明` の形式で記述し、チケット番号を添える。

```cpp
/**
 * @brief 指定した ID のユーザーを取得します。
 * @param id ユーザー識別子
 * @return ユーザーオブジェクト
 * @throws NetworkException HTTP リクエストが失敗した場合
 * @throws std::invalid_argument id が空の場合
 */
User fetchUser(std::string_view id) const;

// TODO: #101 キャッシュ TTL の設定を追加する
```

---

## 9. プロジェクト固有のルール

<!-- プロジェクトに応じて追記してください -->

- {プロジェクト固有のルールをここに記載}
