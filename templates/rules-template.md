# RULES.md 模板（< 80 行硬上限）

> Phase 5.9 文档压缩自动生成 `iteration-vault/RULES.md` 按本模板。

---

```markdown
# Iteration RULES (Hard Constraints)

> 本文件是本迭代的硬约束。所有 Phase 6/7/8/9 实施 phase 必读。
> 违反这些规则的代码 → reviewer 直接 FAIL。

---

## 1. 框架 & 依赖版本（实际填）

- Next.js: 14.x
- React: 18.x
- Prisma: 5.x
- Postgres: 15
- TypeScript: 5.x
- Tailwind: 3.x
- 项目业务协议 SDK: <如有>
- Sentry: 8.x
- Langfuse: 3.x

[其他关键依赖按需追加]

---

## 2. 命名约定

| 类型 | 规则 | 例子 |
|---|---|---|
| 文件 | kebab-case | `user-orders.tsx` |
| 组件 | PascalCase | `UserOrders` |
| 函数 | camelCase | `getUserOrders` |
| DB 表/字段 | snake_case | `user_orders`, `created_at` |
| 项目业务消息 | dot.notation | `order.cancel` |
| 业务码 | UPPER_SNAKE | `YOUR_ERROR_CODE_EXAMPLE` |
| API 路径 | snake_case + 复数 | `/api/v1/user_orders` |

---

## 3. API 鉴权 / 响应标准

- 用户→后端: `Authorization: Bearer <JWT>`
- agent→agent: mTLS + signed envelope
- 第三方→后端: API key + per-key rate limit
- 错误格式: **RFC 9457 Problem Details** 必含 `code` 业务码
- 限流默认: per-user 100/min, per-tenant 1000/min, per-agent 500/min
- 幂等: POST 必支持 `Idempotency-Key`，业务协议消息用 `message_id`（如适用）

---

## 4. 7 Quality Redlines（命中任一 → reviewer FAIL）

| # | 规则 | 触发 |
|---|---|---|
| R1 | 禁占位符 | `TODO/FIXME/HACK/XXX/stub`、空函数体 |
| R2 | 禁 mock 替代 | `mock/dummy/fake` 变量在非测试文件 |
| R3 | 禁降级语言 | "暂时"/"先用"/"for now"/"workaround" |
| R4 | 禁过时技术 | 版本不符 design / deprecated API |
| R5 | OSS 复用优先 | 无 OSS 扫描记录就自研 |
| R6 | 禁 emoji UI icon | JSX text 用 emoji 当图标 |
| R7 | 禁未验证环境断言 | 说"环境没装 X"无 `--version` 输出 |

详见 `gan-engine/quality-redlines.md`。

---

## 5. 禁用模式

- ❌ 直接删 API 接口（必须 deprecate → 6 月 → 真删）
- ❌ LLM 输出 API 没 `confidence` 字段
- ❌ 业务规则硬编码进代码（违反 r4 第 1 维）
- ❌ 跨业务边界共享 DB 表
- ❌ default to full info display（autodev-ui 反模式，**取代 r4 第 4 维**）
- ❌ 同资源混用 REST + GraphQL + RPC 两条路径
- ❌ POST 用作 GET
- ❌ 错误响应不带 code 字段
- ❌ 版本写在 query/cookie/header 三处不统一

---

## 6. r4 哲学（1-3 维保留，第 4 维废除）

- **维度 1 业务规则显性化**: 业务规则进 `business_rule` 表，不进代码
- **维度 2 人工保留点**: 高风险决策 API 必含 `human_review_required: bool`
- **维度 3 数据来源标注**: 
  - LLM 输出必含 `confidence: 0-1`
  - 数据查询必含 `data_source: <表/API 名>`
- ~~维度 4 用户心智隐形~~ **已废除**，UI 层让位 autodev-ui task-first

---

## 7. 本次迭代特殊约束

[每次迭代由 5.9 自动从 PRD/架构提取]

例：
- 本次涉及业务协议升级，所有业务 endpoint 用 v2 信封
- 本次 DB 改动含破坏性 schema 变更，必须有 backward migration
- ...

---

**生成于**: <YYYY-MM-DD HH:mm>
**生成器**: Phase 5.9 compress
**本文件行数**: <X> / 80
```
