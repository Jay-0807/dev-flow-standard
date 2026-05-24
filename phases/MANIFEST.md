# Phases Manifest（17 步骤的输入输出契约）

> **对 PM 暴露的所有文档（RUN-LOG / DEBUG-TRACE / 决策日志 / 早晨复盘）必须用业务名 + emoji，不能用 Phase X 编号。**

## 主表（17 步骤）

| 业务名 | 内部编号 | 输入 | 产出 | 关卡 | AI 互审 | 红线 | 独立命令 |
|---|---|---|---|---|---|---|---|
| 💬 需求澄清 | 1 | PM 原话 | 01-clarified-requirement.md | - | - | - | - |
| 👥 用户研究 | 1.5 | 01-* | 01.5-user-research.md | - | - | - | - |
| 📝 PRD 撰写 | 2 | 01-* + 01.5-* | 02-PRD.md | ⛳1 | ✅ | - | /u-prd |
| 💡 方案发散 | 2.5 | 02-PRD | 02.5-{diverge,converge}.md | - | ✅ | - | - |
| 📐 影响面分析 | 3 | 02-PRD | 03-impact.md + 03-adr.md | - | - | - | - |
| 🏗️ 架构与接口设计 | 4 | 03-* + 05-* | 04-architecture-and-api.md + spec/postman 衍生 | - | ✅ | 🚨 R1 | - |
| 🎨 界面设计 | 5 | 01.5+02+04 | 05-interface-design.md + wireframes/ | - | ✅ | - | - |
| 📚 设计压缩成开发地图 | 5.9 | 04+05 | INDEX.md + RULES.md | - | - | - | /u-compress |
| 📋 任务拆解 | 6 | INDEX+RULES+PRD | 06-task-breakdown.md | - | ✅ | - | - |
| ⌨️ 代码实施 | 7 | 06-* | 07-implementation-log.md | - | ✅ | 🚨 R4 | - |
| 🧹 代码债扫描 | 8 | code | 08-tech-debt-audit.md | - | - | - | /u-debt |
| 🔍 多路审查 | 9 | code+07 | 09-review-reports/ | - | (全局) | 🚨 R2 | - |
| 🛡️ 上线前质量检查 | 10 | all | 10-verification-report.md | - | - | 🚨 R3 | /u-verify |
| 📢 发布说明 | 11 | all | 11-release-notes.md | - | ✅ | - | - |
| 🔄 文档代码漂移检测 | 11.5 | code vs docs | 11.5-sync-report.md | - | - | 🚨 R4 | /u-sync |
| 🚀 git 发版 | 12 | all | 12-release.md + Release PR | - | - | - | - |
| ☀️ 早晨复盘 | 12.5 | all | 12.5-morning-review.md | ⛳2 | - | - | - |

## 业务名 ↔ 文件名 ↔ 编号映射

| 业务名 | 内部 phase 文件 | 内部编号（旧 → 新）|
|---|---|---|
| 💬 需求澄清 | phases/01-clarify.md | (0+1) → 1 |
| 🏗️ 架构与接口设计 | phases/04-architecture-and-api.md | (4+4.5) → 4 |
| 🎨 界面设计 | phases/05-interface-design.md | (5a+5b) → 5 |

## 已删除（不再存在）

- ❌ Phase 10.5 真人用户验收 — ⛳2 早晨复盘已覆盖该角色

## 双向依赖说明

- **Phase 4 §2 API 设计 ← Phase 5 §1 UX wireframe**
  - API 必须从 UI 反推（autodev-api 铁律）
  - 实际操作顺序：Phase 4 §1 大架构 → Phase 5 §1 UX → Phase 4 §2 API 设计 → Phase 5 §2 UI Spec
  - 看似交叉，实际是两个 phase 内部的子步骤穿插

## 完成判定的硬约束

| 步骤 | 通过条件 |
|---|---|
| 📝 PRD 撰写 | PM 选 ✅ 通过 |
| 🏗️ 架构与接口设计 | GAN PASS + R1 检查通过 |
| 🎨 界面设计 | GAN PASS + WCAG AA 100% + §1→§2 可追溯性 ≥ 90% |
| 📋 任务拆解 | 估算工时给出，GAN PASS |
| ⌨️ 代码实施 | 所有任务 ✅，R4 检查通过 |
| 🔍 多路审查 | R2 安全 must-fix = 0 |
| 🛡️ 上线前质量检查 | 五层全 ✅，红线编号清单为空 |
| 🔄 文档代码漂移检测 | 无 ERROR 级漂移 |
| ☀️ 早晨复盘 | PM 选 ✅ merge |

## 红线触发汇总

| 红线 | 触发位置 | 触发条件 |
|---|---|---|
| 🚨 R1 重大架构冲突 | 🏗️ 架构与接口设计 | vendor lock-in / 协议冲突 / 重构核心 ≥ 100 行 / 存量违反协议 ≥ 5 项 / breaking change |
| 🚨 R2 安全 must-fix > 3 | 🔍 多路审查 | 安全审查发现 ≥ 3 项阻塞发布 |
| 🚨 R3 验收 3 次重试仍挂 | 🛡️ 上线前质量检查 | 五层验收任一层失败 3 次回退仍未通过 |
| 🚨 R4 必须删除既有功能 / 漂移 ERROR | ⌨️ 代码实施 / 🔄 漂移检测 | 必须删除既有功能 / 漂移检测 ERROR 级 |

任一红线触发：暂停 + 写 ESCALATION-R[X].md + 同步写 DEBUG-TRACE.md E[NNN] + 等 PM 决策。

详见 `gates.md` 红线 escalation 标准流程。
