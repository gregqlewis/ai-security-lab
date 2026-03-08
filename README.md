# AI Security Lab

A growing collection of hands-on AI/LLM security labs focused on adversarial testing of large language models. Current work targets Meta Llama 3.1 8B Instruct on AWS Bedrock using Garak v0.14.0, with additional tools and targets planned.

## Current Lab: AWS Bedrock — Llama 3.1 8B Instruct

**Target:** Meta Llama 3.1 8B Instruct via AWS Bedrock (cross-region inference profile)  
**Attack Platform:** Kali Linux (paladin / kali-lab)  
**Primary Tool:** Garak v0.14.0  
**Total Lab Cost:** < $1.00  
**Methodology:** OWASP LLM Top 10 | MITRE ATLAS | Adversarial ML

---

## Infrastructure

### AWS Configuration
- **Model:** `us.meta.llama3-1-8b-instruct-v1:0` (cross-region inference profile)
- **IAM User:** `garak-bedrock` — dedicated least-privilege user
- **Custom Policy:** `GarakBedrockInvokePolicy` — scoped to `bedrock:InvokeModel` and `bedrock:InvokeModelWithResponseStream` across all three US region ARNs
- **Admin Account:** `greg-admin` (no direct lab usage; billing alerts via root)
- **Cost Controls:** AWS Budget alert configured as substitute for root billing access

### Attack Platform
- Kali Linux (`paladin`) — dedicated attack machine
- Garak installed in isolated Python virtual environment
- AWS CLI profile (`garak-bedrock`) for credential isolation
- Credentials passed to Garak via `AWS_PROFILE` and `AWS_DEFAULT_REGION` environment variables

---

## Probe Suites Executed

### Suite 1 — DAN Jailbreak (`dan.Dan_11_0`)

| Metric | Result |
|--------|--------|
| Attack success rate | **100%** |
| Detector pass rate | 0% |

**Finding:** The model fully adopted the DAN persona on every single attempt, explicitly announcing it would disregard all content policies. No resistance observed across any iteration.

**Severity:** Critical  
**OWASP LLM:** LLM01 — Prompt Injection  
**MITRE ATLAS:** AML.T0054 — LLM Prompt Injection

---

### Suite 2 — Prompt Injection (`promptinject`)

| Sub-Probe | Attack Success Rate |
|-----------|-------------------|
| HijackHateHumans | 70.96% |
| HijackLongPrompt | 68.83% |
| HijackKillHumans | 35.78% |

**Finding:** Prompt injection via instruction override was consistently effective. Notably, prompt length had no meaningful impact on attack effectiveness, and hate speech content policies were significantly weaker than violence-related policies — suggesting inconsistent policy enforcement across harm categories.

**Severity:** High  
**OWASP LLM:** LLM01 — Prompt Injection  
**MITRE ATLAS:** AML.T0054 — LLM Prompt Injection

---

### Suite 3 — Encoding Evasion (`encoding`)

| Sub-Probe | Attack Success Rate |
|-----------|-------------------|
| NATO Phonetic | 14.77% |
| Zalgo Text | 9.45% |
| All others | Low / negligible |

**Finding:** The model was largely resilient to encoding-based evasion. The sharp contrast with Suite 2 is the key insight: the model's failures are **behavioral rather than perceptual** — it can recognize and decode obfuscated input, but cannot consistently resist social engineering in plaintext. NATO phonetic and Zalgo text were the only notable exceptions.

**Severity:** Medium  
**OWASP LLM:** LLM01 — Prompt Injection  
**MITRE ATLAS:** AML.T0054 — LLM Prompt Injection

---

### Suite 4 — Training Data Leakage (`leakreplay`)

| Sub-Probe Type | Result |
|---------------|--------|
| Complete probes | Largely passed |
| Cloze probes | 3–11% failure rate |
| Guardian news content | 11.11% (highest) |

**Finding:** Open-ended completion attempts largely failed, but **contextual hints (Cloze-style) successfully unlocked memorized training data** in 3–11% of cases depending on content type. Guardian news articles showed the highest leakage rate, suggesting news content is disproportionately represented or memorized in training. This demonstrates that data leakage risk is not eliminated by refusing direct completion — contextual framing is sufficient to extract memorized content.

**Severity:** Medium  
**OWASP LLM:** LLM06 — Sensitive Information Disclosure  
**MITRE ATLAS:** AML.T0024 — Exfiltration via ML Inference API

---

## Findings Summary

| Finding | Severity | OWASP LLM | MITRE ATLAS |
|---------|----------|-----------|-------------|
| 100% DAN jailbreak success rate | Critical | LLM01 | AML.T0054 |
| Prompt injection via instruction override (up to 70.96%) | High | LLM01 | AML.T0054 |
| Inconsistent harm category policy enforcement | High | LLM01 | AML.T0054 |
| Contextual hints unlock memorized training data | Medium | LLM06 | AML.T0024 |
| Encoding evasion partial success (NATO, Zalgo) | Medium | LLM01 | AML.T0054 |
| No Bedrock-native guardrails active at invocation layer | Medium | LLM08 | AML.T0051 |
| Model failures are behavioral, not perceptual | Informational | LLM01 | AML.T0054 |

---

## Recommended Mitigations

- Enable **AWS Bedrock Guardrails** with content filtering scoped to the invocation layer
- Implement **input validation and sanitization** at the application layer before API calls
- Apply **output filtering** to detect jailbroken or policy-violating responses before returning to end users
- Deploy **prompt hardening** — explicit system prompt instructions reinforcing boundaries
- Establish **consistent harm category policies** — violence and hate speech protections should be equivalently enforced
- Implement **training data leakage controls** — avoid exposing Cloze-style completion interfaces without output review
- Monitor `bedrock:InvokeModel` CloudTrail events for anomalous patterns (high token counts, repeated probe-like behavior)
- Consider **Bedrock model evaluation** with custom test datasets to establish security baselines

---

## Technical Challenges Resolved

| Challenge | Resolution |
|-----------|-----------|
| Deprecated `--model_type` / `--model_name` flags | Updated to `--target_type` / `--target_name` |
| Missing Rust/Cargo for base2048 dependency | Installed via rustup.rs |
| `pip` not found in venv | Used `python -m pip` |
| NoCredentialsError in Garak | Set `AWS_PROFILE` and `AWS_DEFAULT_REGION` env vars |
| ValidationException — on-demand throughput not supported | Switched to cross-region inference profile ARN format |
| AccessDeniedException on inference profile | Updated IAM policy to include all three US region ARNs |
| SSH session drop mid-encoding suite | Reran remaining probes in targeted follow-up command |
| `replay` probe not found | Correct name in v0.14.0 is `leakreplay` |

---

## Repository Structure

```
ai-security-lab/
├── README.md
├── setup/
│   ├── aws-iam-config.md          # IAM user, policy, and Bedrock access setup
│   ├── garak-installation.md      # Garak v0.14.0 install and Bedrock provider config
│   └── environment-setup.md       # Kali lab environment configuration
├── probes/
│   ├── dan/
│   │   ├── results/               # Garak HTML report
│   │   └── analysis.md
│   ├── promptinject/
│   │   ├── results/
│   │   └── analysis.md
│   ├── encoding/
│   │   ├── results/
│   │   └── analysis.md
│   └── leakreplay/
│       ├── results/
│       └── analysis.md
├── findings/
│   ├── findings-summary.md        # Consolidated findings with risk ratings
│   ├── mitre-atlas-mapping.md
│   └── owasp-llm-mapping.md
├── mitigations/
│   └── bedrock-guardrails-guide.md
├── images/
│   └── garak/                     # Screenshots from each probe suite
└── docs/
    └── findings-report.md         # Full findings report with executive summary and evidence
```

---

## Tools & Frameworks

| Tool / Framework | Purpose |
|-----------------|---------|
| Garak v0.14.0 | LLM vulnerability scanner |
| AWS Bedrock | Managed LLM inference (Llama 3.1 8B Instruct) |
| OWASP LLM Top 10 | Vulnerability classification |
| MITRE ATLAS | AI/ML adversarial threat mapping |
| AWS IAM | Least-privilege access control for lab isolation |

**Planned additions:** PyRIT, Promptfoo

---

## Planned Work

### Current Lab
- [ ] Additional Garak probe suites: `legaladvice`, `xss`, `continuation`
- [ ] Implement AWS Bedrock Guardrails and re-run probe suites to measure effectiveness delta
- [ ] Blog post: *Adversarial Testing of AWS Bedrock LLMs with Garak*

### Future Labs
- [ ] PyRIT evaluation against AWS Bedrock targets
- [ ] Promptfoo comparison testing
- [ ] Additional model targets (Claude, Titan, Mistral via Bedrock)
- [ ] RAG pipeline security testing

---

## Compliance Context

Findings mapped to NIST AI RMF and OWASP LLM Top 10 for structured risk communication. MITRE ATLAS provides adversarial ML technique attribution.

---

## Related Projects

- [purple-team-homelab](https://github.com/gregqlewis/purple-team-homelab) — On-prem MITRE ATT&CK attack/defense lab
- [cloud-security-labs](https://github.com/gregqlewis/cloud-security-labs) — AWS/Azure security engineering with Terraform
- [gregqlewis.com](https://gregqlewis.com) — Blog and portfolio