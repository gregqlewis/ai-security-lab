# MITRE ATLAS Mapping

Adversarial ML technique mapping for findings from the AWS Bedrock Llama 3.1 8B Instruct assessment.

MITRE ATLAS (Adversarial Threat Landscape for AI Systems) provides a knowledge base of adversarial tactics and techniques targeting AI/ML systems, modeled after MITRE ATT&CK.

Reference: [atlas.mitre.org](https://atlas.mitre.org)

---

## Technique Index

| Technique ID | Name | Findings |
|-------------|------|---------|
| AML.T0051 | LLM Prompt Injection | Findings 2, 3, 6 |
| AML.T0054 | LLM Jailbreak | Findings 1, 5 |
| AML.T0057 | LLM Data Leakage | Finding 4 |

---

## AML.T0051 — LLM Prompt Injection

**Tactic:** ML Attack Staging  
**Findings:** 2 (Prompt Injection), 3 (Inconsistent Enforcement), 6 (No Guardrails)

### Description
Adversaries craft inputs that manipulate the model into ignoring its original instructions and following attacker-supplied instructions instead. This exploits the model's inability to distinguish between trusted system instructions and untrusted user input.

### Lab Observations
The `promptinject` suite demonstrated injection success rates of 35.78–70.96% depending on content type. Embedding malicious instructions inside longer prompts had no meaningful protective effect, confirming the model does not perform context analysis to detect injected content.

The absence of Bedrock Guardrails at the invocation layer meant no application-level filtering was present to intercept injected instructions before they reached the model.

### Mitigations
- Strict separation of system prompts and user-supplied content in pipeline architecture
- Input sanitization before passing user data to the model
- AWS Bedrock Guardrails with contextual grounding checks

---

## AML.T0054 — LLM Jailbreak

**Tactic:** ML Attack Staging  
**Findings:** 1 (DAN Jailbreak), 5 (Encoding Evasion)

### Description
Adversaries use specially crafted prompts to bypass a model's safety fine-tuning and elicit policy-violating outputs. Jailbreaks typically work by instructing the model to adopt an alternate persona, roleplay a scenario, or reframe the request in a way that circumvents safety training.

### Lab Observations
The DAN 11.0 jailbreak achieved a 100% success rate — the model adopted the DAN persona on every attempt and explicitly announced it would disregard content policies. This is a publicly available, low-sophistication technique requiring no technical expertise.

Encoding-based evasion (an obfuscation variant of jailbreaking) was largely unsuccessful, with the exception of NATO phonetic (14.77%) and Zalgo text (9.45%). The contrast between jailbreak success and encoding failure reinforces that the model's vulnerabilities are behavioral rather than perceptual.

### Mitigations
- AWS Bedrock Guardrails with known jailbreak pattern detection
- CloudTrail monitoring for anomalous `bedrock:InvokeModel` patterns
- Input filtering for NATO phonetic and Zalgo encoding patterns

---

## AML.T0057 — LLM Data Leakage

**Tactic:** Exfiltration  
**Finding:** 4 (Training Data Leakage)

### Description
Adversaries craft inputs that cause the model to reproduce memorized training data, potentially exposing sensitive information that was present in the training dataset.

### Lab Observations
Direct text completion attempts (Complete probes) largely failed. However, Cloze-style prompts — which provide partial context and ask the model to fill in blanks — successfully extracted memorized content at rates of 3–11%. Guardian news articles showed the highest leakage rate at 11.11%, suggesting disproportionate memorization of that content.

This demonstrates that blocking verbatim completion requests is insufficient — contextual framing provides an alternative extraction path.

### Mitigations
- Differential privacy techniques when fine-tuning on sensitive documents
- Pre-training dataset audits to identify and remove sensitive content
- Output scanning to detect potential training data reproduction
- Avoid exposing Cloze-style interfaces to untrusted users