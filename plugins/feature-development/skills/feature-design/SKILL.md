---
name: feature-design
description: 用于已经满足 Feature Context Contract 门禁的正式需求或复杂跨服务变更的调查与技术设计；普通单服务 Bug、一次性诊断和一次性只读 Review 不使用本 Skill。
---

# Feature Design

只负责正式 Feature 的调查与技术设计，不定义通用工程或 Agent 执行规则。先读取 [Feature Context Contract](../../references/feature-context-contract.md) 第 1 节门禁；未通过时不执行阶段流程，通过后读取事实源、交接内容和本阶段边界。

## 输入与只读边界

使用有效 `FEATURE_CONTEXT`、原始 Requirement Evidence、用户最新确认、当前代码/配置/数据链路和 Repository Evidence；继续同一 Feature 时读取已有 `DESIGN_HANDOFF`。需求资料确定目标，代码确定 Current State，Handoff 只承载待复核的技术决策，不成为需求事实。

Design 阶段业务仓库、业务代码、配置和数据库保持只读。

可扩大只读调查确认事实，不因此扩大修改范围。

## 设计交付

1. **Current State**：现有实现、真实调用链、结果行为与约束，附代码位置。
2. **Target State**：目标行为，状态、金额、权限、接口及兼容规则保留精确语义，并遵守 Contract 的硬事实要求。
3. **Gap Analysis**：逐项差异，每项对应实现或验证动作；未确认项进入 Open Questions。
4. **Technical Design**：关闭 Gap 的方案，含分支、数据一致性、失败处理、兼容及验证设计。
5. **Task Graph**：按实际依赖划分，不强制拆分；每个 Task 列明目标、边界、依赖和验收条件，依赖满足后可交付与验收。允许前后依赖，不要求每个 Task 无依赖或单独完成整个 Feature。
6. **Risks / Open Questions**：影响实现、发布或验收的风险与未知项。
7. **Validation Strategy**：将本次行为变化和已识别风险对应到验证目标、方式及预期结果；标明证据缺口，遵守现有测试和操作授权。

当 Current State 中的金额、状态、权限、接口字段或契约结论来自其他仓库的生产结果，且当前已读取代码中没有对应生产逻辑或权威定义时，读取产生该结果的 producer 或 contract owner。确认该结论对应的生产逻辑或权威定义后，不因同一结论继续读取其他仓库；跨仓只读调查不扩大业务代码修改范围。

以上输出组成 Contract 中的 `DESIGN_HANDOFF`。按 Contract 第 4 节保存原仓库外目录的设计和交接文档；Handoff 保留决策原因、兼容约束、依赖及未解决问题，其余引用接收方可读取的设计章节，不复制整份报告或调查历史。不自动进入实现阶段。
