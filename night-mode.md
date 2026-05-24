# 夜间模式 (Night Mode) — PRD 通过后自治执行机制

> **别名**：autonomous mode（技术内部术语保留，决策日志文件名 `autonomous-decisions.md` 不变以保审计连续性）
>
> 本 skill v4 的核心设计：**PRD 是 PM 关卡 1**，**Phase 12.5 早晨复盘是 PM 关卡 2**。PRD 通过后，Phase 2.5-12 由 skill 自治跑完。PM 早上回来走 Phase 12.5 4 步结构化复盘 + 4 选项决策（merge / 局部 redo / 整体重做 / 推下夜）。
>
> 这套机制是为**过夜跑场景**设计的——PM 提需求 → 确认 PRD → 睡觉 → 早上走 12.5 复盘 → 选择 merge / 调整。
>
> **v4 在 v2 之上的 4 件增量**：
> - A 重命名（PM 面向语言）
> - B Phase 12.5 早晨复盘（**第 2 个 PM 关卡**）
> - C 决策回放（vault checkpoint，PM 可"replay from decision #N"）
> - D 跨夜分批（单批 / 2 批 / 3 批，估算 > 4h 推荐分批）
> - E（学 autodev）：sleep mode 状态持久化 `state.yaml` + `.claude/handoff.md`

---

## 四个核心机制（v4 含跨夜分批 + 早晨复盘 + 决策回放）

### 机制 1：两个 PM 关卡 = PRD + 早晨复盘

之前 v2 设计 1 个 ⛳ 关卡（PRD），v4 加 1 个早晨复盘关卡：

| 阶段 | v2 行为 | v4 行为 |
|---|---|---|
| Phase 2 PRD | ⛳ PM 三选项 | ⛳ PM 三选项 + 加批数跟进问 |
| Phase 2.5 brainstorm 🆕 | — | 🤖 autonomous + GAN |
| Phase 3 影响面 | 🤖 autonomous | 🤖 autonomous |
| Phase 4 架构 | 🤖 autonomous + R1 | 🤖 autonomous + GAN + R1 |
| Phase 4.5 API 整理 🆕 | — | 🤖 autonomous + GAN + R1 子检查 |
| Phase 5a UX | 🤖 autonomous + GAN | 🤖 autonomous + GAN（autodev-ui task-first）|
| Phase 5b UI Spec | 🤖 autonomous | 🤖 autonomous（三态强制）|
| Phase 5.9 文档压缩 🆕 | — | 🤖 autonomous（INDEX + RULES）|
| Phase 6 任务分解 | 🤖 autonomous | 🤖 autonomous + GAN |
| Phase 7 实施 | /loop autonomous | /loop autonomous + 每 code task GAN |
| Phase 8 代码债 | 🤖 autonomous | 🤖 autonomous |
| Phase 9 三路审查 | 🤖 autonomous | 🤖 autonomous（即 Global GAN，不嵌套）|
| Phase 10 验收 | 🤖 autonomous + 3 次重试 | 🤖 autonomous + 五层 + 失败自动回退 |
| Phase 10.5 真人验收 | 🤖 autonomous | 🤖 autonomous |
| Phase 11 release notes | 🤖 autonomous | 🤖 autonomous + GAN |
| Phase 11.5 漂移检测 🆕 | — | 🤖 autonomous（ERROR 级阻塞 12）|
| Phase 12 git release | 🤖 autonomous | 🤖 autonomous |
| **Phase 12.5 早晨复盘 🆕** | — | **⛳ PM 关卡 2**：4 步清单 + 4 选项 |

**关键设计**：PRD 通过 = "全权委托夜间执行"，但 PM 在 Phase 12.5 **有最终决策权**（merge / 调整 / 重做 / 推下夜）。

### 机制 2：跨夜分批跑（v4 新增 D）

Phase 6 任务分解完成时估算总跑时：

| 估算总跑时 | 分批建议 | PM 决定 |
|---|---|---|
| < 4h | 单批跑完 | 默认 |
| 4-8h | 提示"可以分 2 批" | PM 可选 |
| 8-15h | **强烈建议**分 2-3 批 | PM 可选但建议 |
| > 15h | **autonomous-decisions.md 标 ⚠️** + 必须分批 | PM 拍 |

**推荐切分点**（基于 Karpathy + 流程逻辑）：
- Batch 1: Phase 2.5 → 6（基础设计批，~3-5h）
- Batch 2: Phase 7 → 8（实施批，~3-10h）
- Batch 3: Phase 9 → 12（审查发布批，~2-4h）

**批次启动**：
- Batch 1 自动从 PRD 通过启动
- Batch 2+ 必须 PM 说 "/next-batch" 或 "继续夜间模式" 启动

**跨批级联失败**（Batch 2 发现 Batch 1 错）：
- 不触发 R 红线，写 `iteration-vault/CROSS-BATCH-ISSUE.md`
- 尝试 inline 修；不能修 → 升级类 R3

**未解决 R 红线阻塞下批**：Batch 1 触发 R1 没解决，Batch 2 不启动，etc.

详见 `night-mode-batching.md`。

### 机制 3：4 红线 escalation（子零打扰）

只有以下 4 种情况会**中断 autonomous 跑、叫醒 PM**：

| 红线编号 | 触发条件 | 处理 |
|---|---|---|
| 🚨 R1 重大架构冲突 | 选型导致 vendor lock-in / 重构现有核心模块 / 引入与 项目协议（如有）冲突的库 / **Phase 4.5 存量审计发现 ≥ 5 个接口违反 项目业务协议（如有）** / **新 API 设计无法兼容 项目消息信封（如有）** / **必须 breaking change 既有公开接口** | 暂停 → 写决策报告 → 等 PM 显式决策 |
| 🚨 R2 安全 must-fix > 3 项 | Phase 9 安全审查发现 ≥ 3 项阻塞发布的 must-fix 问题 | 暂停 → 列安全报告 → 等 PM 决定是否降级范围 |
| 🚨 R3 验收 3 次重试仍挂 | Phase 10 五层验收失败，回 Phase 7 修，重试 3 次仍未通过 | 暂停 → 列失败模式分析 → 等 PM 决策（缩范围 / 改方案 / 推迟）|
| 🚨 R4 删除既有功能 | 实施过程中发现必须删除某既有功能才能完成本次需求 | 暂停 → 列删除项 + 影响面 → 等 PM 明确同意 |

**所有其他情况**：autonomous 决策，记入 `iteration-vault/autonomous-decisions.md`，继续跑。

### 机制 4：保守默认决策树 + 决策回放（v4 新增回放 C）

**保守默认决策树**：

| 选择类型 | 保守默认 | Karpathy 原则依据 |
|---|---|---|
| 可逆 vs 不可逆 | **选可逆** | Simplicity First + Surgical Changes |
| 新方案 vs 成熟方案 | **选成熟** | 减少未知风险 |
| 复杂 vs 简单 | **选简单** | Simplicity First |
| 通用化 vs 专用 | **选专用** | YAGNI / Simplicity First |
| 加抽象 vs 直接 inline | **选 inline** | Simplicity First |
| 删除代码 vs 保留并标 deprecated | **选保留并 deprecate** | Surgical Changes |
| 升级依赖大版本 vs 维持当前 | **选维持** | 除非有安全/必需理由 |
| 数据库破坏性 vs 兼容性变更 | **选兼容性** | 可回滚优先 |
| 关闭 feature flag 启动 vs 灰度 | **选灰度** | 风险分摊 |
| **API 命名冲突（新 vs 存量）** | **选存量** | Simplicity First + Surgical Changes |
| **存量 API 命名不一致（< 5 项）** | **flag 不阻塞**（进 backlog）| Surgical Changes |
| **存量 API 命名不一致（≥ 5 项）** | **触发 R1**（系统性问题）| 见 R1 |

每个保守决策都写入 `autonomous-decisions.md`，PM 早上回来能逐条审查 + 回滚。

**决策回放机制（v4 新增 C）**：

vault 在以下点自动 checkpoint（纯文件 copy 到 `iteration-vault/checkpoints/`）：
- 每个 phase 边界（10 个/迭代）
- 每个 ⚠️ 决策（3-8 个/迭代）
- 每个 R 红线解决后（0-3 个/迭代）

PM 在 Phase 12.5 早晨复盘可触发：
- "replay from decision #7 with new direction Y"
- → skill 恢复 vault 到 checkpoint #7 状态
- → 注入 PM 新方向到 autonomous-decisions.md 第 7 条后
- → 从该 phase 重跑下游

详见 `decision-replay.md`。

---

## 决策日志规范

### 文件：`iteration-vault/autonomous-decisions.md`

每个 autonomous 决策必须按下面格式记录：

```markdown
## [时间戳] [Phase X] 决策标题

**触发情境**：当前在做什么，遇到了什么需要决策的点

**候选**：
- 方案 A：[描述]
- 方案 B：[描述]
- 方案 C：[描述]

**选择**：方案 [X]

**依据**：
- 保守默认决策树第 [N] 条 ([依据规则名])
- Karpathy 原则：[Simplicity First / Surgical Changes / ...]
- 项目业务上下文：[业务规则 / 协议 / 技术栈约束]（如适用）
- 现有 PRD 第 [N] 节：[如何引用]

**影响范围**：本次决策影响 [N] 个文件 / [M] 个模块

**可逆性**：✅ 可逆（如何撤回） / ⚠️ 部分可逆（哪些不可逆）/ ❌ 不可逆

**checkpoint 否**：✅ 触发 vault checkpoint / ❌ 不 checkpoint

**PM 早上需要 review 的点**：[1-2 行总结，便于 PM 快速扫 ⚠️ 标记]
```

### 决策密度预期

一次完整的 Phase 2.5-11.5 跑大约会产生 **15-35 条 autonomous 决策**（v4 多了 Phase 2.5 / 4.5 / 5.9 / 11.5）。PM 早上花 10-15 分钟扫一遍决策日志就能知道发生了什么。

如果某次跑产生 > 60 条决策 → 可能是 PRD 不够清晰，应该升级到红线 R1 让 PM 看下是否需要重做 PRD。

---

## Escalation 触发后的标准流程

当任一红线触发时：

### Step 1: 暂停所有后续 phase

- 写 `iteration-vault/ESCALATION-R[X].md`，标 🚨 状态
- 当前 phase 停在原地，已完成的部分保留
- 如果在分批模式：下一 batch 不启动

### Step 2: 生成决策报告

报告结构：

```markdown
# 🚨 ESCALATION: [红线编号] [简短标题]

**触发时间**：[timestamp]
**当前 phase**：[N]
**当前 batch**：[batch_id]（如分批）
**已完成的**：Phase 1-[M] 已成功

## 问题描述
[2-3 段：发生了什么，为什么不能 autonomous 决策]

## 候选方案
- 方案 A：[描述 + 工时 + 风险]
- 方案 B：...
- 方案 C：...

## skill 的倾向（如有）
[基于保守默认，倾向 A，但需要 PM 确认]

## 影响
- 如果选 A：[影响]
- 如果选 B：...

## PM 需要做什么
1. Read iteration-vault/ESCALATION-R[X].md 完整内容
2. Read iteration-vault/autonomous-decisions.md 看前面决策
3. 给出方向 → skill 续跑
```

### Step 3: 通知 PM 并等待

如果 skill 在 `/loop` 模式下：
- 写 escalation 报告
- 把当前 loop 状态标 paused
- 调用 PushNotification（如有）或在下次 PM 回话时主动汇报

如果 skill 在交互模式：
- 直接对 PM 说"🚨 触发 [红线] escalation，详情见 iteration-vault/ESCALATION-R[X].md"
- 等 PM 输入决策

### Step 4: PM 决策后续跑

PM 可以：
- 选某个候选方案 → skill 续跑
- 临时修改 PRD → 回 Phase 2 重新关卡（少见）
- 终止本次迭代 → 全部保留产出在 vault 历史
- 触发决策回放：从某个 checkpoint 重跑

---

## sleep mode 状态持久化（学 autodev，v4 新增）

夜间模式下，autonomous-decisions.md + meta.json 是 vault 内的，但**跨 session 状态**需要额外存：

### `iteration-vault/state.yaml`（autodev sleep-mode 学过来）

```yaml
mode: night-mode | morning-review-pending | escalated | paused
current_phase: 7
current_batch: 2  # 如分批
batch_plan_path: iteration-vault/batches/batch-plan.md
last_phase_completed: 6
last_phase_completed_at: 2026-05-23T22:30:00+08:00
escalation:
  active: false
  type: null  # R1 | R2 | R3 | R4 | null
  file: null  # iteration-vault/ESCALATION-R1.md
checkpoints_count: 7
total_decisions: 12
gan_invocations: 5
```

### `.claude/handoff.md`（context 耗尽时用）

如果跑到 context 上限：
- 写 `.claude/handoff.md` 含：当前状态 + 下次启动需读的文件清单 + 待办
- PM 下次回话时 skill 读此文件 + state.yaml 恢复

---

## 夜间跑标准流程（v4 含分批 + 早晨复盘）

### 启动方式

```
PM："PRD 通过，开始夜间模式。估算 8h，我选 2 批。"

skill：
- 调用 /loop 进入夜间跑模式
- 写 state.yaml: mode=night-mode, current_batch=1
- 写 iteration-vault/batches/batch-plan.md（如 2-3 批）
- Batch 1 自动启动 Phase 2.5
- Batch 1 完成后等 PM "/next-batch" 启动 Batch 2
```

### 内部机制（PM 不感知）— v4 含 Phase 2.5 + 4.5 + 5.9 + 11.5

```
（PRD 通过后）
PHASE 2.5 brainstorm（autonomous，GAN）
   ↓
PHASE 3 影响面（autonomous，~30 min）
   ↓
PHASE 4 架构（autonomous + 保守默认，GAN）
   ↓ (R1 检查)
PHASE 4.5 API 整理（autonomous，GAN Step 1 + 审计 + 注册表）
   ↓ (R1 子检查：项目业务协议（如有）)
PHASE 5a UX（autonomous，GAN，task-first 信息架构）
   ↓
PHASE 5b UI Spec（autonomous）
   ↓
PHASE 5.9 文档压缩（autonomous，INDEX + RULES）
   ↓
PHASE 6 任务分解（autonomous，GAN）+ 估算批数
   ↓
[此处 Batch 1 边界（如分批跑）]
   ↓
PHASE 7 实施（/loop /autodev-iterate 每个任务，每 task GAN，可能 2-6 小时）
   ↓ (R4 检查：是否要删除既有功能)
PHASE 8 代码债（autonomous）
   ↓
[此处 Batch 2 边界（如分 3 批）]
   ↓
PHASE 9 审查（autonomous，即 Global GAN）
   ↓ (R2 检查：安全 must-fix)
PHASE 10 五层验收（autonomous + 最多 3 次重试 + 失败自动回退上一 Step）
   ↓ (R3 检查)
PHASE 10.5 真人用户验收
   ↓
PHASE 11 发布说明（autonomous，GAN）
   ↓
PHASE 11.5 漂移检测（autonomous，5 维度，ERROR 级阻塞 12）
   ↓
PHASE 12 git/GitHub release（autonomous）
   ↓
**PHASE 12.5 早晨复盘（⛳ PM 关卡 2）**
   ↓
[完成]  归档 + autopilot 回流
```

### Phase 12 末"递交摘要"（v4 不再含决策，决策动作迁到 12.5）

Phase 12 完成时，skill 写一段简短摘要到 trace，触发 12.5：

```
🌅 夜间模式跑完。

✅ 总体状态：成功 / ⚠️ 部分完成 / 🚨 触发 [R?]

📊 数字摘要：
- 完成任务：[N]/[M]
- 自治决策：[K] 条（详见 autonomous-decisions.md）
- 代码改动：[X] 文件 / +[Y] -[Z] 行
- GAN 跑了 [N] 个任务，平均 [X.X] 轮 PASS
- 测试新增：[N] 条单测 / [M] 条 e2e / 覆盖率 [P]%
- 安全审查：[已修 / must-fix 数]
- 验收：五层 ✅ / PRD AC [N/M] 通过
- 用户验收：[X.X]/5（[N] 真实用户）

→ 进入 Phase 12.5 早晨复盘（PM 4 步清单 + 4 选项）
→ 见 phases/12.5-morning-review.md
```

PM 进入 12.5 后才做 merge / redo 决策。

---

## 失败模式 & 回退

### 失败模式 1：autonomous 跑挂在某个 phase

- 标记 `iteration-vault/state.yaml` 的 mode = "failed-pending-review"
- 暂停 `/loop`
- 等 PM 回话时主动报告

### 失败模式 2：长时间无进展（如某任务 spawn 的 agent 卡住）

- `/loop` 内置超时机制（建议每 30min 一个 phase）
- 超时 → 标记并升级到 R3（验收挂掉同等级）

### 失败模式 3：连续多次重试都过不了

- 最多 3 次重试（R3 阈值）
- 超出 → 触发 R3 escalation

### 失败模式 4：磁盘 / 网络 / API 异常

- 不是 skill 自身错，但 skill 应该捕获并 graceful pause
- 标记 `iteration-vault/INFRASTRUCTURE_ERROR.md`，等 PM 排查

### 失败模式 5：context 耗尽（v4 新增）

- 写 `.claude/handoff.md` 含恢复指令
- state.yaml 标 mode = "context-exhausted"
- PM 下次回话时按 handoff.md 恢复

### 失败模式 6：跨批级联失败（v4 新增）

- Batch 2 跑到一半发现 Batch 1 错
- 写 `iteration-vault/CROSS-BATCH-ISSUE.md`
- 尝试 inline 修
- 不能修 → 升级类 R3 处理

---

## 与 Karpathy 4 原则的契合

autonomous 模式 = Karpathy "Goal-Driven Execution" 的极致体现：

- **给可验证标准**（PRD + AC）→ skill 自循环
- **不给指令清单** → skill 自己决策怎么走
- **循环迭代直到达成** → /loop + verification-loop 闭环
- **失败 3 次后停下**（不是无限循环）→ 防止失控

但同时：autonomous 模式下，**Think Before Coding 必须更严格**——每个决策都要显性化记到 autonomous-decisions.md，否则 PM 早上无法审计。

---

## 自检问题（每次夜间跑开始前）

1. PRD 是否已经 PM ✅ 通过？（如否，不许跑）
2. PRD 第 5.5 假设检查表是否填了 ≥ 3 条假设？（如否，先回 Phase 1）
3. 是否有未解决的 escalation？（如有，先处理）
4. iteration-vault/ 是否复位？（防止串味）
5. `/loop` 模式还是交互模式？（影响 escalation 处理方式）
6. **如分批**：batch_plan.md 是否就绪？state.yaml current_batch 正确？
7. **GAN 引擎**：integrations/gan-engine.md 是否就绪？

任一答否 → 不许启动夜间模式。

---

## 维护备忘

- autonomous-mode.md 仍保留（向后兼容），但所有 PM 面向文档应引用 night-mode.md
- 每次跑过完整迭代，沉淀经验到本文件
- Phase 12.5 / 决策回放 / 跨夜分批的实操细节见各自 .md 文件
