---
type: concept
title: "TinyML and Edge AI"
address: c-000006
created: 2026-05-23
tags:
  - tinyml
  - edge-ai
  - iot
  - health-tech
  - ml
status: developing
related:
  - "[[Wellness IoT]]"
  - "[[Federated Learning in Healthcare]]"
  - "[[Wearable Foundation Models]]"
  - "[[iot-wellness-sota-2020-2026]]"
---

# TinyML and Edge AI

The deployment of machine learning models on microcontrollers and embedded processors in IoT devices, enabling real-time inference without cloud round-trips. In wellness IoT, TinyML powers first-pass anomaly detection at the sensor level while larger models run in fog (smartphone) or cloud tiers.

## Definition

TinyML: neural networks running under 1 mW power and within <1 MB RAM on ARM Cortex-class or purpose-built AI microcontrollers. The constraint profile (micro-watts, kilobytes) distinguishes it from embedded ML on smartphone-class SoCs.

## Three-Tier Reference Architecture (2024-2026)

```
[Sensor + MCU edge]  ->  [Smartphone fog]  ->  [Cloud back-end]
TinyML first-pass       Multimodal fusion      Federated training
  ECG anomaly           HRV/sleep staging      Foundation models
  Fall detection        OS encryption           Digital twins
  EDA stress spike      App UI layer
```

**Tier 1 - Edge (TinyML):**
- Microcontrollers: STMicroelectronics, Nordic Semiconductor, Ambiq
- Models: CNNs, LSTMs, bio-inspired spiking neural networks
- Use cases: ECG arrhythmia detection, fall detection, lung-sound asthma detection, blood-pressure estimation, EDA stress spikes
- Power: <1 mW; Memory: <1 MB RAM

**Tier 2 - Smartphone fog:**
- Apple Secure Enclave, Samsung Knox for encryption
- HRV analysis, sleep-stage classification, multimodal biosensor fusion
- Local inference for privacy-sensitive data before cloud upload

**Tier 3 - Cloud:**
- Population-scale training via [[Federated Learning in Healthcare]]
- [[Wearable Foundation Models]] trained and updated here
- Digital-twin back-ends (Twin Health, Holisticare, USC Center for Body Computing)

## Key Research (2024-2026)

- **Nguyen et al. (ICTA 2025, Springer LNNS 1831)**: Systematic review of 59 TinyML wearable studies (2020-Apr 2025). Documents migration of CNNs/LSTMs/SNNs from cloud to sub-1mW microcontrollers.
- **Royal Society Open Science (2025)**: Sub-1mW micro-trainers enabling on-device fine-tuning, not just inference - eliminating cloud dependency for model updates.
- **MDPI Healthcare FL review (Dec 2024)**: Positions federated learning as the canonical training paradigm complementing TinyML on-device inference.

## Applications in Wellness IoT

| Application | Tier | Model Type |
|---|---|---|
| ECG arrhythmia detection | Edge | CNN / LSTM |
| Fall detection | Edge | CNN + accelerometry |
| Lung-sound asthma classification | Edge | CNN |
| Sleep-stage classification | Fog | LSTM / transformer |
| HRV stress analysis | Fog | Signal processing + ML |
| Foundation model pretraining | Cloud | Mamba-2 / transformer |

## Ambient/Passive IoT (Emerging)

Battery-less RFID/BLE tags from Wiliot, HaiLa, Lightricity, ONiO operate at energy-harvesting level (micro-watts). Healthcare/wellness is one of the three primary verticals (alongside smart buildings and supply-chain logistics) per Omdia Sept 2025. Still in pilot phase, not mainstream deployment.

## Energy Trends

- Premium consumer wearables moving from 1-day (Apple Watch) to 14-day (Whoop 2025, smart rings) via screen-less design and optimized TinyML inference.
- On-device fine-tuning (Royal Society Open Science 2025) reduces need for cloud retraining rounds.

## Related

- [[Wellness IoT]] - broader category and market context
- [[Federated Learning in Healthcare]] - cloud-tier complement
- [[Wearable Foundation Models]] - models trained in cloud, lightweight inferences run at fog/edge
