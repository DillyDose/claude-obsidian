---
title: "Time Series Hackathon Guide"
tags:
  - data-science
  - time-series
  - hackathon
  - competition
  - business-discovery
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Feature Engineering]]"
  - "[[Time Series ML and DL Models]]"
  - "[[Time Series Evaluation Metrics]]"
  - "[[Time Series Data Cleaning]]"
---

# Time Series Hackathon Guide

## Core Principle

Hackathon wins come from disciplined execution of the fundamentals, not from exotic models. The M5 competition (one of the most rigorous time series competitions ever run) was won with LightGBM + well-engineered lag and rolling features. No attention layers, no transformers. (Source: M5 Accuracy Results, ScienceDirect 2021 — high confidence)

---

## Phase 0: Client Discovery (When a Problem Owner Presents the Brief)

This phase applies to client-facing hackathons where a business stakeholder presents the challenge. The goal is to understand the business context before touching data. Budget 10-15 minutes of focused questions.

### 4 Lenses for Any Time Series Problem

| Lens | Question | Methods |
|------|----------|---------|
| **Predict** | What will happen next? | ARIMA, XGBoost + lag features |
| **Cause** | Why did it happen? | Granger causality, Transfer Entropy |
| **Decompose** | How is the signal built? | STL, MSTL, X11 |
| **Coordinate** | Who leads whom? | Cross-correlation, dependency mapping |

Identify which lens the client actually needs before modeling anything.

### Discovery Questions: 5 Phases

**Context (before seeing any data):**
- What is the problem and how expensive is it if it stays unsolved?
- How do you solve this today? (establishes the baseline to beat)
- Why solve it now? (reveals urgency and constraints)

**Understanding the signal:**
- What is the target variable, how is it measured, and what are its units?
- What is the data frequency? (hourly, daily, weekly, monthly)
- Are there known seasonality patterns? (weekly cycles, annual, event-driven)
- Are there known outliers or special events? (holidays, shutdowns, promotions)

**Business cost and success:**
- Which error is worse: over-estimate or under-estimate? (these have different costs)
- What does a good solution look like in your eyes? How will you measure success?
- Does the model need to run in real-time or is batch prediction acceptable?

**Data and dependencies:**
- How many years of history are available? Are there gaps or missing periods?
- Is there external data available? (weather, promotions, macro indicators)
- If there are multiple entities (factories, stores, regions), do they influence each other?
- Who generates this data and has the collection process changed over time?

**Constraints and deployment:**
- How long must this solution be maintained and who will maintain it?
- Are there technology or infrastructure constraints?
- How often can the model be retrained?

### Rules Before Committing to Anything

- Never commit to a solution before seeing the data. "Let me look at the data structure for 10 minutes before I confirm scope."
- If residuals still show a pattern, the model is not capturing all trend and seasonality.
- Ask about cross-entity dependencies before modeling multiple series together (factories can affect each other).
- Decompose big problems into smaller ones. Start with: "What dependencies exist in this data?"

---

## Business Cost Framing

Decision makers think in risk, not in accuracy metrics. Translate model performance into business language.

**The key shift:** Confusion matrix entries have a currency value (baht, dollars, etc.). A model that is "90% accurate" means nothing. What matters is: "A false positive costs X baht, a false negative costs Y baht."

**Questions that matter to decision makers:**
- What is the cost of being wrong in each direction?
- Can we measure the risk of deploying this model?
- Is this maintainable long-term, or is it a one-shot solution?

**Presenting results:**
- Always explain every feature used. Explainability builds client trust.
- If using a complex method, justify why a simpler one would not work (avoid over-engineering).
- Metric values must be compared against the scale of the data. A small MAE on a low-range series may still represent a large relative error.
- "Accuracy" has a specific definition. Confirm that the term the client uses maps to a measurable quantity.

---

## Phase 1: Understand the Problem (First 30 Minutes)

Before touching data:
1. **Read the metric** — is it MAE, RMSE, WRMSSE, MAPE? This determines your loss function and post-processing.
2. **Understand the forecast horizon** — 1-step or multi-step? Direct forecasting or recursive?
3. **Count the series** — 1 series or thousands? This drives model architecture.
4. **Identify the frequency** — hourly, daily, weekly, monthly? Determines which lag periods and seasonal periods to use.
5. **Check for hierarchical structure** — total = sum of parts? Reconciliation may be required.
6. **Understand what data is provided** — target only? External regressors? Static metadata (store type, region)?

---

## Phase 2: EDA Checklist (First 1-2 Hours)

```python
# 1. Plot the series
df['target'].plot()

# 2. Seasonal decomposition
from statsmodels.tsa.seasonal import STL
STL(df['target'], period=7).fit().plot()

# 3. ACF/PACF
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
plot_acf(df['target'], lags=50)
plot_pacf(df['target'], lags=50)

# 4. Check for missing values
df.isnull().sum()

# 5. Check for outliers
df['target'].describe()
df.boxplot(column='target')

# 6. Check stationarity
from statsmodels.tsa.stattools import adfuller
adfuller(df['target'].dropna())
```

Questions to answer from EDA:
- Is there a clear trend? Is it linear or exponential?
- What seasonality periods are present?
- Are there outliers or anomalies? What caused them?
- Are there structural breaks (regime changes)?
- How many zeros or near-zeros? (affects metric choice and model objective)

---

## Phase 3: Build a Strong Baseline (Hours 2-4)

Always build these in order — each is a checkpoint:

1. **Seasonal naive**: `ŷ(t) = y(t - m)` — takes 5 minutes to code
2. **ETS / Holt-Winters**: handles trend + seasonality
3. **LightGBM with basic features**: the model that usually wins

A good baseline tells you how hard the problem is and gives you a score to improve from.

---

## Phase 4: Feature Engineering (The Highest ROI Activity)

This is where competitions are won. Spend 40-50% of total hackathon time here.

**Must-have features for tabular models:**

```python
# Lags — use ACF to select which ones matter
df['lag_1']   = df['target'].shift(1)
df['lag_7']   = df['target'].shift(7)
df['lag_14']  = df['target'].shift(14)
df['lag_28']  = df['target'].shift(28)

# Rolling means — capture recent trend
df['roll_7']  = df['target'].shift(1).rolling(7).mean()
df['roll_14'] = df['target'].shift(1).rolling(14).mean()
df['roll_28'] = df['target'].shift(1).rolling(28).mean()

# Rolling std — capture recent volatility
df['rstd_7']  = df['target'].shift(1).rolling(7).std()

# Calendar features with cyclical encoding
df['dow_sin'] = np.sin(2 * np.pi * df.index.dayofweek / 7)
df['dow_cos'] = np.cos(2 * np.pi * df.index.dayofweek / 7)
df['month_sin'] = np.sin(2 * np.pi * df.index.month / 12)
df['month_cos'] = np.cos(2 * np.pi * df.index.month / 12)

# Holiday flags
import holidays
holiday_list = holidays.country_holidays('US', years=range(2020, 2027))
df['is_holiday'] = df.index.isin(holiday_list).astype(int)
```

**The `.shift(1)` before rolling is mandatory** — it prevents leaking the current observation into its own features.

---

## Phase 5: Cross-Validation

Use walk-forward validation, not KFold.

```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5)
for fold, (train_idx, val_idx) in enumerate(tscv.split(X)):
    X_train, X_val = X.iloc[train_idx], X.iloc[val_idx]
    y_train, y_val = y.iloc[train_idx], y.iloc[val_idx]
    # train and score
```

A validation score from KFold on time series is meaningless — it will be over-optimistic because the model sees future data during training.

---

## Phase 6: Model Training Tips

**LightGBM for time series:**
```python
import lightgbm as lgb

params = {
    'objective': 'regression_l1',  # MAE objective
    # use 'tweedie' if many zeros, 'huber' if outliers
    'n_estimators': 2000,
    'learning_rate': 0.02,
    'num_leaves': 64,
    'max_depth': 6,
    'subsample': 0.8,
    'colsample_bytree': 0.8,
    'min_child_samples': 20,
}

model = lgb.LGBMRegressor(**params)
model.fit(X_train, y_train,
          eval_set=[(X_val, y_val)],
          callbacks=[lgb.early_stopping(100), lgb.log_evaluation(100)])
```

Match `objective` to the competition metric:
- Competition uses MAE → `regression_l1`
- Competition uses RMSE → `regression_l2`
- Target has many zeros → `tweedie`

---

## Phase 7: Post-Processing

- **Floor at zero** if predictions cannot be negative: `preds = np.maximum(preds, 0)`
- **Round to integers** if the target is count data: `preds = np.round(preds).astype(int)`
- **Clip to historical range** if tree models extrapolate poorly

---

## Phase 8: Ensembling (If Time Allows)

Simple averaging of 3-5 diverse models beats any single model:

```python
ensemble = (lgb_preds * 0.4 + xgb_preds * 0.3 + ets_preds * 0.2 + arima_preds * 0.1)
```

Diversity is more important than individual model quality. Mix tree-based, statistical, and if available, neural models.

---

## Common Mistakes That Kill Scores

| Mistake | Consequence | Fix |
|---------|-------------|-----|
| Rolling features without `.shift(1)` | Data leakage — validation score is fake | Always shift before rolling |
| Scaling before train/test split | Data leakage from future stats | Fit scaler on train only |
| KFold cross-validation | Falsely optimistic CV score | Use TimeSeriesSplit |
| No baseline | No context for whether model is good | Always build seasonal naive first |
| Predicting negatives on count data | MAE penalty for impossible values | Clip at zero |
| Ignoring holidays | Systematic errors on specific dates | Add holiday features |
| Only one model | Leaves ensemble gains on the table | Build 2-3 diverse models |

---

## Quick Reference: Time Budget for a 24-Hour Hackathon

| Activity | Time |
|----------|------|
| Problem + data understanding | 1 hour |
| EDA and cleaning | 2 hours |
| Baseline model | 1 hour |
| Feature engineering | 8 hours |
| Cross-validation and tuning | 4 hours |
| Ensembling + post-processing | 3 hours |
| Buffer / presentation prep | 5 hours |

Feature engineering gets the most time because it has the highest return on investment.
