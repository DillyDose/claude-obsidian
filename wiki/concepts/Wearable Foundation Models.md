---
type: concept
title: "Wearable Foundation Models"
address: c-000005
created: 2026-05-23
tags:
  - ai
  - foundation-models
  - wearables
  - health-tech
  - llm
status: developing
related:
  - "[[Wellness IoT]]"
  - "[[Apple Health Ecosystem]]"
  - "[[Google PH-LLM]]"
  - "[[WESAD Dataset]]"
  - "[[iot-wellness-sota-2020-2026]]"
---

# Wearable Foundation Models

Self-supervised models pretrained on billions of hours of wearable sensor data, then fine-tuned for downstream health prediction tasks. As of 2025, this paradigm has displaced hand-crafted feature engineering as the state of the art in wellness AI.

> [!key-insight] Paradigm shift
> Apple WBM + Google PH-LLM signal a move from WESAD-style feature engineering to billion-hour self-supervised pretraining. Foundation model owners with proprietary cohort data (Apple AHMS, Google/Fitbit, NIH All of Us) have a structural data moat.

## Key Systems (2024-2026)

### Apple Wearable Behavior Model (WBM)

- **Paper**: Erturk et al., "Beyond Sensor Data: Foundation Models of Behavioral Data from Wearables Improve Health Predictions" (arXiv:2507.00191, July 2025) - PREPRINT, not peer-reviewed.
- **Architecture**: Mamba-2 backbone trained on 27 HealthKit behavioral metrics.
- **Scale**: 2.5 billion hours of data, 162,000 Apple Heart and Movement Study participants, 15 billion hourly measurements.
- **Evaluation**: 57 downstream health tasks.
- **Key results**: Outperforms PPG foundation model on 18/47 static tasks and nearly all transient tasks; hybrid WBM+PPG reaches ~92% accuracy on pregnancy detection.
- **Caveat**: All numbers are Apple-reported; no independent replication as of mid-2025.

### Apple PPG/ECG Foundation Model

- **Paper**: Abbaspourazad et al., "Large-scale Training of Foundation Models for Wearable Biosignals" (ICLR 2024, arXiv:2312.05409).
- **Scale**: ~141K AHMS participants, 3 years.
- **Focus**: Self-supervised pretraining on raw biosignals (PPG, ECG).

### Google PH-LLM (Personal Health LLM)

- **Paper**: Cosentino, Belyaeva et al. (Nature Medicine 31:3394-3403, 2025) - peer-reviewed.
- **Approach**: Fine-tuned Gemini on Fitbit sensor data + text context.
- **Results**: 79% on sleep-medicine exam (vs 76% human experts), 88% on fitness exam (vs 71% human experts); 857 case studies; superior AUROC on 12/16 sleep-quality traits.
- **Product link**: Google announced "Transforming Wearable Data into Personal Health Insights Using Large Language Model Agents" alongside PH-LLM.
- See [[Google PH-LLM]] for full entity page.

### NIH All of Us Wearables Dataset

- **Paper**: Nature Medicine 2026 (Master et al. 2022, Brittain et al. 2024 for sub-studies).
- **Scale**: 59,000+ participants, 14-year span, 39M step + 31M sleep observations.
- **Coverage**: 46% with linked EHR and genomics data.
- **Significance**: Largest open population-scale wearable cohort; enables independent replication.

## Why Foundation Models Win

1. **Scale**: Billion-hour pretraining captures seasonal, circadian, and longitudinal patterns impossible to hand-label.
2. **Transfer**: One pretrained backbone fine-tunes to 57+ downstream tasks (Apple WBM demonstrated).
3. **Behavioral biomarkers**: Movement, sleep-pattern, and activity sequences encode latent health state better than raw biosignals alone (WBM insight).
4. **Conversational coaching**: PH-LLM-style models can explain health data in natural language, enabling personalized coaching agents.

## Data Moat

The defensible value in wellness AI is moving from sensor hardware to training cohorts:
- Apple AHMS: 162K participants (proprietary)
- Google/Fitbit: Embedded in PH-LLM (proprietary)
- NIH All of Us: 59K Fitbit participants (open)

> [!gap] Open-source gap
> No open-source wearable foundation model yet matches Apple WBM on the 57-task benchmark. A credible open model would commoditize this moat.

## Limitations

- Demographic bias: Apple AHMS underrepresents women, older adults, and racial/ethnic minorities (explicitly stated in WBM paper).
- Apple WBM is a preprint; peer-reviewed validation pending.
- PH-LLM has 857 case studies but clinical deployment is unproven.
- All of Us Fitbit subset: 73% female, 84% white.

## Related

- [[Apple Health Ecosystem]] - Apple's AHMS, WBM, and sensor stack
- [[Google PH-LLM]] - Google's fine-tuned Gemini for health
- [[WESAD Dataset]] - previous-generation benchmark these models supersede
- [[Wellness IoT]] - broader category context
