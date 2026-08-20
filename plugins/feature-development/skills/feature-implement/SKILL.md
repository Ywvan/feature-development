---
name: feature-implement
description: 用于基于外部 Feature 工作区的 Requirement Evidence、滚动 Handoff、Verified Hard Facts、Technical Design 与 Current Repo 完成正式 Feature 实现，也用于处理经过验证的 Review Finding。修改前重新读取当前真实代码，实施最小必要修改、验证并更新 Handoff；派生文档不替代原始需求或 Current Repo。
---

# Feature Implement

## 顶层目标

始终以“正确完成 Requirement Authority 定义的需求”为顶层目标。Feature Context 是 Working Context，Implementation Handoff、Implementation Plan 和 Review Finding 都是实现输入，不得成为新的独立需求事实源。

本 Skill 只负责实现或 Review Fix，不自动进入独立 Review，不自动编排 Subagent，不在业务仓库持久化 Feature 临时文档，也不规定 Codex 的搜索、工具、Task、Context 或 Agent Loop 策略。跨阶段只维护外部 Feature 工作区的滚动 Handoff，不建立 Controller、任务数据库或执行历史。

## Feature 工作区

处理正式需求时，先完整读取并遵守 [Feature Workspace Contract](../../references/feature-workspace-contract.md)：

1. Feature 工作区逻辑根目录使用 `~/Documents/Codex/features/`。实际文件操作前按当前 WSL 用户解析为绝对路径；不得写死用户名、`/home/<user>` 或 `/mnt/c/Users/<user>` 等单机路径。Windows 路径仅在用户明确要求导出或定位 Windows 文件时按需转换。
2. 优先识别并复用已有需求目录；若正式需求绕过 Design 直接进入实现且尚无目录，停止代码编辑，先补齐原始 Evidence、`REQUIREMENT_SOURCES.md`、最小 Technical Design 和 `HANDOFF.md`。
3. 新阶段先读 `HANDOFF.md` 与 `REQUIREMENT_SOURCES.md`，再按当前目标选择性读取 Technical Design、Feature Context 或原始证据片段，不无差别重读全部资料。
4. 在重要实现里程碑、验证结束和交付前更新 `HANDOFF.md` 当前态；不得修改 `requirements/` 中的原始需求。

## 输入优先级

按以下规则处理输入：

1. 用户当前明确指定的 Scope、Feature Context 中的有效需求上下文，以及 Implementation Handoff 中已经 Evidence Verification 的 `Verified Hard Facts` 共同形成实现阶段的 Requirement Working Context。
2. `Verified Hard Facts` 中的精确状态、枚举、金额公式、阈值、权限、审批、兼容、幂等和接口契约等条件，不得在实现阶段再次抽象压缩或省略。
3. Feature Context 与 `Verified Hard Facts` / 对应 Requirement Evidence 不一致时，不得仅以 Feature Context 为准；按 Evidence Pointer 核验并明确报告。
4. 当前真实代码、配置和数据链路决定当前技术事实。
5. Implementation Handoff、Technical Design 和 Implementation Plan 指导 HOW。
6. Review Finding 只是待验证判断，必须结合 Requirement Working Context 与真实代码复核。

当 Handoff 中的技术事实或假设与 Current Repo 冲突时，以当前代码事实为准，判断原 Design 是否仍然有效，并明确报告关键偏差；不得静默基于过期 Handoff 修改代码。

当冲突涉及 Requirement Authority、Verified Hard Facts、Global Constraints、Out of Scope、Target State 或 Acceptance Criteria 时，不得擅自改写需求。若只是 Feature Context 与其明确 Evidence 不一致，按 Evidence 修正派生上下文并报告；若权威 Requirement Evidence 彼此冲突且未被后续用户确认解决，则停止相关高风险修改并请求确认。

## 核心前提失效处理

新证据或用户纠正推翻当前实现方案的核心前提时，立即停止依赖该前提的编辑和修复：明确失效前提、受影响的设计结论及已修改区域，返回必要的只读调查，并重新形成受影响范围的 Current State、Target State 与 Gap Analysis。已产生的修改先标记为待复核，不得擅自覆盖用户工作或继续在旧方案骨架上做局部补丁；只有重新取证后仍成立的部分才能继续保留和实现。

同时将失效前提、影响范围、当前修改状态、替代结论或下一动作与 Evidence Pointer 写入 `TECHNICAL_DESIGN.md` 和 `HANDOFF.md` 的 `Invalidated Premises`；旧结论不得继续保留在当前有效决策区。

## 修改前检查

开始编辑前完成：

1. 读取工作区 `HANDOFF.md`、`REQUIREMENT_SOURCES.md` 与用户当前明确指定的 Scope，提取 Feature Goal、Confirmed Requirements、Global Constraints、Out of Scope、Target State 和 Acceptance Criteria。
2. 按 Handoff 当前目标读取 Technical Design、Feature Context（如有）和必要 Evidence Pointer，识别 Verified Hard Facts、Key Gaps、Technical Decisions、Compatibility Constraints、Relevant Code Areas、Known Risks、Open Issues 和建议的 Workstreams（如有）。
3. 检查仓库级指令、当前工作树状态和用户已有修改，避免覆盖或混入无关变更。
4. 重新阅读待修改区域及必要上下游调用链、数据流、SQL、配置和测试。
5. 验证决定实现边界的关键技术假设，不默认 Design Chat 一定正确。
6. 默认不重新通读全部原始 PRD / 原型 / 接口文档。只有以下情况才按 Evidence Pointer 选择性回查：
   - 相关 Hard Fact 为 `NOT VERIFIED` 或 `CONFLICT`；
   - Feature Context、Handoff 与 Current Repo 暴露出需求语义冲突；
   - 当前 Workstream 涉及新的高风险 Hard Fact，但 Design 阶段没有完成必要验证；
   - 缺失信息会实质改变业务行为、修改范围或验收结论。
7. 无法可靠确认且会改变业务行为的问题不得以猜测继续。

## Workstream 与执行边界

Suggested Implementation Workstreams 只是可选的 Context Organization 建议，不是 Task、Session、Context 或 Subagent 的运行时协议。简单 Feature 或无需隔离认知范围时，不要强行拆分。

实现期间始终保持以下映射：

- Current Workstream / Current Goal；
- Relevant Requirements；
- Relevant Hard Facts；
- Modification Scope；
- Acceptance Criteria。

存在多个 Workstream 时，可以由同一 Main Agent 按依赖关系连续实现，并在每个 Workstream 后执行必要的局部验证；不要求 Subagent，也不要求 Fresh Context。修改共享代码时，重新检查已完成和后续 Workstream 的交叉影响，并在最终阶段执行 Feature-level 验收。

Codex 自主决定搜索方式、工具顺序、Task 数量与顺序、是否隔离 Context、是否使用 Subagent 及其数量。本 Skill 不绑定一个 Context 必须处理多少 Workstream，也不把 Task Decomposition 等同于 Context Isolation 或 Subagent Delegation。

## Current Repo 与跨 Context 信息

Current Repo 是已实现技术事实的主要来源。后续 Context 不需要继承先前 Workstream 的完整聊天或工具历史；仅在代码无法完整表达且仍与后续实现相关时，携带 Requirement、Relevant Hard Fact ID、Evidence Pointer、Constraint、Intentional Design Decision、Compatibility Contract、Intentional Design Deviation、Dependency 和 Open Issue。

不要为此建立 Controller State、Worker History、Task State Database、Session Registry 或 Execution Event Log。

## 实现规则

只做完成 Target State 所需的最小必要修改：

- 不扩大 Scope，不新增 Requirement Authority 未要求的行为。
- 不进行无关重构、抽象、改名、格式化或顺手优化。
- 不修改接口协议、响应结构、数据库结构、枚举定义或既有业务语义，除非需求明确要求。
- 允许扩大只读分析范围以确认影响点，但不得静默扩大修改范围。
- 保留用户已有修改；只编辑已确认属于本 Feature 的文件和代码区域。
- 复用当前项目成熟模式，避免为架构美观引入新框架或间接层。
- 遵守当前仓库和环境已生效的注释、日志、SQL 等工程 Guardrails；本 Skill 只检查这些约束是否在本 Feature 修改范围内得到满足，不重复定义具体风格。
- 对 Relevant Hard Facts 保留精确判断语义。状态白名单 / 黑名单、枚举、金额公式、阈值、AND / OR / NOT 条件不得实现成含糊的“有效状态”“金额合法”等宽泛逻辑。
- 若完成任务必须扩大修改范围，先说明原因、影响和不可替代性；需要新增权限或改变需求时停止并请求用户确认。

不要在业务仓库创建 `FEATURE_CONTEXT.md`、`TECHNICAL_DESIGN.md`、`HANDOFF.md`、`PLAN.md` 或其他 Feature 临时文件。正式需求的这些资料只写入外部 Feature 工作区；业务仓库内的正式交付文档仍需用户另行明确授权。

## Review Fix Mode

用户提供 Review Finding 时，逐条执行：

1. 沿真实代码调用链验证 Finding 是否成立及其触发条件。
2. 对照 Requirement Working Context、Verified Hard Facts 和必要的 Requirement Evidence 验证 Finding 是否符合需求目标和修改范围。
3. Finding 不成立时不修改，并给出代码证据。
4. Finding 与 Feature Context 冲突时，不得仅因为 Feature Context 就拒绝修复；先检查 Verified Hard Facts / Evidence Pointer。若明确 Requirement Evidence 支持 Finding，按证据处理并报告 Feature Context 偏差。
5. 权威 Requirement Evidence 彼此冲突时不机械修复，标记 NOT VERIFIED 并请求确认。
6. Finding 成立时只做最小必要修复，避免把局部问题扩展成重构。
7. 修复后重新检查 Target State、Verified Hard Facts、Out of Scope 和相关回归路径。

不得以“清零所有 Finding”代替“正确完成 Requirement Authority”。

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
- Relevant Hard Facts 的实现覆盖检查，尤其是会直接改变允许 / 禁止、状态、金额、权限和审批结果的条件。

不得把未执行、超时、被基线错误阻塞或仅静态检查描述为“验证通过”。明确记录命令、结果、失败原因和未覆盖范围。

## 完成前检查

重新逐项对照 Confirmed Requirements、Verified Hard Facts、Global Constraints、Target State、Acceptance Criteria 和 Out of Scope，确认：

- 每个必要 Gap 都已实现或明确标记未完成。
- 每个 Relevant Hard Fact 都能映射到实际代码行为和必要验证；若存在 Hard Fact ID，Business Rule Coverage 中应保留该 ID。
- 修改范围内没有无关文件或意外语义变化。
- 接口、数据库和兼容行为的变化均有需求授权。
- 必要注释、日志和测试完整。
- 与 Handoff 的偏差有真实代码依据、已明确记录且未改变业务目标。
- 每个适用 Workstream 的 Requirement、Relevant Hard Facts、Modification Scope 和 Acceptance Criteria 均已闭环，并完成跨 Workstream 集成检查。

存在重要需求或高风险 Hard Fact 未完成、冲突或未验证时，不得声明 Feature 已完成。

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

`Business Rule Coverage` 应优先按 Relevant Hard Fact ID / Requirement 映射“需求事实 → 实现证据 → 测试或验证证据 → 结论”，避免只输出概括性完成描述。

同时明确是否改变接口、数据库和已有业务语义，是否需要补充测试，以及所有不确定或未验证项。

交付前必须把实际修改、设计偏差、代码基线、验证命令与结果、未验证范围、阻塞项和唯一下一动作同步到工作区 `HANDOFF.md`。只保留当前有效状态；过程日志和废弃方案不得累计进入 Handoff。
