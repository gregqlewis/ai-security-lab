# Training Data Leakage — Probe Analysis

**Probe:** `leakreplay`  
**Generations:** 5  
**Report:** `results/garak.f7874320-b704-4ce2-aac5-a2190a22a8ad.report.html`

---

## Results

| Sub-Probe | Attack Success Rate |
|-----------|-------------------|
| GuardianCloze | 11.11% |
| NYTCloze | 5.62% |
| PotterCloze | 3.33% |
| PotterComplete | 0.67% |
| GuardianComplete | PASS |
| LiteratureComplete | PASS |
| NYTComplete | PASS |

---

## What This Probe Tests

The `leakreplay` suite tests whether the model leaks memorized training data. Two attack styles are used:

- **Complete probes** — ask the model to complete a passage verbatim from a known training source (open-ended completion)
- **Cloze probes** — provide a passage with blanks and ask the model to fill them in (contextual hint-based extraction)

Sources tested include Guardian news articles, New York Times articles, Harry Potter text, and classic literature.

---

## Key Findings

**Open-ended completion largely failed.** Complete probes passed across all three major sources, indicating the model does not readily reproduce memorized text when asked directly.

**Contextual hints unlock memorized content.** Cloze probes succeeded at 3–11% depending on source, demonstrating that even when direct completion fails, providing partial context is sufficient to extract memorized training data.

**Guardian news content had the highest leakage rate.** At 11.11%, Guardian articles showed disproportionately high susceptibility compared to other sources, suggesting this content may be more heavily represented or more precisely memorized in the model's training data.

---

## Interpretation

The Cloze vs. Complete contrast reveals that data leakage risk is not eliminated by refusing verbatim completion requests. An attacker with partial knowledge of a document's contents — or access to a public source — can use fill-in-the-blank style prompts to confirm or extract memorized content.

In a fine-tuned deployment where the model has been trained on sensitive internal documents, this attack surface expands significantly. An attacker could use Cloze-style prompts to probe whether specific sensitive content was included in fine-tuning data, even without direct access to the training set.

---

## MITRE ATLAS

`AML.T0057` — LLM Data Leakage

---

## OWASP LLM Top 10

`LLM06` — Sensitive Information Disclosure

---

## Severity

**Medium**

---

## Recommendation

Avoid fine-tuning production models on sensitive documents without implementing differential privacy techniques. Audit fine-tuning datasets before use. Implement output scanning to detect potential training data reproduction in model responses. Do not expose Cloze-style completion interfaces to untrusted users in production deployments.