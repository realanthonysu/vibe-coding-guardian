# 质量门禁参考（Python）

加载条件：提交前验证、测试策略制定、契约检查、覆盖率分析

---

## 五阶段验证模型

```
阶段一：规则前置 ──── 定义变更如何被证明安全
    ↓
阶段二：基础门槛 ──── ruff · mypy · ty · pytest · bandit
    ↓
阶段三：风险分流 ──── 识别高风险区域，决定验证深度
    ↓
阶段四：深度验证 ──── API 一致性 · 安全扫描 · E2E
    ↓
阶段五：决策闭环 ──── 硬门禁 + 评分 + 人工升级
```

越靠左，问题越确定，越便宜，越适合自动阻断。
越往右，问题越依赖语义和上下文，越需要人的专业介入。

---

## Python 工具链配置

### ruff（Lint + Format）

```toml
# pyproject.toml
[tool.ruff]
target-version = "py312"
line-length = 120

[tool.ruff.lint]
select = [
    "E",    # pycodestyle errors
    "W",    # pycodestyle warnings
    "F",    # pyflakes
    "I",    # isort
    "N",    # pep8-naming
    "UP",   # pyupgrade
    "B",    # flake8-bugbear
    "A",    # flake8-builtins
    "C4",   # flake8-comprehensions
    "SIM",  # flake8-simplify
    "TCH",  # flake8-type-checking
    "RUF",  # ruff-specific
]
ignore = ["E501"]  # line length handled by formatter

[tool.ruff.lint.isort]
known-first-party = ["my_project"]
```

### mypy（类型检查 — 深度）

```toml
# pyproject.toml
[tool.mypy]
python_version = "3.12"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true

# 第三方库可能没有类型存根
[[tool.mypy.overrides]]
module = ["some_untyped_lib.*"]
ignore_missing_imports = true
```

### ty（类型检查 — 快速）

```toml
# pyproject.toml
[tool.ty]
python-version = "3.12"
```

Astral 出品的 Rust 类型检查器，速度比 mypy 快一个数量级。与 mypy 互补：mypy 深度检查，ty 快速反馈。

### pytest（测试）

```toml
# pyproject.toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_functions = ["test_*"]
addopts = [
    "-v",
    "--strict-markers",
    "--tb=short",
]
markers = [
    "slow: marks tests as slow",
    "integration: marks integration tests",
    "e2e: marks end-to-end tests",
]

[tool.coverage.run]
source = ["src"]
branch = true

[tool.coverage.report]
fail_under = 80
show_missing = true
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
    "if __name__ == .__main__.",
    "@overload",
]
```

---

## 测试分层

| 层级 | 建议占比 | 关注点 | Python 典型手段 |
|------|----------|--------|----------------|
| 单元测试 | 70% | 逻辑正确性 | pytest 纯函数测试、`@pytest.mark.parametrize` |
| 集成测试 | 20% | 契约一致性 | pytest + httpx / TestClient、数据库 fixture |
| 端到端测试 | 10% | 流程完整性 | pytest + 真实依赖、Playwright |

### pytest 最佳实践

```python
# ✅ Good — 参数化测试边界矩阵
import pytest

@pytest.mark.parametrize("input_val,expected", [
    ("", False),
    ("a" * 256, False),
    ("valid@email.com", True),
    ("not-an-email", False),
])
def test_email_validation(input_val, expected):
    assert is_valid_email(input_val) == expected

# ✅ Good — fixture 管理测试依赖
@pytest.fixture
def db_session():
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    with Session(engine) as session:
        yield session

# ✅ Good — 使用 tmp_path 而非手动创建临时目录
def test_file_processing(tmp_path):
    test_file = tmp_path / "data.csv"
    test_file.write_text("a,b,c\n1,2,3")
    result = process_csv(test_file)
    assert result.row_count == 1
```

### 测试组织

```
tests/
├── conftest.py          # 全局 fixtures
├── unit/
│   ├── conftest.py      # 单元测试 fixtures
│   ├── test_user.py
│   └── test_order.py
├── integration/
│   ├── conftest.py      # 集成测试 fixtures（数据库、HTTP 客户端）
│   ├── test_api.py
│   └── test_db.py
└── e2e/
    └── test_workflow.py
```

---

## 覆盖率 vs 验证

覆盖率是输入，不是结论。它无法回答：
- 负向路径是否被验证？
- 契约字段是否漂移？
- 跨层不变量是否被破坏？

**目标不是覆盖率数字，而是"变更后核心行为可被验证"。**

### 覆盖率命令

```bash
# 运行测试并生成覆盖率报告
pytest --cov=src --cov-report=term-missing --cov-report=html

# 仅查看变更文件的覆盖率
pytest --cov=src --cov-report=term-missing --cov-fail-under=80
```

---

## 完成条件清单

### 必须项（硬门禁）

- [ ] 正向路径：核心功能已验证
- [ ] 负向路径：错误和异常已处理
- [ ] 无回归：已有功能未受影响
- [ ] ruff check 通过：无 lint 错误
- [ ] mypy 通过：无类型错误

### 条件项（风险驱动）

- [ ] 边界条件：极端输入已测试（🟡中/🔴高风险）
- [ ] 契约一致：接口未破坏兼容性（🟡中/🔴高风险）
- [ ] 安全审查：无漏洞引入（🔴高风险）
- [ ] 性能基线：无明显退化（🟡中/🔴高风险）

### 证据项（可追溯）

- [ ] 验证结果有记录
- [ ] 关键决策有文档
- [ ] 已知限制有标注

---

## Git Hooks 配置（prek）

prek 是 pre-commit 的 Rust 实现替代，兼容 pre-commit 配置格式，速度更快，无需 Python 运行时。

```yaml
# .pre-commit-config.yaml（prek 兼容格式）
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
        additional_dependencies: [types-requests, pydantic]

  - repo: https://github.com/astral-sh/ty-pre-commit
    rev: v0.0.1
    hooks:
      - id: ty

  - repo: https://github.com/PyCQA/bandit
    rev: 1.8.3
    hooks:
      - id: bandit
        args: ["-c", "pyproject.toml"]
        additional_dependencies: ["bandit[toml]"]

  - repo: local
    hooks:
      - id: pytest-related
        name: run related tests
        entry: bash -c 'git diff --cached --name-only --diff-filter=ACM | grep "\.py$" | xargs -I{} dirname {} | sort -u | xargs -I{} sh -c "cd {} && test -d tests && pytest --tb=short -q || true"'
        language: system
        pass_filenames: false
```

> Note: 使用 prek 而非 pre-commit。prek 兼容 pre-commit 配置格式，安装方式：`cargo install prek` 或 `brew install prek`。

### Git Hooks 与铁律对应

| 铁律 | Hook 检查 | 阻断级别 |
|------|----------|----------|
| 铁律二：能跑 ≠ 完成 | pytest + ruff + mypy + ty 全部 exit 0 | 硬阻断 |
| 铁律三：验证结构 | 测试覆盖关键路径（非 getter/setter） | 软阻断（警告） |
| 铁律四：风险分级 | bandit 安全敏感检测 | 条件阻断 |

---

## 证据状态模型

| 状态 | 含义 |
|------|------|
| VERIFIED | 已验证，有证据支撑 |
| TODO | 待验证，尚未执行 |
| BLOCKED | 被阻塞，需外部依赖解决 |
| SKIPPED | 跳过，附理由 |

"应该已经测过了" ≠ VERIFIED。没有证据的验证等于未验证。

---

## 门禁升级规则

| 信号 | 动作 |
|------|------|
| 测试全绿但改动触及高风险目录 | 升级到深度验证 |
| diff 扩散超出单模块 | 触发人工审查 |
| 契约字段变更 | 触发集成测试 + 人工确认 |
| 新增外部依赖 | 触发安全扫描 + 架构审查 |
| 覆盖率下降 | 阻断提交，要求补充测试 |
| mypy `# type: ignore` 新增 | 要求说明理由 |
