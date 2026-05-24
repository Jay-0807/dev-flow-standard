# Connector: user-feedback

> **作用**：W2 加。从 `iteration-vault/history/*/10.5-user-acceptance.md` 抓"用户最痛"反复出现的项作为候选。
> **weight**：1.5

---

## 数据源

`iteration-vault/history/*/10.5-user-acceptance.md` 文件们。

每个 10.5 报告含：
- 最爱
- 最痛
- 痛点聚类（cluster）

本 connector 仅扫"最痛"和"痛点聚类"段。

---

## 提取规则

1. 列出所有历史迭代的 10.5 报告
2. 对每个报告提取"最痛"段的痛点 + 各 cluster 段的痛点
3. 跨迭代聚合：相同/相似痛点出现 ≥ N 次的算"反复痛点"（N 默认 2）
4. 每个反复痛点变成一个 candidate

---

## 字段映射

| Candidate 字段 | 来源 |
|---|---|
| `id` | `feedback-<sha1(normalized_pain)[:8]>` |
| `source` | `"user-feedback"` |
| `source_ref` | 第一次出现的迭代路径 + 行号 |
| `title` | 痛点归一化后的短文本（≤60 字）|
| `description` | 所有相关 cluster 的合并描述 |
| `raw_evidence` | 多个迭代里的原文 quote |
| `priority_signals.business_impact` | 痛点跨迭代重复次数 → 1-5 |
| `priority_signals.user_pain_freq` | 痛点跨迭代重复次数 → 1-5（同上但满分 5）|
| `priority_signals.technical_complexity` | 默认 M |
| `priority_signals.age_days` | 第一次出现到今天的天数 |
| `priority_signals.confidence` | 0.8（用户证据有但需 PM 翻译）|
| `estimated_size` | 默认 M |
| `tags` | 自动识别（"slow" / "confusing" / "missing" 等）|

---

## 跨迭代去重 / 聚合

痛点常常是"同一个问题不同表述"。需要归一化：

1. 把"看不懂"+"不清楚"+"难理解" 归为同一痛点（用 embedding 或简单关键词）
2. 把"慢"+"卡"+"加载时间长" 归为同一痛点
3. 把"找不到"+"在哪里"+"如何" 归为"导航不清"

实现可用：
- 简单关键词匹配（W2 MVP）
- Embedding 相似度（W4+）

---

## 失败降级

| 情况 | 处理 |
|---|---|
| `iteration-vault/history/` 不存在或为空 | 拉 0 候选，回 IDLE |
| 某 10.5 文件格式不对 | 跳过该文件，写 connector-error.log |
| 所有痛点都只出现 1 次 | 拉 0 候选（不算反复，不进 queue）|

---

## 配置

```yaml
connectors:
  user-feedback:
    enabled: true
    weight: 1.5
    source: "iteration-vault/history/*/10.5-user-acceptance.md"
    min_repeat_count: 2          # 至少 2 次重复才算
    normalization:
      method: "keyword"          # keyword / embedding
      synonyms_file: "~/.autopilot/feedback-synonyms.yaml"  # 可选
```

---

## 输出示例

跨 5 次迭代发现"定价计算页加载慢" 痛点出现 4 次：

```yaml
- id: feedback-7d4e9a3c
  source: user-feedback
  source_ref: "iteration-vault/history/2026-04-12-pricing/10.5-user-acceptance.md#L42"
  title: "定价计算页加载慢"
  description: "用户反复反馈定价页慢，4 次迭代里 4 个用户提到，最近一次评分 2.8/5"
  raw_evidence: |
    iter-2026-04-12: "定价页等了 5 秒，以为坏了"（用户 A）
    iter-2026-04-19: "定价计算太慢"（用户 B）
    iter-2026-05-03: "每次定价都得等"（用户 C）
    iter-2026-05-15: "定价页慢得让人无语"（用户 D）
  priority_signals:
    business_impact: 4   # 4 次重复 → 4/5
    user_pain_freq: 4    # 同上
    technical_complexity: M
    age_days: 42         # 第一次出现到今天
    confidence: 0.8
  estimated_size: medium
  tags: [perf, pricing, ui]
  created_at: 2026-05-23T09:01:30+08:00
```

---

## 维护备忘

- normalization 的关键词列表 4 周后视情况切到 embedding
- min_repeat_count 默认 2，可调（PM 觉得噪声多 → 调 3）
- 跨迭代时间窗口（如仅 90 天内的反馈）后续考虑加
