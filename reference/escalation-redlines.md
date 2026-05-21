# Escalation Redlines — 4 红线判定

自治模式下，**任一红线触发 → 暂停自治 → AskUserQuestion 升级到 PM**。这是兜底机制，避免 AI 在 PM 不在场时做不可逆破坏。

## R1：架构冲突

**触发条件**：
- phase 3 ADR 出来后，发现与既有代码 / 既有 ADR 直接冲突
- 例：新 ADR 想用 PostgreSQL，但既有架构是 MongoDB；改 PostgreSQL 意味着重写整个 data layer
- 例：新功能要用同步 API，但现有架构全是 event-driven

**Claude 行为**：
1. 不要"先按新 ADR 写代码再说"
2. 立刻暂停 phase 3
3. AskUserQuestion 给 PM：
```
Q: 阶段 3 发现架构冲突 — 新功能想要 [X]，但现有架构是 [Y]，二者不兼容。
A:
- 保持现有架构，新功能降级实现（描述降级方案）
- 接受架构改造成本，本次迭代范围扩大（描述影响面）
- 暂停本次迭代，先做独立架构升级（开新 PRD）
- 其他（PM 描述）
```

---

## R2：安全 must-fix > 3 条

**触发条件**：
- phase 7c 安全审查发现 > 3 条 must-fix 级别问题
- must-fix 定义：会导致 SQL injection / XSS / RCE / 越权 / 凭证泄露 / 数据库未加密等高危
- 不包括 should-fix（如缺少 rate limit）或 nice-to-fix（如 header 加固）

**Claude 行为**：
1. 不要尝试自动修 > 3 条 must-fix（高风险）
2. 立刻暂停 phase 7
3. AskUserQuestion：
```
Q: 阶段 7 安全审查发现 N 条 must-fix 高危问题（详见 07-security.md）。
A:
- 全部由 Claude 尝试修，每修一条单独 review
- PM 自己看每条决定修 / 接受 / 推迟
- 暂停发版，安全 issue 立独立 ticket
```

---

## R3：验收 3 次重试失败

**触发条件**：
- phase 7 测试通过率 < 80%，重试 3 次仍不过
- 或 phase 9 review 反复打回，3 轮还有 blocking issue

**Claude 行为**：
1. 不要"为了过测试改 assertion"或"为了过 review 删 reviewer comment"
2. 立刻暂停
3. AskUserQuestion：
```
Q: 阶段 [7/9] 重试 3 次仍不过 — 这通常意味着 PRD 表达不准或方案有根本问题。
A:
- 我描述具体哪里不过，PM 帮看看根因
- 回 phase 2 修 PRD 重新走
- 接受当前状态，本次 release 缩小范围（移掉做不下来的部分）
```

---

## R4：删除既有功能

**触发条件**：
- 任意阶段发现实施方案需要删除 / 破坏性修改 既有 user-facing 功能
- 包括：删 API endpoint、删 DB column 含数据、删 UI 元素、改变默认行为

**Claude 行为**：
1. **绝对不要静默删除**
2. 立刻暂停
3. AskUserQuestion：
```
Q: 当前方案需要删除 / 破坏既有功能 [X]（位置：[file:line]）。
A:
- 确认删除（PM 知道影响）
- 改为软删除（标记 deprecated，N 个版本后真删）
- 重新设计方案，不删 X
```

---

## 红线触发后的恢复

PM 给出选择后：
1. 在 `autonomous-decisions.md` 记录：触发的红线 + PM 决策 + 时间戳
2. 按 PM 决策继续执行
3. 不要再触发同一红线（除非 PM 改主意）

## 红线 vs 普通失败的区分

- **普通失败**（编译错 / 测试错 1 次 / lint 错）：Claude 自己修，不触发红线
- **红线**：影响"用户能不能继续用"或"PM 是否同意"，必须人介入

不要把红线机制用成"懒得自己解决问题就找 PM"。
