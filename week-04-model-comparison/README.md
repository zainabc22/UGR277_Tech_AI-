# UGR LAB4 

# Week 4: Model Comparison
Tested 4 AI models on 5 cybersecurity text samples to evaluate their suitability for the alert classification component of our Email Triage & Auto-Response capstone project. 

## Models Tested
- HF Sentiment (distilbert-sst-2)
- HF Zero-Shot (bart-large-mnli)
- HF NER (bert-large-NER)
- Groq Llama 3 8B

## Finding
Recommended Groq Llama 3 8B for the alert classification component because it was the only model to accurately distinguish severity levels across all 5 records while providing a reasoning sentence suitable for inclusion in automated email responses.

See `report.md` for full analysis.