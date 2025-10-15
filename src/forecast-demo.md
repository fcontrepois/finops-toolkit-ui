---
title: Forecast demo
toc: false
---
# Forecasting demo walkthrough 🚀

> “Goal: build a compelling demo using only composable pieces. As context evolved, new micro-tools appeared.”

<iframe width="560" height="315" src="https://www.youtube.com/embed/Gnmemlj8A9M?si=cXaEE2L-iKAKHIGD" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>f

## What we’ll try

We’ll create synthetic time series with:
- Flat baseline (no noise)
- Linear growth (no noise)
- Flat/growth with occasional positive spikes (≤10% and ≤20%)

For each, we’ll produce forecasts for the next month, quarter, and year, then plot actuals vs predictions

### Algorithms in plain language

Here’s what each method tries to do, in simple terms:
- **SMA** (Simple Moving Average): Average of the last N points. Very stable, slow to react.
- **ES** (Exponential Smoothing): Weighted average that favors recent data. Reacts faster than SMA.
- **Holt‑Winters** (Triple Exponential Smoothing): Tracks level and trend (and seasonality if present). Good when there’s a slope and repeating patterns.
- **ARIMA**: Statistical model for short‑term autocorrelation. Can capture persistence and mean‑reversion without seasonality.
- **SARIMA**: ARIMA with seasonality. Useful if there’s a repeating weekly/monthly pattern.
- **Theta**: Blend of level and trend lines. Often a strong baseline on simple series.
- **Prophet**: Piecewise linear trend with changepoints plus seasonality priors. Adaptable when trends change.
- **NeuralProphet**: Neural network version of Prophet—can fit flexible trends/seasonality; needs enough variation.
- **Ensemble**: Average of available models to reduce variance and over‑/under‑reaction.

### Mini-map: how context shaped the tools

1. **Baseline** – Needed clean daily data → `generate_series.py`
2. **Spikes appear** – Added `add_spikes.py` to simulate noisy launches
3. **Seasonality shows up** – Built `add_seasonality.py`
4. **Forecast tuning** – Tweaked parameters for specific business patterns

Each step uses the same pipe-first workflow so you can remix or extend it.

## 0) Setup (one time per shell)

```bash
source .venv/bin/activate
pip install -r requirements.txt
mkdir -p demo/input demo/out
```

## 1) Flat (no noise) — one‑year of daily data
<!-- Load and transform the data -->
```js
const flat = FileAttachment("data/input/daily_flat.csv").csv({typed: true});
const flatOut = FileAttachment("data/out/daily_flat_forecasts.csv").csv({typed: true});
```
```js
const flatOutPivoted = pivotForecastData(flatOut);
```

First, a perfectly flat year with no noise.
```bash
python demo/generate_series.py \
  --pattern flat --granularity daily --periods 365 \
  --baseline 10 --noise 0.0 \
  --out demo/input/daily_flat.csv
```

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Flat</h2>
    <span>${tableInput(flat)}</span>
  </div>
    <div class="card">
    <span>${resize((width) => autoGraphInput(flat, {width}))}</span>
  </div>
</div>

Now I run the forecaster and include an ensemble column which averages available models.
```bash
python forecast.py \
  --input demo/input/daily_flat.csv \
  --date-column PeriodStart --value-column Cost \
  --ensemble > demo/out/daily_flat_forecasts.csv
```
The output CSV contains the original values and one column per forecasting method.

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Forecasted data </h2>
    <span">${tableOutput(flatOut)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(flatOutPivoted, {width}))}</span>
  </div>
</div>

Good news, every forecasting method manage flat well.

## Growing a little each day

<!-- Load and transform the data -->
```js
const growth = FileAttachment("data/input/daily_growth.csv").csv({typed: true});
const growthOut = FileAttachment("data/out/daily_growth_forecasts.csv").csv({typed: true});
```
```js
const growthOutPivoted = pivotForecastData(growthOut);
```

Second, a predictable linear growth pattern—half a unit per day.
```bash
python demo/generate_series.py \
  --pattern upward_trend --granularity daily --periods 365 \
  --baseline 10 --trend 0.5 --noise 0.0 \
  --out demo/input/daily_growth.csv
```
The trend increases the baseline steadily; no noise keeps it clean for comparison.

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Growing slowly</h2>
    <span>${tableInput(growth)}</span>
  </div>
    <div class="card">
    <span>${resize((width) => autoGraphInput(growth, {width}))}</span>
  </div>
</div>

Now I run the forecaster and include an ensemble column which averages available models.
```bash
python forecast.py \
  --input demo/input/daily_growth.csv \
  --date-column PeriodStart --value-column Cost \
  --ensemble > demo/out/daily_growth_forecasts.csv
```
The output CSV contains the original values and one column per forecasting method.

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecasted data </h2>
    <span">${tableOutput(growthOut)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(growthOutPivoted, {width}))}</span>
  </div>
</div>

## 3) Flat with spikes (≤10% daily)
**Why it matters:** Shows how small, infrequent spikes nudge forecasts and why ensemble smoothing helps avoid overreacting.

<!-- Load and transform the data -->
```js
const flatBase10 = FileAttachment("data/input/daily_flat_10_base.csv").csv({typed: true});
const flatSpikes10 = FileAttachment("data/input/daily_flat_spikes_10.csv").csv({typed: true});
const flatSpikes10Out = FileAttachment("data/out/daily_flat_spikes_10_forecasts.csv").csv({typed: true});
```
```js
const flatSpikes10OutPivoted = pivotForecastData(flatSpikes10Out);
```

Third, I introduce occasional positive spikes up to ten percent on a flat baseline.
```bash
python demo/generate_series.py \
  --pattern flat --granularity daily --periods 365 \
  --baseline 10 --noise 0.0 \
  --out demo/input/daily_flat_10_base.csv
python demo/add_spikes.py \
  --input demo/input/daily_flat_10_base.csv \
  --output demo/input/daily_flat_spikes_10.csv \
  --max-pct 0.10 --prob 0.05
```
This applies bounded spikes (max ten percent) with a small daily probability, using a fixed seed for reproducibility.

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Growing slowly</h2>
    <span>${tableInput(flatSpikes10)}</span>
  </div>
    <div class="card">
    <span>${resize((width) => autoGraphInput(flatSpikes10, {width}))}</span>
  </div>
</div>

I'll run forecasts and show how each model handles transient spikes over the three horizons.
```bash
python forecast.py \
  --input demo/input/daily_flat_spikes_10.csv \
  --date-column PeriodStart --value-column Cost \
  --ensemble > demo/out/daily_flat_spikes_10_forecasts.csv
```

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecasted data </h2>
    <span">${tableOutput(flatSpikes10Out)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(flatSpikes10OutPivoted, {width}))}</span>
  </div>
</div>


## 4) Flat with larger spikes (≤33% daily)
**Why it matters:** Bigger spikes with higher probability stress models—useful for promo-heavy or launch-driven workloads.

<!-- Load and transform the data -->
```js
const flatSpikes33 = FileAttachment("data/input/daily_flat_spikes_33.csv").csv({typed: true});
const flatSpikes33Out = FileAttachment("data/out/daily_flat_spikes_33_forecasts.csv").csv({typed: true});
```
```js
const flatSpikes33OutPivoted = pivotForecastData(flatSpikes33Out);
```

Larger spikes up to 33 percent on a flat baseline, stressing the models more.
```bash
python demo/generate_series.py \
  --pattern flat --granularity daily --periods 365 \
  --baseline 10 --noise 0.0 \
  --out demo/input/daily_flat_33_base.csv
python demo/add_spikes.py \
  --input demo/input/daily_flat_33_base.csv \
  --output demo/input/daily_flat_spikes_33.csv \
  --max-pct 0.33 --prob 0.15
```
This applies bounded spikes (max 33 percent) with higher daily probability, using a fixed seed for reproducibility.

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Flat with larger spikes</h2>
    <span>${tableInput(flatSpikes33)}</span>
  </div>
    <div class="card">
    <span>${resize((width) => autoGraphInput(flatSpikes33, {width}))}</span>
  </div>
</div>

I'll run forecasts and show how each model handles transient spikes over the three horizons.
```bash
python forecast.py \
  --input demo/input/daily_flat_spikes_33.csv \
  --date-column PeriodStart --value-column Cost \
  --ensemble > demo/out/daily_flat_spikes_33_forecasts.csv
```

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecasted data </h2>
    <span">${tableOutput(flatSpikes33Out)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(flatSpikes33OutPivoted, {width}))}</span>
  </div>
</div>

## 5) Growth with spikes (≤33% daily) 15% change of happening
**Why it matters:** Real FinOps pipelines often mix trend + events; this section shows interplay of growth and burstiness.

<!-- Load and transform the data -->
```js
const growthBase33 = FileAttachment("data/input/daily_growth_33_base.csv").csv({typed: true});
const growthSpike33 = FileAttachment("data/input/daily_growth_spikes_33.csv").csv({typed: true});
const growthSpike33Out = FileAttachment("data/out/daily_growth_spikes_33_forecasts.csv").csv({typed: true});
```
```js
const growthSpike33OutPivoted = pivotForecastData(growthSpike33Out);
```

Finally, growth with larger spikes up to twenty percent, stressing the models.

```bash
python demo/generate_series.py \
  --pattern upward_trend --granularity daily --periods 365 \
  --baseline 100 --trend 0.5 --noise 0.0 \
  --out demo/input/daily_growth_33_base.csv
python demo/add_spikes.py \
  --input demo/input/daily_growth_33_base.csv \
  --output demo/input/daily_growth_spikes_33.csv \
  --max-pct 0.33 --prob 0.15
```

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Growing slowly</h2>
    <span>${tableInput(growthSpike33)}</span>
  </div>
    <div class="card">
    <span>${resize((width) => autoGraphInput(growthSpike33, {width}))}</span>
  </div>
</div>

I'll run forecasts and show how each model handles transient spikes over the three horizons.
```bash
python forecast.py \
  --input demo/input/daily_growth_spikes_33.csv \
  --date-column PeriodStart --value-column Cost \
  --ensemble > demo/out/daily_growth_spikes_33_forecasts.csv
```

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecasted data </h2>
    <span">${tableOutput(growthSpike33Out)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(growthSpike33OutPivoted, {width}))}</span>
  </div>
</div>

## 6) Daily seasonality — flat baseline (toys and holidays)
**Why it matters:** Retail-style cyclicality; illustrates how external adapters layer in context before forecasting.

<!-- Load and transform the data -->
```js
const dailyFlatToys = FileAttachment("data/input/daily_flat_toys.csv").csv({typed: true});
const dailyFlatToysOut = FileAttachment("data/out/daily_flat_toys_forecasts.csv").csv({typed: true});
const dailyFlatHolidays = FileAttachment("data/input/daily_flat_holidays.csv").csv({typed: true});
const dailyFlatHolidaysOut = FileAttachment("data/out/daily_flat_holidays_forecasts.csv").csv({typed: true});
```
```js
const dailyFlatToysOutPivoted = pivotForecastData(dailyFlatToysOut);
const dailyFlatHolidaysOutPivoted = pivotForecastData(dailyFlatHolidaysOut);
```

Flat baseline (no noise) with two seasonal profiles applied externally, then forecasted.

```bash
python demo/generate_series.py --pattern flat --granularity daily --periods 365 --baseline 100 --noise 0.0 --out demo/input/daily_flat_base.csv && python demo/add_seasonality.py --input demo/input/daily_flat_base.csv --output demo/input/daily_flat_toys.csv --preset toys && python forecast.py --input demo/input/daily_flat_toys.csv --date-column PeriodStart --value-column Cost --ensemble > demo/out/daily_flat_toys_forecasts.csv && python demo/add_seasonality.py --input demo/input/daily_flat_base.csv --output demo/input/daily_flat_holidays.csv --preset holidays && python forecast.py --input demo/input/daily_flat_holidays.csv --date-column PeriodStart --value-column Cost --ensemble > demo/out/daily_flat_holidays_forecasts.csv
```

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Flat + Toys seasonality</h2>
    <span>${tableInput(dailyFlatToys)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphInput(dailyFlatToys, {width}))}</span>
  </div>
</div>

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecasted data (Toys)</h2>
    <span">${tableOutput(dailyFlatToysOut)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(dailyFlatToysOutPivoted, {width}))}</span>
  </div>
</div>

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Flat + Holidays seasonality</h2>
    <span>${tableInput(dailyFlatHolidays)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphInput(dailyFlatHolidays, {width}))}</span>
  </div>
</div>

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecasted data (Holidays)</h2>
    <span">${tableOutput(dailyFlatHolidaysOut)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(dailyFlatHolidaysOutPivoted, {width}))}</span>
  </div>
</div>

## 7) Daily seasonality — growth baseline (toys and holidays)
**Why it matters:** Growth + seasonality is a classic FinOps forecasting headache; compare outputs before tuning.

<!-- Load and transform the data -->
```js
const dailyGrowthToys = FileAttachment("data/input/daily_growth_toys.csv").csv({typed: true});
const dailyGrowthToysOut = FileAttachment("data/out/daily_growth_toys_forecasts.csv").csv({typed: true});
const dailyGrowthHolidays = FileAttachment("data/input/daily_growth_holidays.csv").csv({typed: true});
const dailyGrowthHolidaysOut = FileAttachment("data/out/daily_growth_holidays_forecasts.csv").csv({typed: true});
```
```js
const dailyGrowthToysOutPivoted = pivotForecastData(dailyGrowthToysOut);
const dailyGrowthHolidaysOutPivoted = pivotForecastData(dailyGrowthHolidaysOut);
```

Upward trend baseline with the same two seasonal profiles.

```bash
python demo/generate_series.py --pattern upward_trend --granularity daily --periods 365 --baseline 100 --trend 0.5 --noise 0.0 --out demo/input/daily_growth_base.csv && python demo/add_seasonality.py --input demo/input/daily_growth_base.csv --output demo/input/daily_growth_toys.csv --preset toys && python forecast.py --input demo/input/daily_growth_toys.csv --date-column PeriodStart --value-column Cost --ensemble > demo/out/daily_growth_toys_forecasts.csv && python demo/add_seasonality.py --input demo/input/daily_growth_base.csv --output demo/input/daily_growth_holidays.csv --preset holidays && python forecast.py --input demo/input/daily_growth_holidays.csv --date-column PeriodStart --value-column Cost --ensemble > demo/out/daily_growth_holidays_forecasts.csv
```

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Growth + Toys seasonality</h2>
    <span>${tableInput(dailyGrowthToys)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphInput(dailyGrowthToys, {width}))}</span>
  </div>
</div>

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecasted data (Toys)</h2>
    <span">${tableOutput(dailyGrowthToysOut)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(dailyGrowthToysOutPivoted, {width}))}</span>
  </div>
</div>

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Growth + Holidays seasonality</h2>
    <span>${tableInput(dailyGrowthHolidays)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphInput(dailyGrowthHolidays, {width}))}</span>
  </div>
</div>

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecasted data (Holidays)</h2>
    <span">${tableOutput(dailyGrowthHolidaysOut)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(dailyGrowthHolidaysOutPivoted, {width}))}</span>
  </div>
</div>

# Optional: tuning when seasonality is known

If you know your industry has yearly seasonality (e.g., toys or holidays), the defaults already work reasonably well for daily data. Prophet has yearly seasonality enabled by default and tends to capture these patterns without extra tuning. If you want gentle tweaks:

- Increase Prophet seasonality strength a bit
- Keep weekly seasonality off (unless you truly have weekly patterns)
- Leave Holt‑Winters as‑is for daily yearly seasonality unless you have ≥2 years of data (it needs at least 2×seasonal_periods)

```bash
# Daily data with yearly seasonality: nudge Prophet priors and keep ensemble
python forecast.py \
  --input demo/input/daily_flat_toys.csv \
  --date-column PeriodStart --value-column Cost \
  --prophet-yearly-seasonality true \
  --prophet-weekly-seasonality false \
  --prophet-changepoint-prior-scale 0.05 \
  --prophet-seasonality-prior-scale 12.0 \
  --ensemble > demo/out/daily_flat_toys_forecasts_tuned.csv
```

Notes:
- Holt‑Winters on daily yearly patterns requires `--hw-seasonal-periods 365` and ≥730 days of history; otherwise it falls back to simple ES (current demo has 365 days, so fallback is expected).
- If your data also has a weekly cycle, try setting `--prophet-weekly-seasonality true` and consider `--sarima-seasonal-order 1,0,1,7` (requires statsmodels), but only if you truly expect weekly effects.

# A couple of alternative tuning examples

```bash
# Slightly more responsive moving average and ES
python forecast.py \
  --input demo/input/daily_growth_holidays.csv \
  --date-column PeriodStart --value-column Cost \
  --sma-window 14 \
  --es-alpha 0.7 \
  --ensemble > demo/out/daily_growth_holidays_forecasts_tuned_baselines.csv

# Add weekly effects (only if real weekly pattern exists) and SARIMA weekly seasonality
python forecast.py \
  --input demo/input/daily_growth_holidays.csv \
  --date-column PeriodStart --value-column Cost \
  --prophet-weekly-seasonality true \
  --sarima-order 1,1,1 \
  --sarima-seasonal-order 1,0,1,7 \
  --ensemble > demo/out/daily_growth_holidays_forecasts_with_weekly.csv
```

## 8) Tuned seasonal forecasts (examples)
**Why it matters:** Demonstrates parameter tuning (Prophet priors, SARIMA weekly) once you know the business rhythm.

<!-- Load tuned outputs -->
```js
const dailyFlatToysTunedOut = FileAttachment("data/out/daily_flat_toys_forecasts_tuned.csv").csv({typed: true});
const dailyGrowthHolidaysTunedOut = FileAttachment("data/out/daily_growth_holidays_forecasts_tuned.csv").csv({typed: true});
```
```js
const dailyFlatToysTunedOutPivoted = pivotForecastData(dailyFlatToysTunedOut);
const dailyGrowthHolidaysTunedOutPivoted = pivotForecastData(dailyGrowthHolidaysTunedOut);
```

Two tuned runs to illustrate parameter effects when yearly seasonality is known.

```bash
# Flat + toys seasonality (slightly stronger seasonality prior)
python forecast.py \
  --input demo/input/daily_flat_toys.csv \
  --date-column PeriodStart --value-column Cost \
  --prophet-yearly-seasonality true \
  --prophet-weekly-seasonality false \
  --prophet-changepoint-prior-scale 0.05 \
  --prophet-seasonality-prior-scale 12.0 \
  --ensemble > demo/out/daily_flat_toys_forecasts_tuned.csv

# Growth + holidays seasonality (enable weekly; add SARIMA weekly component)
python forecast.py \
  --input demo/input/daily_growth_holidays.csv \
  --date-column PeriodStart --value-column Cost \
  --prophet-weekly-seasonality true \
  --sarima-order 1,1,1 \
  --sarima-seasonal-order 1,0,1,7 \
  --ensemble > demo/out/daily_growth_holidays_forecasts_tuned.csv
```

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Tuned forecast: Flat + Toys</h2>
    <span">${tableOutput(dailyFlatToysTunedOut)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(dailyFlatToysTunedOutPivoted, {width}))}</span>
  </div>
</div>

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Tuned forecast: Growth + Holidays</h2>
    <span">${tableOutput(dailyGrowthHolidaysTunedOut)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(dailyGrowthHolidaysTunedOutPivoted, {width}))}</span>
  </div>
</div>

# Apendix

```js
function tableInput(data, {width} = {}){
  return Inputs.table(data, {select: false})
}
```

```js
function tableOutput(data, {width} = {}){
  return Inputs.table(data, {
    select: false,
    columns: [
      "PeriodStart",
      "Cost",
      "sma",
      "es",
      "hw", 
      "arima",
      "sarima",
      "theta",
      "prophet",
      "ensemble"
    ]
  })
}
```

```js
function autoGraphInput(data, {width} = {}) {
  return Plot.plot({
    y: {nice: true},
    marks: [
      Plot.ruleY([0]),
      Plot.line(data, {
        x: "PeriodStart", 
        y: "Cost",
        tip: true,
      })
    ]
  })
}
```
```js
function autoGraphOut(data, {width} = {}) {
  return Plot.plot({
    y: {nice: true},
    marks: [
      Plot.ruleY([0]),
      Plot.ruleX([new Date]),
      Plot.line(data, {
        x: "PeriodStart", 
        y: "Cost",
        stroke: "Method",
        tip: true,
      })
    ]
  })
}
```

```js
// Function to pivot forecast data so each method becomes a separate row, including original data
function pivotForecastData(data) {
  return data.flatMap(d => {
    const forecastMethods = ['sma', 'es', 'hw', 'arima', 'sarima', 'theta', 'prophet', 'neural_prophet', 'darts', 'ensemble'];
    
    // Include original cost data
    const originalData = {
      PeriodStart: d.PeriodStart,
      Cost: d.Cost,
      Method: 'actual'
    };
    
    // Include forecast data for each method
    const forecastData = forecastMethods.map(method => ({
      PeriodStart: d.PeriodStart,
      Cost: d[method],
      Method: method
    })).filter(row => row.Cost !== null && row.Cost !== undefined);
    
    // Include original data if it has a cost value
    const result = d.Cost !== null && d.Cost !== undefined ? [originalData, ...forecastData] : forecastData;
    return result;
  });
}
```