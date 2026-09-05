# Feature Context Contract

本契约是 `feature-development` 的 Feature 生命周期与跨阶段 Context 唯一规范。它定义任务门禁、Context Contract、阶段边界和跨阶段传递规则，不重复维护通用工程行为规则，也不要求通过持久化状态文件实现 Agent Memory。

## 1. 任务门禁

### Requirement Development

以下任务进入 Feature 工作流：

- 新增业务能力；
- 需求迭代；
- 修改既定业务规则、状态语义、金额口径、权限 / 审批规则或接口契约；
- 明确提出新的 Target State 与 Acceptance Criteria 的正式需求；
- 明确继续已有 Feature 的任务。

用户不需要显式说出 “Feature”。

### Bug Fix

目标是恢复已经存在或已经确定的正确行为时，仍属于 Bug Fix。读取 PRD、历史需求或产品规则确认预期行为，本身不会把 Bug 升级为 Feature。

以下任务默认不进入 Feature 工作流：

- 单服务 / 单仓库 Bug；
- 单点回归；
- 单个 Review Finding 修复；
- 纯诊断、根因排查；
- 不属于已有 Feature 的一次性只读 Review；
- 解释问答；
- 不改变业务行为的机械修改。

“单服务 / 多服务”按完成任务最终确认会写入修改的可独立部署服务或仓库数量判断，不按只读调查读取了多少服务判断。

### Bug 升级为 Feature

Bug 满足以下任一条件时可以进入 Feature 工作流：

1. 当前修复目标包含新增或改变业务规则、接口契约、Target State 或 Acceptance Criteria；
2. 当前修复方案包含两个或以上可独立部署服务 / 仓库的协同写入修改，且这些修改之间存在跨服务契约、发布顺序或跨阶段 Handoff 依赖；
3. 当前 Bug 属于已有正式 Feature；
4. 用户明确要求使用 Feature 工作流。

任务性质或最终修改范围尚未确认时，不提前构造 Feature 状态；确认满足门禁后再进入对应阶段。

## 2. Context Contracts

Feature 工作流使用以下结构化 Context Contract：

- `FEATURE_CONTEXT`：当前有效需求事实输入；
- `DESIGN_HANDOFF`：Design 向 Implementation 传递的最小有效设计上下文；
- `TASK_DEFINITION`：当前实现 Task 的目标、边界、依赖与验收；
- `TASK_RESULT`：当前 Task 给后续阶段留下的必要事实；
- `REWORK_TASK`：Review 产生的明确返工目标；
- `REVIEW_RESULT`：独立 Feature Review 的 Gate、Finding 与验证结论。

这些 Contract 是结构化上下文协议，不绑定具体文件名、目录或存储形式。

除非用户、项目规则或当前任务明确要求生成文件，否则不得仅为了 Feature 状态传递而创建或维护持久化文件。

## 3. 事实源与 Durable State

需求事实以当前有效 `FEATURE_CONTEXT`、原始 Requirement Evidence 和用户最新明确确认为依据。出现冲突时必须回到原始事实确认，不得由 Technical Design、Handoff 或历史 Review 静默改写需求。

已经完成的实现事实以 `Current Repo` 为主要 Durable State。后续 Task 应重新读取与当前目标相关的真实代码，而不是依赖前一个 Worker 的完整聊天历史。

技术设计与跨阶段 Contract 只保存当前 Repo 无法可靠恢复、且会影响后续阶段的信息，例如：

- Contract；
- Important Decision；
- Design Deviation；
- Dependency；
- Compatibility Constraint；
- Open Issue。

代码能够恢复最终实现结果，不等于能够恢复决策原因；会影响后续判断的原因仍应保留在对应 Contract 中。

## 4. 持久化边界

默认不得为了以下目的创建 Feature / Task / Workflow / Review 状态文件：

- 保存 Agent 记忆；
- 保存聊天历史或工具调用；
- 标记当前执行进度；
- 记录可从 Current Repo 重新确认的实现事实；
- 为下一阶段复制完整上游 Context。

只有满足以下任一条件时，才主动创建或更新文件：

1. 文件本身是用户明确要求的交付物；
2. 项目已有规范明确要求维护该文件；
3. 后续外部阶段只能通过该文件消费必要信息；
4. 当前必要信息无法通过 Current Repo、已有事实源或运行时 Context 可靠恢复，且缺失会影响任务正确性。

文件只是可选承载形式，不改变 Context Contract 的语义和事实优先级。

## 5. Contract 最小内容

### DESIGN_HANDOFF

至少包含：

- Feature Goal；
- Confirmed Requirements；
- Global Constraints；
- Out of Scope；
- Current State Summary；
- Target State；
- Gap Summary；
- Technical Decisions；
- Compatibility Constraints；
- Task Graph；
- Feature-level Acceptance Criteria；
- Known Risks；
- Open Issues。

不要继续传完整搜索过程、工具调用、聊天摘要、大段代码或已失效方案。

### TASK_DEFINITION

至少包含：

- Task ID / Goal；
- In Scope / Out of Scope；
- Dependencies；
- Required Contract / Constraints；
- Acceptance Criteria；
- Verification。

### TASK_RESULT

至少包含：

- Task ID；
- Status；
- Implemented Capability；
- Changed Areas；
- Important Decisions；
- Design Deviations；
- Verification Result；
- Impact on Following Tasks；
- Open Issues。

`TASK_RESULT` 不记录完整执行历史。能够从 Current Repo 可靠恢复且不会影响后续判断的信息，不因“留档”而重复展开。

### REWORK_TASK

至少包含：

- Finding / Requirement；
- Expected Behavior；
- Rework Scope；
- Constraints；
- Acceptance Criteria；
- Verification。

### REVIEW_RESULT

至少包含：

- Review Baseline；
- `READY` / `NOT READY`；
- Current Findings；
- Verification Evidence；
- Remaining Risks / Unverified Items；
- 必要时的 `REWORK_TASK`。

## 6. 阶段边界

### Design

- 使用 `FEATURE_CONTEXT`、Requirement Evidence、Current Repo 和必要的 Repository Evidence 构造 Current State；
- 输出 Current State、Target State、Gap Analysis、Technical Design、Task Graph 与 `DESIGN_HANDOFF`；
- 业务仓库、业务代码、配置和数据库保持只读；
- 不因阶段完成自动创建状态文件。

### Implement

- 使用 `FEATURE_CONTEXT`、`DESIGN_HANDOFF`、当前 `TASK_DEFINITION` / `REWORK_TASK` 与 Current Repo；
- 开始修改前重新验证当前 Task 依赖的关键技术事实；
- 只关闭当前有效 Target State 对应的 Gap；
- 完成后输出轻量 `TASK_RESULT`；
- 实际实现事实由 Current Repo 承载，不另建平行状态源。

### Review

- 使用 Fresh Review Context；
- 基于 `FEATURE_CONTEXT`、`DESIGN_HANDOFF`、Final Diff、Relevant Current Repo、验证结果和必要的 Task Contract / Design Deviation 独立重建结论；
- 不继承 Worker 的完整 Implementation Context；
- 业务仓库、生产代码、配置和数据库保持只读；
- 输出 `REVIEW_RESULT`，需要返工时生成 `REWORK_TASK`。

## 7. 跨阶段传递

新阶段只接收当前目标需要的 Contract 和事实源，并重新读取相关 Current Repo。

默认不要传递：

- 完整聊天历史；
- 完整搜索 / grep / read 历史；
- 工具调用记录；
- 完整代码副本；
- 已可从 Current Repo 重新确认的实现细节；
- 与当前阶段无关的历史调查结果。

任何历史设计、Handoff、Finding 或先前结论都不能替代对当前需求事实和当前真实代码的必要验证。
