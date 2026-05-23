---
type: concept
title: "Continuous Glucose Monitoring"
address: c-000008
created: 2026-05-23
tags:
  - cgm
  - wellness
  - wearables
  - metabolic-health
  - iot
status: developing
related:
  - "[[Wellness IoT]]"
  - "[[iot-wellness-sota-2020-2026]]"
---

# Continuous Glucose Monitoring

Wearable sensors that measure interstitial glucose levels continuously (every 1-15 minutes) via a small subcutaneous filament, transmitting readings via Bluetooth to a smartphone. CGM originated as a medical device for diabetes management and is crossing into the consumer wellness market as of 2024-2025.

## Medical Origins, Wellness Crossover

- **Medical**: Abbott FreeStyle Libre 2/3/3 Plus and Dexcom G7 are FDA/CE-cleared for diabetes management. 15-day wear life (Libre 3, Dexcom G7).
- **Wellness OTC (2024 launches)**: 
  - Abbott **Lingo**: Aimed at metabolic health optimization for non-diabetics.
  - Dexcom **Stelo**: First OTC (no prescription) CGM cleared by FDA, launched 2024.
  - Both integrate with Apple Health.

The crossover is significant: CGM is the most credible real-time metabolic biomarker available outside a lab.

## Key Validation Data

- **Abbott FreeStyle Libre 3** (Alva et al., J. Diabetes Sci. Technol., 2025): 83% wear-completion in adults, 94% pain-free experience; 15-day wear.
- **REFLECT studies** (Abbott, Diabetologia 2025): First real-world evidence of CGM-linked reduction in cardiovascular hospitalization - the strongest economic case for CGM in healthcare/payer decisions.
- **FreeStyle Libre 3 MARD**: Validated in multiple peer-reviewed studies; competitive with Dexcom G7.

## Metabolic Digital Twin Pattern

Companies like Twin Health build a continuously-updated metabolic digital twin from CGM + wearable activity + nutrition data to personalize interventions (e.g., reversing Type 2 diabetes). India digital-twin healthcare market: $800M (2023) projected to $12B (2032) (Saratkar et al., Frontiers in Digital Health 2025).

## Integration with Wellness IoT Stack

CGM readings feed into the smartphone fog tier alongside PPG, ECG, accelerometry, and skin temperature for multimodal biosensor fusion. Foundation models (Apple WBM, Google PH-LLM) do not yet natively include CGM data streams, but Levels-style coaching apps use CGM + LLM for conversational metabolic coaching.

## Market Significance

- Abbott and Dexcom dominate the medical-biosensor wedge that is bleeding into wellness.
- Eversense (implantable, 180-day CGM) and Senseonics target the clinical end.
- The OTC launch of Stelo (Dexcom, 2024) is the clearest signal that CGM is becoming a consumer wellness device.

## Key Players

- **Abbott**: FreeStyle Libre 2/3/3 Plus (medical) + Lingo (wellness)
- **Dexcom**: G7 (medical) + Stelo (OTC wellness, 2024)
- **Eversense/Senseonics**: Implantable long-wear CGM
- **Twin Health**: Metabolic digital twin using CGM + activity + nutrition

## Related

- [[Wellness IoT]] - broader wearable context
- [[Apple Health Ecosystem]] - Apple Health integration partner for Stelo/Lingo
