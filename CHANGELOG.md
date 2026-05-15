# Changelog

> testsmith 和 testweaver 已迁移至独立项目 [vibe-testing](../vibe-testing/)。

## vibe-coding-guardian v1.10.1

- 统一 SKILL.md 与 context-engineering.md 的 Context vs Prompt 对比表（6 行完整版本）
- 铁律五信号 9 增加快速检测 grep 示例（共享路径 / 全局状态 / 无锁写入）

## vibe-coding-guardian v1.10.0

- 新增"Context Engineering 前置工作流"章节（来源：Erik Schluntz, Anthropic）
  - 4 步流程：Context Gathering → Context Synthesis → Plan Formulation → Execution
  - Context Engineering vs Prompt Engineering 对比表
  - 上下文检查清单（6 项）
- 新增 context-engineering.md 参考文件
  - 上下文收集框架、质量检查清单、常见陷阱、多层上下文管理
  - 精准修正模板、上下文沉淀最佳实践
- 铁律二增加"可验证抽象层"方法论（来源：Erik Schluntz）
  - 产品层/测试层/数据层/代码层四级抽象
  - 执行规则：高层验证优先，异常时下钻
- 铁律四增加"叶子节点策略"（来源：Erik Schluntz）
  - 叶子节点/中间层/主干模块分类及差异化验证策略
- 铁律一 Gate Function 增加叶子节点判断步骤
- 铁律一增加"意图声明最佳实践"对比表（来源：社区共识）
- 铁律五升级信号新增信号 9：并发安全隐患（来源：知乎/CSDN 避坑指南）
- 反模式表新增：并发盲区、安全幻觉、上下文断裂 3 个反模式
- 工具映射表铁律五信号数从 8 更新为 9
- 按需加载指南新增 context-engineering.md

## vibe-coding-guardian v1.9.1

- 修复铁律四表格 Prototype 🔴 行与决策流程图的一致性
- Karpathy 约束增加执行时机说明（每个约束标注关联的铁律/信号和检查时间点）
- 工具映射表增加 Karpathy 约束执行方式说明
- 核心工程原则章节增加约束→铁律/信号映射表
- examples.md 新增信号 6（无关改动）、信号 7（假设清单缺失）、信号 8（代码膨胀）正反示例

## vibe-coding-pyguardian v1.1.0

- ty 降级为可选（Beta）— 标注 typing spec 合规率 ~53%，建议作为 mypy 补充而非替代
- 安全扫描栈增加 Semgrep — 推荐 bandit + semgrep 组合使用，覆盖 OWASP Top 10
- 高风险关键词增加 torch.load() / joblib.load() / shelve.open()（ML 反序列化风险）
- security.md 新增"ML 模型反序列化"防护章节（weights_only=True / safetensors）
- Gotchas 增加 Astral 被 OpenAI 收购的生态风险提示

## vibe-coding-pyinit v0.2.0

- Python 版本参数化 — 不再硬编码 3.12，支持用户指定目标版本（默认 3.12）
- prek hook 版本号改为占位符 — 生成时必须查询最新版本，禁止使用过时硬编码版本
- library 项目类型增加 py.typed 标记文件（PEP 561），hatchling 配置同步更新
- Gotchas 增加 Astral 被 OpenAI 收购的生态风险提示

## vibe-coding-guardian v1.9.0

- 新增"核心工程原则"章节（Karpathy 前置约束）：极简优先、手术式修改、假设显性化
- 铁律一 Gate Function 扩展：🔴 高风险模板增加 ASSUMPTIONS、ALTERNATIVES、SUCCESS CRITERIA
- 铁律二 checklist 扩展：增加"目标对齐"项，防御幻影成功
- 铁律五信号扩展：新增信号 6（无关改动）、信号 7（假设清单缺失）、信号 8（代码膨胀）
- 反模式表扩展：新增过度设计、范围蔓延、善意重构、幻影成功 4 个反模式
- 决策流程同步更新：信号检测从 5 项扩展为 8 项
- examples.md 新增：极简优先、手术式修改、假设显性化、幻影成功正反示例

## vibe-coding-guardian v1.8.0

- 铁律四验证深度表改为 4 列，区分 Prototype / Production 模式
- Gotchas 新增"人工确认核心行为"弱约束的已知风险标注
- 自动升级提醒新增 "完成" / "done" / "finished" / "搞定了" 触发词
- 修复铁律四表格与决策流程、模式对比表的内部不一致

## v1.7.0

- Prototype 🔴 高风险改为加载 security.md 的"输入验证"+"认证授权"章节（替代未定义的"安全清单自检"）
- 升级信号 5 从"跳过"改为"警告但不阻断"
- 完成条件改为"lint + type check + 测试套件（或人工确认核心行为）"
- 新增"自动升级提醒"章节：commit / push / deploy 等操作时主动提醒切换 Production
- security.md 加载条件与 SKILL.md 同步
- 工具映射表 Prototype 铁律二工具描述细化
- Gotchas 新增关联测试定义

## v1.6.0

- 引入双运行模式：🚀 Prototype（默认）/ 🏭 Production
- 新增模式对比表
- 决策流程增加"确定运行模式"步骤
- 工具映射表增加 Prototype / Production 双列

## v1.5.0

- 决策流程 ❸ 增加 git diff 只能向上修正风险等级的约束
- 铁律一 Gate Function 改为风险分级化（🟢 一句话 / 🟡 WHAT+CONSTRAINTS / 🔴 完整模板）

## v1.4.0

- 铁律一 Gate Function 按风险等级递进，低风险不再强制完整模板
- 决策流程改为两阶段风险判断（关键词预判 + git diff 辅助验证）
- 所有 git 命令增加"无 git 历史则跳过/手动比对"降级处理

## v1.3.0

- 去掉对 superpowers 插件的依赖
- 去掉 code-review-graph MCP 工具依赖，改用 `git diff` + `grep`
- 轻量化 CI/CD：删除 GitHub Actions，只保留 pre-commit hook
- 铁律五升级信号全部用通用 git 命令实现

## v1.2.0

- 5 条铁律全部增加 Gate Function（可执行的 step-by-step 协议）
- 决策流程重写为 6 步 Gate Function 链
- 铁律一增加 WHAT / WHY / CONSTRAINTS 强制模板
- 铁律五用 5 项可计算信号替代"直觉不安"
- security.md 增加安全扫描工具速查（npm audit / semgrep / gitleaks 等）
- quality-gates.md 增加 CI/CD 集成（pre-commit hook + GitHub Actions）
- 铁律三增加"关键业务规则"判定规则

## v1.1.0

- description 重写为 CSO 规范（Use when... 只写触发条件）
- 新增 "When to Activate" 章节（MUST / SHOULD / SKIP 三级判定）
- 新增与 superpowers 工作流集成图
- 决策流程增加工具辅助节点
- 新增工具映射表

## v1.0.0

- 初始版本
- 五条铁律：先理解再生成 / 能跑≠完成 / 验证结构>数量 / 风险越高验证越深 / 知道何时叫人
- 决策流程（5 步）
- 反模式警示表
- 5 个参考文件：architecture / security / quality-gates / refactoring / examples
