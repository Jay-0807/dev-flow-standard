# RULES.md 模板（< 80 行硬上限）

> Phase 5.9 文档压缩自动生成 `iteration-vault/RULES.md` 按本模板。

---

```markdown
# Iteration RULES (Hard Constraints)

> 本文件是本迭代的硬约束。所有 Phase 6/7/8/9 实施 phase 必读。
> 违反这些规则的代码 → reviewer 直接 FAIL。

---

## 1. 框架 & 依赖版本（启动时检测填入）

- 前端框架: <前端框架+版本>
- ORM: <ORM+版本>
- 数据库: <数据库+版本>
- 语言: <语言+版本>
- 样式方案: <如有，框架+版本>
- 项目自定协议/RPC SDK: <如有>
- 错误监控: <如有，工具+版本>
- LLM 可观测性: <如有，工具+版本>

[其他关键依赖按需追加]

---

## 2. 命名约定

| 类型 | 规则 | 例子 |
|---|---|---|
| 文件 | kebab-case | `user-orders.tsx` |
| 组件 | PascalCase | `UserOrders` |
| 函数 | camelCase | `getUserOrders` |
| DB 表/字段 | snake_case | `user_orders`, `created_at` |
| 项目自定消息（如有）| dot.notation | `resource.action` |
| 业务码 | UPPER_SNAKE | `YOUR_ERROR_CODE_EXAMPLE` |
| API 路径 | snake_case + 复数 | `/api/v1/user_orders` |

---

## 3. API 鉴权 / 响应标准

- 用户→后端: `Authorization: Bearer <JWT>`
- 服务间（如有自定协议）: mTLS + signed envelope；否则用标准 REST / gRPC / 事件
- 第三方→后端: API key + per-key rate limit
- 错误格式: **RFC 9457 Problem Details** 必含 `code` 业务码
- 限流默认: per-user 100/min, per-tenant 1000/min, per-service 500/min
- 幂等: POST 必支持 `Idempotency-Key`，自定协议消息用 `message_id`（如适用）

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
- ❌ 业务规则硬编码进代码（项目合规要求，如有）
- ❌ 跨业务边界共享 DB 表
- ❌ default to full info display（autodev-ui 反模式，UI 层 task-first）
- ❌ 同资源混用 REST + GraphQL + RPC 两条路径
- ❌ POST 用作 GET
- ❌ 错误响应不带 code 字段
- ❌ 版本写在 query/cookie/header 三处不统一

---

## 6. 项目合规要求（如有，按项目实际填）

[以下为可选合规维度示例；无对应要求的项目可删除本节]

- **业务规则显性化**（通用工程实践）: 业务规则进 `business_rule` 表，不进代码
- **人工保留点**（通用工程实践）: 高风险决策 API 必含 `human_review_required: bool`
- **数据来源标注**（通用工程实践）: 数据查询必含 `data_source: <表/API 名>`
- **LLM API 元数据三字段（置信度 / 人工复核标记 / 数据来源）**（仅适用 AI 原生项目，非 AI 项目不强制）:
  - LLM 输出必含 `confidence: 0-1`
  - 高风险 LLM 决策必含 `human_review_required: bool`
  - LLM 响应必含 `data_source: <模型/表名>`
- **UI 层**: task-first，避免 default-to-full-info（让位 autodev-ui）

---

## 7. 本次迭代特殊约束

[每次迭代由 5.9 自动从 PRD/架构提取]

例：
- 本次涉及项目自定协议升级（如有），所有相关 endpoint 用 v2 信封
- 本次 DB 改动含破坏性 schema 变更，必须有 backward migration
- ...

---

**生成于**: <YYYY-MM-DD HH:mm>
**生成器**: Phase 5.9 compress
**本文件行数**: <X> / 80
```
