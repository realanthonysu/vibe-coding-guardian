---
name: vibe-coding-pyinit
description: >
  Use when initializing a new Python project or adding best-practice tooling to an existing one.
  Scaffolds a standardized local development environment with uv, ruff, mypy, ty, pytest, coverage, and prek.
  Triggers on: "初始化 Python 项目", "create a Python project", "scaffold Python", "新建 Python 项目",
  "set up Python dev environment", "给项目加上工程规范", "初始化 FastAPI 项目", "new FastAPI project",
  "Python 项目脚手架", "pyinit", or any request to set up a Python project with best practices.
version: 0.1.0
---

# vibe-coding-pyinit

Scaffold a Python project with a standardized local development environment. The goal is to eliminate decision fatigue — one skill, one opinionated toolchain, zero configuration debates.

## Toolchain (fixed, no alternatives)

| Role | Tool | Why |
|------|------|-----|
| Package manager | **uv** | Fast, Rust-based, becoming the Python standard |
| Linter + Formatter | **ruff** | Replaced flake8, isort, black — one tool for all |
| Type checker (deep) | **mypy** | Industry standard, catches subtle type bugs |
| Type checker (fast) | **ty** | Astral's new Rust-based checker, instant feedback |
| Testing | **pytest** | De facto standard |
| Coverage | **coverage** | Measures test completeness |
| Git hooks | **prek** | Rust-based pre-commit replacement, fast and dependency-free |

## Decision Flow

### Step 1: Detect current state

Check if `pyproject.toml` exists in the current directory.

**If exists → Existing project flow (Step 2a)**
**If not exists → New project flow (Step 2b)**

### Step 2a: Existing project — incremental supplement

Scan for the presence of these configuration items:

1. `[tool.uv]` in pyproject.toml
2. Ruff configuration (`[tool.ruff]` in pyproject.toml or `ruff.toml`)
3. Mypy configuration (`[tool.mypy]` in pyproject.toml or `mypy.ini`)
4. Ty configuration (`[tool.ty]` in pyproject.toml or `ty.toml`)
5. Pytest configuration (`[tool.pytest.ini_options]` in pyproject.toml)
6. Coverage configuration (`[tool.coverage.run]` in pyproject.toml)
7. Prek configuration (`.pre-commit-config.yaml`)
8. `.gitignore`

Present a gap analysis to the user:

```
已有 ✅  pyproject.toml (uv)
缺失 ❌  ruff configuration
已有 ✅  pytest configuration
缺失 ❌  prek configuration
缺失 ❌  mypy configuration
缺失 ❌  ty configuration
缺失 ❌  .gitignore

将生成: ruff.toml, .pre-commit-config.yaml, mypy.ini, ty.toml, .gitignore
```

Wait for user confirmation, then generate only the missing items. Do NOT overwrite existing configurations.

### Step 2b: New project — collect information

**2b.1 Check uv availability**

Run `uv --version`. If uv is not found:

```
未检测到 uv (Python 包管理器)。是否帮你安装？
安装方式: curl -LsSf https://astral.sh/uv/install.sh | sh (Linux/macOS) 或 powershell -c "irm https://astral.sh/uv/install.ps1 | iex" (Windows)
```

Wait for user confirmation before installing.

**2b.2 Gather project info**

If the user already provided a project description in their trigger message, infer from it. Otherwise, ask:

1. **Project description**: "请用一句话描述你要建的项目" (or English equivalent)
2. **Infer project type** from the description using keyword matching:

| Keywords | Inferred type |
|----------|---------------|
| FastAPI, Django, Flask, API, REST, 后端, backend, web service, 服务 | web-api |
| library, 库, package, SDK, publish, PyPI, 发布 | library |
| data, ETL, pipeline, 数据处理, 爬虫, scraper, 数据管道 | data-pipeline |

If no keywords match, present the three types and ask the user to choose:

```
请确认项目类型:
  1. library    — 发布到 PyPI 的 Python 包
  2. web-api    — Web API 服务 (FastAPI)
  3. data-pipeline — 数据处理/ETL 管道
```

3. **Package name**: Infer from the current directory name (convert `-` to `_`, lowercase, strip invalid chars). Present to user for confirmation.
4. **Project description text**: One-line description for pyproject.toml `[project]` field.

**2b.3 Confirm and generate**

Present the complete plan:

```
Project type:    web-api
Package name:    my_api
Description:     A FastAPI backend service
Toolchain:
  - uv           (package manager)
  - ruff         (lint + format)
  - mypy + ty    (type checking)
  - pytest + coverage  (testing)
  - prek         (git hooks)

将生成以下文件:
  pyproject.toml
  .pre-commit-config.yaml
  .gitignore
  README.md
  src/my_api/__init__.py
  src/my_api/main.py
  src/my_api/api/__init__.py
  src/my_api/models/__init__.py
  src/my_api/core/__init__.py
  src/my_api/core/config.py
  .env.example
  tests/test_example.py
```

Wait for user confirmation, then generate all files.

## Project Types

### library

A publishable Python package for PyPI.

**Directory structure:**
```
{project_root}/
├── pyproject.toml
├── .pre-commit-config.yaml
├── .gitignore
├── README.md
├── LICENSE
├── src/
│   └── {package_name}/
│       └── __init__.py
└── tests/
    └── test_example.py
```

**pyproject.toml specifics:**
- `build-system` uses `hatchling`
- Full `[project]` metadata: name, version, description, authors, license, classifiers, requires-python
- Version: start at `0.1.0`
- License: default to `Apache-2.0` (match this project's convention)

**LICENSE file:** Generate Apache-2.0 license text.

### web-api

A FastAPI-based web API service.

**Directory structure:**
```
{project_root}/
├── pyproject.toml
├── .pre-commit-config.yaml
├── .gitignore
├── README.md
├── .env.example
├── src/
│   └── {package_name}/
│       ├── __init__.py
│       ├── main.py
│       ├── api/
│       │   └── __init__.py
│       ├── models/
│       │   └── __init__.py
│       └── core/
│           ├── __init__.py
│           └── config.py
└── tests/
    └── test_example.py
```

**pyproject.toml specifics:**
- Dependencies: `fastapi`, `uvicorn[standard]`
- Dev dependencies group: `httpx` (for testing FastAPI)
- No build-system needed (not published)

**main.py:** Minimal FastAPI app with health check endpoint.

**core/config.py:** Pydantic-settings based configuration loading from environment variables.

**.env.example:** Template with common environment variables (`APP_ENV`, `DEBUG`, `DATABASE_URL`).

### data-pipeline

A data processing / ETL pipeline.

**Directory structure:**
```
{project_root}/
├── pyproject.toml
├── .pre-commit-config.yaml
├── .gitignore
├── README.md
├── .env.example
├── src/
│   └── {package_name}/
│       ├── __init__.py
│       ├── pipeline.py
│       └── config.py
├── data/
│   ├── input/
│   │   └── .gitkeep
│   └── output/
│       └── .gitkeep
└── tests/
    └── test_example.py
```

**pyproject.toml specifics:**
- No build-system needed
- Minimal dependencies (user adds what they need)

**.env.example:** Template with `DATA_DIR`, `LOG_LEVEL`.

## Generated Files — Templates

Read the appropriate reference file based on the confirmed project type:

- `references/library.md` — Full file contents for library projects
- `references/web-api.md` — Full file contents for web-api projects
- `references/data-pipeline.md` — Full file contents for data-pipeline projects

Each reference file contains the exact content to write for every generated file, including pyproject.toml with all tool configurations pre-filled.

## Common Configurations

These configurations are shared across all project types and are embedded in the pyproject.toml template in each reference file.

### ruff

```toml
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
```

### mypy

```toml
[tool.mypy]
python_version = "3.12"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
```

### ty

```toml
[tool.ty]
python-version = "3.12"
```

### pytest + coverage

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short"

[tool.coverage.run]
source = ["src"]

[tool.coverage.report]
show_missing = true
fail_under = 80
```

### prek (.pre-commit-config.yaml)

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

> Note: Pre-commit hook versions should be updated to latest at generation time. Check the latest versions of ruff-pre-commit, mirrors-mypy, and ty-pre-commit before generating.

### .gitignore

Standard Python .gitignore covering: `__pycache__/`, `*.pyc`, `.venv/`, `dist/`, `build/`, `*.egg-info/`, `.env`, `.mypy_cache/`, `.ty/`, `.ruff_cache/`, `htmlcov/`, `.coverage`, `coverage.xml`.

## After Generation

1. Run `uv sync` to create the virtual environment and install dependencies
2. Install prek (if not already installed): `cargo install prek` or `brew install prek`
3. Run `prek install` to set up git hooks
4. Run `uv run pytest` to verify the test setup works
5. Run `uv run ruff check .` to verify linting works

Present these steps to the user as a "next steps" checklist after generation is complete.
