---
name: vibe-coding-guardian
description: >
  Use when: (1) starting any non-trivial coding task to assess risk level before
  implementation; (2) about to claim work is complete — verify completion conditions
  with evidence; (3) touching security-sensitive code (auth, payments, encryption,
  data migration); (4) diff impact radius is unclear and blast analysis needed;
  (5) deciding whether a change requires human review. Also triggers on: risk
  assessment, blast radius, escalation, "should I proceed", 风险评估, 升级判断,
  完成条件, 安全审查, entropy drift, green-light illusion.
license: Apache-2.0
metadata:
  author: vibe-coding-guardian
  version: "1.9.1"
  paradigm: vibe-coding-guardian
  core-thesis: "thinking can be outsourced, understanding cannot — Karpathy"
---

# Vibe Coding Guardian

> 思考可外包，理解不可外包。 — Karpathy

Vibe Coding 提升的是局部推进速度，不会自动提升系统整体可信度。
本技能的使命：**让完成条件、验证证据和审查升级变得可计算。**

---

## 运行模式

Vibe Coding Guardian 有两种运行模式，根据项目阶段选择：

### 🚀 Prototype 模式（默认）

适用于：快速原型、MVP、概念验证、探索性开发

- 🟢 低风险 → 一句话意图，直接生成
- 🟡 中风险 → WHAT + CONSTRAINTS，**不触发安全审查**
- 🔴 高风险 → 完整理解模板 + 用户确认，**加载 security.md 但只执行"输入验证"和"认证授权"章节**
- 升级信号检测：信号 5 改为警告（不阻断）
- 完成条件：lint + type check + 测试套件（或人工确认核心行为）

### 🏭 Production 模式

适用于：正式产品、上线前审查、安全敏感项目

- 🟢 低风险 → 一句话意图，直接生成
- 🟡 中风险 → WHAT + CONSTRAINTS + 人工审查
- 🔴 高风险 → 完整理解模板 + 用户确认 + **加载 security.md** + 安全审查 + 边界测试
- 升级信号检测：5 项信号全部执行
- 完成条件：铁律二完整自检（正向 + 负向 + 回归 + 契约）

### 模式切换

- 用户说 "prototype" / "原型" / "MVP" / "just prototyping" → 切换到 🚀 Prototype
- 用户说 "production" / "生产" / "上线" / "ship it" → 切换到 🏭 Production
- 默认为 🚀 Prototype（尊重 Vibe Coding 的速度优先）
- **切换到 Production 时，建议对已有代码做一次回溯审查**

### 自动升级提醒

当用户在 🚀 Prototype 模式下执行以下操作时，主动提醒是否切换到 🏭 Production：

- "提交" / "commit" / "push" / "merge" / "PR" / "pull request"
- "部署" / "deploy" / "release" / "发布"
- "完成" / "done" / "finished" / "搞定了"

提醒话术：`"你正在 Prototype 模式下准备提交/部署。建议切换到 Production 模式进行完整审查。是否切换？"`

用户选择不切换 → 继续 Prototype 模式，不强制。

### 两种模式对比

| 环节 | 🚀 Prototype | 🏭 Production |
|------|:-----------:|:------------:|
| 🟢 理解确认 | 一句话意图 | 一句话意图 |
| 🟡 理解确认 | WHAT + CONSTRAINTS | WHAT + CONSTRAINTS + 人工审查 |
| 🔴 理解确认 | 完整模板 + 确认 | 完整模板 + 确认 + security.md |
| 🟡 验证深度 | lint + type check + 关联测试 | + 单元测试 + 集成测试 |
| 🔴 验证深度 | + security.md（仅输入验证+认证授权） | + security.md（全部章节）+ 边界测试 + 人工确认 |
| 升级信号 5（安全路径） | 警告但不阻断 | 阻断升级 |
| 完成条件 | lint + type check + 测试套件（或人工确认核心行为） | 铁律二完整自检 |

---

## 核心工程原则（Karpathy 前置约束）

五条铁律之前，先落实三项 Karpathy 核心约束。它们不是可选项，是后续所有 Gate Function 的前置条件。

| 约束 | 主要关联 | 执行时机 |
|------|---------|---------|
| 极简优先 | 铁律四（风险评估）、信号 8（代码膨胀） | 生成前 + 完成自检时 |
| 手术式修改 | 信号 6（无关改动）、信号 8（代码膨胀） | 生成时 + 信号检测时 |
| 假设显性化 | 铁律一 🔴模板（ASSUMPTIONS）、信号 7（假设清单缺失） | 理解确认时 + 信号检测时 |

### 约束一：极简优先

> 代码是负债，不是资产。Minimum code that solves the problem. Nothing speculative.

- **不实现未请求的功能**。用户说"加个搜索框"，不要顺手加上高级筛选、导出 CSV、权限控制。
- **不为单次使用创建抽象**。三处重复再提取，不是一处就提取。
- **不为不可能场景添加错误处理**。`if user is None` 如果上游已保证非空，不要加防御性判断。
- **复杂度超标时主动提出简化**。如果任务描述一句话，实现却超过 3 倍复杂度 → 停下来，给出更简单的方案请用户选择。

**自检问句**：`Would a senior engineer say this is overcomplicated?` 如果是，就简化。

**执行时机**：生成代码前 + 完成条件自检时 + 信号 8（代码膨胀）检测时

### 约束二：手术式修改

> Touch only what you must. Clean up only your own mess.

- **只修改与用户请求直接相关的代码**。不改相邻文件的格式、注释、命名风格。
- **不重构没坏的东西**。发现无关的 dead code → 提一句，不要顺手删掉。
- **匹配现有风格**，即使你个人偏好不同。
- **清理自己产生的 orphan**：你的改动导致某些 import / 变量 / 函数不再被使用 → 清理掉。但**不清理已存在的 dead code**。

**测试标准**：Every changed line should trace directly to the user's request.

**执行时机**：生成代码时 + 信号 6（无关改动）检测时 + 信号 8（代码膨胀）检测时

### 约束三：假设显性化

> State your assumptions explicitly. If uncertain, ask.

- **列出关键假设**：schema 结构、数据范围、并发模型、第三方行为 —— 凡是"我觉得应该是这样"的地方，都要列出来请用户确认。
- **存在多种解释时，全部列出**，不要默默选一个最复杂的。
- **发现更简单的方案时，主动提出**。Push back when warranted.
- **遇到模糊指令时停止**，指出哪里不清楚，要求澄清。

**执行时机**：理解确认时（铁律一 Gate Function）+ 信号 7（假设清单缺失）检测时

---

## When to Activate

### MUST activate (non-negotiable)

- Task involves **auth, payments, encryption, data migration** → 铁律四 + 铁律五
- About to claim "done", "complete", "fixed", "working" → 铁律二
- User asks "is this safe to ship?" / "can I merge?" → 铁律二 + 铁律四
- Diff touches **3+ files across 2+ modules** → 铁律四
- New external dependency being introduced → [architecture.md](references/architecture.md)

### SHOULD activate (recommended)

- Any new feature implementation → 铁律一（理解优先）
- Refactoring existing code → [refactoring.md](references/refactoring.md)
- Test count growing but confidence not increasing → 铁律三

### SKIP (do not activate)

- Pure read/exploration tasks (no code changes)
- Simple typo/formatting fixes
- Configuration changes (non-security)
- User explicitly says "skip guardian" or "just do it"

---

## 五条铁律

### 铁律一：先理解，再生成

生成代码之前，必须先明确三件事：

- **做什么** — 这段代码要解决什么问题？
- **为什么** — 为什么是这个方案而不是其他？
- **边界在哪** — 输入范围、错误情况、性能约束是什么？

无法用一句话说清意图 → 理解不够 → 不要急于写代码。

#### Gate Function：理解确认（风险分级）

理解确认的严格程度与风险等级联动，而非一刀切：

```
收到编码任务
  │
  ├─ ❶ 快速风险预判（基于任务描述，非 git diff）：
  │     触及 auth/payment/crypto/session/token/data-migration → 🔴 高风险
  │     新功能 / API 变更 / 跨模块改动       → 🟡 中风险
  │     UI 文案 / 样式微调 / 非破坏性新增     → 🟢 低风险
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
  │       ASSUMPTIONS:  [关键假设清单 — schema / 数据范围 / 并发模型 / 第三方行为]
  │       ALTERNATIVES: [考虑过但放弃的方案及原因 — 不只有一种解释时必填]
  │       SUCCESS CRITERIA: [如何验证完成 — 可量化的、可自动验证的标准]
  │       → 呈现给用户，等待确认
  │       → 用户修正 → 更新模板 → 重新确认
  │       → 用户未确认 → 不动手写代码
  │
  └─ ❸ 确认通过 → 进入生成阶段
```

**铁律：** 风险越高，理解确认越严格。🔴 高风险没有用户确认不生成代码；🟢 低风险一句话即可。

### 铁律二：能跑 ≠ 完成

完成 = 行为已验证 + 边界已覆盖 + 无回归

提交前自检：
- [ ] 正向路径：核心功能按预期工作？
- [ ] 负向路径：错误输入、异常状态已处理？
- [ ] 回归风险：改动不破坏已有功能？
- [ ] 契约一致：接口契约与调用方期望一致？
- [ ] 目标对齐：重新阅读原始请求，确认解决方案确实解决了用户的问题（非"代码运行了但答非所问"的幻影成功）

### 铁律三：验证结构 > 验证数量

10 个覆盖关键路径的测试 > 100 个覆盖琐碎路径的测试。

| 层级 | 验证目标 | 典型手段 |
|------|----------|----------|
| 单元层 | 逻辑正确性 | 纯函数测试、业务规则测试 |
| 集成层 | 契约一致性 | API 接口测试、模块边界测试 |
| 端到端层 | 流程完整性 | 用户场景测试、跨系统交互测试 |

用 **"关键行为是否可被验证"** 衡量信心，不用测试数量。

#### 判定规则：什么是"关键业务规则"

以下任一条件满足 → 该业务规则是"关键的"，必须有测试覆盖：

| 条件 | 示例 |
|------|------|
| 涉及金钱/财务 | 支付金额计算、退款逻辑、费率转换 |
| 涉及权限/安全 | 用户角色判断、资源访问控制、认证状态检查 |
| 涉及数据完整性 | 唯一性约束、外键关系、事务一致性 |
| 涉及状态机转换 | 订单状态流转、审批流程、生命周期管理 |
| 涉及外部契约 | API 响应格式、事件消息结构、第三方回调处理 |

**不属于关键业务规则的：** getter/setter、纯格式化函数、日志记录、UI 文案渲染。

### 铁律四：风险越高，验证越深

| 风险 | 典型场景 | 🚀 Prototype 验证深度 | 🏭 Production 验证深度 |
|------|----------|----------------------|----------------------|
| 🟢 低 | UI 文案、样式微调 | lint + 类型检查 + 关联测试 | lint + 类型检查 + 关联测试 |
| 🟡 中 | 新功能、API 变更 | 上述 + 单元测试 | 上述 + 单元测试 + 集成测试 + 人工审查 |
| 🔴 高 | 认证逻辑、支付流程、数据迁移 | 上述 + security.md（仅输入验证+认证授权）+ 人工确认 | 上述 + security.md（全部章节）+ 边界测试 + 人工确认 |

### 铁律五：知道何时叫人

以下情况必须升级到人工判断：
- 改动触及安全敏感区域（认证、授权、支付）
- diff 扩散半径超出预期
- 业务规则存在歧义
- 测试全绿但存在以下可计算信号（替代"直觉不安"）

#### Gate Function：升级信号检测

运行以下检查，任一命中 → 必须升级到人工：

```
信号 1：覆盖率下降
  → 运行测试覆盖率工具，对比改动前后
  → 新增代码覆盖率 < 60% → 升级

信号 2：影响面过大
  → git diff --stat HEAD~1 统计改动文件数（无 git 历史则跳过）
  → 改动文件 > 10 个，或跨 3+ 个顶层目录 → 升级

信号 3：新增 TODO/FIXME/HACK
  → git diff HEAD~1 | grep -E "^\+.*(TODO|FIXME|HACK)"（无 git 历史则 grep 当前文件）
  → 存在 → 升级（显式的"还没做完"标记）

信号 4：接口契约变更
  → git diff HEAD~1 检查公开接口变更（无 git 历史则手动比对）
  → 存在函数签名变更、返回类型变更、错误码变更 → 升级

信号 5：安全敏感路径触及
  → git diff --name-only HEAD~1 | grep -iE "(auth|payment|crypto|session|token)"
  → 无 git 历史则检查当前改动文件路径
  → 存在 → 升级

信号 6：无关改动混入
  → 检查 diff 中是否存在与用户请求无关的文件或代码变更
  → 无 git 历史则逐行审查当前改动
  → 存在无关改动 → 升级（要求解释或回滚，禁止"顺手改善"相邻代码）

信号 7：关键假设未确认
  → 🔴 高风险任务检查：是否已列出 ASSUMPTIONS 并获得用户确认
  → 未列出或用户未确认 → 升级（禁止在模糊假设上生成代码）

信号 8：代码膨胀
  → 评估新增代码行数与任务描述复杂度的比值
  → 一句话任务产生了超过 50 行实现代码，或 3 倍以上于必要复杂度的代码 → 警告（触发极简审查，确认是否存在过度设计）
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
  ├─ ❷ 理解确认（铁律一 Gate Function）
  │     → 快速风险预判（基于任务描述关键词）
  │     → 按风险等级执行对应深度的理解确认
  │     → 🟢 一句话意图 / 🟡 WHAT+CONSTRAINTS / 🔴 完整模板+用户确认
  │
  ├─ ❸ 风险等级评估（铁律四 Gate Function）
  │     → 步骤 A：基于任务描述关键词预判风险
  │         触及 auth/payment/crypto/session/token/data-migration → 🔴 高风险
  │         新功能 / API 变更 / 跨模块改动       → 🟡 中风险
  │         UI 文案 / 样式微调 / 非破坏性新增     → 🟢 低风险
  │     → 步骤 B：如已有 git 历史，用 git diff 辅助验证预判
  │         git diff --stat（统计改动规模，修正预判）
  │         改动文件 > 10 或跨 3+ 目录 → 升级风险等级
  │         ⚠️ git diff 只能向上修正（🟢→🟡→🔴），不能向下降级
  │         原因：关键词预判是主信号，git diff 是补充证据
  │         注意：首次提交或未 commit 时跳过此步，以预判为准
  │     → 步骤 C：根据模式 + 风险等级选择验证深度：
  │         🚀 Prototype:
  │           🟢 → lint + type check + 关联测试
  │           🟡 → 上述 + 单元测试（不触发安全审查）
  │           🔴 → 上述 + security.md（仅输入验证+认证授权章节）+ 人工确认
  │         🏭 Production:
  │           🟢 → lint + type check + 关联测试
  │           🟡 → 上述 + 单元测试 + 集成测试 + 人工审查
  │           🔴 → 上述 + 加载 security.md + 安全审查 + 边界测试 + 人工确认
  │
  ├─ ❹ 生成代码
  │
  ├─ ❺ 完成条件自检（铁律二 Gate Function）
  │     🚀 Prototype:
  │       → 运行 lint → 0 errors
  │       → 运行 type check → 0 errors
  │       → 有测试套件：运行测试套件 → exit code = 0
  │         无测试套件：人工确认核心行为（一句话描述"我验证了 XXX"）
  │       → 全部通过 → 进入升级信号检测
  │     🏭 Production:
  │       → 步骤 A：运行测试套件 → exit code = 0
  │       → 步骤 B：运行 lint → 0 errors
  │       → 步骤 C：运行 type check → 0 errors
  │       → 步骤 D：检查接口变更（优先 git diff，无 git 历史则手动比对）
  │       → 步骤 E：全部通过 → 进入升级信号检测
  │
  ├─ ❻ 升级信号检测（铁律五 Gate Function）
  │     🚀 Prototype: 执行信号 1-4（阻断），信号 5（警告但不阻断），信号 6-8（阻断）
  │     🏭 Production: 执行全部 8 项信号（阻断）
  │     → 任一命中 → 输出风险摘要，请用户确认
  │     → 全部未命中 → 完成
  │
  └─ ❼ 完成
```

---

## 反模式警示

| 反模式 | 表现 | 解药 |
|--------|------|------|
| 🚦 绿灯幻觉 | 测试全过，系统已坏 | 验证结构，不只看数量（铁律三） |
| 🔧 局部修补 | 修症状不修原因 | 追问"为什么"到根因（铁律一） |
| 🧠 上下文丢失 | 忘记为什么这样做 | 关键决策写进代码或文档 |
| 📦 依赖爆炸 | 随意引入新库 | 评估必要性、维护状态、替代方案 |
| 💧 抽象泄漏 | 绕过封装走捷径 | 遵守层间契约，不跨层访问 |
| 🏗️ 过度设计 | 为简单任务构建通用框架 | 约束一：不为单次使用创建抽象，三处重复再提取 |
| 📐 范围蔓延 | 实现 A 时顺手做了 B | 约束二：每行改动追溯到用户请求，不改相邻代码 |
| 🧹 善意重构 | 未经请求"改善"相邻代码、格式、注释 | 约束二：只清理自己产生的 orphan，不动已存在的 dead code |
| 🎯 幻影成功 | 代码运行了但没解决真实问题 | 铁律二 checklist + 约束三：SUCCESS CRITERIA 前置定义 |

---

## Gotchas

- "应该已经测过了" 不等于 VERIFIED。没有证据的验证等于未验证。
- 覆盖率是输入不是结论。负向路径、契约漂移、跨层不变量它都无法回答。
- Vibe Coding 的风险不在代码不够快，而在"完成条件"开始漂移。
- 局部信号增长 ≠ 系统整体可信度增长。编译通过、接口通了、覆盖率没掉，不代表"完成"。
- 不是所有风险都适合继续自动化下沉。有些改动最合理的动作是拉人进来。
- 🚀 Prototype 模式不等于"没有质量要求"——lint 和 type check 仍然必须通过。
- 原型代码有变成生产代码的倾向。切换到 🏭 Production 时，务必对已有代码做回溯审查。
- 关联测试 = 与改动文件直接相关的测试（通过文件路径或模块名匹配）。不是全量测试，也不是跳过测试。
- ⚠️ 已知风险：Prototype 无测试套件时的"人工确认核心行为"是弱约束——AI 无法验证用户声称的验证是否真实。这是速度与安全的折中，切换到 🏭 Production 时应补全测试。

---

## 工具映射：铁律 → Gate Function

> **Karpathy 约束通过对话和审查执行，不单独列行**：极简优先 → 生成前自检 + 信号 8；手术式修改 → 生成时约束 + 信号 6/8；假设显性化 → 铁律一 🔴模板 + 信号 7。

| 铁律 | Gate Function | 🚀 Prototype 工具 | 🏭 Production 工具 | 判定规则 |
|------|--------------|-------------------|-------------------|----------|
| 铁律一 | 理解确认（风险分级） | 对话 | 对话 | 🟢 一句话意图 / 🟡 WHAT+CONSTRAINTS / 🔴 完整模板+用户确认 |
| 铁律二 | 完成条件自检 | Bash（lint + type check + 测试套件或人工确认） | Bash（测试套件 + lint + type check + 契约检查） | 🚀 4 项通过 / 🏭 6 项通过 |
| 铁律三 | 验证结构审查 | Bash | Bash | 测试覆盖关键业务规则（金钱/权限/数据完整性），非 getter/setter |
| 铁律四 | 风险等级评估 | 任务描述关键词 + `git diff --stat`（可选，仅向上修正） | 任务描述关键词 + `git diff --stat`（可选，仅向上修正） | 关键词预判 + git diff 辅助（只能 🟢→🟡→🔴，不能降级） |
| 铁律五 | 升级信号检测 | `git diff` + `grep`（可选，信号 5 警告不阻断，信号 6-8 阻断） | `git diff` + `grep`（可选，全部 8 项阻断） | 任一命中 → 升级到人工 |

---

## 按需加载深度指南

以下参考文件仅在任务涉及对应领域时查阅，不要预先加载：

| 参考文件 | 何时加载 | 内容 |
|----------|----------|------|
| [references/architecture.md](references/architecture.md) | 新建模块、添加依赖、修改系统边界 | 模块边界、依赖管理、API 契约、分层架构 |
| [references/security.md](references/security.md) | 🔴 高风险任务（🏭 全部章节 / 🚀 仅输入验证+认证授权） | 输入验证、认证授权、数据保护、漏洞防护 |
| [references/quality-gates.md](references/quality-gates.md) | 提交前验证、测试策略、契约检查 | 五阶段验证、测试分层、证据状态、门禁升级 |
| [references/refactoring.md](references/refactoring.md) | 代码腐化、技术债务、渐进式重构 | 重构模式、债务管理、代码气味、安全网 |
| [references/examples.md](references/examples.md) | 需要理解 good vs bad 模式时 | 五条铁律 + Karpathy 约束 + 信号 6-8 + 幻影成功的正反示例 |
