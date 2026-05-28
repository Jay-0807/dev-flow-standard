# 跨夜分批跑计划模板

> 把本模板复制为 `iteration-vault/batches/batch-plan.md`（仅多批跑时才创建），逐段填充。

---

```markdown
# 夜间模式分批计划: <feature-name>

**总批数**: 2 / 3
**当前批**: 1
**已完成批**: []
**估算总跑时**: <H.H>h
**估算依据**: Phase 6 任务数 [N] × 平均 [X] min + 设计 phase GAN × [M] + 固定 [Y]h

---

## 批次计划

### Batch 1（设计批）— 预计 [X]h

- **Phase 范围**: 2.5-6
  - Phase 2.5 brainstorm
  - Phase 3 影响面
  - Phase 4 架构
  - Phase 4 §2 API 整理
  - Phase 5 §1 UX
  - Phase 5 §2 UI Spec
  - Phase 5.9 文档压缩
  - Phase 6 任务分解
- **启动时间**: <YYYY-MM-DD HH:mm>
- **结束时间**: <运行中 / completed>
- **GAN 任务数**: <运行中 / 7>
- **决策数**: <运行中 / K>
- **红线触发**: <运行中 / 0>
- **batch 结束行为**: ⏸ 等 PM 决定是否启 Batch 2

### Batch 2（实施批）— 预计 [Y]h

- **Phase 范围**: 7-8
  - Phase 7 并行实施
  - Phase 8 代码债
- **启动条件**: PM 显式说 "/next-batch" 或 "继续夜间模式"
- **启动时间**: <未启动>
- **GAN 任务数**: Phase 7 code task × ~22 = ~22 GAN
- **关键风险**: DB migration 必须在 batch 2 开始时确认 schema 仍 OK
- **batch 结束行为**: ⏸ 等 PM

### Batch 3（审发批）— 预计 [Z]h（如分 3 批）

- **Phase 范围**: 9-12
  - Phase 9 三路审查
  - Phase 10 五层验收
  - Phase 11 发布说明
  - Phase 11.5 漂移检测
  - Phase 12 git release
- **启动条件**: Batch 2 完成 + PM 显式说 "/next-batch"
- **启动时间**: <未启动>
- **GAN 任务数**: Phase 11 release notes × 1 = ~1 GAN
- **batch 结束行为**: 自动进入 Phase 12.5

---

## 跨批延续指南（每批结束自动写）

### Batch 1 → Batch 2 衔接备忘

**关键决策**（3-5 条，来自 autonomous-decisions.md）：
- 决策 #3: <一句话>
- 决策 #7: <一句话>
- 决策 #12: <一句话>

**待 PM 复盘的 ⚠️ 决策**：
- #7, #12, #15

**建议优先关注**（1-2 条）：
- <最值得 PM 关注的一条>

**下批读什么**：
- iteration-vault/INDEX.md (来自 5.9 压缩)
- iteration-vault/RULES.md
- iteration-vault/06-task-breakdown.md

**下批跑什么**：
- Phase 7: [N] 个 code task（[M] 含前端，[K] 机械）
- Phase 8: 9 维度代码债扫描

### Batch 2 → Batch 3 衔接备忘

待 batch 2 完成后填。

---

## 状态机

```
[未启动]
    ↓ PRD 通过 + PM 选分批数
[batch-plan 创建]
    ↓ 自动启动
[Batch 1 running]
    ↓ 跑完
[Batch 1 done, waiting]
    ↓ PM "/next-batch"
[Batch 2 running]
    ↓ 跑完
[Batch 2 done, waiting]
    ↓ PM "/next-batch"
[Batch 3 running]
    ↓ 跑完
[All batches done → Phase 12.5]
```

---

## 异常状态

- **Batch X 触发 R 红线**: state.yaml mode=escalated, batch X paused
- **Batch X 跨批级联失败**: 写 CROSS-BATCH-ISSUE.md, batch X paused
- **PM 离线 > 3 天**: state.yaml mode=batch_*_awaiting_kick_since=<date>
- **PM 中断**: PM 说 "/stop" → 当前 phase 跑完即停 → 进 Phase 12.5

---

## 当前 vault 状态（每批末更新）

- 文件数: [N]
- checkpoint 数: [M]
- GAN 目录数: [K]
- 总大小: [X.X] MB
- 跑时累计: [H.H]h（actual）vs [Y.Y]h（estimated）
```
