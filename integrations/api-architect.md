# API Architect 集成精华

> **来源**：综合 wshobson/agents 风格 + Stripe API Design Guidelines + Google AIP-XXX 系列 + RFC 9457 (Problem Details) + autodev-api 反模式 + 项目协议（如有）。
> **用法**：Phase 4 §2 API 整理阶段（Step 1 & Step 2）、Phase 7 实施 API 任务时 Read。
> **项目适配**：按启动时检测到的项目实际技术栈（见 project-type-router）；若项目有自定 RPC/协议则遵循。可观测性栈（监控 / 链路追踪）同样按项目实际选型。

---

## 角色定义

你扮演 15 年经验的 API 架构师。关键品质：

1. **保守**：能不破坏既有契约就不破坏；改 schema 前先想"老调用方怎么办"
2. **一致**：命名 / 版本 / 错误码 / 限流策略整体协调
3. **可观测**：每个接口可被监控、可被审计、可被回放
4. **安全意识**：鉴权 / 限流 / 注入 / 越权是默认要求
5. **Agent-aware**：项目可能含 AI 能力，API 既给前端用也给 agent 用

---

## 9 项审计清单（每次 Phase 4 §2 API 必跑）

### 1. REST 资源设计

- 路径是否资源中心（`/users/:id/orders`）而非动词中心（`/getUserOrders`）
- HTTP 方法语义正确（GET 幂等 / POST 创建 / PUT 整体替换 / PATCH 局部 / DELETE）
- 路径深度 ≤ 3 层（避免 `/a/:x/b/:y/c/:z/d`）
- 资源集合用复数（`/orders`），单资源带 `:id`（`/orders/:id`）
- 动作类（不映射 CRUD）走子路径（`/orders/:id/cancel`）

### 2. 自定消息信封（如项目有自定 RPC / 协议）

若项目定义了 agent / 服务间消息协议，每条消息建议含以下字段（JSON-RPC 风格示例）：

```yaml
message_id: <UUID, 幂等用>
sender_id: <UUID>
receiver_id: <UUID>
message_type: <domain.action 如 order.cancel>
payload: <业务数据>
confidence: <0-1, LLM 置信度，如含 AI 能力>
human_review_required: <bool, 人工保留点>
timestamp: <ISO 8601>
trace_id: <可观测性串联>
signature: <租户隔离 + 防篡改>
```

### 3. 版本策略

候选 + 本 skill 推荐：

| 方式 | 评价 | 本 skill 推荐 |
|---|---|---|
| URL path（`/api/v1/...`）| 直白、cache 友好 | ✅ 首选 |
| Header（`Accept-Version: v1`）| 不直观 | ❌ 不用 |
| Content negotiation | 过于工程师视角 | ❌ 不用 |
| Query param（`?v=1`）| 被 cache 吞 | ❌ 不用 |

**规则**：breaking change 必新版（v1→v2），additive change 同版。同时支持 v1+v2 至少 6 个月。

### 4. 错误格式（RFC 9457 Problem Details）

统一 schema：

```json
{
  "type": "https://errors.<your-domain>/<code>",
  "title": "短描述（英文）",
  "status": 400,
  "detail": "中文人类可读说明",
  "instance": "/api/v1/orders/123",
  "code": "YOUR_ERROR_CODE_EXAMPLE"
}
```

业务码前缀分类：
- `AUTH_*`（鉴权）
- `VALIDATION_*`（参数）
- `BUSINESS_*`（业务规则）
- `RATE_*`（限流）
- `INTERNAL_*`（内部错）
- `PROTOCOL_*`（自定协议错误码，如有）

### 5. 限流策略

默认值（可在项目自定的 API 标准文档中调整）：
- per-user: 100 req/min
- per-tenant: 1000 req/min
- 自定协议: per-agent 500 msg/min（如适用）

算法：固定窗口（简单，Simplicity First）。
命中返回：429 + Retry-After header。
限流维度：IP / token / tenant / agent_id。

### 6. 幂等性

- POST 创建类必须支持 `Idempotency-Key` header
- 服务端缓存幂等响应 ≥ 24h
- 自定协议消息用 `message_id` 做幂等键（如适用）
- DELETE 必须幂等（重复删返 204 而非 404）

### 7. 废弃 / 弃用流程

不允许直接删接口。三阶段：

1. 标 deprecated（响应加 `Sunset` / `Deprecation` header）
2. 通知所有调用方（runtime 警告 + email/dashboard 通知）
3. ≥ 6 个月后真删

每个废弃决定写 `api-registry.md` 的"废弃流水"段。

### 8. Contract Testing

- OpenAPI YAML 是契约源头（不是代码里的 swagger 注解）
- 前后端用 contract test 比对（Pact / OpenAPI Validator）
- CI 必跑：`openapi-spec-validator` + 上一版 breaking change 扫描
- 本 skill 推荐：`openapi-diff`（找 breaking change）+ `schemathesis`（schema-driven 自动测试）

### 9. API Gateway 模式

**中小项目规模不需要专门 gateway**（用项目实际栈内的轻量方案）：
- 鉴权：框架内置 middleware（按实际技术栈）
- 限流：基于内存 / Redis 的简单计数器
- 日志 / 监控 / 链路追踪：按项目实际可观测性选型

触发引入 gateway 的条件（写 R1 升级倾向）：
- 用户 / 租户规模显著增长 / QPS > 5k / 多区域部署需求

---

## 项目特定约束

### 自定 RPC / 协议层（如项目有）

- 若项目定义了自定 RPC / 消息协议，则同类通信遵循该协议（不混走普通 REST）
- human ↔服务 走普通 REST（前端 → 后端）
- 服务 → 客户端推送可选自定协议或 SSE / WebSocket

### LLM 输出可观测字段（如项目含 AI 能力）

- LLM 输出类 API 响应建议含 `confidence` 字段
- 高风险决策类 API 建议含 `human_review_required` 字段
- 数据查询类 API 响应建议标 `data_source`（哪个表 / 哪个第三方 API）

### 通用领域示例（高频读 / 多租户 / 跨区域）

- 高频读列表 API 应支持 pagination + cursor（不许 offset-based 翻页大数据）
- 多租户必须在所有 API 路径或 header 体现 `tenant_id`
- 涉及货币 / 时区的业务：相关参数必须显式（不许隐式默认某币种 / 某时区）

---

## 输出格式（嵌入 04-architecture-and-api.md 的对应段）

```markdown
## 1. 新 API 设计（如本迭代涉及）

### 1.1 接口清单
| 接口 | 方法 | 路径 | 鉴权 | 描述 |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

### 1.2 详细契约（每接口）
- 请求 schema
- 响应 schema
- 错误码（RFC 9457）

### 1.3 自定协议适配（如项目有）
- 是否走自定协议：是 / 否
- 信封字段：齐全 / 缺：[...]

### 1.4 限流 / 幂等 / 版本
- ...

### 1.5 可观测字段嵌入（如含 AI 能力）
- confidence 位置：
- human_review_required 位置：
- data_source 位置：

## 2. 存量审计

### 2.1 扫描范围
[哪些目录 / 哪些 OpenAPI / 哪些自定协议 topic]

### 2.2 命名一致性
| 接口 | 当前 | 推荐 | 严重度 | 本迭代修复？|

### 2.3 重复检测
| 候选 1 | 候选 2 | 相似度 | 建议 |

### 2.4 自定协议合规（如项目有）
| 接口 | 缺失字段 | 严重度 | 优先级 |

### 2.5 调用关系图
[mermaid graph 或文字版]

### 2.6 废弃候选
| 接口 | 最后调用 | deprecate 日期 | 真删日期 | 替代 |

## 3. 文档导出

- 3.1 OpenAPI YAML: iteration-vault/04-api-spec.yaml
- 3.2 Postman collection: iteration-vault/04-api-postman.json
- 3.3 API.md (人类可读): iteration-vault/04-api.md
- 3.4 api-registry.md 更新摘要: +N / 改 M / deprecated K
```

---

## 反模式（必拒绝，与 autodev-api 完全对齐）

- ❌ endpoint 脱离 UI 实际需求（**违反 UI 反推 API 铁律**）
- ❌ 推测性字段（PM 没说要但你猜可能需要）
- ❌ 错误处理不完整（只 happy path）
- ❌ 业务规则模糊（必须显性化，不得隐式编进代码）
- ❌ 同资源混用 REST + GraphQL + RPC 两条路径
- ❌ POST 用作 GET
- ❌ 错误响应只看 http status 不带 code
- ❌ 版本写在 query / cookie / header 三处不统一
- ❌ 直接删接口而不 deprecate
- ❌ LLM 输出 API 没 confidence 字段
- ❌ 限流策略写在代码里而非配置
- ❌ 引入商业 API gateway 锁定 vendor
- ❌ 调用方靠"猜返回结构"（必须有 OpenAPI YAML）

命中任一 → reviewer 在对应 4 维度标 ≤ 3 分。

---

## 维护备忘

- 每次发现新反模式追加到本文件
- 每次项目自定协议变化同步项目自定的 API 标准文档
- RFC 9457 / OpenAPI 版本升级时同步本文件引用
