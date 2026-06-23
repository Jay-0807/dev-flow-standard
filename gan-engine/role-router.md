# GAN 角色路由表

> **作用**：GAN 引擎接到任务时，按本文件路由出：① 该任务的 4 维打分维度 ② reviewer 应注入哪些视角段（其他 .md 文件）
> **维护**：每加一类新任务（如新增 phase），在本文件加一行

---

## 任务类型路由表

每个 task_type 配置一行 YAML-like 块：

```yaml
- task_type: prd-writing
  trigger_keywords: ["写 PRD", "PRD 撰写", "Phase 2"]
  4_dimensions:
    - 需求覆盖度  # 是否覆盖 PRD 应有的全部信息
    - 技术可行性  # 是否考虑了实现路径
    - 可执行度    # 后续 phase 能不能据此干活
    - 风险与依赖识别  # 是否标出已知风险和外部依赖
  perspective_files:  # reviewer prompt 注入的视角段
    - principles/karpathy-llm-coding.md      # Karpathy 4 原则
    - <project-business-context>.md（项目自定，无则跳过）       # 项目业务上下文（如有：业务规则显性化 / 人工保留点 / 数据来源标注）
    - <project-positioning>.md（项目自定，无则跳过）  # 目标客户 + 0 代码 PM 视角
  generator_instruction_extras:
    - "PRD 必须含隐形信息显性化三字段在数据/规则层（confidence/human_review/data_source）"
    - "PRD 必须含 ≥3 条 ⭐ 标关键任务给 Phase 7"
```

```yaml
- task_type: brainstorm-diverge
  trigger_keywords: ["方案发散", "DIVERGE", "Phase 2.5 DIVERGE"]
  4_dimensions:
    - 方案差异度   # 反"只差框架名"反模式
    - 覆盖广度    # 是否真的有 3-5 个可行方向
    - 调研对齐度   # 与 Phase 1.5 用户研究的对齐
    - 风险均衡    # 不能全都是高风险或全都是保守
  perspective_files:
    - principles/karpathy-llm-coding.md
    - <project-business-context>.md（项目自定，无则跳过）
  generator_instruction_extras:
    - "至少给 3 个架构层真正差异的方案（不许只差框架名）"
    - "每个方案标关键依赖 + 关键风险"

- task_type: brainstorm-converge
  trigger_keywords: ["方案收敛", "CONVERGE", "Phase 2.5 CONVERGE"]
  4_dimensions:
    - 选择理由强度  # 是否真有逻辑链
    - 否决理由明确度 # 被否方案理由是否站得住
    - 风险消化     # 选中方案的关键风险是否有应对
    - 与上游对齐    # 与 PRD + 用户研究一致
  perspective_files:
    - principles/karpathy-llm-coding.md
    - <project-business-context>.md（项目自定，无则跳过）
```

```yaml
- task_type: architecture-design
  trigger_keywords: ["架构设计", "Phase 4", "技术选型"]
  4_dimensions:
    - 完整性    # 是否覆盖前端/后端/DB/集成/部署
    - 鲁棒性    # 异常路径 / 降级 / 重试设计
    - 一致性    # 与既有系统一致 + 内部命名一致
    - 简洁性    # Karpathy Simplicity First
  perspective_files:
    - principles/karpathy-llm-coding.md
    - integrations/owasp-llm-2025.md
    - integrations/tech-debt-9d.md
    - integrations/database-architect.md
    - <project-positioning>.md（项目自定，无则跳过）
```

```yaml
- task_type: api-design
  trigger_keywords: ["API 设计", "Phase 4 §2 API", "OpenAPI", "REST 契约", "GraphQL 契约", "gRPC 契约"]
  4_dimensions:
    - 资源建模    # REST/GraphQL/RPC 资源粒度对不对
    - 错误处理    # 错误码全 + RFC 9457
    - 协议合规    # 项目自定协议合规（如有）：若项目有自定协议则遵循，否则用标准 REST/gRPC/事件
    - 演进策略    # 版本兼容 + deprecation 路径
  perspective_files:
    - principles/karpathy-llm-coding.md
    - integrations/api-architect.md       # 9 维度审计
    - <project-api-standards>.md（如有项目专属 API 规约，否则跳过）
    - integrations/owasp-llm-2025.md
  generator_instruction_extras:
    - "必须先从 UI 反推 API（autodev-api 铁律），不许凭空设计 endpoint"
    - "每个 endpoint 含：path / method / req / resp / errors / auth / rate-limit"
    - "项目类型敏感：Web 全栈→REST 为主 / B 端 SaaS→REST+GraphQL / AI 原生→加 stream/SSE/项目自定协议（如有，否则标准 RPC）"
```

```yaml
- task_type: ux-flow
  trigger_keywords: ["UX 设计", "Phase 5 §1 UX", "用户流程"]
  4_dimensions:
    - 视觉层级    # 信息密度 + 主次区分
    - 交互流畅    # 用户路径无跳跃
    - 状态完整    # 空/错/loading 三态覆盖（autodev 强制）
    - 一致性     # 与既有页面 + 项目设计系统一致
  perspective_files:
    - principles/karpathy-llm-coding.md
    - <project-business-context>.md（如有项目业务上下文，否则跳过）
  generator_instruction_extras:
    - "task-first 信息架构（autodev-ui 哲学）"
    - "每个页面必须设计空/错/loading 三态"
    - "反模式：default to full info display（autodev-ui 反对全量展示）"
```

```yaml
- task_type: task-breakdown
  trigger_keywords: ["任务分解", "Phase 6", "sprint"]
  4_dimensions:
    - 颗粒度合适  # 不能太大也不能太碎
    - AC 可验证   # 每个任务有具体可测 AC
    - 依赖清晰   # 任务间依赖图明确
    - 并行机会   # 哪些可并行被标出
  perspective_files:
    - principles/karpathy-llm-coding.md
  generator_instruction_extras:
    - "每个任务标 status: pending / acceptance_criteria 列表（autodev-plan 契约风格）"
    - "标 ⭐ 关键任务（PRD 标的） + mechanical:true 机械任务（跳过 GAN）"
```

```yaml
- task_type: code-task
  trigger_keywords: ["代码任务", "Phase 7 实施", "/autodev-iterate"]
  4_dimensions:
    - 完整性    # 实现是否完整（无 placeholder / TODO）
    - 鲁棒性    # 异常路径 + 边界 case
    - 设计一致  # 与 INDEX/RULES + 既有代码一致
    - 代码质量  # Karpathy 4 + 7 redlines
  perspective_files:
    - principles/karpathy-llm-coding.md
    - gan-engine/quality-redlines.md      # 7 redlines（特别针对代码）
    - integrations/owasp-llm-2025.md
  generator_instruction_extras:
    - "前置加载 iteration-vault/INDEX.md + RULES.md，不读全量设计文档"
    - "本任务的 AC 全部通过（acceptance_criteria 列表）"
    - "PM 标 ⭐ 关键任务时，generator 多花一倍时间想边界 case"
```

```yaml
- task_type: ui-task   # code-task 的 add-on（不替代，叠加）
  trigger_keywords: ["UI 代码", "前端任务", "git diff 含 .tsx/.css"]
  trigger_condition: "git diff --name-only HEAD~1 | grep -E '\\.(tsx|jsx|vue|css|scss)$'"
  4_dimensions:
    - 视觉规范  # 与 ui.md 一致
    - 交互响应  # 真正可点 + 响应符合预期
    - 状态完整  # 空/错/loading 实际渲染
    - 与 5a 一致 # 不背离 UX flow
  perspective_files:
    - principles/karpathy-llm-coding.md
  runtime_evidence_required:
    - screenshot 1440×900
    - screenshot 375×812
    - DOM snapshot
    - state triggers (loading / empty / error / success)
  max_rounds_override: 3   # UI Add-on 上限 3 轮（autodev 实证 + 控成本）
```

```yaml
- task_type: release-notes
  trigger_keywords: ["release notes", "Phase 11", "发布说明"]
  4_dimensions:
    - 用户价值表达  # PM/用户能看懂"这次升级让我得到什么"
    - 风险透明度    # 已知问题 + 影响范围 + 应对
    - 行动指引     # 用户/开发者下一步该做什么
    - 文档完整     # 含本次新增/修改/废弃接口的链接
  perspective_files:
    - principles/karpathy-llm-coding.md
    - <project-positioning>.md（项目自定，无则跳过）
  generator_instruction_extras:
    - "引用 Phase 12.5 早晨复盘（真人用户验收角色）中的'用户最爱'+'用户最痛'"
    - "不要技术术语爆炸，PM 视角"
```

```yaml
- task_type: autopilot-seed-prd
  trigger_keywords: ["autopilot 种子", "候选 → PRD", "13-autopilot-handoff"]
  4_dimensions:
    - 候选合理性  # 是否真值得做（vs 误报）
    - 范围控制    # 不许"顺便加 10 个功能"
    - 红线规避    # 不许触 forbidden_paths
    - PM 可接受度 # PM 不在场，但能预测 PM 会接受
  perspective_files:
    - principles/karpathy-llm-coding.md
    - autopilot/safety-brakes.md       # 失控防护规则
    - <project-positioning>.md（项目自定，无则跳过）
  generator_instruction_extras:
    - "autopilot 候选种子 PRD = 唯一无 PM 监督的 GAN，质量门必须最严"
    - "blast_radius 必须 < 30 文件 / 500 行 / 不触 forbidden_paths"

- task_type: user-research
  trigger_keywords: ["用户研究", "Phase 1.5"]
  4_dimensions:
    - 画像具体度  # personas 是否具体可信
    - 痛点 ranking 站得住 # 排序有数据/调研支撑
    - 旅程完整    # user journey 闭环
    - 心智隐形识别  # 用户心智隐形信息识别（应用到非 UI 层）
  perspective_files:
    - principles/karpathy-llm-coding.md
    - <project-business-context>.md（项目自定，无则跳过）     # 仅 1-3 维 + 心智识别（非 UI 应用）
    - <project-positioning>.md（项目自定，无则跳过）

- task_type: decision-replay-analysis
  trigger_keywords: ["决策回放", "replay from decision"]
  4_dimensions:
    - 回放安全    # vault checkpoint 数据完整
    - 替代方案    # PM 指定方向是否有替代
    - 状态完整    # 回放后下游 phase 的状态预测
    - 收益估计    # 重跑 N 轮的成本 vs 期望收益
  perspective_files:
    - principles/karpathy-llm-coding.md
    - decision-replay.md                  # 回放机制规则
```

---

## 触发关键词 → task_type 映射规则

GAN 引擎接到 phase 的调用时，先按 trigger_keywords 匹配；匹配多个时取**最具体**那个。

例：调用提到"Phase 4 §2 API 设计" → 匹配 `api-design`（更具体），不匹配 `architecture-design`。

匹配失败时 fallback 到通用 `prd-writing` 配置 + 在 trace.md 标 `task_type_unmatched`。

---

## 视角段加载机制

`perspective_files` 列出的 .md 文件由 reviewer prompt 在动态加载阶段拼接（见 `reviewer-skeptical.md`）。

注意：
- 文件路径相对于本 skill 根目录
- 加载时按列表顺序拼接（前面的视角更优先被引用）
- 加载失败（文件不存在）时跳过该视角 + 在 trace.md 标 `perspective_missing: <path>`

---

## 任务类型经验沉淀（每次迭代后追加）

格式：

```
- task_type: <type>
  iteration: <name>
  rounds: <N>
  pivot_triggered: <yes/no>
  notes: <observations>
```

例：
```
- task_type: code-task
  iteration: 2026-05-add-payment
  rounds: 4
  pivot_triggered: yes
  notes: PIVOT 后 1 轮即 PASS。原方案漏了人工保留点（隐形信息显性化的人工兜底维度没放）。
```

跑过 ≥ 5 次迭代后，从经验沉淀里抽规律，反馈到 trigger_keywords / 4_dimensions / perspective_files。
