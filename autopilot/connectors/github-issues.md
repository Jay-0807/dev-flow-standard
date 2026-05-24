# Connector: github-issues

> **作用**：W3 加。用 `gh issue list` 拉 GitHub 上标 bug / enhancement / P0 / P1 的 issue，按 reactions + comments 排序。
> **weight**：1.0

---

## 数据源

GitHub API via `gh` CLI：

```bash
gh issue list \
  --repo <your-org>/<your-repo> \
  --state open \
  --label "bug,enhancement,P0,P1" \
  --json number,title,body,labels,reactions,comments,createdAt,updatedAt \
  --limit 50
```

---

## 提取规则

1. 拉所有标 `bug` / `enhancement` / `P0` / `P1` 的 open issue
2. 按 `(reactions.thumbs_up + reactions.heart) * 2 + comments` 排序
3. Top 20 进 candidate 列表

---

## 字段映射

| Candidate 字段 | 来源 |
|---|---|
| `id` | `github-<issue_number>` |
| `source` | `"github-issues"` |
| `source_ref` | `https://github.com/.../issues/<num>` |
| `title` | issue.title |
| `description` | issue.body 前 300 字 |
| `raw_evidence` | issue.body + 顶层 comments |
| `priority_signals.business_impact` | reactions thumbsup + heart → 1-5 |
| `priority_signals.user_pain_freq` | unique commenters → 1-5 |
| `priority_signals.technical_complexity` | label 含 "complex" → L；"easy" → S；其他 M |
| `priority_signals.age_days` | now - createdAt |
| `priority_signals.confidence` | 0.9（GitHub issue 一般具体）|
| `estimated_size` | label 含 "epic" → large；"chore" → small；其他 medium |
| `tags` | issue.labels |

---

## 失败降级

| 情况 | 处理 |
|---|---|
| `gh` 未登录 | 写 connector-error.log，跳过本 connector |
| GitHub API 限流 | 等 60s 重试 1 次；仍失败 → 跳过 |
| 没有匹配 issue | 拉 0 候选 |
| 仓库不存在 | 跳过本 connector + 通知 PM 检查 config |

---

## 配置

```yaml
connectors:
  github-issues:
    enabled: true
    weight: 1.0
    repo: "<your-org>/<your-repo>"
    labels: ["bug", "enhancement", "P0", "P1"]
    state: "open"
    limit: 50
    top_n: 20                    # 取 Top 20 进 candidate
```

---

## 输出示例

```yaml
- id: github-47
  source: github-issues
  source_ref: "https://github.com/<your-org>/<your-repo>/issues/47"
  title: "后台搜索响应太慢，超过 3 秒"
  description: "搜索框输入后等 3-5 秒才返回结果。用 lighthouse 测..."
  raw_evidence: <issue body + top 3 comments>
  priority_signals:
    business_impact: 4           # 8 thumbsup + 2 heart = 10 → 4/5
    user_pain_freq: 4            # 12 unique commenters → 4/5
    technical_complexity: M
    age_days: 7
    confidence: 0.9
  estimated_size: medium
  tags: [bug, P1, perf]
  created_at: 2026-05-23T09:01:30+08:00
```

---

## 维护备忘

- label 列表可在 config.yaml 调
- 排序公式（reactions × 2 + comments）4 周后视情况调权重
- 加 `--search` 支持后可只看含某关键词的 issue
