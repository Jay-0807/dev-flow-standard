# Phase 12.5 早晨复盘记录模板

> 把本模板复制为 `iteration-vault/12.5-morning-review.md`，逐段填充。

---

```markdown
# Phase 12.5 早晨复盘记录: <feature-name>

**复盘时间**: <YYYY-MM-DD HH:mm>
**复盘方式**: 自动触发 / PM 手动 /morning / PM 手动 /复盘
**夜间模式跑时长**: <H.H>h
**PM 最终决策**: ✅ merge / 🔄 局部 redo (X) / ❌ 整体 redo / ⏸ 推下夜

---

## 1. 30 秒摘要（skill 自动填）

- 任务完成：[N]/[M]
- 自治决策：[K] 条（⚠️ [X] 条值得 PM 看）
- GAN 跑了：[N] 个任务，平均 [X.X] 轮 PASS，[Y] 次 PIVOT
- 代码改动：[X] 文件 / +[Y] -[Z] 行
- 测试新增：[N] 条单测 / [M] 条 e2e / 覆盖率 [P]%
- 安全审查：[已修] / [剩 must-fix]
- 五层验收：[X/5] 通过
- 用户验收：[Y.Y]/5（[N] 真实用户）
- Release PR: [URL]

---

## 2. 4 点关注点结果

### 2.1 ⚠️ 自治决策审计

| 决策 ID | Phase | 摘要 | PM 反应 | checkpoint |
|---|---|---|---|---|
| #7 | 4 | 选 deprecate 而非删除 | ✅ | ckpt-decision-7 |
| #12 | 4 | 引入 swr 而非 react-query | 🔄 redo | ckpt-decision-12 |
| #15 | 6 | DB 字段加 nullable | ✅ | ckpt-decision-15 |

### 2.2 审查 should-fix（来自 09-review-reports）

| 项 | 严重度 | 位置 | PM 决定 |
|---|---|---|---|
| API 错误处理可统一 | 🟡 major | src/api/users.ts | backlog |
| 弃用日志可清理 | 🟢 minor | src/lib/legacy/ | 本次 redo（顺手）|
| ... | ... | ... | ... |

### 2.3 真人用户验收（本阶段 Phase 12.5 复盘角色）

- 评分：[X.Y]/5
- 用户反馈聚类前 3：
  - [反馈 1]（[N] 人）
  - [反馈 2]（[M] 人）
- 最爱：[摘录]
- **最痛**：[摘录]（高亮）
- PM 决定：接受 / 不接受 / 进 backlog 修

### 2.4 Karpathy 4 原则违规扫描

| 原则 | 违规数 | 严重度 | PM 决定 |
|---|---|---|---|
| Think Before Coding | 0 | - | - |
| Simplicity First | 1 | 🟡 | 已在 should-fix 中处理 |
| Surgical Changes | 0 | - | - |
| Goal-Driven Execution | 0 | - | - |

---

## 3. 最终决策

### 选项: <填写>

#### 如选 ✅ merge
- Release PR merge SHA: <sha>
- 归档路径: `history/<timestamp>-<feature>/`
- 发版状态: ✅ 成功 / ⚠️ merge 但未发布 / ❌ merge 失败
- 后续: skill 退出，autopilot 回流

#### 如选 🔄 局部 redo
- redo 范围: Phase X / 决策 #N / 多个
- 注入的新方向: <一句话>
- redo 触发的下游 phase: <列表>
- 启动: 进入夜间模式 redo batch

#### 如选 ❌ 整体重做
- 重做原因: <PM 写 1-3 句>
- 归档路径: `history/REJECTED-<timestamp>-<feature>/`
- 回到 Phase: 2（PRD 重写）/ 1（重新澄清）
- 状态: vault 已归档，下次跑全新

#### 如选 ⏸ 推下夜
- 下次复盘触发条件: PM 回话 / `<date>` 自动提醒
- vault 保留状态: 完整原样
- state.yaml mode: postponed
- morning_review_postponed_until: <next-morning-date>

---

## 4. 后续动作（skill 自动填）

- [ ] 归档 vault（如适用）
- [ ] 通知 autopilot 本轮完成（如适用）
- [ ] 启动 redo batch（如适用）
- [ ] 更新 api-registry.md 的"本迭代变更"段（如适用）
- [ ] 写 `history/<timestamp>-<feature>/release-summary.md`（如成功）

---

## 5. PM 反思（可选填）

PM 在复盘后可加一段反思，沉淀经验：

- 这次跑得好的地方：<...>
- 这次跑得不好的地方：<...>
- 下次迭代要调整的：<...>
- 给 skill 的反馈：<...>
```
