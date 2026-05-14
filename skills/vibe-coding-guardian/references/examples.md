# Good vs Bad 示例

加载条件：需要理解五条铁律在实践中的正反模式时

---

## 铁律一：先理解，再生成

### ❌ Bad — 不理解就动手

用户：给订单列表加个搜索功能

AI 直接生成代码：

```python
def search_orders(query):
    results = Order.objects.filter(name__contains=query)
    return results
```

问题：不知道搜索范围（全字段还是特定字段？）、不知道性能要求（模糊还是精确？）、不知道是否需要分页、不知道排序规则。

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
def search_orders(query: str, page: int = 1, page_size: int = 20):
    if not query or len(query) > 100:
        return PaginatedResult.empty()
    return Order.objects.filter(
        Q(order_no__icontains=query) | Q(customer_name__icontains=query)
    ).order_by("-created_at").paginate(page=page, page_size=page_size)
```

---

## 铁律二：能跑 ≠ 完成

### ❌ Bad — "能跑"就提交

用户：给用户注册加个邮箱验证

AI 实现：

```python
@app.route("/register", methods=["POST"])
def register():
    email = request.json["email"]
    password = request.json["password"]
    send_verification_email(email)
    return {"status": "ok"}
```

AI：完成了，注册接口可以发送验证邮件。

问题：没处理邮箱格式错误、没处理重复注册、没处理发送失败、没存储用户记录、没验证密码强度。

### ✅ Good — 完成条件全部满足

```python
@app.route("/register", methods=["POST"])
def register():
    data = request.json
    email = data.get("email", "")
    password = data.get("password", "")

    if not is_valid_email(email):
        return error(400, "INVALID_EMAIL")
    if User.exists(email=email):
        return error(409, "EMAIL_ALREADY_REGISTERED")
    if not meets_password_policy(password):
        return error(400, "WEAK_PASSWORD")

    try:
        user = User.create(email=email, password=hash_password(password))
        send_verification_email(user.email, user.verification_token)
        return {"status": "verification_sent"}, 201
    except EmailSendError:
        user.mark_verification_pending()
        return {"status": "verification_pending"}, 202
```

AI 自检：
- [x] 正向路径：正常注册流程
- [x] 负向路径：邮箱格式错误、重复注册、弱密码、邮件发送失败
- [x] 回归风险：不影响已有登录接口
- [x] 契约一致：返回结构符合 API 规范

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
```

3 个测试，覆盖了 3 条关键业务规则。每个测试验证一个不可妥协的不变量。

---

## 铁律四：风险越高，验证越深

### ❌ Bad — 一刀切验证

用户：修改了登录页面的按钮颜色，同时也修改了支付回调的签名验证逻辑

AI：跑一遍测试，全绿，提交。

问题：按钮颜色（🟢低风险）和支付签名验证（🔴高风险）用同一验证深度，高风险改动没有得到应有的深度验证。

### ✅ Good — 风险分级验证

AI 评估：
- 按钮颜色修改 → 🟢 低风险 → lint + 关联测试即可
- 支付签名验证修改 → 🔴 高风险 → 需要：

```python
def test_valid_signature_accepted():
    callback = make_payment_callback(valid=True)
    assert process_payment(callback).success

def test_tampered_amount_rejected():
    callback = make_payment_callback(amount=9999.99, tamper_amount=True)
    assert process_payment(callback).rejected

def test_replayed_callback_rejected():
    callback = make_payment_callback(valid=True)
    process_payment(callback)
    assert process_payment(callback).rejected

def test_expired_signature_rejected():
    callback = make_payment_callback(timestamp=hours_ago(2))
    assert process_payment(callback).rejected
```

AI：支付签名验证是 🔴 高风险区域，除单元测试外，建议人工确认签名算法与支付平台文档一致。

---

## 铁律五：知道何时叫人

### ❌ Bad — 万事皆自动

用户：我们要把用户表从 MySQL 迁移到新的 PostgreSQL 集群

AI 直接生成迁移脚本并执行：

```sql
-- Migration script
INSERT INTO pg_users SELECT * FROM mysql_users;
```

AI：迁移完成，所有记录已复制。

问题：数据迁移是 🔴 高风险操作，没有数据校验、没有回滚方案、没有人工确认。

### ✅ Good — 高风险操作升级到人工

AI：

> 这是 🔴 高风险操作（数据迁移），我准备了迁移方案但需要人工确认后再执行：
>
> **迁移方案：**
> 1. 在 PostgreSQL 创建目标表结构
> 2. 小批量试迁移（100 条）并校验数据一致性
> 3. 全量迁移
> 4. 校验行数和关键字段
> 5. 保留 MySQL 数据 7 天作为回滚保障
>
> **需要人工确认：**
> - 迁移窗口期（是否需要停服？）
> - 字段映射是否正确（MySQL 的 `tinyint(1)` → PostgreSQL 的 `boolean`）
> - 是否需要灰度切换（先读旧库写新库，再切换读）
>
> 确认后我将生成可执行的迁移脚本。

---

## 综合场景：Vibe Coding 典型陷阱

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
AI：确认 — 预计数据量多大？是否需要流式导出？
用户：最多 10 万行
AI：（生成流式导出 + 编码处理 + 大文件分页）→ 自检通过
用户：加个导入功能
AI：🟡 中风险（数据写入）。生成导入 + 格式校验 + 重复检测 + 错误回滚 → 自检通过
用户：加个定时任务自动导出
AI：🟡 中风险（后台任务 + 资源占用）。生成定时任务 + 并发锁 + 超时保护 → 自检通过
```

每一步都评估风险、验证完成条件、处理边界情况。速度略慢，但不会积累隐性债务。
