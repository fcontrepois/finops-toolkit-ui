---
title: Use cases
layout: default
---

# FinOps use cases and micro-tools

Reference: FinOps Foundation Use Cases [finops.org/finops-use-cases](https://www.finops.org/finops-use-cases/)

| Use case | Typical question | Example pipeline | Status / notes |
|----------|------------------|------------------|----------------|
| Cost allocation | “Who spent what?” | `aws/cost_and_usage.py --group SERVICE` → (optional adapter) → CSV export | ✅ ready |
| Budget tracking & variance | “Are we burning the budget too fast?” | `cost_and_usage` → `aws/budget_analysis.py --budget-name ...` | ✅ ready; add alert thresholds per team |
| Anomaly detection | “Is today’s spike normal?” | `cost_and_usage` → `forecast.py --ensemble` → `aws/anomaly_detection_forecast.py` | ✅ ready; thresholds configurable |
| Forecasting & planning | “What will next quarter cost?” | source CSV → adapters (`add_spikes`, `add_seasonality`) → `forecast.py` → dashboards | ✅ demo-ready, extend for prod |
| Unit cost tracking | “Is cost per transaction improving?” | spend CSV + usage metrics → join adapter (planned) → `forecast`/report | 🔜 needs join micro-tool |
| Commitment coverage | “Do we have enough RIs/SPs?” | `cost_and_usage` → (planned commitment analyzer) → summary | 🔜 needs commitment analyzer |

Have an idea? File an issue, or drop a PR with a new micro-tool pipe.
