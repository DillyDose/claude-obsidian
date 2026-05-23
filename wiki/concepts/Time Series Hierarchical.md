---
title: "Time Series Hierarchical"
tags:
  - data-science
  - time-series
  - hierarchical
  - reconciliation
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Multivariate]]"
  - "[[Time Series ML and DL Models]]"
  - "[[Time Series Hackathon Guide]]"
---

# Time Series Hierarchical

## What It Is

Hierarchical time series are collections of series that aggregate naturally: total sales = sum of regional sales = sum of store sales = sum of product sales. Every level must be **coherent** — forecasts at lower levels must sum to the forecast at the level above.

Naive model-per-level approaches produce incoherent forecasts (the parts don't add up to the whole). Reconciliation fixes this.

## The Structure

```
Total
├── Region A
│   ├── Store A1
│   │   ├── Product X
│   │   └── Product Y
│   └── Store A2
│       ├── Product X
│       └── Product Y
└── Region B
    └── ...
```

**Grouped time series**: a related concept where series can be grouped by multiple attributes (geography AND product category) without a strict hierarchy.

---

## Forecasting Strategies

### Bottom-Up

Forecast the most disaggregated level (leaf nodes), then sum upward.

- Preserves most granular patterns
- Aggregation reduces noise — top-level accuracy is often good
- Bottom-level accuracy may be poor if individual series are very short or noisy
- Simplest to implement

```python
# Forecast each bottom-level series
# Sum them to get higher-level forecasts
top_forecast = sum(bottom_level_forecasts)
```

### Top-Down

Forecast the total, then disaggregate using historical proportions.

- Top-level forecast is usually more stable (less noise)
- Disaggregation introduces bias — statistically, this method is suboptimal
- Proportions may be unstable over time
- Use when bottom-level series are too short to model individually

Disaggregation methods:
- **Average historical proportions**: `p_i = mean(y_i(t) / y_total(t))`
- **Proportions of historical averages**: `p_i = mean(y_i) / mean(y_total)`

### Middle-Out

Forecast at an intermediate level, then aggregate upward and disaggregate downward. A pragmatic compromise when the middle level has the best data quality.

### Optimal Reconciliation (MinT)

The state-of-the-art approach. Generates **base forecasts** at every level independently, then adjusts all of them simultaneously to minimize total forecast error variance while enforcing coherence.

`ỹ = SG·ŷ`

Where:
- `ŷ` = vector of all base forecasts (all levels)
- `S` = summing matrix that defines the hierarchical structure
- `G` = reconciliation matrix derived from the covariance of base forecast errors

**MinT (Minimum Trace)** estimator finds the optimal `G` by minimizing the trace of the reconciled error covariance matrix. It outperforms bottom-up and top-down in most empirical studies. (Source: Hyndman et al., optimal reconciliation paper — high confidence)

Python: `hierarchicalforecast` library by Nixtla

```python
from hierarchicalforecast.methods import MinTrace
from hierarchicalforecast.core import HierarchicalReconciliation

hrec = HierarchicalReconciliation(reconcilers=[MinTrace(method='mint_shrink')])
reconciled_df = hrec.reconcile(Y_hat_df=base_forecasts, Y_df=train_df, S=S_df, tags=tags)
```

---

## When Reconciliation Matters Most

- **Retail**: SKU-level forecasts must sum to store-level, which must sum to regional and total. Safety stock is calculated at SKU level but budgets are set at regional level.
- **Finance**: desk-level P&L must aggregate to book, to business, to firm.
- **Energy**: zone-level demand must sum to grid total for dispatch planning.

In hackathons, competitions that use hierarchical data often score on **all levels simultaneously** (like M5's WRMSSE, which weights by sales volume). Coherent forecasts are required to score well.

---

## Practical Approach for Hackathons

1. Train one global LightGBM on the bottom level (most granular)
2. Generate base forecasts for all levels separately
3. Apply MinTrace reconciliation via `hierarchicalforecast`
4. This is faster and more accurate than fitting separate models per level

**Quick win**: even just bottom-up aggregation (no reconciliation) usually beats top-down because it preserves local patterns.

---

## Evaluation in Hierarchical Settings

- Compute metrics (MAE, RMSE) at each level separately
- Weight by level importance (higher levels usually weighted more in business)
- MASE is preferred because it is scale-free and comparable across levels
- Competition-specific metrics (WRMSSE in M5) weight by sales volume — learn the weighting scheme before optimizing
