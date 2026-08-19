---
name: feature-design
description: 基于 Feature Context 或等价需求上下文与当前代码完成软件 Feature 的代码调查和技术设计。默认只读，不修改业务代码。输出 Current State、Target State、Gap Analysis、Technical Design、Implementation Plan 和必要的 Open Questions。
---

# Feature Design

## Goal

正确理解需求和当前代码，给出可以落地的最小技术方案。

本 Skill 只负责调查与设计，不修改业务代码，不自动进入实现或 Review，也不规定 Codex 的搜索顺序、工具选择、Task、Context、Session 或 Subagent 策略。

## 事实来源

- 需求事实来自用户最新确认、当前有效 PRD / 原型 / 产品补充说明和第三方接口文档在其职责范围内的定义。
- Feature Context 是高密度需求上下文，默认优先使用；当它与明确的原始需求证据冲突时，以原始证据和用户最新确认处理，并报告冲突。
- 当前代码、配置和数据链路决定 Current State；不得用当前实现反向定义业务需求。
- Technical Design、历史讨论和已有方案只是技术线索，不是事实源。
- 未经验证的判断可以作为调查假设，但不得写成已确认事实。

对会直接改变结果的关键规则要保留精确语义，例如：状态白名单 / 黑名单、枚举、金额公式和阈值、权限和审批条件、幂等 / 并发约束、历史兼容边界、第三方接口契约、关键 AND / OR / NOT 条件。只有当这些规则缺失、冲突或会改变设计结论时，才回查对应原始证据；不要机械重读全部需求资料。

## 调查原则

先理解与当前 Feature 相关的真实代码，再形成设计。

根据任务需要检查入口、必要上下游调用、关键数据流、SQL、事务、缓存、RPC、MQ、Job、异步流程、外部接口、权限、配置和测试。只读取足以支持当前设计判断的上下文，不为满足流程形式穷举所有引用。

如果代码事实推翻已有技术假设，修正技术方案；如果缺失的需求信息会实质改变业务行为、修改范围或验收结论，列为 Open Question，不自行创造业务规则。

## 输出

### 1. Current State

说明当前系统实际上如何工作，只保留与本 Feature 相关的入口、关键调用链、数据流、重要分支和已有约束。引用必要的代码证据，不堆砌文件列表。

### 2. Target State

用清晰业务语言说明需求完成后系统应该如何工作。关键状态、金额、权限、接口和兼容规则必须保留精确条件。

### 3. Gap Analysis

逐项说明 Current State 与 Target State 的差异。每个 Gap 应能对应到后续必要的代码或验证动作；无法确认的内容放入 Open Questions。

### 4. Technical Design

给出实现 Target State 的最小必要方案：

- 优先复用项目已有成熟模式；
- 只修改与 Gap 直接相关的代码和配置；
- 不顺手重构、改名、抽象或优化；
- 不改变接口、数据库结构或既有业务语义，除非需求明确要求；
- 说明关键分支、数据一致性、失败处理、兼容性、日志和测试策略；
- 重点解释“为什么这样改”，不要用 Harness 术语代替工程描述。

### 5. Implementation Plan

按依赖顺序列出可执行步骤。每步只说明：目标、主要影响范围、关键约束、验证方式。简单 Feature 不强行拆分任务或 Workstream；复杂 Feature 如确实存在相对独立的业务闭环，可以建议拆分，但不绑定 Session、Context 或 Subagent。

### 6. Risks / Open Questions

只列真实存在且会影响实现或验收的风险与未确认项。

### 7. Validation Strategy

说明与变更风险匹配的验证方式，包括必要的静态检查、单元测试、构建、集成验证或关键业务场景。

## Implementation Handoff

当后续实现需要跨 Context 复用设计结果时，最后附一个精简 `# IMPLEMENTATION_HANDOFF`；如果当前会话会直接继续实现，也可以保持很短。

Handoff 只保留：

- Feature Goal
- Confirmed Requirements
- Critical Business Rules
- Out of Scope
- Current State Summary
- Target State
- Key Gaps
- Technical Decisions
- Relevant Code Areas
- Acceptance Criteria
- Open Issues

不要携带检索过程、工具调用、已废弃方案、冗长代码摘录、状态机式流程元数据或无关聊天历史。

## Writing Style

默认使用直接、易读的中文工程表达。优先写“当前代码做什么、需求要求什么、差在哪里、准备怎么改、为什么这样改”。除非确实需要区分事实来源或冲突，不要反复使用 Requirement Authority、Evidence Pointer、Working Context、Coverage、Workstream 等元流程术语。
