# Week 5 Report: AutoML Training & Fine-Tuned Model Evaluation

**Name:** Zainab Chaudhry 
**Date:** March 22, 2026
**Capstone Project:** Digital Forensics Email Triage
**My Component:** Email Triage — classifying forensic log entries and emails as suspicious/high-concern vs routine/legitimate

---

## Part A: Teachable Machine Training

### Training Setup
- **Task:** Suspicious vs Normal file browser screenshot classification (Digital Forensics domain)
- **Training images per class:** 20
- **Test images per class:** 5
- **Total training time:** 25 seconds

### Test Results

# 	File Name 		Actual Class 	Predicted Class		Confidence
1. 	test_phish_01.png	Phishing	Legitimate 		79%
2. 	test_phish_02.png	Phishing	Phishing		100%
3. 	test_phish_03.png	Phishing	Phishing 		100%
4. 	test_phish_04.png	Phishing	Phishing 		99%
5.	test_phish_05.png	Phishing	Phishing 		100%
6.	test_legit_01.png	Legitimate	Phishing		99%
7.	test_legit_02.png	Legitimate	Legitimate		100%
8.	test_legit_03.png	Legitimate	Legitimate		100%
9.	test_legit_04.png	Legitimate	Legitimate		100%
10.	test_legit_05.png	Legitimate	Legitimate		100%

|

### Confusion Matrix

| | Predicted: Suspicious | Predicted: Normal |
|---|---|---|
| **Actual: Suspicious** | TP = 4 | FN = 1 |
| **Actual: Normal** | FP = 1| TN = 4 |

### Calculated Metrics
- **Accuracy:** 80%
- **Precision:** 80%
- **Recall:** 80%
- **F1 Score:** 80%

### Interpretation
1. Is your precision higher or lower than your recall? What does that mean for your model's behavior?
- My precision and recall are equal to 80%, meaning my model makes the same number of false alarms as it does missed threads. In practice this means the model is symmetrically uncertain - not biased towards over flagging or underflagging. 

2.  Which errors are more costly for your domain? False positives (false alarms) or false negatives (missed threats)?
- In a phishing detection domain, false negatives are more costly for my domain. A false negative (test #1 — a real phishing email classified as legitimate) means a threat slips through to the user entirely undetected, potentially leading to credential theft, malware, or a data breach. A false positive (test #6 — a legitimate email flagged as phishing) is annoying and creates friction, but the user can recover by reviewing a blocked message. The asymmetry here means I should probably prioritize recall over precision.

3. What confidence threshold would you set? Based on the confidence scores in your test results, at what threshold would you split between "auto-action" and "human review"?
- A reasonable threshold split:
≥ 95% confidence → Auto-action (block or allow)
75–94% confidence → Human review queue
< 75% → Block by default pending review (treat uncertainty as suspicious)

4. What would improve your model? More training data? More diverse data? Different classes?
- More diverse phishing examples. With only 5 phishing training examples, the model likely hasn't seen the full variety of phishing tactic. 
- A third "suspicious / uncertain" class would be valuable. Right now the model is forced into binary decision. 
- More training data overall - 10 test cases are very small. 


---

## Part B: Generic vs Fine-Tuned Model Comparison

### Models Tested
1. **Generic:** distilbert-base-uncased-finetuned-sst-2-english — general sentiment analysis (POSITIVE/NEGATIVE)
2. **Fine-Tuned A:** mrm8488/bert-tiny-finetuned-sms-spam-detection — SMS spam vs ham classification, fine-tuned on suspicious/unwanted message detection
3. **Fine-Tuned B:** cardiffnlp/twitter-roberta-base-sentiment-latest — 3-class sentiment (negative/neutral/positive), fine-tuned on social media text

### Results

Fine-tuned Model A: 
mrm8488/bert-tiny-finetuned-sms-spam-detection 

1. Deleted file recovery initiated by analyst James Chen on NYPD workstation WS-047 at
11:52 PM — file: passwords_backup.xlsx
- Label 1: 0.840
- Label 0: 0.160

2. Registry key modification detected on Maria Torres's laptop in
HKCU\Software\Microsoft\Windows\CurrentVersion\Run — timestamp: 2024-01-15
- Label 1: 0.849
- Label 0: 0.151

3.USB device connected 47 times to air-gapped terminal at Brooklyn Evidence Lab in 6-
hour window — user badge not logged
- Label 1: 0.592
- Label 0: 0.408

4. System audit log shows no events between 14:00 and 18:30 — possible log tampering
or system downtime
- Label 0: 0.648
- Label 1: 0.352

5.  Forensic image hash mismatch detected on evidence drive EV-2024-112 submitted by
Officer Daniels — chain of custody flagged
- Label 1: 0.749
- Label 0: 0.251


### Analysis

**Generic model strengths:** The generic distilbert model performed consistently on clear high-concern records, correctly labeling 4 of 5 inputs as NEGATIVE (suspicious) with very high confidence (0.99+). It required no domain-specific configuration and worked out of the box.

**Generic model weaknesses:** The model misclassified Record 2 (scheduled disk imaging — a routine, legitimate activity) as POSITIVE with only 0.5035 confidence — barely above the 50% threshold. This reflects the core limitation of a generic sentiment model: it interprets "completed successfully — hash verified" as positive sentiment rather than evaluating forensic risk. It has no concept of what is operationally normal vs suspicious in a digital forensics context.

**Fine-tuned model advantage:** Finetuned was not corresponding with n8n data, despite multiple efforts made through many hours and different atttempt methods and modifications. 

**Biggest surprise:** The generic model's near-random confidence on Record 2 (0.5035) was striking — it essentially couldn't decide. This highlights how poorly sentiment polarity maps onto forensic risk classification, even when the generic model otherwise performs well on clearly negative-sentiment threat language.

What should be expected: spam detection model should flag unusual access patterns more reliably; sentiment model should provide a neutral class for routine activities like Record 2 rather than forcing a binary positive/negative

### Recommended Model for My Capstone Component

**Component:** Email Categorization / Forensic Log Triage
**Primary model:** mrm8488/bert-tiny-finetuned-sms-spam-detection (Fine-Tuned A) — spam/suspicious message detection aligns most closely with identifying anomalous forensic activity vs routine operations, and it is lightweight enough for high-volume triage workflows.
**Confidence threshold:** 0.85 — results above this threshold can be auto-flagged for escalation; results between 0.50 and 0.85 should route to human review, as the model's uncertainty in this range is meaningful.
**Priority metric:** **Recall** — in a digital forensics context, missing a genuine threat (false negative) is far more costly than generating a false alarm (false positive). A missed evidence tampering event or unauthorized access incident could compromise a case entirely, whereas a false alarm generates extra review work but no irreversible harm.

---

## Limitations & Next Steps

The primary limitation of this evaluation is sample size — 5 test records is insufficient to draw statistically reliable conclusions about model performance. With only 5 inputs, a single misclassification shifts accuracy by 20 percentage points. A meaningful evaluation would require 50–100 domain-specific records covering a wider range of forensic scenarios, including edge cases such as ambiguous maintenance activities, partially suspicious events, and multi-signal records that combine routine and anomalous indicators.

A second limitation is that none of the three models tested were trained on digital forensics data specifically. Fine-Tuned A is a spam detector and Fine-Tuned B is a social media sentiment model — both are proxies for the actual task. The logical next step would be to fine-tune a model directly on labeled forensic log data using Hugging Face AutoTrain, using the high-concern and low-concern records from the lab's Appendix A as a starting training set (20 labeled examples per class).

Additional models worth testing include `SamLowe/roberta-base-go_emotions` for detecting urgency and distress signals in incident language, and a zero-shot classifier such as `facebook/bart-large-mnli` with forensic-specific candidate labels (e.g., "evidence tampering," "unauthorized access," "routine maintenance") once the JSON encoding issues with em dashes and special characters in the input text are resolved.