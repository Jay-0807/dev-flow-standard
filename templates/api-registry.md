# API Registry — 跨迭代统一注册表（模板）

> **作用**：跨迭代持久化追踪所有 API 接口的元数据。Phase 4 §2 API 每次自动 append 更新。
> **路径**：`<project-root>/api-registry.md`（与项目 git 一起 commit）
> **首次创建**：Phase 4 §2 API 检测到不存在时用本模板初始化
>
> ⚠️ **本模板使用 A2A 协议作示例数据**（来自 firefly 项目）。PM 按实际项目协议调整：A2A? 列改为项目协议名（如 GraphQL / gRPC / REST），路径示例 `a2a:xxx` 改为项目实际格式。

---

```markdown
# Firefly API Registry

> 本文件持久化跨多次迭代，每次 Phase 4 §2 API 自动追加更新。
> 路径：`<project-root>/api-registry.md`（不在 iteration-vault/ 下，因为 vault 每次复位）

---

## 维护规则

- 每次 Phase 4 §2 API 末尾自动 append（新增）/ update（修改既有）
- PM 不直接编辑（防冲突），如需手工改先在 issue 里说明
- 每三个月做一次清理：删 > 6 个月已删除（不是 deprecated）的接口
- 字段顺序固定，新增字段加在末尾
- git commit 本文件时 message 必须有 "[api-registry update]" 前缀

---

## 全局元信息

- **最近一次更新**: <YYYY-MM-DD HH:mm>（迭代 `<iter-name>`）
- **接口总数**: N
- **协议合规率**: M/N (P%)
- **历史迭代数**: K

---

## 接口表（按命名空间分组）

### auth.*

| 接口 ID | 路径 / 主题 | 方法 | A2A? | 版本 | 引入迭代 | 最后审计 | 状态 | 调用方 | r4 字段齐全? |
|---|---|---|---|---|---|---|---|---|---|
| auth.login | POST /api/v1/auth/login | POST | 否 | v1 | iter-001 | 2026-05-20 | active | web-frontend | n/a |
| auth.refresh | POST /api/v1/auth/refresh | POST | 否 | v1 | iter-001 | 2026-05-20 | active | web-frontend | n/a |
| auth.agent_handshake | a2a:auth.handshake | A2A | 是 | v1 | iter-003 | 2026-05-23 | active | all-agents | ✅ |

### user.*

| 接口 ID | 路径 / 主题 | 方法 | A2A? | 版本 | 引入迭代 | 最后审计 | 状态 | 调用方 | r4 字段齐全? |
|---|---|---|---|---|---|---|---|---|---|
| user.get | GET /api/v1/users/:id | GET | 否 | v1 | iter-001 | 2026-05-20 | active | web-frontend, order-agent | n/a |

### order.*

| 接口 ID | 路径 / 主题 | 方法 | A2A? | 版本 | 引入迭代 | 最后审计 | 状态 | 调用方 | r4 字段齐全? |
|---|---|---|---|---|---|---|---|---|---|
| order.create | POST /api/v1/orders | POST | 否 | v1 | iter-001 | 2026-05-23 | active | web-frontend | n/a |
| order.cancel | a2a:order.cancel | A2A | 是 | v1 | iter-002 | 2026-05-23 | active | order-agent, support-agent | ✅ |

### product.* / sku.*

| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

### <your-project-protocol>.*

| 接口 ID | 主题 | 项目消息信封（如有）完整? | 版本 | 引入迭代 | 最后审计 | 状态 | 调用方 |
|---|---|---|---|---|---|---|---|
| a2a.inventory.sync | a2a:inventory.sync | ✅ | v1 | iter-002 | 2026-05-23 | active | inventory-agent |

### llm.*

| 接口 ID | 路径 / 主题 | 方法 | A2A? | 版本 | 引入迭代 | 最后审计 | 状态 | 调用方 | r4 字段齐全? |
|---|---|---|---|---|---|---|---|---|---|
| llm.pricing_suggest | POST /api/v1/llm/pricing | POST | 否 | v1 | iter-005 | 2026-05-23 | active | order-agent | ✅（confidence + data_source）|

---

## 废弃流水（deprecate → 6 月 → 真删）

| 接口 ID | deprecate 日期 | 计划删除日期 | 替代接口 | 通知状态 |
|---|---|---|---|---|
| auth.legacy_login | 2026-03-01 | 2026-09-01 | auth.login | ✅ 已通知 |

---

## 命名空间约定

- `auth.*`: 鉴权 / 会话
- `user.*`: 用户实体
- `tenant.*`: 租户
- `order.*` / `product.*` / `sku.*`: 电商业务
- `inventory.*`: 库存
- `pricing.*`: 价格
- `<your-project-protocol>.*`: agent ↔ agent（仅 AI 原生项目）
- `llm.*`: LLM 调用相关
- `webhook.*`: 第三方回调

---

## 本迭代变更摘要（按时间倒序，每次跑完追加）

### iter-005 @ 2026-05-23

**新增** (2):
- `llm.pricing_suggest` (POST /api/v1/llm/pricing) — AI 选品价格建议
- `a2a.pricing_request` (a2a:pricing.request) — order-agent 向 pricing-agent 请求

**修改** (1):
- `order.create` — 加 `pricing_suggestion_id` 参数（关联 llm.pricing_suggest 结果）

**标 deprecated** (0):

**审计发现 + 本迭代修复** (1):
- `user.notify` 缺 confidence 字段 → 加上

**审计发现 + 进 backlog** (1):
- `inventory.sync` 缺 trace_id → 进 backlog

### iter-004 @ 2026-05-15

**新增** (3):
...

### iter-001 @ 2026-04-01（初始版本）

**新增** (8):
- auth.login / auth.refresh / auth.logout
- user.get / user.update
- order.create / order.get
- product.list

---

## 命名一致性问题清单（backlog，每次 audit 追加）

| 接口 | 当前命名 | 推荐命名 | 发现于 | 严重度 | 拟修复迭代 |
|---|---|---|---|---|---|
| /api/v1/getUserOrders | 动词中心 | /api/v1/users/:id/orders | iter-003 | 🟡 | iter-008 |

---

## 项目业务协议（如有）合规问题清单（backlog）

| 接口 | 缺失字段 | 发现于 | 严重度 | 拟修复迭代 |
|---|---|---|---|---|
| inventory.sync | trace_id | iter-005 | 🟡 | iter-006 |

---

## 维护备忘

- 每三个月清理已删除（不是 deprecated）的接口
- 接口数 > 100 时考虑拆分本文件（按命名空间分文件）
- 每次本文件 git push 后检查是否破坏外部 contract test
```
