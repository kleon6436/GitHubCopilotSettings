---
name: cpp-coding-standards
description: 'Reference and apply C++ coding standards. Use when: applying C++ style guide, reviewing C++ code conventions, smart pointers, RAII, const correctness, header file conventions, modern C++ features, exception handling, Doxygen.'
argument-hint: 'Coding standard item to review or apply (optional)'
---

# C++ Coding Standards

## Overview

This skill defines coding standards for C++ code.
Assumes modern C++ (C++17 or later).
Follow these conventions during code reviews and new implementations.

---

## 1. Naming Conventions

| Type | Convention | Example |
|------|------|-----|
| Class / Struct / Enum | `UpperCamelCase` | `UserProfile`, `NetworkManager` |
| Function / Method | `lowerCamelCase` | `fetchUser()`, `parseResponse()` |
| Variable (local / parameter) | `snake_case` | `user_name`, `retry_count` |
| Member variable (private) | trailing `_` | `cache_`, `logger_` |
| Constant / `constexpr` | `kUpperCamelCase` | `kMaxRetryCount` |
| Macro (minimal use) | `UPPER_SNAKE_CASE` | `NDEBUG` |
| Namespace | `snake_case` (short) | `network`, `app::core` |

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
  std::unique_ptr<HttpClient> Client;  // public member
};
```

---

## 2. Code Formatting

<!-- Adjust values as needed for your project -->

| Item | Value |
|------|-----|
| Indent | {2} spaces (no tabs) |
| Max line length | {100} characters |
| Brace `{` placement | {e.g. K&R style (end of line) / Allman (next line)} |
| Formatter | {e.g. clang-format} |
| Linter | {e.g. clang-tidy} |

```cpp
// ✅ Good (K&R style example)
void NetworkManager::fetchUser(std::string_view user_id) {
  if (user_id.empty()) {
    throw std::invalid_argument("user_id must not be empty");
  }
  // ...
}
```

---

## 3. Memory Management (RAII / Smart Pointers)

- Raw pointer dynamic memory management is prohibited. Use smart pointers.
- Use `std::unique_ptr` for exclusive ownership, `std::shared_ptr` when sharing is required.
- Do not use `new` / `delete` directly. Use `std::make_unique` / `std::make_shared`.
- Follow the RAII idiom for resource management.

```cpp
// ✅ Good
auto client = std::make_unique<HttpClient>(base_url);
auto cache  = std::make_shared<Cache>();

// ❌ Bad
HttpClient* client = new HttpClient(base_url);  // raw pointer
// Risk of forgetting to delete
```

---

## 4. const / constexpr

- Always mark variables and parameters that are not modified with `const` (const correctness).
- Use `constexpr` for compile-time constants (do not use `#define` macros).
- Mark member functions that do not modify the object with `const`.
- Prefer `std::string_view` over `const std::string&` for string parameters.

```cpp
// ✅ Good
constexpr int kBufferSize = 1024;

class Parser {
 public:
  std::string parse(std::string_view input) const;
};

void log(std::string_view message);

// ❌ Bad
#define BUFFER_SIZE 1024  // macro constant
void log(std::string message);  // unnecessary copy
```

---

## 5. Header File Conventions

- Use an **include guard** (`#pragma once`) in all header files.
- Actively use forward declarations to reduce include dependencies.
- Header files should contain only the interface; implementations go in `.cpp` files.
- Include order: corresponding `.h`, standard library, third-party, project-internal.

```cpp
// ✅ Good (user_service.h)
#pragma once

#include <memory>
#include <string>

// Forward declarations (no include needed)
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
#ifndef USER_SERVICE_H  // old-style include guard (use #pragma once)
#define USER_SERVICE_H
#include "http_client.h"  // forward declaration is sufficient here
```

---

## 6. Modern C++ (Leveraging C++17 / C++20 Features)

- Use range-based `for` loops (prefer over index-based loops).
- Use structured bindings (`auto [a, b] = ...`).
- Use `std::optional` to represent optional values (do not use pointers or sentinel values).
- Keep lambda captures minimal (avoid overusing `[&]`).

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

auto transform = [](const std::string& s) { return s.size(); };  // capture only what is needed

// ❌ Bad
for (int i = 0; i < config_map.size(); ++i) { /* ... */ }

User* findUser(std::string_view id) {
  return nullptr;  // use pointer to represent optionality
}

auto transform = [&]() { return data_.size(); };  // overuse of [&]
```

---

## 7. Error Handling

- Propagate errors using exceptions (derived from `std::exception`).
- When representing errors as return values, use `std::expected` (C++23) or `std::optional`.
- Do not return raw integer error codes as return values.
- Do not throw exceptions in destructors (mark with `noexcept`).

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
int fetchUser(const std::string& id);  // return value is an error code
```

---

## 8. Comment Conventions

- Add comments only where the logic is not self-evident.
- Add **Doxygen** documentation comments to all public APIs.
- Write TODO / FIXME in the format `// TODO: description` and include a ticket number.

```cpp
/**
 * @brief Retrieves the user with the specified ID.
 * @param id User identifier
 * @return User object
 * @throws NetworkException If the HTTP request fails
 * @throws std::invalid_argument If id is empty
 */
User fetchUser(std::string_view id) const;

// TODO: #101 Add cache TTL configuration
```

---

## 9. Project-Specific Rules

<!-- Add project-specific content here -->

- {Add project-specific rules here}
