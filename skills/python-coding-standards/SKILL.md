---
name: python-coding-standards
description: 'Pythonのコーディング規約を参照・適用する。Python コーディング規約、PEP8、命名規則、スタイルガイド、型ヒント、インポート順序、docstring、エラーハンドリングを確認・適用したいときに使用。Use when: applying Python style guide, reviewing Python code conventions, PEP8, type hints, import order, docstring format.'
argument-hint: '確認・適用したいコーディング規約の項目（省略可）'
---

# Python コーディング規約

## 概要

このスキルは Python コードのコーディング規約を定義します。
基本的に [PEP 8](https://peps.python.org/pep-0008/) に準拠しつつ、プロジェクト固有のルールを追記しています。
コードレビュー・新規実装の際はこの規約に従ってください。

---

## 1. 命名規則

| 種別 | 規則 | 例 |
|------|------|----|
| モジュール / パッケージ | `snake_case`（短く・簡潔に） | `user_service.py` |
| クラス | `UpperCamelCase` | `UserProfile` |
| 関数 / メソッド | `snake_case` | `fetch_user_data()` |
| 変数 / 引数 | `snake_case` | `user_name` |
| 定数 | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT = 3` |
| プライベートメンバ | 先頭 `_` | `_cache: dict` |
| 名前マングリング | 先頭 `__`（必要な場合のみ） | `__secret` |

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

## 2. コードフォーマット

<!-- プロジェクトに応じて値を変更してください -->

| 項目 | 設定値 |
|------|--------|
| インデント | スペース {4} 個（タブ不可） |
| 1行の最大文字数 | {120} 文字 |
| 文字列クォート | {ダブルクォート `"` を優先} |
| 末尾カンマ | 複数行の場合は末尾カンマを付ける |
| フォーマッター | {例: Ruff / Black} |

```python
# ✅ Good（複数行の末尾カンマ）
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

## 3. 型ヒント (Type Hints)

- すべての関数・メソッドの引数と戻り値に型ヒントを付ける。
- Python 3.10+ の `X | Y` 構文を使用する（`Union[X, Y]` は使わない）。
- `Optional[X]` の代わりに `X | None` を使用する。
- 型チェックには `mypy` または `pyright` を使用する（strict モード推奨）。

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

## 4. インポート順序

以下の順番で記述し、各グループは空行で区切る（`isort` / `Ruff` で自動整形）。

1. 標準ライブラリ
2. サードパーティライブラリ
3. ローカル（プロジェクト内）モジュール

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

ワイルドカードインポート（`from module import *`）は禁止。

---

## 5. Docstring 規約

- すべての公開クラス・関数・メソッドに Docstring を付ける。
- 形式は **{Google スタイル / NumPy スタイル}** を使用する。

```python
# ✅ Good（Google スタイル）
def fetch_user(user_id: int) -> UserProfile:
    """指定された ID のユーザー情報を取得する。

    Args:
        user_id: 対象ユーザーの識別子。

    Returns:
        対象ユーザーの UserProfile オブジェクト。

    Raises:
        UserNotFoundError: 指定された ID のユーザーが存在しない場合。
        NetworkError: ネットワーク接続に失敗した場合。
    """
    ...
```

---

## 6. エラーハンドリング

- 具体的な例外クラスを `except` で捕捉する（`except Exception` は原則禁止）。
- プロジェクト固有の例外は基底クラスを継承して定義する。
- `except` 節では必ずログを記録するかエラーを再送出する。

```python
# ✅ Good
class AppError(Exception):
    """アプリケーション基底例外クラス。"""

class UserNotFoundError(AppError):
    """ユーザーが見つからない場合の例外。"""

def get_user(user_id: int) -> UserProfile:
    try:
        return repository.find(user_id)
    except DatabaseConnectionError as e:
        logger.error("DB接続エラー: %s", e)
        raise
    except RecordNotFoundError as e:
        raise UserNotFoundError(f"ユーザー {user_id} が見つかりません") from e

# ❌ Bad
def get_user(user_id: int):
    try:
        return repository.find(user_id)
    except Exception:
        pass
```

---

## 7. クラス設計

- `dataclass` または `pydantic.BaseModel` をデータ保持クラスに活用する。
- `__init__` が単純なデータ格納のみの場合は `@dataclass` を優先する。

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

## 8. 非同期処理

- I/O バウンドな処理には `async/await` を使用する。
- `asyncio.gather` で並列処理を行う。
- 非同期コンテキストマネージャ（`async with`）を適切に使用する。

```python
# ✅ Good
async def fetch_all(user_ids: list[int]) -> list[UserProfile]:
    tasks = [fetch_user(uid) for uid in user_ids]
    return await asyncio.gather(*tasks)
```

---

## 9. テスト規約

<!-- プロジェクトで使用するテストフレームワークに応じて変更してください -->

- テストフレームワーク: **{pytest}**
- テストファイルは `tests/` ディレクトリに配置し、`test_*.py` の命名規則に従う。
- テスト関数名は `test_<対象>_<条件>_<期待結果>` の形式で記述する。
- フィクスチャは `conftest.py` に定義する。

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

## 10. プロジェクト固有のルール

<!-- プロジェクトに応じて追記してください -->

- {プロジェクト固有のルールをここに記載}
