# 📋 任务拆解

> 内部编号：Phase 6
> 模式：🤖 Autonomous + GAN（任务分解 & sprint 排期）
> v4 含自动估算工时：< 8h 单批跑 / ≥ 8h 主动提示是否分批

## 目标
把 PRD + 架构 + UI（如有）转化为**可被一个一个 check 掉的任务树**，标注依赖关系、工时预估、责任分组（前端/后端/DB/测试）。本 phase 是 **Autonomous 自治 + 异常标记**（不是 PM 关卡；全流程只有 PRD + 早晨复盘 2 个 PM 关卡）——自治拆解后直接进 Phase 7，仅把异常（工时/任务数/关键路径超阈值）标 ⚠️ 进 autonomous-decisions.md 待早晨复盘。

## 输入
- `iteration-vault/02-PRD.md`
- `iteration-vault/04-architecture-and-api.md`
- `iteration-vault/05-interface-design.md`（如有）
- `templates/task-breakdown.md`

## 工作流（4 步）

**Step 1：调 superpowers:writing-plans 起任务树骨架**
触发：自动 invoke writing-plans。
喂给它：PRD 核心段 + 架构方案。
要求它**只列任务**，不重复架构内容：

```
基于附加的 PRD 和架构，输出一个分层任务树：
- 顶层按"工序组"分（前端 / 后端 / 数据库 / 测试 / 部署）
- 每个工序组下列具体任务
- 每个任务标依赖关系（依赖哪些前置任务完成）
- 不要重复架构内容
```

**Step 2：调 product-sprint-prioritizer 评估工时 + 排期**
用 Agent 工具调 `product-sprint-prioritizer`，输入任务清单：

```
请为以下任务列表评估：
1. 每个任务的相对工时（小 / 中 / 大）
2. 推荐执行顺序（关键路径优先）
3. 哪些任务可以并行（不同工程师同时做）
4. 是否需要拆得更细（避免单任务 > 1 天）
5. 整体 sprint 周期建议（1 周 / 2 周 / 3+周）
```

**Step 3：识别并行机会**
本 skill 主线程审查任务树，标注：
- 🚀 **并行组 A（前端）**：所有不依赖后端接口完成的前端任务
- 🚀 **并行组 B（后端）**：所有不依赖前端确定的后端任务
- 🚀 **并行组 C（DB）**：数据迁移、schema 变更
- 🚦 **串行关键路径**：必须按顺序做的、不能并行的任务

这个标注会指导 Phase 7 怎么 spawn 并行 agent。

**Step 4：风险 & 兜底任务补全**
检查任务树是否包含：
- ✅ 测试任务（单元 / 集成 / e2e 至少各 1 条）
- ✅ 文档任务（API doc、README 更新、release notes 草稿）
- ✅ 监控任务（日志埋点 / 告警规则配置）
- ✅ 灰度任务（feature flag / 灰度配置）
- ✅ 回滚任务（数据迁移逆向 / feature flag 关闭）

缺哪个补哪个。

## 产出

**文件**：`iteration-vault/06-task-breakdown.md`（用 templates/task-breakdown.md）

结构：
```markdown
# 任务分解: <feature-name>

## Sprint 概览
- 预计周期：[N] 天
- 关键路径：[Task A → Task B → Task C]
- 并行机会：[3] 组

## 任务树

### 工序组：前端
| ID | 任务 | 工时 | 依赖 | 并行组 | 验收点 |
|---|---|---|---|---|---|
| F1 | 新增 XXX 页面 | M | - | A | UI 还原 90%+ |
| F2 | 接入新接口 | S | B2 | - | 数据加载成功 |

### 工序组：后端
| ID | 任务 | 工时 | 依赖 | 并行组 | 验收点 |
|---|---|---|---|---|---|
| B1 | 新增 API /xxx | M | D1 | B | 通过单元测试 |

### 工序组：数据库
...

### 工序组：测试
...

### 工序组：部署 & 运维
...

## 关键路径甘特图（文字版）
Day 1: D1, B1 // 数据库 + 后端基础
Day 2: B2, F1 // 后端核心 + 前端骨架（并行）
Day 3: F2, T1 // 前端接入 + 测试
Day 4: 部署 + 验收
```

**对 PM 的摘要**：
```
任务分解完成（路径：iteration-vault/06-task-breakdown.md）。
关键数字：
- 共 [N] 个任务，预计 [M] 天 sprint
- 并行机会：[3] 组（前端/后端/DB 大段并行）
- 关键路径：[最关键的 3 个任务]
- 已包含测试 [N] 条、监控埋点 [M] 个、灰度方案
请关注：[1-2 个 PM 需要决定的事，如 deadline 是否合理]
```

## 🤖 Autonomous 决策（v2）

**不再调 AskUserQuestion**。完成任务分解后直接进入 Phase 7，但需：

### 1. autonomous 决策记录

把任务分解的关键决策记入 `iteration-vault/autonomous-decisions.md`，典型 2-5 条：
- 任务粒度（是否进一步拆 XL 任务）
- 并行度（前后端是否同步起）
- 测试占比（是否给关键路径加额外测试）
- sprint 周期估计

### 2. 自检（无 escalation 触发条件）

本 phase 不触发任何红线（红线在 phase 4/7/9/10）。但要做以下自检：

- [ ] 总工时 vs PRD 暗示的 deadline 是否合理？严重不合理（如 PRD 暗示 1 周但任务量需要 1 月）→ 在 autonomous-decisions.md 标 ⚠️ 让 PM 早上注意
- [ ] 任务数是否过多？> 30 个 → 触发"建议拆分"决策（不是 escalation，但记到 ⚠️）
- [ ] 关键路径是否过长？> 5 天 → 同上标 ⚠️
- [ ] 是否包含测试 / 监控 / 灰度任务？缺哪个补哪个（按 Karpathy Goal-Driven 强制要求验收点）

自检通过 → autonomous 进入 Phase 7。

## 失败回退
- 任务数 > 30 个 → 建议拆成多个 sprint，把本次 sprint 范围缩小
- 关键路径过长（> 5 天串行）→ 重新审视架构，看能否更多并行
