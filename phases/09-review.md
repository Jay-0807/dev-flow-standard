# Phase 9 — 三路并行审查（自审 + 对抗审 + 安全审）

## 目标
对本次实施代码做 **3 路并行审查**：常规自审、对抗式怀疑审、安全 + 合规审。三个视角互补，把"自己看着没问题"的盲点挖出来。

## 输入
- 本次所有改动（已 merge 回主分支）
- `iteration-vault/08-tech-debt-audit.md`（已知问题清单）
- `integrations/owasp-llm-2025.md`（LLM 安全检查清单，本机空白补充）
- `principles/karpathy-llm-coding.md`（**必读**：对抗审专门检查"假设是否被默默传播"）

## 工作流（并行 3 路）

### 路 A：常规自审（code-reviewer）

用 Agent 工具：`subagent_type: "code-reviewer"`。
喂给它：
- 本次改动的 diff
- 04-architecture.md（架构期望）
- 06-task-breakdown.md（任务清单）

让它检查：
- 实现是否符合架构设计
- 命名 / 风格 / 注释 / 错误处理
- 边界条件 / 输入校验
- 与现有代码模式的一致性

同时触发 `superpowers:requesting-code-review` 流程，让自审有结构化的输出。

输出到：`iteration-vault/09-review-reports/self-review.md`

### 路 B：对抗审（/autodev-review）

触发 `/autodev-review`。
**特点**：spawn 独立的"怀疑主义 reviewer" agent，从 4 维度严苛打分：
1. 安全性（是否引入新漏洞）
2. 性能（是否埋藏性能炸弹）
3. 可维护性（半年后接手的人能懂吗）
4. 风险（部署后最坏情况是什么）

**额外强制：Karpathy 4 原则审查**（spawn 给对抗 reviewer 的 prompt 必含）：
- 【Think 检查】实现里有没有"做了假设但没浮现给主线程"的痕迹？看 commit message / 代码注释 / PR 描述能不能反推出原假设清单？
- 【Simplicity 检查】有没有为"将来可能需要"引入的抽象/接口/参数？有没有用模式装饰本可以直白的代码？
- 【Surgical 检查】diff 里多少行是本次任务真正需要的？多少行是"顺手"？
- 【Goal 检查】实现是否到"代码写好"就停了？还是真的循环跑到 AC 全部通过？

每个维度独立给 1-10 分。

每个维度给 1-10 分 + 关键问题列表。

输出到：`iteration-vault/09-review-reports/adversarial-review.md`

### 路 C：安全 + 合规审

**3 个子任务并行**：

**C-1：常规 web 安全**
触发 `/security-review`（本机内置）。
扫 OWASP Top10 经典项：注入、XSS、CSRF、auth/session、敏感数据暴露、IDOR、配置错误等。

**C-2：LLM 专属安全**
Read `integrations/owasp-llm-2025.md`，由本 skill 主线程亲自扮演 LLM 安全审查官，按以下 10 项检查（OWASP LLM Top 10 2025）：
1. Prompt Injection（直接 / 间接）
2. Sensitive Information Disclosure
3. Supply Chain（模型 / 嵌入服务来源）
4. Data and Model Poisoning
5. Improper Output Handling（XSS via LLM output）
6. Excessive Agency（agent 权限过大）
7. System Prompt Leakage
8. Vector and Embedding Weaknesses
9. Misinformation（hallucination 风险）
10. Unbounded Consumption（token / API 调用滥用）

逐项标记本次改动是否触及、是否有风险。

**C-3：合规审**
若本次改动涉及用户数据 / 跨境 / 金融，调 Agent 工具：`subagent_type: "support-legal-compliance-checker"`。
检查 GDPR / PIPL / 数据本地化 / 跨境合规。
若 项目内有合同（如电商客户合同），同时比对合同里的人工保留点是否被遵守。

输出汇总到：`iteration-vault/09-review-reports/security-review.md`

## 综合处理

3 路并行完成后，主线程做汇总：
1. 提取**所有标红的问题**（严重 / 高优先级）
2. 判断哪些是 **must-fix-before-release**（阻塞）、哪些是 **should-fix-this-sprint**（不阻塞但要处理）、哪些是 **nice-to-fix**
3. 对 must-fix：立刻回到 Phase 7 修；修完重跑 Phase 9 对应路
4. 对 should-fix：追加到 Phase 6 的剩余任务，本 sprint 内处理
5. 对 nice-to-fix：写入 backlog

## 产出

**目录**：`iteration-vault/09-review-reports/`
- `self-review.md`
- `adversarial-review.md`
- `security-review.md`
- `summary.md`（主线程汇总，列 must/should/nice 三档）

**对 PM 的摘要**：
```
三路审查完成。
- 自审：[N] 个建议，[M] 个已采纳
- 对抗审：4 维度评分 [安全 X / 性能 Y / 可维护 Z / 风险 W]
- 安全审：经典 OWASP [N] 项、LLM 专项 [M] 项、合规 [K] 项

阻塞发布的 must-fix：[N] 项 [简述]
建议本 sprint 内处理的 should-fix：[N] 项
进入多层验收阶段（或：先修 must-fix 再继续）。
```

## 关卡处理
本阶段**不是**⛳ 关卡。但若 must-fix ≥ 3 项，暂停后续，先回 Phase 7 修；must-fix = 0 则直接进 Phase 10。

## 失败回退
- 自审 / 对抗审 / 安全审三方意见严重冲突 → 主线程裁决，必要时升级让 PM 拍板
- 合规审发现根本性违规（如改动违反合同人工保留条款）→ 立刻停止后续，回到 Phase 2 重审 PRD
