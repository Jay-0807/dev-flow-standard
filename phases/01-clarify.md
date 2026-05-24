# Phase 1 — 需求接收 & 澄清反问

## 目标
把 PM 的口语化需求（"我想给后台加个 XX"）转化为结构化的需求 1-pager，确保后续 PRD 不跑偏。

## 输入
- PM 的原始描述（可能是一句话、一段话、几条 bullet）
- （可选）相关参考链接、截图、竞品 demo

## 工作流（5 步）

**Step 1：复述确认**
用 1-2 句结构化方式复述 PM 的需求，让 PM 校对是否准确。
模板：「我理解的是：在 [模块] 中，为 [目标用户] 增加/修改 [具体能力]，使他们能够 [完成什么动作]。对吗？」

**Step 2：调 superpowers:brainstorming 跑发散**
触发关键词："先 brainstorm 一下" 或直接 invoke。让 brainstorming 帮助挖掘：
- 这个需求背后的真实痛点是什么？
- 还有哪些隐藏的相关诉求？
- 反例和边界条件？

**Step 3：调 product-sprint-prioritizer 做初步评估**
用 Agent 工具：`subagent_type: "product-sprint-prioritizer"`。
让它从 PM 视角评估：
- ROI 大致量级
- 是否值得放进当前 sprint
- 与现有 backlog 的关系

**Step 4：用 AskUserQuestion 集中澄清**
基于前 3 步识别出的"信息缺口"，挑 2-4 个最关键的问题，每个问题给 2-4 个选项。常见问题模板：

| 问题类型 | 示例 |
|---|---|
| 目标用户 | 这个功能主要给谁用？(选项：店主 / 客服 / 运营 / 全部) |
| 触发场景 | 用户在什么场景下会用到？(选项：日常运营 / 大促前 / 出问题时) |
| 边界 | 这次只做核心还是含 X？(选项：仅核心 / 含 X / 全做) |
| 优先级 | 排在哪个 sprint？(选项：本 sprint / 下 sprint / 待定) |
| 时间 | 有没有硬 deadline？(选项：本周 / 本月 / 无) |

**Step 5：注入 项目业务哲学（如有）反问**
Read `<project-business-context>.md（universal 版无 项目业务哲学（如有），可由项目自定）`，按其中的"隐形信息检查清单"主动反问 PM：
- 这次需求里有哪些"业务规则是大家心照不宣但没写出来的"？
- 有哪些情况是 AI 不能自动做、必须人工兜底的？
- 数据来源是哪里？置信度如何标记？

## 产出

**文件**：`iteration-vault/01-clarified-requirement.md`

结构：
```markdown
# 需求 1-pager: <feature-name>

## 原始诉求（PM 原话）
<逐字复述>

## 结构化需求
- 模块：
- 目标用户：
- 核心动作：
- 价值假设：

## 澄清后的边界
- 包含：
- 不包含：
- 待定：

## 隐性信息（r4 显性化）
- 隐藏假设：
- 人工保留点：
- 数据来源 & 置信度：

## 初步评估（来自 product-sprint-prioritizer）
- ROI 量级：
- Sprint 建议：

## 待 PRD 阶段进一步细化的点
- ...
```

**对 PM 的摘要**（口头汇报）：
3-5 行讲清楚：澄清后的核心诉求、识别出的最大风险点、下一步要做什么。

## Step 6：决定是否进入 Phase 1.5 用户研究（v2 新增）

完成上面 5 步后，autonomous 判定是否需要进 Phase 1.5：

| 情况 | 决策 |
|---|---|
| PM 显式说"我已经做过用户调研" | 跳过 Phase 1.5，直接进 Phase 2，autonomous-decisions.md 留痕 |
| PM 说不清目标用户具体是谁 | **必须**进 Phase 1.5 |
| PRD 涉及最终用户界面 / 新功能 / 用户体验改造 | **默认**进 Phase 1.5 |
| 内部工具 / 纯后台 / 数据迁移 | 可跳过 Phase 1.5（标"非用户面向"） |

默认决策：**进 Phase 1.5**。PM 没说跳过，就跑。

## 关卡处理
本阶段**不是** ⛳ 关卡，PM 完成 AskUserQuestion 后自动进入 **Phase 1.5（默认）** 或 Phase 2（PM 显式说跳过用户研究时）。
若发现关键信息严重缺失（如目标用户都说不清）→ 暂停并要求 PM 补充。

## 失败回退
- 若 PM 完全说不清需求 → 建议先单独跑一次 `brainstorming`，本 skill 等
- 若 PM 描述的不是"需求"而是"调研需求"或"bug 报告" → 提示切换到 `firefly-deep-research-skill` 或 `systematic-debugging`
- 若 PM 说"我们已经知道用户是谁"但又给不出具体证据 → 默认进 Phase 1.5（PM 强烈反对时跳过并 autonomous-decisions.md 留痕"PM 拒绝用户研究"）
