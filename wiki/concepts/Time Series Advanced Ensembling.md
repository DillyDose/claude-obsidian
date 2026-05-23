---
title: "Time Series Advanced Ensembling"
tags:
  - data-science
  - time-series
  - ensembling
  - competition
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series ML and DL Models]]"
  - "[[Time Series Hackathon Guide]]"
  - "[[Time Series Evaluation Metrics]]"
---

# Time Series Advanced Ensembling

## Why Ensembling Works

No single model dominates across all time series tasks. Different models capture different aspects of the data:
- Statistical models (ETS, ARIMA) capture parametric trends and seasonality cleanly
- Tree-based models (LightGBM) capture non-linear feature interactions
- Neural models (LSTM, N-BEATS) capture complex temporal patterns
- Naive baselines are surprisingly hard to beat on noisy series

An ensemble combines their individual errors in a way that tends to cancel out. Empirically, ensembles consistently dominate Kaggle and M-competition leaderboards. (Source: Learnings from Kaggle Forecasting Competitions, arXiv 2009.07701 — high confidence)

---

## Level 0: Simple Averaging

The fastest ensemble. Take the unweighted average of predictions from multiple models.

```python
ensemble = (lgb_preds + xgb_preds + ets_preds) / 3
```

**Surprisingly strong** — simple averaging often beats individually tuned models. Use this as your first ensemble before trying anything fancier.

---

## Level 1: Weighted Averaging

Assign different weights to models based on their validation score.

```python
# Weights proportional to inverse validation error
mae_lgb = 100
mae_xgb = 110
mae_ets = 130

total_inv = 1/mae_lgb + 1/mae_xgb + 1/mae_ets
w_lgb = (1/mae_lgb) / total_inv   # ~0.42
w_xgb = (1/mae_xgb) / total_inv   # ~0.38
w_ets = (1/mae_ets) / total_inv   # ~0.20

ensemble = w_lgb * lgb_preds + w_xgb * xgb_preds + w_ets * ets_preds
```

Better models get more weight. A small number of trials with Optuna can optimize weights further:

```python
import optuna

def objective(trial):
    w1 = trial.suggest_float('w1', 0, 1)
    w2 = trial.suggest_float('w2', 0, 1)
    w3 = 1 - w1 - w2
    if w3 < 0: return float('inf')
    preds = w1*lgb_preds + w2*xgb_preds + w3*ets_preds
    return mae(y_val, preds)

study = optuna.create_study(direction='minimize')
study.optimize(objective, n_trials=200)
```

---

## Level 2: Stacking (Meta-Learning)

Train a meta-model on out-of-fold predictions from base models. The meta-model learns which base model to trust in which conditions.

```
Step 1: Train base models with walk-forward CV.
        Each model makes out-of-fold predictions on the validation set.

Step 2: Stack OOF predictions as features:
        meta_features = [lgb_oof, xgb_oof, ets_oof, arima_oof]

Step 3: Train a meta-model on meta_features → y_true.
        Use a simple model: Ridge regression, or LightGBM.

Step 4: At test time, generate base predictions, then pass to meta-model.
```

```python
from sklearn.linear_model import RidgeCV

# meta_X: shape (n_val_samples, n_base_models)
meta_X = np.column_stack([lgb_oof, xgb_oof, ets_oof])
meta_model = RidgeCV(alphas=[0.01, 0.1, 1.0, 10.0])
meta_model.fit(meta_X, y_val)

# Test-time: generate base test preds, pass to meta
meta_X_test = np.column_stack([lgb_test, xgb_test, ets_test])
ensemble_test = meta_model.predict(meta_X_test)
```

**Leakage warning**: base model OOF predictions must be generated using walk-forward CV, not random splits. The meta-model trains on validation-set predictions — these must be from a held-out time window, not seen during base model training.

---

## Diversity Is More Important Than Individual Accuracy

Two good models that are highly correlated add little to each other. Two models with different weaknesses complement each other.

Strategies to maximize diversity:
- **Different algorithm families**: LightGBM (trees) + LSTM (sequence) + ETS (statistical)
- **Different feature sets**: model A uses lag features; model B uses only calendar + rolling stats
- **Different training windows**: model A trained on 2 years; model B on 6 months (captures recent regime)
- **Different targets**: one model predicts log-transformed target; another predicts levels — average predictions after inverse-transforming
- **Different random seeds**: train 5 LightGBM models with different seeds, average their predictions (reduces variance by ~30%)

```python
# Seed ensemble — fast and effective
preds_list = []
for seed in [42, 123, 456, 789, 1024]:
    model = lgb.LGBMRegressor(**params, random_state=seed)
    model.fit(X_train, y_train)
    preds_list.append(model.predict(X_test))

seed_ensemble = np.mean(preds_list, axis=0)
```

---

## Post-Processing Before Ensembling

Ensure all models produce comparable outputs before averaging:
1. Apply the same post-processing to each base model (floor at zero, round integers)
2. If models were trained on different transformations (log vs. raw), invert before ensembling
3. Check for outlier predictions from individual models — clip before ensembling to prevent one broken model from ruining the ensemble

---

## Time Budget for Ensembling in Hackathons

| Time Available | Strategy |
|----------------|---------|
| Last 30 minutes | Seed ensemble of your best model (5 seeds, average) |
| Last 2 hours | Weighted average of LightGBM + ETS/ARIMA |
| Last 4 hours | Stacking: add AutoGluon-TS as a base model, Ridge meta-model |
| Full time planned | Full stacking pipeline with Optuna weight optimization |

**Always save time for ensembling** — it is the single highest ROI activity in the final phase of a hackathon. A 3-5% score improvement is typical.

---

## What the Best Kaggle Solutions Look Like

Based on M5 and other forecasting competition analyses:
- Winner: LightGBM-heavy ensemble (weights ~50% LightGBM, ~28% LSTM, ~22% statistical)
- Top 1%: 3-5 diverse models, stacked with Ridge or LightGBM meta-model
- Top 5%: 2-3 models, weighted average with Optuna-tuned weights
- Top 20%: LightGBM alone with good features + seed ensemble

The gap from top 20% to top 5% is usually ensembling + better cross-validation + fine-grained post-processing, not a fundamentally different model.
