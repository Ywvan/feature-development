---
name: feature-implement
description: 基于外部 Feature 工作区的 Requirement Evidence、当前技术设计、滚动 Handoff 与当前代码完成正式 Feature 实现，也用于处理已验证的 Review Finding。只做完成目标所需的最小必要修改、验证并更新 Handoff。
---

# Feature Implement

## Goal

正确实现需求，而不是机械执行旧方案或清零 Review Finding。

本 Skill 不规定 Codex 的搜索顺序、工具、Task、Context、Session 或 Subagent 策略。当前代码决定技术事实；需求事实仍以用户最新确认和当前有效需求资料为准。

## Feature 工作区

处理正式需求时，先完整读取并遵守 [Feature Workspace Contract](../../references/feature-workspace-contract.md)：

- Feature 工作区根目录必须按 Contract 的运行时解析规则确定；不得在 Skill 中写死 Windows、WSL、Linux 路径、用户名、盘符、home 目录或挂载点。
- 优先识别并复用已有需求目录。若正式需求没有工作区，先补齐原始 Requirement Evidence、`REQUIREMENT_SOURCES.md`、最小 `TECHNICAL_DESIGN.md` 与 `HANDOFF.md`，再编辑业务代码。
- 新阶段先读 `HANDOFF.md` 和 `REQUIREMENT_SOURCES.md`，再按当前目标选择性读取 Technical Design、Feature Context 或 Evidence Pointer 指向的原始片段。
- 原始需求只追加新 Evidence 或更新来源状态，不覆盖旧证据。Feature Context、设计和 Handoff 都不能成为新的需求事实源。
- 在重要里程碑、验证结束和交付前更新外部 `HANDOFF.md`；只保留当前有效状态，不累积工具日志、调试过程或废弃方案。

Feature 文档只写入外部工作区，不写入业务仓库。外部文档写入本身不授权修改业务代码或扩大 Scope；代码修改仍以用户已授权的正式 Feature 范围为限。

## 修改前

开始编辑前完成足够的代码核对：

- 读取当前 Feature 的目标、关键业务规则、Out of Scope 和验收条件；
- 读取已有 Technical Design / Implementation Handoff，把它们作为实现指导而不是事实源；
- 重新阅读待修改区域及必要上下游代码，确认设计中的关键技术假设仍然成立；
- 检查当前工作树和用户已有修改，避免覆盖无关变更；
- 只在需求规则缺失、冲突或会改变实现结果时回查对应原始需求证据，不机械重读全部资料。

不得把未经验证的旧设计、Review Finding 或字段含义直接当成当前代码事实。

## 实现原则

只做完成 Target State 所需的最小必要修改：

- 不扩大需求范围；
- 不修改无关文件；
- 不顺手重构、改名、抽方法、格式化或优化；
- 不修改接口协议、响应结构、数据库结构、枚举定义或既有业务语义，除非需求明确要求；
- 允许读取必要的上下游代码确认影响，但修改范围只扩展到完成当前目标确实必需的位置；
- 优先复用项目已有成熟模式；
- 对状态、金额、权限、审批、幂等、兼容和外部接口契约等关键规则保持精确判断条件；
- 遇到会改变业务行为且无法可靠确认的问题，不猜测实现。

## Review Fix

用户要求处理 Review Finding 时，把 Finding 当作“待复核问题”，不是既成事实：

1. 先用当前代码验证 Finding 是否成立、触发条件是什么。
2. 必要时对照当前需求确认它是否真的违反目标行为。
3. Finding 不成立时不修改，直接给出代码证据和结论。
4. Finding 成立时再定位根因并设计最小修复。
5. 修复后检查相关正常、异常和边界路径，避免局部修复破坏原有行为。

不要因为 Review 已给出风险等级、风险编号或修复建议，就跳过重新验证。

## 验证

验证方式与变更风险匹配，优先选择能直接证明本次修改正确性的方式，例如：

- 静态 diff 与关键调用链检查；
- 针对性单元测试；
- 编译或构建；
- 必要的集成验证；
- 关键正常、异常、边界和兼容场景检查；
- SQL / 数据验证。

不得把未执行、超时、环境阻塞或仅静态检查描述为“验证通过”。

## 完成前检查

确认：

- 实现确实覆盖目标行为；
- 关键业务规则没有被遗漏或改写；
- 修改范围内没有无关改动；
- 接口、数据库和既有业务语义的变化均有需求依据；
- 必要日志、注释和测试与本次风险匹配；
- 仍未确认的问题已明确说明。

## 输出

默认使用简洁、直接的中文。重点说明：

- 修改了哪些文件；
- 核心修改是什么；
- 为什么这样改；
- 执行了哪些验证以及结果；
- 仍有哪些风险或未验证项。

只有本次确实涉及接口、数据库、业务语义或测试缺口时才单独说明，不输出无意义的固定模板项。

避免用 Requirement Authority、Working Context、Coverage、Workstream、Evidence Pointer 等流程术语描述普通代码修改；优先使用具体业务和代码语言。

正式需求交付前，将实际修改、设计偏差、当前代码基线、验证结果、未验证项、阻塞项和下一动作同步到外部 `HANDOFF.md`。不得修改 `requirements/` 中的原始 Evidence。
