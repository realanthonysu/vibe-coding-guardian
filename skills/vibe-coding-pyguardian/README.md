# vibe-coding-pyguardian

> Python-specific software engineering guardian for Vibe Coding — because "running ≠ done".

English | [中文](./README_cn.md)

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-Apache--2.0-green)
![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-brightgreen)
![Python](https://img.shields.io/badge/Python-3.10+-3776ab)

An [Agent Skills](https://agentskills.io/specification)-compatible skill that enforces Python-specific software engineering discipline in AI-assisted coding. Built on Karpathy's insight: *thinking can be outsourced, understanding cannot*.

This is the Python-specialized edition of [vibe-coding-guardian](../vibe-coding-guardian/SKILL.md), mapping the universal iron rules to Python's concrete tools and practices.

---

## Table of Contents

- [Why a Python-specific version?](#why-a-python-specific-version)
- [Python Tool Chain](#python-tool-chain)
- [Five Iron Rules](#five-iron-rules)
- [Dual Mode](#dual-mode)
- [Python-specific Risk Keywords](#python-specific-risk-keywords)
- [Progressive Disclosure](#progressive-disclosure)
- [Installation](#installation)
- [Usage](#usage)
- [File Structure](#file-structure)
- [Key Design Decisions](#key-design-decisions)
- [License](#license)

---

## Why a Python-specific version?

The generic vibe-coding-guardian works across languages, but Python has unique characteristics that benefit from specialized guidance:

- **Dynamic typing** — type errors hide until runtime; mypy is essential but not default
- **Batteries included, but also dangerous** — `pickle`, `eval`, `subprocess` are convenient but security-sensitive
- **Ecosystem fragmentation** — ruff vs flake8, pytest vs unittest, uv vs pip vs poetry
- **Unique anti-patterns** — mutable default arguments, bare excepts, circular imports, unclosed resources

This version maps each iron rule to Python's actual tools, so the agent runs `ruff check` instead of generic "lint", and knows that `pickle.loads()` is 🔴 high-risk.

---

## Python Tool Chain

| Responsibility | Tool | Command | Notes |
|----------------|------|---------|-------|
| Lint + Format | **ruff** | `ruff check .` / `ruff format .` | Replaces flake8 + isort + black |
| Type check (deep) | **mypy** | `mypy .` | Strict mode recommended |
| Type check (fast) | **ty** | `ty check .` | Astral's Rust-based checker, instant feedback |
| Testing | **pytest** | `pytest` | De facto standard |
| Coverage | **coverage** | `coverage run -m pytest` | Test completeness measurement |
| Security scan | **bandit** | `bandit -r src/` | Python-specific SAST |
| Dependency audit | **pip-audit** | `pip-audit` | Known vulnerability check |
| Package manager | **uv** | `uv sync` | Python package manager standard |
| Git hooks | **prek** | `prek run --all-files` | Rust-based pre-commit replacement, faster and dependency-free |

The skill auto-detects your project's actual tool choices and respects them.

---

## Five Iron Rules

| # | Rule | Python-specific focus |
|---|------|----------------------|
| 1 | **Understand before generating** | Risk keywords include `eval`, `pickle`, `subprocess`, `asyncio` |
| 2 | **Running ≠ Done** | Completion = `ruff check` + `mypy` + `ty` + `pytest` all pass |
| 3 | **Verification structure > quantity** | `pytest.mark.parametrize` for boundary matrices, not 100 trivial asserts |
| 4 | **Higher risk, deeper verification** | Python-specific 🔴 triggers: eval/exec, pickle, subprocess, YAML, shared mutable state |
| 5 | **Know when to escalate** | 5 computable signals with Python-specific grep patterns |

---

## Dual Mode

| | 🚀 Prototype (default) | 🏭 Production |
|---|:---:|:---:|
| Philosophy | Speed first — Jupyter-friendly | Safety first — PyPI-ready |
| 🟡 Verification | ruff + mypy + ty + related tests | + unit tests + integration tests + human review |
| 🔴 Verification | + bandit (input validation + auth only) | + bandit (full) + boundary tests + human confirmation |
| Signal 5 (security paths) | Warn, don't block | Block and escalate |
| Completion | ruff + mypy + ty + test suite (or manual confirmation) | Full iron rule 2 checklist + ruff format check |

Switch by saying **"prototype"** / **"production"** / **"上线"** / **"ship it"** / **"发布 PyPI"**.

---

## Python-specific Risk Keywords

These keywords automatically trigger 🔴 high-risk assessment:

| Keyword/Pattern | Risk |
|----------------|------|
| `eval()` / `exec()` | Arbitrary code execution |
| `pickle.loads()` / `pickle.load()` | Deserialization = arbitrary code |
| `subprocess.call(shell=True)` / `os.system()` | Command injection |
| `yaml.load()` (no `SafeLoader`) | YAML deserialization attack |
| `tempfile.mktemp()` | Race condition |
| `assert` for security checks | Skipped under `python -O` |
| `# type: ignore` | Type check bypass |
| `random` for security | Use `secrets` instead |
| `__import__()` with user input | Dynamic import attack |

---

## Progressive Disclosure

```
Tier 1: Metadata (~100 tokens)     — name + description, loaded at startup
Tier 2: SKILL.md (~3500 tokens)    — iron rules + decision flow + gotchas, loaded on activation
Tier 3: references/ (per file)     — deep guides, loaded only when relevant
```

| Reference | When to load | Content |
|-----------|-------------|---------|
| `references/architecture.md` | New module, adding dependency, package structure | Module boundaries, pyproject.toml, layered architecture, Protocol |
| `references/security.md` | 🔴 high-risk tasks | Input validation, auth, data protection, Python-specific vulnerabilities |
| `references/quality-gates.md` | Git hooks, test strategy | 5-stage verification, pytest patterns, coverage, prek config |
| `references/refactoring.md` | Code rot, tech debt | Refactoring patterns, Python code smells, safety nets |
| `references/examples.md` | Good vs bad patterns | 5 iron rules with Python ❌ Bad / ✅ Good examples |

---

## Installation

### Claude Code

**Project-level:**

```bash
mkdir -p .claude/skills
cp -r vibe-coding-pyguardian .claude/skills/
```

**User-level:**

```bash
mkdir -p ~/.claude/skills
cp -r vibe-coding-pyguardian ~/.claude/skills/
```

### Other Agents

This skill follows the [Agent Skills specification](https://agentskills.io/specification). Place the `vibe-coding-pyguardian/` folder in your agent's skill directory.

---

## Usage

No explicit activation needed — the skill auto-activates on any Python code generation, modification, or refactoring task.

```
You: Add a CSV export endpoint to our FastAPI app

AI: [auto-activates vibe-coding-pyguardian]

🟡 Medium risk (new feature + file I/O). Need to confirm:
1. Expected data volume — streaming needed for large datasets?
2. Encoding — UTF-8 or need BOM for Excel compatibility?
3. Filtering — export all records or with query params?

You: Up to 100k rows, UTF-8, with date range filter

AI: [generates streaming CSV + Pydantic query params + error handling]
[ruff check ✅] [mypy ✅] [pytest ✅]
Completion: ✅ positive path ✅ negative path ✅ no regression ✅ contract consistent
```

---

## File Structure

```
vibe-coding-pyguardian/
├── SKILL.md                        # Core skill (frontmatter + iron rules + decision flow)
├── README.md                       # English documentation
├── README_cn.md                    # Chinese documentation
└── references/
    ├── architecture.md             # Architecture guardian (on-demand)
    ├── security.md                 # Security guardrails (on-demand)
    ├── quality-gates.md            # Quality gates (on-demand)
    ├── refactoring.md              # Refactoring governance (on-demand)
    └── examples.md                 # Good vs Bad examples (on-demand)
```

---

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Python 3.10+ minimum | `match` statements, `X | Y` union types, `ParamSpec` |
| ruff over flake8+black | Single tool, 10-100x faster, actively maintained |
| mypy over pyright | Wider adoption, more mature, better ecosystem support |
| uv as recommended package manager | Speed, pip compatibility, active development |
| bandit for security | Python-specific SAST, maintained by PyCQA |
| Respect existing tool choices | Don't force ruff on a flake8 project |
| `pickle.loads()` → 🔴 high-risk | Deserialization is the #1 Python-specific attack vector |

---

## License

[Apache-2.0](../vibe-coding-guardian/LICENSE)
