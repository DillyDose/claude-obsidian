---
title: "Time Series Evaluation Metrics"
tags:
  - data-science
  - time-series
  - metrics
  - evaluation
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Classical Models]]"
  - "[[Time Series ML and DL Models]]"
  - "[[Time Series Hackathon Guide]]"
---

# Time Series Evaluation Metrics

## Validation Before Metrics: Time-Based Splits

**Never use random cross-validation for time series.** It shuffles the time order, allowing future data to inform past predictions (leakage). Instead:

**Walk-forward validation (expanding window)**:
- Fold 1: train on months 1-6, validate on month 7
- Fold 2: train on months 1-7, validate on month 8
- Fold 3: train on months 1-8, validate on month 9

**Sliding window validation**:
- Same as walk-forward but with a fixed training window size

Use 3-5 folds. Average metrics across folds for a stable estimate.

---

## Core Metrics

### MAE — Mean Absolute Error

`MAE = mean(|actual − predicted|)`

- Unit: same as the target (dollars, units, etc.)
- Treats all errors equally (no penalty for large errors)
- Robust to outliers
- **Intuition**: on average, my forecast is off by X units

**When to use**: default metric when you want a plain, interpretable summary of average error size.

---

### RMSE — Root Mean Squared Error

`RMSE = sqrt(mean((actual − predicted)²))`

- Unit: same as the target
- Penalizes large errors more heavily than MAE
- Sensitive to outliers
- **Intuition**: RMSE > MAE means your errors are uneven (some very large)

**When to use**: when large errors are disproportionately costly. Many optimization algorithms minimize squared error naturally, so RMSE is a natural training objective.

---

### MAPE — Mean Absolute Percentage Error

`MAPE = mean(|actual − predicted| / |actual|) × 100`

- Unit: percentage (scale-free)
- Lets you compare accuracy across series of different magnitudes
- **Fatal flaw**: explodes when `actual = 0` or is near zero

**When to use**: comparing forecast quality across products/stores/regions. Avoid for intermittent demand series with zeros.

---

### sMAPE — Symmetric MAPE

`sMAPE = mean(2 × |actual − predicted| / (|actual| + |predicted|)) × 100`

- Symmetrizes MAPE by using the average of actual and predicted in the denominator
- Still problematic when both actual and predicted are near zero
- Used in M4 and M5 competitions

**When to use**: competition leaderboards that specify it. Otherwise, prefer MASE.

---

### MASE — Mean Absolute Scaled Error

`MASE = MAE / MAE_naive_seasonal`

Where `MAE_naive_seasonal` is the MAE of a seasonal naive forecast on the training set.

- Scale-free: comparable across series of different magnitudes and frequencies
- MASE < 1: your model beats the seasonal naive benchmark
- MASE > 1: your model is worse than just using last year's same-period value
- Handles zeros correctly
- **The recommended metric for multi-series comparison** (Source: Hyndman & Koehler — high confidence)

**When to use**: whenever you compare across multiple series with different scales.

---

### WRMSSE — Weighted RMSSE

Used in the M5 competition. Scales RMSSE by sales volume, so high-volume products contribute more to the total score. Important for real-world retail where errors on high-revenue products matter more.

---

## Probabilistic Metrics

When forecasting uncertainty (prediction intervals) matters:

| Metric | What It Measures |
|--------|-----------------|
| **Coverage** | What fraction of actuals fall inside the stated 95% interval |
| **Interval Width** | Narrower intervals are better, if coverage is maintained |
| **CRPS** (Continuous Ranked Probability Score) | Compares full predicted distribution to actual. Lower is better |
| **Pinball Loss** (Quantile Loss) | Accuracy of a specific quantile forecast |

Probabilistic forecasting is critical in supply chain (safety stock sizing requires knowing the upper tail, not just the mean).

---

## Choosing the Right Metric

| Situation | Recommended Metric |
|-----------|-------------------|
| Single series, interpretable result | MAE |
| Large errors are expensive | RMSE |
| Comparing across multiple series | MASE |
| Competition leaderboard uses MAPE | sMAPE (check if zeros exist) |
| Quantifying business risk (inventory) | Pinball Loss / CRPS |
| Checking if model beats naive | MASE < 1 |

**Always report at least two metrics** (e.g., MAE + MASE) to capture both absolute error magnitude and relative performance vs. a naive baseline.

---

## Baseline Comparison Requirement

Every metric is meaningless without context. Always compare your model to:
1. Seasonal naive (last year's same period)
2. Simple exponential smoothing
3. A domain expert's current manual forecast (if available)

A model with MASE = 0.85 (15% better than naive) may or may not be good enough for a business decision — that judgment requires domain knowledge.
