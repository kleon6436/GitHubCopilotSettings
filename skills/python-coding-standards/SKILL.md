---
name: python-coding-standards
description: 'Reference and apply Python coding standards. Use when: applying Python style guide, reviewing Python code conventions, PEP8, type hints, import order, docstring format.'
argument-hint: 'Coding standard item to review or apply (optional)'
---

# Python Coding Standards

## Overview

This skill defines coding standards for Python code.
Follows [PEP 8](https://peps.python.org/pep-0008/) as the baseline, with additional project-specific rules.
Follow these conventions during code reviews and new implementations.

---

## 1. Naming Conventions

| Type | Convention | Example |
|------|------|-----|
| Module / Package | `snake_case` (short & concise) | `user_service.py` |
| Class | `UpperCamelCase` | `UserProfile` |
| Function / Method | `snake_case` | `fetch_user_data()` |
| Variable / Parameter | `snake_case` | `user_name` |
| Constant | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT = 3` |
| Private Member | leading `_` | `_cache: dict` |
| Name Mangling | leading `__` (only when necessary) | `__secret` |

```python
# ✅ Good
class NetworkManager:
    MAX_TIMEOUT = 30

    def __init__(self) -> None:
        self._session: httpx.AsyncClient | None = None

    def fetch_data(self, endpoint: str) -> bytes:
        ...

# ❌ Bad
class networkmanager:
    maxTimeout = 30
    def FetchData(self, Endpoint):
        ...
```

---

## 2. Code Formatting

<!-- Adjust values as needed for your project -->

| Item | Value |
|------|-----|
| Indent | {4} spaces (no tabs) |
| Max line length | {120} characters |
| String quotes | {prefer double quotes `"`} |
| Trailing comma | Add trailing commas for multi-line |
| Formatter | {e.g. Ruff / Black} |

```python
# ✅ Good (trailing comma for multi-line)
SUPPORTED_FORMATS = [
    "json",
    "csv",
    "xml",
]

# ❌ Bad
SUPPORTED_FORMATS = ["json", "csv",
    "xml"]
```

---

## 3. Type Hints

- Add type hints to all function/method parameters and return values.
- Use Python 3.10+ `X | Y` syntax (do not use `Union[X, Y]`).
- Use `X | None` instead of `Optional[X]`.
- Use `mypy` or `pyright` for type checking (strict mode recommended).

```python
# ✅ Good
def get_user(user_id: int) -> UserProfile | None:
    ...

def process_items(items: list[str], limit: int = 10) -> dict[str, int]:
    ...

# ❌ Bad
def get_user(user_id):
    ...

from typing import Optional, Union
def get_user(user_id: int) -> Optional[UserProfile]:
    ...
```

---

## 4. Import Order

Write imports in the following order, separated by blank lines (auto-formatted by `isort` / `Ruff`).

1. Standard library
2. Third-party libraries
3. Local (project) modules

```python
# ✅ Good
import os
import sys
from pathlib import Path

import httpx
from fastapi import FastAPI

from app.models import UserProfile
from app.services import user_service
```

Wildcard imports (`from module import *`) are prohibited.

---

## 5. Docstring Conventions

- Add docstrings to all public classes, functions, and methods.
- Use **{Google style / NumPy style}** format.

```python
# ✅ Good (Google style)
def fetch_user(user_id: int) -> UserProfile:
    """Retrieves user information for the specified ID.

    Args:
        user_id: Identifier of the target user.

    Returns:
        The UserProfile object for the target user.

    Raises:
        UserNotFoundError: If no user with the given ID exists.
        NetworkError: If the network connection fails.
    """
    ...
```

---

## 6. Error Handling

- Catch specific exception classes with `except` (`except Exception` is generally prohibited).
- Define project-specific exceptions by inheriting from a base class.
- Always log or re-raise in `except` blocks.

```python
# ✅ Good
class AppError(Exception):
    """Base application exception class."""

class UserNotFoundError(AppError):
    """Exception raised when a user is not found."""

def get_user(user_id: int) -> UserProfile:
    try:
        return repository.find(user_id)
    except DatabaseConnectionError as e:
        logger.error("DB connection error: %s", e)
        raise
    except RecordNotFoundError as e:
        raise UserNotFoundError(f"User {user_id} not found") from e

# ❌ Bad
def get_user(user_id: int):
    try:
        return repository.find(user_id)
    except Exception:
        pass
```

---

## 7. Class Design

- Use `dataclass` or `pydantic.BaseModel` for data-holding classes.
- Prefer `@dataclass` when `__init__` only stores data.

```python
# ✅ Good
from dataclasses import dataclass, field

@dataclass
class UserProfile:
    user_id: int
    name: str
    email: str
    tags: list[str] = field(default_factory=list)
```

---

## 8. Asynchronous Processing

- Use `async/await` for I/O-bound operations.
- Use `asyncio.gather` for concurrent execution.
- Use async context managers (`async with`) appropriately.

```python
# ✅ Good
async def fetch_all(user_ids: list[int]) -> list[UserProfile]:
    tasks = [fetch_user(uid) for uid in user_ids]
    return await asyncio.gather(*tasks)
```

---

## 9. Testing Conventions

<!-- Adjust according to the test framework used in your project -->

- Test framework: **{pytest}**
- Place test files in the `tests/` directory following the `test_*.py` naming convention.
- Name test functions using the `test_<subject>_<condition>_<expected_result>` format.
- Define fixtures in `conftest.py`.

```python
# ✅ Good
def test_fetch_user_with_valid_id_returns_profile() -> None:
    user = fetch_user(user_id=1)
    assert user.name == "Alice"

def test_fetch_user_with_invalid_id_raises_not_found() -> None:
    with pytest.raises(UserNotFoundError):
        fetch_user(user_id=-1)
```

---

## 10. Project-Specific Rules

<!-- Add project-specific content here -->

- {Add project-specific rules here}
