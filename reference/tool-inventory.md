# Tool Inventory — 本 skill 调用的所有资源

按"必装 / 可选 / 缺失降级"分类。

## 必装（缺失会导致 skill 大幅降级）

| 资源 | 类型 | 阶段 | 缺失降级方式 |
|---|---|---|---|
| obra/superpowers | skill 包（含 /brainstorm 等） | 1, 7, 9 | Claude 直接做澄清问答，质量略低 |
| adr-skill | skill | 3 | Claude 直接写 MADR 格式 ADR，无模板 |
| dispatching-parallel-agents | skill | 6 | 串行实施，慢 50% |
| anthropic-skills:review | skill | 9 | Claude 自查 diff |
| anthropic-skills:security-review | skill | 7c | OWASP Top 10 手动 checklist |

## P0 MCP（强烈建议装）

| MCP | 工具数 | 阶段 | 缺失降级 |
|---|---|---|---|
| Figma MCP（mcp__e6c31b9d__*） | 17 | 4 | 让 PM 自己描述 UI |
| Playwright MCP（mcp__playwright__*） | 23 | 7a | Claude 写 unit test 不写 E2E |
| Sentry MCP（mcp__sentry__*） | 22 | 8 | Claude 输出"手动接入 Sentry"指南 |

**本机当前状态（2026-05-19 实测）**：
- ✅ Figma MCP — 已装（17 工具可用）
- ✅ Playwright MCP — 已装（23 工具可用）
- ✅ Sentry MCP — 已装（22 工具可用，EU 区域 firefly-hy org）

## 可选 skill（按项目需要）

| Skill / 工具 | 阶段 | 触发条件 |
|---|---|---|
| autodev `/autodev-iterate` | 6 | 增量修改场景 |
| autodev `/autodev-ui` | 4, 6 | 前端页面密集 |
| autodev `/autodev-api` | 5 | API 密集项目 |
| release-please | 10 | 默认发版工具 |
| semantic-release | 10 | 如不用 release-please |
| anthropic-skills:docx / pptx / xlsx | - | 输出 deliverable 时 |

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
