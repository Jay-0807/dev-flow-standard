# Test Planner 集成精华

> **来源**：levnikolaevich/claude-code-skills 的 test-planner + test-auditor，wshobson/agents `test-automator`、`qa-engineer` 综合提炼。
> **用法**：Phase 10 验收阶段 Read 本文件后扮演测试策略师角色，识别测试缺口并制定补全策略。
> **项目适配**：偏 TypeScript / Jest / Playwright 栈。

---

## 角色定义

你扮演**资深测试架构师**。本机的 `/autodev-verify` 偏"执行"（跑测试 + 验收 5 层），但缺策略层。你的任务：

1. **诊断现有覆盖**：哪些层（单元 / 集成 / e2e）有缺口
2. **补全建议**：什么测试该加、加在哪、价值大小
3. **平衡**：不追求 100% 覆盖，追求"风险大的地方有测试"

---

## 测试金字塔（本 skill 默认）

```
        /\
       /  \    e2e (5%)
      /----\
     /      \  集成 (15%)
    /--------\
   /          \ 单元 (80%)
  /____________\
```

但不是死规则。具体看：
- **业务复杂度高**的模块 → 单元测试占比更高
- **多服务协作多**的场景 → 集成测试占比更高
- **用户主路径关键**的产品 → e2e 占比更高

对于 AI 原生项目，**集成测试和 e2e 测试相对更重要**（因为业务逻辑跨 agent 跨 service，单元测试覆盖不到 agent 间真实通信）。

---

## 各层测试的目标 & 边界

### 单元测试 (Unit)
- **目标**：单个函数 / 类的内部逻辑正确
- **边界**：所有外部依赖 mock（DB / API / file system / 时间）
- **执行速度**：< 100ms 单个
- **频率**：每次提交跑全量

**项目典型单元测试场景**：
- 业务规则计算函数
- 数据格式化 / 校验
- LLM prompt 拼接 / 解析
- 项目业务消息 marshalling

### 集成测试 (Integration)
- **目标**：模块间真实协作正确
- **边界**：内部依赖真实（local DB / local services），外部依赖 mock（第三方 API）
- **执行速度**：< 5s 单个
- **频率**：每次 PR 跑

**项目典型集成测试场景**：
- 完整 API 调用链（HTTP → handler → service → DB）
- 项目业务消息从发送到接收
- LLM 调用集成（用 mock LLM 或 cassette 录制）

### 端到端测试 (E2E)
- **目标**：用户视角主路径走通
- **边界**：所有都真实（dev 环境）
- **执行速度**：< 30s 单个
- **频率**：每次合并主分支跑

**项目典型 e2e 测试场景**：
- 用户登录 → 创建组织 → 添加 agent → 跑一次任务 这种 happy path
- 关键失败路径（agent 失败应有兜底）

---

## 5 维度缺口分析

对本次改动，按以下 5 维度找测试缺口：

### 维度 1：新增函数 / 接口
- 每个新增的 public 函数 → 至少 1 个单元测试
- 每个新增的 API → 至少 1 个集成测试（happy path）

### 维度 2：错误路径
- 每个 try/catch → 一个 negative test
- 每个 if/else 的失败分支 → 一个测试覆盖
- 每个新增的错误码 → 一个测试触发它

### 维度 3：边界条件
- 空输入 / null / undefined / 空数组
- 极大值 / 极小值 / 负数 / 0
- 超长字符串 / 特殊字符 / Unicode
- 时区边界 / 闰年 / 月底

### 维度 4：并发 & 时序
- 多用户同时操作（如抢券、抢库存）
- 异步操作的乱序到达
- 长事务下的隔离性

### 维度 5：与现有功能的回归
- 改了 A 后，B / C / D 是否还正常
- 影响面分析里列出的"高风险"项 → 必须有回归测试

---

## 输出格式（嵌入 10-verification-report.md 的"测试覆盖"段）

```markdown
## 测试覆盖

### 整体
- 单元测试：[N] 条新增 / [M] 条修改，覆盖率 [X]%（关键函数 [Y]%）
- 集成测试：[N] 条新增
- e2e 测试：[N] 条新增
- 总执行时间：[N] 秒

### 5 维度缺口分析

| 维度 | 已覆盖 | 缺口 | 处理 |
|---|---|---|---|
| 1 新增函数 | [N]/[M] | [缺] | 本 sprint 补 / backlog |
| 2 错误路径 | [N]/[M] | [缺] | |
| 3 边界条件 | [N]/[M] | [缺] | |
| 4 并发时序 | [N]/[M] | [缺] | |
| 5 回归 | [N]/[M] | [缺] | |

### 必补（must-have，阻塞发布）
- [ ] 测试 1：[描述]
- [ ] 测试 2：[描述]

### 应补（should-have，本 sprint 内）
- [ ] 测试 3：[描述]

### 可后补（nice-to-have，backlog）
- [ ] 测试 4：[描述]
```

---

## 测试质量准则

写测试时遵循 **AAA**（Arrange / Act / Assert）+ **FIRST**（Fast / Isolated / Repeatable / Self-validating / Timely）。

### 反模式（必拒绝）
- ❌ 一个测试断言 10 件事（拆开成多个测试）
- ❌ 依赖测试执行顺序
- ❌ 用 `sleep()` 等待异步（用 await + done 回调）
- ❌ mock 自己刚写的函数（等于测试 mock 而非测试代码）
- ❌ 测试中跑真实 LLM（慢 + 不稳定 + 费钱）—— 用 cassette 或 mock

### 推荐模式
- ✅ Test naming: `describe('FunctionName').it('does X when Y')`
- ✅ Snapshot 测试只用于稳定的 UI 输出
- ✅ Property-based testing 用于复杂业务规则（fast-check / hypothesis）
- ✅ Contract testing 用于多服务协作（Pact）

---

## 与 /autodev-verify 的关系

`/autodev-verify` 是本机的"5 层验收"工具：
- Layer 1 契约
- Layer 2 红线
- Layer 3 静态
- Layer 4 运行时
- Layer 5 acceptance

本 skill（test-planner 角色）补的是**它跑测试之前的"测试缺口诊断 + 补全策略"**。

调用顺序：
1. 本 skill 跑诊断 → 输出"必补 / 应补 / 可后补"清单
2. 必补的 → 主线程或 spawn agent 实际写测试代码
3. 测试写完后 → `/autodev-verify` 跑全 5 层
4. 跑通 → Phase 10 关卡

---

## 项目专属测试约束

### 项目业务协议（如有）消息测试
- 必须有"发 → 收 → 处理 → 回执"完整链路 e2e
- 必须有"消息丢失" / "消息重复" / "消息乱序" 三种 negative case

### LLM 调用测试
- 推荐用 cassette（pre-recorded responses）
- 关键 prompt 的输出格式要做 schema 校验测试
- prompt injection 防御要有专门 negative test

### 业务规则测试（r4 显性化）
- 每条业务规则（business_rule 表里的）都要有对应单元测试
- "触发条件 / 决策动作 / 失败兜底"三段必测

### 多租户隔离测试
- 任何涉及多租户数据的 API → 必须有"租户 A 不能看到 / 改到租户 B 数据"的 negative test
