# GAN 引擎 — 对抗式生成主入口

> **来源**：综合 PM 反馈 + `Leooo-Huang/autodev-skills` 项目实证（autodev-review / autodev-polish / autodev-verify）+ Karpathy 4 原则。
> **用法**：本 skill 任何 phase 在需要"高质量生成"时调用本引擎完成单段任务（写 PRD / 设计 API / 实施代码任务 / 起草 release notes 等）。Phase 文件用 `## GAN 钩子` 段声明调用。
> **项目适配**：reviewer prompt 注入 r4 哲学（1-3 维）+ 项目业务协议（如有）（如适用）+ Karpathy 4 原则 + 7 Quality Redlines。

---

## 角色定义

你现在扮演一个 **GAN 编排器**。你的职责：把"一段需要高质量产出的任务"按对抗式生成方法跑到 PASS。你的关键品质：

1. **保守判定**：reviewer 默认怀疑，不轻易给 PASS（autodev 实证：怀疑型 reviewer 才能抓出 90% 边界 case）
2. **预算意识**：N ≤ 5 轮硬上限，收敛早退优先，PIVOT 是最后手段
3. **不阻塞下游**：跑满 5 轮仍未 PASS 不阻塞，标 `gan_result: needs_improvement` 留 Phase 9 Global GAN 兜底
4. **痕迹完整**：每轮 gen + critique 留档，PM 可追溯每个决定

---

## 调用契约

任何 phase 都按此格式调用：

```
GAN_INVOKE(
  task_type: <见 role-router.md 任务类型>,
  task_input: <本任务的具体上下文/输入文件路径列表>,
  output_path: iteration-vault/<phase>-gan/,
  N: 5  # 默认上限
) → final.md + scorecard.md + trace.md
```

**契约保证**：
- 返回的 `final.md` 是 PASS 版本（最后一 round PASS 时的 generator 输出）
- 若跑满 5 轮未 PASS，返回最后一 round + `gan_result: needs_improvement` 标记
- 触发 PIVOT 重置轮数，但仍受 N=5 总上限约束（含 pre-PIVOT 轮数 + PIVOT 后轮数）

---

## 编排流程（标准 5 步）

### Step 1：初始化
- 读 `gan-engine/role-router.md`，按 `task_type` 解析出：
  - 4 维打分配置（哪 4 个维度）
  - 视角段（要在 reviewer prompt 注入哪些文件）
- 创建 `<output_path>/round-1/` 目录
- 写 `trace.md` 头：`task_type / N / 开始时间`

### Step 2：Round N 循环
对每个 round（n=1 到 N）：

1. **Generator spawn**：
   - 调用 `gan-engine/generator.md` 的 prompt 模板
   - 输入：task_input + 前一轮的 critique（n>1 时）
   - 输出：`<output_path>/round-n/gen.md`

2. **Reviewer spawn**：
   - 调用 `gan-engine/reviewer-skeptical.md` 的 prompt 模板
   - 注入：当 round 的 gen.md + 视角段文件们 + `quality-redlines.md`
   - 输出：`<output_path>/round-n/scorecard.md`（4 维打分 + 7 redlines 检查 + verdict）

3. **判定 verdict**：
   - PASS ✅：4 维全 ≥7 + 无 critical redline + 无未解决项 → **跳到 Step 4 合成 final**
   - FAIL ❌：任一维 <5 OR 命中 critical redline → **本轮不计入 N**，generator 重做（最多 3 次内重做不计入 N，防止"FAIL 雪崩"占满 N=5 上限）
   - NEEDS_IMPROVEMENT 🟡：5 ≤ 任一维 <7 OR 仅 major 项 → **进入下一 round**

4. **PIVOT 检测**：
   - 连续 2 轮 NEEDS_IMPROVEMENT 且**总分无提升**（甚至下降）→ 触发 PIVOT，进入 Step 3

### Step 3：PIVOT 处理（仅触发时）
按 `gan-engine/pivot-handler.md`：
- 把 round-{n-2..n} 移到 `<output_path>/pivot-archive/`
- Generator 不再 patch 旧版，**从零重写**：
  - 读 task_input
  - 读全部 critique 历史（含被 pivot 的）
  - 重新规划方向
- 轮数计数器**不重置**（仍受 N=5 总上限）
- 继续 Step 2 循环

### Step 4：合成 final
- 把最后一轮 `gen.md` 复制为 `<output_path>/final.md`
- 写 `<output_path>/trace.md` 末段：跑了 N 轮 / PASS 在 round-X / 触发 PIVOT 否

### Step 5：返回
- final.md 路径
- gan_result: pass | needs_improvement
- 调用方（phase）根据 gan_result 决定后续：
  - pass → 继续下一 phase
  - needs_improvement → 标记 phase 输出"待审"，留 Phase 9 兜底

---

## 4 状态判定速查

| 状态 | 触发 | 行动 | 计入 N？ |
|---|---|---|---|
| **PASS** ✅ | 4 维全 ≥7 AND 无 🔴 critical AND 无未解决 redline | 早退到合成 | - |
| **NEEDS_IMPROVEMENT** 🟡 | 5 ≤ 任一维 <7 OR 仅 🟡 major | 进下一 round | 是 |
| **FAIL** ❌ | 任一维 <5 OR 命中 critical redline | 重做本轮 | 否（最多 3 次内重做不计入）|
| **PIVOT** 🔄 | 连续 2 轮 NEEDS_IMPROVEMENT 且总分无提升 | stash 旧版重写 | 否（PIVOT 本身不计入）|

---

## 关键设计约束

1. **agent 类型上限 = 2**：Generator + Skeptical Reviewer 两个 prompt 模板，每轮各 spawn 一次
2. **agent 调用次数 = (1 + 1) × N**：N=3 时 6 次；N=5 时 10 次；触发 PIVOT 时最坏 ~14 次
3. **wall clock 上限**：单次 GAN 调用 ≤ 30min，超时强制用当前最高分版本作 final + 标记 `gan_result: timeout`
4. **并发**：Generator 和 Reviewer **不并发**（reviewer 必须看 gen 后才能审）；但**不同 phase 的 GAN 调用可并发**
5. **失败 fallback**：subagent 调用挂掉 / 输出格式不对 / 5 轮无收敛 → 降级为"单 generator + Karpathy 4 自检"产出最终，标记 `gan_result: degraded`

---

## 与 phase 文件的契约

每个跑 GAN 的 phase 文件必须含「## GAN 钩子」段：

```markdown
## GAN 钩子

本 phase 的 Step X 调用 GAN 引擎：
- 任务类型：<见 role-router.md 中的 task_type 字符串>
- 输入文件：<本 phase 的 input 文件路径>
- 输出路径：iteration-vault/<phase>-gan/
- 默认 N=5（含早退）
- 失败 fallback：单 gen + Karpathy 自检
- 后续行为：
  - gan_result: pass → 进下一 phase
  - gan_result: needs_improvement → 标记 + 留 Phase 9 兜底
  - gan_result: degraded → autonomous-decisions.md 留痕 + 继续
```

---

## 跑 GAN 的 phase 速查（v4 最终版）

| Phase | 跑 GAN？ | 任务类型字符串 |
|---|---|---|
| 0 大小分级 | ❌ | - |
| 1 澄清 | ❌ | - |
| 1.5 用户研究 | ⚠️ 可选 | `user-research` |
| **2 PRD** | ✅ | `prd-writing` |
| **2.5 brainstorm** | ✅ | `brainstorm-diverge` + `brainstorm-converge` |
| 3 影响面 | ❌ | - |
| **4 架构** | ✅ | `architecture-design` |
| **4.5 API 整理 Step 1** | ✅ | `api-design` |
| 4.5 Step 2 审计 | ❌ | - |
| **5a UX 设计** | ✅ | `ux-flow` |
| 5b UI Spec | ❌ | - |
| 5.9 文档压缩 | ❌ | - |
| **6 任务分解** | ✅ | `task-breakdown` |
| **7 所有 code task** | ✅ | `code-task` (+ `ui-task` add-on) |
| 7 机械任务 | ❌ | - |
| 8 代码债 | ❌ | - |
| 9 三路审查 | ⚠️ 本身即 Global GAN，不嵌套 | - |
| 10 验收 | ❌ | - |
| 10.5 用户验收 | ❌ | - |
| **11 release notes** | ✅ | `release-notes` |
| 11.5 漂移 | ❌ | - |
| 12 git release | ❌ | - |
| 12.5 早晨复盘 | ❌ | - |
| **Autopilot 种子 PRD** | ✅ | `autopilot-seed-prd` |
| 决策回放分析 | ⚠️ 可选 | `decision-replay-analysis` |

---

## 产出归档结构

```
iteration-vault/<phase>-gan/
├── round-1/
│   ├── gen.md          # generator 输出
│   └── scorecard.md    # reviewer 4 维 + 7 redlines + verdict
├── round-2/ ... round-N/
├── pivot-archive/      # 仅触发 PIVOT 时有
│   └── pre-pivot-vN/   # 被 stash 的旧版完整目录拷贝
├── final.md            # PASS 时是最终；needs_improvement 时是最后一轮
└── trace.md            # 跑了几轮 + verdicts + PIVOT 否 + 总用时
```

PM 默认只看 `final.md`；想深挖时看 `trace.md` 决定打开哪个 round。

---

## 各种失败模式 & 兜底

| 失败 | 兜底 |
|---|---|
| subagent spawn 挂 | 重试 1 次；2 次都挂 → 降级单 gen + Karpathy 自检 |
| Reviewer 输出格式不对（无 4 维打分）| 重新 spawn 一次；2 次都不对 → 用 round-1 gen 作 final + 标 degraded |
| Generator 5 轮都不 PASS | 用最高分轮版本作 final + 标 needs_improvement |
| PIVOT 后仍卡住 | N=5 总上限到了停 + 标 needs_improvement |
| wall clock > 30min | 强制取当前最高分版本 + 标 timeout |
| 跑 GAN 时 PM 主动停 | 用当前最高分版本作 final + 标 user_aborted |

所有降级都写入 `iteration-vault/autonomous-decisions.md` 留痕。

---

## 反模式（必拒绝）

- ❌ Generator 写 placeholder / TODO / mock 等内容（命中 quality-redlines）
- ❌ Reviewer 给"看起来 OK 就 PASS"（必须严格按 4 维 + 7 redlines）
- ❌ 5 轮没收敛就硬续跑（必须停 + 标 needs_improvement）
- ❌ PIVOT 时复用旧版思路（必须真的从零重写）
- ❌ 跨 phase 串行 spawn agent（不同 phase 的 GAN 应并发）
- ❌ 跳过 trace.md 不留痕（PM 失去审计能力）

---

## 维护备忘

- 每次跑完一次完整迭代，从各 phase 的 `<phase>-gan/trace.md` 提取"哪类任务 PIVOT 多 / 哪类任务 1 轮 PASS"，沉淀到 `role-router.md` 的"任务类型经验"段
- 每次发现 reviewer 老在某维度卡死，检查角色路由是否正确
- 每次 7 Quality Redlines 更新，同步 `quality-redlines.md`
