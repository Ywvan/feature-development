---
name: feature-review
description: 对已有正式 Feature 的最终实现做独立 Requirement Review，基于 Requirement Evidence、当前 Handoff、最终 diff、当前代码和验证证据判断 READY / NOT READY；业务仓库保持只读。
---

# Feature Review

## Goal

独立判断当前实现是否正确完成当前 Feature，而不是重复实现阶段结论或制造 Finding。

本 Skill 只负责 Feature Requirement Correctness 与最终 Gate，不重新定义通用代码 Review、调查方法、SQL / 日志 / 注释规范、严重级别体系或 Agent 执行策略。

## 使用边界

已有正式 Feature 的 Review 按 [Feature Workspace Contract](../../references/feature-workspace-contract.md) 使用现有工作区。

不属于已有 Feature 的一次性只读 Review 不强制创建 Feature 工作区。

Review 阶段业务仓库、生产代码、配置和数据库保持只读；允许写外部 Feature Workspace 的 `REVIEW_RESULT.md` 与 `HANDOFF.md`。

## Review Baseline

开始时恢复并确认：

- 当前 Feature Goal、Confirmed Requirements、Critical Business Rules、Out of Scope 与 Acceptance Criteria；
- 当前 `HANDOFF.md` 中的实现状态、设计偏差、Invalidated Premises 和验证状态；
- 用户指定或当前可确定的最终 diff；
- 与最终 diff 相关的当前代码；
- 已执行验证及其结果。

Requirement Evidence、当前代码和当前验证证据用于重建 Review 结论；Technical Design、Handoff、实现说明和历史 Review Result 只提供阶段线索。

## Feature Review 关注点

判断：

- Confirmed Requirements 是否完整实现；
- Target State 与 Acceptance Criteria 是否达到；
- Critical Business Rules 是否被遗漏或改写；
- 是否存在漏改、过度实现或超出 Out of Scope；
- 实际实现是否偏离当前有效 Technical Design，以及偏离是否影响需求正确性；
- 是否存在会阻塞 Feature 完成的 Missing Scenario；
- 当前验证状态是否足以支持 Feature 完成结论。

通用代码风险如被当前 Review 发现，只在其实际影响当前 Feature 时纳入最终 Gate；不要在本 Skill 中维护平行的通用 Review Checklist。

## Feature Finding

需要记录 Feature Finding 时，在当前环境通用 Finding 信息之外，补充：

- 对应 Requirement / Acceptance Criteria；
- 对 Feature Goal 或 Target State 的具体影响；
- 属于缺失实现、设计偏差、场景遗漏还是 Verification Gap。

历史 Finding 不是当前结论；复审时以当前 baseline 重建结果。

## Final Gate

输出：

- `READY`：当前 Feature 已正确达到目标，且不存在阻塞完成的真实问题；
- `NOT READY`：存在导致需求错误、功能不完整、关键场景遗漏或关键验证不足的问题。

关键项暂时无法确认时，应作为当前 Verification Gap / Remaining Risk 保留，不能通过更新文档把未知状态变成 READY 证据。

## 阶段持久化

正式 Feature Review 完成后：

- 使用 [REVIEW_RESULT 模板](../../assets/feature-workspace/REVIEW_RESULT.md) 更新 `REVIEW_RESULT.md`；
- 将最新 Gate、阻塞项、验证状态和下一动作同步到 `HANDOFF.md`；
- 如果 Review 证据推翻了此前会影响后续决策的技术前提，将该前提加入 `Invalidated Premises`；
- `REVIEW_RESULT.md` 保存本轮独立 Review 结果，`HANDOFF.md` 只保存后续阶段需要的当前状态。
