# vibe-coding-guardian

> Software engineering guardian for Vibe Coding — because "running ≠ done".

English | [中文](./README_cn.md)

![Version](https://img.shields.io/badge/version-1.9.1-blue)
![License](https://img.shields.io/badge/license-Apache--2.0-green)
![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-brightgreen)

An [Agent Skills](https://agentskills.io/specification)-compatible skill suite that enforces software engineering discipline in AI-assisted coding. Built on Karpathy's insight: *thinking can be outsourced, understanding cannot*.

## Skills in this Repository

| Skill | Description | Scope |
|-------|-------------|-------|
| **vibe-coding-guardian** | Core engineering discipline — Karpathy pre-constraints, 5 iron rules, dual mode, 8 computable signals | Language-agnostic |

`vibe-coding-guardian` defines *what* to verify across any language.

> Python-specific skills (`vibe-coding-pyguardian`, `vibe-coding-pyinit`) have moved to the independent project [python-hunter](https://github.com/realanthonysu/python-hunter).
> Testing skills (`testsmith`, `testweaver`) have moved to the independent project [vibe-testing](../vibe-testing/).

## Highlights (v1.9.1)

- **Karpathy pre-constraints** — Minimum code, surgical changes, explicit assumptions before any iron rule
- **8 computable signals** — adds orphan cleanup, assumption validation, and complexity bloat detection
- **Phantom success guard** — "running" is re-checked against "solves the actual problem"
- **Dual mode** — 🚀 Prototype for speed, 🏭 Production for safety
- **Zero dependencies** — works with just git + your project's test/lint commands
- **Auto-remind** — nudges you from Prototype to Production before commit/deploy

---

## Table of Contents

- [Why This Skill?](#why-this-skill)
- [Skills in this Repository](#skills-in-this-repository)
- [Five Iron Rules](#five-iron-rules)
- [Dual Mode](#dual-mode)
- [Progressive Disclosure](#progressive-disclosure)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [File Structure](#file-structure)
- [Key Design Decisions](#key-design-decisions)
- [Compatibility](#compatibility)
- [Evolution History](#evolution-history)
- [Contributing](#contributing)
- [License](#license)

---

## Why This Skill?

Vibe Coding boosts local velocity — code generates fast, tests pass green, terminals keep printing success. But local velocity ≠ system trustworthiness. The real risk isn't that code is too slow; it's that **"done" starts to drift**.

This skill makes completion conditions, verification evidence, and escalation paths **computable** — so AI-assisted coding doesn't quietly accumulate entropy.

## Five Iron Rules

| # | Rule | One-liner |
|---|------|-----------|
| 1 | **Understand before generating** | If you can't state the intent in one sentence, don't write code yet |
| 2 | **Running ≠ Done** | Done = behavior verified + boundaries covered + no regression |
| 3 | **Verification structure > quantity** | 10 tests covering critical paths > 100 tests covering trivial paths |
| 4 | **Higher risk, deeper verification** | 🟢 low → quick check · 🟡 medium → standard verification · 🔴 high → deep verification + human confirmation |
| 5 | **Know when to escalate** | Escalation isn't failure — it's engineering maturity |

All five rules sit on top of three **Karpathy pre-constraints** that guard against AI-specific failure modes:

| Constraint | Guards against | One-liner |
|------------|----------------|-----------|
| **Simplicity First** | Over-engineering | Code is liability — minimum code that solves the problem |
| **Surgical Changes** | Orthogonal side effects | Touch only what you must; every changed line traces to the request |
| **Surface Assumptions** | Unchecked assumptions | State assumptions explicitly; push back when warranted |

## Dual Mode

| | 🚀 Prototype (default) | 🏭 Production |
|---|:---:|:---:|
| Philosophy | Speed first — embrace exponentials | Safety first — ship with confidence |
| 🟡 Verification | lint + type check + unit tests | + integration tests + human review |
| 🔴 Verification | + security.md (input validation + auth only) | + security.md (full) + boundary tests + human confirmation |
| Signal 5 (security paths) | Warn, don't block | Block and escalate |
| Completion | lint + type check + test suite (or manual confirmation) | Full iron rule 2 checklist |

Switch by saying **"prototype"** / **"production"** / **"上线"** / **"ship it"**. Auto-remind when you say "commit" / "push" / "deploy" / "done".

## Progressive Disclosure

This skill is designed for token efficiency with three loading tiers:

```
Tier 1: Metadata (~100 tokens)     — name + description, loaded at startup
Tier 2: SKILL.md (~3000 tokens)    — iron rules + decision flow + gotchas, loaded on activation
Tier 3: references/ (per file)     — deep guides, loaded only when relevant
```

| Reference | When to load | Content |
|-----------|-------------|---------|
| `references/architecture.md` | New module, adding dependency, changing system boundaries | Module boundaries, dependency management, API contracts, layered architecture |
| `references/security.md` | 🔴 high-risk tasks (🏭 full / 🚀 input validation + auth only) | Input validation, auth, data protection, vulnerability prevention |
| `references/quality-gates.md` | Pre-commit verification, test strategy, contract checks | 5-stage verification, test layering, evidence states, gate escalation |
| `references/refactoring.md` | Code rot, tech debt, progressive refactoring | Refactoring patterns, debt management, code smells, safety nets |
| `references/examples.md` | Need to understand good vs bad patterns | 5 iron rules with ❌ Bad / ✅ Good code examples |

## Prerequisites

- **Claude Code** (latest version), **Trae**, or any **Agent Skills-compatible** agent
- **Git** (optional — the skill works without git history, with graceful degradation)
- A project with **lint** and **type check** commands configured (recommended)

## Installation

### Skill CLI (Recommended)

The quickest way to install:

```bash
npx skills add realanthonysu/vibe-coding-guardian
```

### Claude Code

**Project-level** (follows the project):

```bash
mkdir -p .claude/skills
cp -r vibe-coding-guardian .claude/skills/
```

**User-level** (available in all projects):

```bash
mkdir -p ~/.claude/skills
cp -r vibe-coding-guardian ~/.claude/skills/
```

### Trae

1. Zip the `vibe-coding-guardian/` folder
2. Open Trae → **Settings** → **Rules & Skills** → **Import**
3. Upload the zip file
4. Confirm and refresh the skill list

Or create via conversation in **SOLO mode**:

> "Create a vibe-coding-guardian skill for me"

### Other Agents

This skill follows the [Agent Skills specification](https://agentskills.io/specification). Place the `vibe-coding-guardian/` folder in your agent's skill directory (e.g., `.agents/skills/`).

## Usage

No explicit activation needed — the skill auto-activates on any code generation, modification, or refactoring task.

```
You: Write me a user registration API

AI: [auto-activates vibe-coding-guardian]

Need to confirm a few things before proceeding:
1. What fields are required for registration?
2. Any password strength requirements?
3. Is email verification needed?
4. Expected user volume?

You: Email + password, min 8 chars with special characters, email verification required

AI: [🟡 medium risk: new feature + data write]
[generates code + input validation + duplicate detection + error handling]
[completion checklist: ✅ positive path ✅ negative path ✅ no regression ✅ contract consistent]
```

## File Structure

```
vibe-coding-guardian/               # Language-agnostic core
├── SKILL.md                        # Core skill (frontmatter + iron rules + decision flow)
├── README.md                       # English documentation
├── README_cn.md                    # Chinese documentation (中文文档)
├── CHANGELOG.md                    # Version history
├── LICENSE                         # Apache-2.0
└── references/
    ├── architecture.md             # Architecture guardian (on-demand)
    ├── security.md                 # Security guardrails (on-demand)
    ├── quality-gates.md            # Quality gates (on-demand)
    ├── refactoring.md              # Refactoring governance (on-demand)
    └── examples.md                 # Good vs Bad examples (on-demand)
```

> Python-specific skills (`vibe-coding-pyguardian`, `vibe-coding-pyinit`) are now in [python-hunter](https://github.com/realanthonysu/python-hunter).

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Default to 🚀 Prototype | Respects Vibe Coding's speed-first philosophy |
| 8 computable escalation signals | Replaces "gut feeling" with detectable patterns |
| `git diff` is optional, upward-only | Works without git history; can only escalate, never downgrade risk |
| security.md loaded partially in 🚀 | Even prototypes touching auth need input validation |
| Auto-remind on commit/push/deploy | Catches the "prototype → production" drift before it ships |

## Compatibility

| Agent | Status | Notes |
|-------|--------|-------|
| Claude Code | ✅ Tested | Project-level and user-level installation |
| Trae | ✅ Tested | Import via zip or SOLO mode |
| Other Agent Skills-compatible agents | ✅ Compatible | Follow the Agent Skills specification |

## Evolution History

### vibe-coding-guardian (v1.9)

| Version | Key Change |
|---------|-----------|
| v1.0.0 | Initial design — 5 iron rules + progressive disclosure |
| v1.1.0 | CSO description + "When to Activate" activation rules + tool mapping table |
| v1.2.0 | Gate Functions for all 5 rules + 6-step decision flow + 5 computable escalation signals |
| v1.3.0 | Remove external dependencies (superpowers, code-review-graph) — pure git + grep |
| v1.4.0 | Human review — risk-tiered understanding gate, git diff timing fix |
| v1.5.0 | Dual mode (🚀 Prototype / 🏭 Production) |
| v1.6.0 | Internal consistency — unified keywords, upward-only git diff, dual-mode tool table |
| v1.7.0 | Second AI review — security.md partial loading, test/no-test fallback, signal 5 warn-not-block, auto-upgrade reminders |
| v1.8.0 | Iron rule 4 dual-mode table, known-risk documentation, "done/finished" trigger words |
| v1.9.0 | Karpathy pre-constraints (Simplicity / Surgical / Assumptions), 8 signals (+3), phantom success guard, expanded anti-patterns |
| v1.9.1 | Constraint-to-rule mapping, execution timing, signal 6-8 examples, internal consistency fixes |

For detailed changes, see [CHANGELOG.md](./CHANGELOG.md).


## License

[Apache-2.0](./LICENSE)
