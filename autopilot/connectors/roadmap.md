# Connector: roadmap

> **作用**：W1 MVP 必备 connector。读 PM 手维护的 `~/.autopilot-data/product-roadmap.md`，把"下一轮要做"清单转候选。
> **weight**：2.0（最高，因为 PM 显式写）

---

## 数据源

`~/.autopilot-data/product-roadmap.md`（PM 手维护的 markdown）。

格式约定：

```markdown
# Firefly 产品 Roadmap

## 下一轮要做
- [P0] AI 选品建议加二阶反馈（用户最痛）
- [P0] 订单批量导出
- [P1] 后台搜索体验改进
- [P2] 暗黑模式
- [P2] 国际化

## 已完成
- ...

## 长远规划
- ...
```

---

## 提取规则

- 仅扫"## 下一轮要做"段下的 bullet `- [PX] xxx`
- 解析格式：`- [优先级] 一句话描述`
  - 优先级：P0 / P1 / P2 / P3
  - 描述：候选标题
- 已完成段 + 长远规划段不扫

---

## 字段映射

| Candidate 字段 | 来源 |
|---|---|
| `id` | `roadmap-<sha1(title)[:8]>` |
| `source` | `"roadmap"` |
| `source_ref` | `~/.autopilot-data/product-roadmap.md#L<line>` |
| `title` | bullet 文本去 `[PX]` 前缀 |
| `description` | 同 title（roadmap 短不展开）|
| `raw_evidence` | bullet 原文 |
| `priority_signals.business_impact` | P0→5, P1→4, P2→3, P3→2 |
| `priority_signals.user_pain_freq` | 默认 3（roadmap 不含频次信息）|
| `priority_signals.technical_complexity` | 默认 M（roadmap 不含复杂度）|
| `priority_signals.age_days` | bullet 首次出现到今天的天数（需 git blame）|
| `priority_signals.confidence` | 1.0（PM 显式写的）|
| `estimated_size` | 默认 M（PM 可在 bullet 后 `[size: S/M/L]` 标）|
| `blast_radius_hint.affected_files_guess` | 跑 grep 估算 |
| `blast_radius_hint.touches_critical_path` | 跑 grep 估算 |
| `tags` | 从 bullet 自动识别（含"bug" / "perf" / "ui" 等关键词）|
| `created_at` | 当前时间戳 |

---

## age_days 计算

```bash
# 从 git blame 找该 bullet 首次出现的 commit 日期
git -C ~/.autopilot-data blame -L <line>,<line> product-roadmap.md
# 提取 commit 日期，算到今天的天数
```

如 `~/.autopilot-data` 不是 git 仓库（PM 可能不 commit）：
- 用文件 mtime 作 fallback
- 不够准但够用

---

## 高级语法（PM 可选标）

PM 在 bullet 后可加标注影响评分：

```markdown
- [P0] AI 选品建议加二阶反馈 [size:M] [users:5] [age:21d]
```

| 标注 | 影响字段 |
|---|---|
| `[size:S/M/L]` | `estimated_size` |
| `[users:N]` | `priority_signals.user_pain_freq`（覆盖默认 3）|
| `[freq:N]` | 同上 |
| `[age:Xd]` | `priority_signals.age_days`（覆盖 git blame）|
| `[impact:1-5]` | `priority_signals.business_impact`（覆盖 P0 默认）|
| `[blast:N]` | `blast_radius_hint.affected_files_guess`（绕过估算）|

PM 不写标注也能跑（用默认）。

---

## 失败降级

| 情况 | 处理 |
|---|---|
| `~/.autopilot-data/product-roadmap.md` 不存在 | 第一次跑时写一份模板 + INBOX 通知 PM"请维护 roadmap" |
| 文件存在但没有 "## 下一轮要做" 段 | 拉到 0 候选，回 IDLE |
| 文件格式坏 | 写 connector-error.log，跳过本 connector |
| Bullet 解析失败 | 跳过该 bullet，继续其余 |

---

## 配置（config.yaml）

```yaml
connectors:
  roadmap:
    enabled: true
    weight: 2.0
    path: "~/.autopilot-data/product-roadmap.md"
    section: "下一轮要做"          # 仅扫此段（PM 可改）
    fallback_section: "TODO"        # 如 PM 用英文标题
```

---

## 输出示例

```yaml
- id: roadmap-a3f5b8d2
  source: roadmap
  source_ref: "~/.autopilot-data/product-roadmap.md#L5"
  title: "AI 选品建议加二阶反馈"
  description: "AI 选品建议加二阶反馈"
  raw_evidence: "- [P0] AI 选品建议加二阶反馈（用户最痛）"
  priority_signals:
    business_impact: 5         # P0 → 5
    user_pain_freq: 3          # roadmap 默认
    technical_complexity: M    # roadmap 默认
    age_days: 21               # git blame 算出
    confidence: 1.0            # roadmap 默认（PM 显式写）
  estimated_size: medium
  blast_radius_hint:
    affected_files_guess: 8    # grep 估算
    touches_critical_path: false
  tags: [enhancement, ai]      # 从"AI 选品"识别
  created_at: 2026-05-23T09:01:30+08:00
```

---

## 维护备忘

- 解析失败时不要 crash，写日志继续
- PM 调标注（如新增 [strategic:true]）时本文件加映射
- age 算法 4 周后视情况切到更精确（如解析 git log 含日期注释）
