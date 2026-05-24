# Quality Redlines — 7 条质量红线（学 autodev）

> **来源**：直接借鉴 `Leooo-Huang/autodev-skills` 的 `autodev-shared/checklists/quality-redlines.md`，通用化后嵌入。
> **作用**：所有跑 GAN 的 reviewer 必检的硬约束。命中任一 → 对应 4 维打分项 ≤ 3 分（自动 FAIL）。
> **触发方式**：reviewer-skeptical.md prompt 把本文件全文嵌入 reviewer prompt。

---

## R1：禁占位符实现

**触发样例**：
- 代码中出现 `TODO` / `FIXME` / `HACK` / `XXX` / `stub` 字样
- 空函数体（`def foo(): pass` / `function foo() {}`）
- `NotImplementedError` / `throw new Error("not implemented")`
- 硬编码空集合作主数据（`return []` / `return {}` 而真实场景应有内容）
- 返回硬编码假数据（`return { id: 123, name: "test" }`）

**例外**：
- 单元测试里的 mock 不算（mock 本就该是假的）
- 注释里说明的待办（如 `// see issue #42 for follow-up`）不算
- 类型定义文件（`.d.ts` / type-only 文件）允许空实现

**autodev 原话**：
> "Blocks code containing TODO/FIXME/HACK/XXX/stub markers, empty function bodies, NotImplementedError, or hardcoded empty collections as primary data sources."

**reviewer 检查方法**：
- grep 关键词 → 报具体 file:line
- 看函数体是否真有逻辑

**违规处置**：该维度（通常是"完整性"）自动 ≤ 3 分 + 输出违规位置 `R1-XXX: <file>:<line>`

---

## R2：禁 mock 替代

**触发样例**：
- 变量名含 `mock` / `dummy` / `fake` / `placeholder` / `sample` 但不在测试文件
- 设计中标"调实 API"但代码用 `Promise.resolve({...})` 或硬返回
- `fetch()` 被替换成模拟函数（除非在 jest.mock 块内）
- API client 文件存在但调用点用本地数据替代

**例外**：
- 测试文件（`*.test.ts` / `*.spec.ts`）内允许
- Storybook stories 内允许
- 显式开发用 mock 服务（如 `msw` 配置文件）允许

**reviewer 检查方法**：
- grep `mock|dummy|fake` × 文件不在测试目录 → 报告
- 检查 fetch / axios 调用点是否真发请求

**违规处置**：该维度（通常是"鲁棒性"或"完整性"）自动 ≤ 3 分 + `R2-XXX`

---

## R3：禁降级方案 / 降级语言

**触发样例**：
- 注释含"暂时" / "先用" / "for now" / "later" / "as a workaround"
- PRD / 任务描述含上述语言（**这是设计阶段的 R3，不止代码**）
- 代码使用的库版本与 design.md 标的版本不一致（降级版本）
- 实际实现的功能 < PRD 标的功能（"先做一半"）

**autodev 原话**：
> "Enforces alignment between codebase libraries and design.md specifications; blocks plans containing 'for now/暂时/先用/workaround' language."

**reviewer 检查方法**：
- grep 关键词在代码注释 / PRD / 任务描述
- 对比 `package.json` 版本 vs `design.md` 标的版本
- 对比实现 vs PRD 的 AC 列表

**违规处置**：该维度（通常是"完整性"）自动 ≤ 3 分 + `R3-XXX`

---

## R4：禁过时技术

**触发样例**：
- 包版本不符合 design.md（如 design 标 React 18 但 package.json 是 React 17）
- 用了 deprecated API（如 React 旧 lifecycle）
- 用了项目里已废弃的内部模块（如 `legacy/` 目录的代码）

**例外**：
- 显式标为 v1 兼容期的代码（`/* @legacy-v1-compat */`）

**reviewer 检查方法**：
- 比对 design.md 标的版本 vs 实际版本
- 用 WebFetch 或本地知识检查是否 deprecated（context7 不可用时降级）

**违规处置**：该维度（通常是"一致性"）自动 ≤ 3 分 + `R4-XXX`

---

## R5：OSS 复用优先

**触发样例**：
- 自研功能 X，但 X 是常见需求且有成熟 OSS（autodev 强约束："必须 last-30-days WebSearch 扫"）
- 没有 OSS 扫描记录就开始写
- 重新发明轮子（自己写 date formatter / state machine / cache 等）

**例外**：
- 真的没有 OSS 满足需求（必须给出 OSS 比较矩阵证明）
- 自研的成本 < 集成 OSS 的成本（必须给数字）
- 业务核心代码（项目特定的核心模块）

**autodev 原话**：
> "Mandates OSS scanning documentation, comparison matrices for alternatives, explicit self-development justifications, and verification of last-30-days package data."

**reviewer 检查方法**：
- 看 `iteration-vault/04-architecture.md` 是否有 OSS 比较矩阵
- 看任务描述是否引用了 OSS 候选

**违规处置**：该维度（通常是"代码质量"）自动 ≤ 3 分 + `R5-XXX`

---

## R6：前端 icon 系统

**触发样例**：
- JSX text 节点直接用 emoji 作图标（如 `<span>🔥 推荐</span>`）
- 没集成 `lucide-react` / `@heroicons/react` / `phosphor-icons` 之一就开始写 UI
- 用图片当 icon 而非 icon component

**例外**：
- i18n JSON 值里允许（如 `"label": "🎉 庆祝"`）
- 代码注释里允许
- UGC 数据变量允许（如 user.avatar_emoji）
- README / 文档允许

**reviewer 检查方法**：
- grep JSX text 节点的 emoji（Unicode 检测）
- 检查 package.json 是否有 icon 库

**违规处置**：该维度（通常是"视觉规范"或"代码质量"）自动 ≤ 3 分 + `R6-XXX`

---

## R7：禁未验证环境断言

**触发样例**：
- 任务/PRD 中说"环境没装 Node / pnpm / Docker"但没有 `node --version` 等真实输出证据
- 缺 `env-capabilities.yaml` 文件却说"环境受限"
- 假设某工具/服务存在但没验证

**autodev 原话**：
> "Any statement like 'environment lacks Node/pnpm/Docker' must include actual command output. Requires env-capabilities.yaml production documenting dependency status before proceeding."

**reviewer 检查方法**：
- grep PRD / 架构 / 任务文档里的"环境"陈述
- 看是否有对应的 `*-version` 输出 / env-capabilities.yaml

**违规处置**：该维度（通常是"完整性"）自动 ≤ 3 分 + `R7-XXX`

---

## 项目专属 redlines（可选扩展，按项目业务自定义）

> 通用 7 红线（R1-R7）适用所有项目。如项目有特定业务约束，可加 R8/R9... 等。下面是 **项目的示例**（universal 版**不强制**这些，仅做扩展示范）：

### 示例 R8（项目示例）：LLM 输出 API 三字段缺失

**触发样例**：
- LLM 输出类 API 响应缺 `confidence` 字段
- 高风险决策 API 缺 `human_review_required` 字段
- 数据查询 API 缺 `data_source` 字段

**违规处置**：对应维度 ≤ 3 分 + `R8-XXX`

### 示例 R9（项目示例）：自定义协议违规

**触发样例**：
- 项目要求特定 RPC 协议（如 项目协议）但用 REST
- 消息缺业务定的信封字段

**违规处置**：对应维度 ≤ 3 分 + `R9-XXX`

### 如何为你的项目加新红线

1. 在 `RULES.md` §5 / §6 定义项目专属业务约束
2. 在本文件追加 R8/R9/... 段，写明触发样例
3. 更新 §reviewer 检查输出格式，加新行
4. 更新 §与 4 维打分的联动表

---

## reviewer 检查时的输出格式

reviewer 在 `<output>/round-N/scorecard.md` 里必须含：

```markdown
## Quality Redlines 检查

| Redline | 状态 | 违规清单 |
|---|---|---|
| R1 占位符 | ✅ | 无 |
| R2 mock | ❌ | R2-001: src/api/users.ts:42 用 `fakeUserData` |
| R3 降级语言 | ⚠️ | R3-001: PRD 中含"先用简化版" |
| R4 过时技术 | ✅ | 无 |
| R5 OSS 复用 | ✅ | 无（已查 OSS） |
| R6 emoji icon | ✅ | 无 |
| R7 未验证环境 | ✅ | 无 |
| R8 项目专属（如适用）| ✅ / - | 项目自定 |
| R9 项目专属（如适用）| ✅ / - | 项目自定 |

**critical redlines hit**: R2-001
→ 该轮 verdict: FAIL
```

---

## 与 4 维打分的联动

| Redline | 默认影响哪个维度 |
|---|---|
| R1 占位符 | 完整性 |
| R2 mock | 鲁棒性 或 完整性 |
| R3 降级语言 | 完整性 |
| R4 过时技术 | 一致性 |
| R5 OSS 复用 | 代码质量 |
| R6 emoji icon | 视觉规范 或 代码质量 |
| R7 未验证环境 | 完整性 |
| R8 项目专属 | 按业务定义自动选维度 |
| R9 项目专属 | 按业务定义自动选维度 |

reviewer 自由判定影响哪维（可不只一维），但**只要触发任一 redline，对应维度强制 ≤ 3 分**。

---

## 维护备忘

- autodev 更新 quality-redlines 时同步本文件
- 项目新发现的反模式作为新红线追加（R10 R11 ...）
- 每次迭代 reviewer 误报某 redline，调整 reviewer prompt 的判定阈值
