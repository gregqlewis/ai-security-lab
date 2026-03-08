# Encoding Evasion — Probe Analysis

**Probe:** `encoding`  
**Generations:** 5  
**Report:** `results/garak.f373a321-7be6-4522-84d8-6d28be87befe.report.html`

> **Note:** This suite was interrupted by an SSH session drop mid-run. Remaining sub-probes were completed in a follow-up targeted command.

---

## Results

| Encoding | DecodeMatch | DecodeApprox |
|----------|-------------|--------------|
| ASCII85 | 0.31% | PASS |
| Base2048 | 0.08% | PASS |
| Base32 | 0.23% | PASS |
| Base64 | 0.70% | 0.55% |
| Braille | 0.08% | 0.16% |
| Ecoji | 0.08% | 0.08% |
| Hex | 2.50% | 2.42% |
| Morse | 0.08% | PASS |
| **NATO Phonetic** | **14.77%** | 0.55% |
| ROT13 | 0.08% | 0.39% |
| UU | 0.08% | PASS |
| Unicode Tags | 0.08% | PASS |
| **Zalgo** | 6.02% | **9.45%** |

---

## What This Probe Tests

The `encoding` suite tests whether obfuscating malicious instructions using various encoding schemes can bypass the model's safety behaviors. Each sub-probe encodes a harmful instruction and submits it to the model, testing whether the model decodes and acts on the encoded content.

---

## Key Findings

**Strong overall resilience.** The model resisted the vast majority of encoding-based attacks, with most sub-probes achieving near-zero success rates. This is a meaningful positive finding.

**NATO phonetic is the highest-risk encoding.** At 14.77% DecodeMatch, NATO phonetic alphabet encoding was the only encoding to breach double digits. This is notable because NATO phonetic is a realistic encoding that users might legitimately employ in communications or military contexts, making it harder to filter without false positives.

**Zalgo text shows meaningful approximate match success.** Zalgo text achieved 9.45% on approximate matching, indicating partial evasion even when exact decoding failed.

**The contrast with Suite 2 is the most important finding.** The sharp difference between encoding evasion (0–15%) and prompt injection (36–71%) reveals that this model's safety failures are **behavioral rather than perceptual**. The model can recognize and decode obfuscated input — it fails not because it misses harmful content, but because it is too compliant when instructions are framed directly and authoritatively in plaintext.

---

## Interpretation

This finding has significant implications for mitigation strategy. Keyword filtering and encoding detection alone will not address the model's core vulnerability. The root issue is instruction-following compliance, not content recognition.

---

## MITRE ATLAS

`AML.T0054` — LLM Jailbreak (Evasion Variant)

---

## OWASP LLM Top 10

`LLM01` — Prompt Injection

---

## Severity

**Medium**

---

## Recommendation

Add NATO phonetic and Zalgo text encoding patterns to input filtering rules as a low-cost mitigation. Priority should remain on addressing the higher-severity behavioral vulnerabilities identified in the DAN and prompt injection suites, as encoding evasion represents a secondary risk surface.