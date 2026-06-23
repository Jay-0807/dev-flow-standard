---
name: universal-coding-project-development-skill
description: 通用代码项目开发工作流编排器 v5（业务无关、技术栈无关）。PM 或开发者一句话需求，按体量与风险自动分级（🏃快车道 / 🚶标准 / 🐢重型）：中小且无风险面的需求走快车道（砍掉用户研究/方案发散/ADR/厚PRD，2 个 PM 关卡默认静默，一句话需求→直接出可跑代码），其余走完整 17 阶段流水线（Phase 1/1.5/2/2.5/3/4/5/5.9/6/7/8/9/10/11/11.5/12/12.5，+13 autopilot）。⚠️ 凡命中风险面（钱/认证/并发/数据完整性/合规）即便任务很小也强制保留对抗式 GAN——按 SIZE 定深度，按 RISK 定是否保 GAN。8 大模块：API 整理 + 夜间模式 + Autopilot + GAN 引擎 + 文档压缩 + 方案发散 + 五层验收 + 漂移检测。2 个 PM 关卡（PRD + 早晨复盘），其余 autonomous 自治。支持 3 种项目类型分支（Web 全栈 / B 端 SaaS / AI 原生应用）自动路由。本 skill 编排（不替代）本机已装的 superpowers / adr-skill / Figma MCP / Playwright MCP / Sentry MCP 等。整合 Leooo-Huang/autodev-skills 实证设计（compress / brainstorm / verify / sync）+ Karpathy 4 LLM 编码原则 + 7 Quality Redlines。⚠️ 与 firefly-web-os-orchestrator-skill 严格区分：本 skill 不注入任何业务上下文；firefly 业务特化版 是带 项目业务哲学（如有） / 电商客户 / 项目业务协议（如有）的业务特化版，仅在 项目目录下使用。触发关键词："新需求"、"加个功能"、"想做个 X"、"需求来了"、"改一下 X"、"迭代 Y"、"下次版本"、"新一轮开发"、"跑完整代码开发流程"、"完整跑一遍"、"PRD please"、"new feature request"、"iteration"、"feature please"、"implement X"、"ship X"、"build me a"。
---

# Universal Coding Project Development Skill v5 — 通用代码项目开发工作流

> **新手 5 分钟上手** → 先读 `ARCHITECTURE.md`（30 行顶层导航）+ `phases/MANIFEST.md`（17 步骤 I/O 一表），再回来读本文件。
>
> **业务名 vs 内部编号**：对 PM 暴露的所有文档（RUN-LOG / DEBUG-TRACE / 决策日志 / 早晨复盘）**必须用业务名 + emoji**（如 📝 PRD 撰写 / 🛡️ 上线前质量检查），禁止用 "Phase X" 编号。映射表见 `phases/MANIFEST.md`。
>
> **v5 = v4 + 风险分级快车道**（v4 = v2 的 13 phase + 8 大模块：API 整理 / 夜间模式 v2 / Autopilot / GAN 引擎 / 文档压缩 / 方案发散 / 五层验收 / 漂移检测）。v5 在其上加「按 SIZE 定深度 / 按 RISK 保 GAN」的快车道路由。
>
> 从 firefly-web-os-orchestrator-skill v4 backport，去除 firefly 业务上下文（r4 哲学 / 项目业务协议（如有） / 电商客户），保留所有通用机制。

## 这个 skill 解决什么问题

PM 或开发者提出代码需求时，走完"想法 → PRD → 架构 → 设计 → 实施 → 测试 → 上线 → 监控"完整闭环。本 skill 把行业最佳实践 + autodev-skills 实证设计 + Karpathy 4 LLM 编码原则固化成自动编排，从单一入口跑通整个代码项目。

## 与 firefly-web-os-orchestrator-skill 的关系

| 维度 | 本 skill（universal） | firefly 业务特化版 |
|---|---|---|
| 业务上下文注入 | ❌ 无（通用） | ✅ 项目业务哲学（如有） + 电商客户 + 项目业务协议（如有） |
| 适用项目 | 任何代码项目 | 仅 项目 |
| 项目类型分支 | ✅ 3 种自动路由（Web 全栈/B 端 SaaS/AI 原生）| 1 种（firefly Next.js + A2A）|
| GAN 引擎 reviewer 视角 | Karpathy + 通用安全 + 维护性 | 同 + r4 哲学 + 协议合规 |
| Quality Redlines | R1-R7（autodev 7 红线）| R1-R7 + R8 r4 三字段 + R9 协议合规 |

**两个 skill 不互相调用**，是兄弟关系不是父子。判断规则：工作目录是否含 项目特征（project.config.yaml / firefly-context/ 目录）。

## 何时触发

✅ **应该触发**：
- "新需求" / "加个功能" / "想做个 X" / "ship 一个 X"
- "需求来了" / "下次版本" / "新一轮开发"
- "改一下 X" / "迭代 Y" / "implement X"
- PM 或开发者用一句话描述了**代码层面**要做的事

❌ **不应该触发**：
- 用户在问问题（"X 是什么"）→ 直接答
- 用户在做调研 → deep-research（通用深度调研 skill）
- 用户写文档 / 邮件 / PPT → 对应文档 skill
- 用户在 项目目录里提需求 → firefly-web-os-orchestrator-skill
- 用户只想跑一行 PoC → 直接动手，不要走流水线

---

## 六条核心原则（v5 更新）

0. **风险分级路由（v5 新增，最高优先）** — 启动时自动判 **SIZE**（XS/S/M/L）与 **RISK**（是否碰钱/认证/并发/数据完整性/合规）。**深度按 SIZE 定，是否保留 GAN/红线按 RISK 定，两者正交**。中小且无风险 → 🏃 快车道（砍用户研究/发散/ADR/厚PRD，2 关卡静默，直接出码）；命中风险面 → 即便任务小也**强制保留对抗式 GAN**（实证：碰钱/认证的任务"看着小、藏着坑"，按 SIZE 关 GAN 最致命）。完整路由矩阵见 `reference/risk-fast-lane.md`。
1. **2 关卡守门** — **关卡 1 PRD ⛳ + 关卡 2 早晨复盘 ⛳**（Phase 12.5）。其余 autonomous 自治。AskUserQuestion 给选项（PRD 3 选项 / 早晨复盘 4 选项）。**快车道下两关卡默认静默自动 ✅**（PM 可在启动时显式要求打开）。
2. **文档先于代码** — 每个关卡先在 `iteration-vault/` 落地 .md 文档。**v4 新增**：Phase 5.9 把厚设计文档压缩成 INDEX + RULES（"地图不是摘要"），实施 phase 强制读这两份不读全量。
3. **本机优先 + 外部融合** — 优先用本机已装 skill（superpowers/autodev/角色 agent）；本机空白处用 `integrations/` 内化精华。**v4 新增**：integrations/gan-engine.md（对抗式生成）+ integrations/api-architect.md（API 9 维审计）。
4. **项目类型敏感** — 启动时主动检测项目类型（Web 全栈 / B 端 SaaS / AI 原生），按类型注入不同默认（详见 `reference/project-type-router.md`）。
5. **用户视角友好** — Phase 1.5 用户研究证据 / Phase 5 §1 UX 必经 / Phase 12.5 ⛳ 早晨复盘覆盖真人用户验收角色（4 步结构化清单 + 4 选项）。**技术验收通过 ≠ 用户验收通过**。

---

## 全局 LLM 编码准则（Karpathy 4 原则）

本 skill **所有 phase** 默认遵守 Andrej Karpathy 关于 LLM 编码的 4 条全局准则（完整版见 `principles/karpathy-llm-coding.md`）：

1. **Think Before Coding** — 显性化假设、给多种解释、必要时 push back，不默默推进
2. **Simplicity First** — 最少代码解决问题，反过度工程 / 预留扩展 / 多余抽象
3. **Surgical Changes** — 只改任务必需的部分，不顺手优化无关代码
4. **Goal-Driven Execution** — 给可验证的成功标准而非指令清单，循环迭代直到达成

**速查**："不要不浮现假设 / 不要预留未来扩展 / 不要顺手优化无关代码 / 不要不给成功标准"。

---

## 开发路径（3 条 lane，按 SIZE/RISK 自动路由，PM 不用选）

启动时自动分级 → 落到三条 lane 之一，**全部 default-autonomous**（PM 不手动选 lane，系统判；判错可在执行中自动升级）：

- 🏃 **快车道**（XS/S 且无风险）：💬 轻澄清 → 📚 INDEX+RULES → ⌨️ 实施 →（命中风险则内联 GAN）→ 🛡️ 上线前检查 → 🚀 发版。2 个 PM 关卡**默认静默自动 ✅**，一句话需求直接出可跑代码。
- 🚶 **标准**（M）：完整 17 阶段，PRD ⛳ 通过后自动跑完到 🚀，PM 早上走 ☀️ 早晨复盘。
- 🐢 **重型**（L）：17 阶段 + 跨夜分批 + 完整 GAN + 12.5 真人验收角色。

**升级闸**：快车道执行中若改动超出 S / 触发红线 / 风险 GAN 判 FAIL，则**自动升级到标准 lane**——按 `reference/risk-fast-lane.md` 的「🔁 升级回流表」补跑被砍 phase（复用已写 prototype + INDEX/RULES，不推倒重来；2 个 PM 关卡恢复，PRD 关卡回头补做）。完整矩阵 + 回流表见该文件。

Autopilot（AI 主动捡需求开始跑）是**另一条入口**（cron tick 触发），复用本路由但启动方式不同。详见 `autopilot/README.md`。

---

## 完整工作流（v5：17 步骤 + 2 PM 关卡 + 8 模块）

> **下面 ASCII 图保留内部 Phase 编号便于技术追溯**。PM 视角的业务名版（💬 需求澄清 / 🏗️ 架构与接口设计 / 等）见 `phases/MANIFEST.md`。
>
> **v5 vs v4 关键变化**：删 Phase 10.5 真人验收 / 合并 0+1 / 合并 4+4.5 / 合并 5a+5b / 删 PRD 关卡批数跟进问。

```
PHASE 1   💬 需求澄清（含原 Phase 0 接收 + 大小分级 + 项目类型路由）
   │  调用：superpowers:brainstorming + product-sprint-prioritizer
   │  产出：iteration-vault/01-clarified-requirement.md
   ▼  💬 PM 在线
PHASE 1.5 👥 用户研究
   │  产出：iteration-vault/01.5-user-research.md（3 画像 + 痛点 + 旅程）
   ▼  💬 PM 在线
PHASE 2   📝 PRD 撰写 ⛳ 关卡 1（PM 必参与，GAN 引擎）
   │  调用：/autodev-ideation + product-sprint-prioritizer + docx + GAN
   │  产出：iteration-vault/02-PRD.md + 02-prd-gan/
   ▼  ⛳ PM 三选项 → 自动启动夜间模式（不再问批数，Phase 6 自动估算）
═══════════════════════════════════════════════════════════════
🌙 夜间模式区（Phase 2.5-12，PM 离场，仅 4 红线 escalate；可分批）
═══════════════════════════════════════════════════════════════
PHASE 2.5 💡 方案发散（学 autodev-brainstorm）
   │  GAN：DIVERGE 3-5 方案 → EVALUATE 4 维 → CONVERGE 选最优
   │  产出：iteration-vault/02.5-brainstorm-{diverge,converge}.md
   ▼
PHASE 3   📐 影响面分析 + ADR
   │  调用：engineering-codebase-onboarding-engineer + adr-skill + CodeGraph MCP（若已索引）
   │  产出：iteration-vault/03-impact-analysis.md + 03-adr.md
   ▼
PHASE 4   🏗️ 架构与接口设计（含原 4.5 API 整理，GAN）
   │  内部两步：§1 大架构蓝图 / §2 API 整理 + 注册表
   │  调用：writing-plans + database-architect + api-architect + /autodev-api + GAN + CodeGraph MCP（若已索引）
   │  产出：iteration-vault/04-architecture-and-api.md + 04-api-spec.yaml + 04-api-postman.json + 04-api.md
   │       + 持久化追加 <project-root>/api-registry.md
   ▼  🚨 R1 检查（含 API 子检查）
PHASE 5   🎨 界面设计（含原 5a UX + 5b UI Spec，GAN）
   │  内部两步：§1 UX 用户视角 wireframe / §2 UI 工程视角技术 spec（含空/错/loading 三态强制）
   │  调用：Figma MCP + Nielsen 10 + /autodev-ui + GAN
   │  产出：iteration-vault/05-interface-design.md + 05-wireframes/ + 05-gan/
   ▼
PHASE 5.9 📚 设计压缩成开发地图（"地图不是摘要"）
   │  产出：iteration-vault/INDEX.md (<100 行) + RULES.md (<80 行)
   │  Phase 6+ 强制读这两份，不读全量
   ▼
PHASE 6   📋 任务拆解 + sprint 排期（GAN + 自动估算工时）
   │  调用：superpowers:writing-plans + product-sprint-prioritizer + GAN
   │  产出：iteration-vault/06-task-breakdown.md
   │  < 8h 单批跑 / ≥ 8h 主动提示是否分批
   ▼
PHASE 7   ⌨️ 代码实施（三路 worktree 并行 + 每 code task 跑 GAN）
   │  调用：subagent-driven-development + /autodev-iterate + GAN（per task）
   │  产出：iteration-vault/07-implementation-log.md + 07-code-task-*-gan/
   ▼  🚨 R4 检查
PHASE 8   🧹 代码债扫描
   │  调用：simplify + integrations/tech-debt-9d
   │  产出：iteration-vault/08-tech-debt-audit.md
   ▼
PHASE 9   🔍 多路审查（即 Global GAN）
   │  并行：code-reviewer / /autodev-review / /security-review + OWASP LLM 2025
   │  产出：iteration-vault/09-review-reports/{self,adversarial,security}.md
   ▼  🚨 R2 检查
PHASE 10  🛡️ 上线前质量检查（五层验收，最多 3 次重试 + 自动回退）
   │  L1 契约 / L2 红线编号 / L3 静态 / L4 运行时 / L5 acceptance
   │  调用：/autodev-verify + test-driven-development + acceptance-testing
   │  产出：iteration-vault/10-verification-report.md + 10-verification-redlines.md
   ▼  🚨 R3 检查
PHASE 11  📢 发布说明（GAN）
   │  调用：finishing-a-development-branch + docx + GAN
   │  产出：iteration-vault/11-release-notes.md + 11-release-gan/
   ▼
PHASE 11.5 🔄 文档代码漂移检测（5 维度 + 三级警报，ERROR 阻塞 12）
   │  调用：CodeGraph MCP（若已索引，验证 endpoint/依赖存在）
   │  产出：iteration-vault/11.5-sync-report.md
   ▼
PHASE 12  🚀 git 发版
   │  调用：integrations/git-workflow + release-please + gh CLI
   │  产出：iteration-vault/12-release.md + Release PR URL
   ▼  🌅 写"递交摘要" + state.yaml mode=morning-review-pending
═══════════════════════════════════════════════════════════════
PHASE 12.5 ☀️ 早晨复盘 ⛳ 关卡 2（PM 必参与）
   │  Step 1 必先 5 分钟扫 RUN-LOG.md（业务名时间轴）
   │  4 步结构化清单 + 4 选项（✅merge / 🔄局部 redo / ❌整体重做 / ⏸推下夜）
   │  产出：iteration-vault/12.5-morning-review.md
   ▼
[完成]  PM 走 12.5 4 选项 → autopilot 回流（Phase 13）+ vault 归档 history/

PHASE 13 🤖 autopilot 回流（仅 autopilot-triggered 迭代）
   │  写 ~/.autopilot/run-history.jsonl + state.json update
   │  产出：iteration-vault/13-autopilot-handoff.md（仅 autopilot 触发时）
```

---

## 8 大模块速查

| 模块 | 内容 | 位置 |
|---|---|---|
| **模块 1** API 整理 | UI 反推 API + 存量审计 + 文档导出 + 跨迭代注册表（REST/GraphQL/RPC 项目类型敏感）| Phase 4 §2 API 整理 子步骤 |
| **模块 2** 夜间模式 v2 | A 重命名 + B 早晨复盘 + C 决策回放 + D 跨夜分批 | `night-mode.md` + Phase 12.5 + `decision-replay.md` + `night-mode-batching.md` |
| **模块 3** Autopilot W1-W4 | 5 connector（roadmap + user-feedback + GitHub + 通用 error-log + Notion）+ 6 安全闸 + 状态机 + Tier 切换 | `autopilot/` 整目录 |
| **模块 4** GAN 引擎 | 1 gen + 1 怀疑 reviewer + 4 维 + 7 redlines + PIVOT | `integrations/gan-engine.md` + `gan-engine/` |
| **模块 5** 文档压缩 | INDEX + RULES（"地图不是摘要"）| Phase 5.9 |
| **模块 6** 方案发散评估 | DIVERGE → EVALUATE → CONVERGE | Phase 2.5 |
| **模块 7** 五层验收重写 | 契约/红线/静态/运行时/acceptance + 红线编号 | Phase 10 |
| **模块 8** 漂移检测 | 5 维度 + INFO/WARN/ERROR | Phase 11.5 |

---

## 关卡机制 v5

### ⛳ PM 关卡 1：PRD 关卡（Phase 2）
- AskUserQuestion 三选项（✅/🔄/❌）+ docx 产出 + Karpathy 4 原则自检 + GAN 引擎
- ✅ 通过后**直接进入 Phase 2.5 brainstorm**（v5 已删除"批数选择"跟进问；分批由 📋 任务拆解按工时自动估算）
- 🏃 快车道下本关卡默认静默自动 ✅（不弹 AskUserQuestion，决策写 `autonomous-decisions.md`）

### 🌙 夜间模式自治执行（Phase 2.5-12，含分批）
- 不打扰 PM，决策按"保守默认决策树"自治
- 写入 `iteration-vault/autonomous-decisions.md` 留痕
- 跑 GAN 的 7 个 phase 各自 vault/<phase>-gan/ 留 trace
- vault checkpoint 在 phase 边界 + ⚠️决策 + R 解决处自动创建

### 🚨 4 条红线 Escalation
- **R1** 重大架构冲突 / vendor lock-in / breaking change
- **R2** 安全 must-fix > 3 项
- **R3** Phase 10 五层验收 3 次重试仍挂
- **R4** 必须删除既有功能 / Phase 11.5 ERROR 级漂移

### ⛳ PM 关卡 2：早晨复盘（Phase 12.5）
- 4 步结构化清单：⚠️决策审计 / 审查 should-fix / 用户验收 / Karpathy 违规
- 4 选项：✅merge / 🔄局部 redo / ❌整体重做 / ⏸推下夜
- 🔄 redo 走决策回放机制（vault checkpoint）

**全部细节**：`night-mode.md` + `gates.md` + `phases/12.5-morning-review.md` + `decision-replay.md` + `night-mode-batching.md`。

---

## 启动时主动反问（Step 0）

> **Step 0 前置 · 经验召回（v5.1 新增 — 闭合跨迭代学习回路）**
> 在复述需求之前**先翻历史再开干**，否则每轮迭代失忆、把上轮已修的坑重新踩一遍：
> 1. Read 项目根 `CLAUDE.md`（若存在）—— 耐久铁律 / 已知坑
> 2. Grep `iteration-vault/*/findings.md` + `iteration-vault/*/autonomous-decisions.md` + 自动记忆（`~/.claude/projects/<项目>/memory/`），匹配本次需求的领域关键词
> 3. 命中旧教训 → 写进下面的"复述需求"（"上次在这块学到 X，本次沿用 / 避免 Y"），并作为强约束带进后续 phase；无命中则一句话说明"未发现相关历史经验"
> 4. 配套收尾：Phase 12.5 / 13 收工时把本轮耐久教训**回写** `CLAUDE.md` + PRD + 自动记忆（读 = 召回，写 = 沉淀，两边都做回路才闭合）

收到用户需求后，**默认 0 提问，自动判完直接开干**（一句话需求→直接出可跑代码是默认体验）。先在一条消息里**输出 3 件判定结果**（不是 3 个问题）：

1. **复述需求** — 1-2 句话结构化复述（避免误解）
2. **自动分级** — 不打扰 PM，自己判（详见 `reference/risk-fast-lane.md`）：
   - **项目类型**：Read 工作目录清单文件（package.json / pyproject.toml / Cargo.toml…）静默判定 Web 全栈 / B 端 SaaS / AI 原生（详见 `reference/project-type-router.md`）。
   - **SIZE**：XS / S / M / L（粗判）。
   - **RISK**：是否命中钱/认证/并发/数据完整性/合规关键词。
   - → 落到 **Lane**：🏃 快车道 / 🚶 标准 / 🐢 重型。写入 `00-intake.md` 顶部。
3. **跑哪一段** — 默认 **完整流程**（按 lane 跑）；若工作目录已有成熟代码且需求是"上线/修复存量"，自动推断为"仅后段"。

**只有这两种情况才反问 PM**（用 AskUserQuestion）：① 项目类型自动检测**不确定**；② 判为 🐢 重型 **或** 命中**高危子类**——确认范围与 lane。

> ⚠️ **关键边界**：普通 `RISK=✅`（如纯前端金额计算 / 本地数据）**不触发反问**——它走 🏃 风险快车道，静默直跑、靠**强制内联 GAN** 兜底。只有满足"高危子类"判据（真实资金流转 / 销毁既有数据 / 改动认证授权边界，定义见 `reference/risk-fast-lane.md`）才反问。**别把任意"碰钱"升级成提问**，否则等于把刚省掉的 upfront 提问又加回来。

其余一律静默默认，PM 想覆盖可随时说"这次我要看 PRD / 改用 B 端 SaaS / 只跑前段"。

**不问"几批跑"** — 系统在 📋 任务拆解 阶段自动估算：< 8h 单批（默认）；≥ 8h 主动提示"估算 Xh，是否分批"让 PM 二选一。

启动时也**主动检查工作目录**（Read package.json / pyproject.toml / Cargo.toml 等），如果发现是 项目，**主动提示用户**："这看起来是 项目，要不要改用 firefly-web-os-orchestrator-skill（业务特化版）？"

---

## 工作目录约定

启动后，在**用户当前工作目录**（非 ~/.claude/）创建 vault：

```
<当前工作目录>/iteration-vault/<YYYY-MM-DD>-<short-title>/
├── 00-intake.md                       # PM 原话 + 大小分级（或 autopilot 注入）
├── 01-clarified-requirement.md
├── 01.5-user-research.md
├── 02-PRD.md + 02-prd-gan/             # ⛳ 关卡 1
├── 02.5-brainstorm-diverge.md + 02.5-brainstorm-converge.md + GAN 目录
├── autonomous-decisions.md            # 持续追加
├── 03-impact-analysis.md + 03-adr.md
├── 04-architecture-and-api.md + 04-architecture-and-api-gan/    # 含 §1 大架构 + §2 API 整理（原 4 + 4.5 合并）
├── 04-api-spec.yaml + 04-api-postman.json + 04-api.md          # API 衍生（机器消费 + QA 测试 + 人类可读）
├── 05-interface-design.md + 05-wireframes/ + 05-gan/           # 含 §1 UX + §2 UI Spec（原 5a + 5b 合并）
├── INDEX.md (<100 行) + RULES.md (<80 行)
├── 06-task-breakdown.md + 06-task-breakdown-gan/
├── 07-code-task-<id>-gan/             # 每个 code task 一个 GAN 目录
├── 08-tech-debt-audit.md
├── 09-review-reports/{self,adversarial,security}.md
├── 10-verification-report.md + 10-verification-redlines.md
├── 11-release-notes.md + 11-release-gan/
├── 11.5-sync-report.md
├── 12-release.md
├── 12.5-morning-review.md             # ⛳ 关卡 2
├── 13-autopilot-handoff.md            # 仅 autopilot 触发时
├── batches/                            # 仅多批跑时
├── checkpoints/                        # 决策回放快照
├── state.yaml                          # 跨 session 状态
└── meta.json
```

**short-title** 从用户原 prompt 提取（如 "加个数据导出功能" → `data-export`）。

**跨迭代持久化**：
- `<project-root>/api-registry.md`（Phase 4 §2 API 整理 跨迭代追加）
- `~/.autopilot/*`（autopilot 运行时配置 + 状态）

---

## 详细 phase 入口

| 业务名 | 内部编号 | 入口文件 | 模式 |
|---|---|---|---|
| 💬 需求澄清（含原 Phase 0 接收）| 1 | `phases/01-clarify.md` | 💬 PM 在线 |
| 👥 用户研究 | 1.5 | `phases/01.5-user-research.md` | 💬 PM 在线 |
| 📝 PRD 撰写 ⛳ 关卡 1 | 2 | `phases/02-prd.md` | ⛳ **PM 关卡 1** + GAN |
| 💡 方案发散 | 2.5 | `phases/02.5-brainstorm.md` | 🌙 夜间 + GAN |
| 📐 影响面分析 + ADR | 3 | `phases/03-impact.md` | 🌙 夜间 |
| 🏗️ 架构与接口设计（含原 4.5 API 整理）| 4 | `phases/04-architecture-and-api.md` | 🌙 夜间 + GAN + 🚨 R1（含 API 子检查）|
| 🎨 界面设计（含原 5a UX + 5b UI Spec）| 5 | `phases/05-interface-design.md` | 🌙 夜间 + GAN |
| 📚 设计压缩成开发地图 | 5.9 | `phases/05.9-compress.md` | 🌙 夜间 |
| 📋 任务拆解 | 6 | `phases/06-tasks.md` | 🌙 夜间 + GAN |
| ⌨️ 代码实施 | 7 | `phases/07-implement.md` | 🌙 + `/loop` + 每 code task GAN + 🚨 R4 |
| 🧹 代码债扫描 | 8 | `phases/08-tech-debt.md` | 🌙 夜间 |
| 🔍 多路审查 | 9 | `phases/09-review.md` | 🌙 夜间（即 Global GAN）+ 🚨 R2 |
| 🛡️ 上线前质量检查 | 10 | `phases/10-verify.md` | 🌙 夜间 + 🚨 R3（3 次重试上限）|
| 📢 发布说明 | 11 | `phases/11-release.md` | 🌙 夜间 + GAN |
| 🔄 文档代码漂移检测 | 11.5 | `phases/11.5-sync.md` | 🌙 夜间（5 维度，ERROR 触 R4）|
| 🚀 git 发版 | 12 | `phases/12-version-git.md` | 🌙 夜间 + 🌅 递交摘要 |
| ☀️ 早晨复盘 ⛳ 关卡 2 | 12.5 | `phases/12.5-morning-review.md` | ⛳ **PM 关卡 2** |
| 🤖 autopilot 回流 | 13 | `phases/13-autopilot-handoff.md` | 🌙（仅 autopilot 触发时）|

每个 phase 文件包含：本阶段目标、输入、调用哪些子 skill、产出文件、autonomous 决策规则、escalation 触发条件、失败回退、GAN 钩子（如有）。

**夜间模式通用机制**：`night-mode.md`
**GAN 引擎通用机制**：`integrations/gan-engine.md` + `gan-engine/`

---

## 与本机其他资源的协作

**调度的 skill**（不替代）：
- obra/superpowers — `/brainstorm`、`verification-before-completion`、`requesting-code-review`、`subagent-driven-development`
- adr-skill — Phase 3 自动调用
- anthropic-skills:review / security-review — Phase 7/9
- release-please / semantic-release — Phase 12
- /autodev-* — 多 phase 调用（ideation / api / ui / iterate / review / verify）

**调度的 MCP**（详见 `reference/tool-inventory.md`）：
- Figma MCP — Phase 5 §1 UX + §2 UI Spec
- Playwright MCP — Phase 10 §L4 运行时
- Sentry MCP — Phase 8 / autopilot connector
- Notion MCP — Phase 1.5 / autopilot connector
- GitHub MCP / gh CLI — Phase 12
- 🆕 CodeGraph MCP（条件式，若项目已 codegraph init）— Phase 3 影响面 / Phase 4 §2 API 调用图 / Phase 11.5 漂移验证

**项目类型特殊路由**（详见 `reference/project-type-router.md`）：
- **Web 全栈**：Phase 5 强制（Figma MCP），Phase 4 §2 API 偏 REST
- **B 端 SaaS**：Phase 4 §2 API 偏 REST + GraphQL，⛳ 早晨复盘真人用户视角强制
- **AI 原生应用**：Phase 7 加 Promptfoo prompt 测评 + Garak/PyRIT 红队；Phase 8 加 Langfuse 监控

---

## Autopilot Slash Commands

PM 与 autopilot 交互的命令（完整规格见 `autopilot/pm-oversight.md`）：

| 命令 | 行为 |
|---|---|
| `/autopilot-status` | 查看 queue + 状态 + 最近 7 天 history |
| `/autopilot-queue` | 打开 queue.md |
| `/autopilot-pause` / `/autopilot-resume` | 暂停 / 恢复 |
| `/autopilot-stop` / `/autopilot-reset` | 紧急停 / 清除 ERROR |
| `/autopilot-tier 1\|2\|3` | 切换 Tier |
| `/autopilot-blacklist <id>` | 拉黑候选 |

自然语言："autopilot pick #N 启动" / "暂停 autopilot" / "切到 Tier 2" 也触发。

---

## 反方观点与限制

- **不是银弹**：完整 17 阶段适合 M/L"正经迭代"。XS/S 小需求走 🏃 快车道（自动砍外围）；真·1 行 PoC 仍建议直接动手不进流水线。
- **依赖本机已装资源**：若 superpowers / adr-skill / Sentry MCP / Playwright MCP 缺失，对应阶段会**降级而非失败**
- **AI 项目分支不完整**：当前 Langfuse MCP 仅 Prompt Management，trace/cost 仍要 Web UI
- **自治模式风险**：4 红线 + 12.5 复盘是兜底，PRD 表达模糊时仍可能偏离 → PM 应在 Phase 1/2 充分澄清
- **GAN 引擎 token 成本**：标准迭代 ~214 调用，完整迭代 ~380 调用。🏃 快车道默认大幅压缩（GAN 仅在命中风险面时内联 round-1，~30-40 调用即可抓出真 bug）；标准 lane 下 N 按 RISK 自适应（无风险 N=2-3）。**注意：省 GAN 要按 RISK 省，不要按 SIZE 省**——碰钱/认证的小任务正是丢了 GAN 最致命的地方。
- **不绑定具体 Git workflow**：Phase 12 默认 release-please，PM 团队若用别的（GitFlow / trunk-based）可在 Phase 0 声明

---

## 进一步阅读

- `reference/risk-fast-lane.md` — 🆕 风险分级快车道（按 SIZE 定深度 / 按 RISK 保 GAN 的路由矩阵）
- `night-mode.md` — 夜间模式完整机制（4 件套）
- `gates.md` — 关卡机制 + 4 红线
- `gan-engine/role-router.md` — GAN 任务类型 → 4 维 + 视角注入
- `gan-engine/quality-redlines.md` — 7 Quality Redlines
- `principles/karpathy-llm-coding.md` — Karpathy 4 原则
- `reference/project-type-router.md` — 3 种项目类型分支
- `reference/escalation-redlines.md` — 4 红线判定 + 处理
- `reference/tool-inventory.md` — 本 skill 用到的所有 skill / MCP 清单
- `autopilot/README.md` — Autopilot 完整说明

---

## 维护备忘

- 每装/淘汰一个子 skill 时同步更新 `reference/tool-inventory.md`
- 每次跑完一次完整迭代，从 `iteration-vault/` 抽取通用经验沉淀到 phase 文件
- 每次发现关卡机制不顺手，更新 `gates.md`
- 项目类型新增（如 移动 / 嵌入式）→ 更新 `reference/project-type-router.md`
- v4 8 大模块各自有独立 .md，按模块边界维护
