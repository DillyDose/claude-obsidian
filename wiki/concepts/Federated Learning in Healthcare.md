---
type: concept
title: "Federated Learning in Healthcare"
address: c-000007
created: 2026-05-23
tags:
  - federated-learning
  - privacy
  - ml
  - health-tech
  - iot
status: developing
related:
  - "[[Wellness IoT]]"
  - "[[TinyML and Edge AI]]"
  - "[[Wearable Foundation Models]]"
  - "[[iot-wellness-sota-2020-2026]]"
---

# Federated Learning in Healthcare

A distributed machine learning paradigm where model training occurs on decentralized data (devices or hospital nodes) without raw data leaving its source. The aggregated model updates (gradients) are shared, not the data itself. In healthcare and wellness IoT, federated learning is the canonical approach to training population-scale models while preserving patient privacy.

## Why Federated Learning in Healthcare

- **Regulatory**: HIPAA, GDPR, and the EU Data Act create legal barriers to centralizing raw health data.
- **Breach risk**: Change Healthcare ransomware (Feb 2024) affected ~190M individuals - centralized health records are high-value targets. 67% of healthcare orgs experienced ransomware in 2024 (Sophos).
- **IoMT constraints**: Many medical devices cannot run conventional antivirus; Zero-Trust micro-segmentation + FL is the recommended alternative.
- **Scale**: Cross-hospital training on rare conditions requires pooling data without sharing it.

## Architecture Variants

**Standard FL**: Devices train on local data, upload gradients to central aggregator. Server aggregates (FedAvg or variants), pushes updated global model back.

**FedHealthFog (Computer Communications, 2024)**: FL aggregation over a fog-IoT layer (smartphone tier) rather than direct device-to-cloud. Optimizes latency and energy vs single-server FL baselines.

**Blockchain-FL (Nature Scientific Reports, 2025)**: Blockchain for audit trail and gradient integrity verification across institutional participants. Prevents model poisoning in cross-hospital settings.

## Key References

- **MDPI Healthcare FL review (Dec 2024)**: "Federated Learning in Smart Healthcare" - comprehensive treatment of privacy, security, predictive analytics with IoT integration. Canonical reference.
- **FedHealthFog (Computer Communications, 2024)**: Delay- and energy-optimized FL aggregation over fog-IoT.
- **Nature Scientific Reports (2025)**: Blockchain-FL IoT framework for cross-institutional wellness models.
- **Duke University (Jiang et al., Scientific Reports 2024)**: Federated multi-domain learning for health monitoring.

## Integration with Wellness IoT Stack

In the three-tier wellness IoT architecture:
- **Edge (TinyML)**: Inference only; no training.
- **Smartphone fog**: Optional local training for personalized adaptation; only gradient updates leave the device.
- **Cloud**: FL aggregation server; optionally decentralized via blockchain or fog relay.

See [[TinyML and Edge AI]] for full stack description.

## Limitations and Open Problems

- **Communication cost**: Gradient uploads are still bandwidth-intensive for very large models.
- **Non-IID data**: Health data across devices/hospitals is highly heterogeneous; standard FedAvg degrades.
- **Model poisoning**: Malicious participants can corrupt the global model via adversarial gradients; blockchain + differential privacy mitigate this.
- **Convergence speed**: FL typically requires more rounds than centralized training.

## Differential Privacy Connection

FL is often combined with differential privacy (DP): noise is added to gradients before upload, limiting what can be inferred about any individual's data even from the shared gradients. DP + FL is the pattern cited across MDPI Healthcare review, FedHealthFog, and the Nature Scientific Reports 2025 framework.

## Related

- [[Wellness IoT]] - broader deployment context
- [[TinyML and Edge AI]] - edge tier that FL complements
- [[Wearable Foundation Models]] - large models trained via FL on population-scale cohorts
