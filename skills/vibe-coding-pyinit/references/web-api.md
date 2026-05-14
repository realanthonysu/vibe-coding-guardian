# Web API Project Templates

Exact file contents for generating a FastAPI-based web API project.

## pyproject.toml

```toml
[project]
name = "{package_name}"
version = "0.1.0"
description = "{description}"
readme = "README.md"
requires-python = ">=3.12"
authors = [
    { name = "{author_name}", email = "{author_email}" },
]
dependencies = [
    "fastapi>=0.115",
    "uvicorn[standard]>=0.34",
    "pydantic-settings>=2.0",
]

[project.optional-dependencies]
db = [
    "sqlalchemy>=2.0",
    "alembic>=1.14",
]

# ── uv ──
[tool.uv]
dev-dependencies = [
    "pytest>=8.0",
    "coverage>=7.0",
    "httpx>=0.28",
    "mypy>=1.15",
    "ty>=0.0",
    "ruff>=0.11",
]

# ── ruff ──
[tool.ruff]
target-version = "py312"
line-length = 120

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "N",   # pep8-naming
    "UP",  # pyupgrade
    "B",   # flake8-bugbear
    "SIM", # flake8-simplify
    "TCH", # flake8-type-checking
    "RUF", # ruff-specific
]

# ── mypy ──
[tool.mypy]
python_version = "3.12"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

# ── ty ──
[tool.ty]
python-version = "3.12"

# ── pytest + coverage ──
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short"

[tool.coverage.run]
source = ["src"]

[tool.coverage.report]
show_missing = true
fail_under = 80
```

## src/{package_name}/__init__.py

```python
"""{description}"""
```

## src/{package_name}/main.py

```python
from fastapi import FastAPI

from {package_name}.api import router

app = FastAPI(
    title="{package_name}",
    description="{description}",
    version="0.1.0",
)

app.include_router(router)


@app.get("/health")
def health() -> dict[str, str]:
    return {"status": "ok"}
```

## src/{package_name}/api/__init__.py

```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/")
def root() -> dict[str, str]:
    return {"message": "Welcome to {package_name}"}
```

## src/{package_name}/models/__init__.py

```python
# Pydantic models — add your request/response models here.
```

## src/{package_name}/core/__init__.py

```python
```

## src/{package_name}/core/config.py

```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    app_env: str = "development"
    debug: bool = False
    database_url: str = ""

    model_config = {"env_file": ".env", "env_file_encoding": "utf-8"}


settings = Settings()
```

## tests/test_example.py

```python
from fastapi.testclient import TestClient

from {package_name}.main import app

client = TestClient(app)


def test_health():
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json() == {"status": "ok"}


def test_root():
    response = client.get("/")
    assert response.status_code == 200
    assert "message" in response.json()
```

## .env.example

```bash
APP_ENV=development
DEBUG=true
DATABASE_URL=sqlite:///./data.db
```

## .pre-commit-config.yaml

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.11.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.15.0
    hooks:
      - id: mypy
        additional_dependencies:
          - fastapi
          - pydantic-settings

  - repo: https://github.com/astral-sh/ty-pre-commit
    rev: v0.0.1  # CHECK_LATEST: https://github.com/astral-sh/ty-pre-commit/releases
    hooks:
      - id: ty
```

> Note: Update hook versions to latest before generating.

## .gitignore

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so

# Distribution / packaging
dist/
build/
*.egg-info/
*.egg

# Virtual environments
.venv/
venv/
ENV/

# IDE
.idea/
.vscode/
*.swp
*.swo

# Testing
.pytest_cache/
htmlcov/
.coverage
.coverage.*
coverage.xml

# Type checking
.mypy_cache/
.ty/

# Ruff
.ruff_cache/

# Environment
.env
.env.local

# Database
*.db
*.sqlite

# OS
.DS_Store
Thumbs.db
```

## README.md

```markdown
# {package_name}

{description}

## Quick Start

```bash
# Install dependencies
uv sync

# Run the server
uv run uvicorn {package_name}.main:app --reload

# Run tests
uv run pytest

# Run linter
uv run ruff check .

# Run type checker
uv run mypy src/
uv run ty check src/
```

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | /health | Health check |
| GET | / | Welcome message |

## Configuration

Environment variables are loaded from `.env` file. See `.env.example` for available options.
```
