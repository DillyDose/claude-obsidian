---
title: "Time Series Multi-Step Forecasting"
tags:
  - data-science
  - time-series
  - multi-step
  - long-horizon
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series ML and DL Models]]"
  - "[[Time Series Classical Models]]"
  - "[[Time Series Hackathon Guide]]"
---

# Time Series Multi-Step Forecasting

## The Core Problem

Most real decisions require forecasts over a **horizon** (multiple steps ahead), not just the next single step. Forecasting 28 days ahead is fundamentally harder than forecasting 1 day ahead. The choice of strategy significantly affects accuracy.

---

## Four Forecasting Strategies

### 1. Recursive (Iterated One-Step)

Train one model for 1-step-ahead. At prediction time, use its own forecasts as inputs to produce the next step.

```
Train: y(t+1) = f(y(t), y(t-1), y(t-2), ...)
Predict: ŷ(t+1) = f(y(t), ...)
         ŷ(t+2) = f(ŷ(t+1), y(t), ...)    ← uses its own forecast
         ŷ(t+3) = f(ŷ(t+2), ŷ(t+1), ...) ← error accumulates
```

**Advantage**: only one model to train. Simple and fast.
**Disadvantage**: errors compound across steps. By step h, the forecast depends on h-1 previous forecasts, each with their own error. **Bias increases with horizon.**

Best for: short horizons (h ≤ 7), stationary series, classical models like ARIMA (which is natively recursive).

### 2. Direct (MIMO — Multi-Input Multi-Output)

Train a separate model for each horizon. Model `h` directly predicts the value h steps ahead from original features only.

```
Train h=1: ŷ(t+1) = f₁(y(t), y(t-1), ...)
Train h=7: ŷ(t+7) = f₇(y(t), y(t-1), ...)    ← no chaining
Train h=28: ŷ(t+28) = f₂₈(y(t), y(t-1), ...) ← no error propagation
```

**Advantage**: no error propagation. Each model is optimized for its specific horizon.
**Disadvantage**: `h` models to train and maintain. Forecasts at different horizons are **independent** — not forced to be consistent with each other. High variance when training data is limited.

Best for: long horizons where error propagation dominates, or when you need horizon-specific optimization.

### 3. Rectify Strategy (Best of Both)

Start with recursive forecasts (low variance), then apply a correction model trained to adjust the recursive bias at each horizon.

`ŷ_rectified(t+h) = ŷ_recursive(t+h) + bias_correction(h)`

Research shows rectify almost always performs as well as or better than the better of recursive and direct. (Source: Hyndman & Taieb reconciliation paper — high confidence)

Implemented in `skforecast` library.

### 4. Seq2Seq / MIMO (Multi-Output in One Model)

Neural network approaches (LSTM encoder-decoder, Transformer) output all h horizons simultaneously in a single forward pass. The model implicitly handles the bias-variance tradeoff.

```
Input: [y(t-L), ..., y(t)]  (L past steps)
Output: [ŷ(t+1), ..., ŷ(t+h)]  (h future steps)
```

N-BEATS, N-HiTS, and TFT all use this approach. It avoids both error propagation and the multiple-model problem.

---

## Direct Forecasting with LightGBM (Practical)

`skforecast` makes direct multi-step forecasting with LightGBM easy:

```python
from skforecast.ForecasterAutoreg import ForecasterAutoreg
from skforecast.model_selection import backtesting_forecaster
import lightgbm as lgb

# Recursive strategy (default)
forecaster = ForecasterAutoreg(
    regressor=lgb.LGBMRegressor(n_estimators=500),
    lags=28
)
forecaster.fit(y_train)
predictions = forecaster.predict(steps=28)

# Direct strategy
from skforecast.ForecasterAutoregDirect import ForecasterAutoregDirect
forecaster_direct = ForecasterAutoregDirect(
    regressor=lgb.LGBMRegressor(n_estimators=500),
    lags=28,
    steps=28  # trains 28 separate models
)
forecaster_direct.fit(y_train)
predictions = forecaster_direct.predict()
```

---

## Long-Horizon Specific Challenges

### Error Accumulation
The further into the future, the more uncertain. Uncertainty grows (at minimum) as `√h` for a random walk. Communicate this with widening prediction intervals.

### Concept Drift
The longer the horizon, the more likely the world changes before the forecast is realized. A 3-year forecast assumes current trends continue — rarely true.

### Feature Availability
At long horizons, many lag features become unavailable at prediction time. A lag-1 feature is available for any horizon, but a lag-30 feature is unavailable for the first 30 steps of a direct forecast.

**Solution**: use only lags that are available at the maximum forecast horizon. For a 28-day horizon, use lags 28, 35, 42, ... (lag values ≥ horizon).

### Trend Extrapolation
Tree-based models cannot extrapolate beyond the range seen in training. A series trending upward will be under-predicted at long horizons if the training data did not include such high values.

**Solutions**:
- Detrend the series, model the detrended series, add the trend back post-prediction
- Use Prophet (extrapolates trend explicitly) or N-HiTS (designed for long horizon)
- Include the trend component from STL as a feature

---

## Models Designed for Long Horizons

| Model | Horizon Strength | Notes |
|-------|-----------------|-------|
| **N-HiTS** | Excellent | Multi-rate sampling, long horizon specialist |
| **PatchTST** | Good | Patch-based Transformer, strong on 720-step benchmarks |
| **TiDE** | Good | Simple MLP, surprisingly competitive at long horizons |
| **Prophet** | Moderate | Good trend extrapolation, weak at complex patterns |
| **ARIMA** | Short only | Error compounds rapidly beyond 20-30 steps |

---

## Practical Recommendation

| Horizon | Recommended Approach |
|---------|---------------------|
| 1-7 steps | Recursive LightGBM or ARIMA |
| 7-28 steps | Direct LightGBM with skforecast, or N-BEATS |
| 28-90 steps | Direct LightGBM (detrended) + N-HiTS ensemble |
| 90+ steps | Prophet + N-HiTS + trend-aware features |

In hackathons, check the required forecast horizon in the problem statement immediately. If it's >28 steps, switch to direct strategy or a long-horizon model from the start.
