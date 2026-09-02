---
name: feature-implement
description: 基于已有 Feature Workspace 实现当前有效 Target State；仅用于正式需求、已有 Feature 或已满足 Feature Workspace Contract 门禁的复杂跨服务变更。
---

# Feature Implement

## Goal

根据当前有效 Feature 状态完成实现，并把实际结果同步回跨阶段 Handoff。

本 Skill 只负责 Feature Implement 阶段，不定义通用调查方法、修改范围规则、Review 规则、SQL / 日志 / 注释规范、测试策略或 Agent 执行策略。

## 使用门禁

任务是否进入 Feature 工作流、是否创建或复用工作区，只以 [Feature Workspace Contract](../../references/feature-workspace-contract.md) 为准。

没有既有 Feature 工作区时，先按 Contract 判断任务门禁；未通过门禁时，不为实现阶段创建或补齐 Feature 文件。

## 阶段输入

进入实现阶段后，先恢复当前有效 Feature 状态：

- `HANDOFF.md`；
- `REQUIREMENT_SOURCES.md`；
- 当前相关业务仓库与当前代码；
- `TECHNICAL_DESIGN.md`（`HANDOFF.md` 未记录当前实现步骤对应的 Gap、Design 或 Repository Scope 时）；
- `FEATURE_CONTEXT.md` 或 Requirement Evidence（`HANDOFF.md` 未记录当前实现所依赖的业务规则，或记录存在冲突 / 未读取的 Evidence Pointer 时）。

`HANDOFF.md` 用于恢复当前阶段状态，不替代 Requirement Evidence、当前代码或 Technical Design。

## 设计有效性

开始实现前确认当前 Handoff / Technical Design 中与本次修改直接相关的技术前提仍然适用于当前代码基线。

如果当前代码或新 Requirement Evidence 与上述技术前提冲突：

- 不继续机械执行受影响的旧设计；
- 更新 `TECHNICAL_DESIGN.md` 中受影响的 Current State、Gap、Design 或 Open Question；
- 将会影响后续决策的失效前提写入 `HANDOFF.md` 的 `Invalidated Premises`；
- 再以更新后的当前 Feature 状态继续实现。

## 实现范围

Implementation 只负责关闭当前有效 Target State 对应的 Gap。

只允许在当前 Repository Scope 的 `Modification Repositories` 中写入业务代码、配置或其他业务仓库内容；`Read-only Evidence Repositories` 只用于确认 producer、contract owner、共享数据语义或上下游行为，不因实现阶段读取而自动进入修改范围。

仅当不修改某个 Read-only Evidence Repository 就无法满足当前已确认的 Target State / Acceptance Criteria，或当前跨仓契约在保持该仓库不变时无法成立，才将其调整为 Modification Repository。

调整前：

1. 记录触发上述条件的 Target State / Acceptance Criteria 或跨仓契约；
2. 更新 `TECHNICAL_DESIGN.md` 与 `HANDOFF.md` 的 Repository Scope、跨仓依赖和验证范围；
3. 如果调整会扩大用户明确的修改边界、改变业务目标或引入新的接口 / 业务契约，先说明并取得确认；否则按更新后的当前 Feature 状态继续实现。

代码清理、顺带优化、统一实现方式或降低未来维护成本，不构成将 Read-only Evidence Repository 调整为 Modification Repository 的条件。

在完成上述更新前，不对原 Read-only Evidence Repository 写入修改。

如果 Review 产生 Feature Rework，当前 Rework 只作为该 Feature 的当前实现目标；处理结果仍回写同一个 Feature Workspace，不另建平行生命周期。

## 阶段状态更新

在 Repository Scope 变化、Design Deviations 出现、验证结束和交付前更新 `HANDOFF.md`，至少同步：

- Repository Scope 与各仓库当前代码基线；
- 实际修改区域；
- 与 Technical Design 的偏差；
- 当前验证结果与未验证范围；
- 风险 / Blocker；
- 会影响后续决策的 Invalidated Premises；
- 下一动作。

Requirement Evidence 只追加或更新来源状态，不由实现阶段改写原始证据。

## 阶段完成

当前实现目标完成后：

- `HANDOFF.md` 反映实际实现后的最新状态；
- `TECHNICAL_DESIGN.md` 与实际代码存在设计偏差时同步更新当前有效设计；
- 交付结论与 Handoff 中的验证状态、风险和未验证项保持一致。

Feature 文档只写外部工作区，不写入业务仓库。
