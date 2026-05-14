# 架构守护参考（Python）

加载条件：新建模块、添加依赖、修改系统边界、跨模块调用、包结构设计

---

## Python 包结构规范

### 推荐结构

```
my_project/
├── pyproject.toml          # 项目元数据 + 工具配置（PEP 621）
├── src/
│   └── my_project/         # src layout（推荐，避免导入歧义）
│       ├── __init__.py
│       ├── core/
│       ├── services/
│       ├── models/
│       └── utils/
├── tests/
│   ├── conftest.py
│   ├── unit/
│   ├── integration/
│   └── e2e/
└── scripts/                # 开发/运维脚本
```

### src layout vs flat layout

| 布局 | 优点 | 缺点 |
|------|------|------|
| `src/` layout | 避免 `import` 歧义，强制安装后测试 | 需要 `pip install -e .` |
| flat layout | 简单直接 | 可能意外导入本地目录而非安装包 |

**推荐：src layout。** 它确保测试运行在已安装的包上，而非源码目录。

### `__init__.py` 设计原则

- 明确导出公开 API：`__all__` 列表
- 不在 `__init__.py` 中放业务逻辑
- 使用 lazy import 避免循环导入
- 保持 `__init__.py` 精简——它是包的"门面"

```python
# ✅ Good
from my_project.core.service import Service
from my_project.models.user import User

__all__ = ["Service", "User"]

# ❌ Bad — 在 __init__.py 中放业务逻辑
import os
if os.getenv("ENV") == "prod":
    from .prod_service import Service
else:
    from .dev_service import Service
```

---

## 模块边界规则

1. 每个模块有且只有一个变更理由（单一职责）
2. 模块间依赖方向：高层 → 低层，业务 → 基础设施
3. 跨模块调用必须通过公开接口，不直接访问内部实现
4. 模块内聚高、耦合低：改一个理由只动一个模块

边界检查：
- 新模块的职责能否用一句话描述？
- 依赖方向是否一致（无反向依赖、无循环依赖）？
- 公开接口是否最小化？
- 内部实现是否对外不可见？

### 循环导入检测

Python 循环导入是常见的架构问题：

```bash
# 使用 import-linter 检测
pip install import-linter
# 在 setup.cfg 或 pyproject.toml 配置规则后运行
lint-imports
```

或手动检测：
```bash
# 找出互相导入的模块对
grep -rn "from my_project.module_a" my_project/module_b/
grep -rn "from my_project.module_b" my_project/module_a/
```

---

## 依赖管理

### pyproject.toml 规范

```toml
[project]
name = "my-project"
requires-python = ">=3.10"
dependencies = [
    "fastapi>=0.100",
    "pydantic>=2.0",
    "sqlalchemy>=2.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "mypy>=1.0",
    "ruff>=0.1",
    "bandit>=1.7",
]
```

### 引入新依赖前必须回答

| 问题 | 判断标准 |
|------|----------|
| 是否必要？ | 能否用标准库或已有依赖解决？ |
| 维护状态？ | 最近 6 个月有无更新？GitHub stars + 最近 commit |
| 体积影响？ | 是否引入大量传递依赖？ |
| 许可证？ | 是否与项目许可证兼容？（GPL 传染性） |
| 替代方案？ | 有无更轻量的替代？ |
| 类型支持？ | 是否有 py.typed 标记或 .pyi 存根？ |

**原则：** 标准库 → 已有依赖 → 维护活跃的依赖。一个功能只用一个库。

### 包管理器选择

| 工具 | 适用场景 | 特点 |
|------|----------|------|
| **uv** | 新项目首选 | 极速，Rust 实现，兼容 pip |
| **pip + venv** | 简单项目 | 标准方案，无额外依赖 |
| **poetry** | 需要发布到 PyPI | 依赖解析强，lockfile 可靠 |
| **pdm** | PEP 标准优先 | 支持 PEP 621/660/665 |
| **conda** | 数据科学 / 跨语言 | 非 Python 依赖管理 |

---

## API 契约稳定性

| 变更类型 | 处理方式 |
|----------|----------|
| 新增可选参数（有默认值） | ✅ 允许 |
| 新增端点/函数 | ✅ 允许 |
| 修改参数语义 | ⚠️ 需版本升级 |
| 删除参数 / 修改返回类型 | ❌ 禁止（除非主版本升级） |
| 修改异常类型 | ⚠️ 需评估影响范围 |
| 修改 Pydantic model 字段 | ⚠️ 需评估序列化兼容性 |

### Python 契约测试

```python
# 使用 beartype 做运行时契约检查
from beartype import beartype

@beartype
def process_order(order_id: int, amount: Decimal) -> OrderResult:
    ...
```

### 类型注解即契约

公开 API 的类型注解就是契约的一部分：

```python
# ✅ Good — 明确的类型契约
def get_user(user_id: int) -> User | None:
    ...

# ❌ Bad — Any 意味着没有契约
def get_user(user_id: int) -> Any:
    ...
```

---

## 数据流原则

1. 数据流向清晰可追踪，不存在隐式数据通道
2. 避免双向依赖和循环引用
3. 状态变更必须可观测（事件、日志、状态机）
4. 事件/消息必须有明确的 schema（Pydantic model / dataclass）
5. 数据所有权清晰：每个数据有且只有一个写入者

### Python 特有数据流陷阱

- **可变对象共享**：函数间传递 list/dict 可能导致意外修改 → 使用 `tuple` / `frozenset` 或 `copy.deepcopy`
- **全局状态**：模块级变量是隐式全局状态 → 使用依赖注入或配置对象
- **闭包捕获**：闭包可能意外捕获可变外层变量

---

## 分层架构守则

```
表现层（FastAPI Router / CLI）
    ↓ 只调用
应用层（Use Case / Service）
    ↓ 只调用
领域层（Domain / Entity / Model）
    ↓ 只调用
基础设施层（Repository / External API / Database）
```

- 表现层不直接访问基础设施层
- 领域层不依赖任何外层
- 基础设施层实现领域层定义的接口（依赖倒置）
- 跨层调用只通过协议（Protocol）或抽象基类（ABC），不通过具体实现

### Python 依赖倒置实现

```python
# 领域层定义接口
from typing import Protocol

class UserRepository(Protocol):
    def get_by_id(self, user_id: int) -> User | None: ...
    def save(self, user: User) -> None: ...

# 基础设施层实现
class SQLAlchemyUserRepository:
    def get_by_id(self, user_id: int) -> User | None:
        ...
    def save(self, user: User) -> None:
        ...

# 应用层依赖接口，不依赖实现
class UserService:
    def __init__(self, repo: UserRepository) -> None:
        self._repo = repo
```
