# release-please 集成精华 + Conventional Commits

> **来源**：googleapis/release-please（6.8k★，Google 出品）+ Conventional Commits 1.0.0 规范。
> **用法**：Phase 12 Read 本文件，扮演"release 工程师"角色，autonomous 维护 Release PR + CHANGELOG + 版本号。
> **核心价值**：在"AI 自治开发 + PM 一键发版"的链路里，release-please 是**唯一的 PM 关卡**——PM 早上看一个 Release PR，点 merge 就完成发版。

---

## 角色定义

你扮演 **release 工程师**。任务：基于 Conventional Commits 自动维护一个**长期开着的 Release PR**，让 PM 看一个 PR 就完成版本管理。

关键品质：
- **不打扰 PM 代码层**：所有 commit 整理 / 版本号决策 autonomous
- **让 PR 自带 CHANGELOG**：PM 不用读 100 个 commit，看 PR 描述就知道发了什么
- **保守的版本 bump**：能 minor 不 major，能 patch 不 minor

---

## Conventional Commits 规范（必背）

每条 commit message 格式：
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### type 与版本影响

| type | 用途 | 版本影响 |
|---|---|---|
| `feat:` | 新功能 | minor bump (1.2.0 → 1.3.0) |
| `fix:` | bug 修复 | patch bump (1.2.0 → 1.2.1) |
| `perf:` | 性能优化（用户可感知） | patch bump |
| `refactor:` | 内部重构（不影响外部） | 无 bump |
| `chore:` | 杂项维护 | 无 bump |
| `docs:` | 文档 | 无 bump |
| `test:` | 测试 | 无 bump |
| `style:` | 格式化（不改逻辑） | 无 bump |
| `build:` | 构建系统 | 无 bump |
| `ci:` | CI 配置 | 无 bump |
| 含 `BREAKING CHANGE:` footer 或 `feat!:` `fix!:` 等 | 破坏性变更 | **major bump** (1.2.0 → 2.0.0) |

### scope 用法（可选但推荐）

scope 标功能模块：
- `feat(auth): 加 SSO`
- `fix(api): 修 N+1 查询`
- `perf(frontend): 优化首屏渲染`

CHANGELOG 会按 scope 分组。

### footer 关键字段

- `BREAKING CHANGE: <描述>` → 触发 major bump
- `Closes #123` → 自动关联 GitHub issue（merge 时关闭）
- `Co-authored-by: <name> <email>` → 多作者

### 中文友好规则（项目专用）

`<description>` 可以中文写：
```
feat(auth): 加飞书 SSO 登录
fix(api): 修 N+1 查询导致首页慢
```

但 type 关键字必须英文（release-please 解析依赖）。

---

## 整理策略（autonomous 用）

实施过程中（Phase 7）spawn 的子 agent 写的 commit 可能不规范。Phase 12 autonomous 整理时：

### 策略 1：已规范的保留
看 commit log 前 5 条，若都符合 Conventional Commits → 不动，直接进 Step 3。

### 策略 2：散乱时 squash + 重写

`git rebase -i <base>` interactive 重写不规范的 commit。
**注意**：本 skill 在 PowerShell 环境，`-i` 不可用。改用：
```bash
# 用 git reset 软回退后单独 commit
git reset --soft <base>
git commit -m "feat: <汇总描述>"
git commit -m "fix: <汇总描述>"
# 或者更简单：squash 成一个大 commit
git commit -m "feat: <PRD 摘要>

详情见 iteration-vault/02-PRD.md

Closes #<issue-id>"
```

### 策略 3：保守默认 = 多个小 commit 优于一个大 squash

按 PRD 的功能清单 F1/F2/F3 各做一个 commit，便于回溯。

### 策略 4：BREAKING CHANGE 必须显性

如果本次涉及任何破坏性（API 改返回结构 / 删 endpoint / DB 字段删除），**必须**加 BREAKING CHANGE footer。

autonomous 自检：
- [ ] 影响面分析 (03-impact-analysis.md) 是否标了 🔴 高风险？
- [ ] 高风险变更是否在 commit 里加了 BREAKING CHANGE？
- [ ] 没加 → autonomous 补上

---

## GitHub Actions 配置（首次）

`.github/workflows/release-please.yml`：

```yaml
name: release-please
on:
  push:
    branches: [main]
permissions:
  contents: write
  pull-requests: write
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
        with:
          release-type: <type>
          token: ${{ secrets.GITHUB_TOKEN }}
```

### release-type 选择

| type | 适用 | 行为 |
|---|---|---|
| `node` | npm/Node 项目 | 维护 package.json + CHANGELOG |
| `python` | Python 项目 | 维护 pyproject.toml / setup.cfg + CHANGELOG |
| `simple` | 多语言 / 不明确 | 只维护 version.txt + CHANGELOG |
| `go` | Go 项目 | 维护 go.mod + CHANGELOG |
| `rust` | Rust 项目 | 维护 Cargo.toml + CHANGELOG |

**autonomous 决策（本 skill 默认）**：
- 主仓 Next.js + TypeScript → `node`
- 子目录有 python 服务 → `python` (per-package)

### 进阶配置（多包 monorepo）

如果 项目是 monorepo（多个 package），用 manifest 模式：

`release-please-config.json`:
```json
{
  "packages": {
    "web": {"release-type": "node"},
    "agent-runtime": {"release-type": "python"},
    "shared-utils": {"release-type": "node"}
  }
}
```

`.release-please-manifest.json`：
```json
{
  "web": "1.0.0",
  "agent-runtime": "0.2.0",
  "shared-utils": "0.1.0"
}
```

---

## Release PR 内容（自动生成）

merge 到 main 后，release-please 自动维护一个 PR。典型 PR 描述：

```markdown
:robot: I have created a release *beep* *boop*
---

## [1.3.0](https://github.com/owner/repo/compare/v1.2.0...v1.3.0) (2026-05-17)

### Features

* **auth:** 加飞书 SSO 登录 ([abc1234](https://github.com/owner/repo/commit/abc1234))
* **dashboard:** 新增店铺数据看板 ([def5678](https://github.com/owner/repo/commit/def5678))

### Bug Fixes

* **api:** 修 N+1 查询导致首页慢 ([ghi9012](https://github.com/owner/repo/commit/ghi9012))

### BREAKING CHANGES

* **api:** /api/v1/users 返回结构改变，详见迁移指南
---

This PR was generated with [Release Please](https://github.com/googleapis/release-please).
```

PM 早上看的就是这个 PR。

### 当 PM 点 merge 后会发生什么

1. release-please-action 自动 tag（如 v1.3.0）
2. 创建 GitHub release（含上面的描述）
3. CHANGELOG.md 自动更新
4. version.txt / package.json 自动 bump
5. （可选）触发后续 deploy workflow

---

## Release PR 的 PM 关卡设计

虽然本 skill 在 v2 把 PM 关卡缩减为"PRD 关卡 1 个"，但 GitHub 上的 Release PR 是**第二个隐性关卡**——PM 看 PR 描述然后点 merge。

为什么这是好设计：
- PM 不在 Claude Code 里点 AskUserQuestion，而是在 GitHub UI 里看 PR
- PR 描述比 AskUserQuestion 选项**信息量大**得多（含 CHANGELOG）
- PM 可以转发链接给老板 / 客户做最终确认
- 不点 merge = 没发版，最后的 safety net

所以本 skill 的完整 PM 参与模型 = **1 个 PRD 关卡 + 1 个 GitHub Release PR 关卡**。

---

## 与 项目的兼容性

### 项目业务协议（如有）层
项目业务协议（如有）改动需要 BREAKING CHANGE 警告（一改影响所有 agent）。

### r4 哲学
release notes 必须显性化"本次发布对人工保留点的影响"。如果改了某个 AC 规则的人工兜底逻辑，必须在 PR 描述里高亮。

### 合同条款
若发版涉及合同条款（如 9.1 "AI 仅作建议"边界变更），autonomous-decisions.md 必标 ⚠️，让 PM 在 Release PR merge 前再 review 一次合同。

---

## 反模式（必拒绝）

- ❌ 不要直接 push 到 main 跳过 PR
- ❌ 不要混 type（如 `feat: fix bug` 应该用 `fix:`）
- ❌ 不要省略 BREAKING CHANGE（违反就要给用户带来生产事故）
- ❌ 不要用 `chore: bump version` 这种手动版本——让 release-please 全权决定
- ❌ 不要在 hot-fix 时打 minor tag——hot-fix 永远 patch

---

## 速查命令

```bash
# 创建 release-please workflow（首次）
cat > .github/workflows/release-please.yml << EOF
... (上面的 yml)
EOF

# 整理 commits（仅 autonomous 用）
git reset --soft <base>
git commit -m "feat(scope): description

Closes #123"

# 看 release-please 当前状态
gh pr list --label "autorelease: pending"

# 手动触发（极少用）
gh workflow run release-please.yml
```
