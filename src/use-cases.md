---
title: Use cases
layout: default
---

# FinOps use cases and micro-tools

Reference: FinOps Foundation Use Cases [finops.org/finops-use-cases](https://www.finops.org/finops-use-cases/)

## Examples
- Cost allocation (by service/account/tag)
  - Pipeline: `cost_and_usage` → group/aggregate → export
  - Status: exists
- Budget tracking & variance
  - Pipeline: `cost_and_usage` → `budget_analysis` (alerts optional)
  - Status: exists
- Anomaly detection
  - Pipeline: `cost_and_usage` → `forecast` → `anomaly_detection_forecast`
  - Status: exists (threshold-based)
- Forecasting and planning
  - Pipeline: your CSV → `forecast` (optional ensemble)
  - Status: exists
- Unit cost tracking
  - Pipeline: join usage metrics → compute unit metrics → `forecast`
  - Status: planned (join/transform micro-tools)
- Savings coverage/optimization
  - Pipeline: cost baselines → commitment evaluation (RI/SP) → report
  - Status: planned (commitment analysis micro-tool)

Suggest a use case or tool gap: open an issue in the repo.
