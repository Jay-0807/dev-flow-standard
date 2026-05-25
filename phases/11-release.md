# 📢 发布说明

> 内部编号：Phase 11
> 模式：🤖 Autonomous + GAN（发布说明 + 内部完成确认）

## 目标
**v2 重大变化**：v1 的 Phase 11 含部署执行 + 监控接入（部分 PM 关卡），v2 拆为两阶段：
- **Phase 11**（本 phase）：autonomous 生成发布说明 + 整理素材，**不做实际部署**
- **Phase 12**（新增）：git/GitHub release 自动化（创建 PR / merge / tag / push）

本 phase 输出 = Phase 12 的输入。两者都是 autonomous，不打扰 PM。

## 输入
- `iteration-vault/02-PRD.md`
- `iteration-vault/10-verification-report.md`（验收已通过）
- 本次所有 commits
- `templates/release-notes.md`

## 工作流（3 步）

### Step 1：调 finishing-a-development-branch 收尾代码侧

触发 `superpowers:finishing-a-development-branch`。
按其指引：
- 确认所有改动已 commit
- worktree 已合并回主分支
- 临时 branch 已清理（如有）

**注意**：本 step **不做** push / tag / release——这些都是 Phase 12 的事。本 step 只确保本地代码状态干净，准备好进 git workflow。

### Step 2：生成发布说明草稿（按 templates/release-notes.md）

双受众版本：
- **技术受众**：模块改动 + API 变更 + migration 步骤 + 回滚命令
- **业务受众**：功能亮点 + 使用方式 + 已知限制

调 docx 输出 `.docx` 副本。

**注意**：本 step 生成的 release-notes 草稿会被 Phase 12 的 release-please 用作 Release PR 描述的素材。所以**写好 Conventional Commits 友好的描述**：
- 用 `feat:` / `fix:` / `chore:` 开头
- 一条改动一条描述
- 不混业务话术与技术话术（分两段写）

### Step 3：整理部署 checklist（输出给 Phase 12）

把以下 checklist 整理出来作为 Phase 12 的输入：

```
□ feature flag 已配置（默认关闭，灰度名单 = [list]）
□ 数据库 migration 文件已 ready
□ 新增配置项已记录（需要在 GitHub Secrets / env 配置）
□ DNS / CDN / 缓存策略变更（如需）
□ 监控指标 dashboard 已设计（具体接入在部署后）
□ 告警规则已设计
□ 灰度 → 全量的触发条件已明确
□ rollback 命令已 dry-run 过
```

这个 checklist 写到 `iteration-vault/11-release-notes.md` 的最后一段。Phase 12 会把它作为 Release PR 的 "Deployment Checklist" 段贴上去。

## 产出

**文件**：`iteration-vault/11-release-notes.md` + `iteration-vault/11-release-notes.docx`

结构（沿用 templates/release-notes.md）：
- 业务摘要 / 功能清单 / 已知限制
- 老用户影响 / 数据迁移 / 兼容性
- 技术摘要 / API 变更 / migration / 配置
- 验收证据 / backlog / Deployment Checklist

**附加产出**：`iteration-vault/meta.json` 更新本次迭代状态为 `release-ready`

## 🤖 Autonomous 决策记录

本 phase 决策记录到 `autonomous-decisions.md`，典型 1-3 条：
- 发布说明的措辞选择（保守 vs 鼓吹）→ 默认保守
- 是否单独列"已知限制"段 → 有任何限制就列
- 业务摘要的措辞是否需要给 PM 改 → 标 ⚠️ 提示 PM 早上看一下

## 不做 PM 关卡

v1 这里调 AskUserQuestion 问 PM"同意发布吗"。v2 **不调**——PM 在 PRD 关卡已经全权委托。

但 PM 早上回来看 Release PR 时，会再有一次"看 PR 描述 + 点 merge"的关卡，那时才是真正的"上线确认"。本 phase 是为那次关卡准备素材。

## 失败回退

- 若 finishing-a-development-branch 报告"仍有未 commit 改动"→ 主线程接手 commit + 重跑 Phase 11
- 若 release notes 中发现技术细节缺失（如没说清 migration）→ 回 Phase 7 补 log
- 若 deployment checklist 缺关键项（如 feature flag 没配）→ autonomous-decisions.md 标 ⚠️，继续，但 Phase 12 的 PR 描述会显式标红

## 进入 Phase 12

本 phase 完成后 autonomous 进入 Phase 12（git/GitHub release）。无需 PM 介入。
