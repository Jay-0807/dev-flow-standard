# Autopilot Escalation 模板

> autopilot-triggered 迭代中触发 R1-R4 红线时，按本模板写 escalation 报告。
> 与普通 ESCALATION-R[X].md 区别：autopilot 上下文需额外说明"为什么选这个候选"。

---

```markdown
# 🚨 ESCALATION: R<X> (Autopilot-Triggered) <简短标题>

**触发时间**: <YYYY-MM-DD HH:mm>
**触发 phase**: <Phase ID>
**触发 batch**: <如分批跑>
**当前 candidate**: <candidate_id>
**autopilot tier**: <1/2/3>

---

## autopilot 的选题背景（与普通 escalation 的区别）

> 本次迭代不是 PM 主动提的，是 autopilot 自动从 backlog 选的。

- **选这个候选的理由**: 
  - 来源: <source>
  - 优先级 score: <X.XX>
  - autopilot 推测的"PM 会说的话": <quote>
- **selection 决策痕迹**: queue.md 当时显示 Top 1
- **Tier 行为**:
  - Tier 1: PM 当时已经 ✅ 批准过 candidate 启动
  - Tier 2: 自动启动（小 candidate）/ PM 批（大 candidate）
  - Tier 3: 完全自动启动

---

## 问题描述

[2-3 段：发生了什么，为什么不能 autonomous 决策]

---

## 候选方案

- 方案 A：[描述 + 工时 + 风险]
- 方案 B：...
- 方案 C：...

---

## skill 的倾向（如有）

[基于保守默认决策树，倾向 A，但需要 PM 确认]

---

## 影响

- **如果选 A**: [影响]
- **如果选 B**: [影响]
- **如果选 C**: [影响]
- **如果选放弃此候选**: 
  - vault 归档到 `history/AUTOPILOT-ABANDONED-<timestamp>/`
  - candidate 进 blacklist X 天（PM 决定）
  - autopilot 下次 cron 选 Top 2

---

## autopilot 特殊选项

PM 还可选：

1. **撤销 autopilot 这次的选题**（candidate 选错了）
   - 命令: `/autopilot-rollback`
   - 行为: vault 归档 + candidate 进 blacklist + 调整评分算法（如果是 systemic 问题）

2. **降级 autopilot Tier**（信任度降）
   - 命令: `/autopilot-tier <lower>`
   - 行为: 后续 autopilot 选题更保守

3. **拉黑这个 candidate**（不是 systemic 问题，单次错）
   - 命令: `/autopilot-blacklist <candidate_id>`
   - 行为: 该 candidate 一段时间内不再选

---

## PM 需要做什么

1. Read 本文件完整内容
2. Read `iteration-vault/autonomous-decisions.md` 看前面决策
3. Read `~/.autopilot/queue.md` 看选题历史
4. 选一个候选方案 或 选 autopilot 特殊选项 → skill 续跑

---

## 等待 PM 决策

state.yaml mode = escalated
autopilot state.json state = R<X>_ESCALATED
```

---

## 维护备忘

- 每次 autopilot escalation 后总结：是否 candidate 选错？是否评分算法需要调？
- 如某类 escalation 反复出现（如 R3 推迟 > 3 次）→ autopilot 评分降权该 source
