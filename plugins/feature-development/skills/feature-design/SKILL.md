---
name: feature-design
description: 用于已提供 Feature Context 或等价需求上下文的软件 Feature 代码调查和技术设计阶段。结合原始需求资料与当前代码，形成 Current State、Target State、Gap Analysis、Technical Design、Implementation Plan 和可供后续 Implementation Context 复用的精简 Implementation Handoff；仅在认知表面积明显较大时建议 Implementation Workstreams。默认只读调查，第一轮不修改业务代码。
---

# Feature Design

## 顶层目标

始终以“正确完成 Feature Context 定义的需求”为顶层目标。技术设计只决定 HOW，不得替代或重写 Scope、Relevant Business Rules、Target State、Out of Scope 或已确认业务边界。

本 Skill 只负责调查与设计，不自动进入实现或 Review 阶段，不自动编排 Subagent，不在业务仓库持久化 Feature 临时文档，也不规定后续执行使用的 Task、Session、工具或 Agent Loop。

## 事实优先级

按以下优先级处理信息：

1. 将 Feature Context 作为最高业务事实源。
2. 使用 PRD、原型、接口文档和产品补充说明提供需求证据；它们与 Feature Context 冲突时，明确列出冲突并请求确认，不自行取舍或创造折中规则。
3. 将当前代码、配置和数据链路作为 Current State 的事实源；不得用当前实现反向定义业务规则。
4. 将已有 Technical Design、Implementation Plan 和历史讨论仅作为技术线索；代码证据推翻技术假设时可调整 HOW，但不得借此改变业务事实。

对所有关键结论标记为“已确认代码事实”“推测”或“未确认项”。无可靠证据时不得把推测写成事实。

## 输入检查

先完整读取 Feature Context 或等价需求上下文，并识别 Feature Goal、Scope、Relevant Business Rules、Target State、Out of Scope、Acceptance Criteria 和已确认边界。继续读取用户提供的 PRD、原型、接口文档与补充说明。

若缺失的信息会实质改变业务行为、修改范围或验收结论，将其列为 Open Question 并请求用户确认；不要自行补充业务规则。若信息足以开展只读调查，先继续调查并明确剩余不确定项。

## 调查流程

严格先调查，再设计：

1. 搜索并定位与 Scope 相关的真实入口，不因文件名、类名、方法名或局部 grep 结果直接下结论。
2. 阅读入口实现并沿调用链向上下游追踪，直到证据足以解释实际业务行为。
3. 追踪关键参数、状态、金额、权限和结果的数据流，包括创建、转换、持久化、读取、回调和消费位置。
4. 按实际影响检查数据库、SQL、事务、缓存、RPC、MQ、Job、异步任务和外部接口；不机械扩大无关范围。
5. 检查相关测试、配置开关、兼容分支和现有成熟实现模式。
6. 记录关键证据位置，并区分正常路径、异常路径、边界场景和未覆盖场景。

第一轮只允许只读调查和设计。不得修改业务代码、配置、数据库、接口协议或业务仓库文件；不得创建 `FEATURE_CONTEXT.md`、`TECHNICAL_DESIGN.md`、`PLAN.md` 等临时文件。所有设计与 Handoff 默认直接输出在当前 Chat。

## 形成设计

### Current State

基于真实代码证据说明当前系统实际上如何工作，包括相关入口、调用链、数据流、依赖、关键分支和已有验证。不要把需求目标描述成现状。

### Target State

从 Feature Context 抽取完成后的目标行为，逐项覆盖 Scope、Relevant Business Rules、Acceptance Criteria 和 Out of Scope。不得为了适配当前代码结构而重写需求。

### Gap Analysis

逐项列出 Current State 与 Target State 的具体差异。每个 Gap 应能映射到必要的代码或验证动作；不能确认的差异保留为 Open Question。

### Technical Design

设计完成 Target State 所需的最小必要方案：

- 优先复用当前项目已验证的成熟模式。
- 只调整与 Gap 直接相关的代码和配置。
- 不顺手重构、改名、抽象、优化或修复范围外问题。
- 不修改接口协议、数据库结构或既有业务语义，除非 Feature Context 明确要求。
- 明确关键分支、状态流转、数据一致性、兼容性、失败处理、日志和测试策略。
- 技术假设与代码事实冲突时，修正技术方案并说明依据；业务事实不完整时停止创造方案并列出 Open Question。

### Implementation Plan

按依赖顺序给出实现 Target State 所需的工程步骤。每步说明目标、预期影响模块或文件、关键约束和对应验证，保持修改范围最小。Plan 只描述工程动作及依赖关系，不绑定 Task、Session、Context、Subagent 或具体工具。

### Suggested Implementation Workstreams

仅当 Feature 的认知表面积明显较大，且按相对独立的工程能力组织上下文能降低实现阶段的理解成本时，才输出 Suggested Implementation Workstreams。简单 Feature 不要为了形式完整强行拆分。

必须保持以下概念独立：

- Task Decomposition 不等于 Context Isolation；
- Context Isolation 不等于 Subagent Delegation；
- Workstream 是上下文组织建议，不是运行时对象，也不代表必须创建 Task、Session、Context 或 Subagent；
- Workstream 应对应相对独立的工程能力或业务闭环，不得按 DTO、Mapper、Service、单个文件等机械分层。

每个建议的 Workstream 仅包含必要信息：

- ID / Name；
- Goal；
- Relevant Requirements；
- Relevant Design Decisions；
- Scope；
- Dependencies；
- Acceptance Criteria。

不要为 Workstream 设计状态机、任务数据库、控制器、心跳、重试、会话路由或“一 Workstream 一 Task / Session / Context / Subagent”的强制规则。

### Context Boundary

- Task 边界不等于 Session 边界；设计阶段的信息量可以较大，但交付给实现阶段的 Handoff 应保持小而完整；
- 仅当阶段切换、上下文已被大量无效检索或调试污染、过时方案难以剥离、目标持续被稀释时，才建议使用 Fresh Context；不得把新建 Chat 设为固定流程；
- Workstream 边界不等于 Context 边界。是否隔离上下文由认知负载决定，不由文件数量或模块数量机械决定；
- Handoff 只压缩仍然有效的事实、决策、约束和验收契约，不携带过程噪音。

## 输出要求

先输出以下调查与设计结果：

1. Feature Context 理解与事实冲突
2. Current State
3. Target State
4. Gap Analysis
5. Technical Design
6. Implementation Plan
7. Suggested Implementation Workstreams（仅在确有必要时）
8. Risks / Open Questions
9. Validation Strategy

最后必须输出独立章节 `# IMPLEMENTATION_HANDOFF`。该章节供后续 Implementation Context 直接使用，不要求必须新建 Chat；必须自包含、紧凑、可执行，只保留已确认且与实现直接相关的信息，并包含：

- `## Feature Goal`
- `## Confirmed Requirements`
- `## Global Constraints`
- `## Out of Scope`
- `## Current State Summary`
- `## Target State`
- `## Key Gaps`
- `## Technical Decisions`
- `## Compatibility Constraints`
- `## Relevant Code Areas`
- `## Suggested Implementation Workstreams`（仅在确有必要时）
- `## Feature-level Acceptance Criteria`
- `## Known Risks`
- `## Open Issues`

Handoff 不得包含已废弃方案、中间讨论、已推翻假设、原始检索、文件读取、工具调用、调试历史、冗长代码摘录、无关调查记录、聊天历史摘要或运行时编排指令。

Handoff 是设计结论的压缩载体，不替代当前代码。后续实现必须结合 Handoff 与实现时的 Current Repo 重新核对关键事实。
