# Good vs Bad 示例（Python）

加载条件：需要理解五条铁律在 Python 实践中的正反模式时

---

## 铁律一：先理解，再生成

### ❌ Bad — 不理解就动手

用户：给订单列表加个搜索功能

AI 直接生成代码：

```python
def search_orders(query: str) -> list[Order]:
    return Order.objects.filter(name__contains=query)
```

问题：不知道搜索范围（全字段还是特定字段？）、不知道性能要求（模糊还是精确？）、不知道是否需要分页、不知道排序规则。`__contains` 在大数据量上会导致全表扫描。

### ✅ Good — 先理解再生成

用户：给订单列表加个搜索功能

AI 先确认理解：

> 需要确认几点再动手：
> 1. 搜索范围 — 订单号、客户名、还是全部字段？
> 2. 匹配方式 — 模糊匹配还是精确匹配？
> 3. 数据规模 — 预计多少条记录？是否需要分页？
> 4. 排序规则 — 按相关性还是按时间？

确认后生成：

```python
from pydantic import BaseModel, Field

class OrderSearchParams(BaseModel):
    query: str = Field(..., min_length=1, max_length=100)
    page: int = Field(1, ge=1)
    page_size: int = Field(20, ge=1, le=100)
    sort_by: str = Field("created_at", pattern="^(created_at|amount|order_no)$")

def search_orders(params: OrderSearchParams) -> PaginatedResult[Order]:
    if not params.query:
        return PaginatedResult.empty()

    qs = Order.objects.filter(
        Q(order_no__icontains=params.query)
        | Q(customer_name__icontains=params.query)
    ).order_by(f"-{params.sort_by}")

    return PaginatedResult.from_queryset(qs, page=params.page, page_size=params.page_size)
```

---

## 铁律二：能跑 ≠ 完成

### ❌ Bad — "能跑"就提交

用户：给用户注册加个邮箱验证

AI 实现：

```python
@app.post("/register")
async def register(data: dict):
    email = data["email"]
    password = data["password"]
    await send_verification_email(email)
    return {"status": "ok"}
```

AI：完成了，注册接口可以发送验证邮件。

问题：没处理邮箱格式错误、没处理重复注册、没处理发送失败、没存储用户记录、没验证密码强度、没有类型安全。

### ✅ Good — 完成条件全部满足

```python
from pydantic import BaseModel, EmailStr, Field

class RegisterRequest(BaseModel):
    email: EmailStr
    password: str = Field(..., min_length=8, max_length=128)

class RegisterResponse(BaseModel):
    status: str
    message: str

@app.post("/register", response_model=RegisterResponse)
async def register(req: RegisterRequest) -> RegisterResponse:
    if await user_repo.exists(email=req.email):
        raise HTTPException(409, "EMAIL_ALREADY_REGISTERED")

    if not meets_password_policy(req.password):
        raise HTTPException(400, "WEAK_PASSWORD")

    try:
        user = await user_repo.create(
            email=req.email,
            password=hash_password(req.password),
        )
        await send_verification_email(user.email, user.verification_token)
        return RegisterResponse(status="verification_sent", message="Check your email")
    except EmailSendError:
        await user_repo.mark_verification_pending(user.id)
        return RegisterResponse(status="verification_pending", message="Try again later")
```

AI 自检：
- [x] 正向路径：正常注册流程
- [x] 负向路径：邮箱格式错误、重复注册、弱密码、邮件发送失败
- [x] 回归风险：不影响已有登录接口
- [x] 契约一致：Pydantic model 定义了请求/响应契约
- [x] 类型安全：mypy --strict 通过

---

## 铁律三：验证结构 > 验证数量

### ❌ Bad — 数量多但结构空洞

```python
def test_user_model():
    user = User(name="Alice", email="alice@test.com")
    assert user.name == "Alice"
    assert user.email == "alice@test.com"
    assert str(user) != ""
    assert user.id is not None
    assert user.created_at is not None
    assert user.updated_at is not None
    assert user.name != ""
    assert user.email != ""
    assert isinstance(user.id, int)
    assert isinstance(user.created_at, datetime)
    # ... 20 more trivial assertions
```

10 个测试全过，但核心业务规则（邮箱唯一性、密码策略、验证流程）一个都没验证。

### ✅ Good — 数量少但结构精准

```python
import pytest

def test_duplicate_email_rejected():
    User.create(email="alice@test.com", password="Str0ng!pass")
    with pytest.raises(DuplicateEmailError):
        User.create(email="alice@test.com", password="An0ther!pass")

def test_weak_password_rejected():
    with pytest.raises(WeakPasswordError):
        User.create(email="bob@test.com", password="123")

def test_verification_token_generated():
    user = User.create(email="carol@test.com", password="Str0ng!pass")
    assert user.verification_token is not None
    assert len(user.verification_token) == 32

@pytest.mark.parametrize("password,expected", [
    ("short", False),
    ("alllowercase1!", False),
    ("ALLUPPERCASE1!", False),
    ("NoDigits!@#", False),
    ("Str0ng!Pass1", True),
])
def test_password_policy(password: str, expected: bool):
    assert meets_password_policy(password) == expected
```

4 个测试（含参数化），覆盖了关键业务规则。每个测试验证一个不可妥协的不变量。

---

## 铁律四：风险越高，验证越深

### ❌ Bad — 一刀切验证

用户：修改了登录页面的按钮颜色，同时也修改了支付回调的签名验证逻辑

AI：跑一遍测试，全绿，提交。

问题：按钮颜色（🟢低风险）和支付签名验证（🔴高风险）用同一验证深度。

### ✅ Good — 风险分级验证

AI 评估：
- 按钮颜色修改 → 🟢 低风险 → ruff + 关联测试即可
- 支付签名验证修改 → 🔴 高风险 → 需要：

```python
import pytest
from unittest.mock import patch

def test_valid_signature_accepted():
    callback = make_payment_callback(valid=True)
    assert process_payment(callback).success

def test_tampered_amount_rejected():
    callback = make_payment_callback(amount=9999.99, tamper_amount=True)
    assert process_payment(callback).rejected

def test_replayed_callback_rejected():
    callback = make_payment_callback(valid=True)
    process_payment(callback)
    assert process_payment(callback).rejected  # 幂等性

def test_expired_signature_rejected():
    callback = make_payment_callback(timestamp=hours_ago(2))
    assert process_payment(callback).rejected

def test_malformed_payload_rejected():
    with pytest.raises(InvalidPaymentError):
        process_payment(b"not-valid-json")
```

AI：支付签名验证是 🔴 高风险区域，除单元测试外，建议 bandit 安全扫描 + 人工确认签名算法与支付平台文档一致。

---

## 铁律五：知道何时叫人

### ❌ Bad — 万事皆自动

用户：把用户表从 SQLite 迁移到 PostgreSQL

AI 直接生成迁移脚本并执行：

```python
# Migration script
import sqlite3, psycopg2

src = sqlite3.connect("old.db")
dst = psycopg2.connect("postgresql://...")
for row in src.execute("SELECT * FROM users"):
    dst.execute("INSERT INTO users VALUES %s", row)
dst.commit()
```

AI：迁移完成。

问题：数据迁移是 🔴 高风险操作，没有数据校验、没有回滚方案、没有人工确认。

### ✅ Good — 高风险操作升级到人工

AI：

> 这是 🔴 高风险操作（数据迁移），我准备了迁移方案但需要人工确认后再执行：
>
> **迁移方案：**
> 1. 创建 PostgreSQL 目标表结构（Alembic migration）
> 2. 小批量试迁移（100 条）并校验数据一致性
> 3. 全量迁移
> 4. 校验行数和关键字段
> 5. 保留 SQLite 数据 7 天作为回滚保障
>
> **需要人工确认：**
> - 迁移窗口期（是否需要停服？）
> - 字段类型映射是否正确（SQLite 的 `INTEGER` → PostgreSQL 的 `BIGINT`）
> - 是否需要灰度切换（先读旧库写新库，再切换读）
>
> 确认后我将生成可执行的迁移脚本。

---

## 综合场景：Python Vibe Coding 典型陷阱

### ❌ Bad — 典型 Vibe Coding 翻车

```
用户：加个导出 CSV 功能
AI：（直接生成）→ 能跑了
用户：加个导入功能
AI：（直接生成）→ 能跑了
用户：加个定时任务自动导出
AI：（直接生成）→ 能跑了
用户：为什么生产环境内存爆了？
AI：...
```

问题：每一步都是"能跑就行"，没有考虑编码一致性、大文件内存占用、错误处理、并发安全。局部绿灯持续累积，全局熵增到爆。

### ✅ Good — Vibe Coding Guardian 守护下的 Vibe Coding

```
用户：加个导出 CSV 功能
AI：🟡 中风险（文件 I/O + 潜在大数据量）。确认 — 预计数据量多大？是否需要流式导出？
用户：最多 10 万行
AI：（生成流式导出 + csv.writer + 编码处理 + 大文件分页）→ ruff + mypy + pytest 自检通过

用户：加个导入功能
AI：🔴 高风险（数据写入 + 文件上传）。生成导入 + Pydantic 格式校验 + 重复检测 + 错误回滚 + bandit 扫描 → 自检通过

用户：加个定时任务自动导出
AI：🟡 中风险（后台任务 + 资源占用）。生成 APScheduler 定时任务 + 文件锁 + 超时保护 → 自检通过
```

每一步都评估风险、验证完成条件、处理边界情况。速度略慢，但不会积累隐性债务。
