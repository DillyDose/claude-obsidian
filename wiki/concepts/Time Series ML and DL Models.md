---
title: "Time Series ML and DL Models"
tags:
  - data-science
  - time-series
  - models
  - machine-learning
  - deep-learning
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Classical Models]]"
  - "[[Time Series Feature Engineering]]"
  - "[[Time Series Evaluation Metrics]]"
  - "[[Time Series Hackathon Guide]]"
---

# Time Series ML and DL Models

## Two Approaches to ML for Time Series

**Approach A — Tabular ML (Recommended for most problems)**:
Convert the time series into a tabular dataset using lag features and rolling statistics, then apply standard ML algorithms (XGBoost, LightGBM). Fast, interpretable, competition-proven.

**Approach B — Sequence Models (Deep Learning)**:
Feed raw sequences into models that learn temporal structure natively (LSTM, Temporal CNN, Transformer). Higher complexity, more data required, harder to debug.

## Gradient Boosted Trees (XGBoost / LightGBM / CatBoost)

**The default choice for most time series tabular problems.**

Evidence from competitions:
- M5 competition (Kaggle): LightGBM dominated the top solutions. Winner used LightGBM with extensive lag and rolling features. (Source: M5 Results, ScienceDirect 2021 — high confidence)
- XGBoost consistently outperforms LSTM on typical financial time series in accuracy and training speed (100x faster). (Source: D&T Systems analysis — medium confidence)

**Why they work well**:
- Handle mixed feature types (lags, calendar, categorical) naturally
- No need for scaling
- Built-in feature importance for interpretation
- Robust to outliers
- Fast iteration cycle

**Key hyperparameters for time series**:
- `n_estimators`: 500–3000 with early stopping
- `learning_rate`: 0.01–0.05 with higher n_estimators
- `max_depth`: 4–8 (shallower than general ML to avoid overfitting)
- `objective`: `tweedie` for intermittent demand / many zeros; `huber` for robustness to outliers

**Limitation**: tree models cannot extrapolate beyond the range seen in training. A series with an upward trend that continues past the training max will be under-predicted. Solution: detrend before modeling.

## LSTM (Long Short-Term Memory)

A recurrent neural network designed to learn long-range dependencies in sequences. Each cell has gates that control what information to remember, forget, or output.

When it outperforms trees:
- Very long sequences with long-range dependencies
- Multivariate inputs with complex interactions
- When raw sequences are the input (no feature engineering)

When it underperforms trees:
- Short series or small datasets (trees generalize better)
- Stationary or near-stationary data
- When XGBoost + good features already achieves target performance (no reason to pay the complexity cost)

Input shape: `(samples, timesteps, features)`. Requires scaling.

## Temporal Convolutional Networks (TCN)

CNN adapted for sequences using causal dilated convolutions. Faster to train than LSTM, can capture very long dependencies with dilated kernels. Often matches LSTM performance with simpler architecture.

Use when: sequence modeling is needed, but LSTM training is too slow.

## Transformer-Based Models

Attention mechanisms model relationships between any two time steps directly, regardless of distance. Originally designed for NLP, now adapted for time series.

Notable models:
| Model | Description |
|-------|-------------|
| **Temporal Fusion Transformer (TFT)** | Combines LSTM encoder with multi-head attention + static metadata. Top performer on M5, traffic, and electricity datasets. |
| **N-BEATS** | Pure neural network with no recurrence. Double-residual stacking. No feature engineering required. |
| **N-HiTS** | Extends N-BEATS with multi-rate sampling for long-horizon forecasting. |
| **PatchTST** | Treats time series as patches (like image patches in ViT). Strong on long-sequence benchmarks. |
| **TiDE** | Simple MLP-based encoder-decoder. Competitive with Transformers at much lower compute. |

> [!gap] Research in 2024 found that simple linear models sometimes match or outperform Transformers on benchmarks. Transformer superiority is dataset-dependent, not universal. (Source: multiple 2024 papers — medium confidence)

## Foundation Models for Time Series

Large pre-trained models for zero-shot or few-shot time series forecasting:

| Model | Source | Notes |
|-------|--------|-------|
| **TimeGPT** | Nixtla | Commercial API. Zero-shot forecasting. |
| **Lag-Llama** | Open source | Probabilistic, zero-shot. |
| **MOIRAI** | Salesforce | Multi-domain pre-training. |
| **Chronos** | Amazon | Pre-trained on large corpus of public time series. |

These are still maturing (2024-2025). Use them to create baselines, not as first-choice production models unless the series is short or lacks training data.

## Ensemble and Hybrid Models

Competition winners consistently use ensembles. Common patterns:
- Weighted average of XGBoost + LightGBM + ARIMA
- XGBoost (51%) + LSTM (28%) + statistical model (21%) ensemble outperforms all individual components (Source: ScienceDirect 2025 ensemble study — medium confidence)
- Stacking: train a meta-model on out-of-fold predictions from base models

## Model Selection Decision Tree

```
How many series do you have?
├── 1 series:
│   ├── Short (<100 points) → Classical (ETS, ARIMA)
│   └── Long (>100 points) → Try ETS baseline, then LightGBM with features
└── Many series (100+):
    ├── Enough history per series → LightGBM global model
    ├── Short history per series → Foundation model (TimeGPT, Chronos)
    └── Complex multivariate → TFT or N-BEATS

Does extrapolation matter (trend going beyond training range)?
└── Yes → Detrend first, or use Prophet / ETS (they extrapolate trend)
```

## Libraries

| Library | Models | Notes |
|---------|--------|-------|
| `statsmodels` | ARIMA, ETS, VARMA | Classical models, production-ready |
| `pmdarima` | auto_arima | Automated ARIMA selection |
| `prophet` | Prophet | Meta's library |
| `skforecast` | XGBoost, LightGBM | Scikit-learn interface for time series |
| `neuralforecast` (Nixtla) | N-BEATS, N-HiTS, TFT, NHITS, TiDE | Neural models |
| `statsforecast` (Nixtla) | ETS, ARIMA, CES | Fast statistical models |
| `darts` | Everything | Unified API, good for experimentation |
| `autogluon-ts` | AutoML | Best for quick competition baselines |
