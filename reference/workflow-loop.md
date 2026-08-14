# Workflow Loop — 13 阶段详细指令（v2 含用户视角）

按顺序执行。每阶段完成 → 写 `iteration-vault/<NN>-<output>.md` → 进入下一阶段（关卡阶段除外）。

**v2 新增 3 个用户视角阶段**：1.5 用户研究、4a UX 设计（拆分原 4 UI）、9.5 真实用户验收。
**核心原则**：技术验收（phase 9）不等于用户验收（phase 9.5）。从用户出发，不从功能出发。

---

## Phase 1：需求澄清

**目标**：把 PM 的一句话需求扩展为 canonical query（消除歧义、补全缺失上下文）。

**调用**：obra/superpowers `/brainstorm`

**输入**：用户原话 prompt

**关键问题**（Claude 主动反问 PM）：
- 这个功能解决谁的什么痛点？
- 不做会怎样？做了能衡量的好处是什么？
- 范围里包括什么、不包括什么？
- 有没有时间窗口约束？
- 影响哪些既有功能 / 模块？

**输出**：`01-canonical-query.md`（300-800 字，包含问题陈述 + 用户 + 价值 + 范围 + 排除 + 验收）

**完成判定**：PM 看了输出说"对了"或者"差不多"。

---

## Phase 1.5：🆕 用户研究（v2 新增）

**目标**：在 PRD 之前**采集用户证据**，让 PRD 基于真实用户而不是 PM 拍脑袋。

**调用**：
- obra/superpowers `/brainstorm` 中扮演 3 个不同画像的真实用户做需求反观
- 若需求面向**最终用户**且本机装了垂直领域 agent（如 marketing-* 系列）→ 调用对应 agent 扮演真实用户给反馈

**Claude 动作**：
1. Read 01-canonical-query.md
2. 主动追问 PM：
   - 这个功能的目标用户具体是谁？（按角色/规模/场景细分到 ≥ 3 个画像）
   - 用户现在用什么凑合解决这个问题？（替代方案 + 痛点）
   - 用户上次因这事不爽的具体场景是什么？
   - 用户对 AI 的真实期待是"全自动"还是"我说了算"还是"看情况"？
3. 输出 3 个真实用户画像 + 当前替代方案 + 痛点 ranking（频次×强度）+ 用户旅程（mermaid 图）

**输出**：`01.5-user-research.md`，含：
- Section 1：3 个真实用户画像（典型 / 进阶 / 边缘）
- Section 2：当前替代方案 & 痛点 ranking 表
- Section 3：类似产品参考（业界谁怎么做、用户为何不全转过去）
- Section 4：用户旅程对比（当前 vs 新功能后 mermaid）
- Section 5：用户心智隐形信息（用户没说但在意的 / 上次不爽的场景 / 对 AI 期待 / 决策暗信号）
- Section 6：给 PRD 阶段的必做功能 ranking（基于痛点强度排序）

**跳过条件**：
- PM 显式说"已做过用户调研，直接进 PRD"
- 内部工具 / 纯后台数据迁移 / 不涉及最终用户的需求

**完成判定**：PRD（Phase 2）能直接引用 Section 1 的 3 个画像 + Section 5 的用户心智隐形 5 字段。

---

## Phase 2：PRD 起草 ⛳ PM 关卡 1

**目标**：把 canonical query 变成可执行的 PRD（开发能照着做）。

**调用**：本 skill 内置 prompt（无外部 skill）

**输出**：`02-prd.md`，包含：
- 标题 + 一句话价值陈述
- 用户故事（user stories，至少 1 个 happy path + 1 个 edge case）
- 验收标准（acceptance criteria，binary checklist）
- 非功能性要求（性能 / 安全 / 可访问性）
- 依赖项（DB schema 变化、第三方 API、特性 flag）
- 测量指标（怎么知道上线后效果好坏）
- Out of scope（明确不做什么）

**⛳ 关卡 1**：用 `AskUserQuestion` 给 PM 三选项：

```
Q: PRD 写好了，你看一眼？
A: 
- ✅ 通过 - 我去睡觉，你跑剩下阶段
- 🔄 修改 - 我会描述具体哪里改
- ❌ 重做 - PRD 没抓住核心，重新澄清
```

PM 选 ✅ → 进入 Phase 3（**自治模式启动**）
PM 选 🔄 → 等 PM 反馈，改后回到关卡
PM 选 ❌ → 回 Phase 1

---

## Phase 3：架构 + ADR

**目标**：把 PRD 翻译成架构层面的"我们要怎么做"，记录为 ADR。

**调用**：**adr-skill**（本机已装）

**输入**：`02-prd.md`

**Claude 动作**：
1. Read 02-prd.md
2. 主动识别"需要决策"的点（DB 选型、新加依赖、模块拆分、API 设计模式等）
3. 对每个决策点，触发 adr-skill 生成 MADR Full 格式的 ADR
4. 合并到 `03-adr.md`

**输出**：`03-adr.md`，含 1 个或多个 ADR。

**完成判定**：所有"非显然"的决策都有 ADR。

---

## Phase 4a：🆕 UX 设计（v2 拆分，用户视角先于工程视角）

**目标**：从**用户视角**设计 — 用户旅程 → 信息架构 → 低保真线框。**先想清用户在每一屏要做什么，再决定要画什么**。

**调用**：**Figma MCP**（低保真 wireframe）+ 内置 Nielsen 10 可用性自检

**前置条件**：项目类型是 Web 全栈 / B 端 SaaS（AI 原生纯后端跳过）+ Phase 1.5 已产出 user-research.md

**Claude 动作**：
1. Read 01.5-user-research.md，提取 3 个用户画像 + 用户旅程图
2. 标注用户旅程上的 UI 节点：每个用户动作发生在哪一屏、看到什么、能点什么
3. 信息架构（IA）梳理：入口/主要分类/状态切换/错误空载加载态
4. 用 Figma MCP 出灰度低保真 wireframe（**避免颜色和视觉细节让 PM 纠结**）
5. Nielsen 10 可用性原则自检（每条 ✅ / 🟡 / ❌）
6. 用户心智检查：用户没说但在意的点是否在 wireframe 中体现？

**输出**：
- `04a-ux-design.md`（总文档：含 wireframe 引用 + 可用性自检 + 用户心智检查）
- `04a-wireframes/`（PNG 序列）
- `04a-user-journey-annotated.md`（标注 UI 节点的用户旅程）

**完成判定**：可用性自检 ≥ 7/10 通过 + PM 在 wireframe 上没有"这是干嘛"的疑问。

---

## Phase 4b：UI 实现 spec（v2 紧跟 4a，工程视角）

**目标**：基于 4a 的 UX 设计，出**前端工程师可直接消费的技术 UI spec**。

**调用**：**Figma MCP**（高保真）+ /autodev-ui 或内置 prompt

**前置条件**：Phase 4a 已完成

**Claude 动作**：
1. Read 04a-ux-design.md + 04a-wireframes/
2. 出技术 spec：路由 / 状态机 / 组件清单（新增/复用）/ 设计 tokens / 交互细节 / 无障碍 / 响应式
3. **4a → 4b 可追溯性校验**：4a 的每个用户视角决定是否在 4b 里落地？任何 4a 决定丢失必须有理由
4. 若 PM 视觉敏感型，用 Figma MCP 出高保真稿（参考 4a wireframe 结构）

**输出**：`04b-ui-spec.md`（技术 spec）+ 可选高保真 Figma URL

**完成判定**：4a → 4b 可追溯性 ≥ 90% + WCAG 2.1 AA 自检通过。

---

## Phase 5：DB schema + API 契约

---

## Phase 5：DB schema + API 契约

**目标**：把数据模型和接口契约定义清楚，**先于代码生成**（契约优先）。

**调用**：
- DB schema：Prisma（若 Next.js/TS 项目）/ SQLAlchemy（若 Python）/ ActiveRecord（若 Rails）
- API 契约：openapi-generator + Spectral lint

**Claude 动作**：
1. 从 PRD + ADR 提取数据实体（entities）
2. 写 schema 文件（schema.prisma / models.py）
3. 写 OpenAPI spec（openapi.yaml）
4. 跑 Spectral lint 验证 OpenAPI 合规
5. 跑 openapi-generator 生成客户端 stub（可选）

**输出**：`05-schema-and-api.md`（决策摘要）+ 代码层实际文件（schema / openapi.yaml）

**完成判定**：schema lint 通过、openapi.yaml lint 通过。

---

## Phase 6：实施（前后端并行）

**目标**：基于 schema + UI context 写代码。

**调用**：
- 主：subagent-driven-development + dispatching-parallel-agents
- 前端 subagent：vercel-labs/agent-skills（如装）/ frontend-specialist 角色
- 后端 subagent：wshobson/agents（已装）/ VoltAgent subagents（如装）
- AI 项目分支：+ LangGraph 编排（若涉及多 agent）

**Claude 动作**：
1. 把任务拆分成前端 / 后端两个并行 agent 任务
2. 用 dispatching-parallel-agents 同时启动
3. 等待两边完成
4. 主线程合并 + 解决冲突

**输出**：feature 分支代码 + `06-implement-log.md`（实施记录）

**完成判定**：两边代码能编译 / 启动，不要求功能验证（那是 phase 7）。

---

## Phase 7：测试 + 代码债 + 安全（**3 个并行子阶段**）

**目标**：质量门禁。

### 7a. 测试
**调用**：**Playwright MCP**（mcp__playwright__*）— 23 个工具
- planner 自动产 markdown 测试计划
- generator 写测试用例 + 跑
- healer 修脆弱选择器
- AI 项目加：Promptfoo prompt 测评（CLI）

**输出**：`07-test.md` + 实际测试代码 + 通过率报告

### 7b. 代码债
**调用**：内化 9 维度方法（依赖 / 性能 / 错误处理 / 安全 / 文档漂移 / 测试覆盖 / 复杂度 / API 一致性 / 命名）

**Claude 动作**：对 phase 6 新代码逐维度审查

**输出**：`07-debt.md`，含发现的债 + 立刻修 vs 列入待办

### 7c. 安全
**调用**：anthropic-skills:security-review + Semgrep CLI（若装） + OWASP Top 10 自查
- AI 项目加：Garak + PyRIT 红队（CLI）

**输出**：`07-security.md`，分 must-fix / should-fix / nice-to-fix 三级

**完成判定**：
- 测试 ≥ 80% 通过
- 代码债无新增 high severity
- 安全无 must-fix（或全部修完）

---

## Phase 8：性能 + 可观测接入

**目标**：让生产环境**能看见**这个功能（错误、性能、用量）。

**调用**：
- 性能：k6（若装）做负载测试，或 `mcp__playwright__browser_evaluate` 抽样测响应时间
- 可观测：**Sentry MCP**（mcp__sentry__*）
  - `find_projects` 看 org 下有什么 project
  - 若无对应 project，`create_project` 建一个
  - `create_dsn` 生成 DSN 写入项目 .env
  - 在代码里加 SDK init（按技术栈）
- AI 项目加：LiteLLM gateway + Langfuse SDK 接入（trace/cost 上 Web UI 看）

**输出**：`08-perf-obs.md`（接入证明 + 第一个 trace 截图）+ 代码层 SDK 配置

**完成判定**：本地跑一次故意触发的 error，能在 Sentry dashboard 看到（端到端验证）。

---

## Phase 9：代码审查

**目标**：上线前最后质量关。

**调用**：anthropic-skills:review

**Claude 动作**：
1. 跑一次完整 PR diff review
2. 关注：阶段 7 漏审的、命名一致性、commit message 规范、文档同步
3. 高风险变更（DB migration / 鉴权 / 支付）→ 提示 PM 考虑加 Greptile（按需）

**输出**：`09-review.md`，含 reviewer comments + 自动修复的 commits

**完成判定**：无 blocking issue。

---

## Phase 9.5：🆕 真实用户验收（v2 新增，技术 OK ≠ 用户 OK）

**目标**：在技术验收（Phase 9 代码审查）通过之后、正式发布（Phase 10）之前，**让真实用户实际跑一次**。

**核心原则**：技术验收回答"代码对不对"，用户验收回答"用户能不能用得舒服"。两者缺一不可。

**调用**：**Playwright MCP**（录基线 happy path）+ 真实用户测试（3-5 个真人）

**前置条件**：staging 环境可用 + Phase 1.5 user-research.md 中至少 1 个画像有匹配真人

**Claude 动作**：
1. 用 Playwright MCP 录用户路径基线脚本（按 Phase 4a 用户旅程图）
2. 生成邀请测试模板（自动用 Notion MCP / Gmail MCP 出草稿，PM 看过再发）
3. 邀请 3-5 个真实用户跑 happy path（推荐：内部成员 + 已签约客户 + 朋友圈）
4. 收集反馈（每个用户产出 mini-report：完成场景 / 卡点 / 评分 / 1 句话最爱 + 1 句话最痛）
5. 按以下决策树综合判定：
   - 全部完成 + 平均评分 ≥ 4 → ✅ 进 Phase 10 发布
   - 1 人卡死 + 其他 OK + ≥ 3.5 → 🟡 进 Phase 10 + 卡点入 backlog
   - ≥ 2 人卡死 / 评分 < 3 → ❌ 回 Phase 6 重做
   - ≥ 2 人 "看不懂这是干啥" → 🚨 产品方向 escalation（不是技术问题）

**输出**：
- `09.5-user-acceptance.md`（验收报告 + 综合判定）
- `tests/user-acceptance/<feature>.spec.ts`（Playwright 脚本，回归测试库复用）
- `09.5-baseline-screenshots/`（关键节点截图序列）

**跳过条件**：PRD 明确写"内部工具 / 纯后台 / 不涉及最终用户"

**完成判定**：决策为 ✅ 或 🟡（最多 2 轮重试用户验收）。

---

## Phase 10：发布 ⛳ PM 关卡 2

**目标**：合并到主干 + 发版。

**调用**：release-please / semantic-release / 项目自定义 release 流程

**Claude 动作**：
1. 跑 release-please → 生成 Release PR（含自动 changelog + 版本 bump + git tag）
2. 写 `10-release.md` 总结：本次 release 内容 + 测试覆盖 + 已知 risk
3. 整理 `autonomous-decisions.md`（自治期间的决策日志）

**⛳ 关卡 2**：用 AskUserQuestion 给 PM：

```
Q: Release PR 准备好了。你看一眼 diff + changelog，决定 merge？
A:
- ✅ Merge - 直接 merge，触发生产部署
- 🔄 Request changes - 描述需要改什么
- ❌ Hold - 暂时不发，等下次
```

PM 选 ✅ → merge → 进入"反馈循环"（监控 Sentry + Langfuse 第一周）
PM 选 🔄 → 回 phase 9 或对应阶段修改
PM 选 ❌ → PR 留着，下次发版再说

---

## 反馈循环（merge 后，无 PM 接触）

- Sentry MCP 持续监控 — 若新 release 后 24h 内 error rate > 阈值，触发 R3 红线
- AI 项目：Langfuse Web UI 看 cost / trace 是否符合预期
- 一周后自动写一份 `post-release-review.md`，准备进入下一轮迭代
