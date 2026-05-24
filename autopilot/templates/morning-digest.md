# Autopilot 早晨 Digest 模板

> 每次主流水线跑完 Phase 12（autopilot-triggered 迭代），写到 `~/.autopilot/morning-digest-<date>.md`。
> 这是 autopilot 对 PM 的"昨晚做了什么"报告（与 Phase 12.5 早晨复盘配合）。

---

```markdown
# Autopilot Morning Digest — <YYYY-MM-DD>

> 您昨晚在睡觉时, autopilot 自动完成了 [N] 次迭代。

---

## 昨晚做了什么

### 迭代 1：candidate-roadmap-042 "给 AI 选品加二阶反馈"

- **启动**: 2026-05-22 23:00（autopilot trigger）
- **完成**: 2026-05-23 04:30（5.5h）
- **改动**: 12 文件 / +387 -94 行
- **自治决策**: 14 条（其中 ⚠️ 1 条需 PM 看）
- **GAN 调用**: 6 个 phase 跑 GAN，平均 2.5 轮 PASS，1 次 PIVOT
- **测试**: 单测 +12 / e2e +3 / 覆盖率 84%
- **验收**: 五层 5/5 ✅
- **真人验收**: 4.2/5（5 用户测试）
- **Release PR**: https://github.com/.../pull/143

---

## ⚠️ 需要 PM 看的（去 Phase 12.5 处理）

1. **autonomous-decisions.md#decision-7**: 因为 vendor lock-in 边界，选了 lib-A 而不是 lib-B。理由站得住但 PM 需知情。
2. **Phase 10.5 邀请测试**: 3 个用户邮件已草拟在 vault/10.5-invite-template.md，PM 看完发即可。

---

## 今天 (Tier 1 决策待您做)

### autopilot 今日拟选: candidate-sentry-099

"checkout flow TypeError"（昨天 Sentry 新发现）

- 来源: Sentry PROJ-9876
- score: 2.95
- 大小: S
- 估算: 2-3h autonomous

**今晚跑这个？**
- ✅ 跑：「autopilot pick #1 启动」
- 🔄 改：「跑 #N」（见 [完整 queue]）
- ⏸ 不决定：今晚不跑

---

## 状态: AWAITING_MORNING_REVIEW

您接下来：

### 立即可做

1. **昨晚迭代的 Phase 12.5 复盘**（推荐先做）
   - 输入：`/morning` 或"复盘"
   - 看 4 步清单 → 4 选项决定

2. **暂停 autopilot**（如不想今晚跑）
   - `/autopilot-pause`

3. **批准今日 pick**（如已看完想跑）
   - 「autopilot pick #1 启动」

### 后续可做

- `/autopilot-status` 查全部状态
- `/autopilot-tier 2` 升 Tier（小 bug 自动跑）
- 修改 roadmap.md 加新候选

---

## 整体统计（最近 30 天）

- 总 autopilot 迭代: 18 次
- 成功 merge: 14 次（78%）
- 触发红线: 2 次（11%，PM 决策推迟或缩范围）
- INFRA error: 0 次
- 平均迭代时长: 4.8h
- 平均 PM 介入: 8 分钟/天

---

**生成时间**: <YYYY-MM-DD HH:mm:ss>
**生成器**: autopilot v4
```
