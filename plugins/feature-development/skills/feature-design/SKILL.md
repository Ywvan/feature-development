---
name: feature-design
description: 用于已经满足 Feature Workspace Contract 门禁的正式需求或复杂跨服务变更的调查与技术设计；普通单服务 Bug、一次性诊断和一次性只读 Review 不使用本 Skill。
---

# Feature Design

## Goal

把当前有效 Requirement Evidence、经核验的关键需求事实和当前代码事实整理成可执行技术设计，并形成后续阶段可复用的 Feature 状态。

本 Skill 只负责 Feature Design 阶段，不定义通用调查方法、修改范围规则、Review 规则、SQL / 日志 / 注释规范、测试执行策略或 Agent 执行策略。

## Feature 工作区门禁

任务是否进入 Feature 工作流、是否创建或复用工作区，只以 [Feature Workspace Contract](../../references/feature-workspace-contract.md) 为准。

先读取 Contract 的任务门禁。未通过门禁时，不初始化 Feature Workspace，也不继续执行本 Skill 的阶段流程。

通过门禁后，再读取 Contract 中与当前 Design 阶段相关的工作区、Requirement Evidence、派生文档和 Handoff 规则。

## Design 输入与事实关系

根据当前任务使用以下有效输入：

- Requirement Evidence 与 `REQUIREMENT_SOURCES.md`；
- `FEATURE_CONTEXT.md`（如果存在）；
- 当前业务仓库与当前代码；
- 已有 `TECHNICAL_DESIGN.md` / `HANDOFF.md`（如果是继续同一 Feature）。

事实关系保持清晰：

- Requirement Evidence 与用户最新确认用于确定需求事实；
- 当前代码、配置和数据链路用于确定 Current State；
- `FEATURE_CONTEXT.md` 是需求派生上下文与 Evidence Index；
- `TECHNICAL_DESIGN.md` 与 `HANDOFF.md` 是技术决策和阶段状态，不是新的需求事实源。

## Hard Fact Verification

Feature Design 负责防止 Requirement Evidence 在 Context Compression 后丢失会直接改变结果的关键规则。

开始设计时：

1. 从当前 Requirement Working Context 识别 Feature Goal、Scope、Out of Scope、Acceptance Criteria、已列出的 Hard Facts 与 Evidence Pointers。
2. 原始 Requirement Evidence 可用时，对本 Feature Scope 做一次低成本 Hard Fact Discovery Scan，优先定位状态 / 枚举、金额公式 / 比例 / 阈值、权限 / 审批、幂等 / 并发、历史兼容、外部接口契约、明确的必须 / 不得 / 仅 / 只能，以及关键 AND / OR / NOT 条件。
3. Discovery Scan 用于发现被 Feature Context 整体漏掉的关键规则；先 search / find 定位，再读取命中位置及必要上下文，不以关键词未命中证明规则不存在。
4. 对与本 Feature 实现方向直接相关的高风险 Hard Fact，记录简短 ID、精确 Fact、Evidence Pointer 与 `VERIFIED` / `CONFLICT` / `NOT VERIFIED`。Evidence 明确存在但 Feature Context 缺失时，同时标记 Context Gap。
5. 权威 Requirement Evidence 彼此冲突且没有后续用户确认解决时，保留为 Requirement Conflict / Open Question，不自行合并或创造折中规则。

Hard Fact 的跨阶段保存、Rejected Hypothesis 与 Invalidated Premise 的准入条件统一遵守 Feature Workspace Contract。

## 阶段边界

Design 阶段允许写外部 Feature Workspace 文档，但业务仓库、业务代码、配置和数据库保持只读。

Feature 文档不得写入业务仓库。

## Design 输出

### 1. Current State

说明与当前 Feature 直接相关的现有实现、关键调用链、结果行为和现有约束，并保留支撑关键技术判断的代码位置。

### 2. Target State

说明需求完成后应达到的目标行为。经核验的 Hard Facts 必须保留精确语义，不得在设计中再次压缩成宽泛描述。

### 3. Gap Analysis

逐项说明 Current State 与 Target State 的差异。每个 Gap 应对应后续必要的实现或验证动作；无法确认的内容进入 Open Questions。

### 4. Technical Design

说明关闭上述 Gap 所需的当前有效技术方案，包括关键分支、数据一致性、失败处理、兼容约束和必要验证设计。

### 5. Implementation Plan

按依赖关系列出可执行步骤。简单 Feature 不强制拆分；存在相对独立闭环时可以按实际依赖拆分，并关联 Relevant Hard Facts。

### 6. Risks / Open Questions

记录会影响实现、发布或验收的当前风险、Requirement Conflict 与未确认项。

### 7. Validation Strategy

记录与当前 Feature 变更风险匹配的验证目标和验证范围。

## 阶段持久化

完成 Design 后：

- 将当前有效设计、Verified Hard Facts 与必要 Evidence Pointers 写入 `TECHNICAL_DESIGN.md`；
- 按 Contract 与模板初始化或更新 `HANDOFF.md`；
- 只有满足 Contract 持久化准入条件的高价值替代解释才写入 `Rejected Hypotheses`；
- 如果已有前提被当前证据推翻，并且后续重新采用该前提会改变决策，将其记录到 `Invalidated Premises`；
- `FEATURE_CONTEXT.md` 仅在 Contract 规定的场景下创建或更新。

Chat 输出用于说明当前设计结论和工作区位置，不替代外部 Feature 文档。
