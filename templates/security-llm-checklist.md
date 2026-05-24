# 安全审查模板（含 LLM 专项）

> Phase 9 路 C 用。配合 `integrations/owasp-llm-2025.md`。

---

# 安全审查: <feature-name>

**日期**：YYYY-MM-DD
**审查范围**：本次 sprint 改动
**审查者**：自动（/security-review + LLM 专项 + 合规 agent）

---

## A. 经典 Web 安全（OWASP Top 10）

| 项 | 名称 | 是否触及 | 状态 | 备注 |
|---|---|---|---|---|
| A1 | Broken Access Control | [是/否] | ✅/⚠️/❌ | |
| A2 | Cryptographic Failures | | | |
| A3 | Injection（SQL/Command/LDAP） | | | |
| A4 | Insecure Design | | | |
| A5 | Security Misconfiguration | | | |
| A6 | Vulnerable Components | | | |
| A7 | Auth Failures | | | |
| A8 | Software & Data Integrity | | | |
| A9 | Logging & Monitoring | | | |
| A10 | SSRF | | | |

**发现的问题**：
- [问题 1]: [位置] - [严重度]
- [问题 2]: ...

---

## B. LLM 专项安全（OWASP LLM Top 10 - 2025）

| 项 | 名称 | 是否触及 | 状态 | 备注 |
|---|---|---|---|---|
| LLM01 | Prompt Injection | [是/否] | ✅/⚠️/❌ | 直接 / 间接均查 |
| LLM02 | Sensitive Info Disclosure | | | |
| LLM03 | Supply Chain（模型/嵌入服务） | | | |
| LLM04 | Data & Model Poisoning | | | |
| LLM05 | Improper Output Handling | | | XSS via LLM 输出 |
| LLM06 | Excessive Agency（agent 权限） | | | |
| LLM07 | System Prompt Leakage | | | |
| LLM08 | Vector & Embedding Weaknesses | | | |
| LLM09 | Misinformation（hallucination） | | | |
| LLM10 | Unbounded Consumption | | | token/调用滥用 |

**LLM 风险评估**：
- 本次改动是否引入新的 LLM 调用：[是 / 否]
- 是否暴露 system prompt：[否 / 风险点]
- 用户输入是否会进入 LLM 上下文：[是 / 否]
- 若是：注入防护策略 = [描述]
- LLM 输出是否会被直接渲染到 UI：[是 / 否]
- 若是：sanitize 策略 = [描述]
- 是否有 token 消耗上限 / rate limit：[是 / 否]

---

## C. 合规审

### C.1 数据合规
| 法规 | 是否适用 | 状态 | 备注 |
|---|---|---|---|
| GDPR | | | |
| PIPL（中国） | | | 关键 |
| CCPA | | | |
| 行业专项（金融/医疗） | | | |

### C.2 项目合同约束（如适用）
若本功能服务于具体客户，比对其合同：
- A1.4 六类人工保留事项：是否被遵守
- 9.1 "AI 仅作建议"原则：是否被遵守
- 数据本地化要求：是否被遵守

### C.3 跨境合规
- 数据是否跨境传输：[是 / 否]
- 若是：合规依据 = [SCC / 白名单 / 等保]

---

## D. 综合发现

### D.1 阻塞发布的 must-fix
| ID | 来源 | 严重度 | 描述 | 修复建议 | 责任 |
|---|---|---|---|---|---|
| S-001 | OWASP A3 | 🔴 高 | ... | ... | |

### D.2 本 sprint 内 should-fix
| ID | 来源 | 严重度 | 描述 | 修复建议 |
|---|---|---|---|---|

### D.3 进 backlog 的 nice-to-fix
| ID | 来源 | 描述 |
|---|---|---|

---

## E. 后续动作

- [ ] must-fix 全部修完后重跑本表
- [ ] should-fix 追加到任务列表
- [ ] nice-to-fix 写入 backlog
- [ ] 若有 must-fix → 阻塞 Phase 10 验收

---

## 附录：原始输出

<details>
<summary>/security-review 原始输出</summary>
[贴在这]
</details>

<details>
<summary>LLM 专项审查原始输出</summary>
[贴在这]
</details>

<details>
<summary>合规 agent 原始输出</summary>
[贴在这]
</details>
