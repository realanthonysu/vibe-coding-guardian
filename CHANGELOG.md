# Changelog

## v1.8.0

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
