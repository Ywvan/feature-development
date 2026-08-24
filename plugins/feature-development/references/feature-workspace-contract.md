# Feature Workspace Contract

本契约只适用于新增 Feature、需求迭代或业务规则变更等正式需求开发。纯诊断、一次性问答、不属于既有 Feature 的只读 Review，以及不改变业务行为的机械修改不创建工作区。

## 根目录与命名

- Feature 工作区逻辑根目录：`~/Documents/Codex/features/`。
- 实际文件操作前，按当前 WSL 用户将 `~` 解析为绝对路径。不得写死用户名、`/home/<user>` 或 `/mnt/c/Users/<user>` 等单机路径。
- Windows 路径不是固定契约；仅在用户明确要求导出或定位 Windows 文件时，由当前 WSL 绝对路径按需转换。
- 需求目录：`YYYYMMDD-[需求编号-]需求简称`
- 需求简称应直接表达业务能力，移除 Windows 不允许的路径字符，避免 `new-chat`、`需求开发`、`修改代码` 等无法辨识的名称。
- 同名目录仅在需求编号、需求简称和关联仓库一致时复用；否则依次追加 `-02`、`-03`。不得覆盖其他需求目录。

每个需求目录最多保留一层 `requirements/` 子目录：

```text
YYYYMMDD-[需求编号-]需求简称/
├── requirements/
│   ├── ORIGINAL_REQUEST.md
│   └── 原始附件...
├── REQUIREMENT_SOURCES.md
├── TECHNICAL_DESIGN.md
├── HANDOFF.md
├── FEATURE_CONTEXT.md        # 按条件生成
└── REVIEW_RESULT.md          # 独立 Review 后生成
```

使用插件 `assets/feature-workspace/` 下的同名模板初始化 Markdown 文档。模板是结构约束，不是新的 Requirement Evidence。

## 原始需求保真

1. 用户直接在 Chat 中给出的需求文字逐字保存到 `requirements/ORIGINAL_REQUEST.md`，记录获取时间和可定位的 Task / Thread 标识；不得先总结再保存。
2. 用户提供的本地附件原样复制到 `requirements/`，保留原文件名；可计算时在 `REQUIREMENT_SOURCES.md` 记录 SHA-256，并核对复制前后哈希。
3. 对链接、在线文档或外部系统记录，优先保存允许导出的原始快照；不能可靠导出时只记录链接、访问时间、版本或更新时间和未落盘原因，不伪造快照。
4. 外部原始资料发生更新时新增快照或来源记录，将旧记录标记 `SUPERSEDED`，不得覆盖或改写旧证据。
5. `REQUIREMENT_SOURCES.md` 至少记录 Source ID、类型、原始名称或链接、获取时间、版本信息、任务标识、本地路径、SHA-256（可获得时）和状态。

原始资料是 Requirement Evidence。用户后续明确确认可以改变当前有效需求，但必须作为新的 Evidence 记录；不得静默编辑旧证据来制造一致性。

## FEATURE_CONTEXT 生成条件

仅在以下任一情况确实存在时创建 `FEATURE_CONTEXT.md`：

- Requirement Evidence 来自多个来源，需要统一 Scope 和优先级；
- 原始资料篇幅较长，后续阶段直接反复读取会产生明显上下文成本；
- 状态、金额、权限、审批、兼容、并发、接口契约或关键 AND / OR / NOT 等规则复杂，需要高密度索引。

单一、简短且已经明确 Goal、Scope、业务规则和验收条件的需求不生成 Feature Context。Feature Context 中每个 Hard Fact 必须包含 Evidence Pointer 和 Verification Status，不得形成只有 Feature Context 自己能够证明的事实。

关键词 search / find 只能作为 Evidence 定位入口，不能作为需求完整性结论；“没有命中关键词”不能证明某项业务规则不存在。对会直接改变 Feature 结果的 Hard Fact，仍需结合 Scope、Acceptance Criteria、状态矩阵、规则表或对应需求章节做结构性核对。

## Evidence Closure 与 Rejected Hypothesis 边界

Design、Implement 和 Review 对关键 Current State、根因、高风险 Finding 或 Final Gate 形成确定结论前，不能把“存在一条支持证据”等同于“证据已经闭环”。至少需要确认实际入口/触发路径、决定结果的关键分支、关键数据或状态的结果路径，并检查最可能推翻当前解释的主要替代假设；不存在合理替代假设时应明确原因。

单个 grep 命中、单一调用点、单条日志、单个 SQL 片段或一条能够解释现象的路径，均不能单独作为上述关键结论的闭环证据。未验证且可能改变结论的事项必须保留为 Open Question / NOT VERIFIED。

Rejected Hypothesis 只保存高价值反证：该假设曾是主要候选、已有明确排除证据，且后续重新采用会显著改变设计、实现或 Review 结论。每项至少保存 Hypothesis、Rejected By、Evidence 和 Reopen If。普通搜索失败、试错、低价值猜想和完整推理历史不保存。

## 文档职责

### TECHNICAL_DESIGN.md

保存当前有效的 Requirement Verification、Current State、Target State、Gap Analysis、Technical Design、Implementation Plan、Validation Strategy、Risks / Open Questions、少量高价值 Rejected Hypotheses 和失效前提。业务规则必须回指 Requirement Evidence，Current State 必须回指当前代码。

核心前提失效时，将依赖该前提的设计状态改为 `INVALIDATED`，重新调查并更新当前有效设计；在 `Invalidated Premises` 中只保留失效前提、影响范围、替代结论和证据，不保留冗长讨论历史。

### HANDOFF.md

Handoff 是跨阶段唯一的滚动当前态入口。它只保留当前阶段、状态、Feature Goal、Verified Hard Facts、Evidence Pointers、有效技术决策、少量高价值 Rejected Hypotheses、代码基线、修改状态、验证结果、阻塞项、下一动作及精简失效记录。

更新时替换已经过期的当前态，不追加工具日志、原始搜索、调试过程、大段代码、聊天摘要、低价值已推翻猜想或废弃方案。高价值 Rejected Hypothesis 必须保留排除证据和 Reopen If；Reopen If 条件成立时，不得继续把该假设视为已排除。后续阶段先读 Handoff，再按 Evidence Pointer、风险或 Reopen If 条件选择性读取其他资料；Handoff 不替代原始需求和当前代码。

### REVIEW_RESULT.md

保存最新一轮独立 Review 的基线、Requirement Gate、Findings、验证证据和 `READY` / `NOT READY`。复审时更新当前结果，并在 `Review History` 中保留轮次、时间、旧结论及失效原因的短记录。Handoff 只同步最新门禁、阻塞项和下一动作。

## 阶段写入边界

- Design：创建或复用需求目录，保真保存需求，按条件生成 Feature Context，写入 Technical Design 并初始化 Handoff。
- Implement：先读 Handoff 和当前代码；按需读取 Technical Design 与 Evidence；在重要里程碑和结束时更新 Handoff。不得修改原始需求。
- Review：业务仓库保持只读；写入外部 `REVIEW_RESULT.md`，并只同步 Handoff 的 Review 当前态。
- 核心前提被推翻时，任何阶段都必须先记录 `INVALIDATED` 并回到必要调查，不能在旧方案骨架上继续修补。

外部 Feature 工作区的文档写入不授权修改业务仓库、接口、数据库、配置或扩大业务 Scope。

## 读取与 Token 边界

- 新阶段先读取 `HANDOFF.md` 与 `REQUIREMENT_SOURCES.md`。
- 仅按当前目标读取 `TECHNICAL_DESIGN.md` 的相关章节、Feature Context 或 Evidence Pointer 指向的原始片段。
- 不因资料已落盘就同时灌入完整 Chat、全部原始文档、全部设计和工具历史。
- Rejected Hypotheses 只保留高价值反证，不得演变成完整调查历史。
- Token 是否降低必须用同类任务的实际上下文与正确性验证衡量，不承诺固定比例。
