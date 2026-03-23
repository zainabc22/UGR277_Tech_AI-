# Confusion Matrix — Teachable Machine Classifier

**Model:** Phishing vs Legitimate Email Screenshot Classifier
**Test Set:** 10 images (5 per class, held out from training)
**Date:** March 22, 2026

---

## Confusion Matrix

|  | Predicted: Phishing | Predicted: Legitimate |
|---|---|---|
| **Actual: Phishing** | TP = 4 | FN = 1 |
| **Actual: Legitimate** | FP = 1 | TN = 4 |

---

## Calculated Metrics

| Metric | Formula | Result |
|--------|---------|--------|
| **Accuracy** | (TP + TN) / (TP + TN + FP + FN) = 8 / 10 | **80%** |
| **Precision** | TP / (TP + FP) = 4 / 5 | **80%** |
| **Recall** | TP / (TP + FN) = 4 / 5 | **80%** |
| **F1 Score** | 2 × (Precision × Recall) / (Precision + Recall) = 2 × (0.8 × 0.8) / (0.8 + 0.8) | **80%** |

---

## Error Analysis

| Test # | Actual Class | Predicted Class | Confidence | Error Type |
|--------|--------------|-----------------|------------|------------|
| 1 | Phishing | Legitimate | 79% | False Negative |
| 6 | Legitimate | Phishing | 99% | False Positive |

**False Negative (test_phish_01.png):** A real phishing email was classified as legitimate. This is the more dangerous error type for this domain — a threat reaching the user undetected.

**False Positive (test_legit_01.png):** A legitimate email was flagged as phishing at 99% confidence. This creates friction but is recoverable.

---

## Interpretation

Precision and recall are equal at 80%, indicating the model is symmetrically uncertain with no directional bias toward over-flagging or under-flagging. For a phishing detection use case, recall should be prioritized over precision since missed threats carry higher risk than false alarms. With only 10 test images, these metrics are directional estimates — a larger test set of 50–100 images would be needed for statistically reliable conclusions.