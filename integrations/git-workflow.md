# Git Workflow 集成精华（GitHub Flow + gh CLI 标准动作）

> **基于**：GitHub Flow 官方文档 + gh CLI 文档 + GitOps Rollback 最佳实践。
> **用法**：Phase 12 Read 本文件，扮演"git 工程师"角色，autonomous 完成 git 工作流闭环。
> **默认 workflow**：GitHub Flow（短命 feature 分支 + PR + squash merge to main + tag）。
> **目标用户**：PM 非技术背景，0 代码开发，所有 git 操作 autonomous 完成。

---

## 角色定义

你扮演**git 工程师**。任务：

1. 把本地开发好的代码安全推到 GitHub private repo
2. 创建 feature PR 并 autonomous merge
3. 让 release-please 接管版本管理
4. 出错时安全回滚

关键品质：
- **安全优先**：永远不直接 push main，永远不 force push
- **PM 友好**：所有命令 autonomous 跑，PM 只看最终结果（Release PR 链接）
- **可逆**：每个动作都能 revert

---

## 前置：gh CLI 必须已登录

```bash
gh auth status
```

未登录 → 写 `iteration-vault/INFRASTRUCTURE_ERROR.md`：
```
🔧 GitHub CLI 未登录，需要 PM 手动登录：

1. 打开 PowerShell
2. 跑 `gh auth login`
3. 选 GitHub.com → HTTPS → 用浏览器登录
4. 跑完后再次启动本 skill 续跑 Phase 12

skill 已暂停在 Phase 12 Step 1，等 PM 登录后继续。
```

不算 R 红线（不打扰 PM 决策，只需 PM 跑一个命令）。

---

## Private Repo 创建流程（首次）

### Step 1：决定 repo 名

autonomous 决策：
- 默认：`<owner>/<your-repo>`（owner 通常是 PM 的 GitHub 用户名或组织名）
- 若已存在 → autonomous 改名 `<repo-name>` / `<repo-name>-platform`
- 若没决定 owner → 用 `gh api user` 获取登录用户作为 owner

### Step 2：创建 + 推送

```bash
# 在当前工作目录初始化 git（若未初始化）
git init
git branch -M main

# 加 .gitignore（autonomous 写：node_modules, .env, dist 等）
# 加 README.md（autonomous 写：项目名 + 一句话定位）

# 第一次 commit
git add .
git commit -m "chore: initial commit"

# 创建 private repo + 推送
gh repo create <owner>/<your-repo> \
  --private \
  --source=. \
  --remote=origin \
  --push \
  --description "AI-native organization OS for e-commerce"
```

### Step 3：加 branch protection（保护 main）

```bash
# 通过 gh api 加 ruleset（GitHub 新版 branch protection 推荐用 ruleset）
gh api repos/<owner>/<your-repo>/rulesets \
  --method POST \
  --field name="protect-main" \
  --field target="branch" \
  --field enforcement="active" \
  --field "conditions[ref_name][include][]=refs/heads/main" \
  --field "rules[][type]=pull_request" \
  --field "rules[][parameters][required_approving_review_count]=0" \
  --field "rules[][parameters][dismiss_stale_reviews_on_push]=true"
```

**autonomous 设计决策**：
- `required_approving_review_count = 0`：PM 是唯一审阅者，AI 不能审 AI；要求 1 个会卡住 autonomous 流程
- 但要求 **PR 必须存在**（不能直接 push main）→ AI 必须走 PR 流程
- 要求 CI 通过（如果有）

**保守默认**：禁用 `force_push` 和 `deletion` 到 main。

### Step 4：第一次产物

```
iteration-vault/12-release.md 的"仓库信息"段：

- URL: https://github.com/<owner>/<your-repo>
- 默认分支: main（受保护）
- 创建时间: 2026-05-17
- 私有性: ✅
- branch protection: PR required, no force push, no deletion
```

---

## GitHub Flow 标准动作（每次迭代）

### Step 1：从 main 切 feature 分支

```bash
git checkout main
git pull origin main
git checkout -b feature/<slug>
```

`<slug>` 用 kebab-case，描述本次需求：
- `feature/add-shop-dashboard`
- `feature/sso-feishu`
- `feature/fix-n-plus-1-query`

autonomous 命名规则：从 PRD 标题用 LLM 提炼 3-5 个英文词 + 连字符。

### Step 2：开发（Phase 7 实际跑的事）

实施期间所有 commit 都在这个 feature 分支上。

### Step 3：push 到远程

```bash
git push -u origin feature/<slug>
```

### Step 4：创建 PR

```bash
gh pr create \
  --base main \
  --head feature/<slug> \
  --title "feat: <PRD 摘要>" \
  --body-file iteration-vault/11-release-notes.md
```

PR title 用 Conventional Commits 风格（squash merge 时会用作最终 commit message）。

### Step 5：等 CI / branch protection 通过

```bash
gh pr checks <pr-number> --watch --interval 30
```

如果有 CI 失败：
- autonomous 看 log（`gh run view <run-id> --log`）
- 如果是 lint/format → autonomous 修
- 如果是 test → 回 Phase 7 修对应任务
- 如果是基础设施 → 写 INFRASTRUCTURE_ERROR.md

### Step 6：autonomous merge

```bash
gh pr merge <pr-number> --squash --auto --delete-branch
```

**关键 flag 解释**：
- `--squash`：把 feature 分支所有 commit 压成一个，main 历史干净
- `--auto`：检查通过后自动 merge（不立即）
- `--delete-branch`：merge 后自动删 feature 分支（避免堆积）

### Step 7：release-please 接管

merge 触发 release-please workflow，详见 `integrations/release-please.md`。

---

## Hot-fix 流程

生产 bug 出现，紧急修复路径：

### Step 1：从生产 tag 切 hotfix 分支

```bash
# 假设最新 release 是 v1.2.0
git checkout -b hotfix/v1.2.1 v1.2.0
```

### Step 2：修 bug + commit + push

```bash
# 修代码
# commit 用 fix: 前缀
git commit -m "fix(api): hot-fix N+1 query causing prod outage"
git push -u origin hotfix/v1.2.1
```

### Step 3：创建 PR 到 main

```bash
gh pr create \
  --base main \
  --head hotfix/v1.2.1 \
  --title "fix: 紧急 hot-fix" \
  --body "影响: [描述生产事故]
  
修复内容: [描述修了什么]

测试: [简短验证]"
```

### Step 4：加 hot-fix 标签（让 release-please 加急处理）

```bash
gh pr edit <pr-number> --add-label "hotfix"
```

### Step 5：merge 后 release-please 自动 bump patch

v1.2.0 → v1.2.1 自动发版。

### Step 6：（可选）通知

```bash
gh issue create \
  --title "Hotfix v1.2.1 已发布" \
  --body "影响: ... 修复: ..." \
  --label "incident,post-mortem"
```

---

## 回滚流程

发版后发现问题，回滚到上一版本：

### 推荐方案：git revert（保留历史，可追溯）

```bash
# 找到要回滚的 merge commit
git log --oneline --merges -10

# revert（-m 1 表示保留 main 那条线的内容）
git revert -m 1 <merge-commit-sha>

# push 到 main
git push origin main
```

release-please 会自动新建 PR 反向 bump（取决于配置）或保持版本。

### 不推荐：git reset（破坏历史）

```bash
# 仅在 PM 明确同意下，autonomous 不会主动做
git reset --hard <good-commit>
git push --force origin main   # 危险！
```

autonomous **永远不**主动跑 reset / force push。如果 PM 明确说"force reset"，写 `iteration-vault/DESTRUCTIVE_OP_CONFIRMED.md` 留痕。

### GitOps 风格回滚（推荐生产环境）

如果用 ArgoCD / Flux 等 GitOps 工具：
- revert 即回滚部署（GitOps controller 自动重新同步）
- 无需手动跑 deploy 命令

---

## 私有 Repo 特殊注意

### Release notes 可见性
- private repo 的 release notes 默认仅 collaborator 可见
- 无需脱敏（内部使用）

### 未来转 public 的预防
若未来主仓可能转 public，autonomous 在写 release notes 时主动避免：
- 内部代号
- 客户名（用 "Customer A" 代替）
- 内部 URL / IP
- 测试账号

这些限制写到 `iteration-vault/12-release.md` 的"待 public 化检查项"段（PM 早上 review）。

### Secrets 管理

```bash
# 加 secret（autonomous 创建首次 workflow 时）
gh secret set GITHUB_TOKEN --body "..."   # 通常 Actions 自动有
gh secret set CLAUDE_API_KEY --body "..."  # 若 CI 需要调 Claude
```

**autonomous 禁止**：把 secret 写到代码 / commit / autonomous-decisions.md。任何 secret 操作必须只走 `gh secret set` 或 GitHub UI。

---

## 命令速查（Phase 12 autonomous 跑这些）

```bash
# 检查
gh auth status
gh repo view
git status
git log --oneline -10

# 创建 repo（首次）
gh repo create <owner>/<your-repo> --private --source=. --remote=origin --push

# Branch protection
gh api repos/<owner>/<your-repo>/rulesets --method POST ...

# Feature PR
git checkout -b feature/<slug>
git push -u origin feature/<slug>
gh pr create --base main --head feature/<slug> --title "..." --body-file ...
gh pr checks <pr-number> --watch
gh pr merge <pr-number> --squash --auto --delete-branch

# Hot-fix
git checkout -b hotfix/v<x.y.z> v<previous-tag>
gh pr create --base main --head hotfix/... --label hotfix

# Release
gh pr list --label "autorelease: pending"  # 看 release-please PR
gh release list
gh release view v<x.y.z>

# 回滚
git revert -m 1 <merge-sha>
git push origin main
```

---

## 反模式（必拒绝）

- ❌ 直接 push main（违反 branch protection）
- ❌ force push（破坏历史 + 影响他人）
- ❌ 跳过 PR（用 commit + push 直接合）
- ❌ merge 时用 rebase 或 merge（应用 squash 保持 main 历史干净）
- ❌ 把 secret 写到任何文件
- ❌ delete branch 时用 force（用 gh pr merge --delete-branch 安全方式）
- ❌ 给 main 加 `--allow-deletion` 或 `--allow-force-push` 例外
- ❌ autonomous 跑 `git reset --hard`（破坏性，必须 PM 确认）
