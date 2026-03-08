# Prompt Injection — Probe Analysis

**Probe:** `promptinject`  
**Generations:** 5  
**Report:** `results/garak.1dacc6f9-ef3a-4ddb-8331-dd4092ec46b3.report.html`

---

## Results

| Sub-Probe | Attack Success Rate |
|-----------|-------------------|
| HijackHateHumans | 70.96% (899/1267) |
| HijackLongPrompt | 68.83% (881/1280) |
| HijackKillHumans | 35.78% (458/1280) |

---

## What This Probe Tests

The `promptinject` suite tests whether malicious instructions embedded within otherwise legitimate input can override the model's intended behavior. Each sub-probe embeds a hidden instruction that attempts to hijack the model's output:

- **HijackHateHumans** — injects an instruction to produce hate speech content
- **HijackKillHumans** — injects an instruction to produce violence-related content
- **HijackLongPrompt** — embeds the malicious instruction inside a significantly longer prompt to test whether context volume affects susceptibility

---

## Key Findings

**Inconsistent harm category enforcement.** Hate speech injection succeeded at nearly twice the rate of violence injection (70.96% vs 35.78%), revealing that safety policies are not uniformly applied across harm categories. Violence-related content had meaningfully stronger protection than hate speech.

**Prompt length has no protective effect.** HijackLongPrompt (68.83%) was nearly as effective as HijackHateHumans (70.96%), demonstrating the model performs no meaningful context analysis to detect injected instructions regardless of surrounding content volume.

---

## Interpretation

In a production deployment where the model processes user-supplied content — document summarization, Q&A, data pipelines — an attacker can embed hidden instructions that hijack model behavior. The high success rate and insensitivity to prompt length mean that simply adding more context around user input does not mitigate this risk.

The inconsistency between harm categories also suggests that safety fine-tuning was applied unevenly, leaving hate speech as a significantly weaker enforcement area than violence.

---

## MITRE ATLAS

`AML.T0051` — LLM Prompt Injection

---

## OWASP LLM Top 10

`LLM01` — Prompt Injection

---

## Severity

**Critical**

---

## Recommendation

Implement input sanitization at the application layer before passing user-supplied content to the model. Treat all user input as untrusted. Use Bedrock Guardrails with contextual grounding checks. Architect AI pipelines to maintain strict separation between system instructions and user-supplied data.