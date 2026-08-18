---
name: feature-design
description: 用于已提供 Feature Context 或等价需求上下文的软件 Feature 代码调查和技术设计阶段。结合原始需求资料与当前代码，形成 Current State、Target State、Gap Analysis、Technical Design、Implementation Plan 和可复制到新 Chat 的 Implementation Handoff。默认只读调查，第一轮不修改业务代码。
---

# Feature Design

## 顶层目标

始终以“正确完成 Feature Context 定义的需求”为顶层目标。技术设计只决定 HOW，不得替代或重写 Scope、Relevant Business Rules、Target State、Out of Scope 或已确认业务边界。

本 Skill 只负责调查与设计，不自动进入实现或 Review 阶段，不自动编排 Subagent，不在业务仓库持久化 Feature 临时文档。

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

按依赖顺序给出可执行步骤。每步说明目标、预期影响模块或文件、关键约束和对应验证，保持修改范围最小。

## 输出要求

先输出以下调查与设计结果：

1. Feature Context 理解与事实冲突
2. Current State
3. Target State
4. Gap Analysis
5. Technical Design
6. Implementation Plan
7. Risks / Open Questions
8. Validation Strategy

最后必须输出独立章节 `# IMPLEMENTATION_HANDOFF`。该章节用于复制到全新的 Codex Chat，必须自包含、简洁，只保留已确认且与实现直接相关的信息，并包含：

- `## Feature Goal`
- `## Scope`
- `## Relevant Business Rules`
- `## Current State`
- `## Target State`
- `## Gap`
- `## Confirmed Technical Decisions`
- `## Expected Affected Modules / Files`
- `## Implementation Steps`
- `## Must-Not-Change Constraints`
- `## Risks / Open Questions`
- `## Validation Requirements`

Handoff 不得包含已废弃方案、中间讨论、已推翻假设、冗长代码摘录、无关调查记录或聊天历史摘要。
