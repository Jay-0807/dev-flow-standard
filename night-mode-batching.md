# 跨夜分批跑（D 节）

> **作用**：估算 > 4h 的迭代支持分多个夜晚跑（2 批 / 3 批），PM 可在批次间介入。
> **v4 新增**：解决"PRD 通过后 8h+ 大需求一夜跑不完"的问题。

---

## 何时建议分批

**自动检测**：在 Phase 6 任务分解完成时，skill 估算总跑时（含所有 phase + GAN 调用）：

| 估算总跑时 | 分批建议 | PM 决定 |
|---|---|---|
| < 4h | 单批跑完 | 默认单批 |
| 4-8h | 提示"可以分 2 批" | PM 可选 |
| 8-15h | **强烈建议**分 2-3 批 | PM 可选但建议分 |
| > 15h | **autonomous-decisions.md 标 ⚠️** + 建议分批 + 进入"sprint 拆分讨论" | 不许单批 |

估算依据：
- Phase 6 中任务数 × 平均任务时长（30-60 min/任务）
- 设计型 phase × ~6 GAN 调用 × ~2-3min/调用
- 固定开销（Phase 8-12.5 ≈ 2h）

---

## 推荐切分点（基于 Karpathy + 流程逻辑）

```
Phase 2 PRD ⛳
   │
   ▼
┌──────────────────────────────────────┐
│  Batch 1（基础设计批，~3-5h）         │
│  Phase 2.5 brainstorm                 │
│  Phase 3 影响面                       │
│  Phase 4 架构                        │
│  Phase 4.5 API 整理                  │
│  Phase 5a UX 设计                    │
│  Phase 5b UI Spec                    │
│  Phase 5.9 文档压缩                  │
│  Phase 6 任务分解                    │
└──────────────────────────────────────┘
   │
   ▼  ⏸ batch 边界（可选 PM 介入点）
┌──────────────────────────────────────┐
│  Batch 2（实施批，~3-10h）            │
│  Phase 7 并行实施                    │
│  Phase 8 代码债                      │
└──────────────────────────────────────┘
   │
   ▼  ⏸ batch 边界（可选 PM 介入点）
┌──────────────────────────────────────┐
│  Batch 3（审查发布批，~2-4h）         │
│  Phase 9 三路审查                    │
│  Phase 10 五层验收                   │
│  Phase 10.5 真人用户验收             │
│  Phase 11 发布说明                   │
│  Phase 11.5 漂移检测                 │
│  Phase 12 git release                │
└──────────────────────────────────────┘
   │
   ▼
Phase 12.5 早晨复盘 ⛳ 关卡 2
```

**为什么选这 3 个切分点**：
- 切 Phase 6 后：基础设计已稳定，PM 可以中途看 PRD→架构→UI 是否对齐
- 切 Phase 8 后：实施完成，技术债已扫，可以中途看代码质量
- Phase 9-12.5 留在最后批：审查 + 验收 + 发布的逻辑紧密耦合，不宜切

---

## PM 选择批数

PRD 通过后跟进一个问题：

```
question: "夜间模式怎么跑？估算总跑时 {{estimated_hours}}h"
header: "夜间模式批次选择"
options:
  - label: "🌙 单批跑（一晚跑完，默认）"
    description: "适合：连续时段，PM 早上一次性复盘"
  - label: "🌙🌙 2 批跑（Batch1 设计 + Batch2 实施发布）"
    description: "推荐：跑时 ≥ 6h；中间可选给 PM 看设计成果"
  - label: "🌙🌙🌙 3 批跑（设计 + 实施 + 审发）"
    description: "适合：大改动 ≥ 12h；最稳"
```

---

## batch 间状态保留

**核心约束**：iteration-vault/ 是**单一目录**，不在 batch 间分裂。所有 batch 共写同一份 vault，只是分阶段写。

**新增文件**：`iteration-vault/batches/batch-plan.md`（多批跑时才创建）

模板见 `templates/batch-plan.md`。

---

## 多批跑下的红线 escalation

**核心规则**：4 红线（R1/R2/R3/R4）**任何 batch 触发都立即停**，但不跨 batch 等候。

```
Batch 1 跑 → 触发 R1 → batch 1 暂停，写 ESCALATION-R1.md
   ↓
PM 回话解决 R1
   ↓
PM 可选：
  A. "继续 batch 1 剩余" → skill 续跑 batch 1
  B. "重做 batch 1 从 X" → 走决策回放
  C. "整体 redo" → 回 Phase 2
```

**绝对规则**：未解决的 R 红线**阻塞下批启动**。autonomous-decisions.md 标 [BLOCKED-ON-ESCALATION]。

---

## 批次启动方式

| 触发 | 行为 |
|---|---|
| 自动续跑 | 仅 batch 1 自动从 PRD 通过启动 |
| 手动续跑 | batch 2+ 必须 PM 说 "/next-batch" / "继续夜间模式" / "下一批" |
| 状态查询 | PM 说 "/status" → skill 输出 batch-plan.md 当前状态 |
| 跳过某批 | 不允许（每批都必须跑完才能进下一批）|
| 中断当前批 | PM 说 "/stop" → 当前 phase 跑完即停，进入 12.5 复盘 |

---

## ASCII 跨夜流程图

```
夜 1 (PM 睡觉)                  夜 2 (PM 上班 + 睡觉)               夜 3 (PM 上班 + 睡觉)
   │                                  │                                    │
   ▼                                  ▼                                    ▼
[Phase 2 PRD ⛳]                  [选项：继续/调整]                    [选项：merge/redo]
   ✅ 通过                           PM 看 batch 1 摘要                  PM 走 Phase 12.5 复盘
   │                                ✅ /next-batch                       ✅ merge
   ▼                                  │                                  ✅ Release PR merged
[Batch 1 启动 2.5-6]                  ▼                                    │
   ├─ Phase 2.5 brainstorm        [Batch 2 启动 7-8]                    [vault 归档到 history/]
   ├─ Phase 3 影响面                  ├─ Phase 7 实施 /loop
   ├─ Phase 4 架构                    └─ Phase 8 代码债
   ├─ Phase 4.5 API                   │
   ├─ Phase 5a UX                     ▼
   ├─ Phase 5b UI                  [Batch 2 完成]
   ├─ Phase 5.9 压缩                  │
   └─ Phase 6 任务                    写 batch-2-summary.md
   │                                  ⏸ skill 暂停
   ▼                                  │
[Batch 1 完成]                     [等到 PM 上班回话]                  完成 ✅
   写 batch-1-summary.md                                                  │
   ⏸ skill 暂停                                                           ▼
   │                                                                  [Batch 3 启动 9-12]
[等到 PM 上班回话]                                                       ├─ Phase 9 审查
                                                                          ├─ Phase 10 五层
                                                                          ├─ Phase 10.5 真人
                                                                          ├─ Phase 11 发布
                                                                          ├─ Phase 11.5 漂移
                                                                          └─ Phase 12 git
                                                                            │
                                                                            ▼
                                                                       [Batch 3 完成]
                                                                          自动进 Phase 12.5
```

---

## 在 Phase 12.5 之外的 PM 介入

Batch 1 → Batch 2 之间 PM 可以做：

- **更新 PRD**：不推荐，但允许。需 PM 显式说"我要改 PRD"，skill 回 Phase 2 重新走，重新生成 batch-plan。**警告**：所有 batch 1 产出会被标"may be invalid"（不删除，但下游可能需重跑）
- **追加 backlog**：写 `iteration-vault/backlog.md`，batch 2 不读，Phase 12.5 时处理
- **直接发起 12.5 复盘**：PM 说"我现在就要复盘 batch 1" → skill 进 Phase 12.5 但仅复盘 batch 1 范围

---

## batch-N-summary.md（每批结束自动写）

模板：

```markdown
# 夜间模式 Batch <N> 完成总结

**起止**: <start-time> → <end-time>（耗时 X.Yh）
**Phase 范围**: <e.g., 2.5-6>
**总决策数**: K（⚠️ 标记 N 条）
**红线触发**: 0 次 / 1 次（R<X> 已解决）
**GAN 调用**: [N] 任务，平均 [X.X] 轮 PASS，[Y] 次 PIVOT
**INFRASTRUCTURE_ERROR**: 0 次

## 完成产出
- iteration-vault/02.5-brainstorm-converge.md ✅
- iteration-vault/03-impact-analysis.md ✅
- iteration-vault/04-architecture.md ✅
- iteration-vault/04.5-api-design.md ✅
- iteration-vault/05a-ux-design.md ✅
- iteration-vault/05b-ui-spec.md ✅
- iteration-vault/INDEX.md ✅
- iteration-vault/RULES.md ✅
- iteration-vault/06-task-breakdown.md ✅

## ⚠️ PM 可关注的（不阻塞）
1. 决策 #7 选了 swr 不是 react-query — 看 autonomous-decisions.md
2. 决策 #12 引入了新表 user_preferences — 看 04-architecture.md 第 4 节
3. ...

## 下批衔接备忘
- Batch 2 将做：Phase 7 并行实施 + Phase 8 代码债
- 估算时长：6-8h
- 关键依赖：DB migration 必须在 batch 2 开始时确认 schema 仍 OK
- 触发命令：PM 说 "/next-batch" 启动

## 当前 vault 大小
- 文件数：23
- checkpoint 数：6（合计 ~340 KB）
- GAN 目录数：5（合计 ~1.1 MB）
- 总大小：~1.5 MB
```

---

## 失败处理

### 跨批级联失败

```
Batch 2 跑到一半发现 batch 1 的 Phase 5b UI Spec 写错了
   ↓
处理：
1. batch 2 自动暂停（不触发 R 红线，因为不是严重失败）
2. 写 iteration-vault/CROSS-BATCH-ISSUE.md：发现什么 + 影响哪些下游
3. autonomous-decisions.md 标新决策 [CROSS-BATCH-CASCADE]
4. autonomous 决策：是否能"局部修补 5b 然后继续 batch 2"？
   - YES（小修补，能 inline 修）→ 修了继续，记决策
   - NO（5b 错得严重，影响 7 的核心）→ 视同 R3 升级到 PM
```

### PM 离线超长导致 batch 间停滞

- batch 1 完后 PM 3 天不回 → `state.yaml: batch_2_awaiting_kick_since: <date>`
- PM 回话时 skill 提醒并问"是否继续 batch 2 或先复盘 batch 1？"
- 超 7 天 → 自动发 PushNotification 提醒
- 超 30 天 → 强烈建议丢弃 vault 重跑

---

## 维护备忘

- 每次跑出"切分点不合理"（某 batch 太长 / 太短），调整推荐切分点
- 估算时长公式根据实际数据调（log 实际 vs 估算）
- PM 反馈"中间介入功能"加强 → 加新 batch 间介入选项
