# 🏗️ 架构与接口设计

> 内部编号：Phase 4（含原 Phase 4.5 合并）
> 内部分两步：① 大架构蓝图 → ② API 整理 + 注册表
> 模式：🤖 Autonomous（自治 + R1 升级）

## 目标

基于 PRD + 影响面分析 + 02.5 brainstorm 收敛方案，输出可被开发者直接照着做的完整技术方案：
- **§1 大架构**：数据流、模块拆分、技术选型、DB schema、可观测性
- **§2 API 整理**：新接口设计 + 存量审计 + 文档导出 + 跨迭代注册表

**关键约束**：API 设计必须从 UI 反推（autodev-api 铁律），所以本 phase §2 需要 Phase 5 UX wireframe 作为输入 — 详见 §与其他 phase 的接口的"双向迭代"段。

## 触发条件

- 大架构：始终触发
- API 整理：始终触发（即使本迭代不新增 API，也跑存量审计）
  - 例外：PRD 显式说"不涉及任何 API/agent 通信" → 跳过，在 autonomous-decisions.md 留痕

## 输入

**§1 大架构的输入**：
- `iteration-vault/02-PRD.md`
- `iteration-vault/02.5-brainstorm-converge.md`
- `iteration-vault/03-impact-analysis.md`
- `templates/architecture.md`
- `integrations/database-architect.md`（数据库专项 9 项清单）

**§2 API 整理的输入**：
- §1 产出
- `iteration-vault/05-interface-design.md` §1 UX 段（**关键**：UI 反推 API 来源）
- `templates/api-spec.md`
- `templates/api-registry.md`
- `integrations/api-architect.md`（9 项审计清单专家角色）
- `<project-api-standards>.md（universal 版无强制规约，可由项目自定）`（项目自定消息格式（如有） + 错误码 + 限流）
- `<project-positioning>.md（项目自定，无则跳过）`（§3.4 项目自定协议/RPC（如有）段）
- `<project-business-context>.md（项目自定，无则跳过）`（AI 原生项目的 LLM API 元数据三字段：confidence / human_review / data_source）
- `principles/karpathy-llm-coding.md`
- `<project-root>/api-registry.md`（跨迭代持久化文件；首次跑时若不存在则创建）

---

## 第 1 步：大架构蓝图 ✅ 跑 GAN

### 1.1 调 superpowers:writing-plans 起方案骨架

触发：自动 invoke writing-plans。
喂给它的上下文：PRD 核心段 + 影响面摘要 + Phase 2.5 brainstorm 收敛后的方案。

要求它输出**架构方案**（不是任务清单——任务清单是 Phase 6 的事）：
- 数据流图（文字描述即可，关键路径）
- 模块拆分（哪些是新增组件、哪些是改既有）
- 跨模块通信方式（直接调用 / 事件 / 项目自定协议/RPC（如有），否则标准 REST/gRPC/事件）— 仅提**通信方式**，**不设计具体 API 契约**（§2 做）

### 1.2 数据库设计 — 集成 database-architect

本机没有专门 DB 架构 skill。Read `integrations/database-architect.md`，按其中的 9 项检查清单亲自扮演 DB 架构师角色：

1. Schema 变更（新增表/字段、修改、删除）
2. 索引设计（必加 / 可加 / 不加 + 理由）
3. 数据迁移脚本（向前 / 向后兼容）
4. 查询性能预估（N+1 / 全表扫描风险）
5. 数据一致性（事务边界 / 隔离级别）
6. 数据安全（PII 字段 / 加密 / 审计日志）
7. 备份 & 回滚策略
8. 容量预估（1 年内数据量增长）
9. 读写分离 / 分库分表的触发条件

输出：嵌入到产出文件的 §1.2 DB 段。

### 1.3 技术选型对比（如有新引入库/服务）

若架构里引入了新的第三方库/服务：
- 列出 2-3 个候选
- 横向对比：性能 / 维护性 / license / 中文社区 / 与项目现有栈兼容性
- 给出推荐 + 理由
- 风险点 + 撤回方案

---

## 第 2 步：API 整理

### 2.1 新 API 设计 ✅ 跑 GAN

> **autodev-api 铁律**：从 UI 反推数据模型和接口。"每个页面要什么数据？必须存在什么 endpoint？"先 UI 后 API。不许凭空设计 endpoint。

**仅当本迭代涉及新增/修改 API 时**：

a) 读 `iteration-vault/05-interface-design.md` §1 UX 段，列出每个页面所需的数据
b) 反推数据模型 → 反推 endpoint 列表
c) 调 GAN 引擎（`task_type: api-design`）生成详细 API 契约：
   - 路径 / 方法 / 入参 / 出参 / 错误码（RFC 9457 Problem Details）
   - 鉴权 / 限流 / 幂等 / 版本兼容
   - 项目自定协议/RPC（如有）适配（若项目有自定消息格式则遵循，否则用标准 REST/gRPC/事件）
   - AI 原生项目：LLM API 元数据三字段嵌入（置信度 confidence / 人工复核标记 human_review_required / 数据来源 data_source）；非 AI 项目不强制

**GAN 钩子**：
- 任务类型：`api-design`
- 输入：05-interface-design.md §1 UX 段 + §1 大架构 + <project-api-standards>.md
- 输出路径：`iteration-vault/04-api-design-gan/`
- 默认 N=5（含早退）
- 失败 fallback：单 gen + Karpathy 自检
- 后续行为：
  - pass → 进 §2.2
  - needs_improvement → 标记 + 留 Phase 9 兜底，仍进 §2.2
  - degraded → autonomous-decisions.md 留痕，仍进 §2.2

### 2.2 存量审计（每次都跑，**不跑 GAN**，纯机械扫描）

> 🚀 **CodeGraph 加速层（条件式，2026-05-30 加）**：用 Glob 查 `<项目根>/.codegraph/codegraph.db`。存在 → 下方「调用关系图」维度**优先用 `codegraph_callers/callees(symbol=<英文符号>)`** 取精确调用链（替代「谁调谁」的 GPT 判定 + 路径模糊匹配）；不存在 → 照原逻辑（项目未索引时可在 autonomous-decisions.md 建议 `codegraph init`）。铁律：英文符号名、查函数不查常量；只索引代码不含 .md。

读 `<project-root>/api-registry.md`（如不存在则用 `templates/api-registry.md` 初始化）。

调 `integrations/api-architect.md` 的"9 项审计清单"，但仅跑 5 维（其余 4 维由 §2.1 GAN 覆盖）：

| 维度 | 扫描方式 |
|---|---|
| **命名一致性** | 前缀 / 复数 / 动词风格 |
| **重复检测** | 语义相似度（GPT 判定）+ 路径模糊匹配 |
| **契约合规** | 服务间契约一致（若项目有自定消息格式则按其必填字段校验，否则按标准 REST/gRPC 契约校验请求/响应 schema）|
| **调用关系图** | 谁调谁，本次改动影响的下游服务 |
| **废弃候选** | ≥ 6 个月未调用 OR 已被新接口覆盖 |

**审计上限**：扫 ≤ 50 接口 / ≤ 15 min。超阈值降级为仅扫"被本迭代 impact 的接口"，autonomous-decisions.md 留痕降级。

**输出**：审计报告嵌入主文档 §2.2 段。

### 2.3 文档导出 + 注册表更新

生成 4 件产物（API 衍生）：

1. `iteration-vault/04-architecture-and-api.md` — 主文档（§1 大架构 + §2 API 整理 + 审计报告）
2. `iteration-vault/04-api-spec.yaml` — OpenAPI 3.1（机器消费）
3. `iteration-vault/04-api-postman.json` — Postman v2.1（QA 测试）
4. `iteration-vault/04-api.md` — 人类可读 API 文档（中文为主）

并追加更新 `<project-root>/api-registry.md`：
- 新接口（含本迭代标签）
- 存量接口的"最后审计日期 / 协议合规状态 / 调用方"
- 标 deprecated 候选（先 deprecate 不删，6 个月后真删）
- 本迭代变更摘要追加到"### iter-<slug> @ <date>" 段

### 2.4 Karpathy 4 + 项目自定协议（如有）双角自检

逐项确认：
- **Think Before Coding**：API 假设清单 ≥ 3 条已显性写出
- **Simplicity First**：没顺手加无关 endpoint / 没 future-proof header / 单租户不预先做多租户隔离
- **Surgical Changes**：本迭代只动必需接口；存量审计发现的命名不一致仅 flag，不顺手改（除非违规 ≥ 5 触 R1）
- **Goal-Driven Execution**：每个新接口有可测 AC（schema 校验通过 / RFC 9457 错误码 / 项目自定消息格式（如有）齐全）
- **业务规则显性化 / 人工保留点 / 数据来源标注**：业务规则不硬编码进 endpoint / 人工保留点字段存在 / 数据来源标注存在（AI 原生项目额外要求 confidence + data_source 字段）
- **协议合规**：项目自定协议/RPC（如有）消息格式完整

任一项 ❌ → 修，最多重试 2 次仍 ❌ → R1 升级。

---

## 完整产出

主文件 + 衍生：
- `iteration-vault/04-architecture-and-api.md` — 主文档
- `iteration-vault/04-api-spec.yaml` — OpenAPI 3.1
- `iteration-vault/04-api-postman.json` — Postman v2.1
- `iteration-vault/04-api.md` — 人类可读 API 文档
- `iteration-vault/04-architecture-gan/` + `04-api-design-gan/` — GAN trace
- `<project-root>/api-registry.md`（append 更新，跨迭代持久化）

### 04-architecture-and-api.md 主文档结构

```markdown
# 架构与接口设计: <feature-name>

## 1. 大架构
### 1.1 整体架构（数据流图 + 模块拆分 + 通信方式）
### 1.2 数据库设计（9 项 DB 架构清单）
### 1.3 技术选型（候选对比 + 推荐 + 风险）
### 1.4 可观测性（关键日志点 + 监控指标 + 告警）
### 1.5 兼容性 & 撤回方案（灰度策略 + 回滚步骤）

## 2. API 整理
### 2.1 新 API 设计（UI 反推 + GAN 产出 + LLM API 元数据三字段，AI 原生项目）
### 2.2 存量审计（5 维扫描结果）
### 2.3 文档导出 + 注册表更新摘要
### 2.4 Karpathy + 协议合规自检
```

### 对 PM 的摘要

```
🏗️ 架构与接口设计完成，关键决策：

【架构】
1. 数据流：[一句话]
2. 数据库变更：[简要描述]
3. 新引入的库/服务：[列表] 或"无"
4. 灰度 & 回滚：[策略]

【API】
5. 本迭代新增/修改 API：[N] 个
6. 存量审计：扫描 [M] 个，发现 [K] 个命名不一致 / [L] 个废弃候选
7. 协议合规：[N/N] 通过
8. 注册表更新：+[X] / 更新 [Y] / 标 deprecated [Z]
9. GAN 跑了 [N] 轮 PASS / PIVOT [yes/no]

关键关注：[最值得 PM 知道的一条]
```

---

## 🤖 Autonomous 决策 + 🚨 R1 检查

**不调 AskUserQuestion**。本 phase 完成后直接进入 Phase 5，但需做以下两件事：

### Autonomous 决策记录

把本 phase 的关键决策追加到 `iteration-vault/autonomous-decisions.md`，每条按 9 字段格式写：触发情境 / 候选 / 选择 / 依据 / 影响范围 / 可逆性 / PM 早上需要 review 的点。

典型本 phase 会记 5-12 条决策，涵盖：
- 架构选型（A vs B 库 / REST vs GraphQL / 自建 vs 第三方）
- DB schema 决策（拆表 vs 加列 / 索引策略）
- 灰度 & 回滚策略
- 监控接入点
- API 命名冲突处置
- 存量废弃候选处置

### 🚨 R1 红线检查（必跑）

#### 大架构层 R1

| 检查项 | 触发条件 | 命中怎么办 |
|---|---|---|
| Vendor lock-in | 选型绑定某商业 API / 专有数据库 / 闭源 SDK | **触发 R1** |
| 重构既有核心模块 | 不是扩展、是改/删 项目现有核心代码 ≥ 100 行 | **触发 R1** |
| 项目自定协议/RPC（如有）冲突 | 引入与项目自定协议/RPC（如有）设计冲突的库 | **触发 R1** |
| 许可证不兼容 | 引入 GPL / 商业 license 与 项目栈不兼容 | **触发 R1** |

#### API 子层 R1

| 检查项 | 触发条件 | 命中怎么办 |
|---|---|---|
| 新 API 与项目自定协议/RPC（如有）根本冲突 | 不是适配能解决的 | **触发 R1** |
| 存量违反服务间契约 ≥ 5 项 | 全栈契约设计有系统性问题 | **触发 R1** |
| breaking change 公开接口 | v1 → v2 不兼容 | **触发 R1** |
| 引入第三方 API gateway / 商业 API 平台 | vendor lock-in | **触发 R1** |

任一命中 → 暂停后续，按 `gates.md` 的"红线触发后的标准流程"处理：写 `ESCALATION-R1.md` + 同步写 DEBUG-TRACE.md + 等 PM 决策。

未命中 → autonomous 继续进入 Phase 5。

### 保守默认（无 R1 时）

按 `gates.md` 的"保守默认决策树"自治：
- 选型冲突 → 选成熟方案
- 复杂 vs 简单 → 选简单
- 抽象 vs inline → 选 inline
- 破坏性 schema vs 兼容性 → 选兼容性
- API 命名冲突（新 vs 存量）→ 选存量（让本次新接口让步）
- 存量 API 不一致 < 5 项 → flag 不阻塞（进 backlog）
- 存量 API 不一致 ≥ 5 项 → 触发 R1（系统性问题）

每条选择记入 autonomous-decisions.md。

---

## 与其他 phase 的接口

**上游**：
- Phase 3 影响面（找出被影响的模块和 API）
- Phase 2.5 brainstorm（方案收敛结果）

**下游**：
- Phase 5 🎨 界面设计（前端按本 phase API 输出接 mock / 真实 API）
- Phase 6 任务分解（架构 + API 任务从本 phase 拆出）
- Phase 7 实施（后端任务直接 Read 04-api-spec.yaml）
- Phase 9 安全审查（OWASP API Top 10 跑 04-api-spec.yaml）
- Phase 10 验收（contract testing 用 Postman collection）

**特殊关系（与 Phase 5 的双向迭代）**：
- API 设计需要 Phase 5 UX 信息作为输入（UI 反推 API）
- 实际操作顺序：先跑本 phase §1 大架构 → 跑 Phase 5 UX 出 wireframe → 回本 phase §2 做 API 设计

---

## 失败回退

| 失败 | 兜底 |
|---|---|
| PRD 要求功能在现有架构里几乎不可能优雅实现 | 升级，建议把本次需求拆成"重构 + 加功能"两个迭代 |
| 技术选型 3 个候选难抉择 | 调 `deep-research` 跑一次 tech-evaluation |
| /autodev-api 返回空 / 烂 | 降级单 gen + Karpathy 自检 + 手工按 templates/api-spec.md 起草 |
| 注册表持久路径写不进 | 暂存 iteration-vault/04-api-registry-pending.md，下次 PM 决策迁移 |
| 跨迭代注册表与本次设计冲突且无法和解 | R1 升级 |
| 存量审计耗时 > 15min | 自动降级为仅扫"被 impact 的接口" |
| Spectral 不可用 | 内置规则手工跑 + 留痕降级 |
| GAN 跑挂 | 降级单 gen + 标 gan_result: degraded，仍进下一步 |

---

## autodev-api 反模式（必拒绝，写入 reviewer prompt）

- ❌ endpoint 脱离 UI 实际需求（凭空设计）
- ❌ 推测性字段（PM 没说要但你猜可能需要）
- ❌ 错误处理不完整（只考虑 happy path）
- ❌ 业务规则模糊（违反"业务规则显性化"原则）
- ❌ 给同一资源同时设计 REST 和项目自定协议/RPC（如有）两条路径
- ❌ POST 用作 GET（"我懒得设计 query string"）
- ❌ 错误响应不带 code 字段，靠 http status 区分
- ❌ 版本写在 query / cookie / header 三处不统一
- ❌ 直接删接口而不 deprecate
- ❌ LLM 输出 API 没 confidence 字段
- ❌ 限流策略写在代码里而非配置里
- ❌ 引入商业 API gateway 锁定 vendor

---

## 维护备忘

- 每跑完一次本 phase，把"哪类 API 模式经常违反项目自定协议/RPC（如有）"沉淀到 `integrations/api-architect.md`
- 每三个迭代抽样 audit `api-registry.md`，删过期 deprecated 接口
- /autodev-api 工具升级后，同步 §2.1 调用接口
- DB 架构经验沉淀到 `integrations/database-architect.md`
- 合并历史：本文件 v1 = 原 04-architecture.md + 原 04.5-api-design.md 整合
