# 🛡️ 上线前质量检查违规清单模板（带编号）

> 内部编号：Phase 10 五层验收的违规清单模板
> **红线定义唯一来源**：`gan-engine/quality-redlines.md`（SSOT）。本文件只定义**编号格式**和**模板结构**，不复述红线内容。
> 输出位置：`iteration-vault/10-verification-redlines.md`

---

```markdown
# Phase 10 Verification Redlines (Numbered)

**生成时间**: <YYYY-MM-DD HH:mm>
**第 X 次跑**: <retry attempt>
**总判**: PASS / FAIL
**自动回退到**: Phase <N>

---

## 总览

| 层 | 状态 | 违规数 |
|---|---|---|
| L1 契约 | ✅/❌ | <N> |
| L2 红线 | ✅/❌ | <N> critical / <M> major |
| L3 静态 | ✅/❌ | <N> |
| L4 运行时 | ✅/❌ | <N> |
| L5 acceptance | ✅/❌ | <N> |

---

## L1 契约违规

| 编号 | 位置 | 描述 | 关联 PRD AC |
|---|---|---|---|
| L1-001 | 06-task-breakdown.md #5 | acceptance_criteria #3 未满足: "用户登录后看到上次保存的草稿" | PRD §6 AC #3 |
| L1-002 | 04-api-spec.yaml + impl | GET /users/:id 返回 schema 缺 confidence 字段 | PRD §7.3 LLM 元数据字段（仅 AI 原生项目）|

---

## L2 红线违规（按 Quality Redlines 编号）

### R1 占位符（critical）
| 编号 | 位置 | 详情 |
|---|---|---|
| L2-R1-001 | src/api/orders/cancel.ts:42 | 含 `// TODO: handle edge case` |
| L2-R1-002 | src/lib/payment/refund.ts:18 | 空函数体 `async refund() {}` |

### R2 mock 替代（critical）
| 编号 | 位置 | 详情 |
|---|---|---|
| L2-R2-001 | src/lib/llm/pricing.ts:18 | 用 `mockLLMResponse` 在生产代码 |
| L2-R2-002 | src/components/OrderTable.tsx:34 | 用 `fakeOrders` 渲染（应调真 API）|

### R3 降级语言（critical）
| 编号 | 位置 | 详情 |
|---|---|---|
| L3-R3-001 | src/api/users.ts:50 注释 | 含"// 先用简化版" |
| L3-R3-002 | 02-PRD.md §5 | 含"暂时不做权限校验" |

### R4 过时技术（major）
| 编号 | 位置 | 详情 |
|---|---|---|
| L2-R4-001 | package.json | react ^17.0.0 但 design 标 18 |

### R5 OSS 复用（major）
| 编号 | 位置 | 详情 |
|---|---|---|
| L2-R5-001 | src/lib/utils/date.ts | 自研 date formatter, 应用 date-fns |

### R6 emoji UI icon（major）
| 编号 | 位置 | 详情 |
|---|---|---|
| L2-R6-001 | src/components/Header.tsx:12 | `<span>🔥 推荐</span>` 应用 icon 库 |

### R7 未验证环境断言（minor）
| 编号 | 位置 | 详情 |
|---|---|---|
| L2-R7-001 | 02-PRD.md §9 | "假设环境有 Docker" 无验证证据 |

> ⚠️ **R8 / R9 不是通用版默认红线**。通用版（universal skill）只启用 Quality Redlines **R1–R7**（见 `gan-engine/quality-redlines.md`）。R8 仅适用 AI 原生项目，R9 仅有自定协议的项目；非对应项目不启用本两节。

### R8 LLM API 元数据三字段缺失（仅适用 AI 原生项目；通用版默认不启用）
| 编号 | 位置 | 详情 |
|---|---|---|
| L2-R8-001 | GET /api/v1/llm/pricing 实现 | 返回缺 confidence 字段 |
| L2-R8-002 | POST /api/v1/llm/suggest | 缺 data_source 字段 |

### R9 项目自定协议合规（如有）（仅有自定协议的项目；通用版默认不启用）
| 编号 | 位置 | 详情 |
|---|---|---|
| L2-R9-001 | order.cancel 自定协议实现 | 消息信封缺 trace_id 字段 |

---

## L3 静态违规

### TypeScript 错误
| 编号 | 位置 | 错误 |
|---|---|---|
| L3-001 | src/components/OrderTable.tsx:34 | TS7006: Parameter 'item' implicitly has 'any' type |
| L3-002 | src/api/users.ts:12 | TS6196: 'unusedVar' is declared but never read |

### ESLint 错误
| 编号 | 位置 | rule |
|---|---|---|
| L3-101 | src/lib/parse.ts:22 | no-unused-vars |

### 测试失败
| 编号 | 测试文件 | 描述 |
|---|---|---|
| L3-201 | src/api/orders.test.ts | "should cancel order" 失败：expected 200 got 404 |

### 依赖审计
| 编号 | 包 | 漏洞 |
|---|---|---|
| L3-301 | lodash@4.17.20 | high: prototype pollution |

---

## L4 运行时违规

| 编号 | 位置 | 描述 |
|---|---|---|
| L4-001 | GET /api/v1/orders | 返回 500（reproducible）|
| L4-002 | /orders 页面 | loading 状态未实现（应有 spinner）|
| L4-003 | 用户旅程"下单 → 支付" | 第 3 步跳转到 404 |
| L4-004 | DB transaction | 创建订单事务回滚后库存未恢复 |

---

## L5 acceptance 违规

| 编号 | PRD AC | 描述 |
|---|---|---|
| L5-001 | PRD AC #4 "下单后立即扣库存" | 集成测试失败：库存延迟 2s 才扣 |
| L5-002 | PRD AC #7 "支付失败 24h 内不重复扣款" | 测试发现 1h 内可重复 |

---

## 自动回退决策

按本次违规分布：
- L2-R1/R2/R3 critical = 5 项 → 回 Phase 7 修代码
- L4 runtime = 4 项 → 回 Phase 7 修代码
- L5 acceptance = 2 项 → 回 Phase 7 修代码

**autonomous 行动**：
1. 自动回到 Phase 7 spawn agent 修每项违规
2. 修完后自动重跑 Phase 10 全部 5 层
3. 第 N 次重试（最多 3 次，本次是第 X 次）

如第 3 次仍 FAIL → 触发 R3 红线升级 PM。

---

## 与 RULES.md 联动

每条违规可追溯到 `iteration-vault/RULES.md`：
- L2-R1 → RULES.md §4 R1 占位符
- L2-R8 → RULES.md §6 LLM API 元数据三字段（仅 AI 原生项目）
- L2-R9 → RULES.md §3 项目自定消息格式（如有）

PM 在 Phase 12.5 早晨复盘可逐条查违规 + RULES 来源。

---

## 给开发者的修复指引（按编号给具体修复方法）

### L2-R1-001 修复
- 删除 src/api/orders/cancel.ts:42 的 `// TODO: handle edge case`
- 实现该 edge case 的真实处理

### L2-R2-001 修复
- 删除 src/lib/llm/pricing.ts:18 的 mockLLMResponse
- 接入真 LLM API（按 04-architecture-and-api.md §1.2.X）

[每条违规给具体修复指引]

---

**总判定**: FAIL
**自动回退到**: Phase 7
**重试次数**: <X> / 3
**下次重试启动时间**: <iso8601>
```
