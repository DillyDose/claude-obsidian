---
title: "Time Series Classical Models"
tags:
  - data-science
  - time-series
  - models
  - statistics
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Decomposition]]"
  - "[[Time Series ML and DL Models]]"
  - "[[Time Series Evaluation Metrics]]"
---

# Time Series Classical Models

## Overview

Classical statistical models are interpretable, fast, and strong baselines. They require stationarity (or differencing to achieve it) and work best on univariate series. They remain competitive with ML models on many real datasets. Always build a classical baseline before reaching for complex models.

## Naive Baselines (Always Build These First)

| Baseline | Formula | When It Wins |
|----------|---------|--------------|
| Last value | `ŷ(t) = y(t-1)` | Random walks (stock prices) |
| Seasonal naive | `ŷ(t) = y(t-m)` | Strong seasonality, no trend |
| Moving average | `ŷ(t) = mean(y(t-k) … y(t-1))` | Smooth series |
| Drift | `ŷ(t) = y(t-1) + avg(Δy)` | Series with trend |

A model that cannot beat these baselines provides no value.

## Exponential Smoothing (ETS)

Exponential smoothing assigns exponentially decreasing weights to past observations — recent observations matter more.

| Model | Handles | Use Case |
|-------|---------|----------|
| SES (Simple Exponential Smoothing) | Level only | No trend, no seasonality |
| Holt's (Double ES) | Level + trend | Trend, no seasonality |
| Holt-Winters (Triple ES) | Level + trend + seasonality | All three components |

The `statsmodels.tsa.holtwinters.ExponentialSmoothing` class implements all three. ETS is the acronym for the state space formulation: Error, Trend, Seasonality.

ETS is often the hardest classical baseline to beat. It is simple, fast, and interpretable.

## ARIMA (AutoRegressive Integrated Moving Average)

ARIMA(p, d, q):
- **p**: autoregressive order — how many lag values to include
- **d**: differencing order — how many times to difference to achieve stationarity
- **q**: moving average order — how many lagged forecast errors to include

**How to choose p, d, q**:
1. Difference until ADF test passes → that gives `d`
2. Read PACF plot → significant lags give `p`
3. Read ACF plot → significant lags give `q`
4. Or use `auto_arima` from `pmdarima` to search automatically

Python: `statsmodels.tsa.arima.model.ARIMA`

## SARIMA (Seasonal ARIMA)

SARIMA(p, d, q)(P, D, Q, m) extends ARIMA with seasonal terms:
- **m**: seasonal period (12 for monthly, 7 for weekly, 24 for hourly)
- **P, D, Q**: seasonal AR, differencing, and MA orders

Fits a separate ARIMA structure at the seasonal lags. More parameters = more complexity. Use `auto_arima` with `seasonal=True` for selection.

## TBATS

TBATS (Trigonometric, Box-Cox, ARMA, Trend, Seasonal) handles **multiple seasonality** periods via Fourier terms. Example: hourly data has both daily (period=24) and weekly (period=168) seasonality.

Python: `tbats` library. Slower to fit than ARIMA but handles complex seasonality automatically.

## Prophet

Developed by Facebook/Meta. Decomposes series into trend + seasonality + holidays using an additive model:

`y(t) = g(t) + s(t) + h(t) + ε(t)`

Strengths:
- Easy to use — minimal parameter tuning
- Handles missing data and holidays natively
- Interprets changepoints in trend automatically
- Outputs uncertainty intervals

Weaknesses:
- Less accurate than tuned ML models for competition use
- Extrapolates trend linearly by default — can go wrong on long horizons

Python: `prophet` (Meta's library). Best for business analysts and quick prototypes.

## VARMA (Vector ARMA)

Multivariate extension of ARIMA. Models multiple related series simultaneously. Each series can depend on lags of itself and lags of other series.

Use when you have multiple series with known cross-dependencies (e.g., multiple correlated economic indicators).

Expensive to fit and interpret. Rarely used in practice beyond 3-5 series.

## When to Use Classical Models

- Series is short (under a few hundred points)
- You need interpretable confidence intervals
- Univariate problem with clear trend/seasonality
- As a baseline before ML models
- When stakeholders need explainability

## When Classical Models Fall Short

- Many series to model simultaneously (hundreds of products, stores)
- External variables (promotions, weather) matter a lot
- Patterns are complex and nonlinear
- Hierarchical structure matters (total = sum of parts must be respected)

In these cases, move to [[Time Series ML and DL Models]].
