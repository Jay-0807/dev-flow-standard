# 决策回放机制（C 节）

> **作用**：vault 在关键点自动 checkpoint（纯文件 copy），PM 在 Phase 12.5 早晨复盘可"replay from decision #N"——skill 恢复 vault 到该决策前状态，注入新方向重跑下游。
> **v4 新增**：来自 Phase 12.5 4 选项中的 🔄 局部 redo 路径。

---

## 设计哲学

PM 早晨复盘时可能说："决策 #7 我不同意，从这点重新跑。" 想做到这点必须有"vault 时光机"——能把 vault 状态恢复到决策 #7 前一刻，然后给 skill 注入新方向继续跑。

**核心权衡**：完整 git worktree 快照（重）vs 纯文件 copy（轻）。本方案选 **纯文件 copy** 为默认（轻量）。

---

## 何时 checkpoint（决策密度控制）

不是每条决策都做 checkpoint（vault 太大）。规则：

| Checkpoint 触发 | 频率（一次迭代约）|
|---|---|
| ✅ 每个 phase 完成（Phase 2.5-12 边界）| 10-12 次 |
| ✅ 每个 R 红线 escalation 解决后 | 0-3 次 |
| ✅ 自治决策中标 ⚠️（PM 早晨需 review 的）| 3-8 次 |
| ✅ Karpathy 4 原则触发"重大架构变更"的决策 | 0-2 次 |
| ❌ 普通自治决策（不标 ⚠️）不 checkpoint | - |

**一次完整迭代预期 checkpoint 数：13-23 个**（可控）。

---

## 文件结构

```
iteration-vault/
├── ... 原有文件
└── checkpoints/
    ├── README.md                       # checkpoint 索引（PM 可读）
    ├── ckpt-phase-2.5-start/           # Phase 2.5 开始前
    │   ├── meta.json                   # 触发原因 + 时间戳 + 决策 ID（如适用）
    │   ├── 02-PRD.md.snapshot          # 当时的 PRD 副本
    │   └── autonomous-decisions.md.snapshot
    ├── ckpt-phase-4-start/
    ├── ckpt-phase-7-start/
    ├── ckpt-decision-007/              # 决策 #7 前的状态
    │   ├── meta.json
    │   ├── trigger.md                  # 触发本 checkpoint 的决策摘要
    │   ├── 04-architecture.md.snapshot
    │   ├── 04.5-api-design.md.snapshot
    │   ├── 06-task-breakdown.md.snapshot
    │   └── autonomous-decisions.md.snapshot (decisions 1-6)
    ├── ckpt-r1-escalation-resolved/
    └── ...
```

每个 checkpoint 仅 snapshot **该决策点之后可能受影响的文件**，不是整个 vault 全量。Checkpoint 大小预期 10-100 KB 级。

---

## checkpoint meta.json 格式

```json
{
  "checkpoint_id": "ckpt-decision-007",
  "trigger_type": "warning_decision",  // phase_boundary | warning_decision | escalation_resolved
  "trigger_decision_id": 7,            // 如适用
  "trigger_phase": "4-architecture",
  "created_at": "2026-05-23T22:14:00+08:00",
  "vault_state_summary": "decisions 1-6 done, decision 7 pending",
  "snapshotted_files": [
    "04-architecture.md",
    "04.5-api-design.md",
    "06-task-breakdown.md",
    "autonomous-decisions.md"
  ],
  "files_NOT_snapshotted_reason": "07+ 还没跑，不存在"
}
```

---

## 决策回放执行流程（PM 在 Phase 12.5 触发）

```
PM: "我想从决策 #7 回放，方向改为'直接删 deprecated 字段'"
                    ↓
[skill 内部]
1. Read iteration-vault/checkpoints/ckpt-decision-007/
2. 创建回放分支：iteration-vault/replay-from-decision-007/
   ├── 把 ckpt-decision-007/*.snapshot 恢复为活跃文件
   ├── 复制当前 vault 的 02-PRD.md（PRD 不回滚，只回滚决策树）
   └── 在 autonomous-decisions.md 截断到 decision #6
      + 写新方向注入条 [REPLAY-PM-DIRECTIVE]
3. 切到 replay 模式 → state.yaml mode=night-mode-replay
   → 进入夜间模式 batch
   → 从 decision #7 触发点对应的 phase 开始重跑
   → 注入 PM 新方向作为强约束
   → skill 续跑到 Phase 12 + 12.5
4. 完成后 vault 替换：
   - 旧 vault → iteration-vault/history/<timestamp>-replayed-from-d7/
   - replay-from-decision-007/ → iteration-vault/（活跃）
```

---

## git worktree vs 纯 copy 决策

| 维度 | 纯文件 copy（推荐 v4 默认）| git worktree per checkpoint |
|---|---|---|
| 实现复杂度 | 低 | 高 |
| 磁盘占用 | 低（仅 vault 文件）| 高（含 node_modules / dist）|
| 代码状态恢复 | ❌ 不恢复源码 | ✅ 完整恢复 |
| 适用场景 | vault 决策回退 | 同时回退代码改动 |

**v4 默认：纯文件 copy**。理由：
1. 决策回放主要是 vault 文件（PRD/架构/任务）的语义回退，源码可以让 skill 在 replay batch 重新生成
2. git worktree-per-checkpoint 会让磁盘爆炸（10+ worktree × 含 node_modules 体积）
3. 如果 PM 要"代码状态也回到决策 #7"，单独提供"代码紧急回滚"命令（git revert + 重跑 Phase 7-11），不放进决策回放主流程

但**保留 worktree 选项**：PM 在 PRD 阶段可显式说"我要 worktree 级 checkpoint" → skill 切换到 worktree-per-checkpoint 模式（默认不开）。

---

## 失败模式

| 失败 | 处理 |
|---|---|
| Checkpoint 文件缺失（被误删）| skill 拒绝回放该决策，提示 PM 只能选 ❌ 整体 redo |
| 回放后 skill 再次跑出更差结果 | autonomous-decisions.md 标 [REPLAY-WORSE-THAN-ORIGIN]，PM 在下次 12.5 看到 |
| 回放跑到一半触发红线 | 同 v2：暂停 + ESCALATION-R*.md（不影响 origin vault）|
| 连续回放 3 次仍不满意 | skill 主动建议"建议 ❌ 整体 redo + 重审 PRD"，因为方向可能本就不对 |
| Checkpoint snapshot 在 read-only file system | 写 `iteration-vault/CHECKPOINT_ERROR.md` + 标 R1 升级（罕见）|

---

## checkpoints/README.md 自动生成

每次 checkpoint 创建时，更新 vault 的 `checkpoints/README.md` 索引：

```markdown
# Vault Checkpoints Index

> 本文件由 skill 自动维护。PM 在 Phase 12.5 局部 redo 时使用。

| Checkpoint ID | 触发 | Phase | 决策 ID | 时间 | 可回放？|
|---|---|---|---|---|---|
| ckpt-phase-2.5-start | phase 边界 | 2.5 | - | 2026-05-23 21:00 | ✅ |
| ckpt-phase-4-start | phase 边界 | 4 | - | 2026-05-23 21:30 | ✅ |
| ckpt-decision-007 | ⚠️ 决策 | 4 | #7 | 2026-05-23 21:38 | ✅ |
| ckpt-decision-012 | ⚠️ 决策 | 4.5 | #12 | 2026-05-23 22:10 | ✅ |
| ckpt-r1-resolved | R 解决 | 4.5 | - | 2026-05-23 22:30 | ✅ |
| ckpt-phase-6-start | phase 边界 | 6 | - | 2026-05-23 23:00 | ✅ |
...

总数：23
总磁盘占用：~1.4 MB
```

---

## 跨迭代清理

每次迭代归档到 `history/<timestamp>-<feature>/` 时：
- `checkpoints/` 整目录保留（PM 后续追溯）
- 30 天后可手动清理（节省磁盘）

---

## 维护备忘

- 每次跑出"checkpoint 太多影响 vault 浏览"，调整触发规则（如只 phase 边界 + 严重 ⚠️）
- 跑出"checkpoint 该有但没有"的情况（如 PM 想 replay 但缺 ckpt），加新触发点
- worktree 选项使用频率监控，决定是否做默认升级
