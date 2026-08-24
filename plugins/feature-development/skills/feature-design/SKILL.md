---
name: feature-design
description: 用于正式软件 Feature 的需求资料保真、代码调查和技术设计阶段。在外部 Feature 工作区保存 Requirement Evidence、当前有效技术设计和滚动 Handoff；仅在确有压缩价值时生成 Feature Context。结合当前代码形成 Current State、Target State、Gap Analysis、Technical Design 和 Implementation Plan，第一轮不修改业务代码。
---

# Feature Design

## 顶层目标

始终以“正确完成 Requirement Authority 定义的需求”为顶层目标。Feature Context 是需求侧压缩后的 Working Context 与 Evidence Index，不是高于原始 Requirement Evidence 的独立事实源。技术设计只决定 HOW，不得替代或重写 Scope、Relevant Business Rules、Target State、Out of Scope 或已确认业务边界。

本 Skill 只负责调查与设计，不自动进入实现或 Review 阶段，不自动编排 Subagent，不在业务仓库持久化 Feature 临时文档，也不规定后续执行使用的 Task、Session、工具或 Agent Loop。正式需求的 Requirement Evidence、Technical Design 和 Handoff 必须写入业务仓库之外的 Feature 工作区。

## Feature 工作区

处理正式需求时，先完整读取并遵守 [Feature Workspace Contract](../../references/feature-workspace-contract.md)。使用插件 [Feature Workspace 模板](../../assets/feature-workspace/) 创建或复用需求目录：

1. Feature 工作区逻辑根目录使用 `~/Documents/Codex/features/`。实际文件操作前按当前 WSL 用户解析为绝对路径；不得写死用户名、`/home/<user>` 或 `/mnt/c/Users/<user>` 等单机路径。Windows 路径仅在用户明确要求导出或定位 Windows 文件时按需转换。
2. 在压缩、总结或设计前，先将 Chat 原文逐字保存并原样复制可用附件，完成 `REQUIREMENT_SOURCES.md`。
3. 仅在契约规定的多来源、长文本或复杂规则场景创建 `FEATURE_CONTEXT.md`；单一清晰需求直接使用原始 Evidence。
4. 将当前有效设计写入 `TECHNICAL_DESIGN.md`，并初始化或更新唯一的滚动 `HANDOFF.md`。
5. 外部工作区写入不改变第一轮对业务代码、配置、数据库和业务仓库文件的只读边界。

## Requirement Authority 与事实优先级

按以下规则处理信息：

1. Requirement Authority 来自当前有效的 Requirement Evidence，包括用户最新明确确认、当前有效 PRD / 原型 / 产品补充说明，以及第三方接口文档在其接口契约职责范围内的定义。
2. Feature Context 是从 Requirement Evidence 派生的 Working Context 与 Evidence Index。默认优先读取并使用 Feature Context 降低 Context 成本，但不得因为 Feature Context 已压缩就假定其一定完整。
3. Feature Context 与明确的 Requirement Evidence 不一致时，将其视为“派生上下文与证据不一致”，按 Evidence 修正需求理解并显式报告；不要为了迁就 Feature Context 忽略原始证据。
4. 多个有权威性的 Requirement Evidence 彼此冲突，且没有用户后续确认解决冲突时，标记为 Requirement Conflict / NOT VERIFIED，不自行取舍或创造折中规则。
5. Feature Context 中“用户确认”的标签本身不是独立 Evidence。只有当任务中存在可定位的用户确认原文、确认记录或明确 Evidence Pointer 时，才将该事项视为已验证的用户确认；不得因为 Feature Context 自己标记为“用户确认”而形成循环证明，也不得把确认影响范围扩大到相邻规则。
6. 当前代码、配置和数据链路是 Current State 的事实源；不得用当前实现反向定义业务规则。
7. 已有 Technical Design、Implementation Plan 和历史讨论仅作为技术线索；代码证据推翻技术假设时可调整 HOW，但不得借此改变 Requirement Authority。

对所有关键结论区分“已确认需求事实”“已确认代码事实”“推测”“未确认项”。无可靠证据时不得把推测写成事实。

## Hard Fact 与选择性 Evidence Verification

Hard Fact 指一旦丢失、改写或合并就可能直接改变“允许 / 禁止、通过 / 拒绝、状态流转、金额结果、兼容行为或接口契约”的需求事实。至少包括：

- 状态码、状态白名单 / 黑名单、枚举值；
- 金额公式、比例、阈值、精度、舍入规则；
- 权限、数据范围、审批条件；
- 幂等、并发、唯一性条件；
- 历史兼容、存量数据边界；
- 第三方接口字段、必填条件、枚举、错误码、调用约束；
- 明确的 MUST / MUST NOT / ONLY 语义；
- 会改变结果的 AND / OR / NOT 判断关系。

Feature Design 开始时：

1. 完整读取 Feature Context 或等价需求上下文，识别 Feature Goal、Scope、Relevant Business Rules、Target State、Out of Scope、Acceptance Criteria、已列出的 Hard Facts 和 Evidence Pointers。
2. **不要只验证 Feature Context 已经列出的 Hard Facts。** 当原始 Requirement Evidence 可用时，先做一次低成本 `Hard Fact Discovery Scan`，用于发现被 Context Compression 整体漏掉的关键规则：
   - 优先搜索状态/枚举、金额/公式/比例/阈值、权限/审批、幂等/并发、历史兼容、接口契约、`必须/不得/仅/只能`、AND / OR / NOT，以及 Acceptance Criteria / 状态矩阵等高风险信号；
   - 先 search / find，再只读取命中位置及必要上下文；不要因此通读所有原始资料；
   - 将 Evidence 中发现的 Hard Facts 与 Feature Context 对照。Evidence 明确存在但 Feature Context 缺失的，标记为 `Context Gap`，补入本轮 Verified Hard Facts，不能因为 Feature Context 没写就忽略。
3. 对 Feature Context 已列出的 Hard Facts，以及 Discovery Scan 新发现且与本 Feature 实现方向直接相关的高风险 Hard Facts，做 Selective Evidence Verification。优先按 Evidence Pointer 或搜索命中位置回查对应原始片段。
4. STATE / ENUM、AMOUNT / FORMULA / THRESHOLD、PERMISSION、APPROVAL、IDEMPOTENCY / CONCURRENCY、需求明确要求的 COMPATIBILITY、EXTERNAL API CONTRACT，以及关键 AND / OR / NOT 条件，默认属于应验证的高风险类型。
5. 仅在 Evidence Pointer 缺失、Discovery Scan 命中不完整、证据冲突、上下文不足、Feature Context 与代码现实明显矛盾，或高风险判断仍无法可靠完成时扩大 Evidence 回查范围。
6. 对核验后的 Hard Fact 记录简短 ID、精确 Fact、Evidence Pointer 和 Verification Status：`VERIFIED` / `CONFLICT` / `NOT VERIFIED`。若该事实由 Discovery Scan 发现且 Feature Context 原先缺失，同时标记 `Context Gap: MISSING_FROM_FEATURE_CONTEXT`。不要建立外部状态机或任务数据库。

若缺失的信息会实质改变业务行为、修改范围或验收结论，将其列为 Open Question；不要自行补充业务规则。若信息足以开展只读调查，先继续调查并明确剩余不确定项。

## 调查流程

严格先调查，再设计：

1. 搜索并定位与 Scope 相关的真实入口，不因文件名、类名、方法名或局部 grep 结果直接下结论。
2. 阅读入口实现并沿调用链向上下游追踪，验证实际触发路径、决定结果的关键分支以及最终结果位置。
3. 追踪关键参数、状态、金额、权限和结果的数据流，包括创建、转换、持久化、读取、回调和消费位置。
4. 按实际影响检查数据库、SQL、事务、缓存、RPC、MQ、Job、异步任务和外部接口；不机械扩大无关范围。
5. 检查相关测试、配置开关、兼容分支和现有成熟实现模式。
6. 记录关键代码证据位置，并区分正常路径、异常路径、边界场景和未覆盖场景。

### Evidence Closure Gate

“已经找到一条能解释现象的证据”不等于“关键 Current State 已经闭环”。

对会改变 Technical Design、根因判断、关键 Gap、状态/金额/权限结果或高风险兼容行为的结论，在停止调查并进入设计前至少确认：

1. 与当前问题相关的真实入口或实际触发路径；
2. 决定结果的关键分支 / 条件；
3. 关键数据或状态从生产、转换到最终结果位置的路径；
4. 当前主要解释；
5. 至少检查一个最可能推翻当前解释的竞争性假设；若不存在合理竞争性假设，明确说明原因；
6. 未验证且可能改变结论的事项保留为 Open Question / NOT VERIFIED。

单个 grep 命中、单一调用点、单条日志、单个 SQL 片段或一条能够解释现象的调用路径，均不能单独构成上述关键结论的调查闭环。

被明确证据排除且后续阶段重新采用会改变设计或实现方向的主要假设，记录为 `Rejected Hypothesis`，至少保留：Hypothesis、Rejected By、Evidence、Reopen If。不要记录所有搜索失败、试错或低价值猜想。

只有会改变 Technical Design 的关键 Current State 结论已经通过上述 Closure Gate，并取得可定位的代码证据，且对应 Target State 已有 Requirement Evidence 支持时，才进入设计。Review Finding、Issue 描述、历史设计、字段名和先前结论只能作为调查线索；证据不足的部分必须保留为 Open Question，不得先形成确定方案再补证据。

新证据或用户纠正推翻核心技术前提时，立即将依赖该前提的 Gap、Technical Design 和 Implementation Plan 视为失效，返回相关调用链重新调查并重建受影响范围的 Current State、Target State 与 Gap Analysis。不得只修补被指出的局部错误后继续沿用旧方案骨架。

第一轮只允许调查、设计和写入外部 Feature 工作区。不得修改业务代码、配置、数据库、接口协议或业务仓库文件；不得把 `FEATURE_CONTEXT.md`、`TECHNICAL_DESIGN.md`、`HANDOFF.md` 等 Feature 资料写入业务仓库。Chat 只输出便于用户核对的摘要和外部文档位置，不再承担唯一持久化载体。

## 形成设计

### Current State

基于真实代码证据说明当前系统实际上如何工作，包括相关入口、调用链、数据流、依赖、关键分支和已有验证。不要把需求目标描述成现状。

### Target State

从已经过必要 Evidence Verification 的 Requirement Working Context 中形成目标行为，逐项覆盖 Scope、Relevant Business Rules、Verified Hard Facts、Acceptance Criteria 和 Out of Scope。不得为了适配当前代码结构而重写需求。

### Gap Analysis

逐项列出 Current State 与 Target State 的具体差异。每个 Gap 应能映射到必要的代码或验证动作；不能确认的差异保留为 Open Question。Hard Fact 对应的缺口不得被抽象成宽泛描述，应保留决定结果的精确条件。

### Technical Design

设计完成 Target State 所需的最小必要方案：

- 优先复用当前项目已验证的成熟模式。
- 只调整与 Gap 直接相关的代码和配置。
- 不顺手重构、改名、抽象、优化或修复范围外问题。
- 不修改接口协议、数据库结构或既有业务语义，除非 Requirement Authority 明确要求。
- 明确关键分支、状态流转、数据一致性、兼容性、失败处理、日志和测试策略。
- 技术假设与代码事实冲突时，修正技术方案并说明依据；业务事实不完整时停止创造方案并列出 Open Question。

### Implementation Plan

按依赖顺序给出实现 Target State 所需的工程步骤。每步说明目标、预期影响模块或文件、关键约束、Relevant Hard Facts 和对应验证，保持修改范围最小。Plan 只描述工程动作及依赖关系，不绑定 Task、Session、Context、Subagent 或具体工具。

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
- Relevant Hard Facts；
- Relevant Design Decisions；
- Scope；
- Dependencies；
- Acceptance Criteria。

不要为 Workstream 设计状态机、任务数据库、控制器、心跳、重试、会话路由或“一 Workstream 一 Task / Session / Context / Subagent”的强制规则。

### Context Boundary

- Task 边界不等于 Session 边界；设计阶段的信息量可以较大，但交付给实现阶段的 Handoff 应保持小而完整；
- 仅当阶段切换、上下文已被大量无效检索或调试污染、过时方案难以剥离、目标持续被稀释时，才建议使用 Fresh Context；不得把新建 Chat 设为固定流程；
- Workstream 边界不等于 Context 边界。是否隔离上下文由认知负载决定，不由文件数量或模块数量机械决定；
- Handoff 只压缩仍然有效的事实、决策、约束、验收契约以及少量高价值 Rejected Hypotheses，不携带过程噪音；
- 与实现直接相关的 Hard Facts 必须保留精确语义和 Evidence Pointer，不能再次压缩成“状态满足规则”“金额符合要求”等失去决策条件的描述。

## 输出要求

先输出以下调查与设计结果：

1. Requirement Context 理解、Hard Fact Discovery / Evidence Verification、Context Gaps 与事实冲突
2. Current State
3. Target State
4. Gap Analysis
5. Technical Design
6. Implementation Plan
7. Suggested Implementation Workstreams（仅在确有必要时）
8. Rejected Hypotheses（仅保留会显著改变后续判断的主要已排除假设）
9. Risks / Open Questions
10. Validation Strategy

以上当前有效内容必须同步写入 Feature 工作区的 `TECHNICAL_DESIGN.md`。若新证据推翻核心前提，先在 Technical Design 和 Handoff 的 `Invalidated Premises` 中记录失效前提、影响范围、替代结论与证据，再重建受影响内容；不得覆盖原始 Requirement Evidence。

最后必须输出独立章节 `# IMPLEMENTATION_HANDOFF`。该章节供后续 Implementation Context 直接使用，不要求必须新建 Chat；必须自包含、紧凑、可执行，只保留已确认且与实现直接相关的信息，并包含：

- `## Feature Goal`
- `## Confirmed Requirements`
- `## Verified Hard Facts`
- `## Evidence Pointers`
- `## Global Constraints`
- `## Out of Scope`
- `## Current State Summary`
- `## Target State`
- `## Key Gaps`
- `## Technical Decisions`
- `## Rejected Hypotheses`（仅高价值项，包含排除证据与 Reopen If）
- `## Compatibility Constraints`
- `## Relevant Code Areas`
- `## Suggested Implementation Workstreams`（仅在确有必要时）
- `## Feature-level Acceptance Criteria`
- `## Known Risks`
- `## Open Issues`

`Verified Hard Facts` 仅保留与实现直接相关的高风险事实；每项至少包含 ID、精确 Fact、Verification Status 和对应 Evidence Pointer。若事实来自 Evidence Discovery 且原 Feature Context 漏失，保留 `Context Gap` 标记。`Evidence Pointers` 只保存必要定位信息，不复制大段原始资料。

`Rejected Hypotheses` 不是调查历史。只保留曾是主要候选、已经有明确反证、后续重新采用会显著改变设计/实现方向的假设，并记录 `Hypothesis / Rejected By / Evidence / Reopen If`。没有此类假设时写 `None`。

Handoff 不得包含低价值或过程性的已推翻猜想、已废弃方案、中间讨论、原始检索、文件读取、工具调用、调试历史、冗长代码摘录、无关调查记录、聊天历史摘要或运行时编排指令。

Handoff 是设计结论和已验证 Requirement Context 的压缩载体，不替代 Requirement Evidence，也不替代当前代码。必须将本章节写入并维护为工作区根目录的 `HANDOFF.md` 当前态。后续实现先读取 Handoff 与 `REQUIREMENT_SOURCES.md`，再按当前目标选择性读取 Technical Design、Feature Context 或 Evidence Pointer 指向的原始片段；只有相关 Hard Fact 未验证、发生冲突、出现新的高风险歧义，或 Reopen If 条件成立时才扩大回查。
