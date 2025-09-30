---
title: Tools
layout: default
---

# Tools catalog

Pipe-first, CSV in/out. Minimal flags. Composable.

## Data
- generate_series.py
  - Purpose: synthetic time series (daily/monthly; flat/growth/step/spike)
  - Pipe example:
    ```bash
    python demo/generate_series.py --pattern flat --granularity daily --periods 365 --baseline 100
    ```
- add_spikes.py
  - Purpose: apply bounded positive spikes
  - Pipe example:
    ```bash
    ... | python demo/add_spikes.py --max-pct 0.10 --prob 0.05
    ```
- add_seasonality.py
  - Purpose: apply monthly seasonality (presets toys/holidays or --factors)
  - Pipe example:
    ```bash
    ... | python demo/add_seasonality.py --preset toys
    ```

## AWS
- aws/cost_and_usage.py
  - Purpose: CSV export from Cost Explorer (SERVICE/ACCOUNT/TAG/ALL)
  - Example:
    ```bash
    python aws/cost_and_usage.py --granularity daily --group SERVICE
    ```
- aws/budget_analysis.py
  - Purpose: compare costs vs budgets; variance and status
  - Pipe example:
    ```bash
    python aws/cost_and_usage.py --granularity daily | python aws/budget_analysis.py --budget-name "Monthly-Budget"
    ```
- aws/anomaly_detection_forecast.py
  - Purpose: compare forecast vs prior periods; alert if threshold breached
  - Example:
    ```bash
    python aws/anomaly_detection_forecast.py --threshold 20
    ```

## Forecast
- forecast.py
  - Purpose: multi-model forecasting + ensemble
  - Pipe example:
    ```bash
    ... | python forecast.py --date-column PeriodStart --value-column Cost --ensemble
    ```
