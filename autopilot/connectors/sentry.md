# Connector: sentry

> **作用**：W4 加。用 Sentry MCP `search_issues` 拉 unresolved 错误 ≥ 3 天的 Top 20。
> **weight**：1.0

---

## 数据源

Sentry MCP `mcp__sentry__search_issues`：

```yaml
query: "status:unresolved age:>3d level:[error,fatal]"
organization_slug: "<your-org>"
sort: "events"     # 按事件数排序
limit: 20
```

---

## 提取规则

1. 调 Sentry MCP `search_issues`
2. 按 `event_count + users_affected` 排序（Sentry 自带）
3. 过滤已在黑名单的（如已知 false positive）
4. Top 20 进 candidate 列表

---

## 字段映射

| Candidate 字段 | 来源 |
|---|---|
| `id` | `sentry-<issue_short_id>`（如 `PROJ-1234`）|
| `source` | `"sentry"` |
| `source_ref` | Sentry issue URL |
| `title` | issue.title（异常类型 + 文件路径）|
| `description` | issue.culprit + 简短堆栈 |
| `raw_evidence` | 异常详情 + 前 3 个 event 的 context |
| `priority_signals.business_impact` | event_count / users_affected → 1-5 |
| `priority_signals.user_pain_freq` | unique users affected → 1-5 |
| `priority_signals.technical_complexity` | 默认 M（运行时错误一般可定位）|
| `priority_signals.age_days` | now - first_seen |
| `priority_signals.confidence` | 0.95（Sentry 是真实生产错误）|
| `estimated_size` | event_count > 1000 → large；> 100 → medium；其他 small |
| `tags` | issue.tags + `[bug, sentry]` |

---

## 失败降级

| 情况 | 处理 |
|---|---|
| Sentry MCP 不可用 | 跳过本 connector，写 connector-error.log |
| Sentry API 限流 | 等 60s 重试 1 次 |
| organization_slug 错 | 通知 PM 检查 config |
| 0 个 unresolved issue | 拉 0 候选（好事）|

---

## 配置

```yaml
connectors:
  sentry:
    enabled: true
    weight: 1.0
    organization_slug: "<your-org>"
    query: "status:unresolved age:>3d level:[error,fatal]"
    sort: "events"
    limit: 20
    impact_thresholds:
      events_high: 1000   # > 1000 → impact 5
      events_medium: 100  # > 100 → impact 3
```

---

## 输出示例

```yaml
- id: sentry-PROJ-1234
  source: sentry
  source_ref: "https://sentry.io/.../issues/PROJ-1234"
  title: "TypeError: Cannot read property 'price' of undefined in src/checkout/CheckoutPage.tsx:42"
  description: "在 CheckoutPage.tsx:42 反复触发 TypeError, 影响 checkout flow"
  raw_evidence: <stacktrace + 3 sample events>
  priority_signals:
    business_impact: 4   # 500 events / 80 users → impact 4
    user_pain_freq: 4    # 80 unique users → 4/5
    technical_complexity: S  # null 检查问题，简单
    age_days: 7
    confidence: 0.95
  estimated_size: small
  tags: [bug, sentry, checkout, typescript]
  created_at: 2026-05-23T09:01:30+08:00
```

---

## 维护备忘

- query 过滤可调（如增加 `release:>v1.5` 仅看新版错误）
- impact_thresholds 根据项目实际 traffic 量调
- Sentry MCP 工具新增能力时同步本文件
