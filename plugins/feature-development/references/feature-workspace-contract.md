# Feature Workspace Contract

本契约适用于新增 Feature、需求迭代和业务规则变更等正式需求开发。纯诊断、解释问答、不属于既有 Feature 的一次性只读 Review，以及不改变业务行为的机械修改不强制创建工作区。

## 路径与目录

- 逻辑根目录固定为 `~/Documents/Codex/features/`。
- 文件操作前使用当前 WSL 用户的 home 目录解析 `~`；文档、模板和 Skill 中不得写死用户名或任何机器专属的绝对 home 路径。
- 不得改用 `~/.codex/features/`。只有用户明确要求导出或定位 Windows 文件时，才按需转换当前 WSL 绝对路径。
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

- Design：创建或识别工作区，保真保存需求，按需生成 Feature Context，写入 Technical Design 并初始化或更新 Handoff。业务仓库、代码、配置和数据库保持只读。
- Implement：先读 Handoff、Requirement Sources 和当前真实代码，按需回查设计与 Evidence；只修改实现目标所必需的业务文件，并在里程碑和交付前更新外部 Handoff。不得修改原始需求证据。
- Review：独立重建结论，业务仓库、生产代码、配置和数据库保持只读；Review 结果只写入外部 `REVIEW_RESULT.md`，并将最新门禁、阻塞项和下一动作同步到 Handoff。

外部工作区的文档写入不授权修改业务仓库或扩大业务 Scope。任何阶段发现新证据推翻核心前提时，都必须停止沿用受影响方案，回到必要的只读调查并更新当前有效设计；不得只修补局部错误后继续保留失效骨架。

## 读取边界

新阶段先读取 `HANDOFF.md` 与 `REQUIREMENT_SOURCES.md`，再按当前目标读取 Technical Design、Feature Context 或 Evidence Pointer 指向的原始片段。资料已落盘不代表要把完整 Chat、全部附件、全部设计和过程历史同时装入上下文。
