# Feature Context Contract

本契约是 `feature-development` 的 Feature 生命周期与跨阶段 Context 唯一规范。它定义任务门禁、Context Contract、阶段边界和跨阶段传递规则，不重复维护通用工程行为规则。

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

Contract 定义信息内容，文件位置与维护按第 4 节执行；字段统一在第 5 节定义。可引用接收方能读取的对应资料，不重复全文或使用不可访问的“见上文”。

## 3. 事实源与 Durable State

需求以有效 `FEATURE_CONTEXT`、原始 Requirement Evidence 与用户最新确认为依据；冲突回到原始事实核验，Design、Handoff、Review 不得静默改写需求。用户明确的新需求可更新目标，现状推测仍需验证。

实现事实以 Current Repo 为准，各阶段读取当前目标相关代码；历史设计、Handoff、Finding 和 Task Result 是待验证输入，不替代当前事实。保留代码无法恢复且影响后续判断的 Contract、Important Decision、Design Deviation、Dependency、Compatibility Constraint、Open Issue 和决策原因。

跨阶段不复制完整聊天、搜索/工具历史、大段代码、无关调查或失效方案；对已可从当前代码恢复且不影响后续判断的实现细节不重复留档。精简上下文不得省略需求硬事实、验收证据和未验证项。

`HANDOFF.md` 维护当前有效交接状态，不按时间追加工作记录。事实被新证据替代、问题已解决，或信息已能从 Current Repo 可靠恢复且不再影响后续判断时，更新或删除旧内容，不同时保留多个历史版本；仍影响后续决策的原因、约束、偏差、依赖和未解决问题继续保留。

## 4. 仓库外资料与交接

沿用原有业务代码仓库外目录、子目录与文件名；需求身份及关联仓库一致时复用，不迁移或另设平行目录。

新 Feature 根目录使用当前 Codex 设置中的 `Projectless task folder` 下的 `features/` 子目录；子目录为 `YYYYMMDD-[需求编号-]需求简称/`。当前运行环境无法读取 `Projectless task folder` 的实际配置值时，报告无法确定目录并停止 Feature 文档写入；不得猜测路径、写死盘符或用户名，也不得使用环境变量或其他目录作为回退。

- 原始资料保存在 `requirements/`（聊天原始需求使用 `ORIGINAL_REQUEST.md`），来源及新确认登记到 `REQUIREMENT_SOURCES.md`，不覆盖原始证据。
- Design 维护 `TECHNICAL_DESIGN.md` 和 `HANDOFF.md`；Implement 交付前更新 `HANDOFF.md`；独立 Review 更新 `REVIEW_RESULT.md`，并把 Gate、阻塞项与验证状态同步到 `HANDOFF.md`。
- 已有或用户要求的 `FEATURE_CONTEXT.md` 按第 5 节维护。交接保留代码无法恢复的需求、决定、约束、依赖及未解决问题；按本次 Contract 字段引用已有资料；对应内容与当前有效事实一致且接收方可读取才可引用，缺项补充、变化更新，不能以原生运行记忆代替。
- 不为每个 Task 另建进度文件、完整工具记录或重复代码摘要；不规定宿主如何管理自身运行笔记。

业务代码仓库仅允许写入当前任务直接修改或新增的源码、配置、构建文件、数据库迁移、测试资源、接口定义和仓库内既有文档，以及用户明确要求放入该仓库的其他文件。除此之外，Feature Context、设计 / 交接文档、临时分析文件、导出文件、日志、截图、工具输出等不得写入、复制或移动到业务代码仓库。

用户禁止写文件时，输出内容并说明未落盘。外部文档写入不授予业务代码仓库写权限；Design / Review 的整个业务代码仓库保持只读，既有未提交修改不得清除，提交/推送仍受用户授权与确认点限制。

## 5. Contract 最小内容

### FEATURE_CONTEXT

包含目标、范围/排除项、验收、来源及待确认项。状态白名单/黑名单、枚举、金额公式与单位口径、比例/阈值、必填、状态流转条件、AND/OR/NOT 及例外逐项保留，不同判断维度分开；不得抽象省略会改变业务结果的条件。

生成或更新时逐项回查原始证据与用户确认；未成功读取的资料、冲突和未知项注明来源及受影响规则，不自行补齐。

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
- 按第 4 节维护仓库外技术设计与交接资料。

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
