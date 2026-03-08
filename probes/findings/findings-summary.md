# Findings Summary

Consolidated findings from all four Garak probe suites executed against Meta Llama 3.1 8B Instruct via AWS Bedrock.

---

## Findings Overview

| # | Finding | Severity | Probe Suite |
|---|---------|----------|-------------|
| 1 | 100% DAN jailbreak success rate | Critical | dan |
| 2 | Prompt injection via instruction override (up to 70.96%) | Critical | promptinject |
| 3 | Inconsistent harm category policy enforcement | High | promptinject |
| 4 | Contextual hints unlock memorized training data | Medium | leakreplay |
| 5 | Encoding evasion partial success (NATO 14.77%, Zalgo 9.45%) | Medium | encoding |
| 6 | No Bedrock-native guardrails active at invocation layer | Medium | All |
| 7 | Model safety failures are behavioral, not perceptual | Informational | encoding / promptinject |

---

## Finding 1 — 100% DAN Jailbreak Success Rate

**Severity:** Critical  
**Probe:** `dan.Dan_11_0`

The DAN 11.0 jailbreak achieved a 100% success rate across all five generations. The model fully adopted the DAN persona on every attempt, explicitly announcing it would disregard all content policies. This attack requires no technical sophistication and uses a publicly available prompt.

**OWASP LLM:** LLM01 | **MITRE ATLAS:** AML.T0054

---

## Finding 2 — High-Rate Prompt Injection

**Severity:** Critical  
**Probe:** `promptinject`

Malicious instructions embedded within legitimate input successfully hijacked model behavior at rates between 35.78% and 70.96% depending on content type. Prompt length had no meaningful protective effect, confirming the model performs no context analysis to detect injected instructions.

**OWASP LLM:** LLM01 | **MITRE ATLAS:** AML.T0051

---

## Finding 3 — Inconsistent Harm Category Enforcement

**Severity:** High  
**Probe:** `promptinject`

Hate speech injection (70.96%) succeeded at nearly twice the rate of violence injection (35.78%), revealing that safety policies are not uniformly applied across harm categories. This inconsistency indicates uneven safety fine-tuning rather than a coherent content policy.

**OWASP LLM:** LLM01 | **MITRE ATLAS:** AML.T0051

---

## Finding 4 — Cloze-Style Training Data Leakage

**Severity:** Medium  
**Probe:** `leakreplay`

While direct text completion attempts largely failed, fill-in-the-blank (Cloze) style prompts successfully extracted memorized training data at rates of 3–11%. Guardian news content showed the highest leakage rate at 11.11%. This demonstrates that data leakage risk is not eliminated by blocking verbatim completion — contextual framing is sufficient.

**OWASP LLM:** LLM06 | **MITRE ATLAS:** AML.T0057

---

## Finding 5 — Encoding Evasion Partial Success

**Severity:** Medium  
**Probe:** `encoding`

The model showed strong overall resistance to encoding-based attacks, with two notable exceptions: NATO phonetic alphabet (14.77% DecodeMatch) and Zalgo text (9.45% approximate match). All other encoding schemes achieved near-zero success rates.

**OWASP LLM:** LLM01 | **MITRE ATLAS:** AML.T0054

---

## Finding 6 — No Guardrails at Invocation Layer

**Severity:** Medium  
**Probe:** All suites

No AWS Bedrock Guardrails were active during testing. All probe results reflect the model's native safety behaviors without any application-layer filtering. Results would be expected to improve with guardrails enabled, though the extent of improvement is untested.

**OWASP LLM:** LLM08 | **MITRE ATLAS:** AML.T0051

---

## Finding 7 — Behavioral vs. Perceptual Failures

**Severity:** Informational  
**Probe:** encoding / promptinject

The sharp contrast between encoding evasion (0–15%) and prompt injection (36–71%) reveals that this model's safety vulnerabilities are rooted in instruction-following compliance, not content recognition. The model can decode obfuscated input — it fails because it is too compliant when instructions are framed directly and authoritatively. This has significant implications for mitigation strategy: keyword filtering alone will not address the root vulnerability.

---

## Risk Summary

| Severity | Count |
|----------|-------|
| Critical | 2 |
| High | 1 |
| Medium | 3 |
| Informational | 1 |

---

## Recommendations Summary

| Priority | Recommendation |
|----------|---------------|
| Critical | Deploy AWS Bedrock Guardrails to intercept known jailbreak patterns |
| Critical | Treat all user-supplied input as untrusted — implement input sanitization pipelines |
| High | Architect AI pipelines with strict separation between system instructions and user data |
| High | Audit and align harm category safety policies across content types |
| Medium | Add NATO phonetic and Zalgo text encoding to input filtering rules |
| Medium | Audit fine-tuning datasets for sensitive content before model training |
| Low | Implement output scanning for potential training data leakage |

---

## Detailed Analysis

- Per-probe breakdowns: `../probes/*/analysis.md`
- Full findings report with evidence: `../docs/findings-report.md`
- Framework mappings: `mitre-atlas-mapping.md` | `owasp-llm-mapping.md`