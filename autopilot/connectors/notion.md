# Connector: notion

> **作用**：W4 加。从 PM 在 Notion 维护的 backlog database 拉候选。
> **weight**：1.0

---

## 数据源

Notion MCP `mcp__notion-query-database-view`：

```yaml
database_id: <PM 配置>
view_id: <PM 配置，通常是"待开发"视图>
filter:
  status: "待开发"
  priority: ["高", "中"]
```

PM 在 Notion 维护一个 backlog database，字段约定：

| 字段名 | 类型 | 说明 |
|---|---|---|
| Title | title | 候选标题 |
| Status | select | 待开发 / 进行中 / 已完成 |
| Priority | select | 高 / 中 / 低 |
| Size | select | S / M / L |
| User Pain | number | 1-5 |
| Business Impact | number | 1-5 |
| Created | date | |
| Tags | multi-select | |

---

## 提取规则

1. 调 Notion MCP query database view
2. 过滤 `Status = "待开发"` 且 `Priority in [高, 中]`
3. 按 `(business_impact + user_pain)` 排序
4. Top 20 进 candidate

---

## 字段映射

| Candidate 字段 | 来源 |
|---|---|
| `id` | `notion-<page_id_short>` |
| `source` | `"notion"` |
| `source_ref` | Notion page URL |
| `title` | Title 字段 |
| `description` | page body 前 300 字 |
| `raw_evidence` | page 完整 body |
| `priority_signals.business_impact` | Business Impact 字段（PM 显式填）|
| `priority_signals.user_pain_freq` | User Pain 字段（PM 显式填）|
| `priority_signals.technical_complexity` | Size 字段：S → S，M → M，L → L |
| `priority_signals.age_days` | now - Created |
| `priority_signals.confidence` | 0.85（PM 写的，但可能没维护得很新）|
| `estimated_size` | Size 字段 |
| `tags` | Tags 字段 |

---

## 失败降级

| 情况 | 处理 |
|---|---|
| Notion MCP 不可用 | 跳过本 connector，写 connector-error.log |
| database_id 错 | 通知 PM 检查 config |
| view 不存在 | 通知 PM 检查 config |
| 0 候选 | 拉 0 候选 |
| Notion 字段缺失 | 用默认值（business_impact 3 等）|

---

## 配置

```yaml
connectors:
  notion:
    enabled: true
    weight: 1.0
    database_id: "<replace-with-actual>"
    view_id: "<replace-with-actual>"
    filter_status: "待开发"
    filter_priorities: ["高", "中"]
    top_n: 20
    field_mapping:
      business_impact: "Business Impact"
      user_pain: "User Pain"
      size: "Size"
      tags: "Tags"
```

---

## 输出示例

```yaml
- id: notion-7d4e9a3c
  source: notion
  source_ref: "https://notion.so/<your-workspace>/page-id"
  title: "暗黑模式"
  description: "为后台提供暗黑模式切换，覆盖全部页面..."
  raw_evidence: <Notion page full body>
  priority_signals:
    business_impact: 3   # PM 填
    user_pain_freq: 3    # PM 填
    technical_complexity: M
    age_days: 30
    confidence: 0.85
  estimated_size: medium
  tags: [enhancement, ui]
  created_at: 2026-05-23T09:01:30+08:00
```

---

## 维护备忘

- field_mapping 让 PM 改 Notion 字段名时不破坏 connector
- filter 条件根据 PM 实际工作流调
- 大 backlog（> 200 项）时考虑加 view filter 进一步限制
