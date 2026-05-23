---
title: "Time Series Probabilistic Forecasting"
tags:
  - data-science
  - time-series
  - probabilistic
  - uncertainty
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Evaluation Metrics]]"
  - "[[Time Series ML and DL Models]]"
  - "[[Time Series Production Use]]"
---

# Time Series Probabilistic Forecasting

## What It Is

Probabilistic forecasting outputs a **distribution** over future values, not just a single point. This quantifies uncertainty and enables risk-aware decisions.

A point forecast says "we expect 1,000 units." A probabilistic forecast says "there is a 90% chance demand falls between 750 and 1,350 units." The latter enables:
- Safety stock sizing (use the 95th percentile to avoid stockouts)
- Budget planning (use the 50th percentile as base case, 80th as stretch)
- Risk management (know the downside tail)

---

## Output Types

| Output | Description | Use Case |
|--------|-------------|---------|
| **Prediction intervals** | Lower and upper bounds for a given coverage level (e.g., 80%, 95%) | Communication, dashboards |
| **Quantile forecasts** | Specific percentiles (Q10, Q50, Q90) | Supply chain, finance |
| **Full predictive distribution** | Complete probability distribution over future values | Risk modeling, simulation |
| **Samples** | Draws from the predictive distribution | Monte Carlo simulation |

---

## Method 1: Conformal Prediction

**The most practical and model-agnostic approach** for prediction intervals.

Conformal prediction wraps any existing point-forecast model and adds statistically valid coverage guarantees without distributional assumptions.

**How it works**:
1. Train a model on the training set
2. Compute residuals on a calibration set: `residual(t) = |actual(t) − predicted(t)|`
3. Find the quantile of residuals that gives the desired coverage (e.g., 90th percentile for 90% interval)
4. At prediction time: interval = `[ŷ − q, ŷ + q]`

**Key property**: if the calibration set is representative, the stated coverage is guaranteed regardless of the model used.

**Conformalized Quantile Regression (CQR)**: a stronger variant that produces adaptive intervals (wider when uncertainty is higher, narrower when the model is confident). (Source: arXiv 2202.08756, IEEE Xplore — high confidence)

```python
from mapie.time_series import MapieTimeSeriesRegressor
mapie = MapieTimeSeriesRegressor(estimator=base_model, method="enbpi")
mapie.fit(X_train, y_train)
y_pred, y_pis = mapie.predict(X_test, alpha=0.1)  # 90% interval
```

---

## Method 2: Quantile Regression

Train the model to directly predict a specific quantile instead of the mean.

**LightGBM quantile loss**:
```python
# Predict the 90th percentile (upper bound)
params_q90 = {'objective': 'quantile', 'alpha': 0.90}
model_q90 = lgb.LGBMRegressor(**params_q90)

# Predict the 10th percentile (lower bound)
params_q10 = {'objective': 'quantile', 'alpha': 0.10}
model_q10 = lgb.LGBMRegressor(**params_q10)
```

Train separate models for each quantile you need (Q10, Q50, Q90). The 50th percentile model is equivalent to MAE minimization.

**Advantage**: intervals automatically adapt to local variance in the data (heteroscedastic uncertainty).

**Disadvantage**: quantile crossing can occur — Q90 may dip below Q10 for some inputs. Apply isotonic regression post-processing to fix.

---

## Method 3: Bootstrapping

Generate multiple future paths by sampling from historical residuals:

1. Train a base model, collect residuals on the training set
2. For each forecast, add randomly sampled residuals to the point forecast
3. Repeat N times (N=500) to get a distribution of future paths
4. Compute percentiles across paths

This captures uncertainty but assumes residuals are stationary and independent — both are often violated. Use conformal prediction instead when possible.

---

## Method 4: Native Probabilistic Models

Some models output distributions natively:

| Model | Distribution Type |
|-------|-----------------|
| **DeepAR** (Amazon) | Parametric (Gaussian, negative binomial, etc.) |
| **TFT** | Multiple quantiles simultaneously |
| **N-BEATS** (with quantile head) | Multiple quantiles |
| **GARCH** | Volatility modeling for financial returns |
| **Bayesian Structural Time Series** | Full posterior distribution |
| **GPyTorch GP regression** | Gaussian process posterior |

GARCH specifically models **time-varying variance** (volatility clustering) — when markets are volatile, tomorrow is also likely to be volatile. Essential for financial risk modeling.

---

## Evaluation of Probabilistic Forecasts

**Coverage**: what fraction of actuals fall inside the stated interval?
- A stated 95% interval should contain 95% of actuals. If it contains 99%, the interval is too wide (conservative). If 85%, too narrow (overconfident).

**Interval width**: narrower intervals are better, assuming coverage is maintained.

**CRPS (Continuous Ranked Probability Score)**: the standard metric for comparing full predictive distributions. Penalizes both overconfidence and underconfidence.

```python
from properscoring import crps_ensemble
crps_score = crps_ensemble(y_true, ensemble_samples)
```

**Pinball loss (Quantile loss)**: for evaluating a specific quantile prediction.

```python
def pinball_loss(y_true, y_pred, alpha):
    error = y_true - y_pred
    return np.mean(np.maximum(alpha * error, (alpha - 1) * error))
```

---

## Practical Recommendation

For hackathons:
1. Start with conformal prediction wrapped around your LightGBM model — fast to implement, valid coverage
2. If the competition scores on quantiles (e.g., Q10, Q50, Q90), train separate quantile regression models
3. If scoring on CRPS, use DeepAR or ensemble conformal prediction for best results

For production:
- Conformal prediction for robustness and simplicity
- Quantile LightGBM for supply chain optimization
- GARCH for financial volatility
