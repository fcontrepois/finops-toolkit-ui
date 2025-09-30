---
theme: dashboard
title: Forecast demo
toc: false
---
# Forecasting demo walkthrough 🚀

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


## 4) Flat with spikes (≤40% daily) 5% change of happening

<!-- Load and transform the data -->
```js
const flatBase40 = FileAttachment("data/input/daily_flat_40_base.csv").csv({typed: true});
const flatSpikes40 = FileAttachment("data/input/daily_flat_spikes_40.csv").csv({typed: true});
const flatSpikes40Out = FileAttachment("data/out/daily_flat_spikes_40_forecasts.csv").csv({typed: true});
```
```js
const flatSpikes40OutPivoted = pivotForecastData(flatSpikes40Out);
```

Third, I introduce occasional positive spikes up to fourty percent on a flat baseline.
```bash
python demo/generate_series.py \
  --pattern flat --granularity daily --periods 365 \
  --baseline 10 --noise 0.0 \
  --out demo/input/daily_flat_40_base.csv
python demo/add_spikes.py \
  --input demo/input/daily_flat_40_base.csv \
  --output demo/input/daily_flat_spikes_40.csv \
  --max-pct 0.40 --prob 0.05
```
This applies bounded spikes (max 40 percent) with a small daily probability, using a fixed seed for reproducibility.

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Growing slowly</h2>
    <span>${tableInput(flatSpikes40)}</span>
  </div>
    <div class="card">
    <span>${resize((width) => autoGraphInput(flatSpikes40, {width}))}</span>
  </div>
</div>

I'll run forecasts and show how each model handles transient spikes over the three horizons.
```bash
python forecast.py \
  --input demo/input/daily_flat_spikes_40.csv \
  --date-column PeriodStart --value-column Cost \
  --ensemble > demo/out/daily_flat_spikes_40_forecasts.csv
```

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecasted data </h2>
    <span">${tableOutput(flatSpikes40Out)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(flatSpikes40OutPivoted, {width}))}</span>
  </div>
</div>

## 5) Flat with spikes (≤33% daily) 15% change of happening

<!-- Load and transform the data -->
```js
const flatBase33 = FileAttachment("data/input/daily_flat_33_base.csv").csv({typed: true});
const flatSpikes33 = FileAttachment("data/input/daily_flat_spikes_33.csv").csv({typed: true});
const flatSpikes33Out = FileAttachment("data/out/daily_flat_spikes_33_forecasts.csv").csv({typed: true});
```
```js
const flatSpikes33OutPivoted = pivotForecastData(flatSpikes33Out);
```

Third, I introduce occasional positive spikes up to 33 percent on a flat baseline.
```bash
python demo/generate_series.py \
  --pattern flat --granularity daily --periods 365 \
  --baseline 10 --noise 0.0 \
  --out demo/input/daily_flat_33_base.csv
python demo/add_spikes.py \
  --input demo/input/daily_flat_33_base.csv \
  --output demo/input/daily_flat_spikes_33.csv \
  --max-pct 0.33 --prob 0.05
```
This applies bounded spikes (max ten percent) with a 15% daily probability, using a fixed seed for reproducibility.

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Growing slowly</h2>
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

## 6) Growth with spikes (≤33% daily) 15% change of happening

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

## 7) Daily seasonality — flat baseline (toys and holidays)

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

## 8) Daily seasonality — growth baseline (toys and holidays)

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