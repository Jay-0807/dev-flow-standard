# Universal Coding Project Development Skill

> **通用代码项目开发工作流编排器** — 一句话需求，自动走完 17 阶段完整开发流水线。
> 业务无关 · 技术栈无关 · 为 [Claude Code](https://docs.claude.com/en/docs/claude-code) Skill 体系设计。

PM 或开发者用一句话描述需求，这个 skill 自动编排「想法 → PRD → 架构 → 设计 → 实施 → 测试 → 上线 → 监控」完整闭环。全程**只在 2 个关键节点**找你拍板（📝 PRD + ☀️ 早晨复盘），其余阶段 autonomous 自治、可过夜跑。

它把行业最佳实践 + [autodev-skills](https://github.com/Leooo-Huang/autodev-skills) 实证设计 + Karpathy 的 4 条 LLM 编码原则，固化成一套可从单一入口跑通的自动编排。

---

## ✨ 它解决什么问题

传统「让 AI 写代码」常见的痛点：

- 一次性丢需求给 AI，**中间过程不可控**，出来的东西经常跑偏；
- 缺少 PRD / 架构 / 验收等**工程化关卡**，质量靠运气；
- AI 容易**偷懒**（占位实现、跳过边界条件、顺手优化无关代码）；
- 每轮迭代**失忆**，上轮修过的坑这轮重新踩。

这个 skill 用「**文档先于代码 + 双关卡守门 + AI 对抗式互审 + 跨迭代经验召回**」把上面每一条都补上，让非技术背景的 PM 也能驱动一个**接近正经研发团队**的开发流程。

---

## 🚀 快速开始

### 1. 安装（作为 Claude Code Skill）

把本仓库克隆到 Claude Code 的 skills 目录即可：

```bash
# macOS / Linux
git clone https://github.com/Jay-0807/universal-coding-project-development-skill.git \
  ~/.claude/skills/universal-coding-project-development-skill
```

```powershell
# Windows (PowerShell)
git clone https://github.com/Jay-0807/universal-coding-project-development-skill.git `
  "$env:USERPROFILE\.claude\skills\universal-coding-project-development-skill"
```

重启 Claude Code 后，skill 会被自动发现并按 `SKILL.md` 的 `description` 触发。

### 2. 触发

在 Claude Code 里直接用自然语言提需求，命中以下任一信号即自动启动：

> 「新需求」「加个功能」「想做个 X」「需求来了」「改一下 X」「迭代 Y」「下次版本」「新一轮开发」「跑完整代码开发流程」
> `new feature request` · `iteration` · `implement X` · `ship X` · `build me a ...`

也可显式调用：`/universal-coding-project-development-skill`

### 3. 不该触发的场景

- 只是**问问题**（"X 是什么"）→ 直接答，别走流水线
- 只想跑 **1 行 PoC** → 直接动手就行
- 在做**调研 / 写文档 / 做 PPT** → 用对应的专门 skill

---

## 🧭 17 阶段工作流

```
💬 需求澄清 ─▶ 👥 用户研究 ─▶ 📝 PRD 撰写 ⛳关卡1(PM)
                                          │
        ┌─────────────────────────────────┘
        ▼   🌙 夜间模式区（PM 离场，仅 4 条红线 escalate，可过夜分批）
   💡 方案发散 ─▶ 📐 影响面分析+ADR ─▶ 🏗️ 架构与接口设计 ─▶ 🎨 界面设计
        │                                                       │
        ▼                                                       ▼
   📚 设计压缩成开发地图 ─▶ 📋 任务拆解 ─▶ ⌨️ 代码实施 ─▶ 🧹 代码债扫描
        │                                                       │
        ▼                                                       ▼
   🔍 多路审查 ─▶ 🛡️ 上线前质量检查(五层验收) ─▶ 📢 发布说明 ─▶ 🔄 漂移检测
        │
        ▼
   🚀 git 发版 ─▶ 🌅 递交摘要
        │
        ▼
   ☀️ 早晨复盘 ⛳关卡2(PM) ─▶ ✅ merge / 🔄 局部redo / ❌ 整体重做 / ⏸ 推下夜
```

> 完整带内部编号（Phase 0–13）的版本、每阶段输入输出一览表见
> [`ARCHITECTURE.md`](ARCHITECTURE.md) 和 [`phases/MANIFEST.md`](phases/MANIFEST.md)。

**核心特点**：PRD ⛳ 通过后**一气呵成自动跑到 🚀 git 发版**，PM 第二天早上回来只需走 ☀️ 早晨复盘关卡验收。不存在 fast-track / standard / manual 多模式，就这一条路径。

---

## 🧱 8 大模块

| # | 模块 | 做什么 | 位置 |
|---|------|--------|------|
| 1 | **API 整理** | UI 反推 API + 存量审计 + 文档导出 + 跨迭代注册表 | `phases/04-architecture-and-api.md` §2 |
| 2 | **夜间模式** | 重命名 + 早晨复盘 + 决策回放 + 跨夜分批 | `night-mode.md` · `decision-replay.md` · `night-mode-batching.md` |
| 3 | **Autopilot** | AI 主动捡需求自动跑（5 connector + 12 状态机 + 6 安全闸） | `autopilot/` |
| 4 | **GAN 引擎** | AI 对抗式互审：1 生成 + 1 怀疑 reviewer + 4 维评分 + PIVOT 重写 | `gan-engine/` |
| 5 | **文档压缩** | 把厚设计文档压成 INDEX + RULES（"地图不是摘要"） | `phases/05.9-compress.md` |
| 6 | **方案发散** | DIVERGE 3–5 方案 → EVALUATE 4 维 → CONVERGE 选最优 | `phases/02.5-brainstorm.md` |
| 7 | **五层验收** | 契约 / 红线编号 / 静态 / 运行时 / acceptance | `phases/10-verify.md` |
| 8 | **漂移检测** | 文档↔代码 5 维度比对 + INFO/WARN/ERROR 三级警报 | `phases/11.5-sync.md` |

---

## 🛡️ 2 个 PM 关卡 + 4 条红线

只在两处需要 PM 拍板，其余全自治：

- **⛳ 关卡 1 — 📝 PRD**：三选项确认（✅ 通过 / 🔄 局部改 / ❌ 重写），过 GAN 引擎 + Karpathy 4 原则自检。
- **⛳ 关卡 2 — ☀️ 早晨复盘**：4 步结构化清单（决策审计 / 审查 should-fix / 用户验收 / Karpathy 违规）+ 4 选项收尾。

夜间自治期间，只有撞上 **4 条红线**才会把 PM 叫醒：

| 红线 | 触发条件 |
|------|----------|
| **R1** | 重大架构冲突 / vendor lock-in / breaking change |
| **R2** | 安全 must-fix > 3 项 |
| **R3** | 上线前五层验收重试 3 次仍挂 |
| **R4** | 必须删除既有功能 / 漂移检测 ERROR 级 |

---

## 🧠 内置原则：Karpathy 4 条 LLM 编码准则

所有阶段默认遵守（完整版见 [`principles/karpathy-llm-coding.md`](principles/karpathy-llm-coding.md)）：

1. **Think Before Coding** — 显性化假设、给多种解释、必要时 push back，不默默推进
2. **Simplicity First** — 最少代码解决问题，反过度工程 / 反预留扩展 / 反多余抽象
3. **Surgical Changes** — 只改任务必需的部分，不顺手优化无关代码
4. **Goal-Driven Execution** — 给可验证的成功标准而非指令清单，循环迭代直到达成

---

## 🔌 编排（不替代）的本机资源

这个 skill 是**编排器**，优先调度本机已装的 skill / MCP，而非重复造轮子。缺失时对应阶段**降级而非失败**：

- **Skills**：obra/superpowers（brainstorm / writing-plans / subagent-driven-development）· adr-skill · review / security-review · release-please · `/autodev-*`
- **MCP**：Figma（界面设计）· Playwright（运行时验收）· Sentry（监控）· Notion（用户研究 / autopilot）· GitHub（发版）· CodeGraph（影响面 / 调用图 / 漂移验证，条件式）

完整清单见 [`reference/tool-inventory.md`](reference/tool-inventory.md)。

---

## 📂 目录结构

```
SKILL.md                  ← 主入口（先读这个）
ARCHITECTURE.md           ← 5 分钟看懂全貌
phases/                   ← 每个步骤一个文件 + MANIFEST.md（17 步 I/O 一览表）
gan-engine/               ← AI 对抗式互审引擎（quality-redlines.md 是红线 SSOT）
autopilot/                ← AI 主动捡需求模式（独立入口，非默认）
integrations/             ← 8 个外部能力内化（api-architect / git-workflow / owasp-llm 等）
principles/               ← Karpathy 4 LLM 编码原则
reference/                ← slash 命令表 / 项目类型路由 / 工具清单 / 红线判定
templates/                ← 15 个产出模板（PRD / API spec / RUN-LOG / ...）
night-mode.md             ← 夜间自动跑机制
gates.md                  ← PM 关卡 + 4 红线 escalation
decision-replay.md        ← 决策回放 checkpoint
```

支持 **3 种项目类型自动路由**（Web 全栈 / B 端 SaaS / AI 原生应用），按类型注入不同默认，详见 [`reference/project-type-router.md`](reference/project-type-router.md)。

---

## 🆚 与业务特化姊妹版的区别

本 skill（universal）是**纯通用版，不注入任何业务上下文**，适用于任何代码项目。

作者另有一个 `firefly-web-os-orchestrator-skill` 是**业务特化版**（绑定特定项目的业务哲学 / 客户 / 协议），仅在该项目目录下使用。两者是**兄弟关系、不互相调用**，靠工作目录特征（如 `project.config.yaml`）自动区分。

---

## ⚠️ 限制与反方观点

- **不是银弹**：17 阶段适合「正经迭代」，1 行 PoC 直接动手更快。
- **依赖本机资源**：superpowers / adr-skill / 各 MCP 缺失时对应阶段降级。
- **AI 项目分支不完整**：Langfuse 等 trace/cost 能力部分仍需 Web UI。
- **自治有风险**：4 红线 + 早晨复盘是兜底，PRD 表达模糊仍可能偏离 → 在需求澄清/PRD 阶段充分对齐最关键。
- **GAN 引擎有 token 成本**：标准迭代 ~214 次调用，完整迭代 ~380 次；可关闭部分阶段的 GAN 省约 30%。

---

## 📖 进一步阅读

| 想了解 | 看这个 |
|--------|--------|
| 5 分钟全貌 | [`ARCHITECTURE.md`](ARCHITECTURE.md) |
| 17 步输入输出一览 | [`phases/MANIFEST.md`](phases/MANIFEST.md) |
| 防偷懒规则（红线 SSOT） | [`gan-engine/quality-redlines.md`](gan-engine/quality-redlines.md) |
| 过夜跑机制 | [`night-mode.md`](night-mode.md) |
| 关卡 + 红线细节 | [`gates.md`](gates.md) |
| Autopilot 完整说明 | [`autopilot/README.md`](autopilot/README.md) |

---

<sub>一个为 Claude Code Skill 体系打造的个人项目。设计上整合了 [autodev-skills](https://github.com/Leooo-Huang/autodev-skills) 的实证方法、Andrej Karpathy 的 LLM 编码原则与 OWASP LLM Top 10 安全清单。</sub>
