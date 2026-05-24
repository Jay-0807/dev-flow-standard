# Autopilot 6 层安全闸

> **作用**：autopilot 自治选题前 + 启动前 + 运行中的硬约束。任一闸触发 → 跳过本候选或暂停 autopilot。
> **优先级**：Kill Switch > Budget > Pre-flight > Blast Radius > Working Hours > Consecutive Failures

---

## L1：Kill Switch（最优先）

```
~/.autopilot/PAUSE              # 文件存在 → 暂停（可恢复）
~/.autopilot/EMERGENCY_STOP     # 文件存在 → 立刻停 + 任何在跑的 phase 中断
```

每轮唤醒前**第一件事**就是检查这两个文件。`touch ~/.autopilot/PAUSE` 即可暂停。

**PM 命令** alias：
- `/autopilot-pause` → 创建 PAUSE
- `/autopilot-resume` → 删除 PAUSE
- `/autopilot-stop` → 创建 EMERGENCY_STOP

恢复方式：
- PAUSE → `/autopilot-resume` 删除 PAUSE
- EMERGENCY_STOP → 需 PM 手动 `/autopilot-reset` 后才能重新 resume

---

## L2：每日预算上限

```yaml
budget:
  max_iterations_per_day: 1
  max_iterations_per_week: 5
  max_llm_tokens_per_day: 5_000_000
  max_consecutive_failures_before_pause: 2
```

**检查时机**：唤醒后 HARVEST 前。
**违规处理**：
- 超 daily 上限 → 写 `BUDGET_EXHAUSTED.md` + 跳过本轮，次日 00:00 自动重置
- 超 weekly 上限 → 写 `BUDGET_EXHAUSTED.md` + 跳过到下周一
- token 用尽 → 暂停 + 通知 PM

---

## L3：Pre-flight Check（每轮唤醒前必跑）

```
☐ main 分支无未 commit 改动 (`git status` clean)
☐ main 分支与 origin/main 同步 (`git fetch && git status`)
☐ 没有正在跑的 iteration-vault（即没有上一轮卡住）
☐ 没有 ESCALATION-R*.md 未处理
☐ 测试套件最近一次 CI 状态 = passing (`gh run list -L 1`)
☐ Sentry 当前 incident 数 == 0（如 Sentry MCP 可用）
☐ gh CLI 已登录 (`gh auth status`)
☐ 必需 MCP 都连得通（按配置 connector 列表）
```

任一项失败 → 跳过本轮 + 写 `~/.autopilot/logs/pre-flight-failed-<date>.log` + INBOX 通知 PM。

---

## L4：Blast Radius 限制

```yaml
blast_radius_limits:
  max_files_changed: 30
  max_loc_added: 500
  forbidden_paths:
    - "src/auth/**"
    - "infra/**"
    - "**/migrations/**"
    - ".github/workflows/**"
    - "package.json"
    - "next.config.js"
```

**判定时机**：
- HARVEST 阶段 connector 给 `blast_radius_hint`（粗估）
- PICK 阶段 autopilot 跑一个轻量 grep 估算实际 blast radius
- 超限 → 候选**仍进 queue.md**（让 PM 知道有这事），但**不自动启动**，必须 PM 手动批

**估算方法**：
- 读 candidate.title 关键词
- grep 代码库哪些文件可能受影响
- 给出 `files_estimated_to_change: N` 和 `forbidden_path_touched: bool`

**Tier 分级**：
- forbidden_paths 全 Tier 禁
- forbidden_for_tier_2_only 仅 Tier 2 自动跑时禁
- forbidden_for_tier_3_only 仅 Tier 3 全自动时禁

---

## L5：工作时段栅栏

```yaml
working_hours_fence:
  allowed_start_hours: [21, 22, 23, 0, 1, 2, 3, 4]
  forbidden_days: ["Sat", "Sun"]
  forbidden_dates: ["2026-10-01..2026-10-07"]
```

cron 触发时机不变（09:00 选题），但**实际 Phase 7 实施 loop 只在允许时段启动**。

**违规处理**：
- HARVEST + RANK 仍跑（这些不耗资源）
- 但 Phase 0 启动延后到下一个 allowed_start_hour
- 写 `deferred-start.log` 记录延后

---

## L6：连续失败暂停

跑过的迭代记录在 `~/.autopilot/run-history.jsonl`，每条：

```json
{"candidate_id": "...", "started_at": "...", "ended_at": "...", "outcome": "ok|r1|r2|r3|r4|infra-err"}
```

每次唤醒检查最近 N 次 outcome：

```
if outcome[-2:] 中 ≥ 1 个非 ok:
    降级 (Tier 3 → Tier 2 → Tier 1)
if outcome[-2:] 全部非 ok:
    自动 PAUSE，写 CONSECUTIVE_FAILURE.md
if outcome[-3:] 全部非 ok:
    EMERGENCY_STOP（PM 必须 reset 才能继续）
```

---

## 优先级（处理顺序）

```
唤醒触发
   ↓
L1 Kill Switch ──❌─→ 立即停
   ↓ pass
L2 Budget ─────❌─→ 跳过本轮 + 重置时间
   ↓ pass
L3 Pre-flight ─❌─→ 跳过本轮 + 写日志
   ↓ pass
HARVEST + RANK
   ↓
PICK 候选
   ↓
L4 Blast Radius ❌─→ 候选不自动跑（仍进 queue）
   ↓ pass
L5 Working Hours ❌─→ 延后到 allowed 时段
   ↓ pass
L6 Consecutive Failures ❌─→ 降级 / 暂停 / 紧急停
   ↓ pass
启动 Phase 0
```

---

## 维护备忘

- 每次 autopilot 触发安全闸，记日志，PM 早晨复盘时关注
- forbidden_paths 列表根据实际项目结构调整
- L6 阈值（2 次失败 vs 3 次失败）根据 autopilot 稳定性调
- Tier 切换（PM 主动 /autopilot-tier 2）时检查 L4 forbidden_for_tier_X
