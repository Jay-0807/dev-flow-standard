# 🌙 夜间执行总览  {{date}}  功能：{{feature-name}}

> 本文件是 PM 早上**第一眼**就看的。
> 所有步骤名必须用业务名 + emoji，禁止用 "Phase X" 编号。
> 任何细节都从 anchor 跳到 drill-down 文档（决策日志.md / DEBUG-TRACE.md / 各 phase 产出）。

---

## 时间轴

[{{HH:MM}}] 📝 PRD 撰写 → ✅ 你已确认（{{选项}}，{{单批/N 批}}跑）
[{{HH:MM}}] 💡 方案发散 → 比较了 {{N}} 套（{{方案 A}}/{{方案 B}}/{{方案 C}}）→ 选 {{方案名}}
                          → AI 互审 {{N}} 轮通过
[{{HH:MM}}] 📐 影响面分析 → 改动涉及 {{模块清单}} 共 {{N}} 个模块
                           → {{N}} 条架构决策已记录
[{{HH:MM}}] 🏗️ 架构与接口设计 → AI 互审 {{N}} 轮通过
                                → 新增 {{N}} 个 API，已加入项目接口总目录
                                → R1 检查 ✅
[{{HH:MM}}] 🎨 界面设计 → AI 互审 {{N}} 轮通过
                         → 涉及 {{N}} 个屏 + {{M}} 个组件
                         → WCAG AA ✅，§1→§2 可追溯性 {{X/Y}}
                         → ⚠️ AI 自己拍板了一个决策（#{{N}}）：{{决策简述}}
[{{HH:MM}}] 📚 设计压缩成开发地图 → INDEX.md ({{N}} 行) + RULES.md ({{M}} 行) 已就绪
[{{HH:MM}}] 📋 任务拆解 → 拆成 {{N}} 个任务，估算 {{X}} 小时
                          → AI 互审 {{N}} 轮通过
[{{HH:MM}}] ⌨️ 代码实施 → 启动 {{N}} 个任务并行
                          {{任务清单状态：✅/❌/⚠️ 逐个列出}}
                          → 总耗时 {{X}}h{{Y}}min
[{{HH:MM}}] 🧹 代码债扫描 → 9 维度扫完，发现 {{N}} 处债，自动修 {{M}} 处，余 {{K}} 入 backlog
[{{HH:MM}}] 🔍 多路审查 → 代码审查 + GAN + 安全审计跑完
                         → 安全 must-fix: {{N}}（R2 检查 {{✅/🚨}}）
[{{HH:MM}}] 🛡️ 上线前质量检查 → {{第 N/3 次尝试}}
                                 → {{L1 ✅ / L2 ❌ R3-001 placeholder src/api.ts:42 / ...}}
                                 → {{自动回退到 ⌨️ 代码实施 修 / 全 ✅ 通过}}
[{{HH:MM}}] 📢 发布说明 → release notes 已写
[{{HH:MM}}] 🔄 文档代码漂移检测 → 5 维度扫完，{{无漂移 / N 处 INFO / M 处 WARN / 0 处 ERROR}}
[{{HH:MM}}] 🚀 git 发版 → PR #{{N}} 已创建（{{URL}}）
[{{HH:MM}}] ⏸ 等你早晨回来复盘

---

## 📌 一夜要点（PM 早上必看）

```
⛳ 你的关卡： {{N}} 处已通过（{{PRD}}） + {{N}} 处等你（{{☀️ 早晨复盘}}）

⚠️ AI 自己拍板的决策： {{N}} 条
   → 详见 决策日志.md
   → 其中 ⚠️ 标红需重点 review 的：{{N}} 条（决策 #{{X,Y,Z}}）

🚨 需要你处理的重大事件： {{无 / R[X] 列表 + 链接}}
   → 详见 ESCALATION-R[X].md（如有）

🔄 AI 推倒重来（GAN PIVOT）： {{N}} 次
   → {{触发的任务清单}}

❌ 失败重试： {{N}} 次
   → {{已自动修复（详见 DEBUG-TRACE.md E001-E00X）/ 触发 R3 升级}}

💰 大概花了： ~{{N}} 次 AI 调用 / ~{{N}} 小时 wall clock

📊 数字摘要：
  - 完成任务： {{N/M}}
  - 代码改动： {{X 文件 / +Y -Z 行}}
  - GAN 平均轮数： {{X.X}} 轮 PASS
  - 测试新增： {{N 单测 / M e2e}}，覆盖率 {{P}}%
  - 用户验收： N/A（已由 ⛳ 早晨复盘覆盖）
```

---

## 下一步

→ 打开 ☀️ 早晨复盘（4 步清单 + 4 选项）
→ 入口：`iteration-vault/<run-id>/12.5-morning-review.md`

---

## Drill-down 入口

| 想了解什么 | 打开这个文件 |
|---|---|
| AI 自治决策细节 | `iteration-vault/<run-id>/autonomous-decisions.md` |
| 错误链 / 失败追踪 | `iteration-vault/<run-id>/DEBUG-TRACE.md` |
| 各步骤完整产出 | `iteration-vault/<run-id>/0X-*.md` |
| GAN 互审过程 | `iteration-vault/<run-id>/<phase>-gan/` |
| 决策回放（想重跑某段）| `iteration-vault/<run-id>/checkpoints/` |
| 跨批进度（如分批跑）| `iteration-vault/<run-id>/batches/batch-plan.md` |
| 跨 session 状态 | `iteration-vault/<run-id>/state.yaml` |
