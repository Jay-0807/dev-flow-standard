# 🛡️ 上线前质量检查

> 内部编号：Phase 10  |  独立命令：/u-verify [--quick | --layer contract|static|runtime]
> 模式：🤖 Autonomous + 五层验收 + 红线编号 + 失败自动回退（v4 重写，学 autodev-verify）
> 五层：L1 契约 / L2 红线 / L3 静态 / L4 运行时 / L5 acceptance

## 这是什么

正式发布前的**最后一道技术关**。autodev 实证设计的 5 层独立验收 + 红线编号 + 失败自动回退上一 Step：

| 层 | 内容 | 失败处理 |
|---|---|---|
| **L1 契约** | 读 `06-task-breakdown.md` 任务，对照 `acceptance_criteria` 逐条验 | 任意条 ❌ → 回 Phase 6 重订 |
| **L2 红线** | 7 Quality Redlines（学 autodev）+ 项目专属红线 R8/R9（如有）| 任意红线触发 → 回 Phase 7 修代码 |
| **L3 静态** | tsc + lint + 类型检查 + 单测 + 依赖审计 | 失败 → 回 Phase 7 修 |
| **L4 运行时** | 真发 API 请求 + Playwright 测页面 + 旅程连续性 | 失败 → 回 Phase 7 修 |
| **L5 acceptance** | 调外部 `acceptance-testing` skill 跑 E2E | 失败 → 回 Phase 7 |

**v2/v3 → v4 变化**：
- ❌ v2/v3：5 层"糊"，无编号 + 无自动回退
- ✅ v4：每层独立 + **红线编号系统**（如 `R3-001: placeholder in src/foo.ts:42`） + **失败自动回退上一 Step**

---

## 目标

输出**可追溯的违规清单** + 自动回退到对应修复 phase。**3 次重试仍挂触发 R3 红线**。

---

## 输入

- `iteration-vault/RULES.md`（来自 Phase 5.9 压缩 — 含 7 Quality Redlines）
- `iteration-vault/06-task-breakdown.md`（含 acceptance_criteria）
- `iteration-vault/02-PRD.md` 的 AC
- `iteration-vault/04.5-api-spec.yaml`（contract test 用）
- `iteration-vault/04-architecture.md` 的契约定义
- `iteration-vault/09-review-reports/summary.md`（确保 must-fix 已修）
- `integrations/test-planner.md`（测试策略层）
- `principles/karpathy-llm-coding.md`
- `gan-engine/quality-redlines.md`
- `templates/verification-redlines-numbered.md`

---

## 工作流（5 层独立执行）

### Step 1：L1 契约层

**调用**: `/autodev-verify --layer contract`

**检查项**：
- 每个 `06-task-breakdown.md` 任务的 `acceptance_criteria` 逐条
- API 入参出参符合 `04.5-api-spec.yaml`
- DB schema 与实际 migration 一致

**输出**: 违规清单（含 `L1-XXX` 编号）

**失败处理**: 任意条 ❌ → 回 Phase 6 重订任务（不是回 7 修代码 — L1 是任务定义层）

### Step 2：L2 红线层

**调用**: 跑 `gan-engine/quality-redlines.md` 全部红线扫描 + 比对 RULES.md

**红线定义和 grep 命令唯一来源**：`gan-engine/quality-redlines.md` §reviewer 检查方法。
本文件不复述（SSOT 约定）。要改红线只改 quality-redlines.md。

**输出**：每条违规带编号写到 `iteration-vault/10-verification-redlines.md`：
- 格式：`L2-R<X>-<NNN>: <file>:<line>: <详情>`
- 编号体系模板：见 `templates/verification-redlines-numbered.md`

**任何 FAIL → 同时写 DEBUG-TRACE.md E[NNN]** 一条记录（含触发检测器 / 命中位置 / 关联红线编号 / 当前重试次数 / 自动行动）。

**失败处理**: 任意 critical（R1/R2/R3 必为 critical）→ 回 ⌨️ 代码实施（Phase 7）修代码

### Step 3：L3 静态层

**调用**: `/autodev-verify --layer static`

**检查项**：
- TypeScript `tsc --noEmit` 0 错误
- ESLint 0 错误
- 单元测试 `npm test` 通过
- 依赖审计 `npm audit` 无 high/critical 漏洞
- 类型检查覆盖率 ≥ 95%

**失败处理**: 任意失败 → 回 Phase 7 修

### Step 4：L4 运行时层

**调用**: `/autodev-verify --layer runtime`

**检查项**：
- API 真发请求（用 Postman collection from 04.5）+ 响应 schema 校验
- Playwright 测每个新 UI 页面：能 render + 关键交互能点 + 状态切换正确（loading/empty/error/success 三态实际渲染）
- 用户旅程连续性测：从 05a 用户旅程图自动跑，确保跳转无 404 / 死链
- 数据库 transaction 完整性测

**失败处理**: 任意失败 → 回 Phase 7 修

### Step 5：L5 acceptance（E2E）

**调用**: `acceptance-testing` skill（外部）

**检查项**：
- PRD AC 逐条跑（自动化能跑的）
- 关键用户故事 E2E 跑通

**失败处理**: 失败 → 回 Phase 7 修

---

## 红线编号系统

所有违规带编号，写到 `iteration-vault/10-verification-redlines.md`（按 `templates/verification-redlines-numbered.md`）：

```
L1-001: 任务 #5 acceptance_criteria 第 3 条未满足
L1-002: API /users/:id 返回 schema 缺 confidence 字段（PRD §7.3 要求）

L2-R1-001: src/api/orders/cancel.ts:42 含 `// TODO: handle edge case`
L2-R2-001: src/lib/llm/pricing.ts:18 用 `mockLLMResponse`
L2-R8-001: GET /api/v1/llm/pricing 返回缺 confidence 字段

L3-001: src/components/OrderTable.tsx:34 TypeScript 类型错（implicit any）
L3-002: ESLint no-unused-vars 在 src/api/users.ts:12

L4-001: GET /api/v1/orders 返回 500（reproducible）
L4-002: /orders 页面 loading 状态未实现（应有 spinner）

L5-001: PRD AC #4 "下单后立即扣库存" — 集成测试失败
```

每个编号关联 `RULES.md` 第 N 条（如 `R3-001` 关联 R3 降级语言）→ 可追溯。

---

## Quick mode（v4 新增）

中间 phase（Phase 5b / 6 / 7 间隔）可用 quick mode 只跑 L1-L3（轻量），Phase 10 跑 L1-L5（全量）。

```
/autodev-verify --quick           # L1-L3 only
/autodev-verify                   # 默认 L1-L5
/autodev-verify --layer contract  # 单层
```

---

## 失败回退（v4 自动化）

| 层失败 | 自动回退到 |
|---|---|
| L1 | Phase 6 重订任务（`06-task-breakdown.md`）|
| L2 (R1/R2/R3 critical) | Phase 7 修代码 |
| L2 (R4-R9) | Phase 7 修代码 |
| L3 | Phase 7 修代码 |
| L4 | Phase 7 修代码 |
| L5 | Phase 7 修代码 + 可能 Phase 4 调架构 |

**自动重跑**：修完后 skill 自动重跑 Phase 10。最多 3 次重试。

---

## R3 红线触发条件

3 次重试仍未通过任一层 → 触发 R3 红线 → 暂停 + 写 `ESCALATION-R3.md` + 升级 PM。

PM 决策：
- 缩范围 → 删除导致失败的任务 → 跑 L1-L5
- 改方案 → 回 Phase 4 调架构
- 推迟 → 归档 vault，下次重新跑

---

## 产出

- `iteration-vault/10-verification-report.md`（5 层总结）
- `iteration-vault/10-verification-redlines.md`（违规清单 + 编号）
- `iteration-vault/10-baseline-screenshots/`（L4 Playwright 截图）
- 各层 raw 日志（`10-l1.log` 等）

---

## 对 PM 的摘要

```
Phase 10 五层验收完成（第 [N] 次跑）:
- L1 契约: ✅ / ❌ [X 项违规]
- L2 红线: ✅ / ❌ [X 项 critical / Y 项 major]
- L3 静态: ✅ / ❌
- L4 运行时: ✅ / ❌
- L5 acceptance: ✅ / ❌
总判: [PASS / FAIL]

如 FAIL，违规清单见 iteration-vault/10-verification-redlines.md
自动回退到 Phase [X] 修复中...
```

---

## 与其他 phase 的接口

**上游**：所有 Phase 2-9 输出

**下游**：
- L5 PASS → Phase 10.5 真人用户验收
- 任意层 FAIL + 3 次重试不过 → R3 红线 → PM

---

## Standalone 模式

可独立触发：`/u-verify [--quick | --layer contract|static|runtime]`

```
/u-verify                       # 默认 L1-L5 全跑
/u-verify --quick               # 仅 L1-L3（轻量，开发中用）
/u-verify --layer contract      # 仅 L1 契约层
/u-verify --layer static        # 仅 L3 静态层
/u-verify --layer runtime       # 仅 L4 运行时层
```

不依赖主流水线 / iteration-vault，直接对当前工作目录 + 当前 git HEAD 跑五层验收。

输入解析：
- 没有 iteration-vault/ → 降级跑（L1 跳过，L2-L5 照跑）
- 有 iteration-vault/ → 全套跑

输出：
- 单跑模式：`.u-verify-report-<timestamp>.md` 到当前目录
- 流水线模式：`iteration-vault/10-verification-report.md`

任何 FAIL → 同时写 DEBUG-TRACE.md 一条 E[NNN] 记录。

---

## 维护备忘

- 红线编号格式（`L1-XXX` / `L2-RY-XXX`）跨迭代稳定，PM 历史追溯
- Quick mode 在 Phase 5/6/7 间用，加快开发循环
- L4 runtime 测试 mocking 与不 mocking 的平衡（autodev 实证：能不 mock 就不 mock）
- L5 acceptance-testing 外部 skill 升级时同步本文件
