# Data Pipeline Project Templates

Exact file contents for generating a data processing / ETL pipeline project.

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
dependencies = []

# ── uv ──
[tool.uv]
dev-dependencies = [
    "pytest>=8.0",
    "coverage>=7.0",
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

## src/{package_name}/pipeline.py

```python
"""Main pipeline entry point."""

from pathlib import Path

from {package_name}.config import settings


def run_pipeline(input_path: Path | None = None, output_path: Path | None = None) -> None:
    """Run the data pipeline.

    Args:
        input_path: Override input directory from settings.
        output_path: Override output directory from settings.
    """
    src = input_path or settings.input_dir
    dst = output_path or settings.output_dir

    print(f"Running pipeline: {src} → {dst}")
    # TODO: Implement your pipeline logic here.
```

## src/{package_name}/config.py

```python
from pathlib import Path

from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    data_dir: Path = Path("data")
    log_level: str = "INFO"

    model_config = {"env_file": ".env", "env_file_encoding": "utf-8"}

    @property
    def input_dir(self) -> Path:
        return self.data_dir / "input"

    @property
    def output_dir(self) -> Path:
        return self.data_dir / "output"


settings = Settings()
```

## tests/test_example.py

```python
from pathlib import Path

from {package_name}.pipeline import run_pipeline


def test_run_pipeline(tmp_path: Path):
    input_dir = tmp_path / "input"
    output_dir = tmp_path / "output"
    input_dir.mkdir()
    output_dir.mkdir()

    # Should not raise
    run_pipeline(input_path=input_dir, output_path=output_dir)
```

## data/input/.gitkeep

(empty file)

## data/output/.gitkeep

(empty file)

## .env.example

```bash
DATA_DIR=./data
LOG_LEVEL=INFO
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
        additional_dependencies: []

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

# Data (keep structure, ignore contents)
data/input/*
!data/input/.gitkeep
data/output/*
!data/output/.gitkeep

# OS
.DS_Store
Thumbs.db
```

## README.md

```markdown
# {package_name}

{description}

## Project Structure

```
data/
├── input/    # Place input data files here
└── output/   # Pipeline output will be written here
```

## Quick Start

```bash
# Install dependencies
uv sync

# Run the pipeline
uv run python -m {package_name}.pipeline

# Run tests
uv run pytest

# Run linter
uv run ruff check .

# Run type checker
uv run mypy src/
uv run ty check src/
```

## Configuration

Environment variables are loaded from `.env` file. See `.env.example` for available options.
```
