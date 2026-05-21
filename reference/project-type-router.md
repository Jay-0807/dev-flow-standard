# Project Type Router — 3 种项目类型的阶段差异

启动时**主动判定项目类型**，影响阶段 4 / 7 / 8 的子动作。

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
- `.firefly` / 工作目录在 firefly path 下 → **停止，提示用户改用 firefly orchestrator**

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

| 阶段 | 默认动作 | 备注 |
|---|---|---|
| 4 UI | ✅ Figma MCP 拉设计 | 必做 |
| 7 测试 | Playwright Test Agents（E2E 浏览器） | 主要 |
| 7 安全 | OWASP Top 10 + Semgrep | XSS / SQL injection / CSRF 重点 |
| 8 可观测 | Sentry（前端 + 后端 SDK） | Web Vitals 监控 |

### 类型 B：B 端 SaaS

类似 Web 全栈，**多加**：

| 阶段 | 额外动作 |
|---|---|
| 3 ADR | 多租户隔离方案（schema-per-tenant / row-level / column）必出 ADR |
| 5 schema | 所有表加 tenant_id + RLS 策略 |
| 6 实施 | 权限矩阵 implementation（RBAC / ABAC） |
| 7 安全 | 加跨租户越权测试（IDOR） |
| 8 可观测 | Sentry 按 tenant 分组 + 关键操作审计日志 |

### 类型 C：AI 原生应用

LLM 为核心，可能纯 backend / API 模式。

| 阶段 | 调整 |
|---|---|
| 4 UI | ⚠️ **可选** — 纯 API 产品跳过；有 chat UI 仍做 Figma |
| 5 schema | 加 prompts 表 / runs 表 / evaluations 表 |
| 6 实施 | + LangGraph 编排（多 agent 场景）/ + 状态机记录 |
| 7 测试 | **加 Promptfoo** prompt 回归测评（YAML 声明式） |
| 7 安全 | **加 Garak**（广扫描）+ **PyRIT**（精攻）红队三件套；OWASP LLM Top 10 |
| 7 代码债 | 加"prompt 硬编码"扫描（应该走 Langfuse 而非 inline） |
| 8 可观测 | **加 Langfuse**（Web UI 看 trace/cost）+ **LiteLLM** gateway |
| 9 审查 | 加幻觉检测（Vectara HHEM 离线评估） |

⚠️ **AI 项目特殊提示**：
- 当前 Langfuse 官方 MCP 仅 Prompt Management，**trace/cost 走 Web UI**（不能让 Claude 直接调）
- Promptfoo / Garak / PyRIT 都是 **CLI 工具非 MCP**，调用要走 Bash

## 输出

在 `01-canonical-query.md` 顶部加一行：
```
Project type: <Web 全栈 | B 端 SaaS | AI 原生应用 | 其他: ...>
```

之后所有阶段的子动作都按这个类型路由。
