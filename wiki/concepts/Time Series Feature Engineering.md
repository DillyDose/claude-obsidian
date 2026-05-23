---
title: "Time Series Feature Engineering"
tags:
  - data-science
  - time-series
  - feature-engineering
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Data Cleaning]]"
  - "[[Time Series ML and DL Models]]"
  - "[[Time Series Hackathon Guide]]"
---

# Time Series Feature Engineering

## Why It Matters

Tree-based models (XGBoost, LightGBM) have no built-in notion of time. They see only a tabular row. Feature engineering converts temporal structure into columns the model can use. Good features are more important than model choice for these algorithms.

Deep learning models (LSTM, Transformer) can learn some temporal features automatically, but explicit lag features still help them converge faster and generalize better.

## Category 1: Lag Features

A lag feature is the value of the target (or a related series) at a previous time step.

```python
df['lag_1'] = df['target'].shift(1)   # yesterday's value
df['lag_7'] = df['target'].shift(7)   # same day last week
df['lag_28'] = df['target'].shift(28) # same day last month
df['lag_365'] = df['target'].shift(365) # same day last year
```

**How to choose lags**: examine the ACF plot. Lags where ACF is significantly non-zero are informative. For weekly seasonality, include lag 7. For yearly, include lag 365.

**Danger**: lags introduce NaN at the start of the series. These rows must be dropped or imputed before training.

## Category 2: Rolling Statistics (Window Features)

Rolling statistics summarize recent history over a sliding window.

```python
df['roll_mean_7'] = df['target'].rolling(7).mean()
df['roll_std_7']  = df['target'].rolling(7).std()
df['roll_max_28'] = df['target'].rolling(28).max()
df['roll_min_28'] = df['target'].rolling(28).min()
df['roll_median_7'] = df['target'].rolling(7).median()
```

Common windows: 3, 7, 14, 28, 84 days. Use multiple windows to capture both short-term and long-term patterns.

**Exponential Weighted Mean (EWM)**: gives more weight to recent observations. Better than simple rolling mean when the series changes quickly.

```python
df['ewm_7'] = df['target'].ewm(span=7).mean()
```

## Category 3: Calendar / Datetime Features

Extract temporal attributes from the timestamp.

```python
df['hour']        = df.index.hour
df['day_of_week'] = df.index.dayofweek   # 0=Monday
df['day_of_month']= df.index.day
df['week_of_year']= df.index.isocalendar().week
df['month']       = df.index.month
df['quarter']     = df.index.quarter
df['year']        = df.index.year
df['is_weekend']  = (df.index.dayofweek >= 5).astype(int)
df['is_month_end']= df.index.is_month_end.astype(int)
```

**Cyclical encoding**: day_of_week encoded as 0-6 makes Monday and Sunday appear far apart, but they are one day apart in a cycle. Use sine/cosine encoding instead:

```python
df['dow_sin'] = np.sin(2 * np.pi * df['day_of_week'] / 7)
df['dow_cos'] = np.cos(2 * np.pi * df['day_of_week'] / 7)
```

Apply the same pattern to hour (period=24), month (period=12), etc.

## Category 4: Holiday and Event Features

- Binary flag for public holidays (`is_holiday`)
- Days until/since a holiday (`days_to_holiday`)
- Promotional event flags (if retail)
- Weather events (if energy or agriculture)

These require external data (holiday calendars, event logs). Libraries: `holidays` (Python), `workalendar`.

## Category 5: Trend and Decomposition Features

Extract the trend and seasonality components from STL decomposition and use them as features:

```python
from statsmodels.tsa.seasonal import STL
stl = STL(df['target'], period=7)
result = stl.fit()
df['stl_trend']    = result.trend
df['stl_seasonal'] = result.seasonal
df['stl_residual'] = result.resid
```

## Category 6: Statistical Features (Tsfresh / Catch22)

Libraries like `tsfresh` and `catch22` extract 100+ time series statistics (entropy, autocorrelation, complexity) automatically. Useful for classification tasks. Too slow for large datasets without feature selection.

## Category 7: Target-Based Statistics (Careful with Leakage)

Aggregated statistics of the target by group (store, product, region):

```python
# Mean sales for this store, computed over TRAINING data only
df['store_mean'] = df.groupby('store_id')['target'].transform('mean')
```

**Leakage risk**: these must be computed using only training data and then applied as a lookup to validation/test. Never compute them on the full dataset before splitting.

## Combining Features

A strong feature set combines multiple categories:
- Lag features from ACF-identified lags
- Rolling means at 7, 14, 28 windows
- Cyclical calendar features
- Holiday flags
- STL trend component

The M5 competition winners (LightGBM) used rolling statistics over windows of 7, 14, 28, and 84 days plus lag features of 1, 7, 14, and 28 days as their core feature set. (Source: M5 competition analysis — high confidence)

## Data Leakage: The Most Common Mistake

**Rule**: when computing a feature for row at time `t`, only use data from time `t-1` and earlier.

Violations that cause leakage:
- Rolling mean computed without `.shift(1)` first
- Scaling the whole series before splitting train/test
- Target encoding computed on full dataset

Leakage gives falsely optimistic validation scores that fail in production. Always verify with a time-ordered cross-validation.
