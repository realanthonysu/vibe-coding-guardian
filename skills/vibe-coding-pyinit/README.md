# vibe-coding-pyinit

> Scaffold a Python project with a standardized local development environment — zero configuration debates.

[English](./README.md) | [中文](#简介)

![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-brightgreen)
![Python](https://img.shields.io/badge/Python-3.12+-blue)
![License](https://img.shields.io/badge/license-Apache--2.0-green)

An [Agent Skills](https://agentskills.io/specification)-compatible skill that scaffolds Python projects with an opinionated, standardized toolchain. Eliminates decision fatigue — one skill, one toolchain, one way to start.

---

## Table of Contents

- [Why This Skill?](#why-this-skill)
- [Toolchain](#toolchain)
- [Supported Project Types](#supported-project-types)
- [How It Works](#how-it-works)
- [Installation](#installation)
- [Usage](#usage)
- [File Structure](#file-structure)
- [Related Skills](#related-skills)
- [License](#license)

---

## Why This Skill?

Starting a Python project involves too many decisions: pip vs poetry vs uv vs pdm, flake8 vs ruff, black vs ruff format, mypy vs pyright, pre-commit vs lefthook... The fragmentation creates decision fatigue and inconsistent project setups.

This skill makes one opinionated choice for each decision point and generates a complete, working development environment. The goal: `uv sync && uv run pytest` should just work after scaffolding.

## Toolchain

| Role | Tool | Why |
|------|------|-----|
| Package manager | **uv** | Fast, Rust-based, becoming the Python standard |
| Linter + Formatter | **ruff** | Replaced flake8, isort, black — one tool for all |
| Type checker (deep) | **mypy** | Industry standard, catches subtle type bugs |
| Type checker (fast) | **ty** | Astral's new Rust-based checker, instant feedback |
| Testing | **pytest** | De facto standard |
| Coverage | **coverage** | Measures test completeness |
| Git hooks | **prek** | Rust-based pre-commit replacement, fast and dependency-free |

These choices are fixed — the skill doesn't ask you to choose. If you need different tools, use a different skill or configure manually after scaffolding.

## Supported Project Types

| Type | Description | Key Features |
|------|-------------|--------------|
| **library** | Publishable Python package for PyPI | src layout, hatchling build backend, LICENSE |
| **web-api** | FastAPI-based web API service | FastAPI + uvicorn, pydantic-settings, .env |
| **data-pipeline** | Data processing / ETL pipeline | Pipeline structure, data/ directories |

## How It Works

```
Trigger skill
    │
    ▼
Detect directory state
    │
    ├─ pyproject.toml exists → Incremental supplement (only add missing configs)
    │
    └─ No pyproject.toml → New project flow
            │
            ▼
        Collect info (description → infer type → confirm)
            │
            ▼
        Generate skeleton
            │
            ▼
        Next steps (uv sync, prek install, pytest)
```

### New Project

1. Describe your project in one sentence
2. The skill infers the project type from keywords (FastAPI → web-api, library → library, etc.)
3. Confirm the type, package name, and description
4. All files are generated at once

### Existing Project

1. The skill scans for missing tool configurations
2. Presents a gap analysis (what's configured vs what's missing)
3. Only generates the missing items — never overwrites existing configs

## Installation

### Skill CLI (Recommended)

```bash
npx skills add realanthonysu/vibe-coding-pyinit
```

### Claude Code

**Project-level:**

```bash
mkdir -p .claude/skills
cp -r vibe-coding-pyinit .claude/skills/
```

**User-level:**

```bash
mkdir -p ~/.claude/skills
cp -r vibe-coding-pyinit ~/.claude/skills/
```

## Usage

Trigger with a slash command or natural language:

```
/vibe-coding-pyinit

or

"帮我初始化一个 FastAPI 后端项目"
"Create a new Python library for data validation"
"给这个项目加上工程规范配置"
```

## File Structure

```
vibe-coding-pyinit/
├── SKILL.md                    # Core skill (decision flow + templates)
├── README.md                   # This file
└── references/
    ├── library.md              # Library project templates
    ├── web-api.md              # FastAPI project templates
    └── data-pipeline.md        # Data pipeline project templates
```

## Related Skills

| Skill | Relationship |
|-------|-------------|
| [vibe-coding-guardian](../vibe-coding-guardian/SKILL.md) | Core engineering discipline — defines *what* to verify |
| [vibe-coding-pyguardian](../vibe-coding-pyguardian/SKILL.md) | Python edition — defines *how* to verify in Python |

`pyinit` builds the house. `pyguardian` makes sure you don't wreck it after moving in.

## License

[Apache-2.0](../vibe-coding-guardian/LICENSE)

---

## 简介

`vibe-coding-pyinit` 是一个 Python 项目脚手架技能，用固定的工具链（uv + ruff + mypy + ty + pytest + coverage + prek）标准化本地开发环境，消除决策疲劳。

支持三种项目类型：library（PyPI 包）、web-api（FastAPI 服务）、data-pipeline（数据管道）。

触发方式：`/vibe-coding-pyinit` 或自然语言描述（如"初始化一个 FastAPI 项目"）。
