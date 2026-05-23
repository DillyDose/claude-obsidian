---
title: "Time Series Decomposition"
tags:
  - data-science
  - time-series
  - decomposition
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Data Cleaning]]"
  - "[[Time Series Feature Engineering]]"
---

# Time Series Decomposition

## What It Is

Decomposition separates a time series into its structural parts: trend, seasonality, and residual noise. It is both an analytical tool (to understand data) and a preprocessing step (to help models focus on meaningful signal).

## The Three Components

**Trend (T)**: The long-run direction of the series. Can be linear, exponential, or irregular. Identified by smoothing or regression.

**Seasonality (S)**: Fixed, calendar-driven repetition. Examples: retail sales peak every December, energy demand peaks every weekday morning. Period is known in advance.

**Residual / Noise (R)**: What remains after removing trend and seasonality. Ideally random; if structure remains, the model missed something.

## Additive vs. Multiplicative

| Type | Formula | When to Use |
|------|---------|-------------|
| Additive | `Y = T + S + R` | Seasonal fluctuation is constant in size |
| Multiplicative | `Y = T × S × R` | Seasonal fluctuation grows proportionally with trend |

A log transform converts multiplicative to additive: `log(Y) = log(T) + log(S) + log(R)`

## Classical Decomposition

Two methods:
- **Moving average decomposition**: Estimate trend via centered moving average, subtract to get seasonal + residual, then average residuals by period to isolate seasonality.
- **X-11 / X-13-ARIMA-SEATS**: US Census Bureau methods used in official statistics. Handles trading day effects, holiday effects.

Limitation of classical decomposition: assumes seasonality is identical each year (no evolution).

## STL Decomposition

STL (Seasonal and Trend decomposition using Loess) is the modern standard. It:
- Allows seasonality to change gradually over time
- Is robust to outliers
- Handles any seasonality period (daily, weekly, hourly)
- Is available in Python via `statsmodels.tsa.seasonal.STL`

## MSTL (Multiple Seasonality)

For data with multiple seasonal periods (e.g., hourly data has daily and weekly seasonality), use MSTL (Multiple Seasonal and Trend decomposition using Loess).

## Stationarity

A stationary series has constant mean, variance, and autocorrelation over time. Most classical models (ARIMA) require stationarity. Most ML models benefit from it.

**Tests for stationarity:**
- ADF (Augmented Dickey-Fuller): p < 0.05 means stationary
- KPSS: p > 0.05 means stationary
- They test complementary null hypotheses — run both

**To achieve stationarity:**
- Differencing: `Δy(t) = y(t) − y(t−1)` (removes trend)
- Seasonal differencing: `Δ_m y(t) = y(t) − y(t−m)` (removes seasonality)
- Log transform: stabilizes variance
- Box-Cox transform: generalizes log transform

## Autocorrelation

- **ACF (Autocorrelation Function)**: Correlation of the series with its own lags. Shows which lags are informative.
- **PACF (Partial Autocorrelation Function)**: Correlation at each lag after removing the effect of shorter lags. Used to identify ARIMA order.

These plots are essential diagnostic tools — always generate them before modeling.
