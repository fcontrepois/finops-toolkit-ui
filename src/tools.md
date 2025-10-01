---
title: Tools
layout: default
---

# Tools catalog

Pipe-first, CSV in/out. Minimal flags. Compose them like LEGO® bricks.

| Lifecycle | Tool | Purpose | Quick pipe example |
|-----------|------|---------|--------------------|
| Collect | `demo/generate_series.py` | Synthetic time series (daily/monthly; flat/growth/step/spike) | `python demo/generate_series.py --pattern flat --granularity daily --periods 365 --baseline 100` |
| Transform | `demo/add_spikes.py` | Add bounded positive spikes | `... | python demo/add_spikes.py --max-pct 0.10 --prob 0.05` |
| Transform | `demo/add_seasonality.py` | Apply monthly seasonality (presets or custom factors) | `... | python demo/add_seasonality.py --preset toys` |
| Collect | `aws/cost_and_usage.py` | Export spend data from Cost Explorer (SERVICE/ACCOUNT/TAG/ALL) | `python aws/cost_and_usage.py --granularity daily --group SERVICE` |
| Analyze | `aws/budget_analysis.py` | Compare costs vs budgets; compute variance & status | `python aws/cost_and_usage.py --granularity daily | python aws/budget_analysis.py --budget-name "Monthly-Budget"` |
| Analyze | `aws/anomaly_detection_forecast.py` | Compare latest forecast vs prior periods; alert on threshold | `python aws/anomaly_detection_forecast.py --threshold 20` |
| Forecast | `forecast.py` | Multi-model forecast + ensemble | `... | python forecast.py --date-column PeriodStart --value-column Cost --ensemble` |

*Maturity hints*: `generate_series`, `add_spikes`, `add_seasonality`, `forecast.py` are demo-ready; AWS commands are production-grade with proper credentials.

See the [Docs](./docs) page for deeper dive links and standards.
