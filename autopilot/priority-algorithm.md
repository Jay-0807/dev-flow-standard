# Autopilot 优先级算法

> **作用**：autopilot 从 N 个 connector 拉到候选后，按本算法排序，输出 ranked queue.md。
> **v4 默认**：固定公式（fixed_formula），4 周后视情况切到 hybrid。

---

## 统一候选格式

每个 connector 输出 candidate 列表，每个 candidate 字段（见 `connectors/README.md`）：

```yaml
- id: <stable-uuid>            # 跨轮稳定，用于黑名单 / cooldown
  source: sentry|github|user-feedback|notion|roadmap
  source_ref: <url-or-path>
  title: <≤60 字>
  description: <≤300 字>
  raw_evidence: <full-text>
  priority_signals:
    business_impact: 1-5
    user_pain_freq: 1-5
    technical_complexity: S|M|L
    age_days: <int>
    confidence: 0-1
  estimated_size: small|medium|large
  blast_radius_hint:
    affected_files_guess: <int>
    touches_critical_path: bool
  tags: [bug, enhancement, perf, security, ...]
  created_at: <iso8601>
```

---

## 固定公式（v4 默认）

```
final_score = (
    w_impact * normalized(business_impact, 1-5)          # 0.30
  + w_freq   * normalized(user_pain_freq, 1-5)           # 0.25
  + w_age    * age_boost(age_days)                        # 0.15
  + w_conf   * confidence                                 # 0.10
  - w_cplx   * complexity_penalty(estimated_size)         # 0.20
) * source_weight                                         # connector 自带权重
```

**`age_boost(days)`**：
```
min(1, log2(days+1) / log2(60))
```

线性老化太慢，对数老化让 30 天的项也有提升但 60 天后封顶。

**`complexity_penalty(size)`**：
```
S → 0
M → 0.5
L → 1.0
```

**`normalized(x, 1-5)`**：
```
(x - 1) / 4   # 把 1-5 映射到 0-1
```

---

## 权重表（PM 可在 config.yaml 调）

```yaml
priority_algorithm:
  weights:
    business_impact: 0.30
    user_pain_freq: 0.25
    age_boost: 0.15
    confidence: 0.10
    complexity_penalty: 0.20
  age_cap_days: 60
```

加起来不一定 = 1（complexity 是负权重）。

---

## source_weight（各 connector 的加权）

```yaml
connectors:
  roadmap:       { weight: 2.0 }   # PM 显式写的，加权
  user-feedback: { weight: 1.5 }
  sentry:        { weight: 1.0 }
  github-issues: { weight: 1.0 }
  notion:        { weight: 1.0 }
```

---

## 排序输出

按 final_score 降序，输出到 `~/.autopilot/queue.md`（用 `templates/queue.md`）。

Top 1 作为本轮 autopilot 拟选；Top 2-10 留给 PM 备选。

---

## 升级路径

### Phase 1: 固定公式（v4 起步）

优点：可复现、可调试、PM 看得懂。
缺点：僵化，不能识别"虽然 impact 低但战略重要"的项。

### Phase 2: 混合（v4 + 4 周后视情况）

```
Step 1: 固定公式出 Top 10
Step 2: LLM 在 Top 10 里调整顺序（解释每条调整的理由写入 queue.md）
Step 3: PM 在 INBOX 看到时同时显示固定公式排序 + LLM 调整后排序，看是否一致
```

LLM prompt 模板：

```
你是 项目产品经理顾问。下面是固定公式给出的 Top 10 候选，按分数降序：

{{candidate_list}}

请综合下面因素重新排序：
- 隐形信息显性化对齐加分（与"业务规则显性化 / 人工保留点 / 数据来源标注"对齐的候选加分）
- 用户研究历史（iteration-vault/history/*/01.5）的累积反馈
- 战略协同（多个候选可能合并做更高效）

输出新排序 + 每条变动的解释（如果有）。
```

### Phase 3: LLM-judged 主导（长期）

完全 LLM 判分，固定公式作为 sanity check（防止 LLM 给"明显错的排序"）。

---

## queue.md 输出结构

见 `templates/queue.md`。核心段：

```markdown
# Autopilot Queue — 2026-05-23 09:00

> 本次扫描: 5 个 connector / 共 27 候选 / 去重黑名单后 19 / 排序后呈现 Top 10

## Top 10 (按 score 降序)

| 排名 | 来源 | 标题 | score | 大小 | 信号 | 证据 |
|---|---|---|---|---|---|---|
| 1 | roadmap | 给后台加二阶反馈建议模块 | 3.42 | M | impact=5/age=21d/conf=1.0 | roadmap.md#L42 |
| 2 | sentry | TypeError in core processing flow | 2.95 | S | impact=4/freq=5/age=7d | sentry url |
| ...

## 本轮 autopilot 拟选：#1

## 安全闸预检
- blast_radius 估算: 8 文件 / 不触及 critical path → ✅
- 估算大小: M → Tier 2 模式下需 PM 在 Phase 2 PRD 关卡显式确认
- 估算工时: 3-5h autonomous + PM PRD review ≤ 10min

## 历史已 done (近 7 天)
- 2026-05-22: #candidate-abc done (release PR #142)
- 2026-05-21: #candidate-xyz failed (R3 escalation, PM 决策推迟)
```

---

## 调试模式

PM 可用 `/autopilot-debug <候选 ID>` 查看某候选的评分细节：

```
Candidate: candidate-roadmap-042 "给后台加二阶反馈建议模块"

Score breakdown:
- business_impact: 5 → 1.0 × weight 0.30 = +0.300
- user_pain_freq:  5 → 1.0 × weight 0.25 = +0.250
- age_boost (21d): log2(22)/log2(60) = 0.755 × weight 0.15 = +0.113
- confidence: 1.0 × weight 0.10 = +0.100
- complexity_penalty (M): 0.5 × weight 0.20 = -0.100

Subtotal: 0.663
Source weight (roadmap): 2.0
Final score: 0.663 × 2.0 = 1.326... × normalization → 3.42
```

---

## 维护备忘

- 每次 autopilot 跑出"显然错的排序"，PM 反馈，调权重
- 切到 hybrid mode 前先观察固定公式 4 周稳定性
- LLM-judged 模式开启后必须有 sanity check 防止 LLM 跑偏
