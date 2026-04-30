---
name: javascript-coding-standards
description: 'Reference and apply JavaScript coding standards. Use when: applying JavaScript style guide, reviewing JavaScript code conventions, naming rules, variable declarations, async/await, error handling patterns, JSDoc.'
argument-hint: 'The coding standard item to check or apply (optional)'
---

# JavaScript Coding Standards

## Overview

This skill defines coding standards for JavaScript code.
Follow these conventions during code reviews and new implementations.

---

## 1. Naming Conventions

| Type | Rule | Example |
|------|------|---------|
| Variables / Functions / Methods | `lowerCamelCase` | `fetchUserData()` |
| Classes / Constructors | `UpperCamelCase` | `UserProfile` |
| Constants (module scope) | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| Private members (convention) | Leading `_` | `_cache` |
| File names | `kebab-case` | `user-service.js` |

```js
// ✅ Good
const MAX_TIMEOUT = 30;

class NetworkManager {
  constructor() {
    this._session = null;
  }

  fetchData(endpoint) { }
}

// ❌ Bad
const maxTimeout = 30;
class networkmanager { }
function FetchData() { }
```

---

## 2. Code Formatting

<!-- Adjust values to match your project -->

| Item | Setting |
|------|--------|
| Indentation | {2} spaces (no tabs) |
| Max line length | {120} characters |
| String quotes | {Prefer single quotes `'`} |
| Semicolons | {yes / no} |
| Trailing commas | Add trailing commas for multi-line |
| Formatter | {e.g. Prettier} |
| Linter | {e.g. ESLint} |

```js
// ✅ Good (trailing comma in multi-line)
const SUPPORTED_FORMATS = [
  'json',
  'csv',
  'xml',
];

// ❌ Bad
const SUPPORTED_FORMATS = ['json', 'csv',
  'xml']
```

---

## 3. Variable Declarations

- Prefer `const`. Use `let` only when reassignment is needed.
- Do not use `var`.

```js
// ✅ Good
const userId = 'abc123';
let retryCount = 0;

// ❌ Bad
var userId = 'abc123';
let userId2 = 'abc123'; // let without reassignment
```

---

## 4. Functions

- Use **arrow functions** for callbacks and short functions.
- Use **`function` declarations** for named functions at the module top level.
- Make use of default parameters.

```js
// ✅ Good
function fetchUser(id) {
  return apiClient.get(`/users/${id}`);
}

const doubled = numbers.map((n) => n * 2);

function createUser(name, role = 'viewer') { }

// ❌ Bad
const fetchUser = function(id) { /* ... */ };  // anonymous function at top level
const doubled = numbers.map(function(n) { return n * 2; });
```

---

## 5. Modules

- Use ES Modules (`import` / `export`). Do not use `require`.
- Use `default export` only when a file exports a single class or function.
- Prefer **named exports** for multiple exports.

```js
// ✅ Good
export function fetchUser(id) { }
export const MAX_RETRY = 3;

import { fetchUser, MAX_RETRY } from './user-service.js';

// ❌ Bad
module.exports = { fetchUser };
const { fetchUser } = require('./user-service');
```

---

## 6. Error Handling

- Use `try / catch` and do not swallow errors.
- Classify errors using custom classes that extend `Error`.
- Always log appropriately or rethrow in `catch` blocks.

```js
// ✅ Good
class ApiError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = 'ApiError';
    this.statusCode = statusCode;
  }
}

try {
  const data = await fetchData();
} catch (error) {
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
  // Do nothing
}
```

---

## 7. Async Processing

- Prefer `async / await`. Avoid `.then()` chains.
- Use `Promise.all` / `Promise.allSettled` for parallel processing.
- Be careful not to forget `await` (check the return type).

```js
// ✅ Good
async function loadDashboard(userId) {
  const [user, posts] = await Promise.all([
    fetchUser(userId),
    fetchPosts(userId),
  ]);
  return { user, posts };
}

// ❌ Bad
function loadDashboard(userId) {
  return fetchUser(userId).then((user) => {
    return fetchPosts(userId).then((posts) => ({ user, posts }));
  });
}
```

---

## 8. Comment Conventions

- Add comments only where code logic is not self-evident.
- Add **JSDoc** documentation comments to public APIs.
- Write TODO / FIXME in the format `// TODO: description` and include a ticket number.

```js
/**
 * Fetches the user with the specified ID.
 * @param {string} id - User identifier
 * @returns {Promise<User>} User object
 * @throws {ApiError} When the user does not exist
 */
async function fetchUser(id) {
  // ...
}

// TODO: #456 Implement cache strategy
```

---

## 9. Project-Specific Rules

<!-- Add project-specific items here -->

- {Add project-specific rules here}
