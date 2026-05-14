# 重构治理参考（Python）

加载条件：代码腐化、技术债务、渐进式重构、功能扩展到已有代码

---

## 何时重构

- 添加新功能前，先让代码接纳新功能
- 修复 bug 时，消除导致 bug 的结构问题
- 代码重复三次，必须提取
- 命名不再表达意图时，立即重命名
- 注释需要解释"怎么做"而非"为什么"时，代码需要重构
- 函数超过 50 行或类超过 300 行时
- 循环导入出现时

## 何时 NOT 重构

- 代码虽然丑但稳定且不需要修改
- 重写成本低于重构成本
- 没有测试覆盖的重构（先补测试再重构）
- 截止日期前的紧急交付

---

## Python 代码气味检测

| 气味 | 检测信号 | 重构手法 |
|------|----------|----------|
| 长函数 | >50 行 | 提取子函数 |
| 深嵌套 | >3 层 | 卫语句提前返回 / `itertools.groupby` |
| 过多参数 | >4 个 | 引入 dataclass / Pydantic model |
| 重复代码 | 三处以上相同逻辑 | 提取共享函数 |
| 霰弹式修改 | 一个变化点散布多处 | 收敛职责到同一模块 |
| 依恋情结 | 大量访问其他对象数据 | 移动行为到数据所在类 |
| 数据泥团 | 多个参数总是一起传递 | 提取 dataclass |
| 过长的 import | 单文件导入 >15 个 | 拆分模块职责 |
| 可变默认参数 | `def f(x=[])` | `def f(x=None)` |
| 裸 except | `except:` | `except Exception:` |
| 上帝类 | 单个类 >500 行 | 拆分为多个职责单一的类 |
| 过度继承 | 继承层级 >3 层 | 组合优于继承 |

---

## 渐进式重构模式

### 绞杀者（Strangler Fig）

新功能走新路径，逐步替换旧路径，直到旧路径可安全移除。

```python
# 阶段 1：新旧并存
class OrderService:
    def process(self, order):
        if feature_flag("new_processor"):
            return self._new_process(order)  # 新路径
        return self._legacy_process(order)   # 旧路径

# 阶段 2：新路径稳定后，移除旧路径
class OrderService:
    def process(self, order):
        return self._new_process(order)
```

适用：大型系统迁移、ORM 切换、框架升级

### 分支抽象

先提取协议/ABC，再切换实现，最后移除旧实现。

```python
# 1. 提取协议
from typing import Protocol

class CacheBackend(Protocol):
    def get(self, key: str) -> Any | None: ...
    def set(self, key: str, value: Any, ttl: int) -> None: ...

# 2. 新旧实现都满足协议
class RedisCache:
    def get(self, key: str) -> Any | None: ...
    def set(self, key: str, value: Any, ttl: int) -> None: ...

class InMemoryCache:
    def get(self, key: str) -> Any | None: ...
    def set(self, key: str, value: Any, ttl: int) -> None: ...

# 3. 通过配置切换
cache: CacheBackend = RedisCache() if USE_REDIS else InMemoryCache()
```

适用：替换组件实现、切换底层存储、mock 测试

### 两步修改

添加新代码和删除旧代码不在同一次提交。

```
提交 A：添加新代码（旧代码仍在，通过 deprecation warning 标记）
提交 B：删除旧代码（新代码已验证）
```

```python
# 提交 A：新旧并存
def new_function():
    ...

def old_function():
    import warnings
    warnings.warn("Use new_function()", DeprecationWarning, stacklevel=2)
    return new_function()

# 提交 B：移除旧函数
```

适用：任何可能破坏功能的修改

### 小步提交

每步可独立验证，不积累大爆炸。

```
每步：修改 → ruff check → mypy → pytest → 提交
而不是：修改 → 修改 → 修改 → ruff check → mypy → pytest → 提交
```

---

## 技术债务分类

| 债务类型 | 特征 | 处理策略 |
|----------|------|----------|
| 故意欠债 | 为赶进度有意妥协 | 记录原因和还债计划 |
| 无意欠债 | 因认知不足产生 | 发现即修，不留过夜 |
| 过时债务 | 环境变化导致 | 评估影响，按优先级处理 |
| 架构债务 | 设计缺陷累积 | 制定渐进式迁移方案 |
| 类型债务 | 缺少类型注解 | 逐模块补全，新代码必须有 |

还债优先级：影响用户的核心问题 > 阻碍开发效率的瓶颈 > 增加故障风险的隐患 > 降低可读性的混乱

---

## Python 特有重构技巧

### 用 dataclass 替代字典传递

```python
# ❌ Before — 字典传递，无类型安全
def create_order(data: dict):
    amount = data["amount"]  # KeyError?
    currency = data.get("currency", "USD")  # 类型?

# ✅ After — dataclass
from dataclasses import dataclass

@dataclass
class OrderInput:
    amount: Decimal
    currency: str = "USD"

def create_order(data: OrderInput):
    amount = data.amount  # 有类型，有补全
```

### 用 Protocol 替代 ABC

```python
# ❌ ABC — 强制继承
from abc import ABC, abstractmethod

class Serializer(ABC):
    @abstractmethod
    def serialize(self, data: Any) -> bytes: ...

# ✅ Protocol — 结构化子类型（duck typing）
from typing import Protocol

class Serializer(Protocol):
    def serialize(self, data: Any) -> bytes: ...

# 任何有 serialize 方法的类都自动满足协议，无需继承
```

### 用 pathlib 替代 os.path

```python
# ❌ os.path
import os
config_path = os.path.join(os.environ["HOME"], ".config", "app", "settings.json")

# ✅ pathlib
from pathlib import Path
config_path = Path.home() / ".config" / "app" / "settings.json"
```

### 用 contextlib 简化上下文管理

```python
from contextlib import contextmanager

@contextmanager
def database_transaction(session):
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
```

---

## 重构安全网

### 重构前

- 现有测试是否覆盖要重构的代码？
- 重构目标是否明确？
- 重构范围是否可控？
- mypy 基线是否通过？（确保重构后能对比）

### 重构中

- 每步修改后 `ruff check` + `mypy` + `pytest` 是否仍然通过？
- 行为是否保持不变（重构不改行为）？
- 是否遵循两步修改原则？
- 是否使用了 `# type: ignore`？如果是，为什么？

### 重构后

- 新结构是否更清晰？
- 测试是否需要更新？
- 是否有遗留旧代码需要清理？
- 重构结果是否记录（commit message 说明原因）？
- mypy 严格度是否可以提高？
