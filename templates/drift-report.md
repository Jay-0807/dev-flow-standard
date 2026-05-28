# Phase 11.5 漂移检测报告模板

> Phase 11.5 自动写到 `iteration-vault/11.5-sync-report.md` 按本模板。

---

```markdown
# Phase 11.5 Drift Detection Report

**生成时间**: <YYYY-MM-DD HH:mm>
**总判定**: ✅ 可继续 Phase 12 / 🚨 阻塞（含 ERROR 漂移）

---

## 总览

| 维度 | INFO | WARN | ERROR |
|---|---|---|---|
| 技术栈 | 0 | 1 | 0 |
| API endpoint | 2 | 0 | 0 |
| 数据模型 | 0 | 0 | 0 |
| 页面路由 | 1 | 0 | 0 |
| 核心依赖 | 0 | 1 | 0 |
| **合计** | **3** | **2** | **0** |

**总判**: ✅ 可继续（仅 INFO/WARN）

---

## 任务状态更新（来自 06-task-breakdown.md）

| 任务 ID | 旧状态 | 新状态 | 判定理由 |
|---|---|---|---|
| #1 | `[~]` | `[x]` | 代码完整 + 单测通过 + 真实调用链 |
| #2 | `[~]` | `[x]` | 同上 |
| #3 | `[ ]` | `[~]` | 代码存在但缺单测 |
| #4 | `[~]` | `[ ]` | **代码空函数体**（autodev-sync 严判）|
| #5 | `[~]` | `[ ]` | **mock 数据驱动**（autodev-sync 严判）|
| ... | ... | ... | ... |

**统计**: 
- 已完成 `[x]`: [N]
- 进行中 `[~]`: [M]
- 未完成 `[ ]`: [K]（其中 [P] 项因严判被打回）

---

## INFO 级漂移（已自动同步文档）

### I-001: package.json patch 版本
- 文件：`package.json`
- 详情：`react: 18.2.0 → 18.2.1`（patch）
- 行动：✅ 已更新 `04-architecture.md §4` 技术栈段

### I-002: 新增 endpoint
- 文件：`04-api-spec.yaml`
- 详情：新增 `POST /api/v1/users/:id/preferences`（不影响既有）
- 行动：✅ 已追加到 `api-registry.md`

### I-003: 新增页面
- 文件：`app/orders/[id]/preferences/`
- 详情：新增个人偏好页面
- 行动：✅ 已追加到 `05-interface-design.md §3 用户旅程`

---

## WARN 级漂移（PM 在 12.5 看）

### W-001: 依赖 minor 版本变化
- 文件：`package.json`
- 详情：`next: 14.0.0 → 14.2.0`（minor）
- 影响：可能含新 feature 或 API 变化
- 行动：⚠️ 已记 autonomous-decisions.md，PM 在 Phase 12.5 决定是否需要 verify

### W-002: 跨模块新依赖
- 文件：`src/order-flow/api/orders.ts`
- 详情：新增 import `src/llm/pricing` （order-flow 模块原不依赖 LLM）
- 影响：架构边界变化
- 行动：⚠️ 已记，PM 决定是否需调 `04-architecture.md §1 数据流`

---

## ERROR 级漂移（**阻塞 Phase 12**）

### E-001: 删除已发布 endpoint
- 文件：missing in code
- 详情：`GET /api/v1/legacy/users` 在 04-api-spec.yaml 中标 active，但代码已删
- 影响：**老调用方会 404**
- 行动：🚨 触发 R4 红线 → 写 `ESCALATION-R4-drift.md`

[**仅当有 ERROR 时本段才存在，否则跳过**]

---

## 严格判定记录（autodev-sync 反模式触发）

本次有 [X] 项任务被严判打回：

| 任务 | 严判类型 | 详情 |
|---|---|---|
| #4 | 空函数体 | src/api/users/preferences.ts:42 `async getPreferences() {}` |
| #5 | mock 数据 | src/components/OrderTable.tsx:34 用 `fakeOrders` |
| #8 | 调用链断 | API `/api/v1/llm/pricing` 写好但 OrderForm 不调（dead code）|

这些任务在文档中标 `[ ]` 而非 `[~]` —— 避免虚假进度。

---

## 自动同步的文档（INFO 级）

本次自动改动的设计文档：
- `04-architecture.md §4 技术栈`: react 18.2.1
- `04-architecture-and-api.md §2.6 废弃流水`: 加 1 行
- `api-registry.md`: 追加 1 endpoint
- `05-interface-design.md §3 用户旅程`: 补充 preferences 页面

PM 可 git diff 看具体改动。

---

## 下一步行动

**如总判 = ✅ 可继续**:
- 进入 Phase 12 git release

**如总判 = 🚨 阻塞**:
- 写 `iteration-vault/ESCALATION-R4-drift.md`
- 暂停后续，等 PM 决策（R4 红线流程）：
  - 修：回 Phase 7 把 deleted endpoint 加回
  - 删：PM 明确同意删除 → 改 04-api-spec.yaml + 通知所有调用方
  - 推迟：本轮不发版

---

**生成时间**: <YYYY-MM-DD HH:mm>
**生成器**: Phase 11.5 sync
```
