---
name: feature-implement
description: 基于当前有效 Feature Context、Design Handoff 与 Current Repo 实现 Target State；仅用于正式需求、已有 Feature 或已满足 Feature Context Contract 门禁的复杂跨服务变更。
---

# Feature Implement

只负责当前有效 Target State 的实现，不定义通用工程、验证或 Agent 执行策略。已有有效 `FEATURE_CONTEXT` 与 `DESIGN_HANDOFF` 时，不重复执行 Feature 任务门禁；仅读取 [Feature Context Contract](../../references/feature-context-contract.md) 中与事实源 / Durable State、仓库外资料、`TASK_DEFINITION` / `TASK_RESULT` 和 Implement 阶段边界相关的内容。缺少有效 Feature Context 时再读取任务门禁；未通过时不补建 Feature 状态或文件。

## 输入与设计有效性

读取有效 `FEATURE_CONTEXT`、`DESIGN_HANDOFF`、当前 `TASK_DEFINITION` / `REWORK_TASK`、Current Repo 及当前 Task 依赖的 Requirement / Repository Evidence。历史设计、Handoff、Task Result 只提供线索，不能替代当前需求和代码核验。

修改前核验当前 Task 依赖的入口、接口契约、数据读写及状态/金额/权限规则。代码或新证据推翻前提时，停止受影响的旧方案，重新确认 Current State、Gap 与 Technical Decision；将影响后续决策的 Design Deviation、Invalidated Premise 或 Open Issue 记入 `TASK_RESULT`，再按当前事实实现。

## 授权范围

只关闭有效 Target State 对应的 Gap。用户授权整个实现阶段时，按依赖继续完成授权内的后续 Task；仅被分配一个 Task 时只完成该 Task。任务拆分不扩大业务目标，不越过阶段授权或用户指定的确认节点，相邻改进不并入范围。

Review 产生的 `REWORK_TASK` 作为当前 Task 处理，不另建 Feature 生命周期。

## 交付

实现事实由 Current Repo 承载；后续 Task 重新读取相关代码，不依赖本 Task 完整聊天历史。代码不能恢复的决策原因、兼容约束、Design Deviation、Dependency、Open Issue，以及对后续 Task / Review 的影响，进入 `TASK_RESULT`。

按 Contract 的 `TASK_RESULT` 最小内容输出结果，验证结果、风险及未验证项与实际一致；不复制搜索/工具/修改全过程、中间推理、无关调查或可从代码恢复且不影响后续判断的细节。交付前更新原仓库外当前有效 `HANDOFF.md` 快照，不为每个 Task 另建状态文件。
