# Universal 架构地图（5 分钟看懂）

## 目录结构

```
SKILL.md                  ← 主入口（先读这个）
ARCHITECTURE.md           ← 本文件（5 分钟看懂全貌）
phases/                   ← 每个步骤一个文件
  MANIFEST.md             ← 17 步骤的输入输出一张表（必读）
  01-clarify.md           ← 💬 需求澄清
  01.5-user-research.md   ← 👥 用户研究
  02-prd.md               ← 📝 PRD 撰写 ⛳ 关卡 1
  02.5-brainstorm.md      ← 💡 方案发散
  03-impact.md            ← 📐 影响面分析
  04-architecture-and-api.md  ← 🏗️ 架构与接口设计（含原 4.5）
  05-interface-design.md  ← 🎨 界面设计（含原 5a + 5b）
  05.9-compress.md        ← 📚 设计压缩成开发地图
  06-tasks.md             ← 📋 任务拆解
  07-implement.md         ← ⌨️ 代码实施
  08-tech-debt.md         ← 🧹 代码债扫描
  09-review.md            ← 🔍 多路审查
  10-verify.md            ← 🛡️ 上线前质量检查
  11-release.md           ← 📢 发布说明
  11.5-sync.md            ← 🔄 文档代码漂移检测
  12-version-git.md       ← 🚀 git 发版
  12.5-morning-review.md  ← ☀️ 早晨复盘 ⛳ 关卡 2
  13-autopilot-handoff.md ← 🤖 autopilot 回流（仅 autopilot 触发时）
gan-engine/               ← AI 互审引擎 5 文件
  quality-redlines.md     ← 质量红线 SSOT（改红线只改这里）
  generator.md            ← Generator prompt 模板
  reviewer-skeptical.md   ← Reviewer prompt 模板
  pivot-handler.md        ← PIVOT 重写机制
  role-router.md          ← 任务类型 → 4 维度路由
autopilot/                ← AI 主动捡需求模式（独立入口，非默认）
  state-machine.md        ← 12 状态机
  safety-brakes.md
  pm-oversight.md
  priority-algorithm.md
  config.yaml
  connectors/             ← 5 个数据源（roadmap/feedback/github/sentry/notion）
  templates/              ← autopilot 专属模板
night-mode.md             ← 夜间自动跑机制
night-mode-batching.md    ← 跨夜分批
decision-replay.md        ← 决策回放 checkpoint
gates.md                  ← PM 关卡 + 4 红线 escalation
integrations/             ← 8 个外部能力内化
  gan-engine.md
  api-architect.md
  database-architect.md
  git-workflow.md
  owasp-llm-2025.md
  release-please.md
  tech-debt-9d.md
  test-planner.md
principles/
  karpathy-llm-coding.md  ← Karpathy 4 LLM 编码原则
reference/
  slash-commands.md       ← 所有 slash 命令一表（必读）
  project-type-router.md  ← 3 种项目类型分支
  tool-inventory.md       ← 所有 skill / MCP 清单
  escalation-redlines.md  ← 4 红线判定 + 处理
  workflow-loop.md        ← 工作流循环说明
templates/                ← 15 个产出模板
  run-log.md              ← 业务名执行轨迹模板（PM 早上看这个）
  debug-trace.md          ← 标准化错误追踪
  prd.md / api-spec.md / architecture.md / ...
```

## 8 大模块速查

| 模块 | 主文件 |
|---|---|
| 1. API 整理 | phases/04-architecture-and-api.md §2 |
| 2. 夜间模式 | night-mode.md + night-mode-batching.md + decision-replay.md |
| 3. Autopilot | autopilot/ 整目录（独立模式，不是默认）|
| 4. AI 互审（GAN） | gan-engine/ 整目录 |
| 5. 文档压缩 | phases/05.9-compress.md |
| 6. 方案发散 | phases/02.5-brainstorm.md |
| 7. 五层验收 | phases/10-verify.md |
| 8. 漂移检测 | phases/11.5-sync.md |

## 5 个最常改的文件

| 想改什么 | 改哪个文件 |
|---|---|
| 一条质量红线 | `gan-engine/quality-redlines.md`（唯一权威）|
| AI 互审阈值 / 4 维度 | `gan-engine/role-router.md` |
| 某个步骤的内部流程 | `phases/<编号>-<名字>.md` |
| 启动反问 / 默认路径 | `SKILL.md` §Step 0 |
| 红线编号格式 | `templates/verification-redlines-numbered.md` |

## 新手 5 分钟上手顺序

1. `SKILL.md` 跳读（374 行，看完知道 17 阶段干啥）
2. 本文件（你正在读）
3. `phases/MANIFEST.md`（一表看完所有阶段输入输出）
4. `gan-engine/quality-redlines.md`（理解防偷懒规则 SSOT）
5. `night-mode.md`（理解过夜跑机制）

## debug 时只需打开这 5 个文件

- `SKILL.md`（看主流程定位在哪个 phase）
- `phases/<当前 phase>.md`（看具体步骤）
- `iteration-vault/<run-id>/RUN-LOG.md`（看 PM 视角执行轨迹）
- `iteration-vault/<run-id>/DEBUG-TRACE.md`（看错误链 E001/E002）
- `iteration-vault/<run-id>/autonomous-decisions.md`（看 AI 自治决策）

## 业务名 ↔ 内部编号约定

任何**对 PM 暴露的文档**（RUN-LOG / DEBUG-TRACE / autonomous-decisions / 早晨复盘 / 对 PM 摘要）
都**必须用业务名 + emoji**，禁止用 "Phase X" 这种内部编号。

**内部技术文档**（phases/*.md / SKILL.md / gan-engine/*.md 等）可继续用编号。

详见 `phases/MANIFEST.md` §业务名 ↔ 内部编号映射。

## v4 8 大模块完成度

- ✅ API 整理 / 夜间模式 / GAN 引擎 / 文档压缩 / 方案发散 / 五层验收 / 漂移检测
- ✅ Autopilot 整目录（含 5 connector + 12 状态机 + 6 安全闸）

## 已知简化（v4 → 当前优化版）

- ❌ 删除真人用户验收（原 Phase 10.5），由 ⛳ 关卡 2 早晨复盘覆盖
- ✅ 合并 0+1 → 💬 需求澄清
- ✅ 合并 4+4.5 → 🏗️ 架构与接口设计
- ✅ 合并 5a+5b → 🎨 界面设计
- ✅ 质量红线 SSOT 化（只在 gan-engine/quality-redlines.md 维护）
- ✅ 删除"批数选择"PM 关卡（系统自动估算 ≥8h 才提示分批）
- ✅ 只保留 1 条标准开发路径（不再有 fast-track / standard / manual 三选项）
