---
title: Context is everything
---

# Context is everything

- Generic tools solve generic problems for generic users. But no one is generic.
- The same "all good" signal means different things in different contexts.
- Good FinOps answers are situational: same data, different actions depending on time, audience, and constraints.

## Everyday context shifts
- **All is OK**
  - ICU patient: stabilizing = success; tiny regressions matter.
  - Dinner plans: restaurant open and a table soon = success; small delays don't.
- **Will they close?**
  - Seller day 5: "Likely" = nurture; build momentum.
  - Seller day 89 (end of Q): "Likely" = act now; escalate resources.
- **Cost is flat**
  - Startup: flat = risk (no growth); investigate adoption.
  - Cost crisis: flat = win (stopped the bleed).

## FinOps context shifts
- **Under budget by 5%**
  - Before RI/SP commitment window: may be fine to commit more.
  - After procurement freeze: could be a miss (under-investment).
- **Forecast up 12% next month**
  - Seasonal retail in Nov/Dec: expected; align with revenue plan.
  - SaaS mid‑quarter with no launches: anomaly; triage tags/services/owners.
- **High spend on Service X**
  - Launch week: planned; focus on unit cost vs feature success.
  - Steady-state: unplanned; check autoscaling/idle/misrouting.

## How we respond to context
- Start with generic micro-tools (CSV in/out; composable with pipes).
- Add thin adapters encoding local context (seasonality, thresholds, SLA windows).
- Tune parameters (don't hard-code "universal" rules).

## Get started
- **[Forecast demo](./forecast-demo)**: See how context added tools (generator → spikes → seasonality → tuning).
- **[Tools](./tools)**: Browse micro-tools with pipe-first examples.
- **[Use cases](./use-cases)**: Classic FinOps activities mapped to micro-tools.
- **[Docs](./docs)**: Quick starts and links to full documentation.
