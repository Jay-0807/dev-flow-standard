# Tool Inventory — 本 skill 调用的所有资源（v4 更新）

按"必装 / 可选 / 缺失降级"分类。**v4 新增**：GAN 引擎 / Autopilot / autodev-skills 集成精华（compress / brainstorm / verify / sync）。

## 必装（缺失会导致 skill 大幅降级）

| 资源 | 类型 | Phase | 缺失降级方式 |
|---|---|---|---|
| obra/superpowers | skill 包（含 /brainstorm 等） | 1, 7, 9 | Claude 直接做澄清问答，质量略低 |
| adr-skill | skill | 3 | Claude 直接写 MADR 格式 ADR，无模板 |
| dispatching-parallel-agents | skill | 7 | 串行实施，慢 50% |
| anthropic-skills:review | skill | 9 | Claude 自查 diff |
| anthropic-skills:security-review | skill | 9 | OWASP Top 10 手动 checklist |

## v4 内化资源（本 skill 自带，无需安装）

| 资源 | Phase | 说明 |
|---|---|---|
| **integrations/gan-engine.md** + `gan-engine/`（5 文件）🆕 | 2/4/4.5/5a/6/7/11 跑 GAN | 1 gen + 1 怀疑 reviewer + 4 维 + 7 redlines + PIVOT |
| **autopilot/** 整目录（17 文件）🆕 | 主流水线外 | 5 connector + 6 安全闸 + 状态机 + Tier 切换 |
| **night-mode.md** + `decision-replay.md` + `night-mode-batching.md` 🆕 | 2.5-12 | 夜间模式 ABCD 四件套 |
| **integrations/api-architect.md** 🆕 | 4.5 | API 9 维度审计 + UI 反推 API 铁律 |
| **integrations/database-architect.md** | 4 | DB 9 维度审计 |
| **integrations/tech-debt-9d.md** | 8 | 代码债 9 维度 |
| **integrations/owasp-llm-2025.md** | 9 | LLM 安全 OWASP Top 10 |
| **integrations/test-planner.md** | 10 | 测试策略层 |
| **integrations/git-workflow.md** | 12 | GitHub Flow + gh CLI |
| **integrations/release-please.md** | 12 | Conventional Commits + 自动发版 |
| **principles/karpathy-llm-coding.md** | 全 phase | Karpathy 4 LLM 编码原则 |

## P0 MCP（强烈建议装）

| MCP | 工具数 | Phase | 缺失降级 |
|---|---|---|---|
| Figma MCP（mcp__e6c31b9d__*） | 17 | 5a/5b | 让 PM 自己描述 UI |
| Playwright MCP（mcp__playwright__*） | 23 | 10/12.5 | Claude 写 unit test 不写 E2E |
| Sentry MCP（mcp__sentry__*） | 22 | 8 + autopilot connector | Claude 输出"手动接入 Sentry"指南 |
| Notion MCP 🆕 | - | autopilot connector | 仅 roadmap connector，跳过 Notion |
| Gmail MCP 🆕（可选）| - | autopilot 用户反馈 | 跳过邮件源 |
| CodeGraph MCP 🆕（条件式）| 10 | 3 影响面 / 4 §2 API 调用图 / 11.5 漂移 | 项目未 codegraph init 时 fallback Explore+Grep |

**本机当前状态（2026-05-19 实测）**：
- ✅ Figma MCP — 已装（17 工具可用）
- ✅ Playwright MCP — 已装（23 工具可用）
- ✅ Sentry MCP — 已装（22 工具可用）
- ✅ Notion MCP — 已装
- ✅ CodeGraph MCP — 已装（2026-05-30，10 工具，条件式，项目已索引后可用）

## 可选 skill（按项目需要）

| Skill / 工具 | Phase | 触发条件 |
|---|---|---|
| autodev `/autodev-ideation` | 2 | PRD 起草 |
| autodev `/autodev-iterate` | 7 | Phase 7 实施每 code task |
| autodev `/autodev-ui` | 5a/5b | 前端页面密集 |
| autodev `/autodev-api` | 4.5 | Phase 4 §2 API 设计 |
| autodev `/autodev-verify` | 10 | Phase 10 五层验收 |
| autodev `/autodev-review` | 9 | Phase 9 三路审查 |
| release-please | 12 | 默认发版工具 |
| semantic-release | 12 | 如不用 release-please |
| anthropic-skills:docx / pptx / xlsx | 2 / 11 | 输出 deliverable 时 |
| /loop | 7 (over-night) | Phase 7 过夜跑 |
| /schedule | autopilot | Autopilot 定时唤醒 |

## AI 项目专项工具（仅 AI 原生应用类型用）

| 工具 | 类型 | 阶段 | 接入方式 |
|---|---|---|---|
| Promptfoo | CLI | 7a | `npx promptfoo eval` |
| Garak | CLI (Python) | 7c | `garak --model_type X --probes Y` |
| PyRIT | CLI (Python) | 7c | Python script |
| Ragas | Python lib | 7a | 嵌入测试代码 |
| LangGraph | npm/Python lib | 6 | 多 agent 编排时 import |
| Langfuse | Web UI（MCP 暂仅 prompt） | 8 | SDK init + Web UI 看 trace/cost |
| LiteLLM | gateway + SDK | 8 | 替代直接 SDK 调用 |
| Vectara HHEM | HF 模型 | 9 | 离线幻觉评估 |

## 调用规范

### 调 skill
- 用关键词触发（如 PM 说"加个功能" → 自动入本 skill）
- 子 skill 通过 Read `~/.claude/skills/<name>/SKILL.md` 加载指令，按其工作流执行

### 调 MCP 工具
- 必须先 ToolSearch 加载 schema：`ToolSearch(query="select:mcp__xxx__yyy,mcp__xxx__zzz")`
- 加载后直接调用，schema 在 deferred tools 列表

### 调 CLI 工具
- 用 Bash / PowerShell（按平台）
- 优先用绝对路径（如 `C:\Program Files\nodejs\npx.cmd`），UWP 沙盒下 PATH 可能不全
- 长时命令用 `run_in_background: true`

## 资源检查

skill 启动时，**先检查必装资源是否齐全**：

```python
required_skills = ["obra/superpowers", "adr-skill", "anthropic-skills:review"]
required_mcps = ["mcp__e6c31b9d__use_figma", "mcp__playwright__browser_navigate", "mcp__sentry__whoami"]

for s in required_skills:
    if s not in available_skills:
        log_warning(f"Skill {s} 未装，对应阶段会降级")

for m in required_mcps:
    if m not in available_tools:
        log_warning(f"MCP {m} 未装，对应阶段会降级")
```

降级不阻塞 skill 运行，只是质量下降。Claude 主动告诉 PM 哪些降级了。

## 完整调研依据

所有工具选择来自：`D:\cursor_project\research-vault\universal-dev-workflow\report.md`（2026-05-18，含 24 个模块 + 7 个冲突标注 + WebFetch 实测修正）
