---
title: "Time Series Intermittent Demand"
tags:
  - data-science
  - time-series
  - intermittent-demand
  - sparse
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Classical Models]]"
  - "[[Time Series ML and DL Models]]"
  - "[[Time Series Evaluation Metrics]]"
---

# Time Series Intermittent Demand

## What It Is

Intermittent (or sparse) demand series have many zero values punctuated by occasional non-zero demand. Common in retail (slow-moving products, spare parts), manufacturing (replacement components), and healthcare (rare drug prescriptions).

Standard forecasting methods fail on these series because:
- ARIMA assumes continuous, normally distributed errors — zeros violate this
- Simple mean forecasting over-smooths and misses non-zero events
- MAPE is undefined or unstable near zero
- Standard neural networks treat zeros like any small value

---

## Characteristics to Measure

Before choosing a method, diagnose the series:

**Average Inter-Demand Interval (ADI)**: average number of periods between non-zero demands.
- ADI < 1.32: non-intermittent (use standard methods)
- ADI ≥ 1.32: intermittent

**Coefficient of Variation Squared (CV²)**: squared coefficient of variation of non-zero demand sizes.
- CV² < 0.49: demand size is stable (lumpy if ADI is high)
- CV² ≥ 0.49: demand size is erratic

```
              CV² < 0.49       CV² ≥ 0.49
ADI < 1.32   Smooth           Erratic
ADI ≥ 1.32   Intermittent     Lumpy
```

Each quadrant benefits from a different approach.

---

## Classical Methods

### Croston's Method

Separately models:
1. The **size** of non-zero demands (using simple exponential smoothing, updated only on demand-occurring periods)
2. The **interval** between demands (using simple exponential smoothing on inter-arrival times)

Forecast = `(demand size estimate) / (interval estimate)`

```python
from statsforecast.models import CrostonOptimized
from statsforecast.core import StatsForecast

sf = StatsForecast(models=[CrostonOptimized()], freq='D')
sf.fit(df)
forecasts = sf.predict(h=28)
```

**Limitation**: Croston's method is biased. It systematically over-forecasts.

### Syntetos-Boylan Approximation (SBA)

Applies a bias correction factor to Croston's forecast:

`ŷ_SBA = ŷ_Croston × (1 − α_interval/2)`

SBA outperforms Croston's in most empirical studies and is preferred in practice. Available as `CrostonSBA` in Nixtla's statsforecast.

### TSB (Teunter-Syntetos-Babai)

Unlike Croston (which updates only on demand-occurring periods), TSB updates the probability of a demand event occurring every period. Better at handling obsolescence — when a product stops selling, TSB adapts faster.

---

## Distribution-Based Approaches

### Zero-Inflated Models

Treat the series as a mixture:
- With probability `p`: demand = 0 (no event)
- With probability `1-p`: demand follows a Poisson or negative binomial distribution

Fit the two components separately. Good when the zero-generating process is distinct from the demand-generating process (e.g., a store is closed on Sundays).

### Tweedie Distribution

The Tweedie distribution naturally handles a mix of exact zeros and positive continuous values. It is parameterized by a power `p`:
- `p = 1`: Poisson
- `p = 2`: Gamma (no zeros)
- `1 < p < 2`: Tweedie compound Poisson-Gamma (exact zeros + positive values)

LightGBM and XGBoost have a built-in Tweedie objective:

```python
params = {
    'objective': 'tweedie',
    'tweedie_variance_power': 1.5,  # tune this (1.0-1.9)
    'n_estimators': 1000,
    'learning_rate': 0.03,
}
model = lgb.LGBMRegressor(**params)
```

**The tweedie objective is the standard choice for intermittent retail demand in LightGBM/XGBoost**. The M5 competition had extensive intermittent demand, and top solutions used tweedie. (Source: M5 competition analysis — high confidence)

**Tuning the power parameter**:
- Power closer to 1: more weight on count-like behavior (bursty)
- Power closer to 2: more weight on continuous behavior

Tune via cross-validation. Try `[1.1, 1.3, 1.5, 1.7, 1.9]`.

---

## Deep Learning Approaches

### DeepAR (Amazon)

Handles intermittent demand by using negative binomial or zero-inflated Poisson output distributions instead of Gaussian. Learns from many series simultaneously (global model).

### Deep Renewal Processes

A probabilistic framework that models demand arrivals as a renewal process. Natively handles intermittent demand patterns including aging, clustering, and quasi-periodicity. (Source: PMC Nature paper — medium confidence)

---

## Evaluation Metrics for Intermittent Demand

**Do not use MAPE** — undefined when actual = 0.

| Metric | Notes |
|--------|-------|
| **MAE** | Robust, but treats 0s and non-zeros equally |
| **MASE** | Scale-free, handles zeros, compare against seasonal naive |
| **RMSSE** | Like MASE but for RMSE — used in M5 |
| **Pinball loss** | When forecasting quantiles (stock management) |
| **Service level / fill rate** | Business metric: fraction of demand met from stock |

---

## Practical Approach

1. **Diagnose**: compute ADI and CV². Is it truly intermittent?
2. **Quick baseline**: Croston SBA (fast, strong baseline for intermittent)
3. **Global ML model**: LightGBM with `tweedie` objective + lag features
4. **If zeros are structural** (e.g., store closed): encode the closure as a feature, not as zero demand to model
5. **Post-process**: floor predictions at zero (`preds = np.maximum(preds, 0)`)
6. **Evaluate with MASE**, not MAPE

In hackathons with retail data, always check the fraction of zeros in the target. If >30% are zeros, apply the tweedie objective immediately — it will improve your score noticeably.
