# DEBUG TRACE for run {{run-id}}

> 任何错误 / FAIL / escalation / retry / PIVOT 都必须按 E001, E002, E003... 编号顺序写入本文件。
> RUN-LOG.md 在对应业务名行用 ❌ 或 🔄 或 🚨 标记 + 指向本文件的 anchor（E[NNN]）。
> PM 早上扫 RUN-LOG.md 后，按需 hot-jump 到此处看错误链详情。

---

## E001 [{{时间戳}}] {{业务名 emoji}} → {{结果摘要}}

- **内部位置**: {{phases/<NN>-<name>.md 或 gan-engine/<file>.md 或 night-mode.md §X}}
- **触发检测器**: {{quality-redlines.md R[X] / verify L[N] / GAN reviewer / R[X] escalation}}
- **命中内容**:
  ```
  {{具体违规：file:line + 代码片段}}
  ```
- **关联红线编号**: {{L2-R3-001 / R1 子检查 / 无}}
- **当前重试**: {{N/3}}
- **自动行动**:
  - 已回退到: {{业务名}}（{{phases/XX 文件}}）
  - 修复 prompt 路径: `iteration-vault/<run-id>/.debug-prompts/E001-fix.md`
  - 预计下次尝试: {{HH:MM}}
- **解决状态**: {{⏸ 待 retry / 🔄 retry 中 / ✅ 已解决 (耗时 X 分钟) / 🚨 升级 R[X] 等 PM}}
- **PM 早上需关注**: {{1 行总结，便于 ☀️ 早晨复盘扫}}

---

## E002 [{{时间戳}}] {{业务名 emoji}} → {{结果摘要}}

（同上结构 ...）

---

## 错误类型分类（按业务名 emoji 索引）

| 业务名 | 错误编号 |
|---|---|
| 📝 PRD 撰写 | - |
| 💡 方案发散 | - |
| 🏗️ 架构与接口设计 | E[X], E[Y] |
| 🎨 界面设计 | - |
| ⌨️ 代码实施 | E[Z] |
| 🔍 多路审查 | - |
| 🛡️ 上线前质量检查 | E[A], E[B], E[C] |
| 🔄 漂移检测 | - |

## 红线编号 → 错误编号映射

| 红线 | 错误编号 |
|---|---|
| L2-R1-001 (占位符) | E[X] |
| L2-R2-001 (mock 替代) | E[Y] |
| L4-001 (运行时 API 500) | E[Z] |

## PIVOT 记录

| PIVOT 发生位置 | 错误编号 | stash 路径 |
|---|---|---|
| 💡 方案发散 GAN Round 3 | E[X] | `iteration-vault/02.5-gan/pivot-archive/pre-pivot-3/` |

## R 红线 escalation 记录

| 红线 | 错误编号 | escalation 文件 |
|---|---|---|
| R1 重大架构冲突 | E[X] | ESCALATION-R1.md |
| R3 验收 3 次重试仍挂 | E[Y] | ESCALATION-R3.md |

---

## 维护备忘

- 错误编号 E001+ **单调递增**，跨 session 不重置（context 耗尽后接续也用相同序列）
- 每个错误**必须**写到本文件 + RUN-LOG.md 对应行加 ❌ 标记
- 解决后**不删**错误条目，只更新"解决状态"字段（保留历史用于 PM 复盘）
- PIVOT / Escalation 必须双写：本文件 + 各自专用文件（pivot-archive / ESCALATION-R*.md）
