---
name: vibe-coding-pyguardian
description: >
  Use when: (1) starting any non-trivial Python coding task to assess risk level
  before implementation; (2) about to claim Python work is complete — verify with
  ruff + mypy + pytest evidence; (3) touching security-sensitive Python code (auth,
  payments, encryption, pickle, subprocess, eval); (4) diff impact radius unclear
  and blast analysis needed; (5) deciding whether a Python change requires human
  review. Also triggers on: risk assessment, blast radius, escalation, "should I
  proceed", 风险评估, 升级判断, 完成条件, 安全审查, entropy drift, green-light
  illusion, Python-specific: type hints, virtualenv, pyproject.toml, async safety.
license: Apache-2.0
metadata:
  author: vibe-coding-guardian
  version: "1.0"
  paradigm: vibe-coding-pyguardian
  core-thesis: "thinking can be outsourced, understanding cannot — Karpathy"
  language: python
  toolchain:
    linter: ruff
    type_checker: mypy + ty
    formatter: ruff format
    test_runner: pytest
    coverage: coverage
    security_scanner: bandit
    dependency_audit: pip-audit
    package_manager: uv
    pre_commit: prek
---

# Vibe Coding Python Guardian

> 思考可外包，理解不可外包。 — Karpathy

Vibe Coding 提升的是局部推进速度，不会自动提升系统整体可信度。
本技能的使命：**让 Python 项目的完成条件、验证证据和审查升级变得可计算。**

本技能是 [vibe-coding-guardian](../vibe-coding-guardian/SKILL.md) 的 Python 专用版本，将通用铁律映射到 Python 生态的具体工具和实践。

---

## Python 工具链约定

| 职责 | 工具 | 命令 | 备注 |
|------|------|------|------|
| Lint + Format | **ruff** | `ruff check .` / `ruff format .` | 替代 flake8 + isort + black，一个工具搞定 |
| 类型检查（深度） | **mypy** | `mypy .` | 严格模式推荐 `mypy --strict` |
| 类型检查（快速） | **ty** | `ty check .` | Astral 出品，Rust 实现，即时反馈 |
| 测试 | **pytest** | `pytest` | 配合 coverage 做覆盖率 |
| 覆盖率 | **coverage** | `coverage run -m pytest` / `coverage report` | 测试完备性度量 |
| 安全扫描 | **bandit** | `bandit -r src/` | Python 专用 SAST |
| 依赖审计 | **pip-audit** | `pip-audit` | 检查已知漏洞依赖 |
| 包管理 | **uv** | `uv sync` / `uv pip install` | Python 包管理标准 |
| Git hooks | **prek** | `prek run --all-files` | Rust 实现的 pre-commit 替代，更快更轻 |

### 工具链探测

执行验证前，先检测项目实际使用的工具：

```
项目根目录存在 pyproject.toml?
  ├─ 存在 → 读取 [tool.ruff] / [tool.mypy] / [tool.pytest] 配置
  │         检查是否有 uv.lock / poetry.lock / pdm.lock → 确定包管理器
  └─ 不存在 → 检查 setup.py / setup.cfg / requirements.txt
              使用默认工具配置

有 Makefile 或 justfile?
  └─ 存在 → 优先使用其中定义的命令（如 make lint / make test）

有 tox.ini / noxfile.py?
  └─ 存在 → 使用 tox/nox 作为测试入口
```

**原则：尊重项目已有的工具选择，不强推工具。** 如果项目用 flake8 不用 ruff，就用 flake8。如果项目用 pytest 不用 unittest，就用 pytest。

> 对于新建项目，推荐先使用 [vibe-coding-pyinit](../vibe-coding-pyinit/SKILL.md) 标准化工具链（uv + ruff + mypy + ty + pytest + coverage + prek），然后由本技能守护后续开发过程。

---

## 运行模式

### 🚀 Prototype 模式（默认）

适用于：快速原型、MVP、概念验证、探索性开发、Jupyter 实验

- 🟢 低风险 → 一句话意图，直接生成
- 🟡 中风险 → WHAT + CONSTRAINTS，**不触发安全审查**
- 🔴 高风险 → 完整理解模板 + 用户确认，**加载 security.md 但只执行"输入验证"和"认证授权"章节**
- 升级信号检测：信号 5 改为警告（不阻断）
- 完成条件：`ruff check` + `mypy` + 测试套件（或人工确认核心行为）

### 🏭 Production 模式

适用于：正式产品、上线前审查、安全敏感项目、发布到 PyPI

- 🟢 低风险 → 一句话意图，直接生成
- 🟡 中风险 → WHAT + CONSTRAINTS + 人工审查
- 🔴 高风险 → 完整理解模板 + 用户确认 + **加载 security.md** + 安全审查 + 边界测试
- 升级信号检测：5 项信号全部执行
- 完成条件：铁律二完整自检（正向 + 负向 + 回归 + 契约）

### 模式切换

- 用户说 "prototype" / "原型" / "MVP" / "just prototyping" → 切换到 🚀 Prototype
- 用户说 "production" / "生产" / "上线" / "ship it" / "发布 PyPI" → 切换到 🏭 Production
- 默认为 🚀 Prototype（尊重 Vibe Coding 的速度优先）
- **切换到 Production 时，建议对已有代码做一次回溯审查**

### 自动升级提醒

当用户在 🚀 Prototype 模式下执行以下操作时，主动提醒是否切换到 🏭 Production：

- "提交" / "commit" / "push" / "merge" / "PR" / "pull request"
- "部署" / "deploy" / "release" / "发布" / "publish to PyPI"
- "完成" / "done" / "finished" / "搞定了"

提醒话术：`"你正在 Prototype 模式下准备提交/部署。建议切换到 Production 模式进行完整审查。是否切换？"`

### 两种模式对比

| 环节 | 🚀 Prototype | 🏭 Production |
|------|:-----------:|:------------:|
| 🟢 理解确认 | 一句话意图 | 一句话意图 |
| 🟡 理解确认 | WHAT + CONSTRAINTS | WHAT + CONSTRAINTS + 人工审查 |
| 🔴 理解确认 | 完整模板 + 确认 | 完整模板 + 确认 + security.md |
| 🟡 验证深度 | ruff + mypy + ty + 关联测试 | + 单元测试 + 集成测试 |
| 🔴 验证深度 | + bandit（仅输入验证+认证授权） | + bandit（全部）+ 边界测试 + 人工确认 |
| 升级信号 5（安全路径） | 警告但不阻断 | 阻断升级 |
| 完成条件 | ruff + mypy + 测试套件（或人工确认核心行为） | 铁律二完整自检 |

---

## When to Activate

### MUST activate (non-negotiable)

- Task involves **auth, payments, encryption, data migration** → 铁律四 + 铁律五
- Task uses **eval/exec, pickle, subprocess, os.system** → 铁律四（🔴 Python 特有高风险）
- About to claim "done", "complete", "fixed", "working" → 铁律二
- User asks "is this safe to ship?" / "can I merge?" → 铁律二 + 铁律四
- Diff touches **3+ files across 2+ modules** → 铁律四
- New external dependency being introduced → [architecture.md](references/architecture.md)
- Modifying **async/await** code or introducing concurrency → 铁律四

### SHOULD activate (recommended)

- Any new feature implementation → 铁律一（理解优先）
- Refactoring existing code → [refactoring.md](references/refactoring.md)
- Test count growing but confidence not increasing → 铁律三
- Adding or modifying **type hints** → 铁律三

### SKIP (do not activate)

- Pure read/exploration tasks (no code changes)
- Simple typo/formatting fixes
- Configuration changes (non-security)
- User explicitly says "skip guardian" / "跳过守护" / "just do it"
- Jupyter notebook exploration (非生产代码)

---

## 五条铁律

### 铁律一：先理解，再生成

生成代码之前，必须先明确三件事：

- **做什么** — 这段代码要解决什么问题？
- **为什么** — 为什么是这个方案而不是其他？
- **边界在哪** — 输入范围、错误情况、性能约束是什么？

无法用一句话说清意图 → 理解不够 → 不要急于写代码。

#### Gate Function：理解确认（风险分级）

```
收到编码任务
  │
  ├─ ❶ 快速风险预判（基于任务描述，非 git diff）：
  │     触及 auth/payment/crypto/session/token/data-migration → 🔴 高风险
  │     触及 eval/exec/pickle/subprocess/os.system/shelve     → 🔴 高风险
  │     触及 asyncio 并发/多线程/共享可变状态                  → 🔴 高风险
  │     新功能 / API 变更 / 跨模块改动 / 新依赖               → 🟡 中风险
  │     UI 文案 / 样式微调 / 非破坏性新增 / 类型注解补充       → 🟢 低风险
  │
  ├─ ❷ 按风险等级执行理解确认：
  │
  │     🟢 低风险 → 一句话说清意图即可：
  │       INTENT: [一句话描述要做什么]
  │       → 直接进入生成阶段
  │
  │     🟡 中风险 → 输出 WHAT + CONSTRAINTS：
  │       WHAT:       [一句话描述要做什么]
  │       CONSTRAINTS: [输入范围 / 错误处理 / 性能要求]
  │       → 呈现给用户，确认后继续
  │
  │     🔴 高风险 → 完整理解模板 + 用户确认：
  │       WHAT:       [一句话描述要做什么]
  │       WHY:        [为什么是这个方案]
  │       CONSTRAINTS: [输入范围 / 错误处理 / 性能要求 / 兼容性 / 安全考量]
  │       → 呈现给用户，等待确认
  │       → 用户修正 → 更新模板 → 重新确认
  │       → 用户未确认 → 不动手写代码
  │
  └─ ❸ 确认通过 → 进入生成阶段
```

### 铁律二：能跑 ≠ 完成

完成 = 行为已验证 + 边界已覆盖 + 无回归

提交前自检：
- [ ] 正向路径：核心功能按预期工作？
- [ ] 负向路径：错误输入、异常状态已处理？
- [ ] 回归风险：改动不破坏已有功能？
- [ ] 契约一致：接口契约与调用方期望一致？

#### Python 完成条件 Gate Function

```
🚀 Prototype:
  → 运行 ruff check . → 0 errors
  → 运行 mypy . → 0 errors（或项目配置的严格度）
  → 运行 ty check . → 0 errors（如项目已配置 ty）
  → 有测试套件：运行 pytest → exit code = 0
    无测试套件：人工确认核心行为（一句话描述"我验证了 XXX"）
  → 全部通过 → 进入升级信号检测

🏭 Production:
  → 步骤 A：运行 pytest → exit code = 0
  → 步骤 B：运行 ruff check . → 0 errors
  → 步骤 C：运行 mypy . → 0 errors
  → 步骤 D：运行 ty check . → 0 errors（如项目已配置 ty）
  → 步骤 E：运行 ruff format --check . → 0 errors
  → 步骤 F：检查接口变更（优先 git diff，无 git 历史则手动比对）
  → 全部通过 → 进入升级信号检测
```

**项目无 ruff/mypy 时的降级策略：**
- 无 ruff → 尝试 flake8，都没有则跳过 lint（记录 SKIPPED + 理由）
- 无 mypy → 尝试 pyright，都没有则跳过类型检查（记录 SKIPPED + 理由）
- 无 pytest → 尝试 unittest discover，都没有则要求人工确认

### 铁律三：验证结构 > 验证数量

10 个覆盖关键路径的测试 > 100 个覆盖琐碎路径的测试。

| 层级 | 验证目标 | Python 典型手段 |
|------|----------|----------------|
| 单元层 | 逻辑正确性 | pytest 纯函数测试、`@pytest.mark.parametrize` 边界矩阵 |
| 集成层 | 契约一致性 | pytest + httpx/TestClient、数据库 fixture |
| 端到端层 | 流程完整性 | pytest + 真实依赖、Playwright (Web) |

用 **"关键行为是否可被验证"** 衡量信心，不用测试数量。

#### 判定规则：什么是"关键业务规则"

| 条件 | Python 示例 |
|------|------------|
| 涉及金钱/财务 | 支付金额计算、退款逻辑、费率转换 |
| 涉及权限/安全 | 用户角色判断、资源访问控制、认证状态检查 |
| 涉及数据完整性 | 唯一性约束、外键关系、事务一致性 |
| 涉及状态机转换 | 订单状态流转、审批流程、生命周期管理 |
| 涉及外部契约 | API 响应格式、Pydantic model schema、事件消息结构 |
| 涉及并发安全 | asyncio 共享状态、线程安全、锁的正确使用 |

**不属于关键业务规则的：** getter/setter、`__repr__`、纯格式化函数、日志记录、配置常量。

### 铁律四：风险越高，验证越深

| 风险 | Python 典型场景 | 🚀 Prototype 验证深度 | 🏭 Production 验证深度 |
|------|----------------|----------------------|----------------------|
| 🟢 低 | 类型注解补充、docstring、格式化 | ruff + mypy + ty + 关联测试 | ruff + mypy + ty + 关联测试 |
| 🟡 中 | 新功能、API 变更、新依赖 | 上述 + 单元测试 | 上述 + 单元测试 + 集成测试 + 人工审查 |
| 🔴 高 | auth、payment、eval/exec、pickle、subprocess、asyncio 并发、数据迁移 | 上述 + bandit（仅输入验证+认证授权）+ 人工确认 | 上述 + bandit（全部）+ 边界测试 + 人工确认 |

#### Python 特有高风险关键词

以下关键词出现在代码中时，自动升级为 🔴 高风险：

| 关键词/模式 | 风险原因 |
|------------|---------|
| `eval()` / `exec()` | 任意代码执行 |
| `pickle.loads()` / `pickle.load()` | 反序列化可执行任意代码 |
| `subprocess.call(shell=True)` / `os.system()` | 命令注入 |
| `yaml.load()` (无 `Loader=SafeLoader`) | YAML 反序列化攻击 |
| `tempfile.mktemp()` | 竞态条件（应使用 `mkstemp`） |
| `assert` 用于安全检查 | 生产环境 `-O` 会跳过 assert |
| `# type: ignore` | 类型检查绕过，需审查原因 |
| `random` 用于安全场景 | 应使用 `secrets` 模块 |
| `__import__()` / `importlib.import_module()` 用户输入 | 动态导入攻击 |

### 铁律五：知道何时叫人

以下情况必须升级到人工判断：
- 改动触及安全敏感区域（认证、授权、支付）
- diff 扩散半径超出预期
- 业务规则存在歧义
- 测试全绿但存在以下可计算信号

#### Gate Function：升级信号检测

```
信号 1：覆盖率下降
  → 运行 pytest --cov 对比改动前后
  → 新增代码覆盖率 < 60% → 升级

信号 2：影响面过大
  → git diff --stat HEAD~1 统计改动文件数（无 git 历史则跳过）
  → 改动文件 > 10 个，或跨 3+ 个顶层目录 → 升级

信号 3：新增 TODO/FIXME/HACK/XXX
  → git diff HEAD~1 | grep -E "^\+.*(TODO|FIXME|HACK|XXX)"（无 git 历史则 grep 当前文件）
  → 存在 → 升级（显式的"还没做完"标记）

信号 4：接口契约变更
  → git diff HEAD~1 检查公开接口变更（无 git 历史则手动比对）
  → 存在函数签名变更、返回类型变更、Pydantic model 字段变更 → 升级

信号 5：安全敏感路径触及
  → git diff --name-only HEAD~1 | grep -iE "(auth|payment|crypto|session|token|admin)"
  → 无 git 历史则检查当前改动文件路径
  → 存在 → 升级
```

**升级不是失败，是工程成熟度的体现。**

---

## 决策流程

```
收到编码任务
  │
  ├─ ❶ 确定运行模式（默认 🚀 Prototype）
  │     用户说 "production" / "上线" / "ship it" → 🏭 Production
  │     否则 → 🚀 Prototype
  │
  ├─ ❷ 探测项目工具链
  │     检查 pyproject.toml / setup.py / Makefile / tox.ini
  │     确定 lint / type check / test / security 使用的工具
  │
  ├─ ❸ 理解确认（铁律一 Gate Function）
  │     → 快速风险预判（基于任务描述关键词 + Python 特有高风险词）
  │     → 按风险等级执行对应深度的理解确认
  │     → 🟢 一句话意图 / 🟡 WHAT+CONSTRAINTS / 🔴 完整模板+用户确认
  │
  ├─ ❹ 风险等级评估（铁律四 Gate Function）
  │     → 步骤 A：基于任务描述关键词预判风险
  │         触及 auth/payment/crypto/session/token/data-migration → 🔴
  │         触及 eval/exec/pickle/subprocess/os.system            → 🔴
  │         触及 asyncio 并发/多线程/共享可变状态                  → 🔴
  │         新功能 / API 变更 / 跨模块改动 / 新依赖               → 🟡
  │         类型注解 / docstring / 格式化 / 非破坏性新增           → 🟢
  │     → 步骤 B：如已有 git 历史，用 git diff 辅助验证预判
  │         改动文件 > 10 或跨 3+ 目录 → 升级风险等级
  │         ⚠️ git diff 只能向上修正（🟢→🟡→🔴），不能向下降级
  │     → 步骤 C：根据模式 + 风险等级选择验证深度（见铁律四表格）
  │
  ├─ ❺ 生成代码
  │
  ├─ ❻ 完成条件自检（铁律二 Gate Function）
  │     🚀 Prototype:
  │       → ruff check . → 0 errors
  │       → mypy . → 0 errors
  │       → pytest → exit code = 0（或人工确认核心行为）
  │       → 全部通过 → 进入升级信号检测
  │     🏭 Production:
  │       → pytest → exit code = 0
  │       → ruff check . → 0 errors
  │       → mypy . → 0 errors
  │       → ruff format --check . → 0 errors
  │       → 接口变更检查
  │       → 全部通过 → 进入升级信号检测
  │
  ├─ ❼ 升级信号检测（铁律五 Gate Function）
  │     🚀 Prototype: 执行信号 1-4（阻断），信号 5（警告但不阻断）
  │     🏭 Production: 执行全部 5 项信号（阻断）
  │     → 任一命中 → 输出风险摘要，请用户确认
  │     → 全部未命中 → 完成
  │
  └─ ❽ 完成
```

---

## Python 反模式警示

| 反模式 | 表现 | 解药 |
|--------|------|------|
| 🚦 绿灯幻觉 | 测试全过，系统已坏 | 验证结构，不只看数量（铁律三） |
| 🔧 局部修补 | 修症状不修原因 | 追问"为什么"到根因（铁律一） |
| 🧠 上下文丢失 | 忘记为什么这样做 | 关键决策写进代码或文档 |
| 📦 依赖爆炸 | 随意 pip install | 评估必要性、维护状态、替代方案 |
| 💧 抽象泄漏 | 绕过封装走捷径 | 遵守层间契约，不跨层访问 |
| 🐍 Python 特有：可变默认参数 | `def f(x=[])` | 使用 `None` 哨兵：`def f(x=None)` |
| 🐍 Python 特有：裸 except | `except:` 捕获一切 | `except Exception:` 或更具体的异常类型 |
| 🐍 Python 特有：循环引用 | 模块间互相 import | 提取公共模块或使用延迟导入 |
| 🐍 Python 特有：未关闭资源 | 文件/连接未用 `with` | 始终使用 context manager (`with` 语句) |
| 🐍 Python 特有：类型注解缺失 | 公开 API 无类型提示 | 公开函数必须有完整类型注解 |

---

## Gotchas

- "应该已经测过了" 不等于 VERIFIED。没有证据的验证等于未验证。
- 覆盖率是输入不是结论。负向路径、契约漂移、跨层不变量它都无法回答。
- Vibe Coding 的风险不在代码不够快，而在"完成条件"开始漂移。
- 局部信号增长 ≠ 系统整体可信度增长。ruff 通过、mypy 通过、覆盖率没掉，不代表"完成"。
- 不是所有风险都适合继续自动化下沉。有些改动最合理的动作是拉人进来。
- 🚀 Prototype 模式不等于"没有质量要求"——ruff 和 mypy 仍然必须通过。
- 原型代码有变成生产代码的倾向。切换到 🏭 Production 时，务必对已有代码做回溯审查。
- 关联测试 = 与改动文件直接相关的测试（通过文件路径或模块名匹配）。不是全量测试，也不是跳过测试。
- ⚠️ 已知风险：Prototype 无测试套件时的"人工确认核心行为"是弱约束——AI 无法验证用户声称的验证是否真实。这是速度与安全的折中，切换到 🏭 Production 时应补全测试。
- Python 的动态特性是双刃剑：灵活但也容易隐藏错误。mypy --strict 能捕获大部分问题，ty 能提供更快的反馈，但 duck typing 的边界仍需测试覆盖。
- `__init__.py` 的设计影响整个包的 API 表面——谨慎选择导出内容。

---

## 工具映射：铁律 → Gate Function

| 铁律 | Gate Function | 🚀 Prototype 工具 | 🏭 Production 工具 | 判定规则 |
|------|--------------|-------------------|-------------------|----------|
| 铁律一 | 理解确认（风险分级） | 对话 | 对话 | 🟢 一句话意图 / 🟡 WHAT+CONSTRAINTS / 🔴 完整模板+用户确认 |
| 铁律二 | 完成条件自检 | Bash（ruff + mypy + ty + pytest 或人工确认） | Bash（pytest + ruff + mypy + ty + ruff format + 契约检查） | 🚀 4 项通过 / 🏭 6 项通过 |
| 铁律三 | 验证结构审查 | Bash | Bash | 测试覆盖关键业务规则（金钱/权限/数据完整性/并发安全），非 getter/setter |
| 铁律四 | 风险等级评估 | 任务描述关键词 + `git diff --stat`（可选，仅向上修正） | 任务描述关键词 + `git diff --stat`（可选，仅向上修正） | 关键词预判 + git diff 辅助（只能 🟢→🟡→🔴，不能降级） |
| 铁律五 | 升级信号检测 | `git diff` + `grep`（可选，信号 5 警告不阻断） | `git diff` + `grep`（可选，全部 5 项阻断） | 任一命中 → 升级到人工 |

---

## 按需加载深度指南

以下参考文件仅在任务涉及对应领域时查阅，不要预先加载：

| 参考文件 | 何时加载 | 内容 |
|----------|----------|------|
| [references/architecture.md](references/architecture.md) | 新建模块、添加依赖、修改系统边界、包结构设计 | 模块边界、依赖管理、pyproject.toml、分层架构 |
| [references/security.md](references/security.md) | 🔴 高风险任务（🏭 全部章节 / 🚀 仅输入验证+认证授权） | 输入验证、认证授权、数据保护、Python 特有漏洞防护 |
| [references/quality-gates.md](references/quality-gates.md) | 提交前验证、测试策略、契约检查 | 五阶段验证、pytest 策略、coverage 配置、prek hooks |
| [references/refactoring.md](references/refactoring.md) | 代码腐化、技术债务、渐进式重构 | 重构模式、Python 代码气味、安全网 |
| [references/examples.md](references/examples.md) | 需要理解 good vs bad 模式时 | 五条铁律的 Python 正反示例 |
