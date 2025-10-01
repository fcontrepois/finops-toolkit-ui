---
title: Docs
layout: default
---

# Documentation

Need more depth? Jump into the repo docs with context-aware breadcrumbs.

- **Forecast tool** – high-level concepts, CLI flags, tuning guidance (`documentation/FORECAST.md`).
- **Coding standards** – pipe-first expectations, error codes, stdin/stdout etiquette (`documentation/CODING_STANDARDS.md`).
- **Command architecture** – how commands stay independent yet composable (`documentation/COMMAND_ARCHITECTURE.md`).
- **Contributing** – short version: fork, branch, pipe everything; see CONTRIBUTING.md (coming soon) or open an issue.

## Pipe-first quick starts
```bash
# Flat + toys → forecast
python demo/generate_series.py --pattern flat --granularity daily --periods 365 --baseline 100 --noise 0.0 \
| python demo/add_seasonality.py --preset toys \
| python forecast.py --date-column PeriodStart --value-column Cost --ensemble
```

```bash
# Growth + holidays → forecast
python demo/generate_series.py --pattern upward_trend --granularity daily --periods 365 --baseline 100 --trend 0.5 --noise 0.0 \
| python demo/add_seasonality.py --preset holidays \
| python forecast.py --date-column PeriodStart --value-column Cost --ensemble
```

Want more? Open an issue with your context and we’ll help design the pipeline.

