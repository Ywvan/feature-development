---
name: feature-implement
description: 用于基于 Feature Goal、Confirmed Requirements、Global Constraints、Out of Scope、Implementation Handoff 与 Current Repo 完成软件 Feature 实现，也用于处理经过验证的 Review Finding。修改前必须重新读取当前真实代码；Handoff 不替代 Current Repo。只做实现 Target State 所需的最小必要修改，并完成与风险匹配的验证。
---

# Feature Implement

## 顶层目标

始终以“正确完成 Feature Context 定义的需求”为顶层目标。Implementation Handoff、Implementation Plan 和 Review Finding 都是实现输入，不得成为新的顶层目标。

本 Skill 只负责实现或 Review Fix，不自动进入独立 Review，不自动编排 Subagent，不创建跨会话状态或在业务仓库持久化 Feature 临时文档，也不规定 Codex 的搜索、工具、Task、Context 或 Agent Loop 策略。

## 输入优先级

按以下优先级处理输入：

1. Feature Context 中的 Feature Goal、Confirmed Requirements、Global Constraints、Out of Scope、Target State 和 Acceptance Criteria，以及用户当前明确指定的 Scope，共同决定需求边界；存在冲突时明确报告，不自行取舍。
2. 当前真实代码、配置和数据链路决定当前技术事实。
3. Implementation Handoff、Technical Design 和 Implementation Plan 指导 HOW。
4. Review Finding 只是待验证判断，优先级低于 Feature Context 与真实代码事实。

当 Handoff 中的技术事实或假设与 Current Repo 冲突时，以当前代码事实为准，判断原 Design 是否仍然有效，并明确报告关键偏差；不得静默基于过期 Handoff 修改代码。当冲突涉及 Confirmed Requirements、Global Constraints、Out of Scope、Target State 或 Acceptance Criteria 时，不得擅自修改需求，必须报告并请求确认。

## 修改前检查

开始编辑前完成：

1. 读取 Feature Context 与用户当前明确指定的 Scope，提取 Feature Goal、Confirmed Requirements、Global Constraints、Out of Scope、Target State 和 Acceptance Criteria。
2. 读取 Implementation Handoff，识别 Key Gaps、Technical Decisions、Compatibility Constraints、Relevant Code Areas、Known Risks、Open Issues 和建议的 Workstreams（如有）。
3. 检查仓库级指令、当前工作树状态和用户已有修改，避免覆盖或混入无关变更。
4. 重新阅读待修改区域及必要上下游调用链、数据流、SQL、配置和测试。
5. 验证决定实现边界的关键技术假设，不默认 Design Chat 一定正确。
6. 将无法确认且会实质改变业务行为或修改范围的问题提交给用户，不以猜测继续。

## Workstream 与执行边界

Suggested Implementation Workstreams 只是可选的 Context Organization 建议，不是 Task、Session、Context 或 Subagent 的运行时协议。简单 Feature 或无需隔离认知范围时，不要强行拆分。

实现期间始终保持以下映射：

- Current Workstream / Current Goal；
- Relevant Requirement；
- Modification Scope；
- Acceptance Criteria。

存在多个 Workstream 时，可以由同一 Main Agent 按依赖关系连续实现，并在每个 Workstream 后执行必要的局部验证；不要求 Subagent，也不要求 Fresh Context。修改共享代码时，重新检查已完成和后续 Workstream 的交叉影响，并在最终阶段执行 Feature-level 验收。

Codex 自主决定搜索方式、工具顺序、Task 数量与顺序、是否隔离 Context、是否使用 Subagent 及其数量。本 Skill 不绑定一个 Context 必须处理多少 Workstream，也不把 Task Decomposition 等同于 Context Isolation 或 Subagent Delegation。

## Current Repo 与跨 Context 信息

Current Repo 是已实现技术事实的主要来源。后续 Context 不需要继承先前 Workstream 的完整聊天或工具历史；仅在代码无法完整表达且仍与后续实现相关时，携带 Requirement、Constraint、Intentional Design Decision、Compatibility Contract、Intentional Design Deviation、Dependency 和 Open Issue。

不要为此建立 Controller State、Worker History、Task State Database、Session Registry 或 Execution Event Log。

## 实现规则

只做完成 Target State 所需的最小必要修改：

- 不扩大 Scope，不新增 Feature Context 未要求的行为。
- 不进行无关重构、抽象、改名、格式化或顺手优化。
- 不修改接口协议、响应结构、数据库结构、枚举定义或既有业务语义，除非需求明确要求。
- 允许扩大只读分析范围以确认影响点，但不得静默扩大修改范围。
- 保留用户已有修改；只编辑已确认属于本 Feature 的文件和代码区域。
- 复用当前项目成熟模式，避免为架构美观引入新框架或间接层。
- 遵守当前仓库和环境已生效的注释、日志、SQL 等工程 Guardrails；本 Skill 只检查这些约束是否在本 Feature 修改范围内得到满足，不重复定义具体风格。
- 若完成任务必须扩大修改范围，先说明原因、影响和不可替代性；需要新增权限或改变需求时停止并请求用户确认。

不要在业务仓库创建 `FEATURE_CONTEXT.md`、`TECHNICAL_DESIGN.md`、`PLAN.md` 或其他 Feature 临时文件，除非用户明确要求创建该交付物。

## Review Fix Mode

用户提供 Review Finding 时，逐条执行：

1. 沿真实代码调用链验证 Finding 是否成立及其触发条件。
2. 对照 Feature Context 验证 Finding 是否符合需求目标和修改范围。
3. Finding 不成立时不修改，并给出代码证据。
4. Finding 与 Feature Context 冲突时明确拒绝机械修复，并说明冲突规则。
5. Finding 成立时只做最小必要修复，避免把局部问题扩展成重构。
6. 修复后重新检查 Target State、Out of Scope 和相关回归路径。

不得以“清零所有 Finding”代替“正确完成 Feature Context”。

## 验证

根据项目条件和变更风险执行并区分：

- 静态 diff 与调用链检查。
- 针对性单元测试。
- 编译或构建。
- 必要的集成验证。
- 关键正常、异常与边界业务分支检查。
- 数据库迁移、SQL 或数据验证。
- 回归风险检查。
- 多 Workstream 场景下的局部验证与最终 Feature-level Acceptance Criteria 检查。

不得把未执行、超时、被基线错误阻塞或仅静态检查描述为“验证通过”。明确记录命令、结果、失败原因和未覆盖范围。

## 完成前检查

重新逐项对照 Confirmed Requirements、Global Constraints、Target State、Acceptance Criteria 和 Out of Scope，确认：

- 每个必要 Gap 都已实现或明确标记未完成。
- 修改范围内没有无关文件或意外语义变化。
- 接口、数据库和兼容行为的变化均有需求授权。
- 必要注释、日志和测试完整。
- 与 Handoff 的偏差有真实代码依据、已明确记录且未改变业务目标。
- 每个适用 Workstream 的 Requirement、Modification Scope 和 Acceptance Criteria 均已闭环，并完成跨 Workstream 集成检查。

存在重要需求未完成或未验证时，不得声明 Feature 已完成。

## 输出要求

最终至少包含：

- `### 修改文件`
- `### 实现内容`
- `### Target State Coverage`
- `### Business Rule Coverage`
- `### 与 Implementation Handoff 的偏差`
- `### Tests / Validation`
- `### Known Risks`
- `### Unfinished Items`

同时明确是否改变接口、数据库和已有业务语义，是否需要补充测试，以及所有不确定或未验证项。
