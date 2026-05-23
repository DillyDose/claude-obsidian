---
type: entity
title: "Google PH-LLM"
address: c-000011
created: 2026-05-23
tags:
  - google
  - llm
  - health-tech
  - foundation-model
  - fitbit
status: developing
related:
  - "[[Wearable Foundation Models]]"
  - "[[Wellness IoT]]"
  - "[[iot-wellness-sota-2020-2026]]"
---

# Google PH-LLM

Google's Personal Health Large Language Model, a fine-tuned Gemini model trained on Fitbit sensor data and health text, capable of providing personalized coaching on sleep and fitness. As of 2025, PH-LLM is the only peer-reviewed wearable foundation model demonstrating superhuman performance on standardized medical examinations.

## Paper

**Cosentino, Belyaeva et al.** "A personal health large language model for sleep and fitness coaching" - Nature Medicine 31:3394-3403, 2025. Peer-reviewed.

## Architecture

- Base model: Gemini (specific version not disclosed)
- Fine-tuning data: Fitbit sensor streams + health text
- Training approach: Multimodal (time-series sensor data + natural language)

## Key Results

| Task | PH-LLM | Human Expert |
|---|---|---|
| Sleep-medicine examination | 79% | 76% |
| Fitness examination | 88% | 71% |
| Sleep-quality AUROC (vs traits) | Superior on 12/16 sleep-quality traits | - |

- Evaluated on 857 case studies.
- Outperforms human experts on both standardized examinations.

## Companion Work

Google simultaneously announced "Transforming Wearable Data into Personal Health Insights Using Large Language Model Agents" - the product-facing application of PH-LLM enabling conversational health coaching from Fitbit data streams.

## Data Source

Underlying sensor stream: Google/Fitbit device network, one of the three largest proprietary wearable cohorts globally (alongside Apple AHMS and NIH All of Us).

## Samsung Xealth Comparison

Google/Fitbit competes with Samsung's July 2025 acquisition of Xealth, which bridges Samsung Health wearable data with 500+ hospital systems. PH-LLM takes the AI-model approach; Xealth takes the clinical-integration approach.

## Limitations

- 857 case studies is a moderate evaluation set; real-world clinical deployment unproven.
- Training data has demographic skew (Fitbit user base).
- Gemini base model version not disclosed; replicability limited.

## Related

- [[Wearable Foundation Models]] - Apple WBM, PH-LLM, and the paradigm they represent
- [[Wellness IoT]] - broader market context
- [[Apple Health Ecosystem]] - primary competitor in wearable foundation models
