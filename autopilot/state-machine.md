# Autopilot 状态机

> **作用**：autopilot 的全部运行状态定义 + 状态间转换规则。
> **持久化**：`~/.autopilot/state.json`

---

## 状态图

```
                ┌──────────┐
                │   IDLE   │ ← 默认状态（kill switch 不存在时）
                └────┬─────┘
                     │ cron tick 触发
                     ▼
              ┌──────────────────┐
              │   PRE_FLIGHT     │
              └────┬─────────┬───┘
                   │ pass    │ fail
                   ▼         ▼
        ┌──────────────┐  ┌────────────────────────┐
        │  HARVESTING  │  │ SKIPPED_PRE_FLIGHT     │ → IDLE
        └──────┬───────┘  └────────────────────────┘
               ▼
        ┌──────────────┐
        │   RANKING    │
        └──────┬───────┘
               ▼
        ┌──────────────────┐
        │ AWAITING_APPROVAL│  (Tier 1, or Tier 2/3 for large candidates)
        └──┬──────────┬────┘
   ❌ veto │          │ ✅ or timeout (Tier 2/3 small)
           ▼          ▼
        IDLE       ┌──────────────┐
                   │   RUNNING    │ → 启动 14-phase 主流水线
                   └──┬───────┬───┘
                      │ ok    │ fail
                      ▼       ▼
                 ┌──────────────┐  ┌──────────────────┐
                 │ AWAITING_    │  │ R[1-4] ESCALATED │
                 │ MORNING_     │  │ (PM 决策中)      │
                 │ REVIEW       │  └────────┬─────────┘
                 └──────┬───────┘           │
                        │                   │
                  PM ack│            PM 决策 ▼
                        ▼              ┌──────────────┐
                      IDLE             │ RESUMED 或   │
                                       │ TERMINATED   │ → IDLE
                                       └──────────────┘

ERROR STATES（任何时刻可进入，不可自愈，需 PM reset）:

  ┌────────────┐  ┌─────────────────────┐  ┌──────────────────┐
  │  PAUSED    │  │ KILL_SWITCHED       │  │ EMERGENCY_       │
  │ (PAUSE     │  │ (≥2 连续失败)        │  │ STOPPED          │
  │  file 存在) │  │                     │  │ (≥3 连续失败 OR  │
  │            │  │                     │  │  PM 手动 stop)   │
  └────────────┘  └─────────────────────┘  └──────────────────┘
        │              │                          │
        │ rm PAUSE     │ PM /autopilot-reset      │ PM 手动 reset
        ▼              ▼                          ▼
       IDLE           IDLE                      IDLE

  ┌──────────────────────┐
  │ BUDGET_EXHAUSTED     │  → 次日 00:00 自动 reset
  └──────────────────────┘
```

---

## 状态持久化

`~/.autopilot/state.json`：

```json
{
  "state": "AWAITING_MORNING_REVIEW",
  "since": "2026-05-23T05:42:00+08:00",
  "current_candidate_id": "candidate-roadmap-042",
  "current_iteration_vault": "/path/to/iteration-vault/",
  "history_pointer": "run-history.jsonl#L17",
  "tier": 1,
  "last_pre_flight_check_at": "2026-05-23T09:00:01+08:00",
  "last_harvest_at": "2026-05-23T09:01:30+08:00",
  "last_pick_at": "2026-05-23T09:02:00+08:00"
}
```

---

## 状态详细定义

### IDLE
默认状态。下次 cron 触发时进入 PRE_FLIGHT。
**进入**：每个迭代结束后 / kill switch 移除后 / EMERGENCY_STOP reset 后。
**离开**：cron tick / PM 手动触发。

### PRE_FLIGHT
跑 `safety-brakes.md` L3 的 8 项检查。
**通过**：进入 HARVESTING。
**失败**：进入 SKIPPED_PRE_FLIGHT，写日志，回 IDLE。

### HARVESTING
按 config.yaml `connectors.<name>.enabled=true` 的 connector 逐个拉候选。
并行（不同 connector 互不依赖）。
**完成**：进入 RANKING。
**MCP 整体崩**：用 fallback connector 列表，继续；全部崩 → SKIPPED_PRE_FLIGHT。

### RANKING
按 `priority-algorithm.md` 排序，写 queue.md。
**完成**：进入 AWAITING_APPROVAL（Tier 1）或直接 PICK + RUNNING（Tier 2 small / Tier 3）。

### AWAITING_APPROVAL
queue.md + INBOX.md 已写。等 PM 通过 `/autopilot-status` 或自然语言 "autopilot pick #N 启动" 触发。
**最长等待**：到下一个 cron tick（24h）。超期 → 写"待审 24h"标记，仍 IDLE。
**PM 选 ✅**：进入 RUNNING。
**PM 选 ❌**：candidate 进 blacklist，回 IDLE。
**PM 选改候选**：取 queue 中其他候选，重新进入 AWAITING_APPROVAL（其实可以跳过这步直接 RUNNING）。

### RUNNING
触发主流水线 Phase 0（注入 00-intake.md + autopilot-seed.md）。
主流水线运行 Phase 0 → 12.5。
**主流水线 normal end**（Phase 12 完成 + Phase 12.5 待 PM）：进入 AWAITING_MORNING_REVIEW。
**主流水线触发 R 红线**：进入 R[1-4] ESCALATED。
**主流水线 INFRASTRUCTURE_ERROR**：暂停 + 写 log，回 IDLE（等次日重试）。

### AWAITING_MORNING_REVIEW
等 PM 走 Phase 12.5 4 步清单 + 4 选项。
**PM 选 ✅ merge**：写 run-history.jsonl(outcome=ok)，归档 vault，回 IDLE。
**PM 选 🔄 redo**：进入 RUNNING redo batch（仍同 candidate_id）。
**PM 选 ❌ 重做**：归档到 history/REJECTED-，回 IDLE + candidate 进 blacklist 7 天 cooldown。
**PM 选 ⏸ 推下夜**：state 继续 AWAITING_MORNING_REVIEW，下次 PM 回话重入。

### R[1-4] ESCALATED
主流水线触发任一红线。
**等 PM 在 ESCALATION-R[X].md 给方向**。
**PM 给方向后**：进入 RUNNING 续跑。
**PM 选终止**：进入 TERMINATED，写 run-history(outcome=r[1-4]_aborted)，回 IDLE。

### TERMINATED
PM 主动终止本次迭代。
归档到 history/，回 IDLE。

### PAUSED
`~/.autopilot/PAUSE` 文件存在。
**Cron 触发也不动**（直接跳过）。
**恢复**：删 PAUSE → 回 IDLE。

### KILL_SWITCHED
连续 2 次失败（outcome != ok）。
**自动转入**：从 AWAITING_MORNING_REVIEW（PM 看到 outcome=r3 等）转入。
**恢复**：PM `/autopilot-reset` 后回 IDLE。

### EMERGENCY_STOPPED
连续 3 次失败 OR PM 手动 `/autopilot-stop`。
**比 KILL_SWITCHED 严**。
**恢复**：PM 手动检查 + `/autopilot-reset` 后回 IDLE。

### BUDGET_EXHAUSTED
当日 max_iterations_per_day / token 上限触发。
**次日 00:00 自动 reset**：回 IDLE。

---

## 状态转换的副作用

| 转换 | 必跑动作 |
|---|---|
| IDLE → PRE_FLIGHT | 写 wake-up log header |
| PRE_FLIGHT → SKIPPED_PRE_FLIGHT | 写 pre-flight-failed.log |
| RANKING → AWAITING_APPROVAL | 写 queue.md + INBOX.md |
| AWAITING_APPROVAL → RUNNING | 写 00-intake.md + autopilot-seed.md + state.json 更新 candidate_id |
| RUNNING → AWAITING_MORNING_REVIEW | Phase 12 完成时 |
| AWAITING_MORNING_REVIEW → IDLE (merge) | 归档 vault + run-history append + INBOX clear |
| RUNNING → R*_ESCALATED | 写 ESCALATION-R[X].md |
| → KILL_SWITCHED | 写 CONSECUTIVE_FAILURE.md + 通知 PM |
| → EMERGENCY_STOPPED | 写 EMERGENCY_STOP file + 强通知 PM |

---

## 重启 / context 恢复

skill 进程重启或 context 耗尽时，按以下顺序恢复：

```
1. Read ~/.autopilot/state.json
2. 根据 state 字段决定当前应该做什么：
   - IDLE → 等下次 cron
   - PRE_FLIGHT → 重新跑（idempotent）
   - HARVESTING / RANKING → 重跑（idempotent）
   - AWAITING_APPROVAL → 显示 INBOX 等 PM
   - RUNNING → 读 iteration-vault/state.yaml + .claude/handoff.md 恢复主流水线
   - AWAITING_MORNING_REVIEW → 显示"夜间跑完，请走 Phase 12.5"
   - PAUSED / KILL_SWITCHED / EMERGENCY_STOPPED → 显示对应通知 + 等 PM
3. 不要清 state.json，下次操作继续 update
```

---

## 维护备忘

- 新增状态时同步本文件 + state-machine 实现代码
- 失败 outcome 类型增加（如 outcome=cancelled_by_pm）需在 KILL_SWITCHED 触发条件加上
- 监控状态分布（跑久了哪个状态比例高？）→ 优化
