# GAN 输出目录结构模板

> **作用**：定义 GAN 引擎每次调用产出在 iteration-vault 里的标准结构。Phase 调用 GAN 时应按此模板创建目录。

---

## 标准结构

```
iteration-vault/<phase-id>-gan/
├── round-1/
│   ├── gen.md           # generator 输出
│   └── scorecard.md     # reviewer 4 维打分 + 7 redlines 检查 + verdict
├── round-2/
│   ├── gen.md
│   └── scorecard.md
├── round-3/
│   └── ...
├── round-N/             # 最多 5
│
├── pivot-archive/       # 仅触发 PIVOT 时存在
│   └── pre-pivot-N/     # PIVOT 触发时的轮数 N
│       ├── round-1/     # stash 的旧 round
│       ├── round-2/
│       └── round-N/
│
├── final.md             # 最终产出（PASS 时是 PASS 轮的 gen.md；needs_improvement 时是最高分轮）
└── trace.md             # GAN 决策痕迹
```

---

## 命名约定

- `<phase-id>` 用 phase 数字 + 简短标识，例如：
  - `02-prd-gan/`
  - `02.5-brainstorm-diverge-gan/`
  - `02.5-brainstorm-converge-gan/`
  - `04-architecture-gan/`
  - `04-architecture-and-api-gan/`
  - `05-gan/`
  - `06-task-breakdown-gan/`
  - `07-code-task-<task-id>-gan/`  ← Phase 7 每个 task 一个目录
  - `11-release-notes-gan/`
  - `autopilot-seed-prd-<candidate-id>-gan/`

- Phase 7 含 UI Add-on 时，UI 部分另起子 GAN：
  - `07-code-task-<id>-ui-gan/`

---

## round-N/gen.md 标准头

```markdown
<!--
task_type: <见 role-router.md>
round: <1-5>
generator_version: 1
phase_id: <如 04 或 05>
-->

# <任务标题>

<正文内容>

---

## Generator 自检
- [ ] 4 维打分维度都覆盖了
- [ ] 7 Quality Redlines 无命中
- [ ] Karpathy 4 原则自检通过
- [ ] 任务专属指令都落地
```

---

## round-N/scorecard.md 标准结构

```markdown
<!--
task_type: <...>
round: <1-5>
reviewer_version: 1
-->

# Round N Reviewer Scorecard

## Quality Redlines 检查
| Redline | 状态 | 违规清单 |
|---|---|---|
| R1 | ✅/⚠️/❌ | ... |
| ... | ... | ... |

**critical redlines hit**: <列表或"无">

## 4 维打分
| 维度 | 分数 | 理由 |
|---|---|---|
| <dim1> | X/10 | <一句话> |
| <dim2> | X/10 | <一句话> |
| <dim3> | X/10 | <一句话> |
| <dim4> | X/10 | <一句话> |

**总分**: X/40

## 视角审查
### <perspective 1>
<观察>

### <perspective 2>
<观察>

## Verdict
**PASS / FAIL / NEEDS_IMPROVEMENT**

## 给下一轮的具体改进建议
1. <具体改进 1>: <怎么改>
2. <具体改进 2>: <怎么改>

## 严重度标注
- 🔴 critical: <列表>
- 🟡 major: <列表>
- 🟢 minor: <列表>
```

---

## final.md 头

```markdown
<!--
gan_result: pass | needs_improvement | degraded | timeout | user_aborted
final_round: <从哪轮取的>
total_rounds: <跑了几轮>
pivot_triggered: true/false
task_type: <...>
phase_id: <...>
-->

# <任务标题> (Final)

<最终产出内容>
```

---

## trace.md 结构

```markdown
# GAN Trace: <phase_id> <task_type>

## 元信息
- start_at: <iso8601>
- end_at: <iso8601>
- total_duration_min: <分钟>
- task_type: <见 role-router.md>
- N_max: 5
- total_rounds: <实际跑了几轮>
- total_agent_calls: <round × 2 = ?>
- pivot_triggered: <true/false>
- gan_result: <pass | needs_improvement | degraded | timeout | user_aborted>

## 每轮摘要
### Round 1
- generator spawn at: <ts>
- reviewer spawn at: <ts>
- verdict: <PASS / FAIL / NEEDS_IMPROVEMENT>
- 总分: X/40
- 关键问题: <一句话>

### Round 2
...

## PIVOT 记录（如有）
- 触发轮: Round N
- 触发原因: <连续 2 轮无提升 + 倒退/持平>
- stash 路径: pivot-archive/pre-pivot-N/
- generator 收到的旧方向问题: <3 条>

## 最终
- 取自 round-X 作 final
- gan_result: <见元信息>
- 总成本估算: <agent 调用数 × 平均 token>
```

---

## PM 视角的 vault 浏览路径

PM 一般只需要看 `final.md`。

但 PM 在 Phase 12.5 早晨复盘时可能想：
1. 看 `trace.md` 摘要 → 决定要不要深挖
2. 深挖时打开 `round-N/scorecard.md` 看 reviewer 怎么判的
3. 必要时打开 `round-N/gen.md` 看原始 generator 产出
4. PIVOT 时打开 `pivot-archive/pre-pivot-N/` 看旧版

所以 final.md / trace.md 是"快速面"，round-* 是"深挖面"。

---

## 跨迭代清理策略

每次 iteration 完成归档到 `iteration-vault/history/<timestamp>-<feature>/` 时，**整个 gan/ 目录原样保留**（不删 round-*）。这是为了：

- PM 后续可追溯任何 GAN 决策
- 跨迭代沉淀经验（哪类任务 PIVOT 多 / 哪类 1 轮 PASS）
- 检查 GAN 引擎本身是否需要调整

如果磁盘空间紧张（罕见），可以在归档时只保留 `final.md + trace.md`，删 `round-*` 和 `pivot-archive/`。这是 trade-off，默认不做。

---

## 维护备忘

- 每次新增 phase 跑 GAN，确保 phase 文件用本模板的目录命名
- 如果发现 trace.md 字段不够用（如想追加成本统计），同步本模板
