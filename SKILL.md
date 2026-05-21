---
name: universal-coding-project-development-skill
description: 通用代码项目开发工作流编排器，**业务无关、技术栈无关**。PM 或开发者一句话需求（哪怕只是"加个 X 功能"），自动走完 13 阶段完整开发流水线（v2 含用户视角强化）：需求澄清 → 🆕用户研究 → PRD → ADR → 🆕UX 设计 → UI 实现 → schema/API → 实施 → 测试/代码债/安全 → 性能/可观测 → 代码审查 → 🆕真实用户验收 → 发布。中间只在 2 个关卡（PRD / Release PR）需要 PM 决策，其余全自治。支持 3 种项目类型分支（Web 全栈 / B 端 SaaS / AI 原生应用）自动路由。本 skill 编排（不替代）本机已装的 superpowers / adr-skill / Figma MCP / Playwright MCP / Sentry MCP 等。⚠️ 与 firefly-web-os-orchestrator-skill 严格区分：本 skill 不注入任何业务上下文；firefly orchestrator 是带 firefly r4 哲学 / 电商客户 / Next.js+A2A 栈的业务特化版，仅在 firefly 项目目录下使用。设计依据：D:\cursor_project\research-vault\universal-dev-workflow\report.md 完整调研（2026-05-18）。触发关键词："新需求"、"加个功能"、"想做个 X"、"需求来了"、"改一下 X"、"迭代 Y"、"下次版本"、"新一轮开发"、"跑完整代码开发流程"、"完整跑一遍"、"PRD please"、"new feature request"、"new feature please"、"iteration"、"feature please"、"implement X"、"ship X"、"build me a"、"please add a feature"。
---

# Universal Coding Project Development Skill — 通用代码项目开发工作流

## 这个 skill 解决什么问题

PM 或开发者提出一个代码需求时，需要走完"想法 → PRD → 架构 → 实施 → 测试 → 上线 → 监控"完整闭环。手动走每个阶段：
- **慢** — 每个阶段都要找对应工具 / skill
- **易漏** — 容易跳过代码债、安全审查、文档更新
- **不一致** — 每次都重新决定流程，团队成员各自不同

本 skill 把 2026-05-18 调研出来的"业界最佳 24 个模块 → 10 阶段闭环"固化成自动编排，从单一入口跑通整个代码项目。

## 与 firefly-web-os-orchestrator-skill 的关系

| 维度 | 本 skill（universal） | firefly orchestrator |
|---|---|---|
| 业务上下文注入 | ❌ 无（通用） | ✅ firefly r4 哲学 + 电商客户 + Next.js+A2A 栈 |
| 适用项目 | 任何代码项目 | 仅 firefly 项目 |
| 选择规则 | 工作目录 ≠ firefly | 工作目录 ∈ `D:\cursor_project\projects\e-commerce_multihuman_multiagent_os\firefly\` |

**两个 skill 不互相调用**，是兄弟关系不是父子。

## 何时触发

✅ **应该触发**：
- "新需求" / "加个功能" / "想做个 X"
- "需求来了" / "下次版本" / "新一轮开发"
- "改一下 X" / "迭代 Y"
- "实现一下 X" / "ship 一个 X"
- PM 或开发者用一句话描述了**代码层面**要做的事

❌ **不应该触发**：
- 用户在问问题（"这个怎么用"、"X 是什么意思"）→ 直接回答
- 用户在做调研 → firefly-deep-research-skill
- 用户在写文档 / 写邮件 / 做 PPT → 对应文档 skill
- 用户在 firefly 项目目录里提需求 → firefly-web-os-orchestrator-skill
- 用户只想跑一行 PoC 验证 → 直接动手，不要走 10 阶段

## 工作流总览（v2 含用户视角）

```
PM 一句话 → ① 澄清 → 🆕 ①.5 用户研究 → ② PRD ⛳关卡1 → [自治模式启动]
  → ③ ADR
  → 🆕 ④a UX 设计（用户旅程+IA+低保真线框，若适用）
  → ④b UI 实现（高保真技术 spec）
  → ⑤ schema+API → ⑥ 实施(前后端并行)
  → ⑦ 测试+代码债+安全 → ⑧ 性能+可观测 → ⑨ 代码审查
  → 🆕 ⑨.5 真实用户验收（3-5 真人 + Playwright）
  → ⑩ 发布 ⛳关卡2 → merge 上线 → 反馈循环
```

**PM 总接触点 = 2 次（PRD 5-15 min + Release PR 5-10 min）**

**v2 核心变化**：在阶段 ①、④、⑨ 后各加一个 "用户视角" 子阶段，确保**从用户出发、技术验收 ≠ 用户验收**。

详细阶段定义见 `reference/workflow-loop.md`，按需 Read。

## 启动时主动反问（Step 0）

收到用户需求后，在进入阶段 1 之前，用 **AskUserQuestion** 一次性问清两件事（详见 `reference/project-type-router.md`）：

1. **项目类型**：Web 全栈（默认）/ B 端 SaaS / AI 原生应用 / 别的（让用户描述）
2. **本次目标粒度**：完整 10 阶段（默认）/ 仅前 5 阶段（PRD-实施前的"做规划"）/ 仅后 5 阶段（已有代码、跑测试到上线）

启动时也**主动检查工作目录**（Read package.json / pyproject.toml / Cargo.toml 等），如果发现是 firefly 项目，**主动提示用户**："这看起来是 firefly 项目，要不要改用 firefly-web-os-orchestrator-skill（业务特化版）？"

## 13 阶段流水线（v2 含用户视角，精简版）

详细每阶段调用见 `reference/workflow-loop.md`。下面是 Claude 走的顺序：

| # | 阶段 | 主调 skill/MCP | 输出 | ⛳关卡 |
|---|---|---|---|---|
| 1 | 需求澄清 | obra/superpowers `/brainstorm` | `01-canonical-query.md` | - |
| **1.5** | **🆕 用户研究** | obra `/brainstorm` 扮真实用户 + 通用 user-interview prompt（前端项目强烈推荐） | `01.5-user-research.md` | - |
| 2 | PRD 起草 | 直接写（参 `templates/prd.md`，**必引用 01.5 用户画像**） | `02-prd.md` | **⛳ PM 关卡 1** |
| 3 | 架构 + ADR | **adr-skill** | `03-adr.md` | - |
| **4a** | **🆕 UX 设计** (前端项目) | **Figma MCP**（低保真 wireframe）+ Nielsen 10 可用性自检 | `04a-ux-design.md` + `04a-wireframes/` | - |
| **4b** | UI 实现 spec (前端项目) | **Figma MCP**（高保真）+ 技术 spec | `04b-ui-spec.md` | - |
| 5 | DB schema + API 契约 | Prisma / openapi-generator | `05-schema-and-api.md` | - |
| 6 | 实施 (前后端并行) | subagent-driven-development + 项目类型对应 subagent | feature 分支 | - |
| 7 | 测试 + 代码债 + 安全 | **Playwright MCP** + anthropic /security-review + 9 维代码债内化方法 | `07-test.md` + `07-debt.md` + `07-security.md` | - |
| 8 | 性能 + 可观测接入 | **Sentry MCP** SDK 接入 + (AI 项目加 Langfuse Web UI) | `08-perf-obs.md` | - |
| 9 | 代码审查 | anthropic-skills:review | `09-review.md` | - |
| **9.5** | **🆕 真实用户验收** | **Playwright MCP** 录基线 + 3-5 个真实用户跑 happy path | `09.5-user-acceptance.md` + `tests/user-acceptance/` | - |
| 10 | 发布 | release-please / semantic-release | Release PR | **⛳ PM 关卡 2** |

## ⛳ PM 关卡机制

**关卡 1（阶段 2 后）**：PRD 写完，用 AskUserQuestion 三选项：
- ✅ 通过 → 进入自治，PM 可"去睡觉"
- 🔄 修改 → PM 给反馈，本 skill 重写 PRD
- ❌ 重做 → 回阶段 1 重新澄清

**关卡 2（阶段 10）**：Release PR 创建，PM 看：
- diff 是否符合 PRD
- 自动 changelog
- autonomous-decisions.md（自治期间的决策日志）
- 决定 merge 或 request changes

## 4 红线 escalation（自治模式下任一触发即暂停）

详见 `reference/escalation-redlines.md`，简版：

- **R1 架构冲突**：阶段 3 ADR 与现有代码不可调和
- **R2 安全 must-fix > 3 条**：阶段 7 高危发现 > 3
- **R3 验收 3 次重试失败**：阶段 7/9 任一 3 次重试仍不过
- **R4 删除既有功能**：变更涉及破坏性删除

任一红线触发 → AskUserQuestion 升级到 PM → PM 决策后再续。

## 工作目录约定

启动后，在**用户当前工作目录**（非 ~/.claude/）创建 vault：

```
<当前工作目录>/iteration-vault/<YYYY-MM-DD>-<short-title>/
├── 01-canonical-query.md
├── 01.5-user-research.md       # 🆕 v2：用户画像 + 旅程 + 痛点 ranking
├── 02-prd.md                   # 引用 01.5
├── 03-adr.md
├── 04a-ux-design.md            # 🆕 v2：UX 设计（用户旅程标注 + 低保真 wireframe）
├── 04a-wireframes/             # 🆕 v2：低保真 PNG 序列
├── 04b-ui-spec.md              # UI 技术 spec（4a 下游，仅前端项目）
├── 05-schema-and-api.md
├── 06-implement-log.md
├── 07-test.md / 07-debt.md / 07-security.md
├── 08-perf-obs.md
├── 09-review.md
├── 09.5-user-acceptance.md     # 🆕 v2：3-5 真人 + Playwright 基线
├── 10-release.md
└── autonomous-decisions.md     # 自治模式决策日志
```

**short-title** 从用户原 prompt 提取（如 "加个 AI 选品助手" → `ai-product-picker`）。

## 与本机其他资源的协作

**调度的 skill**（不替代）：
- obra/superpowers — `/brainstorm`、`verification-before-completion`、`requesting-code-review`
- **adr-skill** — phase 3 自动调用
- subagent-driven-development / dispatching-parallel-agents — phase 6 并行执行
- anthropic-skills:review / security-review — phase 7/9
- release-please / semantic-release — phase 10

**调度的 MCP**（详见 `reference/tool-inventory.md`）：
- Figma MCP（17 工具）— phase 4
- Playwright MCP（23 工具）— phase 7
- Sentry MCP（22 工具）— phase 8

**项目类型特殊路由**（详见 `reference/project-type-router.md`）：
- AI 项目：phase 7 加 Promptfoo prompt 测评 + Garak/PyRIT 红队；phase 8 加 Langfuse Web UI 监控

## 反方观点与限制

- **不是银弹**：10 阶段适合"正经迭代"，不适合 1 行 PoC（PoC 直接动手就行）
- **依赖本机已装资源**：若 superpowers / adr-skill / Sentry MCP / Playwright MCP 缺失，对应阶段会**降级而非失败**（Claude 会主动提示用户）
- **AI 项目分支不完整**：当前 Langfuse MCP 仅 Prompt Management，trace/cost 仍要 Web UI（这是 2026-05 现状，未来会变）
- **自治模式风险**：4 红线 escalation 是兜底，PRD 表达模糊时仍可能偏离意图 → PM 应在 phase 1/2 充分澄清
- **不绑定具体 Git workflow**：阶段 10 默认用 release-please，但 PM 团队若用别的（GitFlow / trunk-based），可在启动时声明

## 进一步阅读（按需 Read）

- `reference/workflow-loop.md` — 10 阶段每阶段详细指令
- `reference/project-type-router.md` — 3 种项目类型的阶段差异
- `reference/escalation-redlines.md` — 4 红线判定 + 处理
- `reference/tool-inventory.md` — 本 skill 用到的所有 skill / MCP 清单
- 完整调研依据：`D:\cursor_project\research-vault\universal-dev-workflow\report.md`
