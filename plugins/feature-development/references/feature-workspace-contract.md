# Feature Workspace Contract

本契约只适用于需要持久化跨阶段上下文的正式工作：新增 Feature、需求迭代、业务规则变更，以及确实需要多个可独立部署服务或多个仓库协同修改的复杂 Bug。普通单服务 Bug 排查与修复不创建 Feature 工作区。

## 任务识别与工作区创建门禁

创建或读取 Feature 工作区前，先判断当前任务属于 Requirement Development 还是 Bug Fix。**不得仅因为修复会改变当前业务结果，就把 Bug 自动升级为 Feature。** 如果当前行为本身就是缺陷，恢复既有正确行为仍然属于 Bug Fix。

如果一个任务同时具有“Bug 表现”和“需求资料”，以**任务目标**而不是资料类型或标题判断：目标是恢复已经存在 / 已经确定的正确行为时，仍按 Bug Fix；只有目标需要新增或改变产品应有语义时，才按 Requirement Development。读取 PRD、历史需求或产品规则来确认 Bug 的预期行为，本身不会把 Bug 升级为 Feature。

### Requirement Development

满足以下语义时，优先识别为需求开发并使用 Feature 工作区：

- 用户提供或引用 PRD、正式需求、原型、产品规则、Acceptance Criteria，并要求新增、调整或实现目标行为；
- 任务目标是新增业务能力、修改既定业务规则、扩展状态 / 金额 / 权限 / 审批 / 接口契约等目标语义；
- 需要先明确新的 Target State，再基于 Current State 设计和实现；
- 已存在对应 Feature 工作区，当前任务明确是在继续该正式需求。

需求开发不要求用户必须说出“Feature”字样；应根据任务目标和需求资料自动识别。

### Bug Fix

满足以下语义时，优先识别为 Bug Fix：

- 用户描述失败、异常、报错、回归、结果不正确、线上问题或已有功能没有按预期工作；
- 目标是定位根因并恢复已经存在或已经确定的正确行为，而不是新增产品规则；
- 正确行为能够从现有产品语义、当前有效需求、历史兼容约束或成熟实现中确认。

以下单服务 Bug 默认**不创建、不初始化、不补齐** Feature 工作区，也不生成 `REQUIREMENT_SOURCES.md`、`FEATURE_CONTEXT.md`、`TECHNICAL_DESIGN.md`、`HANDOFF.md` 或 `REVIEW_RESULT.md`：

- 单服务 / 单仓库线上 Bug；
- 单点回归；
- 单个 Review Finding 的修复；
- 已有条件、状态、金额、SQL、权限或异常处理实现错误；
- 纯诊断、根因排查和一次性修复。

“单服务 / 多服务”按**完成修复实际需要写入修改的可独立部署服务或仓库数量**判断，不按调查读取范围判断。为了确认调用链、接口契约或数据流而只读检查多个服务，不会把单服务 Bug 升级成多服务 Feature 工作流。

### Bug 升级为 Feature 工作流

Bug 只有满足以下任一条件时才允许升级并创建 Feature 工作区：

1. 根因调查确认不能只恢复既有行为，而是必须新增或改变业务规则、接口契约、Target State 或 Acceptance Criteria；
2. 完成修复确实需要在两个或以上可独立部署服务 / 仓库中协同写入修改，并需要维护跨服务契约、发布顺序或跨阶段 Handoff；
3. 当前 Bug 明确属于一个已经存在的正式 Feature 工作区，需要继续该 Feature；
4. 用户明确要求使用 Feature 工作流或持久化设计 / Handoff。

如果任务性质或最终修改范围尚不确定，**默认先不创建工作区**，只做足以判断根因和修改边界的只读调查。只有取得满足上述升级条件的证据后才创建或复用 Feature 工作区。不得为了“可能以后会复杂”提前生成文件。

纯诊断、解释问答、不属于既有 Feature 的一次性只读 Review、不改变业务行为的机械修改，以及未满足升级条件的单服务 Bug 均不强制也不建议创建工作区。

## 路径与目录

- Feature 工作区根目录不得写死为任何固定的 Windows、WSL 或 Linux 绝对路径，也不得假定固定用户名、盘符、`/home/<user>`、`/mnt/<drive>`、`~/Documents` 等机器目录结构。
- 根目录必须在运行时按以下优先级解析：
  1. 若环境变量 `CODEX_FEATURES_DIR` 已设置且非空，使用其解析后的绝对路径；这是显式覆盖项。
  2. 否则若 `CODEX_HOME` 已设置且非空，在其下使用 `features/`。
  3. 否则使用 `~/.codex/features/`；其中 `~` 必须由当前 OS / Runtime 解析为当前用户 home，禁止自行拼接用户名、盘符或 WSL 挂载路径。
- 默认 fallback 的实际结果示例：Windows 原生通常为 `%USERPROFILE%\.codex\features\`，WSL / Linux 通常为 `~/.codex/features/`。这些只是运行时解析结果示例，不得把示例中的用户名、盘符或绝对路径写入 Skill。
- 所有路径拼接、标准化和绝对路径解析都使用当前平台原生路径语义或标准路径 API；不得手工把 Windows 路径改写成 `/mnt/c/...`，也不得把 WSL / POSIX 路径强行改写成 `C:\...`。
- Windows 原生、WSL 和其他 POSIX 环境都遵守同一解析顺序。若用户希望 Windows 与 WSL 指向同一个物理工作区，应通过各自环境中的 `CODEX_FEATURES_DIR` 显式指定等价位置，而不是在 Skill 中推断 Windows 用户名、盘符或挂载点。
- 显式设置的 `CODEX_FEATURES_DIR` 或 `CODEX_HOME` 无法解析、不可访问或不可写时，应明确报告并停止写入，不得静默回退到另一个目录导致资料分散。
- 每个正式需求使用独立目录 `YYYYMMDD-[需求编号-]需求简称/`。只有需求身份和关联业务仓库均一致时才复用；否则创建新目录，不覆盖其他需求资料。

标准结构如下：

```text
YYYYMMDD-[需求编号-]需求简称/
├── requirements/
│   ├── ORIGINAL_REQUEST.md
│   └── 原始附件或快照...
├── REQUIREMENT_SOURCES.md
├── FEATURE_CONTEXT.md       # 仅在确有压缩价值时创建
├── TECHNICAL_DESIGN.md
├── HANDOFF.md
└── REVIEW_RESULT.md         # 独立 Review 后创建
```

使用插件 `assets/feature-workspace/` 下的模板初始化对应文档。模板规定文档职责，不构成新的需求事实。

## Requirement Evidence

- 用户在 Chat 中给出的原始需求逐字保存到 `requirements/ORIGINAL_REQUEST.md`，不得先总结再落盘。
- 本地附件原样复制到 `requirements/` 并保留文件名；可计算时记录 SHA-256，并核对复制前后哈希。
- 在线资料优先保存允许导出的原始快照；无法可靠导出时，记录 URL、访问时间、版本或更新时间以及未落盘原因，不伪造快照。
- `REQUIREMENT_SOURCES.md` 维护来源索引、版本、位置、状态和冲突说明。资料更新时新增 Evidence 或快照，并将旧来源标记为 `SUPERSEDED`；不得覆盖或静默改写旧证据。
- 用户后续明确确认可以改变当前有效需求，但该确认本身也要作为新的 Requirement Evidence 保存。

Requirement Evidence 和用户最新确认是需求事实源。当前代码决定已实现的技术事实。技术设计、Handoff、Review Finding 和其他派生文档都只能作为线索或当前工作载体。

## 派生文档

### FEATURE_CONTEXT.md

`FEATURE_CONTEXT.md` 只是从 Requirement Evidence 派生的 Working Context，不是独立事实源。仅在多来源需要归并、原始资料较长，或关键业务规则复杂且确有上下文压缩价值时创建。单一、简短、已明确的需求不机械生成。

对会直接改变结果的状态、金额、权限、审批、幂等、兼容、外部接口和关键 AND / OR / NOT 规则，应保留精确语义与 Evidence Pointer。Feature Context 与原始证据冲突时，以原始证据和用户最新确认处理，并更新或标记派生上下文失效。

### TECHNICAL_DESIGN.md

保存当前有效的 Current State、Target State、Gap Analysis、Technical Design、Implementation Plan、Validation Strategy 和 Open Questions。业务规则回指 Requirement Evidence，Current State 回指当前代码。

### HANDOFF.md

作为跨阶段滚动当前态，只保留当前有效的目标、约束、代码基线、技术决策、修改状态、验证结果、风险和下一动作。更新时替换过期状态，不累积工具日志、检索过程、聊天摘要、废弃方案或运行时编排信息。Handoff 不替代 Requirement Evidence、Technical Design 或当前代码。

### REVIEW_RESULT.md

独立保存最新一轮 Review 的基线、Findings、验证证据、剩余风险和 `READY` / `NOT READY`。复审时更新当前结果，并用简短历史记录说明上一轮结论为何失效；不得把 Review 结果混入或改写 Requirement Evidence。

## 阶段边界

- Design：仅在通过工作区创建门禁后创建或识别工作区，保真保存需求，按需生成 Feature Context，写入 Technical Design 并初始化或更新 Handoff。业务仓库、代码、配置和数据库保持只读。
- Implement：仅对已有 Feature 工作区或已通过升级条件的跨服务 Bug 使用；先读 Handoff、Requirement Sources 和当前真实代码，按需回查设计与 Evidence；只修改实现目标所必需的业务文件，并在里程碑和交付前更新外部 Handoff。不得修改原始需求证据。
- Review：独立重建结论，业务仓库、生产代码、配置和数据库保持只读；属于正式 Feature 时将 Review 结果写入外部 `REVIEW_RESULT.md` 并同步 Handoff；普通一次性 Bug Review 不创建工作区。

外部工作区的文档写入不授权修改业务仓库或扩大业务 Scope。任何阶段发现新证据推翻核心前提时，都必须停止沿用受影响方案，回到必要的只读调查并更新当前有效设计；不得只修补局部错误后继续保留失效骨架。

## 读取边界

新阶段先读取 `HANDOFF.md` 与 `REQUIREMENT_SOURCES.md`，再按当前目标读取 Technical Design、Feature Context 或 Evidence Pointer 指向的原始片段。资料已落盘不代表要把完整 Chat、全部附件、全部设计和过程历史同时装入上下文。
