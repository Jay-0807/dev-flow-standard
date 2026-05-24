# Autopilot Connectors

> **作用**：从各信息源（roadmap / Sentry / GitHub / Notion / user-feedback）拉候选。每个 connector 是独立的 markdown spec。
> **可插拔**：PM 在 `~/.autopilot/config.yaml` enable/disable。

---

## 统一候选格式

每个 connector 输出 candidate 列表，每个 candidate：

```yaml
- id: <stable-uuid>            # 跨轮稳定，用于黑名单 / cooldown
                               # 通常用 source + hash(source_ref + title) 生成
  source: sentry|github|user-feedback|notion|roadmap
  source_ref: <url-or-path>    # 证据链接
  title: <≤60 字>               # 候选简称
  description: <≤300 字>        # 含背景 + 现象 + 期望
  raw_evidence: <full-text>    # 完整原文（给 LLM 判定时引用）
  priority_signals:
    business_impact: 1-5
    user_pain_freq: 1-5
    technical_complexity: S|M|L
    age_days: <int>
    confidence: 0-1
  estimated_size: small|medium|large
  blast_radius_hint:
    affected_files_guess: <int>
    touches_critical_path: bool
  tags: [bug, enhancement, perf, security, ...]
  created_at: <iso8601>
```

---

## Connector 接口约定

每个 connector spec（`.md` 文件）必须含以下段：

```markdown
# <Connector Name>

## 数据源
[读哪些文件 / API / MCP]

## 提取规则
[如何把数据源转为 candidate]

## 字段映射
[每个 candidate 字段从源数据怎么算]

## 失败降级
[如 MCP 不可用怎么办]

## 配置
[config.yaml 里的字段]
```

---

## Connector 列表

| Connector | 启用时机 | 默认 weight |
|---|---|---|
| `roadmap.md` | W1（MVP）| 2.0（加权，PM 显式写）|
| `user-feedback.md` | W2 | 1.5 |
| `github-issues.md` | W3 | 1.0 |
| `sentry.md` | W4 | 1.0 |
| `notion.md` | W4 | 1.0 |

---

## 并行 vs 串行

- HARVEST 阶段不同 connector **并行**（互不依赖）
- 单 connector 内部串行（不要并发 GitHub API）
- 全部完成 → 合并 candidate 列表 → 进 RANKING

---

## 失败降级策略

| 情况 | 处理 |
|---|---|
| 单 connector 失败 | skip 该 connector，写 connector-error.log，继续用其余 |
| 全部 connector 失败 | 用 roadmap.md 兜底（连 roadmap 都没 → SKIPPED_PRE_FLIGHT）|
| 拉到 0 候选 | 写 wake-up log，回 IDLE（不报错，可能是真的没活）|
| 拉到 > 100 候选 | 警告 + 只取 Top 20（防止 RANK 阶段过慢）|

---

## 黑名单过滤

每个 connector 拉到 candidate 后，过滤 `~/.autopilot/blacklist.yaml`：

```yaml
blacklist:
  - candidate_id: candidate-sentry-1234
    reason: "Known false positive, ad blocker triggers it"
    until: 2026-12-31
  - candidate_id: candidate-github-47
    reason: "Blocked on third-party API"
    until: never
```

命中黑名单 → 丢弃 + 写日志。

---

## 增量 vs 全量扫描

每个 connector 必须支持两种模式：

- **全量**（首次 / 状态重置）：扫所有候选
- **增量**（默认）：仅扫上次 last_harvest_at 之后的变更

实现方法：
- Sentry: API 查询 `first_seen > last_harvest_at`
- GitHub: `created_at > last_harvest_at` + `updated_at > last_harvest_at`
- roadmap.md: file mtime > last_harvest_at（如未变，跳过 + 用上次结果）
- user-feedback: 仅扫 last_harvest_at 之后新增的迭代 history

---

## 维护备忘

- 新增 connector 时按本文件 §接口约定写规格
- weight 根据 connector 准确率调（误报率高 → weight 降）
- 失败降级链按实际 MCP 稳定性调
