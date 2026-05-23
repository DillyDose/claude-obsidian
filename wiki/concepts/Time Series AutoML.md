---
title: "Time Series AutoML"
tags:
  - data-science
  - time-series
  - automl
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series ML and DL Models]]"
  - "[[Time Series Classical Models]]"
  - "[[Time Series Hackathon Guide]]"
---

# Time Series AutoML

## What It Is

AutoML for time series automates model selection, hyperparameter tuning, and ensembling. It reduces the time to a strong baseline from days to minutes. In a hackathon, AutoML is best used to generate a baseline quickly, then beaten with targeted feature engineering and tuning.

---

## Key Libraries

### AutoGluon-TimeSeries

The most comprehensive open-source AutoML framework for time series. Fits and ensembles:
- Statistical models (ETS, ARIMA, Theta via StatsForecast)
- Tree-based models (LightGBM with engineered features)
- Deep learning models (DeepAR, TFT, PatchTST)
- Zero-shot foundation model (Chronos)

Then combines them via a weighted ensemble selected by forward model selection.

```python
from autogluon.timeseries import TimeSeriesDataFrame, TimeSeriesPredictor

train_data = TimeSeriesDataFrame.from_data_frame(df,
    id_column='series_id', timestamp_column='ds')

predictor = TimeSeriesPredictor(
    prediction_length=28,
    target='y',
    eval_metric='MASE',
    freq='D'
)
predictor.fit(train_data, time_limit=3600)  # 1 hour budget
predictions = predictor.predict(train_data)
```

**Best for**: getting a strong baseline fast. Often top 20% in competitions with no customization. (Source: AutoGluon documentation, Towards Data Science review — high confidence)

**Limitation**: the ensemble is a black box. Feature importance and model interpretability are limited.

### Nixtla StatsForecast

Lightning-fast statistical model selection. Fits AutoARIMA, AutoETS, AutoCES, AutoTheta, AutoTBATS — selects the best per series by AIC. Parallelized across series.

```python
from statsforecast import StatsForecast
from statsforecast.models import AutoARIMA, AutoETS, AutoTheta

models = [AutoARIMA(season_length=7),
          AutoETS(season_length=7),
          AutoTheta(season_length=7)]

sf = StatsForecast(models=models, freq='D', n_jobs=-1)
sf.fit(df)  # df: columns ['unique_id', 'ds', 'y']
forecasts = sf.predict(h=28)
```

**Best for**: large-scale univariate forecasting (thousands of series). Extremely fast — can process millions of series in minutes on a multi-core machine.

**Benchmarks**: StatsForecast AutoARIMA is 20x faster than pmdarima auto_arima. (Source: Nixtla benchmarks — medium confidence)

### pmdarima auto_arima

The original Python implementation of automated ARIMA selection. Tests combinations of (p, d, q)(P, D, Q, m) by AIC/BIC. Slow on large datasets but well-tested and widely used.

```python
from pmdarima import auto_arima

model = auto_arima(y_train,
                   seasonal=True, m=7,
                   stepwise=True,      # faster search
                   information_criterion='aic',
                   error_action='ignore',
                   suppress_warnings=True)
model.summary()
forecast = model.predict(n_periods=28)
```

### Prophet (with auto-tuning)

Prophet does automatic changepoint detection, seasonality decomposition, and holiday effects with minimal configuration. Add `add_regressor()` calls for exogenous variables. Prophet's hyperparameters (changepoint_prior_scale, seasonality_prior_scale) can be tuned via cross-validation.

```python
from prophet import Prophet
from prophet.diagnostics import cross_validation, performance_metrics

model = Prophet()
model.fit(df_train)

df_cv = cross_validation(model, horizon='28 days',
                         period='14 days', initial='365 days')
metrics = performance_metrics(df_cv)
```

---

## Automated Hyperparameter Tuning

### Optuna

General-purpose hyperparameter optimization library. Use it to tune LightGBM parameters over the time-series cross-validation score.

```python
import optuna

def objective(trial):
    params = {
        'n_estimators': trial.suggest_int('n_estimators', 500, 3000),
        'learning_rate': trial.suggest_float('lr', 0.01, 0.1, log=True),
        'num_leaves': trial.suggest_int('num_leaves', 31, 256),
        'max_depth': trial.suggest_int('max_depth', 4, 10),
        'subsample': trial.suggest_float('subsample', 0.5, 1.0),
    }
    model = lgb.LGBMRegressor(**params)
    score = time_series_cv_score(model, X, y)  # your CV function
    return score

study = optuna.create_study(direction='minimize')
study.optimize(objective, n_trials=100)
best_params = study.best_params
```

### Ray Tune

Distributed hyperparameter tuning. Use when you need to search a large parameter space quickly across multiple CPUs/GPUs.

---

## Foundation Models (Zero-Shot)

These models are pre-trained on large corpora of time series and can forecast without fine-tuning.

| Model | Source | Notes |
|-------|--------|-------|
| **Chronos** | Amazon | Pre-trained on 100k+ series. Available via AutoGluon. |
| **TimeGPT-2** | Nixtla | MASE 0.43 on benchmarks, 72x faster than StatsForecast ensemble (Source: Nixtla benchmarks — medium confidence) |
| **Lag-Llama** | Open source | Probabilistic, distributional output |
| **MOIRAI** | Salesforce | Multi-domain zero-shot |

**Use case in hackathons**: run a foundation model as one member of an ensemble. It provides a strong signal for series with short history (few data points to train on).

**Limitation**: foundation models can underperform when the target series has unique patterns not represented in pre-training data.

---

## AutoML Strategy for Hackathons

**Hour 1**: run AutoGluon-TS with a 30-minute time limit. Submit this as the baseline.

**Hours 2-6**: engineer features manually, train LightGBM with Optuna tuning. This usually beats AutoGluon.

**Hours 7-8**: ensemble AutoGluon output + your tuned LightGBM. This is almost always better than either alone.

The value of AutoML in competitions is not as the final answer — it is as a strong, fast baseline that tells you the ceiling to beat and as an ensemble member.
