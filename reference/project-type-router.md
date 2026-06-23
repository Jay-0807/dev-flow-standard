# Project Type Router — 3 种项目类型的阶段差异（v4 更新）

启动时**主动判定项目类型**，影响 v4 17 phase 的子动作（重点：Phase 4 / 5 / 7 / 8 / 10）。

## 判定方法

### Step 1：主动检测（不打扰 PM）

Read 工作目录下的清单文件：
- `package.json` → 看 dependencies
  - 有 `next` / `react` / `vue` → Web 全栈（强信号）
  - 有 `langchain` / `@langchain/*` / `langgraph` / `@anthropic-ai/sdk` / `openai` → AI 应用（强信号）
- `pyproject.toml` / `requirements.txt` → 看依赖
  - 有 `fastapi` / `django` + 前端目录 → Web 全栈
  - 有 `langchain` / `crewai` / `langgraph` → AI 应用
- `Cargo.toml` / `go.mod` / `pom.xml` → 通常是后端服务
- `.firefly` / 工作目录在 firefly path 下 → **停止，提示用户改用 firefly 业务特化版**

### Step 2：兜底问 PM

如果自动检测**不确定**，用 AskUserQuestion：

```
Q: 这个项目主要是什么类型？
A:
- Web 全栈（前端 + 后端 + DB）
- B 端 SaaS（同 Web 全栈 + 多租户/权限）
- AI 原生应用（LLM 为核心，可能没传统 UI）
- 其他（请描述）
```

## 三种类型的阶段差异

### 类型 A：Web 全栈（默认）

技术栈典型：Next.js / Nuxt / Remix / Rails / Django + React

| Phase | 默认动作 | 备注 |
|---|---|---|
| 4 架构 | + 前端架构 + 后端架构分别设计 | 必做 |
| **4 §2 API 整理** 🆕 | REST 为主；GraphQL 仅在多 client 复杂查询时考虑 | API 9 维审计 |
| 5 §1 UX | ✅ Figma MCP 拉低保真线框 | 必做 |
| 5 §2 UI Spec | ✅ 三态强制（空/错/loading）| 必做 |
| **5.9 文档压缩** 🆕 | INDEX 含路由表 + 组件树 | 必做 |
| 7 测试 | Playwright Test Agents（E2E 浏览器） | 主要 |
| 7 安全 | OWASP Top 10 + Semgrep | XSS / SQL injection / CSRF 重点 |
| 8 可观测 | Sentry（前端 + 后端 SDK） | Web Vitals 监控 |
| **10 五层验收** 🆕 | L4 Playwright 强制 + L5 acceptance-testing | UI 必走真实浏览器 |
| **11.5 漂移检测** 🆕 | 5 维度（页面路由维度必查）| - |

### 类型 B：B 端 SaaS

类似 Web 全栈，**多加**：

| Phase | 额外动作 |
|---|---|
| 3 ADR | 多租户隔离方案（schema-per-tenant / row-level / column）必出 ADR |
| 4 架构 | DB 设计加 tenant_id + RLS 策略 |
| **4 §2 API 整理** 🆕 | REST + GraphQL（B 端工作台常多端复用，GraphQL 减少 over-fetching）|
| 5 §1 UX | autodev-ui task-first 信息架构强制（B 端工作台反对全量展示）|
| 6 实施 | 权限矩阵（RBAC / ABAC） |
| 7 安全 | 加跨租户越权测试（IDOR）|
| 8 可观测 | Sentry 按 tenant 分组 + 关键操作审计日志 |
| **12.5 早晨复盘** 🆕 | B 端 SaaS：12.5 早晨复盘强制真人用户视角（邀真实 B 端用户复盘，payback 高）|

### 类型 C：AI 原生应用

LLM 为核心，可能纯 backend / API 模式。

| Phase | 调整 |
|---|---|
| **2.5 brainstorm** 🆕 | DIVERGE 多走"LLM 实时 vs 规则 vs 混合"几种方案 |
| 4 架构 | + Langfuse Prompt Management + LiteLLM Gateway 集成方案 |
| **4 §2 API 整理** 🆕 | REST + Server-Sent Events (SSE) 流式响应必做；可能含 agent-to-agent 协议（如有自研，定义信封格式）|
| 5 §1 UX | ⚠️ **可选** — 纯 API 产品跳过；有 chat UI 仍做 Figma |
| 5 schema | 加 prompts 表 / runs 表 / evaluations 表 |
| 6 实施 | + LangGraph 编排（多 agent 场景）/ + 状态机记录 |
| 7 测试 | **加 Promptfoo** prompt 回归测评（YAML 声明式）|
| 7 安全 | **加 Garak**（广扫描）+ **PyRIT**（精攻）红队三件套；OWASP LLM Top 10 |
| 7 代码债 | 加"prompt 硬编码"扫描（应该走 Langfuse 而非 inline）|
| 8 可观测 | **加 Langfuse**（Web UI 看 trace/cost）+ **LiteLLM** gateway |
| 9 审查 | 加幻觉检测（Vectara HHEM 离线评估）|
| **10 五层验收** 🆕 | L4 加 LLM 输出回归 + Promptfoo 跑 |
| **autopilot** 🆕 | 候选种子 PRD 评分公式中"LLM 调用成本"加权 |

⚠️ **AI 项目特殊提示**：
- 当前 Langfuse 官方 MCP 仅 Prompt Management，**trace/cost 走 Web UI**（不能让 Claude 直接调）
- Promptfoo / Garak / PyRIT 都是 **CLI 工具非 MCP**，调用要走 Bash
- LLM 输出 API 强制加 `confidence` 字段（质量红线建议启用）

---

## 输出

在 `00-intake.md` 顶部加一行：
```
Project type: <Web 全栈 | B 端 SaaS | AI 原生应用 | 其他: ...>
```

之后所有阶段的子动作都按这个类型路由。GAN 引擎的 reviewer prompt 也会按此类型注入不同视角段。

---

## v4 项目类型敏感的 8 模块行为

| 模块 | 项目类型敏感性 |
|---|---|
| 模块 1 API 整理 | ✅ 高（REST/GraphQL/SSE 按类型选）|
| 模块 2 夜间模式 | 低（机制通用）|
| 模块 3 Autopilot | 中（AI 项目种子 PRD 加 LLM cost 维度）|
| 模块 4 GAN 引擎 | 中（reviewer 视角注入按类型）|
| 模块 5 文档压缩 | 低（INDEX/RULES 内容差异由 phase 数据驱动）|
| 模块 6 方案发散 | 中（AI 项目 DIVERGE 维度不同）|
| 模块 7 五层验收 | ✅ 高（L4 runtime 检查内容按类型）|
| 模块 8 漂移检测 | 低（5 维度通用）|
