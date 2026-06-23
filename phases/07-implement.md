# ⌨️ 代码实施

> 内部编号：Phase 7
> 模式：🤖 Autonomous + /loop + 每 code task GAN
> 工作流：前端/后端/DB 三路 git worktree 并行实施

## 目标
按 Phase 6 的任务分解，开启**多路并行实施**。前端、后端、数据库各自在独立 worktree 跑，互不阻塞。

## 输入
- `iteration-vault/06-task-breakdown.md`（PM 已通过的任务树）
- `iteration-vault/04-architecture-and-api.md`
- `integrations/database-architect.md`（DB 任务用）
- `principles/karpathy-llm-coding.md`（**必读**：全局 LLM 编码 4 原则，spawn agent 时强制注入）

## 工作流（4 步）

> **本 phase 是 Karpathy 4 原则最关键的应用点**。所有 spawn 的子 agent prompt 必须包含 4 原则的关键检查点。Read `principles/karpathy-llm-coding.md` 后再开干。

**Step 1：用 git worktree 隔离工作区**
触发 `superpowers:using-git-worktrees`，按以下规则建 worktree：
- 主 worktree：当前目录（用于做 schema 迁移、配置、不冲突的小改）
- worktree-fe：前端任务专用
- worktree-be：后端任务专用

DB 任务通常在主 worktree 做（因为 migration 是单线串行的）。

**Step 2：并行 spawn 实施 agent**
用 `subagent_type` = Agent 工具，触发 `superpowers:subagent-driven-development` + `superpowers:dispatching-parallel-agents`。

**每个 spawn 的 agent prompt 必含以下 Karpathy 4 原则检查点**（不可省略）：
```
执行任务前先输出：
1. 【Think Before Coding】我对本任务的核心假设（≤5 条）+ 任何模糊的地方
   → 如果假设关键且无法独立判断，停下来问主线程
2. 【Simplicity First】我会用什么模式实现？有没有用了"为将来扩展"的抽象？
   → 反过度工程：能 inline 就不抽函数，能 30 行就不写 200 行
3. 【Surgical Changes】我只改本任务必需的代码
   → 看到无关代码即使有问题也只记到 backlog，不顺手优化
4. 【Goal-Driven Execution】成功标准 = [PRD AC 编号 + 单测通过 + lint/type 0]
   → 循环迭代直到全部达成，不达成不 claim done
```

具体 spawn 三路：

**🚀 前端组（worktree-fe）**：
- 用 `/autodev-iterate` 处理每个前端任务
- 适配重点：调用前注入项目实际 UI 风格 / 现有 design tokens（启动时检测）+ 任务 ID + 验收点
- 任务示例：
```
基于 06-task-breakdown.md 的任务 F1（新增 XXX 页面）和 F2（接入新接口），
请按 05-interface-design.md 的 UI 规格实施。
风格遵循项目现有 design tokens（启动时检测），所有新增组件先看是否能复用现有组件目录。
验收：UI 还原 90%+，dev server 跑起来 PM 能看到。
```

**🚀 后端组（worktree-be）**：
- 任务依赖 API：用 `/autodev-api` 先定契约，再 `/autodev-iterate` 实现
- 任务不依赖 API：直接 `/autodev-iterate`
- 适配重点：检查跨服务契约——若项目有自定协议/RPC（如有，见项目定位文档（如有，无则跳过））则遵循，否则用标准 REST/gRPC/事件

**🚀 数据库组（主 worktree）**：
- Read `integrations/database-architect.md` 作为提示
- 由本 skill 主线程亲自做（而不是 spawn agent），因为 migration 涉及破坏性操作，需要 PM 主线确认每一步
- 关键步骤：
  1. 写迁移脚本（向前 + 向后）
  2. 在 dev DB 跑一次，验证 schema 正确
  3. 检查现有数据兼容性
  4. 准备回滚 SQL

**Step 3：周期性同步**
每完成一组任务（约 30-60 分钟），主线程做一次同步：
- 检查三路进度（agent 跑得怎么样）
- 检查接口契约是否一致（前端在调的接口、后端真的实现了吗）
- 检查冲突点（多路在改同一文件吗）

发现冲突 → 暂停涉及的路，主线程协调后再继续。

**Step 4：合并回主分支**
所有任务完成后，触发 `superpowers:finishing-a-development-branch`：
- 把 worktree-fe、worktree-be 的改动 merge 回主
- 处理 merge 冲突（手动 / 由本 skill 协调）
- 主 worktree 上做最后的"全栈联调"快速跑通

## 产出

**文件**：`iteration-vault/07-implementation-log.md`

结构：
```markdown
# 实施日志: <feature-name>

## Worktree 布局
- main: <commit hash>
- fe: <branch> @ <commit>
- be: <branch> @ <commit>

## 任务完成情况
| 任务 ID | 状态 | 实际工时 | 备注 |
|---|---|---|---|
| F1 | ✅ | 2h | - |
| F2 | ✅ | 1h | - |
| B1 | ✅ | 3h | 改了原计划的 X 字段 |
| D1 | ✅ | 1h | 迁移脚本通过 |

## 冲突 & 调整记录
- [时间] [冲突点] [解决方式]

## 联调结果
- 前后端契约：✅ 通过
- DB schema：✅ 应用启动正常
- 已知遗留：[1-2 个发现但本 sprint 不处理的小问题]

## 提交记录
- fe: [commits]
- be: [commits]
- db: [migration files]
```

**对 PM 的摘要**：
```
实施完成。三路并行情况：
- 前端：[N] 个任务 ✅，预估 Xh，实际 Yh
- 后端：[N] 个任务 ✅
- 数据库：[N] 个迁移 ✅，可回滚
全栈联调通过。已经合并回主分支。
进入代码债扫描阶段。
```

## 关卡处理
本阶段**不是** ⛳ 关卡。但若实施过程中发现架构设计与实际不符（如 API 设计在前端调用时不顺手），暂停三路，回到 Phase 4 调整。

## 失败回退
- 某路 agent 卡住 / 跑出错误代码 → 主线程接手，单步排查（用 systematic-debugging）
- 多路冲突严重（合并不动） → 放弃 worktree，回退到串行模式，按依赖顺序逐个做
- DB 迁移在 dev 库失败 → 暂停所有路，先解决 DB，再续

## 🤖 过夜跑模式（v2 标准做法，不再是可选）

**v2 默认就是过夜跑场景**——PM 已在 PRD 关卡 ✅ 通过，离场了。本 phase 配合 autonomous 全链路用 `/loop` 跑完。

### 启动 /loop 的标准命令

按任务量决定循环间隔：

| 任务数 | 循环命令 | 总时间预期 |
|---|---|---|
| ≤ 5 | `/loop 15m /autodev-iterate <next-task>` | 1-2h |
| 6-15 | `/loop 30m /autodev-iterate <next-task>` | 3-6h |
| 16-30 | `/loop 45m /autodev-iterate <next-task>` | 8-15h（过夜级） |
| > 30 | **触发"建议拆 sprint" 决策**，不直接跑 | 应该 R0：在 Phase 6 标 ⚠️ |

### 每轮 loop 内部做什么

```
Loop tick:
  1. 检查 06-task-breakdown.md 找下一个未完成任务
  2. spawn /autodev-iterate 实施该任务（含 Karpathy 4 原则注入）
  3. 实施完成后自动调 verification-loop 单点验证
  4. 通过 → 标该任务 ✅，更新 implementation-log
     失败 → 标 ⚠️ 留给主线程后续处理（不阻塞 loop）
  5. 写一行进度到 autonomous-decisions.md
  6. 进入下一 tick
```

### loop 终止条件

| 触发 | 行为 |
|---|---|
| 所有任务 ✅ | loop 自动停 → 进入 Phase 8 |
| 同一任务连续失败 3 次 | loop 暂停 → 写 ESCALATION-R3 → 等 PM |
| 触发 R4（要删既有功能）| loop 暂停 → 写 ESCALATION-R4 → 等 PM |
| 单 tick 超过 60 min 无进展 | loop 暂停 → 写 INFRASTRUCTURE_ERROR.md → 等 PM |
| PM 显式说"停止" | 立即停 |

### 早上 PM 回来看什么

- `iteration-vault/07-implementation-log.md` 总览所有任务状态
- `iteration-vault/autonomous-decisions.md` 看 loop 里每个 tick 做了什么
- 如有 escalation：`ESCALATION-R*.md` 必看

### 与 continuous-agent-loop / verification-loop 的关系

- `/loop` 是入口
- 内部决策"是否继续 / 怎么继续"由 `continuous-agent-loop` 提供
- 每轮验证由 `verification-loop` 接管

本 phase 是这三者组合用的标准案例。
