# Feature Workspace Contract

本契约是 `feature-development` 的 Feature 工作流、工作区和跨阶段状态唯一规范。它只定义 Feature 生命周期，不重复维护通用工程行为规则。

## 1. 任务门禁

### Requirement Development

以下任务进入 Feature 工作流：

- 新增业务能力；
- 需求迭代；
- 修改既定业务规则、状态语义、金额口径、权限 / 审批规则或接口契约；
- 明确提出新的 Target State 与 Acceptance Criteria 的正式需求；
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

“单服务 / 多服务”按完成任务最终确认会写入修改的可独立部署服务或仓库数量判断，不按只读调查读取了多少服务判断。

### Bug 升级为 Feature

Bug 满足以下任一条件时可以进入 Feature 工作流：

1. 当前修复目标包含新增或改变业务规则、接口契约、Target State 或 Acceptance Criteria；
2. 当前修复方案包含两个或以上可独立部署服务 / 仓库的协同写入修改，且这些修改之间存在跨服务契约、发布顺序或跨阶段 Handoff 依赖；
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
- `REQUIREMENT_SOURCES.md` 维护来源、版本、位置、状态和冲突；
- 新版本 Evidence 追加保存，旧来源标记 `SUPERSEDED`，不覆盖旧证据；
- 用户后续明确确认会改变当前有效需求时，将该确认作为新的 Evidence 登记。

Requirement Evidence 与用户最新确认用于确定需求事实。派生文档不能替代或静默改写原始 Evidence。

## 5. 派生文档职责

### FEATURE_CONTEXT.md

仅在多来源需要归并、原始资料较长或关键业务规则复杂且有明显上下文压缩价值时创建。

保存当前有效的 Feature Goal、Scope / Out of Scope、Critical Business Rules、Acceptance Criteria 与 Open Questions，并通过 Evidence Pointer 回指 Requirement Evidence。

### TECHNICAL_DESIGN.md

保存当前有效的：

- Current State；
- Target State；
- Gap Analysis；
- Technical Design；
- Implementation Plan；
- Validation Strategy；
- Risks / Open Questions；
- Invalidated Premises。

Current State 回指当前代码，业务规则回指 Requirement Evidence。

### HANDOFF.md

保存跨阶段继续工作所需的当前有效状态：

- Feature Goal / Confirmed Requirements；
- Critical Business Rules / Out of Scope；
- 当前代码基线；
- Current State / Target State / Key Gaps；
- Current Technical Decisions；
- Modification State / Design Deviations；
- Validation Results / Unverified Scope；
- Risks / Blockers；
- Invalidated Premises；
- Next Action。

`Invalidated Premises` 只记录“后续重新采用后会改变决策”的失效技术前提，通常应保持为 0～3 条；不保存所有调查假设、搜索过程或已废弃方案。

Handoff 是滚动当前态。更新时替换已过期状态，不累积工具日志、完整检索过程、聊天摘要、运行时编排信息或冗长代码摘录。

### REVIEW_RESULT.md

保存最新一轮独立 Feature Review：

- Review baseline / final diff；
- READY / NOT READY；
- 当前 Findings；
- Verification Evidence；
- Remaining Risks / Unverified Items；
- 上一轮结果为何失效的简短历史。

Review Result 不修改 Requirement Evidence。

## 6. 阶段边界

### Design

- 创建或识别已通过门禁的 Feature Workspace；
- 保存 Requirement Evidence；
- 生成或更新 Technical Design 与 Handoff；
- `FEATURE_CONTEXT.md` 仅在有压缩价值时生成；
- 业务仓库、业务代码、配置和数据库保持只读。

### Implement

- 使用已有 Workspace；
- 从 Handoff 恢复当前阶段状态；
- 实现当前有效 Target State 对应的 Gap；
- 在验证结束和交付前更新 Handoff；
- 不改写原始 Requirement Evidence。

### Review

- 基于 Requirement Evidence、当前 Handoff、最终 diff、当前代码和验证证据独立重建 Feature 结论；
- 业务仓库、生产代码、配置和数据库保持只读；
- 更新 `REVIEW_RESULT.md`；
- 将最新 Gate、阻塞项、验证状态、Invalidated Premises 和下一动作同步到 Handoff。

外部 Feature Workspace 的文档写入不等于业务仓库写权限，也不自动扩大 Feature Scope。

## 7. 跨阶段读取

新阶段默认先读取：

1. `HANDOFF.md`；
2. `REQUIREMENT_SOURCES.md`。

然后根据当前目标读取：

- `TECHNICAL_DESIGN.md`；
- `FEATURE_CONTEXT.md`；
- Evidence Pointer 指向的 Requirement Evidence；
- `REVIEW_RESULT.md`（进入 Rework 或复审时）。

不要因为资料已经落盘，就机械把完整 Chat、全部附件、所有设计版本和全部历史过程同时装入当前 Context。

当 `HANDOFF.md` 的 `Invalidated Premises` 指向一个会影响当前决策的失效前提时，应读取对应 Technical Design / Evidence 位置确认其失效原因，而不是重新采用该前提。
