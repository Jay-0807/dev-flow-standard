# Autopilot 种子模板

> Autopilot 选定候选后，把候选信息按本模板写到 `iteration-vault/00-intake.md`，让主流水线 Phase 0 接管。

---

```markdown
# Intake (autopilot-triggered)

> **本轮迭代由 autopilot 自动触发**（不是 PM 主动提需求）

**触发时间**: <YYYY-MM-DD HH:mm>
**Tier**: [1/2/3]
**候选 ID**: <candidate-id>
**autopilot 状态机记录**: state.json snapshot

---

## PM 原话（如有）

[如 source=roadmap，引 PM 在 roadmap.md 写的原话]
[如 source=sentry/github，N/A]

---

## autopilot 推荐做这个的理由

- **来源**: <source + source_ref>
- **优先级 score**: <X.XX>（top 1 / top N）
- **商业影响**: <X>/5（<理由>）
- **用户疼痛频次**: <X>/5（<理由>）
- **age**: <N> 天
- **复杂度估算**: <S/M/L>

---

## 完整证据链

- **主证据**: <link to source>
- **旁证 1**: <如有>
- **旁证 2**: <如有>
- **Sentry 相关 issue**: <如有>

---

## 大小分级 (autopilot 估算)

<small/medium/large>

依据：
- candidate.estimated_size 来自 connector
- blast_radius_hint: <N> 文件 / 触 critical path: <yes/no>
- 估算工时: <X-Y>h

---

## autopilot 推测的"PM 会说的话"

> 这是 autopilot 自动生成的"假装 PM 提需求"段，用于触发主流水线 Phase 1 澄清。
> Phase 2 PRD 关卡 PM 仍会看到（Tier 1/2）或 autopilot 自批（Tier 3 small）。

"<一句话需求>"

例：
"给导出功能加二阶反馈（PM 可以告诉系统上次导出结果为啥不对），下一次导出时系统据此修正。"

---

## 是否需要 PM 进一步澄清？

是 → 主流水线 Phase 1 照常跑（澄清反问 + AskUserQuestion）

---

## autopilot 元信息

```yaml
autopilot_metadata:
  candidate_id: <id>
  source: <connector>
  triggered_at: <iso8601>
  tier: <1/2/3>
  safety_brakes_passed:
    - L1: kill_switch
    - L2: budget
    - L3: pre_flight
    - L4: blast_radius
    - L5: working_hours
    - L6: consecutive_failures
  pre_flight_check:
    - main_clean: true
    - origin_synced: true
    - no_active_vault: true
    - no_pending_escalation: true
    - ci_green: true
    - sentry_no_incident: true
    - gh_logged_in: true
    - mcp_alive: true
  blast_radius_estimate:
    files: 8
    loc_estimated: 200
    touches_critical_path: false
```

---

## 进 Phase 1 时的 prompt 注入

主流水线 Phase 1 看到 00-intake.md 后，prompt 应注入：

```
本次需求由 autopilot 自动选择（不是 PM 主动提）。
请按 00-intake.md 的"autopilot 推测的 PM 会说的话"作为需求初稿，
跑 Phase 1 澄清流程：
- 列出假设
- 反问 PM 关键信息
- 调用 brainstorming 把隐藏假设挖出来

注意：autopilot 已估算 size=<X>，blast_radius=<Y>。
若 Phase 1 澄清后发现 size 实际更大，触发 R3 升级（size mismatch）。
```
```
