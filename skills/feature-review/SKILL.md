---
name: feature-review
description: 用于对已完成或部分完成的 Feature 做独立 Review。基于 Feature Context、最终 diff 和相关当前代码，同时检查 Code Correctness 与 Requirement Correctness，验证 Scope、业务规则、Target State、Out of Scope 和场景覆盖。默认只 Review，不修改生产代码。
---

# Feature Review

## 顶层目标与独立性

始终以“正确完成 Feature Context 定义的需求”为顶层目标。Review 的目标不是制造或清零 Finding，而是独立判断实现是否正确且完整地满足需求。

主要从以下事实重新得出结论：

```text
Feature Context
+ Current Diff
+ Relevant Current Code
```

不得因为 Implementation Summary、Technical Design、历史实现过程或开发者解释而默认实现正确。可将这些信息作为调查线索，但必须用 Feature Context 和真实代码证据验证。

本 Skill 默认只读 Review，不修改生产代码、配置、数据库或业务仓库文件，不自动编排 Subagent，不自动进入 Review → Fix 循环。用户明确要求修复时，建议在新的 Chat 使用 `feature-implement` 的 Review Fix Mode。

## 事实与范围

按以下优先级判断：

1. Feature Context 是最高业务事实源。
2. 原始 PRD、原型、接口文档用于提供需求证据；与 Feature Context 冲突时只报告冲突，不自行取舍。
3. 最终 diff 与相关当前代码是实现和 Current State 的事实源。
4. Technical Design、Implementation Handoff 和 Review Finding 都是待验证信息。

先确定 Review 基线、目标 diff、Feature Scope 和 Out of Scope。检查工作树、提交范围或用户指定 diff，防止遗漏新增文件、跨模块改动或未纳入 diff 的相关实现。允许扩大只读调查范围以完成调用链和需求验收，但只报告有真实证据且与本次 diff 或未实现需求直接相关的问题。

## Review 流程

### 1. 重建需求检查表

从 Feature Context 独立提取：

- Feature Goal
- Scope
- Relevant Business Rules
- Target State
- Acceptance Criteria
- Out of Scope
- 必须覆盖的业务场景和已确认边界

若关键业务事实缺失或互相冲突，标为 NOT VERIFIED；不得自行发明验收标准。

### 2. 理解最终实现

完整阅读目标 diff，并按必要程度检查相关入口、调用链、数据流、数据库、SQL、事务、缓存、RPC、MQ、Job、异步流程、外部接口、权限、配置和测试。不要仅凭局部 diff、方法名或开发者说明下结论。

### 3. Part A：Code Correctness

检查与本次变更相关的真实风险，包括但不限于：

- 逻辑错误、分支遗漏、边界场景与 Null。
- 异常处理、事务、并发、幂等和数据一致性。
- 外部调用、MQ、Job、回调、异步与补偿路径。
- 状态流转、金额计算、权限、配置开关和兼容性。
- SQL 正确性、可维护性和性能风险。
- 回归风险、日志缺口和测试缺口。

只报告能够说明触发条件、影响和代码证据的问题。默认不报告纯风格、无实际风险、无业务价值或纯猜测的问题。

### 4. Part B：Requirement Validation

逐项验证：

- **Scope**：需求要求的所有范围是否完整实现。
- **Relevant Business Rules**：每条业务规则是否在正确路径和条件下落地。
- **Target State**：目标行为是否由最终代码实际实现，而非只在说明中声称完成。
- **Out of Scope**：是否误改本期明确不应改变的行为。
- **Scenario Coverage**：是否遗漏完整业务路径、调用链、数据源、角色或边界场景。
- **Over-Implementation**：是否增加需求未要求的行为或扩大影响范围。
- **Semantic Correctness**：代码本身可运行时，是否仍实现错了需求。
- **Validation Coverage**：测试和验证是否覆盖验收关键路径，证据层级是否被准确陈述。

建立需求项到代码证据、测试证据和结论的映射。缺少证据时使用 NOT VERIFIED，不将推测写成 PASS。

## Finding 规则

Finding 按严重级别使用 P0、P1、P2、P3。每个 Finding 至少包含：

- 标题与严重级别
- 触发条件
- 影响范围
- 代码证据
- 对应 Feature Context 规则
- 最小修复建议
- 是否确定 / Confidence

Review Finding 只是待验证判断。若历史 Finding 不成立或与 Feature Context 冲突，明确说明并不建议修改。修复建议只描述恢复需求正确性所需的最小方向，不将无关重构包装成修复。

输出时先列 Findings，并按严重级别排序；没有确认 Finding 时明确写“未发现有充分证据且与本次 Feature 相关的 Finding”。随后列出残余风险、测试缺口和未确认项。

## Final Requirement Gate

Review 结束时单独进行最终需求验收，对以下各项只使用 `PASS`、`FAIL` 或 `NOT VERIFIED`：

- Scope
- Relevant Business Rules
- Target State
- Out of Scope
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
