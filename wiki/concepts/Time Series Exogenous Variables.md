---
title: "Time Series Exogenous Variables"
tags:
  - data-science
  - time-series
  - exogenous
  - covariates
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Feature Engineering]]"
  - "[[Time Series Classical Models]]"
  - "[[Time Series ML and DL Models]]"
---

# Time Series Exogenous Variables

## What They Are

Exogenous variables (also called external regressors or covariates) are inputs that influence the target series but are **not modeled as part of the system** — they are given, not predicted. Examples: temperature, promotions, holidays, day-of-week, competitor price.

Contrast with endogenous variables: modeled as part of the system (e.g., in VAR, every series is both input and output).

---

## The Critical Distinction: Known vs Unknown Future

The most important question when adding any exogenous variable:

**Is this variable known at prediction time for the entire forecast horizon?**

| Type | Examples | Can Use for Forecasting? |
|------|---------|--------------------------|
| **Future-known** | Calendar features, planned promotions, holiday schedule, day-of-week | Yes — always available |
| **Future-unknown** | Temperature (must be forecasted), competitor actions, market news | Only if you have a reliable forecast of it |
| **Past-only** | Sales of a product already discontinued | Only as lagged features |

Using a future-unknown variable without forecasting it first introduces data leakage in training and breaks predictions at inference time. (Source: Statsmodels SARIMAX documentation — high confidence)

---

## Classical Models with Exogenous Variables

### ARIMAX / SARIMAX

Extends ARIMA by adding external regressors as linear terms. The ARIMA part models the autocorrelated residuals after accounting for the regressors.

`Y(t) = β·X(t) + ARIMA errors`

**Key nuance**: ARIMAX is **not** the same as regressing Y on X and then fitting ARIMA to the residuals. The correct formulation fits both simultaneously so the regression errors follow the ARIMA structure.

```python
from statsmodels.tsa.statespace.sarimax import SARIMAX

model = SARIMAX(endog=y_train,
                exog=X_train,           # external regressors
                order=(p, d, q),
                seasonal_order=(P, D, Q, m))
result = model.fit()

# At prediction time, future X must be provided
forecast = result.forecast(steps=h, exog=X_future)
```

**Stationarity**: if X is non-stationary, difference it before including. Non-stationary regressors can cause spurious relationships.

**Multicollinearity**: if regressors are highly correlated, coefficient estimates become unstable. Check VIF (Variance Inflation Factor).

### Prophet with Regressors

Prophet natively supports additional regressors via `add_regressor()`:

```python
model = Prophet()
model.add_regressor('temperature')
model.add_regressor('is_promotion')
model.fit(df_train)  # df must contain 'ds', 'y', 'temperature', 'is_promotion'
```

Prophet assumes a linear relationship between each regressor and the target. Non-linear relationships require feature engineering first.

---

## ML Models with Exogenous Variables

Tree-based models handle exogenous variables naturally as additional columns. No special treatment needed — just include them as features alongside lag features and calendar features.

```python
features = ['lag_1', 'lag_7', 'roll_mean_28',        # time series features
            'temperature', 'is_promotion', 'price',    # exogenous features
            'day_of_week_sin', 'month_cos']             # calendar features

model.fit(X_train[features], y_train)
```

**Advantage over ARIMAX**: tree models capture non-linear interactions between exogenous variables and the target automatically.

**Handling future-unknown variables**: if temperature is needed for the next 14 days, first obtain a weather forecast, then use those forecasted temperatures as inputs. The quality of the exogenous forecast directly caps the quality of the final forecast.

---

## Types of Exogenous Variables to Consider

### Planned Events
- Promotions / discounts (retail)
- Marketing campaigns
- Product launches
- Earnings announcements (finance)

### Calendar and Time
- Public holidays (country-specific)
- School calendar (term vs. break)
- Fiscal periods (month-end, quarter-end)
- Sporting events (Super Bowl, World Cup)

### Environmental
- Temperature, precipitation (energy, agriculture, retail)
- Daylight hours (energy)

### Economic Indicators
- Interest rates, CPI (financial forecasting)
- Competitor pricing
- Consumer confidence index

### Operational
- Number of stores open
- SKU availability / stockouts
- Shipping delays

---

## Feature Engineering for Exogenous Variables

**Lag the exogenous variable**: if a promotion causes a demand spike that lasts 3 days, include `promo_lag_0`, `promo_lag_1`, `promo_lag_2` to capture the decay.

**Lead indicators**: if a marketing campaign runs next week, include a flag `days_until_campaign` in the current week's features — the model can learn anticipatory effects.

**Interaction terms**: multiply the exogenous variable with a seasonal dummy to capture seasonal amplification (e.g., temperature matters more in summer than winter).

---

## Diagnostics

After fitting a model with exogenous variables:
1. Check residuals for autocorrelation (ACF plot) — if residuals still show structure, the model missed something
2. Compare AIC/BIC to the same model without regressors — lower is better
3. Check coefficient signs make business sense (if higher promotion coefficient, does sales go up?)
4. Test on a holdout period that includes realistic variation in the exogenous variables
