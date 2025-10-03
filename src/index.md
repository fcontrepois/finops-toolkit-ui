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

Imagine you open a weather app. It says: “12 degrees.”

Okay… but what does that mean?

In London in July, that’s cold.
In Oslo in January, that’s warm.
If you’re going for a run, it’s uncomfortable.
If you’re storing vaccines, it’s a disaster.

Same number. Same data. But the meaning changes completely depending on the context.

And this is the trap with generic dashboards and generic tools. They stop one step short of being useful. They give you data, but not your data. They don’t bridge the “last mile” — the point where information becomes specific, relevant, and actionable.

That’s what Coblan (and me) is about. We build that last mile.
Not with giant platforms or one-size-fits-all dashboards — but with micro tools: tiny, fast, cheap applications that transform generic data into your data.

A micro tool can be built in 15 minutes. It doesn’t try to solve everything. It solves your problem, right now, in your context.

The future isn’t generic tools. It’s context-specific micro tools that make data meaningful for me. At Coblan, that’s what we do: we engineer the context, so that every answer, every report, every dashboard is not just data — it’s my data.

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
