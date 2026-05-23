---
title: "Time Series Data Cleaning"
tags:
  - data-science
  - time-series
  - preprocessing
  - data-cleaning
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Decomposition]]"
  - "[[Time Series Feature Engineering]]"
---

# Time Series Data Cleaning

## Why Time Series Cleaning Is Different

Standard tabular cleaning treats rows as independent. Time series cleaning must preserve **temporal order** and **continuity**. Dropping a row breaks the sequence. Filling a value incorrectly propagates error into lag features and rolling statistics.

## Step 0: Validate the Time Index

Before anything else:
- Check for duplicate timestamps — keep one, discard others
- Check for gaps (missing time steps) — decide if they are true zeros, true missing, or calendar effects (weekends, holidays)
- Ensure uniform frequency — resample if needed (pandas `resample()`)
- Sort by timestamp ascending

## Common Cleaning Methods

### Missing Values

| Situation | Method |
|-----------|--------|
| Single isolated gap | Linear or cubic spline interpolation |
| Long run of missing values | Forward fill (last known value) or seasonal fill (same period last cycle) |
| Missing due to system outage | Mark as a feature flag, then interpolate |
| Structural absence (e.g., no sales Sunday) | Fill with 0, not NaN |

**Key rule**: only use information available at that point in time. Do not interpolate using future values in training-time cleaning — it leaks future data.

### Outliers

Detection methods:
- **IQR method**: values beyond `Q1 − 1.5×IQR` or `Q3 + 1.5×IQR` are outliers
- **Z-score / modified Z-score**: for approximately normal distributions
- **Grubbs test**: formal statistical test for single outlier in normal distribution
- **STL residual threshold**: decompose first, then threshold the residual component — this correctly distinguishes outliers from legitimate seasonal peaks

Treatment:
- Replace with interpolated value (cubic spline for single points)
- Winsorize: cap at the 1st/99th percentile
- Add a binary indicator column (`is_outlier`) and impute — the model can learn the outlier effect separately
- **Do not blindly delete** outliers in time series — deletion creates gaps

### Non-Stationarity

Most ML models work better on stationary or near-stationary series.

| Transform | Removes | Code |
|-----------|---------|------|
| First differencing | Linear trend | `df.diff()` |
| Log transform | Exponential trend, stabilizes variance | `np.log(df)` |
| Box-Cox transform | General variance instability | `scipy.stats.boxcox()` |
| Seasonal differencing | Seasonal pattern | `df.diff(period)` |
| Percent change | Scale differences between series | `df.pct_change()` |

Always run ADF and KPSS tests before and after to verify stationarity was achieved.

### Resampling and Frequency Alignment

- **Downsampling** (e.g., minutes to hours): aggregate with mean, sum, or max depending on domain
- **Upsampling** (e.g., monthly to daily): interpolate or forward-fill
- **Aligning multiple series**: if you have exogenous variables at different frequencies, resample all to a common frequency before merging

### Scaling and Normalization

Needed for neural network models and distance-based algorithms; not required for tree-based models.

- **StandardScaler** (zero mean, unit variance): use for LSTM, Transformer inputs
- **MinMaxScaler** (0 to 1 range): use if bounded output is expected
- **Per-series normalization**: normalize each individual time series separately — critical when modeling many series at once (retail SKUs, store locations)

**Important**: fit the scaler on training data only. Apply to validation/test. Never fit on the full dataset — that leaks future statistics.

## Advanced Cleaning Techniques

### Robust STL Denoising
Decompose with STL (robust=True), then reconstruct `T + S` while discarding the residual. This is aggressive — use only when the residual is pure noise and not signal.

### Kalman Smoothing
State-space filtering that optimally smooths a series given a noise model. Used in finance and sensor data. More principled than moving-average smoothing.

### Changepoint Detection
Before modeling, detect structural breaks in the series (e.g., a policy change, a new competitor). Treat pre/post-changepoint data as separate. Libraries: `ruptures` (Python), Prophet has built-in changepoint detection.

### Anomaly Detection as a Cleaning Step
Run an anomaly detector (Isolation Forest, LSTM autoencoder) first, flag anomalies, then impute. This is the production-grade approach. (Source: Perforce/IMSL blog — medium confidence)

## Checklist Before Modeling

- [ ] Timestamps sorted, no duplicates, correct frequency
- [ ] Missing values handled — no NaN in target column
- [ ] Outliers reviewed — replaced or flagged
- [ ] Stationarity checked (ADF + KPSS)
- [ ] Series length sufficient for the model (ARIMA needs ~50+ points minimum)
- [ ] Train/val/test split is time-ordered (no shuffle)
- [ ] No future data leaked into past-time-step features
