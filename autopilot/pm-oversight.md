# Autopilot PM 监督命令规格

> **作用**：PM 与 autopilot 交互的全部命令。Tier 1 PM 主要用这些。
> **实现**：本 skill SKILL.md 末尾加一段 "## Autopilot Slash Commands" 引用本文件。

---

## 命令列表

### `/autopilot-status`

显示当前状态全貌。

输出：
```
Autopilot State: AWAITING_APPROVAL
Tier: 1
Last cron: 2026-05-23 09:00:00
Last harvest: 27 candidates from 5 connectors
Last ranked: 19 (after blacklist filter)

Today's pick (#1):
  - candidate-roadmap-042 "给导出功能加二阶反馈"
  - score 3.42 / size M / 估算 5-7h
  - blast_radius: 8 文件 / 不触 critical path → ✅
  - safety brakes: 全通过

Recent 7-day history:
  - 2026-05-22: candidate-abc done (PR #142 merged)
  - 2026-05-21: candidate-xyz failed (R3 escalation, PM 决策推迟)
  - 2026-05-20: candidate-def done (PR #140 merged)
  ...

INBOX status: 1 待审批
PAUSE 文件: 不存在 → 启用
EMERGENCY_STOP 文件: 不存在
```

### `/autopilot-queue`

打开 `~/.autopilot/queue.md`。

### `/autopilot-pause`

```bash
touch ~/.autopilot/PAUSE
echo "Autopilot 已暂停。/autopilot-resume 恢复。"
```

### `/autopilot-resume`

```bash
rm ~/.autopilot/PAUSE 2>/dev/null
echo "Autopilot 已恢复。下次 cron tick 时启动。"
```

### `/autopilot-stop`

```bash
touch ~/.autopilot/EMERGENCY_STOP
echo "🚨 Autopilot 紧急停。需 /autopilot-reset 才能继续。"
```

### `/autopilot-reset`

```bash
rm ~/.autopilot/PAUSE 2>/dev/null
rm ~/.autopilot/EMERGENCY_STOP 2>/dev/null
rm ~/.autopilot/CONSECUTIVE_FAILURE.md 2>/dev/null
# 不清 state.json，仍可看到 last_state
echo "Autopilot 已重置。下次 cron tick 时启动。"
```

### `/autopilot-tune`

PM 调 config.yaml 常见 knob，AskUserQuestion 给选项：

- 调 tier（1/2/3）
- 调每日预算
- 启用/禁用 connector
- 修改 forbidden_paths

### `/autopilot-blacklist <candidate-id>`

把某候选拉黑：

```yaml
# 追加到 ~/.autopilot/blacklist.yaml
- candidate_id: <id>
  reason: <PM 输入>
  until: <date or "never">
```

### `/autopilot-veto <today>`

否决今晚 Tier 2 自动启动（仅 Tier 2 / 3 模式有意义）：

```
echo "今晚 autopilot 不会自动启动。queue.md 仍可看，但不会跑。"
```

### `/autopilot-tier <1|2|3>`

切换 Tier：

```bash
# 修改 ~/.autopilot/config.yaml tier 字段
# 提示 PM 当前 Tier 行为差异
```

### `/autopilot-debug <candidate-id>`

显示某候选的评分详细（见 `priority-algorithm.md` §调试模式）。

### `/autopilot-history [N]`

显示最近 N 次跑历史（默认 7 天）：

```
2026-05-22 23:00 - candidate-roadmap-042 - RUNNING → ok (PR #143)
2026-05-22 09:00 - candidate-sentry-099 - vetoed by PM
2026-05-21 09:00 - candidate-roadmap-040 - RUNNING → r3 (推迟)
2026-05-20 ... 
```

### `/autopilot-blacklist-list`

显示当前黑名单。

```yaml
- candidate-sentry-1234 (until 2026-12-31): "Known false positive"
- candidate-github-47 (until never): "Blocked on third-party API"
```

### `/autopilot-blacklist-remove <candidate-id>`

从黑名单移除。

---

## 自然语言触发

PM 不一定记得 slash 命令，自然语言也应触发：

| 自然语言 | 等价命令 |
|---|---|
| "autopilot pick #N 启动" / "今晚跑 #N" | 确认 candidate N + 转 RUNNING |
| "autopilot pause" / "暂停" / "今晚别跑" | `/autopilot-pause` |
| "autopilot 状态" / "queue 是什么" | `/autopilot-status` + `/autopilot-queue` |
| "拉黑 #N，理由 X" | `/autopilot-blacklist <id>` |
| "切到 Tier 2" / "提升信任级别" | `/autopilot-tier 2` |
| "紧急停" / "立刻停 autopilot" | `/autopilot-stop` |
| "重置 autopilot" | `/autopilot-reset` |

---

## INBOX.md 格式

`~/.autopilot/INBOX.md`（PM 每天看）：

```markdown
# Autopilot INBOX

> 最后更新: 2026-05-23 09:02

## 待决策（1）

### #1 candidate-roadmap-042 "给导出功能加二阶反馈"

- 来源: roadmap.md L42
- 优先级 score: 3.42
- 大小: M（估算 5-7h）
- 商业影响: 5/5 (PM 显式标 P0)
- 用户疼痛: 5/5 (3 轮反馈重复)
- 安全闸: 全通过

**今晚跑这个？** 
- ✅ 跑：「autopilot pick #1 启动」
- 🔄 改：选其他 candidate（见 queue.md）
- ❌ 拒：「拉黑 #1，理由 X」

---

## 最近 7 天 done

- 2026-05-22: candidate-abc done (PR #142)
- 2026-05-21: candidate-xyz failed (R3 推迟)

## 黑名单（最近加的）
- candidate-sentry-1234 (PM 标"已知误报")
```

---

## 维护备忘

- 新增 slash 命令时同步本文件 + skill SKILL.md
- 自然语言映射根据 PM 实际用法迭代
- INBOX 格式让 PM 一眼能扫
