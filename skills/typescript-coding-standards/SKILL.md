---
name: typescript-coding-standards
description: 'Reference and apply TypeScript coding standards. Use when: applying TypeScript style guide, reviewing TypeScript code conventions, type definitions, generics, type guards, null handling, async/await, TSDoc.'
argument-hint: 'Coding standard item to check or apply (optional)'
---

# TypeScript Coding Standards

## Overview

This skill defines the coding standards for TypeScript code.
Follow these standards during code reviews and new implementations.

---

## 1. Naming Conventions

| Category | Rule | Example |
|----------|------|---------|
| Variable / Function / Method | `lowerCamelCase` | `fetchUserData()` |
| Class / Interface / Type / Enum | `UpperCamelCase` | `UserProfile`, `ApiError` |
| Constant (module scope) | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| Private member | Leading `#` (class field) or `_` (convention) | `#cache`, `_session` |
| File name | `kebab-case` | `user-service.ts` |
| Type parameter | Single uppercase letter or descriptive name | `T`, `TKey`, `TValue` |

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

## 2. Code Formatting

<!-- Change values as needed for your project -->

| Item | Value |
|------|-------|
| Indentation | {2} spaces (no tabs) |
| Max line length | {120} characters |
| String quotes | {prefer single quotes `'`} |
| Semicolons | {use semicolons} |
| Trailing commas | Add trailing commas in multi-line expressions |
| Formatter | {e.g., Prettier} |
| Linter | {e.g., ESLint + typescript-eslint} |

---

## 3. Type Definitions (interface vs type alias)

- **`interface`** is used for object type and class contract definitions. Use `interface` when declaration merging is needed as well.
- **`type`** is used for union types, intersection types, primitive type aliases, and tuples.
- Do not use `any`. If unavoidable, use `unknown` and narrow with a type guard.

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
type User = {  // using type for an object type (reduces extensibility)
  id: string;
};
const response: any = await fetch('/api');  // using any
```

---

## 4. Generics

- Use meaningful type parameter names (single letters only for general-purpose containers).
- Avoid unnecessary generics.
- Actively use constraints (`extends`) on type parameters.

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

## 5. Type Assertions & Type Guards

- Use type assertions (`as`) only when unavoidable.
- Actively use user-defined type guards (`is` keyword).
- Double-step assertions such as `as unknown as T` are prohibited.

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
const user = response as User;  // assertion without validation
const user2 = response as unknown as User;  // double-step assertion
```

---

## 6. null / undefined Handling

- Enable `strict` mode; always enable `strictNullChecks`.
- Use `null` and `undefined` explicitly and distinctly (non-existent values → `undefined`; intentional empty values → `null`).
- Use optional chaining (`?.`) and the null-coalescing operator (`??`).

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

## 7. Error Handling

- Use custom error classes that extend `Error`.
- Receive `catch` arguments as `unknown` and narrow with a type guard.
- Do not swallow errors.

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
  console.log(e);  // used without type narrowing
}
```

---

## 8. Asynchronous Processing

- Prefer `async / await`. Avoid `.then()` chains.
- Use `Promise.all` / `Promise.allSettled` for parallel execution.
- Explicitly declare the return type as `Promise<T>`.

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

## 9. Comment Conventions

- Add comments only where the code logic is not self-evident.
- Add **TSDoc** documentation comments to all public APIs.
- Write TODO / FIXME in the format `// TODO: description` and include a ticket number.

```ts
/**
 * Retrieves the user with the specified ID.
 * @param id - User identifier
 * @returns User object
 * @throws {@link ApiError} if the user does not exist
 */
async function fetchUser(id: string): Promise<User> {
  // ...
}

// TODO: #456 Implement caching strategy
```

---

## 10. Project-Specific Rules

<!-- Add project-specific rules as needed -->

- {Add project-specific rules here}
