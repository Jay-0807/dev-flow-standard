# 📝 PRD 撰写

> 内部编号：Phase 2  |  ⛳ PM 关卡 1  |  独立命令：/u-prd
> 模式：💬 PM 必参与 + GAN
> v4 含 GAN 钩子（已删除"批数跟进问"，工时估算下放到 Phase 6 自动估算）

## 目标
把 Phase 1 的需求 1-pager 扩展为完整 PRD（产品需求文档），覆盖背景、目标、用户故事、功能清单、验收标准、风险，确保后续架构/开发有据可依。这是**第一个 PM 关卡**（**v4 共 2 个 PM 关卡**：本 phase + Phase 12.5 早晨复盘）。

## GAN 钩子

本 phase 的 Step 2 起草 PRD 时调用 GAN 引擎：
- 任务类型：`prd-writing`
- 输入：01-clarified-requirement.md + 01.5-user-research.md + templates/prd.md
- 输出路径：`iteration-vault/02-prd-gan/`
- 默认 N=5（含早退）
- 失败 fallback：单 gen + Karpathy 自检
- 后续行为：
  - pass → final.md 作为 PRD 提交给 PM 关卡 1
  - needs_improvement → 提交但标记 + PM 在关卡知情
  - degraded → autonomous-decisions.md 留痕 + 继续

## 输入
- `iteration-vault/01-clarified-requirement.md`
- `iteration-vault/01.5-user-research.md`（**🆕 v2 必读** — 3 个用户画像 + 痛点 ranking + 用户心智隐形 5 字段。若 Phase 1.5 被跳过，PRD 第 3、3.5、4、7.4 节会显性标"假设性"待 10.5 验收）
- `<project-business-context>.md（universal 版无 项目业务哲学（如有），可由项目自定）`（r4 四维显性化要求，**v2 含用户心智维度**）
- `<project-positioning>.md`（项目自定，含业务协议约束如有）
- `templates/prd.md`（PRD 模板，**v2 含 3.5 用户旅程强制段 + 7.4 用户心智隐形必填段**）

## 工作流（4 步）

**Step 1：调 /autodev-ideation 做领域思考**
触发：`/autodev-ideation <feature-name>`。
**适配重点**：调用前先把 项目业务上下文（如有）喂给它，让它以"AI 原生组织 OS / 电商场景"为前提思考，而不是泛泛而谈。

输入给 /autodev-ideation 的前置 prompt：
```
背景：本次需求服务于：[项目名称]，目标客户：[业务方]。
约束：技术栈 Next.js + 项目业务协议（如有）；r4 哲学要求隐形信息显性化。
请用上述前提思考本需求的：
- 产品形态可能性（短中长 3 个版本）
- 与竞品（钉钉 / 飞书 / Notion AI）的差异点
- 电商客户最在意的功能切片
```

**Step 2：套用 templates/prd.md 起草（v2 用户视角强化版）**
Read `templates/prd.md`，按模板逐段填充：

1. **背景 & 问题描述**：从 01 文件继承
2. **目标 & 非目标**：明确做什么、明确不做什么
3. **目标用户 & 场景**：**强制引用** 01.5-user-research.md 的 3 个画像（不允许 PM 凭空写"店主"）
3.5. **🆕 用户旅程**（强制段）：复制 01.5 的 mermaid 图（当前 vs 新功能后），列关键改善点
4. **核心用户故事**：每条挂证据来源（频次×强度 / 工单 / 访谈），不允许空挂"As a... I want..."
5. **功能清单**：必做 / 可做 / 不做 三档，**P0 必须挂用户痛点 ranking**
6. **验收标准 (AC)**：每个功能点至少 3 条可测试条件
7. **隐形信息显性化**（r4 **四维**强制段）：
   - 7.1 业务规则（七字段）
   - 7.2 人工保留点
   - 7.3 数据来源 & 置信度
   - 7.4 **🆕 用户心智隐形**（5 字段，来自 01.5）
8. **指标 & 度量**：怎么知道做对了？
9. **风险 & 依赖**：技术/合规/业务三类
10. **里程碑（粗）**：留给 Phase 6 细化

**Step 3：调 product-sprint-prioritizer 优先级复核**
用 Agent 工具调 `product-sprint-prioritizer`，专门校 PRD 的"必做/可做/不做"分档是否合理，并给出本迭代建议范围。

**Step 4：调 docx 双格式输出**
Phase 2 是 PM 关卡，PRD 要给 PM 易读形式：
- 主版本：`iteration-vault/02-PRD.md`（Claude 用）
- 副版本：`iteration-vault/02-PRD.docx`（PM 在 Word 里读、改批注）

## 产出

**主文件**：`iteration-vault/02-PRD.md` + `02-PRD.docx`

**对 PM 的摘要**（必须在调 AskUserQuestion 前给出）：
```
PRD 已完成（路径：iteration-vault/02-PRD.md，Word 版同目录）。
核心要点：
1. 本次将做 [X、Y、Z] 三件事，不做 [A、B]
2. 隐形信息已显性化：[关键人工保留点 1-2 个]
3. 主要风险：[1-2 个待 PM 决策]
4. 预计涉及前端/后端/数据库 [若干] 个模块（影响面在 Phase 3 详查）
```

## ⛳关卡处理（关键）

调用 `AskUserQuestion`，唯一一个问题：
```
"PRD 是否通过，可以进入架构设计阶段？"
选项：
- ✅ 通过 — 我看完了，没问题，进入下一阶段
- 🔄 局部修改 — 大方向 OK，但需要改 [我会写出具体点]
- ❌ 完全重做 — PRD 方向不对，需要重新沟通
```

**PM 选 🔄**：要求 PM 用文字写出具体改动点（功能加减/AC 调整/边界变更），本 skill 重跑 Step 2-3，再次生成 PRD，再次确认。
**PM 选 ❌**：回到 Phase 1 重新澄清。
**PM 选 ✅**：先做批数选择（v4 新增），再进入 Phase 2.5 brainstorm。

## v4 新增：批数选择跟进问（PRD 通过后立即问）

PRD 通过后，skill 先估算总跑时（基于 PRD 大小预判），然后追加一个 AskUserQuestion：

```
question: "夜间模式怎么跑？估算总跑时 {{estimated_hours}}h"
header: "夜间模式批次选择"
options:
  - label: "🌙 单批跑（一晚跑完，默认）"
    description: "适合：连续时段 < 4h，PM 早上一次性复盘"
  - label: "🌙🌙 2 批跑（Batch1 设计 + Batch2 实施发布）"
    description: "推荐：跑时 4-8h；中间可选给 PM 看设计成果"
  - label: "🌙🌙🌙 3 批跑（设计 + 实施 + 审发）"
    description: "适合：大改动 ≥ 8h；最稳"
```

PM 选完后：
- 写 `iteration-vault/batches/batch-plan.md`（用 `templates/batch-plan.md`）
- state.yaml 标 `mode=night-mode, current_batch=1`
- 启动 Phase 2.5 brainstorm（夜间模式开跑）

详见 `night-mode-batching.md`。

## 失败回退
- 若 PRD 写出来 PM 反复修改 3 次以上 → 暂停，建议 PM 与 brainstorming + product-sprint-prioritizer 单独深聊
- 若 PRD 涉及法律/合规风险 → 在 Step 3 之后追加调 `support-legal-compliance-checker` agent

---

## Standalone 模式

可独立触发：`/u-prd "<feature>"`
不依赖前置 vault（如无 iteration-vault/，主线程会自动创建一个简化 vault）。
适合：PoC 期 / 只想出文档不写代码。
输出：`iteration-vault/<date>-<title>/02-PRD.md` + docx 副本。
任何 GAN FAIL → 同时写 DEBUG-TRACE.md。
跳过下游 phase（不自动启动夜间模式），等 PM 后续命令。
