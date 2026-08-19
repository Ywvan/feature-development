---
name: feature-review
description: 对已完成或部分完成的 Feature 做独立 Review。基于当前需求、最终 diff、相关当前代码和验证证据，检查需求是否实现正确、关键场景是否遗漏、集成是否可靠。默认只读，不修改生产代码。
---

# Feature Review

## Goal

独立判断当前实现是否正确完成需求，而不是制造 Finding 或清零 Checklist。

Review 默认只读。Technical Design、Implementation Handoff、实现说明和历史 Review Finding 都只能作为线索，不能代替当前需求、最终 diff、当前代码和实际验证证据。

## Review Baseline

开始时确认：

- 当前 Feature Goal、范围、关键业务规则、Out of Scope 和验收条件；
- 用户指定或当前可确定的最终 diff；
- 与该 diff 直接相关的当前代码和必要上下游；
- 已执行的构建、测试或其他验证结果。

只有当关键需求规则缺失、冲突或会改变 Review 结论时，才回查对应原始需求证据；不要机械重新读取全部 PRD、原型或接口资料。

## Review 方法

先理解变更实际做了什么，再判断是否存在问题。

根据当前 Feature 的真实影响按需检查：

- 需求是否完整落地；
- 状态、金额、权限、审批、幂等、兼容和接口契约等关键规则是否实现准确；
- 正常、异常、边界和历史兼容场景；
- 相关调用链、数据流、事务、SQL、RPC、MQ、Job、缓存和异步边界；
- 是否存在漏改、误改、过度实现或回归风险；
- 验证证据是否足以支持“已完成”。

不要因为某类风险出现在 Checklist 中就主动寻找一个对应 Finding。Finding 必须来自当前 diff / 当前代码 / 需求或验证证据之间的真实矛盾。

通用代码正确性、SQL、日志和注释风格可以使用当前环境中的工程 Guardrail，但 Feature Review 负责最终判断这些问题是否真的影响当前 Feature；不要重复执行一套平行的大型 Review 流程。

## Finding 规则

一个有效 Finding 至少要说明：

- 问题是什么；
- 触发条件；
- 代码或验证证据；
- 为什么会违反当前需求或造成真实工程风险；
- 最小修复方向（如果能够可靠判断）。

如果证据不足，标记“不确定”并说明缺什么证据，不把推测包装成问题。

Review Finding 是“待后续复核的问题判断”，不是新的需求事实。后续进入修复阶段时仍应重新读取相关当前代码验证 Finding 是否成立。

## Final Gate

Review 结束时只需要回答以下问题：

- 当前需求是否正确实现；
- 是否存在阻塞完成的真实 Finding；
- 关键业务场景和集成路径是否有明显遗漏；
- 当前验证是否足以支持完成结论。

没有阻塞问题时输出 `READY`；存在会导致需求错误、功能不完整或高风险回归的问题时输出 `NOT READY`。必要但暂时无法验证的关键项应明确说明，不为了形式完整构造复杂矩阵。

## 输出

默认使用易读、直接的中文工程表达。

推荐结构：

1. `## Review 结论`
2. `## Findings`（没有问题时直接写未发现明确阻塞问题）
3. `## 验证与剩余风险`

每个 Finding 优先用具体业务和代码语言描述，不反复使用 Requirement Authority、Hard Fact Coverage、Evidence Pointer、Integration Correctness、Final Gate 等元流程术语。

不要输出没有证据的风格偏好，不为了凑数量输出低价值 Finding。
