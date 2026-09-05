---
name: feature-implement
description: 基于当前有效 Feature Context、Design Handoff 与 Current Repo 实现 Target State；仅用于正式需求、已有 Feature 或已满足 Feature Context Contract 门禁的复杂跨服务变更。
---

# Feature Implement

## Goal

根据当前有效 Feature Context 完成实现，并把后续阶段真正需要的信息收敛为轻量 `TASK_RESULT`。

本 Skill 只负责 Feature Implement 阶段，不定义通用调查方法、修改范围规则、Review 规则、SQL / 日志 / 注释规范、测试策略或 Agent 执行策略。

## 使用门禁

任务是否进入 Feature 工作流，只以 [Feature Context Contract](../../references/feature-context-contract.md) 为准。

未通过门禁时，不为当前任务补建 Feature 状态或持久化文件。

## 阶段输入

进入实现阶段后，根据当前目标读取：

- `FEATURE_CONTEXT`；
- `DESIGN_HANDOFF`；
- 当前 `TASK_DEFINITION` 或 `REWORK_TASK`；
- 当前业务仓库与当前代码；
- 当前 Task 真正需要的 Requirement Evidence / Repository Evidence。

历史 Design、Handoff 或 Task Result 只提供线索，不能替代对当前真实代码和当前需求事实的必要验证。

## 设计有效性

开始实现前确认当前 Task 依赖的关键技术前提仍然适用于 Current Repo。

如果当前代码或新 Requirement Evidence 与既有设计前提冲突：

- 不继续机械执行受影响的旧设计；
- 重新确认受影响的 Current State、Gap 和 Technical Decision；
- 将会影响后续决策的 Design Deviation、Invalidated Premise 或 Open Issue 记录到 `TASK_RESULT`；
- 再以当前真实状态继续实现。

## 实现范围

Implementation 只负责关闭当前有效 Target State 对应的 Gap。

存在 `TASK_DEFINITION` 时，只完成当前 Task；不得因为发现后续 Task 或相邻改进而自动扩大当前修改范围。

如果 Review 产生 `REWORK_TASK`，将其作为普通当前 Task 处理，不另建平行 Feature 生命周期。

## Current Repo 作为 Durable State

已经完成的实现事实以 Current Repo 为主要 Durable State。

后续 Task 应重新读取与自身目标相关的当前代码，而不是依赖本 Task 的完整聊天历史。

代码能够恢复最终实现结果，不等于能够恢复决策原因；会影响后续 Task 的决策原因、兼容约束、Design Deviation、Dependency 和 Open Issue 必须进入 `TASK_RESULT`。

## TASK_RESULT

当前 Task 完成后，按 Contract 输出轻量 `TASK_RESULT`，至少包含：

- Task ID；
- Status；
- Implemented Capability；
- Changed Areas；
- Important Decisions；
- Design Deviations；
- Verification Result；
- Impact on Following Tasks；
- Open Issues。

不要包含：

- 完整搜索 / grep / read 过程；
- 完整工具调用；
- 完整修改过程；
- 大量中间推理；
- 与后续阶段无关的调查结果；
- 已可从 Current Repo 可靠确认且不影响后续判断的实现细节。

## 阶段完成

当前实现目标完成后：

- Current Repo 反映实际实现结果；
- 输出与实际验证状态、风险和未验证项一致的 `TASK_RESULT`；
- 存在设计偏差时，在 `TASK_RESULT` 中明确说明其对后续 Task / Review 的影响；
- 不因阶段完成自动创建或维护 Feature / Task / Progress / Handoff 状态文件；
- 用户、项目规则或当前任务明确要求文件交付时，再按该要求生成对应文件。
