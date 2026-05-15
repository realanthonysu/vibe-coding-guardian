# vibe-coding-guardian

> Vibe Coding 的软件工程守护者 — 因为"能跑 ≠ 完成"。

[English](./README.md) | 中文

![Version](https://img.shields.io/badge/version-1.10.1-blue)
![License](https://img.shields.io/badge/license-Apache--2.0-green)
![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-brightgreen)

一个兼容 [Agent Skills](https://agentskills.io/specification) 规范的技能套件，在 AI 辅助编码中强制执行软件工程纪律。基于 Karpathy 的洞察：*思考可以外包，理解不能外包*。

## 技能一览

| 技能 | 说明 | 适用范围 |
|------|------|----------|
| **vibe-coding-guardian** | 核心工程纪律 — Karpathy 前置约束、五条铁律、双模式、9 项可计算信号 | 语言无关 |

`vibe-coding-guardian` 定义*验证什么*，适用于任何语言。


## 亮点（v1.10.1）

- **Context Engineering 工作流** — 4 步流程：收集上下文 → 综合 → 制定计划 → 执行（生成代码前必须完成）
- **可验证抽象层** — 在产品/测试/数据/代码层验证；高层能验证就不下沉到代码层
- **叶子节点策略** — 区分叶子节点/中间层/主干模块，匹配不同验证深度
- **9 项可计算信号** — 新增并发/竞态条件检测（社区最高频陷阱）
- **意图声明最佳实践** — 优质 vs 糟糕的意图声明对比示例
- **Karpathy 前置约束** — 在五条铁律之前落实极简优先、手术式修改、假设显性化
- **双运行模式** — 🚀 Prototype 追求速度，🏭 Production 追求安全
- **零外部依赖** — 只需要 git + 项目的 test/lint 命令即可运行
- **自动提醒** — 在 commit/deploy 前主动提醒从 Prototype 切换到 Production

---

## 目录

- [vibe-coding-guardian](#vibe-coding-guardian)
  - [亮点（v1.10.0）](#亮点v1100)
  - [技能一览](#技能一览)
  - [目录](#目录)
  - [为什么需要这个技能？](#为什么需要这个技能)
  - [五条铁律](#五条铁律)
  - [双运行模式](#双运行模式)
  - [渐进式加载](#渐进式加载)
  - [前置条件](#前置条件)
  - [安装](#安装)
    - [Skill CLI（推荐）](#skill-cli推荐)
    - [Claude Code](#claude-code)
    - [Trae](#trae)
    - [其他 Agent](#其他-agent)
  - [使用方式](#使用方式)
  - [文件结构](#文件结构)
  - [关键设计决策](#关键设计决策)
  - [兼容性](#兼容性)
  - [版本演进](#版本演进)
  - [许可证](#许可证)

---

## 为什么需要这个技能？

Vibe Coding 提升的是局部推进速度 — 代码生成得很快，测试全绿，终端不断输出成功。但局部速度 ≠ 系统可信度。真正的风险不在于代码不够快，而在于**"完成条件"开始漂移**。

这个技能让完成条件、验证证据和审查升级路径变得**可计算** — 让 AI 辅助编码不会悄悄积累熵。

## 五条铁律

| # | 规则 | 一句话 |
|---|------|--------|
| 1 | **先理解，再生成** | 无法用一句话说清意图，就不要急于写代码 |
| 2 | **能跑 ≠ 完成** | 完成 = 行为已验证 + 边界已覆盖 + 无回归 |
| 3 | **验证结构 > 验证数量** | 10 个覆盖关键路径的测试 > 100 个覆盖琐碎路径的测试 |
| 4 | **风险越高，验证越深** | 🟢 低 → 快速检查 · 🟡 中 → 标准验证 · 🔴 高 → 深度验证 + 人工确认 |
| 5 | **知道何时叫人** | 升级不是失败 — 是工程成熟度的体现 |

五条铁律之上，有三项 **Karpathy 前置约束**，专门防御 AI 编码的典型陷阱：

| 约束 | 防御的陷阱 | 一句话 |
|------|-----------|--------|
| **极简优先** | 过度设计 | 代码是负债 — 最小代码解决当前问题 |
| **手术式修改** | 越界副作用 | 只碰必须碰的代码；每行改动追溯到用户请求 |
| **假设显性化** | 未经验证的假设 | 明确列出关键假设；该 push back 时就 push back |

## 双运行模式

| | 🚀 Prototype（默认） | 🏭 Production |
|---|:---:|:---:|
| 理念 | 速度优先 — 拥抱指数级迭代 | 安全优先 — 带着信心上线 |
| 🟡 验证 | lint + 类型检查 + 单元测试 | + 集成测试 + 人工审查 |
| 🔴 验证 | + security.md（仅输入验证+认证授权） | + security.md（全部）+ 边界测试 + 人工确认 |
| 信号 5（安全路径） | 警告，不阻断 | 阻断并升级 |
| 完成条件 | lint + 类型检查 + 测试套件（或人工确认） | 铁律二完整自检 |

通过说 **"prototype"** / **"production"** / **"上线"** / **"ship it"** 切换模式。当你说 "commit" / "push" / "deploy" / "done" 时自动提醒切换。

## 渐进式加载

本技能采用三层加载设计，优化 token 效率：

```
第一层：元数据（~100 tokens）     — name + description，启动时加载
第二层：SKILL.md（~3000 tokens）  — 铁律 + 决策流程 + Gotchas，激活时加载
第三层：references/（按文件）      — 深度指南，仅在需要时加载
```

| 参考文件 | 何时加载 | 内容 |
|----------|----------|------|
| `references/architecture.md` | 新建模块、添加依赖、修改系统边界 | 模块边界、依赖管理、API 契约、分层架构 |
| `references/security.md` | 🔴 高风险任务（🏭 全部 / 🚀 仅输入验证+认证授权） | 输入验证、认证授权、数据保护、漏洞防护 |
| `references/quality-gates.md` | 提交前验证、测试策略、契约检查 | 五阶段验证、测试分层、证据状态、门禁升级 |
| `references/refactoring.md` | 代码腐化、技术债务、渐进式重构 | 重构模式、债务管理、代码气味、安全网 |
| `references/context-engineering.md` | 复杂代码库、多模块改动、AI 输出不符合业务约束 | 上下文收集框架、质量检查清单、常见陷阱、最佳实践 |
| `references/examples.md` | 需要理解 good vs bad 模式时 | 五条铁律的 ❌ Bad / ✅ Good 代码示例 |

## 前置条件

- **Claude Code**、**Trae** 或任何兼容 **Agent Skills 规范**的 Agent
- **Git**（可选 — 技能在没有 git 历史时也能工作，会自动降级）
- 建议项目已配置 **lint** 和 **type check** 命令

## 安装

### Skill CLI（推荐）

最快捷的安装方式：

```bash
npx skills add realanthonysu/vibe-coding-guardian
```

### Claude Code

**项目级**（跟随项目）：

```bash
mkdir -p .claude/skills
cp -r vibe-coding-guardian .claude/skills/
```

**用户级**（所有项目可用）：

```bash
mkdir -p ~/.claude/skills
cp -r vibe-coding-guardian ~/.claude/skills/
```

### Trae

1. 将 `vibe-coding-guardian/` 文件夹打包为 zip
2. 打开 Trae → **设置** → **规则与技能** → **导入**
3. 上传 zip 文件
4. 确认并刷新技能列表

或在 **SOLO 模式**下通过对话创建：

> "帮我创建一个 vibe-coding-guardian 技能"

### 其他 Agent

本技能遵循 [Agent Skills 规范](https://agentskills.io/specification)。将 `vibe-coding-guardian/` 文件夹放到你的 Agent 技能目录中即可（如 `.agents/skills/`）。

## 使用方式

无需手动激活 — 技能会在任何代码生成、修改或重构任务时自动激活。

```
你：帮我写一个用户注册接口

AI：[自动激活 vibe-coding-guardian]

需要确认几点再动手：
1. 注册需要哪些字段？
2. 密码强度有什么要求？
3. 是否需要邮箱验证？
4. 预计多少用户？

你：邮箱+密码，8位以上含特殊字符，需要邮箱验证

AI：[🟡 中风险：新功能 + 数据写入]
[生成代码 + 输入验证 + 重复检测 + 错误处理]
[完成条件自检：✅ 正向路径 ✅ 负向路径 ✅ 无回归 ✅ 契约一致]
```

## 文件结构

```
vibe-coding-guardian/               # 语言无关的核心技能
├── SKILL.md                        # 核心技能（frontmatter + 铁律 + 决策流程）
├── README.md                       # 英文说明
├── README_cn.md                    # 中文说明
├── CHANGELOG.md                    # 版本变更记录
├── LICENSE                         # Apache-2.0
└── references/
    ├── architecture.md             # 架构守护（按需加载）
    ├── security.md                 # 安全护栏（按需加载）
    ├── quality-gates.md            # 质量门禁（按需加载）
    ├── refactoring.md              # 重构治理（按需加载）
    ├── context-engineering.md      # 上下文工程指南（按需加载，v1.10.0 新增）
    └── examples.md                 # 正反示例（按需加载）
```

> Python 专用技能（`vibe-coding-pyguardian`、`vibe-coding-pyinit`）已迁移至 [python-hunter](https://github.com/realanthonysu/python-hunter)。

## 关键设计决策

| 决策 | 原因 |
|------|------|
| 默认 🚀 Prototype | 尊重 Vibe Coding 速度优先的理念 |
| 9 项可计算升级信号 | 用可检测的模式替代"直觉不安"；信号 9 捕获并发陷阱 |
| git diff 可选，仅向上修正 | 无 git 历史时也能工作；只能升级风险，不能降级 |
| 🚀 模式部分加载 security.md | 即使是原型，触及认证也需要输入验证 |
| 生成代码前先做 Context Engineering | 模型能力不是瓶颈，上下文质量才是 |
| 叶子节点差异化验证 | 不是所有代码都值得同等深度的验证 |
| commit/push/deploy 时自动提醒 | 在"原型→生产"漂移上线前拦截 |

## 兼容性

| Agent | 状态 | 备注 |
|-------|------|------|
| Claude Code | ✅ 已测试 | 支持项目级和用户级安装 |
| Trae | ✅ 已测试 | 支持 zip 导入和 SOLO 模式 |
| 其他兼容 Agent Skills 的 Agent | ✅ 兼容 | 遵循 Agent Skills 规范即可 |

## 版本演进

### vibe-coding-guardian（v1.10）

| 版本 | 关键变更 |
|------|----------|
| v1.0.0 | 初始设计 — 五条铁律 + 渐进式加载 |
| v1.1.0 | CSO description + "When to Activate" 激活规则 + 工具映射表 |
| v1.2.0 | 五条铁律全部增加 Gate Function + 6 步决策流程 + 5 项可计算升级信号 |
| v1.3.0 | 移除外部依赖（superpowers、code-review-graph）— 纯 git + grep 实现 |
| v1.4.0 | 人工评审 — 风险分级化理解确认、git diff 时序修正 |
| v1.5.0 | 双运行模式（🚀 Prototype / 🏭 Production） |
| v1.6.0 | 内部一致性 — 统一关键词、git diff 仅向上修正、双模式工具表 |
| v1.7.0 | 二次 AI 评审 — security.md 部分加载、无测试降级、信号 5 警告不阻断、自动升级提醒 |
| v1.8.0 | 铁律四双模式表格、已知风险标注、"done/finished"触发词 |
| v1.9.0 | Karpathy 前置约束（极简 / 手术式 / 假设显性化）、8 项信号（+3）、幻影成功防御、反模式扩展 |
| v1.9.1 | 约束→铁律映射表、执行时机说明、信号 6-8 正反示例、内部一致性修正 |
| v1.10.0 | Context Engineering 工作流、可验证抽象层、叶子节点策略、信号 9（并发安全）、意图声明最佳实践、context-engineering.md |
| v1.10.1 | 统一 Context vs Prompt 对比表、信号 9 增加 grep 检测示例 |

详细变更记录见 [CHANGELOG.md](./CHANGELOG.md)。



## 许可证

[Apache-2.0](./LICENSE)
