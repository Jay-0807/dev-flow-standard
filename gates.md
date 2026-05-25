# 关卡机制 v4 — 2 个 PM 关卡 + Autonomous + 4 红线 Escalation + 跨夜分批

> **v4 重大变化**：v2 是 1 个关卡（PRD）+ Phase 12 末"完成汇报"。v4 把"完成汇报"升级为正式 PM 关卡 = **Phase 12.5 早晨复盘**（结构化 4 步清单 + 4 选项）。
>
> 设计动机：v2 的"完成汇报"只是一段文字提示，PM 没有显式决策动作；v4 让 PM 在 12.5 有"merge / 局部 redo / 整体重做 / 推下夜"4 选项，且 redo 走决策回放机制。详见 `night-mode.md` + `phases/12.5-morning-review.md`。

---

## 关卡总览（v4）

| 类型 | 触发位置 | 数量 | PM 参与 |
|---|---|---|---|
| ⛳ **PRD 关卡（关卡 1）** | Phase 2 完成 | 1 个 | **必参与** |
| ⛳ **早晨复盘关卡（关卡 2，v4 新增）** | Phase 12.5 | 1 个 | **必参与** |
| ⏸ **batch 间断点（v4 新增，分批跑才有）** | 各 batch 结束 | 0-2 次 | PM 可选介入 |
| 🚨 **红线 Escalation** | Phase 2.5-12 任意点 | 0-N 次 | 触发才参与 |
| 📋 **Autonomous 决策** | Phase 2.5-12 持续 | 15-35 条 | 不参与（12.5 审计）|
| 🔄 **决策回放（v4 新增，可选）** | 12.5 选 🔄 时 | 0-3 次 | PM 触发 + 选方向 |

---

## ⛳ PRD 关卡的标准动作（沿用 v1 设计）

### Step 1：落地文档（c）
把 PRD 写到 `iteration-vault/02-PRD.md`，并通过 docx skill 输出 `.docx` 副本。

### Step 2：对 PM 做摘要
3-5 行中文摘要，告诉 PM：
- 本次做什么
- 关键假设（Karpathy Think Before Coding 第 5.5 段提炼）
- 主要风险
- 通过后预计自治跑多久（如：3-6 小时）

### Step 2.5：Karpathy 4 原则自检（必跑）

调 AskUserQuestion 前自检并展示给 PM：

| 原则 | 自检结果 | 备注 |
|---|---|---|
| Think Before Coding | ✅/⚠️ | PRD 5.5 假设表是否 ≥ 3 条业务+技术假设？ |
| Simplicity First | ✅/⚠️ | 5.3 不做清单是否明确？没埋"扩展性" |
| Surgical Changes | ✅/⚠️ | 范围是否聚焦本次需求？没顺手加无关项 |
| Goal-Driven Execution | ✅/⚠️ | AC 是否每条可测？ |

任一 ⚠️ → 主动声明给 PM，让他在 ✅ 前知情。

### Step 3：AskUserQuestion 三选项

```typescript
AskUserQuestion({
  questions: [{
    question: "PRD 是否通过？通过后我将 autonomous 跑完后续所有阶段，遇到 4 条红线才会叫醒你。",
    header: "PRD 关卡",
    multiSelect: false,
    options: [
      {
        label: "✅ 通过，开始 autonomous 跑",
        description: "PRD 方向正确，我去做别的 / 睡觉，遇到红线再叫我"
      },
      {
        label: "🔄 局部修改",
        description: "大方向 OK，但要改 [我会写出具体点]"
      },
      {
        label: "❌ 完全重做",
        description: "PRD 方向不对，重新沟通需求"
      }
    ]
  }]
})
```

### Step 4：PRD 通过 = 全权委托

PM 选 ✅ 后，本 skill 自动：
1. 在 `iteration-vault/meta.json` 记 `prd_approved_at`
2. 启动 autonomous 模式（Read `autonomous-mode.md`）
3. 依次跑 Phase 3 → 4 → ... → 12
4. 跑完后做完成汇报（详见下文）

PM 选 🔄 → 按 v1 流程重跑 Phase 2
PM 选 ❌ → 回 Phase 1 重新澄清

---

## 🚨 4 条红线 Escalation 规则

只有以下 4 种情况会**中断 autonomous 跑、暂停并等 PM**：

### 🚨 R1：重大架构冲突 / vendor lock-in

**触发条件**：
- 选型导致 vendor lock-in（绑定某商业 API、某专有数据库等）
- 必须重构现有核心模块（不是扩展，是改 / 删既有）
- 引入与 项目协议（如有）冲突的库
- 引入许可证不兼容的库（GPL / 商业等）

**处理**：
1. 暂停 Phase 4 架构设计
2. 写 `iteration-vault/ESCALATION-R1.md`：候选方案 + 影响 + skill 倾向
3. 等 PM 决策

### 🚨 R2：安全 must-fix > 3 项

**触发条件**：
- Phase 9 安全审查（任一路：自审 / 对抗审 / 安全审）发现 ≥ 3 项**阻塞发布**的 must-fix
- 或：1 项极严重（CVSS ≥ 9）

**处理**：
1. 暂停 Phase 10 验收
2. 写 `iteration-vault/ESCALATION-R2.md`：每项问题 + 严重度 + 修复成本
3. 等 PM 决策（修 / 推迟 / 砍范围）

### 🚨 R3：验收 3 次重试仍挂

**触发条件**：
- Phase 10 验收失败
- 自动回 Phase 7 修
- 修完再验
- 重复 3 次仍未全部通过

**处理**：
1. 暂停所有后续动作
2. 写 `iteration-vault/ESCALATION-R3.md`：3 次失败的具体原因 + 模式分析
3. 等 PM 决策（继续修 / 砍范围 / 改方案 / 推迟）

### 🚨 R4：必须删除既有功能

**触发条件**：
- 实施过程中发现必须删除某既有功能（不是 deprecate，是真删）才能完成本次需求
- 涉及 ≥ 100 行代码删除 + 影响其他模块

**处理**：
1. 暂停当前 phase
2. 写 `iteration-vault/ESCALATION-R4.md`：要删什么 / 为什么必须删 / 影响面 / 用户感知
3. 等 PM 显式同意

---

## 红线触发后的标准流程

### 1. 立即暂停
- 当前 phase 停在原地，已完成部分保留
- 标记 `iteration-vault/meta.json` 的 phase 状态为 `escalated-pending-pm`
- 如果在 `/loop` 模式 → 暂停 loop
- **必同步**：往 `iteration-vault/<run-id>/DEBUG-TRACE.md` 追加 E[NNN] 一条记录（模板：`templates/debug-trace.md`），含触发的 R 红线编号 / 触发位置 / 关联 ESCALATION-R[X].md 路径 / 当前状态
- **同步更新 RUN-LOG.md**：在当前业务名行加 🚨 标记 + 指向 DEBUG-TRACE.md E[NNN] anchor

### 2. 写 ESCALATION-R[N].md
统一格式（详见 `autonomous-mode.md` 的"Escalation 触发后的标准流程"段）。
关键：必有"skill 的倾向（基于保守默认）+ 候选方案 + 影响"。

### 3. 通知 PM
- 如果 PM 在线：直接对话发"🚨 触发 R[N]，详情：iteration-vault/ESCALATION-R[N].md"
- 如果 PM 离线（过夜跑）：标志在 vault，等下次 PM 回话主动报

### 4. 等 PM 决策 → 续跑
PM 给出方向后：
- skill 把决策记入 `autonomous-decisions.md` 标 `[ESCALATED-RESOLVED-R[N]]`
- 重置 loop 状态为 running
- 从触发点续跑

---

## 📋 Autonomous 决策记录（PM 不参与，但留痕）

Phase 3-12 跑的过程中，每个**不构成红线但需要选择**的决策都按 `autonomous-mode.md` 的格式记入 `iteration-vault/autonomous-decisions.md`。

**保守默认决策树（重点抄录，便于 skill 在线决策时查）**：

| 选择类型 | 选 | 原因 |
|---|---|---|
| 可逆 vs 不可逆 | 可逆 | Karpathy Simplicity First |
| 新方案 vs 成熟方案 | 成熟 | 风险最小化 |
| 复杂 vs 简单 | 简单 | Simplicity First |
| 通用 vs 专用 | 专用 | YAGNI |
| 抽象 vs inline | inline | Simplicity First |
| 删除 vs deprecate | deprecate | Surgical Changes |
| 升级依赖大版本 vs 维持 | 维持 | 除非必需 |
| 破坏性 schema vs 兼容性 | 兼容性 | 可回滚 |
| 启动即全量 vs 灰度 | 灰度 | 风险分摊 |
| 全自动 vs 留人工兜底 | 留人工 | r4 哲学 + 合同 9.1 |

---

## 🌅 完成汇报（Phase 12 完成后）

skill 跑完后**一次性**对 PM 汇报：

```
🌅 [PM 昵称，可选] 早上好/下午好！本次迭代已 autonomous 完成。

✅ 总体状态：成功
   （或：⚠️ 部分完成 / 🚨 触发 R[N] 中途暂停）

📊 数字摘要：
- 完成任务：[N]/[M]
- autonomous 决策：[K] 条（详见 iteration-vault/autonomous-decisions.md）
- 代码改动：[X] 个文件 / +[Y] / -[Z] 行
- 测试：单测 +[A] / e2e +[B] / 覆盖率 [P]%
- 安全：[全过 / 已修 N 项 must-fix]
- 验收：5 层 ✅ / PRD AC [N/M] 通过

📍 重点关注（PM 必看 3 个 ）：
1. iteration-vault/autonomous-decisions.md 标 ⚠️ 的 [N] 条
2. iteration-vault/09-review-reports/summary.md 的 should-fix
3. Release PR：[GitHub URL]

🚀 下一步（PM 一次点击）：
- 看 Release PR，点 merge → 自动发版到 private repo
- 或：在 vault 标 🔄 重做某段 → 我重跑
```

---

## v1 vs v2 关卡对比表

| 维度 | v1 | v2 |
|---|---|---|
| PM 关卡数 | 4 个 ⛳ | 1 个 ⛳（PRD） |
| 过夜跑可行性 | ❌ 不行（4 个关卡需 PM 在线） | ✅ 可以（PRD 后自治） |
| 决策审计 | 关卡时口头 | 完整 autonomous-decisions.md |
| Escalation | 没有专门机制（每个关卡都可以反复） | 4 条红线显式定义 |
| 失败回退 | 关卡时让 PM 选 ❌ | 红线 + 3 次重试机制 |
| 完成汇报 | 散在各 phase | 统一一次性汇报 + Release PR |
| 适用场景 | PM 全程参与的小项目 | 过夜跑 + 长迭代 + PM 时间紧张 |

---

## 何时回到 v1 4 关卡模式

少数场景下 PM 希望全程参与（如学习阶段、敏感的客户首次交付、合规要求高）：

PM 可以在 PRD 关卡说："这次我要全程参与，4 关卡模式"。skill 切换到 v1 行为，每个 phase 完成都问 PM。

但**默认是 v2**。
