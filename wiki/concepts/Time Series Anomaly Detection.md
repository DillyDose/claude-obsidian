---
title: "Time Series Anomaly Detection"
tags:
  - data-science
  - time-series
  - anomaly-detection
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Data Cleaning]]"
  - "[[Time Series Decomposition]]"
  - "[[Time Series ML and DL Models]]"
---

# Time Series Anomaly Detection

## What It Is

Anomaly detection identifies time points or intervals where the series behaves unexpectedly — outside the pattern learned from historical data. Unlike forecasting (predicting future values), anomaly detection evaluates past and present observations.

**Types of anomalies**:
- **Point anomaly**: a single time step with an extreme value (e.g., sensor spike)
- **Contextual anomaly**: a value that is normal globally but abnormal in context (e.g., a sale on Christmas Day for a store that normally closes)
- **Collective anomaly**: a subsequence that is abnormal as a group even if individual points look normal (e.g., a gradual drift before a machine failure)

---

## Why It Matters

- **Data cleaning**: detect anomalies before training a forecasting model to prevent them from distorting lag features and rolling statistics
- **Operations**: alert when a metric behaves unexpectedly (server CPU spike, fraud transaction, production defect rate)
- **Finance**: detect market manipulation, abnormal trading patterns
- **Healthcare**: patient vital sign alerts, disease outbreak detection

---

## Statistical Methods (Fast, Interpretable)

### Z-Score / Modified Z-Score

Flag points more than k standard deviations from the mean (or median for modified Z-score).

```python
from scipy import stats
z_scores = np.abs(stats.zscore(df['value']))
anomalies = z_scores > 3
```

Modified Z-score uses MAD (median absolute deviation) instead of standard deviation — more robust when outliers inflate the std.

### IQR Method

Flag points below `Q1 − 1.5×IQR` or above `Q3 + 1.5×IQR`.

```python
Q1, Q3 = df['value'].quantile([0.25, 0.75])
IQR = Q3 - Q1
anomalies = (df['value'] < Q1 - 1.5*IQR) | (df['value'] > Q3 + 1.5*IQR)
```

### STL Residual Thresholding

The correct approach for seasonal data. Decompose with STL, then apply a threshold on the residual component. This distinguishes genuine anomalies from legitimate seasonal peaks.

```python
from statsmodels.tsa.seasonal import STL
result = STL(df['value'], period=7, robust=True).fit()
residual = result.resid
anomalies = np.abs(residual) > 3 * residual.std()
```

**Always use STL residual thresholding on seasonal data** — plain Z-score will flag every seasonal peak as an anomaly.

### CUSUM (Cumulative Sum)

Detects shifts in the mean level of a series. Accumulates deviations from a target and signals when the sum exceeds a threshold. Used in manufacturing quality control.

---

## Machine Learning Methods

### Isolation Forest

Randomly partitions the feature space using decision trees. Anomalies are isolated with fewer splits (shorter path length) because they are sparse and different.

```python
from sklearn.ensemble import IsolationForest

clf = IsolationForest(contamination=0.05, random_state=42)
# Features: value, lag_1, lag_7, rolling_mean_7, hour, day_of_week
df['anomaly'] = clf.fit_predict(features)  # -1 = anomaly, 1 = normal
```

**Strengths**: unsupervised, no distributional assumptions, handles high-dimensional feature spaces, fast.

**Weakness**: can miss collective anomalies (subsequences). Works best for point anomalies.

### One-Class SVM

Learns a boundary around normal data. Points outside the boundary are anomalies. Effective for low-dimensional data but slow on large datasets.

### Local Outlier Factor (LOF)

Measures the local density of each point relative to its neighbors. Points in low-density regions compared to their neighbors are anomalies. Good at detecting contextual anomalies.

---

## Deep Learning Methods

### LSTM Autoencoder

Train an LSTM autoencoder on **normal data only**. The model learns to reconstruct normal patterns efficiently. When anomalous data is passed in, the reconstruction error is high — this error becomes the anomaly score.

```python
# Architecture:
# Encoder: LSTM → compressed representation
# Decoder: LSTM → reconstructed sequence
# Loss: MSE between input and reconstruction

# At inference:
recon_error = mse(original_sequence, reconstructed_sequence)
anomaly = recon_error > threshold  # threshold set on validation set
```

**Strengths**: captures complex temporal patterns, no feature engineering required, handles multivariate anomalies.

**Weaknesses**: needs substantial normal training data, black-box, threshold selection requires calibration.

### Variational Autoencoder (VAE)

Like an autoencoder but learns a probabilistic latent space. The reconstruction probability serves as the anomaly score. Better calibrated uncertainty than a standard autoencoder.

### Transformer-Based (2024-2025)

Recent models like **Anomaly Transformer** use association discrepancy between self-attention patterns to detect anomalies. Achieves state-of-the-art on benchmark datasets. (Source: arXiv survey 2412.20512 — medium confidence, benchmark-dependent)

---

## Evaluation

Anomaly detection is usually evaluated on labeled datasets with known anomaly times.

| Metric | Description |
|--------|-------------|
| **Precision** | Of flagged anomalies, how many are real? |
| **Recall** | Of real anomalies, how many were flagged? |
| **F1 score** | Harmonic mean of precision and recall |
| **AUC-ROC** | Model's ability to rank anomalies higher than normals |

In practice, anomaly detection is often unsupervised (no labels). Evaluation then relies on domain expert review or downstream impact (did the alert lead to a discovered issue?).

---

## Practical Decision Guide

| Situation | Recommended Method |
|-----------|------------------|
| Seasonal data, quick clean before modeling | STL residual threshold |
| No seasonality, fast exploration | IQR or modified Z-score |
| Many features, no labeled data | Isolation Forest |
| Long sequence, need to capture temporal patterns | LSTM Autoencoder |
| Multivariate sensors | LSTM Autoencoder or Isolation Forest on joint features |
| Need calibrated probability scores | VAE |

---

## Libraries

- `pyod`: Python Outlier Detection — 40+ algorithms including Isolation Forest, LOF, VAE, LSTM Autoencoder
- `statsmodels`: STL decomposition for residual-based detection
- `merlion` (Salesforce): production-grade anomaly detection + forecasting
- `prophet`: has built-in changepoint detection (structural breaks, not point anomalies)
