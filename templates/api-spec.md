# API 设计模板（Phase 4 §2 API 输出）

> 把本模板复制为 `iteration-vault/04-architecture-and-api.md`，逐段填充。
>
> ⚠️ **本模板使用 A2A 协议作示例数据**。PM 按实际项目协议（REST / GraphQL / gRPC / 自研）改造接口清单中的"是否 A2A"列。

---

```markdown
# API 设计 & 整理: <feature-name>

**版本**: <X.Y>
**日期**: <YYYY-MM-DD>
**对应 PRD**: iteration-vault/02-PRD.md
**对应架构**: iteration-vault/04-architecture.md
**对应 UX**: iteration-vault/05-interface-design.md

---

## 0. API 假设检查表（Karpathy "Think Before Coding"）

### 0.1 继承自架构的假设
从 04-architecture.md 引用：
- [假设 1]
- [假设 2]

### 0.2 API 新增假设表

| # | 假设 | 后果 | 验证方式 | 阻塞性 |
|---|---|---|---|---|
| 1 | 用户量 < 1000/QPS | per-user 100/min 够 | prod 监控 | 否 |
| 2 | 项目业务消息平均 200B | 信封不会撑爆 | 单元测试 | 否 |

### 0.3 刻意不做什么（Simplicity First）
- 不引入 [X gateway]，理由：QPS < 5k 不需要
- 不预留 [Y endpoint]，理由：YAGNI
- 不做 [Z 高级特性]，理由：MVP 不必

---

## 1. 新 API 设计（如本迭代涉及）

### 1.1 接口清单

| 接口 ID | 方法 | 路径 | 鉴权 | 描述 |
|---|---|---|---|---|
| order.create | POST | /api/v1/orders | JWT | 创建订单 |
| order.cancel | A2A | a2a:order.cancel | mTLS | agent 间取消订单 |

### 1.2 详细契约

#### 1.2.1 `POST /api/v1/orders`
- **鉴权**: Bearer JWT
- **限流**: per-user 100/min
- **请求**:
  ```json
  {
    "product_id": "uuid",
    "quantity": 1,
    "shipping_address": "..."
  }
  ```
- **响应 (201)**:
  ```json
  {
    "order_id": "uuid",
    "status": "pending",
    "data_source": "orders_table",
    "created_at": "..."
  }
  ```
- **错误**:
  - 400 `VALIDATION_INVALID_QUANTITY`
  - 402 `BUSINESS_INSUFFICIENT_FUNDS`
  - 409 `BUSINESS_DUPLICATE_ORDER`
  - 429 `RATE_LIMIT_EXCEEDED`

#### 1.2.2 `a2a:order.cancel`（项目业务消息）
信封 9 字段齐全 + payload 含 `order_id` + `reason`：

```json
{
  "message_id": "uuid",
  "sender_agent_id": "uuid",
  "receiver_agent_id": "uuid",
  "message_type": "order.cancel",
  "payload": { "order_id": "uuid", "reason": "..." },
  "confidence": 0.95,
  "human_review_required": false,
  "timestamp": "...",
  "trace_id": "uuid",
  "signature": "..."
}
```

### 1.3 项目业务协议（如有）适配（如适用）

| 接口 | 是否 A2A | 信封 9 字段 |
|---|---|---|
| order.cancel | ✅ | 齐全 |
| order.create | ❌（用户发起，走 REST）| n/a |

### 1.4 限流 / 幂等 / 版本

- order.create: 100/min per-user, 支持 Idempotency-Key, v1
- order.cancel: 500/min per-agent, message_id 幂等, v1

### 1.5 r4 字段嵌入（1-3 维）

| 接口 | confidence | human_review_required | data_source |
|---|---|---|---|
| order.create | n/a（非 LLM）| n/a（低风险）| ✅ orders_table |
| ai_pricing_suggest | ✅ 必填 | ✅ 必填（高风险）| ✅ pricing_model_v3 |

---

## 2. 存量审计报告

### 2.1 扫描范围
- 目录: `src/api/`, `src/lib/a2a/`
- OpenAPI 文件: `openapi/v1.yaml`
- 业务协议 topics（如有）: 从 `project.config.yaml` 加载
- 扫描接口总数: [N]
- 扫描耗时: [X] min

### 2.2 命名一致性

| 接口 | 当前命名 | 推荐命名 | 严重度 | 本迭代修复？|
|---|---|---|---|---|
| /api/v1/getUserOrders | 动词中心 | /api/v1/users/:id/orders | 🟡 major | 否（Surgical Changes，进 backlog）|

### 2.3 重复检测

| 候选 1 | 候选 2 | 相似度 | 建议 |
|---|---|---|---|
| /orders + /shop/orders | 99% | 合并到 /orders |

### 2.4 项目业务协议（如有）合规

| 接口 | 缺失字段 | 严重度 | 优先级 |
|---|---|---|---|
| user.notify | confidence | 🔴 critical | 本迭代修 |
| inventory.sync | trace_id | 🟡 major | 进 backlog |

### 2.5 调用关系图

```mermaid
graph LR
  A[order-agent] -->|a2a:order.cancel| B[inventory-agent]
  B -->|a2a:inventory.release| C[notification-agent]
  C -->|REST /api/v1/users/:id/notify| D[user-frontend]
```

本次改动影响：order-agent 新增 `pricing_suggest` 调用 → 下游 `pricing-agent`、`inventory-agent` 受影响。

### 2.6 废弃候选

| 接口 | 最后调用 | deprecate 日期 | 真删日期 | 替代 |
|---|---|---|---|---|
| /api/v0/orders | 2026-01-15 | 2026-06-01 | 2026-12-01 | /api/v1/orders |

---

## 3. 文档产物

- **3.1 OpenAPI YAML**: `iteration-vault/04-api-spec.yaml`
- **3.2 Postman collection**: `iteration-vault/04-api-postman.json`
- **3.3 人类可读 API.md**: `iteration-vault/04-api.md`
- **3.4 api-registry.md diff**: +2 新增 / 改 1 / deprecated 1

---

## 4. 三角自检表

| 维度 | 状态 | 备注 |
|---|---|---|
| Karpathy Think Before Coding（≥ 3 假设）| ✅ | 假设清单见 §0 |
| Karpathy Simplicity First（无顺手 endpoint）| ✅ | 见 §0.3 |
| Karpathy Surgical Changes（只改本迭代必需）| ✅ | 命名问题进 backlog 不顺手改 |
| Karpathy Goal-Driven（每接口可测 AC）| ✅ | 见 §1.2 错误码 + schema |
| r4 第 1 维（业务规则不硬编码）| ✅ | 走 business_rule 表 |
| r4 第 2 维（人工保留点字段）| ✅ | 见 §1.5 |
| r4 第 3 维（confidence + data_source）| ✅ | 见 §1.5 |
| 项目消息信封（如有）完整（9 字段）| ✅ | 见 §1.3 |
| RFC 9457 错误格式 | ✅ | 见 §1.2 |
| 版本策略一致（URL path）| ✅ | v1 |
| 限流策略一致 | ✅ | 默认值 + 显式覆盖 |
| 幂等性策略到位 | ✅ | Idempotency-Key + message_id |
| 废弃流程到位 | ✅ | deprecate → 6 月 → 真删 |
| Contract test 可跑 | ✅ | OpenAPI YAML + Postman |

任一 ❌ → 修，2 次仍 ❌ → R1 升级。

---

## 5. 给 autonomous-decisions.md 的摘要

```
[Phase 4 §2 API 摘要]
- 本迭代新增/修改 API: 2 个
- 存量审计: 扫 [N] 个，发现 [M] 个命名不一致 / [K] 个废弃候选
- 协议合规: [X/X] 通过
- 注册表更新: +2 / 改 1 / deprecated 1
- GAN 跑了 [N] 轮，PIVOT [yes/no]，PASS at round-[X]
- 关键决定: [一条 PM 关注的]
```
```
