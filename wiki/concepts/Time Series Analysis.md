---
title: "Time Series Analysis"
tags:
  - data-science
  - time-series
  - hub
status: evergreen
related:
  - "[[Time Series Real-World Applications]]"
  - "[[Time Series Data Cleaning]]"
  - "[[Time Series Feature Engineering]]"
  - "[[Time Series Classical Models]]"
  - "[[Time Series ML and DL Models]]"
  - "[[Time Series Evaluation Metrics]]"
  - "[[Time Series Decomposition]]"
  - "[[Time Series Hackathon Guide]]"
  - "[[Time Series Production Use]]"
  - "[[Time Series Multivariate]]"
  - "[[Time Series Hierarchical]]"
  - "[[Time Series Exogenous Variables]]"
  - "[[Time Series Probabilistic Forecasting]]"
  - "[[Time Series Anomaly Detection]]"
  - "[[Time Series Intermittent Demand]]"
  - "[[Time Series AutoML]]"
  - "[[Time Series Multi-Step Forecasting]]"
  - "[[Time Series Advanced Ensembling]]"
---

# Time Series Analysis

## What It Is

Time series analysis is the study of data points collected or recorded at successive points in time, typically at uniform intervals. The goal is to extract meaningful structure — trends, cycles, patterns — and use that understanding to explain, monitor, or predict behavior over time.

It is one of the most applied data science techniques across business, science, and engineering. (Source: InfluxData, 2025 — high confidence)

## How It Differs from Regular Data Analysis

In ordinary tabular analysis, rows are independent observations. In time series, **order matters**: each observation depends on its history. This breaks standard ML assumptions (i.i.d.) and requires special techniques.

Key structural differences:
- Temporal ordering is non-negotiable
- Past values predict future values (autocorrelation)
- Patterns repeat at known intervals (seasonality)
- Data may drift over time (non-stationarity)

## Four Core Tasks

| Task | What It Answers |
|------|----------------|
| **Description** | What patterns exist in this data? |
| **Decomposition** | What are the trend, seasonal, and noise components? |
| **Anomaly Detection** | Which observations deviate from expected behavior? |
| **Forecasting** | What will happen next? |

Most practical projects use all four.

## Structural Components

Every time series can be decomposed into three components:

- **Trend**: Long-term direction (upward, downward, flat)
- **Seasonality**: Repeating patterns tied to calendar cycles (hourly, daily, weekly, yearly)
- **Residual / Noise**: Random variation after trend and seasonality are removed

Additive model: `Y(t) = T(t) + S(t) + R(t)`
Multiplicative model: `Y(t) = T(t) × S(t) × R(t)`

Use additive when variation is constant across time; multiplicative when variation grows with the level.

## Why This Is Central to Data Science

Time series data is everywhere: server logs, sensor readings, financial prices, sales transactions, patient vitals. A data scientist who cannot handle temporal structure is limited to a small subset of real problems.

Key skills it requires that overlap with general ML:
- Feature engineering (but time-aware)
- Model selection (but with temporal constraints)
- Cross-validation (but forward-only)
- Metric selection (but scale-aware)

## Topic Map

### Foundations
- [[Time Series Decomposition]] — trend, seasonality, residuals, STL, stationarity, ACF/PACF
- [[Time Series Real-World Applications]] — finance, retail, energy, healthcare, weather
- [[Time Series Data Cleaning]] — missing values, outliers, stationarity transforms, checklist

### Modeling
- [[Time Series Feature Engineering]] — lags, rolling stats, calendar features, leakage rules
- [[Time Series Classical Models]] — naive baselines, ETS, ARIMA, SARIMA, TBATS, Prophet
- [[Time Series ML and DL Models]] — XGBoost, LightGBM, LSTM, TFT, N-BEATS, foundation models
- [[Time Series Multivariate]] — VAR, global models, Granger causality, TFT
- [[Time Series Exogenous Variables]] — ARIMAX, future-known vs future-unknown covariates
- [[Time Series Multi-Step Forecasting]] — recursive vs direct vs MIMO, long-horizon strategies
- [[Time Series Intermittent Demand]] — Croston, SBA, Tweedie, zero-inflated models
- [[Time Series Hierarchical]] — bottom-up, top-down, MinT reconciliation
- [[Time Series Anomaly Detection]] — STL residuals, Isolation Forest, LSTM autoencoder

### Evaluation and Uncertainty
- [[Time Series Evaluation Metrics]] — MAE, RMSE, MAPE, sMAPE, MASE, walk-forward CV
- [[Time Series Probabilistic Forecasting]] — prediction intervals, quantile regression, conformal prediction

### Competition and Production
- [[Time Series AutoML]] — AutoGluon-TS, StatsForecast, Optuna, foundation models
- [[Time Series Advanced Ensembling]] — weighted averaging, stacking, seed ensembles
- [[Time Series Hackathon Guide]] — 8-phase execution plan, feature recipes, time budget
- [[Time Series Production Use]] — decision windows, deployment, monitoring
