---
type: entity
title: "WESAD Dataset"
address: c-000012
created: 2026-05-23
tags:
  - dataset
  - benchmark
  - stress-detection
  - wearables
  - research
status: developing
related:
  - "[[Wellness IoT]]"
  - "[[Wearable Foundation Models]]"
  - "[[iot-wellness-sota-2020-2026]]"
---

# WESAD Dataset

Wearable Stress and Affect Detection dataset. The de facto benchmark for multimodal wearable stress detection research since 2018. Collected by Schmidt, Reiss et al. (ACM ICMI 2018), still the standard evaluation corpus for stress-classification models in 2024-2026.

## Provenance

- **Paper**: Schmidt, Reiss et al., "Introducing WESAD, a Multimodal Dataset for Wearable Stress and Affect Detection" - ACM ICMI 2018.
- **Collection**: Laboratory (controlled stress protocol: Trier Social Stress Test, reading task, meditation).
- **Sensors**: Chest-worn RespiBAN (ECG, EDA, EMG, respiration, temperature, accelerometer) + wrist-worn Empatica E4 (BVP/PPG, EDA, temperature, accelerometer).
- **Participants**: 15.
- **Labels**: Baseline, stress, amusement, meditation (four-class); binary (stress/non-stress); three-class.

## SOTA Results (2024)

| Model | Accuracy | Features | Source |
|---|---|---|---|
| Gradient Boosting + GA-MI + Bayesian optimization | 98.28% binary / 97.02% three-class | EDA + BVP + HRV | IEEE Sensors Journal, 2024 |

> [!key-insight] Lab benchmark, not field metric
> WESAD 97-99% figures are consistently flagged by systematic reviews (Vos et al., ScienceDirect 2023; Mall 2024) as NOT generalizing to free-living conditions. Motion artifacts on wrist-worn EDA (Empatica E4) severely degrade performance outside controlled settings. Treat as an upper-bound lab benchmark only.

## Why Still in Use

WESAD remains the canonical benchmark because:
1. It is publicly available (archived, accessible to researchers without agreements).
2. Multimodal (chest + wrist sensors) covering the main modalities.
3. Enough papers have reported on it to allow direct comparison.
4. No free-living real-world benchmark dataset with equivalent coverage exists yet.

## Generalization Problem

Vos et al. (ScienceDirect 2023) systematic review explicitly: "most stress-related wearable machine learning studies lack generalization." EDA on motion-active wrist devices is severely compromised by motion artifact. High WESAD accuracy does not predict real-world performance.

> [!gap] Missing benchmark
> The highest-leverage research contribution in 2026 is a generalization benchmark: train on WESAD, test on a never-seen real-world cohort. No such benchmark yet exists.

## Successor Efforts

- **NIH All of Us Wearables Dataset** (Nature Medicine 2026): 59,000+ participants, 14-year span, population-scale. But covers step counts and sleep, not controlled stress labels.
- **SWELL**: Another benchmark for stress detection but smaller and less cited.
- Real-world free-living stress benchmark: does not exist as of 2025.

## Related

- [[Wearable Foundation Models]] - foundation models that supersede WESAD-style feature engineering
- [[Wellness IoT]] - application domain
