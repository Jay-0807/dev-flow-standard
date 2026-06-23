# Autopilot — 项目自动驾驶 (W1-W4 全套)

> **作用**：让 本 skill **不靠 PM 主动触发**，定时从 backlog 自选下一迭代点、自启 Phase 0、跑完 14 阶段、Phase 12.5 等 PM 复盘。
> **定位**：Sibling Cycle（与 14-phase 主流水线解耦），通过 `iteration-vault/00-intake.md` 注入种子触发主流水线。

---

## PM 视角的 5 句话

1. autopilot 是一条**和主流水线平行的"唤醒循环"**——每天 09:00 扫描真实世界状态，决定下一轮该做什么
2. **5 个 connector**（roadmap / user-feedback / GitHub / Sentry / Notion）各拉候选；**统一格式 + 评分公式**排序；输出到 `~/.autopilot/queue.md`
3. **3 档触发**（Tier 1/2/3）控制 PM 时间投入——起步 Tier 1（每天 10 秒决策），跑 4 周后视情况升级
4. **6 层安全闸**（kill switch / 预算 / blast radius / 连续失败 / 工作时段 / pre-flight）层层兜底；4 红线机制照常生效
5. **PM 起步只需做 3 件事**：
   - 写一份 `~/.autopilot-data/product-roadmap.md`
   - 用 `/schedule` 配 09:00 cron
   - 每天扫一眼 `~/.autopilot/INBOX.md` 决定跑不跑

---

## 首次设置（PM 一次性，~10 分钟）

### Step 1：创建配置 + 数据目录

```bash
# 假设 PM 在 项目 git repo 根目录
mkdir -p ~/.autopilot
mkdir -p ~/.autopilot/logs

# 复制 autopilot/config.yaml 模板到运行时位置
cp <this-skill-path>/autopilot/config.yaml ~/.autopilot/config.yaml
```

### Step 2：维护 product-roadmap.md

PM 在 `~/.autopilot-data/product-roadmap.md` 写下"下一轮要做"清单：

```markdown
# 产品 Roadmap

## 下一轮要做
- [P0] 给导出功能加二阶反馈（用户最痛）
- [P0] 列表批量导出（12.5 早晨复盘反复反馈）
- [P1] 后台搜索体验改进
- [P2] 暗黑模式
- [P2] 国际化（多语言）
```

格式：`- [优先级] 一句话描述`。优先级 P0/P1/P2（用于评分）。

### Step 3：用 /schedule 配 cron

```bash
# 在 Claude Code 内用 /schedule 命令
/schedule autopilot daily at 09:00
```

或手动 cron（如果 PM 偏好系统 cron）：
```cron
0 9 * * 1-5 cd /path/to/your/project && claude --resume autopilot-wake-up
```

### Step 4：首次验证

```bash
# 触发一次 autopilot 唤醒
claude "/autopilot-status"
```

应看到：
- state.json 已创建
- queue.md 含 1-3 个候选
- INBOX.md 含今日推荐

---

## PM 日常操作（4 个 slash 命令足够）

| 命令 | 行为 |
|---|---|
| `/autopilot-status` | 查看当前 queue + 状态 + 最近 7 天 run-history |
| `/autopilot-pause` | 立刻暂停（`touch ~/.autopilot/PAUSE`）|
| `/autopilot-resume` | 恢复（`rm ~/.autopilot/PAUSE`）|
| `autopilot pick #N 启动` | 自然语言确认今晚跑哪个候选 |

---

## 内部架构（系统图）

```
┌─────────────────────────────────────────────────────────────────────┐
│                AUTOPILOT WAKE-UP LOOP (Sibling Cycle)                │
│                                                                       │
│   ⏰ Cron daily 09:00 / /loop / /schedule                             │
│           │                                                           │
│           ▼                                                           │
│   ┌──────────────────┐    ┌──────────────────┐                       │
│   │ Pre-flight Check │ ─→ │ FAIL → 跳过本轮  │ → 记日志 → idle       │
│   └──────┬───────────┘    └──────────────────┘                       │
│          │ PASS                                                       │
│          ▼                                                            │
│   ┌──────────────────────────────────────────┐                       │
│   │  HARVEST (从 N 个 connectors 拉候选)      │                       │
│   │  ┌──────┐ ┌──────┐ ┌──────┐ ┌────────┐  │                       │
│   │  │Sentry│ │GitHub│ │ User │ │ Notion │ + roadmap                 │
│   │  └──────┘ └──────┘ │Feedb.│ └────────┘                           │
│   └──────────┬───────────────────────────────┘                       │
│              ▼                                                        │
│   ┌──────────────────────┐                                           │
│   │  RANK (优先级算法)    │ → ~/.autopilot/queue.md          │
│   └──────────┬───────────┘                                           │
│              ▼                                                        │
│   ┌──────────────────────────┐                                       │
│   │  PICK TOP CANDIDATE       │                                      │
│   │  + 估算 blast radius      │                                      │
│   │  + 估算所需 phase         │                                      │
│   └──────────┬───────────────┘                                       │
│              │                                                        │
│              ▼                                                        │
│   ┌──────────────────────────────────────────────────────┐           │
│   │  TIER 判定 (Tier 1 / 2 / 3 + 安全闸检查)             │           │
│   └──┬────────────────┬────────────────┬────────────────┘           │
│      │ Tier 1         │ Tier 2 (混合)  │ Tier 3 (自动)              │
│      ▼                ▼                ▼                              │
│   PM 显式确认 → → → 中等候选问 PM → → → 直接启动                     │
│   小候选自动启动      大候选问 PM                                    │
│                                                                       │
└──────────┬──────────────────────────────────────────────────────────┘
           │ 选定种子 → GAN 引擎生成 PRD 草稿 → 注入 iteration-vault/00-intake.md
           ▼
┌─────────────────────────────────────────────────────────────────────┐
│              MAIN 14-PHASE PIPELINE (现有，零修改)                   │
│  Phase 0 (auto-seeded) → Phase 1 → 1.5 → 2 ⛳PRD                     │
│  └─ PM 在此决策 (或 Tier 3 下 autopilot 自批准小候选)               │
│                       │                                              │
│                       ▼                                              │
│              Phase 2.5 → ... → Phase 12  (autonomous + 4 红线)       │
│                       │                                              │
│                       ▼                                              │
│              Phase 12.5 → PM merge → 归档 history/                   │
└──────────┬──────────────────────────────────────────────────────────┘
           │
           ▼
   ┌────────────────────────────────────────┐
   │  POST-RUN HOOK (Phase 13)              │
   │  - 给 autopilot 反馈 (本轮成功/失败)    │
   │  - 更新 queue.md (该候选标 done)        │
   │  - 触发下次唤醒前的冷却期               │
   └────────────────────────────────────────┘
```

---

## 文件清单（运行时 vs skill 资源）

### Skill 资源（`<this-skill-path>/autopilot/`）

```
autopilot/
├── README.md                    # 本文件（PM 入口）
├── config.yaml                  # 默认配置模板
├── safety-brakes.md             # 6 层安全闸完整规则
├── priority-algorithm.md        # 评分公式 + LLM-judged 升级路径
├── state-machine.md             # 状态机定义
├── pm-oversight.md              # /autopilot-* 命令规格
├── connectors/
│   ├── README.md
│   ├── roadmap.md               # 读 ~/.autopilot-data/product-roadmap.md
│   ├── user-feedback.md         # 读 iteration-vault/history/*/12.5-morning-review.md
│   ├── github-issues.md         # gh issue list
│   ├── sentry.md                # Sentry MCP
│   └── notion.md                # Notion MCP
└── templates/
    ├── queue.md                 # ~/.autopilot/queue.md 模板
    ├── autopilot-seed.md        # 注入 iteration-vault/00-intake.md 种子
    ├── morning-digest.md        # 早晨给 PM 的汇报模板
    └── escalation-autopilot.md  # autopilot 专属 escalation
```

### PM 运行时（`~/.autopilot-data/`）

```
~/.autopilot-data/
├── product-roadmap.md           # PM 手维护的清单
└── autopilot/
    ├── config.yaml              # 实际生效配置（首次从 skill 模板复制）
    ├── PAUSE                    # touch 即暂停
    ├── EMERGENCY_STOP           # 紧急停（需 PM reset）
    ├── state.json               # 状态机当前状态
    ├── queue.md                 # 每次唤醒后重写
    ├── INBOX.md                 # Tier 1 模式下给 PM 的待办
    ├── run-history.jsonl        # 历次跑记录
    ├── blacklist.yaml           # 候选黑名单
    └── logs/
        ├── wake-up-<date>.log
        └── pre-flight-failed-<date>.log
```

---

## 4 周 ramp 计划（按周加能力）

| 周 | 加什么 | 跑通什么 |
|---|---|---|
| **W1** | 仅 roadmap connector + Tier 1 + kill switch | PM 早晨看 INBOX 决定跑不跑 |
| **W2** | 加 user-feedback connector + queue.md 多候选 | PM 看到 queue.md 中有 2-3 个候选 |
| **W3** | 加 GitHub connector + 完整优先级公式 + 6 层安全闸 + pre-flight | 真正的"自动选题"，所有安全闸生效 |
| **W4** | 加 Sentry + Notion + 状态机 + run-history + 早晨 digest + slash 命令 | autopilot 完整版（仍 Tier 1）|
| **W5+** | 切 Tier 2 试运行 1-2 周 | 小 bug fix 自动跑 |
| **W8+** | 评估是否切 Tier 3 | 完全自治（如果 PM 信任了）|

每周末复盘：autopilot 选的候选 PM 是否同意？候选质量分布？误报率？决定下周加什么。

---

## 关键文件交叉引用

- 6 层安全闸 → `autopilot/safety-brakes.md`
- 评分公式 → `autopilot/priority-algorithm.md`
- 状态机 → `autopilot/state-machine.md`
- PM 命令规格 → `autopilot/pm-oversight.md`
- 各 connector 实现 → `autopilot/connectors/*.md`
- 与主流水线对接（Phase 0 注入 + Phase 13 回流）→ `phases/13-autopilot-handoff.md`

---

## 与现有 4 红线 + 夜间模式的关系

- autopilot 选定候选后注入 `00-intake.md` → 进 Phase 0/1/2 → 主流水线照常
- Phase 2 PRD 关卡：Tier 1/2 仍 PM 把关；Tier 3 autopilot 自批（仅 size=small）
- 后续 Phase 2.5-12 走夜间模式 autonomous
- 4 红线 R1-R4 触发 → autopilot 暂停 + escalate（同 PM 触发的迭代）
- Phase 12.5 早晨复盘：PM 走 4 步清单决策 merge/redo

---

## 维护备忘

- 4 周 ramp 计划可压缩或拉长，按 PM 信任曲线调
- 新增 connector 时同步 `priority-algorithm.md` 权重
- W3 切到 Tier 2 前确保安全闸全部就位
