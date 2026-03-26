---
name: python
description: |
  Python scripting, PEP 8 kod stili, type hints, pytest, asyncio, CLI
  araclari ve veri isleme kurallari icin kullan.
---

# PYTHON KURALLARI

Bu kurallar Python scripting, otomasyon ve veri isleme icin gecerlidir.

---

## 1. KOD STILI (PEP 8)

### Naming Convention

| TIP | FORMAT | ORNEK |
|-----|--------|-------|
| Module | snake_case | data_processor.py |
| Class | PascalCase | DataProcessor |
| Function/Method | snake_case | process_data |
| Variable | snake_case | file_path |
| Constant | UPPER_SNAKE_CASE | MAX_RETRIES |
| Private | _leading | _internal_method |
| Name-mangling | __double | __private_attr |
| Type variable | PascalCase | T, TResult |

### Import Sirasi

```python
# 1. Standard library
import os
import sys
from pathlib import Path
from typing import Optional, List

# 2. Third-party
import numpy as np
import pandas as pd

# 3. Local
from myproject.utils import helpers
```

### Dosya Yapisi

```python
#!/usr/bin/env python3
"""Module docstring."""

import os
from typing import Optional

# Constants
DEFAULT_TIMEOUT = 30

# Classes
class MyClass:
    """Class docstring."""
    pass

# Functions
def my_function(param: str) -> bool:
    """Function docstring."""
    pass

# Main guard
if __name__ == "__main__":
    main()
```

---

## 2. TYPE HINTS

```python
from typing import Optional, List, Dict, Callable

def process_data(
    items: list[str],
    config: dict[str, int],
    callback: Optional[Callable[[str], None]] = None
) -> tuple[bool, str]:
    pass

# Python 3.10+ union syntax
def get_value(key: str) -> int | None:
    pass
```

### Dataclass

```python
from dataclasses import dataclass, field

@dataclass
class SensorConfig:
    name: str
    address: int
    channels: int = 4
    calibration: list[float] = field(default_factory=list)

    def is_valid(self) -> bool:
        return self.channels > 0 and self.address > 0
```

### Pydantic Model

```python
from pydantic import BaseModel, Field, field_validator

class DeviceConfig(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    ip_address: str
    port: int = Field(default=8080, ge=1, le=65535)
    timeout: float = 30.0

    @field_validator("ip_address")
    @classmethod
    def validate_ip(cls, v: str) -> str:
        parts = v.split(".")
        if len(parts) != 4:
            raise ValueError("Invalid IP address")
        return v
```

---

## 3. ASYNC/AWAIT (asyncio)

### Temel Pattern

```python
import asyncio

async def fetch_data(url: str) -> dict:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()

async def main():
    tasks = [fetch_data(url) for url in urls]
    results = await asyncio.gather(*tasks)
    return results

if __name__ == "__main__":
    asyncio.run(main())
```

### Async Context Manager

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def managed_connection(host: str, port: int):
    conn = await connect(host, port)
    try:
        yield conn
    finally:
        await conn.close()

# Kullanim
async with managed_connection("localhost", 8080) as conn:
    await conn.send(data)
```

### Async Generator

```python
async def read_chunks(path: str, chunk_size: int = 4096):
    async with aiofiles.open(path, "rb") as f:
        while chunk := await f.read(chunk_size):
            yield chunk
```

---

## 4. CLI TOOL GELISTIRME

### argparse

```python
import argparse

def create_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(description="Tool aciklamasi")
    parser.add_argument("input", help="Giris dosyasi")
    parser.add_argument("-o", "--output", default="out.txt", help="Cikis dosyasi")
    parser.add_argument("-v", "--verbose", action="store_true", help="Detayli cikti")
    return parser

def main():
    args = create_parser().parse_args()
    process(args.input, args.output, args.verbose)
```

### click/typer

```python
import typer

app = typer.Typer()

@app.command()
def process(
    input_file: str = typer.Argument(..., help="Giris dosyasi"),
    output: str = typer.Option("out.txt", help="Cikis dosyasi"),
    verbose: bool = typer.Option(False, help="Detayli cikti"),
):
    """Tool aciklamasi."""
    pass

if __name__ == "__main__":
    app()
```

---

## 5. ERROR HANDLING

```python
# YANLIS: Cok genel
try:
    process_data()
except:
    pass

# DOGRU: Spesifik
try:
    result = process_file(path)
except FileNotFoundError:
    logger.error(f"Not found: {path}")
    raise
except PermissionError as e:
    logger.error(f"Permission denied: {e}")
    return None
```

### Custom Exception

```python
class AppError(Exception):
    """Base application error."""
    pass

class ConfigError(AppError):
    """Configuration error."""
    def __init__(self, key: str, message: str):
        self.key = key
        super().__init__(f"Config error [{key}]: {message}")
```

### Context Manager

```python
# YANLIS
f = open("file.txt")
try:
    data = f.read()
finally:
    f.close()

# DOGRU
with open("file.txt") as f:
    data = f.read()
```

---

## 6. PATHLIB

```python
from pathlib import Path

# YANLIS
import os
path = os.path.join("dir", "file.txt")

# DOGRU
path = Path("dir") / "file.txt"
path.mkdir(parents=True, exist_ok=True)
content = path.read_text(encoding="utf-8")
path.write_text("content", encoding="utf-8")

# Glob
for py_file in Path("src").rglob("*.py"):
    print(py_file)
```

---

## 7. TESTING (pytest)

### Temel Test

```python
import pytest

def test_add():
    assert add(2, 3) == 5

def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        divide(1, 0)
```

### Fixture

```python
@pytest.fixture
def sample_config():
    return {"timeout": 30, "retries": 3}

@pytest.fixture
def temp_file(tmp_path):
    f = tmp_path / "test.txt"
    f.write_text("test content")
    return f

def test_load_config(sample_config):
    result = load_config(sample_config)
    assert result.timeout == 30
```

### Parametrize

```python
@pytest.mark.parametrize("input_val,expected", [
    (1, 1),
    (2, 4),
    (3, 9),
    (-1, 1),
])
def test_square(input_val, expected):
    assert square(input_val) == expected
```

### Mock

```python
from unittest.mock import Mock, patch

@patch("mymodule.requests.get")
def test_fetch_data(mock_get):
    mock_get.return_value.json.return_value = {"key": "value"}
    result = fetch_data("http://example.com")
    assert result["key"] == "value"
    mock_get.assert_called_once()
```

---

## 8. ANTI-PATTERNS

### Mutable Default

```python
# YANLIS
def add_item(item, items=[]):
    items.append(item)
    return items

# DOGRU
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

### Global Degisken

```python
# YANLIS
counter = 0
def increment():
    global counter
    counter += 1

# DOGRU
class Counter:
    def __init__(self):
        self.value = 0
    def increment(self):
        self.value += 1
```

### == None

```python
# YANLIS
if value == None:
    pass

# DOGRU
if value is None:
    pass
```

### range(len())

```python
# YANLIS
for i in range(len(items)):
    print(items[i])

# DOGRU
for item in items:
    print(item)

# Index gerekiyorsa
for i, item in enumerate(items):
    print(f"{i}: {item}")
```

### Bare except

```python
# YANLIS
try:
    risky()
except:
    pass

# DOGRU
try:
    risky()
except ValueError as e:
    logger.error("Value error: %s", e)
    raise
```

---

## 9. PYTHONIC YAPILAR

### List Comprehension

```python
squares = [x**2 for x in range(10)]
evens = [x for x in range(10) if x % 2 == 0]
word_lengths = {word: len(word) for word in words}
```

### Generator

```python
# Memory-efficient
sum_squares = sum(x**2 for x in range(1000000))

def read_large_file(path: Path):
    with open(path) as f:
        for line in f:
            yield line.strip()
```

### Walrus Operator

```python
if (n := len(items)) > 10:
    print(f"Too many: {n}")
```

### Unpacking

```python
a, b = 1, 2
a, b = b, a  # Swap
first, *rest = [1, 2, 3, 4, 5]
defaults = {"timeout": 30}
config = {**defaults, "timeout": 60}
```

---

## 10. LOGGING

```python
import logging

logger = logging.getLogger(__name__)

def process_data(data: list) -> list:
    logger.debug("Processing %s items", len(data))  # Lazy eval
    try:
        result = transform(data)
        logger.info("Success: %s items", len(result))
        return result
    except ValueError as e:
        logger.error("Failed: %s", e, exc_info=True)
        raise
```

---

## 11. VIRTUAL ENVIRONMENT

```
# venv (standart)
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
.venv\Scripts\activate     # Windows

# poetry
poetry init
poetry add requests
poetry install

# pip-tools
pip-compile requirements.in
pip-sync requirements.txt
```

---

## 12. PROJE YAPISI

```
my_project/
├── src/
│   └── my_project/
│       ├── __init__.py
│       ├── main.py
│       └── utils.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   └── test_main.py
├── requirements.txt
├── pyproject.toml
└── README.md
```
