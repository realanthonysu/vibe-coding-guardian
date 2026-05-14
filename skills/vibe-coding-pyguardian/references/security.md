# 安全护栏参考（Python）

加载条件：
- 🏭 Production 模式下 🔴 高风险任务 → 执行全部章节
- 🚀 Prototype 模式下 🔴 高风险任务 → 只执行"输入验证"和"认证与授权"章节

---

## 核心原则

**零信任假设**：所有外部输入都是恶意的，直到被证明安全。

---

## 输入验证

1. 验证在入口处完成，不信任下游验证
2. 白名单优于黑名单
3. 验证失败返回通用错误，不泄露系统信息
4. 验证逻辑集中管理，不散布在业务代码中

### Python 输入验证工具

| 工具 | 适用场景 | 示例 |
|------|----------|------|
| **Pydantic** | API 请求/响应验证 | `class UserCreate(BaseModel): email: EmailStr` |
| **marshmallow** | 序列化 + 验证 | `Schema().load(data)` |
| **cerberus** | 轻量验证 | `Validator(schema).validate(data)` |
| **type hints + beartype** | 运行时类型检查 | `@beartype` 装饰器 |

### 检查清单

- 所有外部输入是否经过类型校验？
- 字符串输入是否经过长度限制和格式校验？
- 数值输入是否经过范围校验？
- 文件上传是否限制类型和大小？
- 错误信息是否避免泄露内部细节？
- Pydantic model 是否设置了 `max_length` / `gt` / `lt` 等约束？

```python
# ✅ Good — Pydantic 输入验证
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(..., min_length=8, max_length=128)
    age: int = Field(..., ge=0, le=150)

# ❌ Bad — 无验证的原始输入
def create_user(data: dict):
    email = data["email"]      # KeyError? 类型?
    password = data["password"]  # 长度? 空?
    ...
```

---

## 认证与授权

### 认证（Authentication）— 你是谁

- 密码必须哈希存储（bcrypt / argon2），永不明文
- Python 推荐：`bcrypt` / `argon2-cffi` / `passlib`
- 敏感操作需要重新认证
- 会话管理：合理的超时、安全的存储
- JWT：使用 `python-jose` 或 `PyJWT`，设置过期时间，不在 payload 放敏感数据

```python
# ✅ Good — 密码哈希
from passlib.context import CryptContext
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

# ❌ Bad — 明文或弱哈希
import hashlib
def hash_password(password: str) -> str:
    return hashlib.md5(password.encode()).hexdigest()  # ❌ MD5 不适合密码
```

### 授权（Authorization）— 你能做什么

- 最小权限原则：默认拒绝，按需授权
- 认证和授权分离，不混为一谈
- 权限检查在后端入口处执行，不依赖前端隐藏
- 资源级权限：不同用户只能访问自己的资源

```python
# ✅ Good — 装饰器式权限检查
from functools import wraps

def require_role(role: str):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            user = get_current_user()
            if role not in user.roles:
                raise PermissionDenied()
            return func(*args, **kwargs)
        return wrapper
    return decorator

@require_role("admin")
def delete_user(user_id: int):
    ...
```

---

## 数据保护

| 数据类型 | 处理方式 |
|----------|----------|
| 密码 | `bcrypt` / `argon2-cffi` 哈希存储 |
| PII | `cryptography.fernet` 加密存储 + 访问控制 |
| API 密钥/令牌 | 环境变量或 `python-dotenv` + `.env`（不提交到 git） |
| 日志 | 使用 `structlog` 脱敏，不记录敏感字段 |
| 配置 | 使用 `pydantic-settings` 管理，敏感值从环境变量读取 |

传输安全：HTTPS、安全头部（CSP、HSTS、X-Frame-Options）、不在 URL 传递敏感参数。

### .gitignore 必须包含

```
.env
.env.*
*.pem
*.key
credentials.json
secrets.yaml
```

---

## Python 特有漏洞防护

### 高危：反序列化攻击

```python
# ❌ 绝对禁止 — pickle 反序列化不可信数据
import pickle
data = pickle.loads(user_uploaded_bytes)  # 任意代码执行！

# ✅ 安全替代 — 使用 JSON 或 msgpack
import json
data = json.loads(user_uploaded_bytes)
```

**规则：永远不要 `pickle.loads()` 来自不可信来源的数据。** pickle 反序列化等同于 `exec()`。

### 高危：命令注入

```python
# ❌ 命令注入风险
import os
os.system(f"convert {filename} output.png")  # filename 可以是 "; rm -rf /"

# ✅ 安全 — 使用 subprocess + 列表参数
import subprocess
subprocess.run(["convert", filename, "output.png"], check=True)

# ❌ shell=True 风险
subprocess.run(f"grep {pattern} file.txt", shell=True)

# ✅ 安全
subprocess.run(["grep", pattern, "file.txt"], check=True)
```

### 高危：eval / exec

```python
# ❌ 任意代码执行
result = eval(user_input)
exec(user_code)

# ✅ 安全替代 — ast.literal_eval（仅解析字面量）
import ast
result = ast.literal_eval(user_input)  # 只接受 str, bytes, int, float, tuple, list, dict, set, bool, None

# ✅ 安全替代 — 使用专用解析器
# 需要计算表达式？用 simpleeval 库
from simpleeval import simple_eval
result = simple_eval(user_expression)
```

### 高危：YAML 反序列化

```python
# ❌ 任意代码执行
import yaml
data = yaml.load(user_input)  # 可以执行任意 Python 代码

# ✅ 安全
data = yaml.safe_load(user_input)  # 只解析基本类型
# 或使用 yaml.safe_load 全局替代
```

### 中危：SQL 注入

```python
# ❌ SQL 注入
query = f"SELECT * FROM users WHERE id = {user_id}"
cursor.execute(query)

# ✅ 参数化查询
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# ✅ ORM（自动参数化）
User.query.filter(User.id == user_id).first()
```

### 中危：路径遍历

```python
# ❌ 路径遍历
filepath = os.path.join(base_dir, user_filename)
with open(filepath) as f:  # user_filename 可以是 "../../etc/passwd"
    ...

# ✅ 安全 — 验证路径在允许目录内
import os
filepath = os.path.realpath(os.path.join(base_dir, user_filename))
if not filepath.startswith(os.path.realpath(base_dir)):
    raise ValueError("Invalid filename")
```

### 中危：随机数安全

```python
# ❌ 不安全随机数用于安全场景
import random
token = random.randint(100000, 999999)  # 可预测

# ✅ 安全随机数
import secrets
token = secrets.token_urlsafe(32)
```

### 低危：assert 用于安全检查

```python
# ❌ 生产环境 -O 会跳过 assert
assert user.is_admin, "Not admin"

# ✅ 正式检查
if not user.is_admin:
    raise PermissionError("Not admin")
```

---

## 安全扫描工具速查

### 依赖漏洞扫描

```bash
# pip-audit（推荐）
pip-audit
pip-audit --fix  # 自动升级修复

# safety
safety check

# pip 自带
pip install pip-audit && pip-audit
```

### 静态安全分析（SAST）

```bash
# bandit（Python 专用，推荐）
bandit -r src/
bandit -r src/ -ll  # 只显示中高风险

# semgrep（多语言通用）
semgrep --config=auto
semgrep --config=p/python
```

### 密钥泄露检测

```bash
# gitleaks
gitleaks detect

# trufflehog
trufflehog git file://.
```

### 执行顺序

```
触发安全审查
  │
  ├─ ❶ 依赖漏洞扫描（pip-audit）
  │     └─ 存在高危漏洞 → 阻断，先修复
  │
  ├─ ❷ 静态安全分析（bandit -r src/）
  │     └─ 存在安全问题 → 评估严重性后决定
  │
  ├─ ❸ 密钥泄露检测（gitleaks detect）
  │     └─ 存在泄露 → 阻断，立即轮换
  │
  └─ ❹ 全部通过 → 安全审查通过
```

---

## 安全审查触发条件

以下场景必须执行安全审查：
- 新增认证/授权逻辑
- 处理支付或金融数据
- 修改数据访问权限
- 引入新的外部服务集成
- 修改加密/哈希相关代码
- 处理用户上传内容
- 使用 `pickle` / `eval` / `exec` / `subprocess`
- 修改 YAML/XML 解析逻辑
- 处理 JWT / session token
