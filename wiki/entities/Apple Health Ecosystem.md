---
type: entity
title: "Apple Health Ecosystem"
address: c-000009
created: 2026-05-23
tags:
  - apple
  - wearables
  - health-tech
  - foundation-model
status: developing
related:
  - "[[Wellness IoT]]"
  - "[[Wearable Foundation Models]]"
  - "[[iot-wellness-sota-2020-2026]]"
---

# Apple Health Ecosystem

Apple's integrated hardware, software, and research ecosystem for health monitoring, spanning Apple Watch, HealthKit, the Apple Heart and Movement Study (AHMS), and the Wearable Behavior Model (WBM) foundation model.

## Products and Features (2024-2026)

**Apple Watch Series 9/10/Ultra 2** (2024):
- FDA 510(k) cleared sleep-apnea notifications ("Breathing Disturbances" metric, Sept 2024)
- ECG (leads I and II), irregular-rhythm notification, blood oxygen, wrist temperature
- High/low heart rate alerts, fall detection, retrospective ovulation tracking

**Apple Watch Series 11** (Sept 2025):
- Added hypertension notifications
- Comprehensive structured Sleep Score
- Full feature set includes ECG, AF notifications, blood oxygen, retrospective ovulation, wrist temperature, high/low HR

**AirPods Pro 2** (Sept 2024): FDA-cleared hearing-aid functionality.

**Apple Vision Pro**: Mindfulness and biofeedback experiences.

## Diagnostic Accuracy

- **AF detection via ECG**: Pooled sensitivity 94.8% (CI 91.7-96.8), specificity 95.0%, AUROC 0.96 (JACC: Advances 2024 meta-analysis, 11 studies, n=4,241).
- **Note**: The Basel Wearable Study found passive irregular-rhythm notification sensitivity only 21.4% (vs 94.8% for on-device ECG modality) - very different modalities.
- **Sleep apnea**: FDA-cleared algorithm trained on clinical-grade apnea datasets; validated for moderate-severe OSA in adults 18+.
- **Four-stage sleep staging**: Apple Watch Series 8 sensitivity 50.5-86.1% (Brigham & Women's 2024, n=35) - not the most accurate consumer device for sleep staging.

## Apple Heart and Movement Study (AHMS)

- Estimated >250,000 participants enrolled by 2024.
- Source data for both the PPG/ECG Foundation Model (Abbaspourazad et al., ICLR 2024) and the Wearable Behavior Model (Erturk et al., arXiv:2507.00191, 2025).
- **Known demographic skew**: Women, older adults, and racial/ethnic minorities remain underrepresented (explicitly stated in WBM paper).

## Wearable Behavior Model (WBM)

Foundation model trained on 27 HealthKit behavioral metrics. See [[Wearable Foundation Models]] for full treatment.

Key figures (Apple-reported; preprint not peer-reviewed):
- 2.5B hours, 162K participants, 15B hourly measurements
- 57 downstream health tasks
- ~92% pregnancy detection accuracy (hybrid WBM+PPG)

## Market Position

- ~28% of 2025 global wearables market share (IDC).
- HealthKit is the largest consumer health data platform by participants.
- FDA 510(k) clearances (sleep apnea 2024, AF ECG ongoing) position Apple Watch as a de facto clinical device.

## Research Collaborations

AHMS partners include Stanford (Apple Heart Study), Brigham & Women's Hospital, and UCSF. Apple publishes AHMS-derived research in peer-reviewed journals (Nature Medicine, ICLR, JACC: Advances).

## Related

- [[Wearable Foundation Models]] - WBM and PPG/ECG FM trained on AHMS
- [[Wellness IoT]] - market context
- [[Oura Health]] - key competitor in sleep tracking
