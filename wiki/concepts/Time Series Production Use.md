---
title: "Time Series Production Use"
tags:
  - data-science
  - time-series
  - production
  - business
status: evergreen
related:
  - "[[Time Series Analysis]]"
  - "[[Time Series Real-World Applications]]"
  - "[[Time Series Evaluation Metrics]]"
---

# Time Series Production Use

## What Decision-Makers Actually Want

A model that achieves low MAE is useless if the output cannot be acted on. Decision-makers need:

1. **A point forecast** — the expected value (e.g., "we expect 1,200 units sold next week")
2. **Uncertainty bounds** — the range of likely outcomes (e.g., "90% chance between 900 and 1,600")
3. **Interpretability** — which factors drove the forecast
4. **Timeliness** — forecast must be ready before the decision must be made
5. **Stability** — the forecast should not swing wildly day to day without cause

## Forecast Horizon and Decision Windows

The forecast horizon must match the decision cycle:

| Decision | Horizon Needed |
|----------|---------------|
| Reorder inventory today | 2–4 weeks |
| Schedule workforce | 1–4 weeks |
| Budget planning | 3–12 months |
| Capacity investment | 1–5 years |

Mismatched horizons are a common failure mode: a model trained on 1-step-ahead forecasts is useless for a business that needs 90-day planning.

## Outputs Beyond the Point Forecast

**Prediction intervals**: 80% and 95% intervals communicate risk. Overstock uses the upper bound; stockout prevention uses the lower bound.

**Scenario forecasting**: run the model under multiple assumption sets (optimistic, base, pessimistic). Useful for strategic planning.

**Anomaly alerts**: detect when actuals deviate from forecast by more than X standard deviations. Triggers investigation.

**Attribution**: which features drove this forecast? Feature importance from LightGBM or attention weights from TFT help explain "why did sales spike?"

## How Forecasts Drive Decisions

### Retail / Supply Chain
The forecast feeds directly into the replenishment system. Safety stock = `z × σ × sqrt(lead_time)`, where σ comes from the forecast error distribution. A more accurate forecast reduces safety stock (working capital saved) while maintaining the same service level.

### Energy
Grid operators use 24-hour load forecasts to dispatch generation assets. Forecast error causes either wasted generation (over-forecast) or emergency purchases at spot price (under-forecast). Intraday updates refine the forecast every hour.

### Finance
Algorithmic trading systems use short-horizon return forecasts as one signal among many. A forecast with a Sharpe ratio contribution above a threshold gets weight in the portfolio.

### Healthcare
Hospital admission forecasts trigger staffing adjustments 48-72 hours ahead. Surge predictions trigger supply pre-positioning. Outbreak models inform public health messaging.

## Deployment Patterns

**Batch forecasting**: run once per day/week, store results in a database, served to dashboards and planning tools. Most common.

**Real-time forecasting**: model served as an API; called per event (e.g., each user session, each sensor reading). Requires low latency (<100ms).

**Continuous retraining**: as new actuals arrive, retrain or update the model on a rolling window. Prevents model drift (when the world changes and the model's training distribution no longer matches).

## Model Monitoring

A deployed forecast must be monitored:
- **Forecast accuracy tracking**: log MAE/RMSE per period. Alert when it degrades.
- **Distribution shift**: is the incoming data still similar to training data? Use PSI (Population Stability Index).
- **Residual autocorrelation**: if residuals show systematic patterns, the model is missing structure. Retrain.
- **Business KPI tracking**: did acting on the forecast produce the expected outcome (e.g., service level, waste reduction)?

## What Stakeholders Expect From the Model

| Expectation | What It Means |
|-------------|--------------|
| "It should be accurate" | Beats naive baseline on the agreed metric |
| "It should be consistent" | Doesn't flip wildly without data justification |
| "It should explain itself" | Feature importance, trend/seasonality breakdown |
| "It should know what it doesn't know" | Prediction intervals, not just a point |
| "It should improve over time" | Continuous retraining pipeline |
| "It should handle exceptions" | Graceful behavior at holidays, missing data, outlier events |

## Common Failure Modes in Production

- **Training/serving skew**: features computed differently at training time vs. inference time
- **Horizon drift**: model trained on 1-step-ahead used for 30-step-ahead (error compounds)
- **Silent data quality failures**: upstream data pipeline breaks, model receives stale or null inputs and silently produces bad forecasts
- **Ignoring calendar effects**: a model that was never told about a national holiday will be wrong every year on that day
