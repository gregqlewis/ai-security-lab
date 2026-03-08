# OWASP LLM Top 10 Mapping

Vulnerability mapping for findings from the AWS Bedrock Llama 3.1 8B Instruct assessment.

The OWASP Top 10 for Large Language Model Applications identifies the most critical security risks in LLM deployments.

Reference: [owasp.org/www-project-top-10-for-large-language-model-applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

---

## Vulnerability Index

| OWASP ID | Name | Findings |
|----------|------|---------|
| LLM01 | Prompt Injection | Findings 1, 2, 3, 5 |
| LLM06 | Sensitive Information Disclosure | Finding 4 |
| LLM08 | Excessive Agency | Finding 6 |

---

## LLM01 — Prompt Injection

**Findings:** 1 (DAN Jailbreak), 2 (Prompt Injection), 3 (Inconsistent Enforcement), 5 (Encoding Evasion)

### Description
Prompt injection occurs when an attacker manipulates an LLM through crafted inputs, causing the model to unintentionally execute attacker instructions. This includes both direct injection (manipulating the model directly) and indirect injection (embedding instructions in content the model processes).

### Lab Observations
This was the highest-impact vulnerability class identified in the assessment, accounting for four of seven findings.

**Direct injection (DAN jailbreak):** 100% success rate using a publicly available persona-adoption technique. The model offered no resistance across any of the five generations tested.

**Instruction hijacking (prompt injection):** 35.78–70.96% success rate across three sub-probes. Hate speech content policies were significantly weaker than violence-related policies, indicating inconsistent safety enforcement.

**Encoding evasion:** Largely unsuccessful overall, with NATO phonetic (14.77%) and Zalgo text (9.45%) as notable exceptions. The low encoding success rate relative to plaintext injection confirms the model's failures are behavioral rather than perceptual.

### Risk in Production
Any deployment where the model processes user-supplied content — document summarization, Q&A pipelines, chatbots, data extraction — is exposed to this risk. An attacker with access to the input layer can reliably manipulate model behavior.

### Mitigations
- AWS Bedrock Guardrails for jailbreak and injection pattern detection
- Input validation and sanitization at the application layer
- Strict architectural separation between system instructions and user content
- Consistent harm category policy enforcement across content types

---

## LLM06 — Sensitive Information Disclosure

**Finding:** 4 (Training Data Leakage)

### Description
LLMs can inadvertently reveal sensitive information, proprietary data, or memorized training content through their outputs. This includes personal data, confidential business information, or copyrighted material present in training datasets.

### Lab Observations
Complete probes (direct verbatim completion requests) largely passed. Cloze probes (fill-in-the-blank style) succeeded at 3–11%, with Guardian news content at the highest rate (11.11%). The pattern demonstrates that partial contextual framing is sufficient to unlock memorized content even when direct requests fail.

### Risk in Production
For base models this risk is limited to publicly available training sources. For fine-tuned models trained on sensitive internal documents, the risk increases significantly — an attacker with partial knowledge of document contents could use Cloze-style prompts to confirm or extract memorized sensitive content including PII, proprietary data, or internal procedures.

### Mitigations
- Differential privacy techniques during fine-tuning on sensitive data
- Training dataset audits prior to fine-tuning
- Output scanning for sensitive content patterns
- Restrict Cloze-style completion interfaces in production

---

## LLM08 — Excessive Agency

**Finding:** 6 (No Guardrails at Invocation Layer)

### Description
Excessive agency refers to an LLM being granted more capability, access, or autonomy than necessary for its intended function, without appropriate controls to limit the impact of unexpected or manipulated outputs.

### Lab Observations
No AWS Bedrock Guardrails were configured at the model invocation layer during testing. All results reflect raw model behavior with no application-level content filtering, output validation, or anomaly detection. The model had unrestricted ability to produce any output in response to probe inputs.

### Risk in Production
Deploying a model without guardrails grants it effective agency to produce any output in response to any input. Combined with the jailbreak and injection vulnerabilities identified in this assessment, an attacker has a reliable path to fully unconstrained model behavior.

### Mitigations
- Enable AWS Bedrock Guardrails with content filtering policies
- Implement output validation before returning responses to end users
- Apply least-privilege principles to model capabilities — restrict to only the output types required for the application's intended function
- Monitor and log all model invocations via CloudTrail