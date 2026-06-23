# Autopilot Queue 模板

> Autopilot 每次唤醒后重写 `~/.autopilot/queue.md`，按本模板。

---

```markdown
# Autopilot Queue — <YYYY-MM-DD HH:mm>

> **本次扫描**: [N] 个 connector / 共 [M] 候选 / 去重黑名单后 [K] / 排序后呈现 Top [L]
> **当前 Tier**: [1/2/3]
> **安全闸状态**: 全通过 / 部分失效（[列表]）

---

## Top 10 (按 final_score 降序)

| 排名 | 来源 | 标题 | score | 大小 | 信号 | 证据 |
|---|---|---|---|---|---|---|
| 1 | roadmap | 给导出功能加二阶反馈 | 3.42 | M | impact=5/freq=3/age=21d/conf=1.0 | [link] |
| 2 | sentry | TypeError in checkout flow | 2.95 | S | impact=4/freq=4/age=7d/conf=0.95 | [link] |
| 3 | user-feedback | 导出页加载慢 | 2.78 | M | freq=4(4 轮重复)/age=42d | [link] |
| 4 | github-issues | 后台搜索响应太慢 | 2.65 | M | impact=4/freq=4/age=7d | [link] |
| 5 | notion | 暗黑模式 | 2.10 | M | impact=3/freq=3/age=30d | [link] |
| 6 | ... | ... | ... | ... | ... | ... |
| 10 | ... | ... | ... | ... | ... | ... |

---

## 本轮 autopilot 拟选：#1

### 候选详情

- **ID**: candidate-roadmap-042
- **标题**: 给导出功能加二阶反馈
- **来源**: ~/.autopilot-data/product-roadmap.md L5
- **优先级 score**: 3.42（Top 1）

### 信号

- 商业影响: 5/5（PM 显式标 P0）
- 用户疼痛: 3/5（默认；roadmap 无重复频次信息）
- 复杂度: M
- age: 21 天
- confidence: 1.0（PM 显式写）

### 估算

- size: medium
- 工时估算: 5-7h autonomous + PM PRD review ≤ 10min
- blast_radius_hint: 8 文件 / 不触及 critical path

---

## 安全闸预检

| 安全闸 | 状态 | 备注 |
|---|---|---|
| L1 Kill Switch | ✅ | PAUSE/EMERGENCY_STOP 文件不存在 |
| L2 Budget | ✅ | 0/1 daily, 2/5 weekly |
| L3 Pre-flight | ✅ | 全部 8 项通过 |
| L4 Blast Radius | ✅ | 8 文件 < 30，不触 forbidden_paths |
| L5 Working Hours | ✅ | 当前 09:00，跑时段 21:00-04:00 |
| L6 Consecutive Failures | ✅ | 最近 0/2 失败 |

**总结**：可启动。Tier 1 模式下需 PM 显式 ✅。

---

## Tier 模式下的行为

**Tier 1**（当前）：
- PM 必须显式说"autopilot pick #1 启动"才跑
- 否则一直等

**Tier 2**：
- 如本候选 size=small + score > 2.5 + 不触 critical path → 自动启动今晚
- 本例 size=M → 仍需 PM 批

**Tier 3**：
- 全自动启动今晚
- PM 仅看早晨结果

---

## 历史已 done (近 7 天)

| 日期 | 候选 | 结果 | PR |
|---|---|---|---|
| 2026-05-22 | candidate-roadmap-040 | ✅ done | #142 (merged) |
| 2026-05-21 | candidate-sentry-PROJ-1234 | ⚠️ R3 推迟 | - |
| 2026-05-20 | candidate-roadmap-039 | ✅ done | #140 (merged) |
| 2026-05-19 | candidate-feedback-7d4e9a3c | ✅ done | #139 (merged) |

---

## 当前黑名单（近 30 天加的）

- candidate-sentry-9999 (PM 标"已知误报", 永久)
- candidate-github-50 (PM 标"等三方依赖", until 2026-12-31)

---

## 待 PM 决策

**今晚是否跑 #1？**

- ✅ 跑：「autopilot pick #1 启动」
- 🔄 改：「跑 #N」（指定其他候选）
- ❌ 拒：「拉黑 #1，理由 X」
- ⏸ 不决定：今晚不跑

下次 cron tick：明天 09:00
```
