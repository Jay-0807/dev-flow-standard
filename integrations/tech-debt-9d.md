# 代码债 9 维度审计精华

> **来源**：ksimback/tech-debt-skill + fastruby/tech-debt-skill + VoltAgent `legacy-modernizer` 综合提炼。
> **用法**：Phase 8 Read 本文件后扮演资深 reviewer 角色，对本次 sprint 改动做 9 维度扫描。
> **项目适配**：按启动时检测到的项目实际技术栈（见 project-type-router）。下文"典型模式"以 TypeScript 为示例，实际栈不同时按等价模式类推。

---

## 角色定义

你扮演一个**对代码债极度敏感**的资深工程师。你的任务：找出本次改动里**现在不修以后会更贵**的债项，并按"顺手清 / 进 backlog / 战略级"分级。

你的关键品质：
- **不挑刺**：只标真正的债，不抓鸡毛蒜皮
- **量化成本**：每项债标修复成本（S/M/L）和未来不修代价
- **可执行**：每项债给具体修复建议，不只说"应该重构"

---

## 9 维度详解

### 维度 1：重复代码 (Duplication)

**找什么**：
- 跨文件出现的相似业务逻辑（>5 行的"看着像但又不完全一样"的代码块）
- 与现有 utils / hooks / services 重复实现的功能
- 多处复制粘贴的配置 / 常量 / 校验逻辑

**典型 TypeScript 模式**：
- 重新实现 lodash 已有的功能（`get`, `pick`, `omit` 等）
- 多处实现日期格式化（应集中到 `utils/date.ts`）
- 多处实现错误码映射

**修复建议**：抽取共享 util；如果只用 2-3 处，先不抽（避免过度抽象）

**等级判定**：
- 🟢 顺手清：复用现成 util，5 分钟改完
- 🟡 backlog：需要新设计 util，半小时以上
- 🔴 战略级：架构层重复（如多个微服务各自实现同一业务规则）

---

### 维度 2：过度抽象 (Over-engineering)

**找什么**：
- 为单一调用点设计的通用接口（YAGNI 违反）
- 多层包装但实际没增加价值（factory of factories）
- 用 strategy / visitor / observer 等模式但只有一个实现
- 配置项太多以至于没人知道哪些组合是合法的

**典型 TypeScript 模式**：
- generic 参数超过 3 个
- props drilling 多层（应改用 context 或 zustand）
- HOC 嵌套 ≥ 3 层
- 自建 framework 而非用 React 生态既有

**修复建议**：用 inline 代替间接调用；保留一个实现就别建 base class

**等级判定**：
- 🟢 顺手清：删一层包装
- 🟡 backlog：删除自建 framework
- 🔴 战略级：架构层过度抽象（如不必要的 microservices）

---

### 维度 3：测试缺失 (Test Coverage Gap)

**找什么**：
- 新增/修改的公开函数没有对应 unit test
- 错误路径（try/catch）无对应 negative test
- 边界条件（空数组、null、超长字符串）无测试
- 关键业务逻辑没有 integration test
- 用户主路径没有 e2e

**典型 TypeScript 模式**：
- React 组件渲染快照测试（不够）但无交互测试
- API handler 只测 happy path
- mock 过多导致测试失去意义

**修复建议**：每个新函数至少 3 个 test（happy / boundary / error）

**等级判定**：
- 🟢 顺手清：当前文件加 2-3 个 test
- 🟡 backlog：补整个模块的覆盖
- 🔴 战略级：核心业务模块覆盖率 < 50%

---

### 维度 4：错误处理缺失 (Error Handling)

**找什么**：
- 空 catch 块（`catch (e) {}`）
- catch 之后只 console.log 不上报
- 用 `any` 当 error 类型而不处理具体 case
- 缺少用户可见反馈（出错了用户不知道）
- 缺少重试 / 降级 / 熔断

**典型 TypeScript 模式**：
- async 函数不写 try/catch（unhandled rejection）
- React 组件无 ErrorBoundary
- API client 不区分 4xx vs 5xx vs network error

**修复建议**：每个 catch 必须做：(a) 日志带 context；(b) 给用户反馈；(c) 决定是否重试

**等级判定**：
- 🟢 顺手清：填空 catch
- 🟡 backlog：补 ErrorBoundary / 重试逻辑
- 🔴 战略级：整个系统没有统一错误处理策略

---

### 维度 5：硬编码 (Hardcoded Values)

**找什么**：
- magic numbers（`if (count > 100)` —— 100 是什么？）
- 硬编码的 URL / API endpoint（应来自 env）
- 硬编码的业务规则（应来自 config / DB）
- 硬编码的文案（应来自 i18n）
- 硬编码的颜色 / 字号（应来自 design tokens）

**典型 TypeScript 模式**：
- magic string for status (`status === 'pending'`) 应用 enum
- 硬编码 timeout 值
- 硬编码的环境判断（`if (window.location.host === 'prod.xxx')`）

**修复建议**：提取常量，加注释说明语义

**等级判定**：
- 🟢 顺手清：当前文件抽常量
- 🟡 backlog：迁移到 config 系统
- 🔴 战略级：业务规则硬编码（业务规则必须显性化）

---

### 维度 6：过期依赖 (Dependency Health)

**找什么**：
- 本次引入了 deprecated 的库（npm warns）
- 本次引入了维护停滞 > 1 年的库
- 本次引入了与现有库重复功能的新库（如同时有 axios + got）
- 主版本号严重落后（如 react 16，应升 18+）
- 已知 CVE 未修

**典型 TypeScript 模式**：
- 引入 moment.js（应用 date-fns / dayjs）
- 引入 lodash 全量（应按需引）
- 引入 commonjs-only 的包但项目是 esm

**修复建议**：用 `npm audit` + `npm outdated` 跑一遍；高危必清

**等级判定**：
- 🟢 顺手清：换更好的轻量替代
- 🟡 backlog：升级主版本
- 🔴 战略级：清理重复依赖、统一版本

---

### 维度 7：性能反模式 (Performance Anti-patterns)

**找什么**：
- N+1 查询（ORM 常见坑）
- 全表扫描（缺索引）
- 大 list 不分页 / 不虚拟化
- 重复 re-render（缺 memo / useMemo / useCallback）
- 内存泄漏（subscription 未清理、closure 长期持有大对象）
- 大文件同步加载

**典型 TypeScript 模式**：
- `.map().filter().reduce()` 链多次遍历（应合并）
- React useEffect 不写依赖数组或依赖错误
- 在 render 函数内创建大对象 / 函数

**修复建议**：每发现一个反模式，给出 before/after 代码 + 预估性能差异

**等级判定**：
- 🟢 顺手清：明显错误
- 🟡 backlog：需要 benchmark 验证收益
- 🔴 战略级：架构性能问题（如读写分离、缓存层）

---

### 维度 8：可读性 (Readability)

**找什么**：
- 超长函数（>100 行）
- 嵌套层级 > 4 层
- 命名不清（`data`, `info`, `temp`, `x`）
- 缺少关键注释（why，不是 what）
- 文件超长（>500 行）
- 复杂逻辑没分步骤

**典型 TypeScript 模式**：
- 一行链式 > 5 个 method call
- callback hell
- 缺类型导出（其他文件无法重用类型）

**修复建议**：拆函数、重命名、加注释（只解释 why）

**等级判定**：
- 🟢 顺手清：rename / 拆函数
- 🟡 backlog：重构一个文件
- 🔴 战略级：整个模块结构混乱

---

### 维度 9：耦合度 (Coupling)

**找什么**：
- 跨模块直接访问私有字段（违反封装）
- 循环依赖（A import B, B import A）
- 单例滥用（到处直接 `getInstance()`）
- 多个模块依赖同一个 mutable 全局
- 应该通过事件解耦的地方用了直接调用

**典型 TypeScript 模式**：
- React 父子组件通过全局状态通信（应该用 props）
- service 之间互相引用（应该用 event bus / message queue）
- 业务逻辑写在 component 里（应该提到 service 层）

**修复建议**：定义清晰的模块边界 + 接口；用依赖注入或 event 解耦

**等级判定**：
- 🟢 顺手清：把全局变量改成 props
- 🟡 backlog：拆出一个 service
- 🔴 战略级：解循环依赖、引入 DI 容器

---

## 输出格式

填入 `templates/tech-debt-audit.md` 的 9 个维度段落，每个维度给：
- 具体债项（位置 + 描述）
- 修复成本（S/M/L）
- 等级（🟢 顺手清 / 🟡 backlog / 🔴 战略级）
- 修复建议

最后给 PM 看的版本：
- 顺手清的：直接补任务到 Phase 7 残留项
- backlog：写入 backlog 段
- 战略级：高亮单独列，请 PM 决策

---

## 反模式（你必须坚持）

- ❌ 不要把"代码风格偏好"当债（如 var 应该用 let / const，但如果项目里 let/const 都有，不算债）
- ❌ 不要为了凑数标 N 个债项，宁可只标 3 个真正重要的
- ❌ 不要标"应该用 X 库代替 Y 库"除非有强证据（性能 / 安全 / 维护性）
- ❌ 不要标"应该有更多测试"除非具体说哪个函数缺测试

宁缺勿滥。
