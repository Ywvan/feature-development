# Feature Workspace Contract

本契约是 `feature-development` 的 Feature 工作流、工作区和跨阶段状态唯一规范。它只定义 Feature 生命周期、Requirement Evidence 完整性和跨阶段状态，不重复维护通用工程行为规则。

## 1. 任务门禁

### Requirement Development

以下任务进入 Feature 工作流：

- 新增业务能力；
- 需求迭代；
- 修改既定业务规则、状态语义、金额口径、权限 / 审批规则或接口契约；
- 需要明确新的 Target State 与 Acceptance Criteria 的正式需求；
- 明确继续已有 Feature Workspace 的任务。

用户不需要显式说出 “Feature”。

### Bug Fix

目标是恢复已经存在或已经确定的正确行为时，仍属于 Bug Fix。读取 PRD、历史需求或产品规则确认预期行为，本身不会把 Bug 升级为 Feature。

以下任务默认不创建 Feature Workspace：

- 单服务 / 单仓库 Bug；
- 单点回归；
- 单个 Review Finding 修复；
- 纯诊断、根因排查；
- 不属于已有 Feature 的一次性只读 Review；
- 解释问答；
- 不改变业务行为的机械修改。

“单服务 / 多服务”按完成任务最终实际需要写入修改的可独立部署服务或仓库数量判断，不按只读调查读取了多少服务判断。

### Bug 升级为 Feature

Bug 满足以下任一条件时可以进入 Feature 工作流：

1. 必须新增或改变业务规则、接口契约、Target State 或 Acceptance Criteria；
2. 确实需要在两个或以上可独立部署服务 / 仓库中协同写入修改，并需要维护跨服务契约、发布顺序或跨阶段 Handoff；
3. 当前 Bug 属于已有正式 Feature Workspace；
4. 用户明确要求使用 Feature 工作流或持久化 Feature 资料。

任务性质或最终修改范围尚未确认时，不提前创建 Feature Workspace。确认满足门禁后再创建或复用。

## 2. 工作区路径

Feature Workspace 根目录按以下顺序解析：

1. `CODEX_FEATURES_DIR` 非空时，使用其解析后的绝对路径；
2. 否则 `CODEX_HOME` 非空时，使用其下的 `features/`；
3. 否则使用当前运行时解析的 `~/.codex/features/`。

不得在 Skill 中写死用户名、盘符、Windows / WSL / Linux 绝对路径，也不得手工在 Windows 路径与 `/mnt/<drive>` 之间转换。

显式配置的目录不可访问或不可写时应报告该状态，不静默切换到另一目录。

每个正式需求使用独立目录：

```text
YYYYMMDD-[需求编号-]需求简称/
```

只有需求身份和关联业务仓库一致时才复用现有目录。

## 3. 标准结构

```text
YYYYMMDD-[需求编号-]需求简称/
├── requirements/
│   ├── ORIGINAL_REQUEST.md
│   └── 原始附件或快照...
├── REQUIREMENT_SOURCES.md
├── FEATURE_CONTEXT.md       # 仅在有压缩价值时创建
├── TECHNICAL_DESIGN.md
├── HANDOFF.md
└── REVIEW_RESULT.md         # 独立 Review 后创建
```

初始化文档时使用 `assets/feature-workspace/` 下的对应模板。

## 4. Requirement Evidence

Requirement Evidence 保存原始需求事实：

- Chat 中的原始需求保存到 `requirements/ORIGINAL_REQUEST.md`；
- 本地附件原样复制到 `requirements/`；
- 可计算时记录并核对 SHA-256；
- 在线资料优先保存允许导出的原始快照；无法保存时登记 URL、访问时间、版本 / 更新时间和未落盘原因；
- `REQUIREMENT_SOURCES.md` 维护 Source ID、来源、版本、位置、状态和冲突；
- 新版本 Evidence 追加保存，旧来源标记 `SUPERSEDED`，不覆盖旧证据；
- 用户后续明确确认会改变当前有效需求时，将该确认作为新的 Evidence 登记。

Requirement Evidence 与用户最新确认用于确定需求事实。派生文档不能替代或静默改写原始 Evidence。

## 5. Hard Fact 与 Context Integrity

Hard Fact 指一旦丢失、改写或合并就可能直接改变允许 / 禁止、状态流转、金额结果、权限 / 审批、兼容行为、幂等 / 并发或接口契约的需求事实。

Feature Context 只在多来源需要归并、原始资料较长或关键业务规则复杂且存在明显上下文压缩价值时创建。每个持久化 Hard Fact 应保留：

- 简短 Fact ID；
- 精确 Fact；
- Evidence Pointer；
- `VERIFIED` / `CONFLICT` / `NOT VERIFIED`；
- 如由 Evidence 发现但派生 Context 原先缺失，标记对应 Context Gap。

关键词 search / find 只能用于定位 Evidence，不能作为需求完整性结论；没有命中关键词不能证明某项规则不存在。Design 和 Review 应使用各自 Skill 定义的低成本 Hard Fact Discovery / Integrity Scan，检查 Context Compression 是否整体漏掉会改变结果的规则。

## 6. Rejected Hypothesis 与 Invalidated Premise

`Rejected Hypothesis` 只用于跨阶段保存少量高价值、已经被直接反证的主要替代解释。只有同时满足以下条件才持久化：

1. 曾是当前问题的主要候选；
2. 若成立会实质改变设计、实现或 Review 结论；
3. 已实际检查，而非仅凭推测；
4. 存在直接 Falsifying Evidence；
5. Rejected Scope 明确；
6. `Reopen If` 具体且可观察。

搜索未命中、暂未发现证据或“当前没看到”不构成反证。多数替代解释无需跨阶段保存。

Handoff 中只保留：

- Hypothesis；
- Rejected Scope；
- Falsifying Evidence；
- Reopen If。

`Reopen If` 成立时，该排除结论立即失效。

`Invalidated Premise` 与 Rejected Hypothesis 不同：它表示某个曾经被当作成立事实用于设计或实现的前提，后来被新证据推翻。只保留会影响后续决策的失效前提、影响范围、替代结论 / 下一动作和 Evidence；旧结论不得继续留在当前有效决策区。

## 7. 派生文档职责

### FEATURE_CONTEXT.md

保存当前有效的 Feature Goal、Scope / Out of Scope、Hard Facts、Acceptance Criteria 与 Open Questions，并通过 Evidence Pointer 回指 Requirement Evidence。

### TECHNICAL_DESIGN.md

保存当前有效的：

- Requirement / Hard Fact Verification；
- Current State；
- Target State；
- Gap Analysis；
- Technical Design；
- Implementation Plan；
- Validation Strategy；
- Risks / Open Questions；
- 符合准入条件的少量 Rejected Hypotheses；
- Invalidated Premises。

Current State 回指当前代码，业务规则回指 Requirement Evidence。

### HANDOFF.md

Handoff 是跨阶段滚动当前态入口，保存：

- 当前 Stage / Status 与代码基线；
- Feature Goal / Confirmed Requirements；
- Verified Hard Facts 与 Evidence Pointers；
- Global Constraints / Out of Scope；
- Current State / Target State / Key Gaps；
- Active Technical Decisions；
- 符合准入条件的少量 Rejected Hypotheses；
- Modification State / Design Deviations；
- Validation Results / Unverified Scope；
- Blockers / Known Risks；
- Invalidated Premises；
- Next Action。

更新时替换已经过期的当前态，不累积工具日志、完整检索过程、聊天摘要、运行时编排信息、低价值被推翻猜想或冗长代码摘录。

### REVIEW_RESULT.md

保存最新一轮独立 Feature Review 的 baseline、Requirement Gate、Findings、Verification Evidence、Remaining Risks / Unverified Items、适用的少量 Rejected Hypotheses，以及上一轮结果为何失效的简短历史。

Review Result 不修改 Requirement Evidence。

## 8. 阶段边界

### Design

- 创建或识别已通过门禁的 Feature Workspace；
- 保存 Requirement Evidence；
- 执行 Feature-specific Hard Fact Discovery / Verification；
- 生成或更新 Technical Design 与 Handoff；
- `FEATURE_CONTEXT.md` 仅在有压缩价值时生成；
- 业务仓库、业务代码、配置和数据库保持只读。

### Implement

- 使用已有 Workspace；
- 从 Handoff 恢复当前阶段状态；
- 根据当前任务读取 Technical Design、Feature Context 或对应 Requirement Evidence；
- 保持 Relevant Hard Facts 与当前实现目标的映射；
- 实现当前有效 Target State 对应的 Gap；
- 在里程碑、验证结束和交付前更新 Handoff；
- 不改写原始 Requirement Evidence。

### Review

- 基于 Requirement Evidence、Verified Hard Facts、当前 Handoff、最终 diff、当前代码和验证证据独立重建 Feature 结论；
- 执行 Feature-specific Hard Fact Integrity Check；
- 业务仓库、生产代码、配置和数据库保持只读；
- 更新 `REVIEW_RESULT.md`；
- 将最新 Gate、阻塞项、验证状态、Rejected Hypotheses、Invalidated Premises 和下一动作同步到 Handoff。

外部 Feature Workspace 的文档写入不等于业务仓库写权限，也不自动扩大 Feature Scope。

## 9. 跨阶段读取与 Token 边界

新阶段默认先读取：

1. `HANDOFF.md`；
2. `REQUIREMENT_SOURCES.md`。

然后根据当前目标读取：

- `TECHNICAL_DESIGN.md`；
- `FEATURE_CONTEXT.md`；
- Evidence Pointer 指向的 Requirement Evidence；
- `REVIEW_RESULT.md`（进入 Rework 或复审时）。

不要因为资料已经落盘，就机械把完整 Chat、全部附件、所有设计版本和全部历史过程同时装入当前 Context。

Rejected Hypotheses 只保留满足准入条件的高价值反证；Invalidated Premises 只保留会改变后续决策的失效前提。二者都不得演变成完整调查历史。
