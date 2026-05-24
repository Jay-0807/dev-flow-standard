# PIVOT Handler — 卡死时强制重写机制

> **作用**：GAN 引擎检测到 generator "卡在错误方向"时，触发 PIVOT 强制从零重写而非继续 patch。
> **来源**：直接学 autodev-polish gan-loop step 03 + autodev-review 的 pivot 机制（autodev 实证：连续 2 轮无提升 = 方向错了，patch 改不动）。

---

## PIVOT 触发条件

连续 2 轮 reviewer verdict 都是 NEEDS_IMPROVEMENT **且总分无提升甚至倒退**：

```
Round N-2 总分: 25
Round N-1 总分: 24
Round N 总分:   23

→ 触发 PIVOT
```

或：

```
Round N-1 总分: 25
Round N 总分:   25（持平 = 卡死，不算提升）

→ 触发 PIVOT
```

或：

```
Round N-1 总分: 26
Round N 总分:   24（倒退）

→ 触发 PIVOT
```

不触发的反例：

```
Round N-1 总分: 23
Round N 总分:   27（提升 4 分）

→ 不触发 PIVOT，继续 N+1 轮
```

---

## PIVOT 执行步骤

### Step 1：stash 旧版

把当前 GAN 输出目录里的全部 round-N 子目录复制到 `pivot-archive/pre-pivot-N/`：

```
iteration-vault/<phase>-gan/
├── round-1/         ← stash
├── round-2/         ← stash
├── round-3/         ← stash（触发 PIVOT 时的 N）
├── pivot-archive/
│   └── pre-pivot-3/
│       ├── round-1/ ← 拷贝
│       ├── round-2/ ← 拷贝
│       └── round-3/ ← 拷贝
└── trace.md         ← 追加一段说明
```

### Step 2：清理 round-* 但保留 trace.md

把 round-1/2/3 目录从主路径删除（已 stash 到 archive 里）。保留 trace.md 持续追加。

trace.md 追加段：

```
## PIVOT @ Round {{N}}

触发原因：连续 {{X}} 轮 NEEDS_IMPROVEMENT 且总分趋势 {{flat/declining}}：
- Round {{N-2}}: 总分 {{X}}
- Round {{N-1}}: 总分 {{Y}}
- Round {{N}}:   总分 {{Z}}

stash 路径：pivot-archive/pre-pivot-{{N}}/

generator 接下来从零重写。
```

### Step 3：Generator 重写

spawn Generator 时用 PIVOT 版 prompt（见 `generator.md` 的 "PIVOT 后第一轮" 部分），核心：

- 不读 stash 的 gen.md（避免被旧思路污染）
- 读所有 stash 的 scorecard（理解为什么旧版方向是错的）
- 重新规划方向
- 输出新的 round-{{N+1}}/gen.md，含"PIVOT 后的方向变化"段

### Step 4：N 计数器规则

PIVOT 本身**不重置** N 计数器（避免无限 PIVOT）。继续从 round-{{N+1}} 开始。

例：触发 PIVOT 时 N=3（已跑 3 轮），PIVOT 后下一轮是 round-4，全局 N 上限仍是 5。所以 PIVOT 后最多还有 2 轮（round-4 和 round-5）。

如果 round-4 / round-5 又卡死（再次满足 PIVOT 条件），**不再触发第二次 PIVOT**（防止无限循环），直接标 `gan_result: needs_improvement` 用最高分版本作 final。

---

## PIVOT 与 N 上限的硬约束

| N | 状态 | 行动 |
|---|---|---|
| 1 | NEEDS_IMPROVEMENT | 进 round-2 |
| 2 | NEEDS_IMPROVEMENT 且无提升 | 还没到 2 轮无提升标准，继续 round-3 |
| 3 | NEEDS_IMPROVEMENT 且 N-1→N 无提升 | 触发 PIVOT，stash round-1/2/3，进 round-4 重写 |
| 4 | PASS | 用 round-4 作 final ✅ |
| 4 | NEEDS_IMPROVEMENT | 进 round-5 |
| 5 | PASS | 用 round-5 作 final ✅ |
| 5 | NEEDS_IMPROVEMENT | **不再 PIVOT**，取所有 round 总分最高的版本作 final + 标 needs_improvement |

---

## PIVOT 后的 generator prompt 额外指令

PIVOT generator prompt 必含：

```
**PIVOT 重写 — 关键指令**：

1. 不要读 pivot-archive 里的 gen.md（避免被旧思路污染）。可以读 scorecard.md 理解为什么旧方向错。

2. 旧方向核心问题（来自 scorecard 总结）：
   - {{problem_1}}
   - {{problem_2}}
   - {{problem_3}}

3. 你的任务：用**完全不同的方法**重新生成，不要"在旧版基础上改"。

4. 写完后在 gen.md 末尾必加一段：
   ```
   ## PIVOT 后的方向变化
   - 旧方向：<一句话总结 stash 的方向>
   - 新方向：<一句话总结本次新方向>
   - 突破点：<为什么这次能 work>
   ```

5. 如果新方向看起来又会陷入旧问题，先**push back**——在 gen.md 顶部写"⚠️ 我感觉再换方向也可能卡住，建议 PM 介入重审 task_input"，然后**仍尽力生成最好的产出**。
```

---

## 当 PIVOT 不触发但 generator 持续低分时

不是所有"低分" = PIVOT。有些情况是 generator 持续小步提升，只是没到 PASS：

```
Round 1: 总分 18
Round 2: 总分 22
Round 3: 总分 24（仍 NEEDS_IMPROVEMENT 但提升了）
Round 4: 总分 26（仍未 PASS 但有提升）
Round 5: 总分 27（用尽 N=5 但未 PASS）

→ 不触发 PIVOT（每轮都有提升）
→ N=5 用尽，取 round-5（最高分）作 final，标 needs_improvement
```

PIVOT 是"卡死救援"，不是"超时救援"。两者要分清。

---

## PIVOT 触发的边界 case

### Case 1：第 2 轮就总分倒退
```
Round 1: 总分 25
Round 2: 总分 22（倒退 3 分）
```
不触发 PIVOT。PIVOT 需要"连续 2 轮无提升"——本例只是 1 轮回退，可能是 reviewer 不一致。给 round-3 一个机会。

### Case 2：分数微动但 critical redline 一直触发
```
Round 1: 总分 24，R2 命中
Round 2: 总分 24，R2 仍命中
```
不触发 PIVOT（总分持平不算无提升的标准触发，但这种情况 reviewer 应该 verdict = FAIL 而不是 NEEDS_IMPROVEMENT；如果 reviewer 给了 NEEDS_IMPROVEMENT，GAN 引擎应该强制 verdict = FAIL）。

### Case 3：某一维提升但其他维下降
```
Round 1: dim1=8, dim2=5, dim3=7, dim4=6 = 26
Round 2: dim1=8, dim2=8, dim3=4, dim4=6 = 26（持平）
```
持平 + 某维倒退 = 触发 PIVOT（因为 generator 在"按下葫芦浮起瓢"，方向有问题）。

---

## 与 trace.md 的联动

trace.md 完整结构（含 PIVOT 时）：

```
# GAN Trace for {{task_type}} @ {{phase}}

start_at: 2026-05-23T22:00:00+08:00
N_max: 5

## Round 1
- generator spawn at: ...
- reviewer spawn at: ...
- verdict: NEEDS_IMPROVEMENT
- 总分: 18
- 关键问题: <一句话>

## Round 2
- ... verdict: NEEDS_IMPROVEMENT 总分: 19

## Round 3
- ... verdict: NEEDS_IMPROVEMENT 总分: 17（倒退）

## PIVOT @ Round 3
- 触发原因: 连续 2 轮无提升 + 倒退
- stash 路径: pivot-archive/pre-pivot-3/
- generator 收到的旧方向问题：<3 条>

## Round 4 (post-PIVOT)
- generator spawn ... 新方向: <一句话>
- reviewer spawn ... verdict: PASS
- 总分: 30

## Final
end_at: 2026-05-23T22:14:00+08:00
total_rounds: 4 (含 PIVOT 后 1 轮)
total_calls: 8 (4 round × 2 agent)
gan_result: pass
pivot_triggered: true
final_path: final.md
```

---

## 维护备忘

- 每次跑出"PIVOT 后又 FAIL"的情况，分析 generator prompt 是否需要更多上下文
- PIVOT 频率太高（>30% 任务）→ generator 起始 prompt 质量有问题，需要调整
- PIVOT 频率太低（<5% 任务）→ reviewer 太宽松，需要调严
