---
title: "Time Series Multivariate"
tags:
  - data-science
  - time-series
  - multivariate
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Classical Models]]"
  - "[[Time Series ML and DL Models]]"
  - "[[Time Series Exogenous Variables]]"
  - "[[Time Series Feature Engineering]]"
---

# Time Series Multivariate

## What It Is

Multivariate time series analysis models **multiple series simultaneously**, allowing each series to influence the others. This is different from adding exogenous variables (where external inputs are fixed inputs, not modeled): in a multivariate system, every series can both cause and be caused by the others.

**When it matters**: stock prices of related companies, temperature and energy demand, multiple sensor readings on one machine, macro economic indicators that co-move.

## Two Approaches

**Approach A — True multivariate models (VAR family)**: model all series jointly in one system. Captures cross-series causality. Requires all series to be available at prediction time.

**Approach B — Global ML models**: train one ML model (LightGBM) across all series, using each series as a row with its own lag features. Implicitly shares structure but does not model cross-series causality explicitly.

For hackathons, Approach B (global LightGBM) almost always wins on performance and speed. Approach A is more interpretable and theoretically principled.

---

## Classical Multivariate Models

### VAR — Vector AutoRegression

The extension of AR to multiple series. Each series is regressed on its own lags and the lags of every other series.

`Y(t) = A₁·Y(t-1) + A₂·Y(t-2) + ... + Aₚ·Y(t-p) + ε(t)`

Where `Y(t)` is a vector of all series at time t and `Aᵢ` are coefficient matrices.

**Order selection**: use AIC/BIC to choose lag order `p`.

**Stationarity**: all series must be stationary. Apply differencing before fitting.

**Limitation**: number of parameters grows as `k² × p` where `k` = number of series. With many series, VAR overfits badly. Use Bayesian VAR (BVAR) with shrinkage priors for 10+ series.

Python: `statsmodels.tsa.vector_ar.var_model.VAR`

### VECM — Vector Error Correction Model

Use when series are non-stationary but **cointegrated** (they share a long-run equilibrium relationship, e.g., price and cost tend to move together over time).

First test for cointegration using the Johansen test. If cointegration exists, use VECM instead of VAR on differenced data — VECM preserves the long-run relationship.

### VARMA

Extends VAR with moving-average terms. Rarely used in practice due to identification difficulties. VAR is usually sufficient.

---

## Global Models (the Practical Multivariate Approach)

Train **one model across all series**, with a series ID as a categorical feature. The model learns shared patterns while series-specific features provide individuality.

```python
import lightgbm as lgb

# Stack all series vertically
# Each row: one time step of one series
# Features: lag_1, lag_7, roll_mean_28, month, series_id, store_id, ...

df_all = pd.concat([series_A, series_B, series_C], keys=['A','B','C'])
df_all['series_id'] = df_all.index.get_level_values(0)

model = lgb.LGBMRegressor(...)
model.fit(X_train, y_train)
```

**Why it works**: series with little history borrow strength from series with more history. The model generalizes across similar patterns.

**When it works best**: many series of the same type (100+ products, stores, regions) with shared seasonal patterns.

---

## Cross-Correlation and Granger Causality

Before building a multivariate model, test whether series actually influence each other.

**Cross-correlation function (CCF)**: measures correlation between series A at time t and series B at time t+k. A spike at lag k suggests A leads B by k periods.

**Granger causality test**: tests whether past values of series X improve the forecast of series Y beyond what Y's own past provides. "Granger-causes" does not mean true causation — it is a predictive test only.

```python
from statsmodels.tsa.stattools import grangercausalitytests
grangercausalitytests(df[['Y', 'X']], maxlag=4)
# p < 0.05: X Granger-causes Y
```

---

## Temporal Fusion Transformer (TFT) for Multivariate

TFT is the leading deep learning model for multivariate time series. It handles:
- Static metadata (store type, region)
- Past-observed covariates (past weather, past sales of related products)
- Future-known covariates (planned promotions, calendar)
- Multiple output quantiles (probabilistic forecasting)

It uses variable selection networks to automatically weight which inputs matter at each time step. Strong on the M5 and electricity benchmarks. (Source: Lim et al. 2021 — high confidence)

Python: `neuralforecast` library, `pytorch-forecasting` library.

---

## Practical Tips

- Start with a global LightGBM model. Add series_id as a categorical feature. This beats VAR on most practical datasets.
- Use Granger causality to identify which cross-series features to add as lag features.
- Only use VAR if you need impulse-response analysis or cointegration modeling (econometrics context).
- TFT is worth trying when you have rich metadata and multiple covariate types — budget extra time for setup.
