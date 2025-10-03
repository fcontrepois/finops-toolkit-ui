---
title: Context is everything
---

# Context is everything

- Generic tools solve generic problems for generic users. But no one is generic.
- The same "all good" signal means different things in different contexts.
- Good FinOps answers are situational: same data, different actions depending on time, audience, and constraints.

## Why this project exists

- **Before**: generic dashboards shout "cost anomaly" without telling you if it is launch day, month-end, or a billing bug.
- **After**: tiny, composable tools let you adjust the message to *your* reality—seasonality, closing windows, on-call posture.
- **How**: treat every question as a pipeline (`source → adapter → model → decision`) and add thin layers of context exactly where they matter.

```
┌────────────┐   ┌──────────┐   ┌────────┐   ┌──────────┐
│ CSV / API │ → │ adapters │ → │ models │ → │ decision │
└────────────┘   └──────────┘   └────────┘   └──────────┘
```

## Everyday context shifts

## How we respond to context
- Start with generic micro-tools (CSV in/out; composable with pipes).
- Add thin adapters encoding local context (seasonality, thresholds, SLA windows).
- Tune parameters (don't hard-code "universal" rules).

> “Once we added the seasonal adapter, the on-call stopped getting paged every time marketing ran a promo.” – A relieved FinOps lead

## Get started
- **[Forecast demo](./forecast-demo)** – Watch the “build a demo” journey and see each adapter/tool appear as context evolves.
- **[Tools](./tools)** – Browse every micro-tool, grouped by lifecycle, with pipe-first examples and deep links to documentation.
- **[Use cases](./use-cases)** – Map classic FinOps questions to pipelines; spot what exists today and where contributions are welcome.
- **[Docs](./docs)** – Quick launchpad into standards, architecture, and contribution guidelines.
- Feeling playful? Try imagining how context changes a non-FinOps scenario (concert planning, cooking, road-trips)—it works there too.
