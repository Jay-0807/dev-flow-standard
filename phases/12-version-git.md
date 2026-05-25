# 🚀 git 发版

> 内部编号：Phase 12
> 模式：🤖 Autonomous（git 工作流 + GitHub Release + release-please）

## 目标
本 skill 的**最后一个 phase**。完成本次迭代的 git 工作流闭环：
- Conventional Commits 整理
- feature branch / squash merge / tag
- release-please 自动维护 Release PR
- 私有 GitHub 仓库 push（首次会自动创建仓）
- 监控接入 / 告警规则
- 完成汇报给 PM

PM 早上回来看到的不是"代码做完了"，而是"GitHub 上有个 Release PR，你点 merge 就发版"。

## 输入
- `iteration-vault/11-release-notes.md`（含 Deployment Checklist）
- `iteration-vault/12-release.md`（本 phase 输出，初始为空）
- 本次所有 commits（待整理为 Conventional Commits）
- `integrations/release-please.md`（内化精华）
- `integrations/git-workflow.md`（内化精华）
- `principles/karpathy-llm-coding.md`（Surgical Changes 要求 PR diff 干净）

## 工作流（6 步）

### Step 1：检查前置（gh CLI / git 状态）

```bash
# 必须的前置条件
gh auth status          # 必须已登录 GitHub
git status              # 工作区必须 clean（无未 commit 改动）
git log --oneline -5    # 看最近 commit 是否符合 Conventional Commits
```

任一项不通过 → **不要触发 R 红线**，而是 autonomous 处理：
- gh 没登录 → 写 `iteration-vault/INFRASTRUCTURE_ERROR.md` 让 PM 早上登录后续跑
- 工作区脏 → 主线程 commit 干净后继续
- commit 不符合 Conventional Commits → autonomous 整理（用 `git rebase -i` 改 message 或新建一个 squash commit）

### Step 2：Conventional Commits 整理

Read `integrations/release-please.md` 的"Conventional Commits 规范"段。

把本次实施期间的 commits 整理为：
- `feat: 用户故事 X 的实现`
- `fix: bug Y 修复`
- `chore: 重构 Z`
- `docs: 更新 README`
- `test: 加 e2e 测试`
- `refactor: 内部重构（不影响外部行为）`
- `perf: 性能优化`
- 含 `BREAKING CHANGE: 描述` 的 commit 会触发 major 版本

**autonomous 决策**：
- 优先保留原 commit 历史（如果已经符合规范）
- 不符合规范的 → autonomous 整理（详见 release-please.md "整理策略"段）
- 保守默认：宁可多个小 commit，也不要"big bang squash"——便于回溯

### Step 3：建 / 检查 GitHub private repo

Read `integrations/git-workflow.md` 的"private repo 创建流程"段。

**首次场景**（你的情况）：
```bash
# autonomous 创建 项目主仓
gh repo create <owner>/<your-repo> --private --source=. --remote=origin --push
# autonomous 加 branch protection（保护 main 分支）
gh api repos/<owner>/<your-repo>/rulesets --method POST -f ...
```

**已有仓场景**（未来用）：
```bash
git remote -v          # 检查 origin 是否已配
gh repo view           # 确认仓已存在
```

仓信息写到 `iteration-vault/12-release.md` 的"仓库信息"段：
- repo URL
- 默认分支
- branch protection 规则

### Step 4：GitHub Flow 创建 feature PR

```bash
# 切到 feature 分支
git checkout -b feature/<slug>

# push 到远程
git push -u origin feature/<slug>

# 创建 PR（标题用 Conventional Commits 风格）
gh pr create \
  --base main \
  --head feature/<slug> \
  --title "feat: <一句话功能摘要>" \
  --body-file iteration-vault/11-release-notes.md
```

PR 描述里附：
- 业务摘要（给 PM 看）
- 技术摘要（给开发看）
- Deployment Checklist
- 验收证据链接（iteration-vault/10-verification-report.md 摘要）
- 链接到 iteration-vault/autonomous-decisions.md（PM 早上审计）

### Step 5：配 release-please-action（首次）

Read `integrations/release-please.md` 的"GitHub Actions 配置"段。

**首次需要**：写 `.github/workflows/release-please.yml`（autonomous 写，PM 不感知）：
```yaml
name: release-please
on:
  push:
    branches: [main]
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
        with:
          release-type: simple  # 或 node / python / 等
          token: ${{ secrets.GITHUB_TOKEN }}
```

这个 workflow 在 feature PR merge 到 main 后会自动跑，维护一个长期开着的 Release PR。

**autonomous 决策**：release-please 配置选哪种 `release-type`？
- node 项目 → `node`
- python 项目 → `python`
- 多语言 / 不明确 → `simple`（最通用，只维护版本号 + CHANGELOG，不动 manifest）
- 保守默认：项目主仓 Next.js + 自研栈 → 选 `node`

### Step 6：自动合并 feature PR（autonomous）+ 自动触发 Release PR

**关键设计决策**（v2 默认）：

Feature PR **不需要 PM 关卡**——PM 在 PRD 关卡已经全权委托。直接 autonomous merge：

```bash
# 等 PR 的 CI / branch protection 通过
gh pr checks <pr-number> --watch

# autonomous squash merge
gh pr merge <pr-number> --squash --auto --delete-branch
```

merge 后，release-please-action 自动检测 commits，更新长期 Release PR（含 CHANGELOG + 版本号 bump）。

**Release PR 才是 PM 早上看到的关卡**：
- PR 描述里有所有 feature 改动汇总（来自 conventional commits）
- 自动 bump 的版本号（如 v1.2.0 → v1.3.0）
- PM 看完后**点 merge** → 自动 tag + 发 release

> PM 视角：早上回来看 GitHub 两个 PR
> - Feature PR：已 autonomous merge，PM 可以看历史
> - Release PR：等 PM 点 merge → 发版

## 产出

**文件**：`iteration-vault/12-release.md`

结构：
```markdown
# git/GitHub Release 记录: <feature-name>

## 仓库信息
- URL: https://github.com/[owner]/[repo]
- 默认分支: main
- branch protection: [规则]

## 本次 Feature PR
- URL: [pr-url]
- 状态: ✅ autonomous merged
- 合并方式: squash
- Conventional Commits 摘要:
  - feat: [N] 个
  - fix: [N] 个
  - chore/docs/test: [N] 个
  - BREAKING CHANGE: [N] 个

## release-please Release PR（PM 早上需点 merge 的）
- URL: [release-pr-url]
- 状态: ⏳ 等 PM merge
- 版本号变化: vX.Y.Z → vX'.Y'.Z'
- CHANGELOG 自动生成: 是

## CI / Branch Protection
- 所有 check 是否通过: [yes/no]
- 失败的 check: [list]

## 监控接入（Deployment Checklist 状态）
[来自 11-release-notes.md 的 checklist，标 ✅ 已完成项 / ⏳ 待 PM merge 后跑]

## autonomous 决策
- 决策 1: ...
- 决策 2: ...
```

## 🌅 递交摘要（v4 缩减版：决策动作迁到 Phase 12.5）

> **v4 变化**：v2 这里有 "PM 一键决策" 动作，v4 全部迁到 Phase 12.5 早晨复盘的结构化清单。本段仅做"递交报告"。

写完 12-release.md 后，写一段简短摘要到 state.yaml + 触发 12.5：

- 写 `state.yaml: mode=morning-review-pending`
- 写"递交摘要"到终端 / 通知
- 不调 AskUserQuestion，决策动作留给 Phase 12.5

递交摘要模板：
```
🌅 夜间模式跑完。详细复盘请走 Phase 12.5。

✅ 总体状态：成功

📊 数字摘要：
- 完成任务：[N]/[M]
- autonomous 决策：[K] 条
- 代码改动：[X] 文件 / +[Y] -[Z] 行
- 测试：单测 +[A] / e2e +[B] / 覆盖率 [P]%
- 安全：[全过 / 已修 N 项]
- 验收：5 层 ✅ / PRD AC [N/M] 通过

📍 重点关注（PM 在 Phase 12.5 看的 4 个）：
1. iteration-vault/autonomous-decisions.md 标 ⚠️ 的 [N] 条
2. iteration-vault/09-review-reports/summary.md 的 should-fix
3. iteration-vault/10.5-user-acceptance.md 的"最痛"
4. **Release PR**: [URL] ← Phase 12.5 PM 决策后 merge

🚀 下一步：
→ 进入 Phase 12.5 早晨复盘（PM 4 步清单 + 4 选项）
→ 见 phases/12.5-morning-review.md
```

## v4 加 11.5 漂移检测前置依赖

本 phase 启动前**必须先跑 Phase 11.5 漂移检测**（见 `phases/11.5-sync.md`）。如果 11.5 报告含 ERROR 级漂移 → 阻塞本 phase，回 Phase 7 修代码或回 Phase 4 调架构。仅 INFO/WARN 级可继续。

## 失败回退

| 失败 | 处理 |
|---|---|
| gh CLI 未登录 | 写 INFRASTRUCTURE_ERROR.md，等 PM 登录后续跑（不算 R 红线）|
| 创建 private repo 失败（权限/名字冲突） | autonomous 改名重试 1 次，再失败 → 写 INFRASTRUCTURE_ERROR.md |
| PR 创建后 CI 失败 | 回 Phase 7 修对应项，重跑 Phase 10-12 |
| release-please-action 没触发 | autonomous 检查 yml 配置 + secret 权限，最多重试 2 次 |
| Release PR 描述格式错误 | autonomous 编辑 PR 描述，主线程 review |

## Hot-fix 流程（出现在已发布版本中的 bug）

不在本次 phase 跑，但 skill 应记住此流程，PM 后续发起 hot-fix 时引用：

```bash
# 从生产 tag 切 hotfix 分支
git checkout -b hotfix/v1.2.1 v1.2.0
# 修 bug
# 测试 + commit（用 fix: 前缀）
# push + create PR
gh pr create --base main --head hotfix/v1.2.1 ...
# merge 后 release-please 自动 bump 到 v1.2.1 + 发版
```

详见 `integrations/git-workflow.md` 的"Hot-fix"段。

## 回滚流程

```bash
# 用 git revert 而不是 reset（保留历史）
git revert -m 1 <merge-commit-sha>
git push origin main
# release-please 会自动新建 Release PR 反向 bump（或保持版本，看配置）
```

详见 `integrations/git-workflow.md` 的"回滚"段。
