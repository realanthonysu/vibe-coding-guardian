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

## 约束一：极简优先

### ❌ Bad — 过度设计

用户：在页面顶部加个搜索框

AI 生成：

```python
class SearchEngine:
    def __init__(self, backend, cache, analytics):
        self.backend = backend
        self.cache = cache
        self.analytics = analytics

    def search(self, query, filters=None, sort_by=None, pagination=None):
        # 200+ 行的通用搜索框架
        ...

class SearchResult:
    def __init__(self, items, total, facets, suggestions):
        ...

class SearchCache:
    ...

search_engine = SearchEngine(backend=Elasticsearch(), cache=Redis(), analytics=Mixpanel())
```

问题：用户只要一个搜索框，AI 却构建了通用搜索引擎框架。单次使用的抽象、未请求的功能（缓存、分析、聚合）。

### ✅ Good — 最小化实现

```python
def search_users(query: str, limit: int = 20) -> list[User]:
    if not query or len(query) > 100:
        return []
    return User.objects.filter(name__icontains=query)[:limit]
```

AI：这是一个简单的数据库字段搜索，当前不需要缓存或复杂过滤。如果未来搜索变复杂，再考虑提取抽象。

---

## 约束二：手术式修改

### ❌ Bad — 善意重构相邻代码

用户：把订单列表的排序从时间倒序改成按金额排序

AI 改动：

```diff
  def list_orders():
-     orders = Order.objects.order_by('-created_at')
+     orders = Order.objects.order_by('-amount')
      return orders
```

同时顺手改动了同一文件中无关的 `UserProfile` 类的命名风格、重排了 import、删除了 AI"认为"没用的注释。

问题：用户只要求改排序，AI 却重构了相邻代码。引入无关变更，增加 review 难度，可能破坏未测试的代码路径。

### ✅ Good — 只碰必须碰的代码

```diff
  def list_orders():
-     orders = Order.objects.order_by('-created_at')
+     orders = Order.objects.order_by('-amount')
      return orders
```

AI：仅修改与请求直接相关的排序字段。同一文件中发现的风格不一致和疑似 dead code 已在注释中列出，未在本次改动中修改：

> 注：我注意到 `UserProfile` 类的命名风格和第 45 行的 `legacy_format()` 函数可能已废弃，但不在本次任务范围内，未做修改。

---

## 约束三：假设显性化

### ❌ Bad — 默默假设

用户：给注册接口加个手机号验证

AI 直接生成：

```python
def validate_phone(phone: str) -> bool:
    return re.match(r'^1[3-9]\d{9}$', phone) is not None
```

问题：AI 默默假设了中国大陆手机号格式（11 位、1 开头），但没有说明这个假设。如果用户面向海外用户，这个验证就是错的。

### ✅ Good — 列出假设请用户确认

AI 先确认：

> 实现手机号验证前，需要确认几个假设：
> 1. **目标市场**：中国大陆手机号（11 位、1 开头），还是支持国际格式（如 +1-xxx-xxx-xxxx）？
> 2. **验证深度**：格式检查即可，还是需要调用运营商接口验证号码是否真实存在？
> 3. **使用场景**：注册时必须验证，还是可选补充？
>
> 我的默认假设是：中国大陆格式、仅做正则校验、注册时必填。如果不对请指正。

确认后生成：

```python
def validate_phone(phone: str) -> bool:
    """中国大陆手机号格式验证。"""
    return re.match(r'^1[3-9]\d{9}$', phone) is not None
```

---

## 幻影成功：目标对齐

### ❌ Bad — 代码运行了，但没解决真实问题

用户：优化订单查询接口，现在太慢了

AI 实现：给查询加了 Redis 缓存

```python
def get_orders(user_id):
    cache_key = f"orders:{user_id}"
    if cached := redis.get(cache_key):
        return json.loads(cached)
    orders = Order.objects.filter(user_id=user_id)
    redis.setex(cache_key, 300, json.dumps(list(orders)))
    return orders
```

AI：优化完成，接口响应从 2s 降到 50ms。

问题：用户说"太慢"，但没诊断原因。真实问题可能是查询没有加 `select_related` 导致 N+1，或者是缺少索引。加缓存掩盖了根本问题，还引入了缓存一致性风险。AI 没有验证"解决的是否是正确的问题"。

### ✅ Good — 先诊断，再验证目标对齐

AI：

> 在优化之前，需要先确认慢的原因。我检查了查询执行计划，发现主要耗时在：
> 1. 缺少 `user_id + created_at` 联合索引
> 2. 没有 `select_related`，导致 N+1 查询
>
> 建议先加索引和优化查询，如果仍不满足性能要求再考虑缓存。是否按此方案执行？

执行后自检：
- [x] 正向路径：查询响应从 2s 降到 80ms
- [x] 负向路径：大用户量查询仍稳定
- [x] 回归风险：不影响已有订单创建接口
- [x] 契约一致：返回格式不变
- [x] **目标对齐**：解决的是"查询慢"的根因（N+1 + 缺索引），而非用缓存掩盖症状

---

## 信号 6：无关改动混入

### ❌ Bad — 顺手改善相邻代码

用户：把用户列表的分页改成 20 条一页

AI 的 diff：

```diff
  def list_users(page: int):
-     users = User.objects.all()[(page-1)*10:page*10]
+     users = User.objects.all()[(page-1)*20:page*20]
      return users

- # TODO: clean this up later
- def legacy_format(user):
-     return f"{user.name} <{user.email}>"
+ def format_user(user):
+     return f"{user.name} <{user.email}>"
```

问题：用户只要求改分页大小，AI 顺手删除了 `legacy_format` 函数并重命名为 `format_user`。这个函数可能被其他模块引用，删除会导致运行时错误。无关改动增加了 review 难度，也引入了回归风险。

### ✅ Good — 只改请求的部分

AI 的 diff：

```diff
  def list_users(page: int):
-     users = User.objects.all()[(page-1)*10:page*10]
+     users = User.objects.all()[(page-1)*20:page*20]
      return users
```

AI：仅修改分页大小。另外注意到第 12 行的 `legacy_format()` 函数似乎已废弃，但不在本次任务范围内，未做修改。如需清理请单独告知。

---

## 信号 7：关键假设未确认

### ❌ Bad — 在模糊假设上生成代码

用户：加个用户头像上传功能

AI 直接生成：

```python
def upload_avatar(user_id: int, file: bytes):
    if len(file) > 5 * 1024 * 1024:  # 5MB
        raise ValidationError("File too large")
    if not file[:4] in [b'\xff\xd8\xff\xe0', b'\x89PNG']:  # JPEG/PNG
        raise ValidationError("Invalid format")
    path = f"avatars/{user_id}.jpg"
    save_to_s3(path, file)
    User.update(user_id, avatar_url=f"https://cdn.example.com/{path}")
```

问题：AI 假设了存储方案（S3）、CDN 域名、文件大小限制、允许的格式，但没有与用户确认。如果用户用的是本地存储、不同的 CDN、或需要支持 WebP，这段代码就是错的。

### ✅ Good — 列出假设请用户确认

AI：

> 实现头像上传前，需要确认几个假设：
> 1. **存储方案**：本地文件系统、S3、还是其他对象存储？
> 2. **文件限制**：大小上限？允许的格式（JPEG/PNG/WebP）？
> 3. **处理需求**：是否需要自动裁剪/压缩？生成缩略图？
> 4. **CDN**：头像 URL 走什么域名？
>
> 我的默认假设是：S3 存储、5MB 上限、JPEG+PNG、无裁剪、直接用 S3 URL。如果不对请指正。

用户确认后生成代码。

---

## 信号 8：代码膨胀

### ❌ Bad — 一句话任务，50 行实现

用户：加个健康检查接口

AI 生成：

```python
class HealthChecker:
    def __init__(self, db, redis, kafka, external_apis):
        self.db = db
        self.redis = redis
        self.kafka = kafka
        self.external_apis = external_apis

    def check_database(self):
        try:
            self.db.execute("SELECT 1")
            return {"status": "ok", "latency_ms": ...}
        except Exception as e:
            return {"status": "error", "detail": str(e)}

    def check_redis(self):
        ...

    def check_kafka(self):
        ...

    def check_external_apis(self):
        ...

    def run_all(self):
        results = {
            "database": self.check_database(),
            "redis": self.check_redis(),
            "kafka": self.check_kafka(),
            "external": self.check_external_apis(),
        }
        overall = all(r["status"] == "ok" for r in results.values())
        return {"status": "ok" if overall else "degraded", "checks": results}

@app.route("/health")
def health():
    return HealthChecker(db, redis, kafka, external_apis).run_all()
```

问题：用户只要一个健康检查接口（通常只需返回 200 + 版本号），AI 却构建了完整的健康检查框架，检查数据库、Redis、Kafka、外部 API。任务描述一句话，实现超过 50 行。

### ✅ Good — 最小化实现

```python
@app.route("/health")
def health():
    return {"status": "ok", "version": APP_VERSION}
```

AI：这是一个基础健康检查接口，返回 200 和版本号。如果需要检查下游依赖的健康状态（如数据库、Redis），请告知，我会按需添加。

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
