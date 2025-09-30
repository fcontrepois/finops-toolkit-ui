---
title: Docs
layout: default
---

# Documentation

- Forecast tool: see repository `documentation/FORECAST.md`
- Coding standards (pipe-first): see `documentation/CODING_STANDARDS.md`
- Command architecture: see `documentation/COMMAND_ARCHITECTURE.md`

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

