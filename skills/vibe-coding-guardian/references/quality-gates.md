# 质量门禁参考

加载条件：提交前验证、测试策略制定、契约检查、覆盖率分析

---

## 五阶段验证模型

```
阶段一：规则前置 ──── 定义变更如何被证明安全
    ↓
阶段二：基础门槛 ──── 编译 · lint · 静态检查 · 单测
    ↓
阶段三：风险分流 ──── 识别高风险区域，决定验证深度
    ↓
阶段四：深度验证 ──── API 一致性 · 安全扫描 · E2E
    ↓
阶段五：决策闭环 ──── 硬门禁 + 评分 + 人工升级
```

越靠左，问题越确定，越便宜，越适合自动阻断。
越往右，问题越依赖语义和上下文，越需要人的专业介入。

---

## 测试分层

| 层级 | 建议占比 | 关注点 | 原则 |
|------|----------|--------|------|
| 单元测试 | 70% | 逻辑正确性 | 测试行为不测实现，每个测试只验证一件事 |
| 集成测试 | 20% | 契约一致性 | 验证模块间契约，不依赖外部真实调用 |
| 端到端测试 | 10% | 流程完整性 | 覆盖关键场景 10-15 个，稳定可重复 |

---

## 覆盖率 vs 验证

覆盖率是输入，不是结论。它无法回答：
- 负向路径是否被验证？
- 契约字段是否漂移？
- 跨层不变量是否被破坏？

**目标不是覆盖率数字，而是"变更后核心行为可被验证"。**

---

## 完成条件清单

### 必须项（硬门禁）

- [ ] 正向路径：核心功能已验证
- [ ] 负向路径：错误和异常已处理
- [ ] 无回归：已有功能未受影响
- [ ] 编译通过：无类型错误、无 lint 错误

### 条件项（风险驱动）

- [ ] 边界条件：极端输入已测试（🟡中/🔴高风险）
- [ ] 契约一致：接口未破坏兼容性（🟡中/🔴高风险）
- [ ] 安全审查：无漏洞引入（🔴高风险）
- [ ] 性能基线：无明显退化（🟡中/🔴高风险）

### 证据项（可追溯）

- [ ] 验证结果有记录
- [ ] 关键决策有文档
- [ ] 已知限制有标注

---

## 证据状态模型

| 状态 | 含义 |
|------|------|
| VERIFIED | 已验证，有证据支撑 |
| TODO | 待验证，尚未执行 |
| BLOCKED | 被阻塞，需外部依赖解决 |
| SKIPPED | 跳过，附理由 |

"应该已经测过了" ≠ VERIFIED。没有证据的验证等于未验证。

---

## 门禁升级规则

| 信号 | 动作 |
|------|------|
| 测试全绿但改动触及高风险目录 | 升级到深度验证 |
| diff 扩散超出单模块 | 触发人工审查 |
| 契约字段变更 | 触发集成测试 + 人工确认 |
| 新增外部依赖 | 触发安全扫描 + 架构审查 |
| 覆盖率下降 | 阻断提交，要求补充测试 |

---

## Pre-commit Hook（提交前阻断）

在本地提交前自动运行质量检查，拦截不合格代码：

```bash
# .pre-commit-config.yaml 示例
repos:
  - repo: local
    hooks:
      - id: lint
        entry: eslint --max-warnings=0
        language: system
        types: [javascript, typescript]
      - id: type-check
        entry: tsc --noEmit
        language: system
        types: [typescript]
      - id: test-related
        entry: bash -c 'git diff --cached --name-only | xargs -I{} dirname {} | sort -u | xargs -I{} sh -c "cd {} && test -f *.test.* && npm test -- --bail || true"'
        language: system
```

Python 项目示例：

```yaml
repos:
  - repo: local
    hooks:
      - id: lint
        entry: ruff check --fix
        language: system
        types: [python]
      - id: type-check
        entry: mypy --strict
        language: system
        types: [python]
      - id: test-related
        entry: bash -c 'git diff --cached --name-only | xargs -I{} dirname {} | sort -u | xargs -I{} sh -c "cd {} && test -d tests && pytest --tb=short -q || true"'
        language: system
```

### 与 Vibe Coding Guardian 铁律的对应关系

| 铁律 | Pre-commit 检查 | 阻断级别 |
|------|----------------|----------|
| 铁律二：能跑 ≠ 完成 | 测试 + lint + type check 全部 exit 0 | 硬阻断 |
| 铁律三：验证结构 | 测试覆盖关键路径（非 getter/setter） | 软阻断（警告） |
| 铁律四：风险分级 | 安全敏感路径检测（grep auth/payment/crypto） | 条件阻断 |
