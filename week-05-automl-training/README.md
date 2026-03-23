# Week 5: AutoML & No-Code Model Training

Trained a custom image classifier with Google Teachable Machine and compared generic vs fine-tuned Hugging Face models for the Email Triage component of our Digital Forensics Email Triage capstone project.

## Custom Model Training
- Built a Phishing vs Legitimate email screenshot classifier with Teachable Machine
- Achieved 80% accuracy on 10 held-out test images
- Precision: 80% | Recall: 80% | F1: 80%

## Fine-Tuned Model Comparison

Compared 3 models (1 generic + 2 fine-tuned) on 5 test inputs:
- Generic: distilbert-sst-2 (sentiment)
- Fine-Tuned A: mrm8488/bert-tiny-finetuned-sms-spam-detection
- Fine-Tuned B: cardiffnlp/twitter-roberta-base-sentiment-latest

## Finding

Recommended mrm8488/bert-tiny-finetuned-sms-spam-detection for Email Triage because its spam vs ham classification most closely mirrors the task of distinguishing suspicious forensic activity from routine operations, and it produced more meaningful confidence scores than the generic sentiment model on domain-specific inputs. Fine-tuned models showed more relevant labels with better handling of edge cases — notably, the generic model nearly failed on Record 2 (a routine disk imaging event) with only 0.5035 confidence, while Fine-Tuned A correctly assigned it a lower suspicion score.

See `report.md` for full analysis.