# Slash Commands 全清单

> universal-coding-project-development-skill 的所有外部触发命令。
> autopilot 命令独立列出（autopilot 是另一条入口，跟标准开发路径分开）。

---

## 标准开发路径（默认）

| 命令 | 等价于 | 用途 |
|---|---|---|
| `/u` | SKILL.md 完整 17 阶段 | 跑完整流程 |
| `/u "需求"` | 同上 + 直接注入需求 | 一句话启动 |

**唯一默认路径**：PM 一句话需求 → Step 0 反问 3 件事 → 💬 需求澄清 → 👥 用户研究 → 📝 PRD ⛳ 关卡 1 → 自动跑到底 → ☀️ 早晨复盘 ⛳ 关卡 2。

不存在 fast-track / standard / manual 三种模式选项 — 就这一条路径。

---

## 跑单一步骤（独立可触发，不依赖前置 vault）

| 命令 | 对应业务步骤 | 用途 |
|---|---|---|
| `/u-prd` | 📝 PRD 撰写 | PoC 期 / 只想出文档不写代码 |
| `/u-gan <task>` | AI 互审引擎 | 对任意单一产出跑对抗审查 |
| `/u-verify [--quick] [--layer <N>]` | 🛡️ 上线前质量检查 | 对当前代码跑五层验收 |
| `/u-sync` | 🔄 文档代码漂移检测 | 检查代码 vs 设计是否对得上 |
| `/u-debt` | 🧹 代码债扫描 | 9 维度扫一遍代码质量 |
| `/u-compress` | 📚 设计压缩 | 把现有厚设计文档压成开发地图（INDEX + RULES）|
| `/u-replay <decision-N>` | 决策回放 | 从某个 AI 自治决策点重跑下游 phase |

### `/u-verify` 选项详解

```
/u-verify                       # 默认 L1-L5 全跑
/u-verify --quick               # 仅 L1-L3（轻量，开发中用）
/u-verify --layer contract      # 仅 L1 契约层
/u-verify --layer static        # 仅 L3 静态层
/u-verify --layer runtime       # 仅 L4 运行时层
```

### 单跑模式 vs 流水线模式

- **流水线模式**（被主 SKILL.md 触发）：写到 `iteration-vault/<run-id>/<NN>-*.md`
- **单跑模式**（PM 用 `/u-XXX` 直接触发）：
  - 没有 iteration-vault/ → 降级跑（如 L1 跳过）
  - 输出到当前目录 `.u-<cmd>-report-<timestamp>.md`
  - 任何 FAIL → 同时写 DEBUG-TRACE.md

---

## Autopilot 命令（独立入口，不是标准路径）

> Autopilot = AI 主动从 roadmap / 反馈 / GitHub issues / Sentry error / Notion 捡候选 → 排序 → 等 PM 批准 → 跑标准开发路径。
> 跟标准路径**入口完全不同**（cron tick vs PM 主动），但**实现共享**（汇合在 📝 PRD ⛳ → 自动跑到底 → ☀️ 早晨复盘）。

| 命令 | 用途 |
|---|---|
| `/autopilot-status` | 查看 queue + 状态 + 最近 7 天 history |
| `/autopilot-queue` | 打开 queue.md 看候选清单 |
| `/autopilot-pause` | 暂停（保留状态，可恢复）|
| `/autopilot-resume` | 恢复暂停 |
| `/autopilot-stop` | 紧急停止 |
| `/autopilot-reset` | 清除 ERROR 状态 |
| `/autopilot-tier 1\|2\|3` | 切换 autopilot tier |
| `/autopilot-blacklist <id>` | 拉黑某候选 |

### 自然语言触发（除 slash 命令外也接受）

- "autopilot pick #N 启动" → 等价于 PM 选某候选进 RUNNING
- "暂停 autopilot" → `/autopilot-pause`
- "切到 Tier 2" → `/autopilot-tier 2`
- "看下 autopilot 在跑什么" → `/autopilot-status`

详见 `autopilot/pm-oversight.md`。

---

## 命令安装位置

每个 `/u-*` 命令需要在 `.claude/commands/` 下有对应转发文件：

```
.claude/commands/
  u.md              → 转发到 SKILL.md 完整流程
  u-prd.md          → 转发到 phases/02-prd.md standalone 段
  u-gan.md          → 转发到 gan-engine/ standalone 段
  u-verify.md       → 转发到 phases/10-verify.md standalone 段
  u-sync.md         → 转发到 phases/11.5-sync.md standalone 段
  u-debt.md         → 转发到 phases/08-tech-debt.md standalone 段
  u-compress.md     → 转发到 phases/05.9-compress.md standalone 段
  u-replay.md       → 转发到 decision-replay.md standalone 段
```

每个 phase 文件末尾有 §Standalone 模式段，说明该步骤如何独立触发、降级行为、输出位置。

---

## 维护备忘

- 新加 phase 想开 standalone 命令 → 改本文件 + 在该 phase 文件末尾加 §Standalone 段 + 加 `.claude/commands/u-XXX.md` 转发
- autopilot 命令清单跟 `autopilot/pm-oversight.md` 同步
- 命令命名约定：`/u-<动词>` 或 `/u-<业务简称>`（如 verify / sync / debt / compress / prd），不超过 8 字符
