# 🤖 Autopilot 回流

> 内部编号：Phase 13
> 模式：🤖 Autonomous（仅 autopilot 触发时跑）
> 用途：写 run-history.jsonl + state.json update，回流给 Autopilot 模式

> **作用**：主流水线 Phase 12 完成 + Phase 12.5 PM 复盘后，把"本轮结果"反馈给 autopilot 唤醒循环。这是 autopilot Sibling Cycle 的接口点。
> **触发**：Phase 12.5 PM 选 ✅ merge 或 ⏸ 推下夜或 ❌ 重做 时。

---

## 工作流（4 步）

### Step 1：判断本轮是否 autopilot-triggered

读 `iteration-vault/00-intake.md`：
- 如含 `autopilot_metadata` 段 → 是 autopilot-triggered，进 Step 2
- 否则 → PM 主动提的迭代，本 phase 不做事，直接结束

### Step 2：写 run-history 记录

追加一行到 `~/.autopilot/run-history.jsonl`：

```json
{
  "candidate_id": "<from 00-intake.md>",
  "started_at": "<from 00-intake.md autopilot_metadata.triggered_at>",
  "ended_at": "<now>",
  "duration_hours": <X.X>,
  "outcome": "ok | r1 | r2 | r3 | r4 | infra-err | pm-rejected | pm-postponed",
  "release_pr": "<URL or null>",
  "phase_12_5_decision": "merge | redo | reject | postpone",
  "vault_archived_to": "<path>",
  "decision_count": <K>,
  "warning_decisions": <X>,
  "gan_invocations": <N>,
  "tier_at_time": <1/2/3>
}
```

### Step 3：更新 autopilot state.json

按 Phase 12.5 PM 选项分流：

| Phase 12.5 选项 | state.json 转换 | autopilot 下次行为 |
|---|---|---|
| ✅ merge | state = IDLE | 下次 cron 正常选题 |
| 🔄 redo | state = RUNNING (continue) | 不进 IDLE，继续当前 candidate |
| ❌ 重做 | state = IDLE + candidate 进 7 天 cooldown | 下次 cron 跳过此 candidate |
| ⏸ 推下夜 | state = AWAITING_MORNING_REVIEW（不变）| 下次 PM 回话 |

### Step 4：连续失败检测

读 `run-history.jsonl` 最近 2-3 条：

- 如最近 2 次都 outcome != ok → 降 Tier（3→2→1）
- 如最近 2 次都 outcome != ok → state = KILL_SWITCHED
- 如最近 3 次都 outcome != ok → state = EMERGENCY_STOPPED

写 `~/.autopilot/CONSECUTIVE_FAILURE.md`（如适用）。

---

## 输入

- `iteration-vault/00-intake.md`（判断 autopilot-triggered）
- `iteration-vault/12.5-morning-review.md`（PM 选项）
- `iteration-vault/12-release.md`（Release PR URL）
- `iteration-vault/autonomous-decisions.md`（决策数）

## 产出

- `~/.autopilot/run-history.jsonl`（append 一行）
- `~/.autopilot/state.json`（update）
- `~/.autopilot/CONSECUTIVE_FAILURE.md`（如适用）
- 触发 `templates/morning-digest.md` 生成（如适用，写到 `~/.autopilot/morning-digest-<date>.md`）

## 失败回退

- `run-history.jsonl` 写不进 → 写到 vault `13-autopilot-handoff.error.md`，下次唤醒重试
- state.json 更新失败 → 同上

---

## 与 autopilot 的契约

本 phase 是 autopilot Sibling Cycle 的"出口"：

```
Autopilot Wake-up Loop
   ↓ (注入 00-intake.md)
Main Pipeline (Phase 0 → 12 → 12.5)
   ↓ (PM 复盘)
Phase 13 Autopilot Handoff  ← 本文件
   ↓ (回流 run-history.jsonl + state.json)
Autopilot Wake-up Loop（下次 cron 启动时读 run-history）
```

---

## 维护备忘

- run-history 字段升级时同步 autopilot/state-machine.md
- outcome 类型新增时同步 safety-brakes.md L6
- Phase 12.5 选项新增时同步本文件 Step 3 表
