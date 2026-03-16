# Model Comparison Report — Week 4
**Name:** Zainab Chaudhry 
**Date:** 3/15/26
**Capstone Project:** Email Triage and Auto-Responder
**My Component:** Alert classification and severity routing for incoming security emails. 

## Test Setup
**Input dataset:** 5 cybersecurity text samples covering:
- 2 clearly concerning/high-severity records (unauthorized login from Moscow, multiple failed SSH attempts from Beijing)
- 1 ambiguous/edge case record (phishing email targeting finance department)
- 2 routine/benign records (scheduled firewall maintenance, normal system resource utilization)

**Models tested:**
1. distilbert-base-uncased-finetuned-sst-2-english (sentiment)
2. facebook/bart-large-mnli (zero-shot classification)
3. dslim/bert-large-NER (named entity recognition)
4. Groq Llama 3 8B (LLM classification)

**Evaluation criteria:** label accuracy, confidence score, speed, ease of
integration in n8n

## Results Summary
| Record | Sentiment | Zero-Shot | NER Entities | Groq |
|--------|-----------|-----------|-------------|------|
| 1 - Unauthorized login from Moscow | NEGATIVE (0.9961) | possible anomaly (0.90) | Moscow (LOC) | CRITICAL |
| 2 - Routine firewall update | NEGATIVE (0.9986) | routine activity (1.0) | none | INFORMATIONAL |
| 3 - Phishing email via Amazon domain | NEGATIVE (0.9959) | possible anomaly (0.80) | Amazon (ORG) | HIGH |
| 4 - 47 failed SSH attempts from Beijing | NEGATIVE (0.9994) | possible anomaly (0.80) | SS (MISC) | HIGH |
| 5 - Normal system resource utilization| NEGATIVE (0.9880) | routine activity | none | INFORMATIONAL |

## Analysis
**Where models agreed:** 
Records 2 and 5 (the two benign records) produced consistent results across models — Zero-Shot labeled both "routine activity" with near-perfect confidence, Groq classified both as INFORMATIONAL, and NER returned no entities for either, which is correct since they contain no named people, places, or organizations.

**Where models disagreed:** 
Record 1 (unauthorized login from Moscow) showed the sharpest disagreement. Groq escalated it to CRITICAL, while Zero-Shot only reached "possible anomaly" — a meaningful gap for a real security workflow where missing a CRITICAL alert has serious consequences. Record 4 (SSH brute force) also split: Groq said HIGH while Zero-Shot again stopped at "possible anomaly," suggesting Zero-Shot is systematically conservative with its top label.

**Most accurate model overall:** 
Groq Llama 3 8B. It produced accurate, contextually appropriate severity levels for all 5 records and included a one-sentence rationale for each, making its outputs directly actionable. It was the only model that correctly distinguished between CRITICAL and HIGH across the three concerning records.

**Fastest/most practical:** 
Zero-shot classification (bart-large-mnli) is the most practical for scale — it requires no prompt engineering, returns a structured label and confidence score, and integrates cleanly into n8n with a single HTTP Request node. Groq is slightly more powerful but introduces LLM latency and prompt dependency.

## Recommended Models for My Capstone Component
**Component:** 
Alert classification and severity routing for incoming security emails

**Primary model:** 
Groq Llama 3 8B — its ability to return a severity level with a reasoning sentence makes it ideal for triage, since the auto-responder needs not just a label but a justification to include in the reply.

**Secondary model:** 
facebook/bart-large-mnli (zero-shot) — serves as a fast pre-filter to separate routine emails from anomalies before passing candidates to Groq, reducing unnecessary LLM calls.

**Rejected models and why:**
- distilbert-sst-2 (Sentiment): Classified all 5 records as NEGATIVE regardless of actual threat level, making it useless for security triage. Sentiment analysis detects emotional tone, not operational severity.

- dslim/bert-large-NER: Useful as a supplementary enrichment tool to extract entities like IPs, names, and organizations from email bodies, but cannot perform classification on its own and produced errors on technical terms like "SSH."

## Failure Cases and Limitations
The most significant failure was NER misidentifying "SSH" as the entity "SS" (tagged as MISC) in Record 4. This occurred because the NER model was trained primarily on news and general text corpora, not cybersecurity terminology. In a production email triage system, this kind of entity extraction error could cause alert enrichment to miss or misattribute key technical indicators. It suggests that a general-purpose NER model would need fine-tuning on security-domain text before being deployed in a real SOC pipeline.

Sentiment's complete failure across all 5 records is also worth noting: in production, blindly trusting a high-confidence NEGATIVE score (0.99) would give false assurance that a model "strongly detected" something meaningful, when in reality it was measuring linguistic negativity, not threat severity.


## Next Steps
With more time, I would test the following:

- A larger record set (20–30 emails) including more edge cases such as internal policy violations and false positives from legitimate but unusual activity

- Custom candidate labels for zero-shot tuned specifically to email triage (e.g., "requires immediate response," "needs human review," "safe to auto-respond")

- A security-domain fine-tuned NER model to correctly handle technical entities like IP addresses, CVE numbers, and protocol names

- Prompt variations for Groq to evaluate whether more specific instructions improve consistency across ambiguous records