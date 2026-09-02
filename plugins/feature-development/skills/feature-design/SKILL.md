---
name: feature-design
description: 用于已经满足 Feature Workspace Contract 门禁的正式需求或复杂跨服务变更的调查与技术设计；普通单服务 Bug、一次性诊断和一次性只读 Review 不使用本 Skill。
---

# Feature Design

## Goal

把当前有效需求和当前代码事实整理成可执行技术设计，并形成后续阶段可复用的 Feature 状态。

本 Skill 只负责 Feature Design 阶段，不定义通用调查方法、修改范围规则、Review 规则、SQL / 日志 / 注释规范、测试策略或 Agent 执行策略。

## Feature 工作区门禁

任务是否进入 Feature 工作流、是否创建或复用工作区，只以 [Feature Workspace Contract](../../references/feature-workspace-contract.md) 为准。

先读取 Contract 的任务门禁。未通过门禁时，不初始化 Feature 工作区，也不继续执行本 Skill 的阶段流程。

通过门禁后，再读取 Contract 中与当前 Design 阶段相关的工作区、Requirement Evidence、派生文档和 Handoff 规则。

## Design 输入

根据当前任务使用以下有效输入：

- Requirement Evidence 与 `REQUIREMENT_SOURCES.md`；
- `FEATURE_CONTEXT.md`（如果存在）；
- 当前相关业务仓库与当前代码；
- 已有 `TECHNICAL_DESIGN.md` / `HANDOFF.md`（如果是继续同一 Feature）。

事实关系保持清晰：

- Requirement Evidence 与用户最新确认用于确定需求事实；
- 当前代码、配置和数据链路用于确定 Current State；
- `FEATURE_CONTEXT.md` 是需求派生上下文；
- `TECHNICAL_DESIGN.md` 与 `HANDOFF.md` 是技术决策和阶段状态，不是新的需求事实源。

## Repository Scope

按 Contract 区分并维护：

- `Modification Repositories`：当前 Target State / Gap 的关闭方案包含写入修改，且写入未超出用户明确边界的仓库；
- `Read-only Evidence Repositories`：仅用于确认 Modification Repository 外部产生的字段、状态、金额、接口契约或行为的只读仓库。

当 Design 结论依赖 Modification Repository 外部产生的金额、状态、权限、接口字段、接口契约或行为，且当前已读取代码中不存在对应生产逻辑或权威定义时，读取对应 producer / authoritative source。

确认该结论所依赖的生产逻辑、参数、基准或契约后，不再因同一结论继续读取新的仓库；不得为了完整性遍历全部上下游仓库。

如果无法读取对应 producer / authoritative source，或者其中不存在该生产逻辑或权威定义，将对应结论标记为未验证项或 Open Question；不得根据 consumer 持久化结果、字段名、局部快照或历史结论补齐推断。

## 阶段边界

Design 阶段允许写外部 Feature 工作区文档，但业务仓库、业务代码、配置和数据库保持只读。

Feature 文档不得写入业务仓库。

## Design 输出

### 1. Current State

说明与当前 Feature 直接相关的现有实现、调用链、结果行为和现有约束，并保留支撑上述结论的代码位置。结论依赖跨仓生产逻辑或权威定义时，应保留对应 producer / authoritative source 的证据位置及其 Repository Role。

### 2. Target State

说明需求完成后应达到的目标行为。会直接改变结果的状态、金额、权限、接口和兼容规则必须保留精确语义。

### 3. Gap Analysis

逐项说明 Current State 与 Target State 的差异。每个 Gap 应对应至少一个实现动作或验证动作；无法确认的内容进入 Open Questions。

### 4. Technical Design

说明关闭上述 Gap 的当前有效技术方案，包括分支、数据一致性、失败处理、兼容约束和对应验证设计。

### 5. Implementation Plan

按依赖关系列出可执行步骤。简单 Feature 不强制拆分；存在相对独立闭环时可以按实际依赖拆分。

### 6. Risks / Open Questions

记录会影响实现、发布或验收的当前风险与未确认项。

### 7. Validation Strategy

记录与当前 Feature 变更风险匹配的验证目标和验证范围。

## 阶段持久化

完成 Design 后：

- 将当前 Repository Scope 和有效设计写入 `TECHNICAL_DESIGN.md`；
- 按 Contract 与模板初始化或更新 `HANDOFF.md`，同步当前 Repository Scope；
- 如果已有前提被当前证据推翻，并且后续重新采用该前提会改变决策，将其记录到 `Invalidated Premises`；
- `FEATURE_CONTEXT.md` 仅在 Contract 规定的场景下创建或更新，不写入 Repository Scope 作为需求事实。

Chat 输出用于说明当前设计结论和工作区位置，不替代外部 Feature 文档。
