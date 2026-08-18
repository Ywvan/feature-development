---
name: feature-review
description: 用于对已完成或部分完成的 Feature 做独立 Review。基于 Feature Context / Confirmed Requirements、Implementation Handoff、Final Diff、Relevant Current Repo、验证证据和重要设计偏差，重点检查 Requirement Correctness、Scenario Coverage、Integration Correctness，并给出 READY 或 NOT READY 最终门禁。默认只 Review，不修改生产代码。
---

# Feature Review

## 顶层目标与独立性

始终以“正确完成 Feature Context 定义的需求”为顶层目标。Review 的目标不是制造或清零 Finding，而是独立判断实现是否正确且完整地满足需求。

优先在 Fresh Review Context 中从以下输入重新得出结论：

```text
FEATURE_CONTEXT / Confirmed Requirements
+ IMPLEMENTATION_HANDOFF
+ Final Diff
+ Relevant Current Repo
+ Build / Test / Verification Evidence
+ Important Design Deviations
```

Fresh Review Context 是推荐的独立性边界，不是必须创建的 Session 或运行时对象。Review 不应继承 Implementation Agent 的完整推理、Debug 历史、Tool 调用历史或实现过程中已废弃的方案。

不得因为 Implementation Summary、Technical Design、Implementation Handoff、历史实现过程或开发者解释而默认实现正确。可将这些信息作为调查线索，但必须用 Feature Context、Final Diff、Relevant Current Repo 和验证证据重新验证。

本 Skill 默认只读 Review，不修改生产代码、配置、数据库或业务仓库文件，不自动编排 Subagent，不自动进入 Review → Fix 循环。用户明确要求修复时，将 Findings 交给 `feature-implement` 的 Review Fix Mode；是否使用 Fresh Implementation Context 由问题规模和当前 Context 状况决定。

## 事实与范围

按以下优先级判断：

1. Feature Context / Confirmed Requirements 是最高业务事实源。
2. 原始 PRD、原型、接口文档用于提供需求证据；与 Feature Context 冲突时只报告冲突，不自行取舍。
3. 最终 diff 与相关当前代码是实现和 Current State 的事实源。
4. Build、Test 和其他 Verification Evidence 只证明其实际覆盖的层级，不得扩大解释。
5. Technical Design、Implementation Handoff、Important Design Deviations 和 Review Finding 都是待验证信息。

先确定 Review 基线、目标 diff、Feature Scope 和 Out of Scope。检查工作树、提交范围或用户指定 diff，防止遗漏新增文件、跨模块改动或未纳入 diff 的相关实现。允许扩大只读调查范围以完成调用链和需求验收，但只报告有真实证据且与本次 diff 或未实现需求直接相关的问题。

## Review 流程

### 1. 重建需求检查表

从 Feature Context / Confirmed Requirements 独立提取：

- Feature Goal
- Confirmed Requirements
- Global Constraints
- Target State
- Acceptance Criteria
- Out of Scope
- 必须覆盖的业务场景和已确认边界

若关键业务事实缺失或互相冲突，标为 NOT VERIFIED；不得自行发明验收标准。

### 2. 理解最终实现

完整阅读目标 diff，并按必要程度检查相关入口、调用链、数据流、数据库、SQL、事务、缓存、RPC、MQ、Job、异步流程、外部接口、权限、配置和测试。不要仅凭局部 diff、方法名或开发者说明下结论。

### 3. Code Correctness 边界

使用当前环境已生效的工程 Review Guardrails 检查 Code Correctness，并将确认的代码风险作为 Feature Final Gate 的输入。本 Skill 不重复维护完整 SQL Rule、Logging Rule、Comment Rule、P0～P3 定义、通用 Code Review 方法论或通用 Finding Format。

如果没有额外工程 Guardrail，仍需识别会直接阻止 Feature Goal、Requirement 或集成行为成立的代码事实，但不要在本 Skill 内扩展一套平行的通用 Review 体系。

### 4. Requirement Correctness

重新检查并逐项映射：

- **Feature Goal**：业务目标是否真正完成，而非仅完成了代码 Task。
- **Confirmed Requirements**：每条确认需求是否在正确路径和条件下落地。
- **Target State**：目标行为是否由最终代码实际实现，而非只在说明中声称完成。
- **Out of Scope**：是否误改本期明确不应改变的行为。
- **Over-Implementation**：是否增加需求未要求的行为或扩大影响范围。
- **Design Deviation**：Technical Decision 是否出现未解释或未验证的偏离。
- **Completion Integrity**：是否存在“代码 Task 已完成，但业务 Requirement 未完成”。

建立需求项到代码证据、测试证据和结论的映射。缺少证据时使用 NOT VERIFIED，不将推测写成 PASS。

### 5. Scenario 与 Integration Correctness

检查关键正常、异常、边界与兼容场景，并重点检查跨 Workstream 组合后的行为：

- 各 Workstream 单独正确但组合错误；
- 跨服务 Contract 与端到端数据流；
- Transaction Boundary 与 Async Boundary；
- Compatibility、Exception Path 与 Regression Risk；
- 测试和验证是否覆盖 Feature-level Acceptance Criteria，且证据层级是否被准确陈述。

## NOT READY Findings

整体结论为 NOT READY 时，对每个阻塞项输出：

- Finding
- Relevant Requirement
- Expected Behavior
- Affected Area
- Required Fix
- Acceptance Criteria
- Verification Expectation

Finding 必须有真实代码或验证证据，并可直接作为下一轮 `feature-implement` Review Fix Mode 的输入。Review Finding 仍是待实现阶段复核的判断；若历史 Finding 不成立或与 Feature Context 冲突，明确说明并不建议修改。Required Fix 只描述恢复需求正确性所需的最小方向，不将无关重构包装成修复。

不要新增 `feature-rework`、REWORK_TASK Runtime 或 Review Fix Controller。修复完成后再次进入 Independent Review。

## Final Requirement Gate

Review 结束时单独进行最终需求验收，对以下各项只使用 `PASS`、`FAIL` 或 `NOT VERIFIED`：

- Feature Goal
- Confirmed Requirements
- Target State
- Scenario Coverage
- Integration Correctness
- Out of Scope
- Important Design Deviations
- Validation / Test Coverage

为每项给出简短证据或阻塞原因。最后整体状态只能是：

```text
# READY
```

或：

```text
# NOT READY
```

仅当不存在阻塞性 Finding，且完成 Feature 验收所必需的项目均为 PASS 时输出 READY。存在 FAIL 或需求完成所必需的 NOT VERIFIED 时输出 NOT READY，并列出阻止 Feature 完成的具体 blocker。不得因“没有发现代码 Bug”而跳过 Requirement Validation。
