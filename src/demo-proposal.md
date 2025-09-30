---
theme: dashboard
title: FinOps Toolkit: Small Tools, Big Answers
toc: false
---

# Small Tools, Big Answers 🔧

## The Philosophy

Instead of monolithic FinOps platforms, we built **small, inexpensive tools** that work together to answer your specific questions, right here, right now.

**One question → One pipeline → One answer**

---

## Common FinOps Questions (and how our tools answer them)

### ❓ "Why did my AWS bill spike last week?"

```bash
# Pipeline: Cost data → Anomaly detection → Root cause analysis
python aws/cost_and_usage.py --granularity daily --group SERVICE --start 2024-01-01 | \
  python aws/anomaly_detection_forecast.sh --threshold 20 --method prophet
```

### ❓ "Will we exceed our Q1 budget?"

```bash  
# Pipeline: Current costs → Forecast → Budget comparison
python aws/cost_and_usage.py --granularity daily | \
  python forecast.py --date-column PeriodStart --value-column Cost --ensemble | \
  python aws/budget_analysis.py --budget-name "Q1-Budget" --threshold 90
```

### ❓ "Which services are growing fastest?"

```bash
# Pipeline: Cost data → Trend analysis → Service ranking
python aws/cost_and_usage.py --granularity daily --group SERVICE | \
  python forecast.py --date-column PeriodStart --value-column Cost --method hw --hw-beta 0.3
```

### ❓ "Should we be worried about next month's costs?"

```bash
# Pipeline: Historical data → Forecast → Risk assessment
python aws/cost_and_usage.py --granularity daily | \
  python forecast.py --date-column PeriodStart --value-column Cost --ensemble --milestone-summary
```

---

## The Power of Composition

Each tool does **one thing well**, but together they answer **complex questions**:

- `cost_and_usage.py` → Gets your data
- `forecast.py` → Predicts the future  
- `anomaly_detection_forecast.sh` → Finds the unexpected
- `budget_analysis.py` → Compares against targets

**Compose them differently = Answer different questions**

---

## Let's Answer a Real Question

**"How do I know if my cost patterns are predictable enough to budget accurately?"**

We'll test this by:
1. **Creating realistic cost scenarios** (flat, growing, spiky)
2. **Running the same forecasting pipeline** on each
3. **Seeing which patterns are predictable** and which need attention

*This answers the fundamental FinOps question: "Can I trust my forecasts?"*

### Algorithms in plain language

Here's what each method tries to do, in simple terms:
- **SMA** (Simple Moving Average): Average of the last N points. Very stable, slow to react.
- **ES** (Exponential Smoothing): Weighted average that favors recent data. Reacts faster than SMA.
- **Holt‑Winters** (Triple Exponential Smoothing): Tracks level and trend (and seasonality if present). Good when there's a slope and repeating patterns.
- **ARIMA**: Statistical model for short‑term autocorrelation. Can capture persistence and mean‑reversion without seasonality.
- **SARIMA**: ARIMA with seasonality. Useful if there's a repeating weekly/monthly pattern.
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

**Question**: "Are flat cost patterns predictable?"

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
    <h2>Flat Cost Pattern</h2>
    <span>${tableInput(flat)}</span>
  </div>
    <div class="card">
    <span>${resize((width) => autoGraphInput(flat, {width}))}</span>
  </div>
</div>

**Tool Pipeline**: 
```bash
python forecast.py \
  --input demo/input/daily_flat.csv \
  --date-column PeriodStart --value-column Cost \
  --ensemble > demo/out/daily_flat_forecasts.csv
```

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Forecast Results</h2>
    <span>${tableOutput(flatOut)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(flatOutPivoted, {width}))}</span>
  </div>
</div>

**Answer**: ✅ **Highly Predictable** - Every forecasting method handles flat patterns perfectly. You can budget with confidence.

**Next Steps**: Use SMA or ES for simple, stable forecasts on flat cost patterns.

## 2) Growing a little each day

**Question**: "Are growing cost patterns predictable?"

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

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Growing Cost Pattern</h2>
    <span>${tableInput(growth)}</span>
  </div>
    <div class="card">
    <span>${resize((width) => autoGraphInput(growth, {width}))}</span>
  </div>
</div>

**Tool Pipeline**:
```bash
python forecast.py \
  --input demo/input/daily_growth.csv \
  --date-column PeriodStart --value-column Cost \
  --ensemble > demo/out/daily_growth_forecasts.csv
```

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecast Results</h2>
    <span>${tableOutput(growthOut)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(growthOutPivoted, {width}))}</span>
  </div>
</div>

**Answer**: ✅ **Very Predictable** - Linear growth is well-captured by trend-aware methods. Holt-Winters and Prophet excel.

**Next Steps**: Use Holt-Winters or Prophet for growth patterns. Monitor trend acceleration.

## 3) Flat with spikes (≤10% daily)

**Question**: "Are spiky cost patterns predictable?"

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

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Spiky Cost Pattern (10% spikes)</h2>
    <span>${tableInput(flatSpikes10)}</span>
  </div>
    <div class="card">
    <span>${resize((width) => autoGraphInput(flatSpikes10, {width}))}</span>
  </div>
</div>

**Tool Pipeline**:
```bash
python forecast.py \
  --input demo/input/daily_flat_spikes_10.csv \
  --date-column PeriodStart --value-column Cost \
  --ensemble > demo/out/daily_flat_spikes_10_forecasts.csv
```

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecast Results</h2>
    <span>${tableOutput(flatSpikes10Out)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(flatSpikes10OutPivoted, {width}))}</span>
  </div>
</div>

**Answer**: ⚠️ **Moderately Predictable** - Small spikes (≤10%) are handled reasonably well. Ensemble methods smooth out the noise.

**Next Steps**: Use ensemble forecasts for spiky patterns. Set up anomaly detection alerts for spikes >15%.

## 4) Flat with spikes (≤40% daily) 5% chance of happening

**Question**: "What about larger spikes?"

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

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Spiky Cost Pattern (40% spikes)</h2>
    <span>${tableInput(flatSpikes40)}</span>
  </div>
    <div class="card">
    <span>${resize((width) => autoGraphInput(flatSpikes40, {width}))}</span>
  </div>
</div>

**Tool Pipeline**:
```bash
python forecast.py \
  --input demo/input/daily_flat_spikes_40.csv \
  --date-column PeriodStart --value-column Cost \
  --ensemble > demo/out/daily_flat_spikes_40_forecasts.csv
```

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecast Results</h2>
    <span>${tableOutput(flatSpikes40Out)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(flatSpikes40OutPivoted, {width}))}</span>
  </div>
</div>

**Answer**: ❌ **Poorly Predictable** - Large spikes (≥40%) significantly impact forecast accuracy. Models struggle to predict these events.

**Next Steps**: Implement real-time monitoring and alerts. Consider scenario-based planning rather than point forecasts.

## 5) Growth with spikes (≤33% daily) 15% chance of happening

**Question**: "What about growth patterns with spikes?"

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
    <h2>Growth + Spikes Pattern</h2>
    <span>${tableInput(growthSpike33)}</span>
  </div>
    <div class="card">
    <span>${resize((width) => autoGraphInput(growthSpike33, {width}))}</span>
  </div>
</div>

**Tool Pipeline**:
```bash
python forecast.py \
  --input demo/input/daily_growth_spikes_33.csv \
  --date-column PeriodStart --value-column Cost \
  --ensemble > demo/out/daily_growth_spikes_33_forecasts.csv
```

<div class="grid grid-cols-1">
  <div class="card">
    <h2>Forecast Results</h2>
    <span>${tableOutput(growthSpike33Out)}</span>
  </div>
  <div class="card">
    <span>${resize((width) => autoGraphOut(growthSpike33OutPivoted, {width}))}</span>
  </div>
</div>

**Answer**: ⚠️ **Challenging** - Growth + spikes creates the most complex pattern. Ensemble methods provide the best balance.

**Next Steps**: Use ensemble forecasts with wider confidence intervals. Implement both trend monitoring and spike detection.

---

## The Toolkit Advantage

### ✅ **Small & Focused**
- Each tool does one thing well
- Easy to understand, modify, and extend
- Low cost to build and maintain

### ✅ **Composable**  
- Chain tools together for complex analysis
- Mix and match based on your question
- Adapt the pipeline to your context

### ✅ **Context-Aware**
- Works with your data format
- Adapts to your business rules
- Answers your specific questions

### ✅ **Right Here, Right Now**
- No waiting for platform updates
- No vendor lock-in
- Immediate answers to immediate questions

---

## 🚀 **This Demo: Built with Observable**

This entire dashboard is created using **[Observable](https://observablehq.com)** - a powerful tool for building interactive data dashboards fast.

**Why Observable for FinOps dashboards?**
- ✅ **Rapid Prototyping**: Build dashboards in minutes, not weeks
- ✅ **Interactive Visualizations**: Dynamic charts that respond to your data
- ✅ **Code-First**: Version control your dashboards like any other code
- ✅ **Collaborative**: Share dashboards with your team instantly
- ✅ **Composable**: Mix data sources, charts, and controls seamlessly

**This dashboard demonstrates:**
- Real-time data loading from CSV files
- Interactive table inputs with filtering
- Dynamic charts that resize and update
- Pivot transformations for forecast comparison
- Responsive grid layouts

*Observable is perfect for FinOps teams who need to create dashboards quickly and iterate based on changing requirements.*

---

## 🔮 **The Toolkit Vision: Tools We Could Build**

The beauty of the composable toolkit approach is that **anyone can extend it**. Here are tools we could build next:

### **Cost Optimization Tools**

#### ❓ "What's my biggest waste opportunity?"
```bash
# Pipeline: Cost analysis → Resource utilization → Optimization recommendations
python aws/cost_and_usage.py --granularity daily --group SERVICE | \
  python aws/resource_utilization.py --threshold 20 | \
  python aws/optimization_recommender.py --potential-savings
```

#### ❓ "Which resources are over-provisioned?"
```bash
# Pipeline: Resource metrics → Utilization analysis → Right-sizing suggestions
python aws/ec2_metrics.py --instances | \
  python aws/utilization_analyzer.py --cpu-threshold 30 --memory-threshold 50 | \
  python aws/rightsizing_recommender.py --cost-impact
```

### **Multi-Cloud & Hybrid Tools**

#### ❓ "How do my cloud costs compare?"
```bash
# Pipeline: Multi-cloud cost collection → Normalized comparison → Cost allocation
python aws/cost_and_usage.py --granularity daily | \
  python gcp/cost_and_usage.py --granularity daily | \
  python azure/cost_and_usage.py --granularity daily | \
  python cloud/cost_comparison.py --normalize-by-region --allocation-method fair-share
```

#### ❓ "Should I move this workload to another cloud?"
```bash
# Pipeline: Workload analysis → Cost modeling → Migration calculator
python aws/workload_analyzer.py --workload-id production-api | \
  python cloud/cost_modeler.py --target-cloud gcp --instance-mapping | \
  python cloud/migration_calculator.py --include-transfer-costs --roi-timeline 12
```

### **Advanced Analytics Tools**

#### ❓ "What's driving my cost variance?"
```bash
# Pipeline: Cost variance analysis → Root cause detection → Attribution modeling
python aws/cost_and_usage.py --granularity daily --group SERVICE | \
  python analytics/variance_analyzer.py --baseline-period last-month | \
  python analytics/attribution_modeler.py --drivers usage,price,volume | \
  python analytics/root_cause_detector.py --confidence-threshold 0.8
```

#### ❓ "How do I forecast capacity needs?"
```bash
# Pipeline: Usage forecasting → Capacity planning → Resource provisioning
python aws/usage_metrics.py --metrics cpu,memory,storage --granularity daily | \
  python forecast.py --date-column timestamp --value-column cpu_utilization --method prophet | \
  python capacity/capacity_planner.py --target-utilization 70 --buffer-days 30 | \
  python capacity/provisioning_scheduler.py --auto-scale
```

### **Business Intelligence Tools**

#### ❓ "What's my cost per customer/feature?"
```bash
# Pipeline: Cost allocation → Business metrics → Unit economics
python aws/cost_and_usage.py --granularity daily --group TAG --tag-key Customer | \
  python allocation/cost_allocator.py --method activity-based --drivers api-calls,storage-gb | \
  python business/unit_economics.py --revenue-column monthly_revenue --margin-analysis
```

#### ❓ "How do costs correlate with business metrics?"
```bash
# Pipeline: Business metrics → Cost correlation → Impact analysis
python business/metrics_collector.py --metrics active-users,api-calls,revenue | \
  python analytics/correlation_analyzer.py --cost-data aws-costs.csv | \
  python analytics/impact_analyzer.py --predictor-model regression --confidence-intervals
```

### **Automation & Alerting Tools**

#### ❓ "How do I automate cost governance?"
```bash
# Pipeline: Cost monitoring → Policy enforcement → Automated actions
python aws/cost_and_usage.py --granularity daily | \
  python governance/policy_engine.py --rules cost-increase-threshold,resource-tagging | \
  python automation/action_executor.py --actions email-alert,resource-stop,schedule-review
```

#### ❓ "How do I set up smart alerts?"
```bash
# Pipeline: Anomaly detection → Alert routing → Escalation management
python aws/anomaly_detection_forecast.sh --threshold 20 --method ensemble | \
  python alerts/alert_router.py --channels slack,email,pagerduty --severity-based | \
  python alerts/escalation_manager.py --timeout 4h --escalate-to manager
```

---

## 🛠️ **Building Your Own Tools**

### **The Toolkit Philosophy**

Every tool follows these principles:

1. **Single Responsibility**: One tool, one job
2. **Standard I/O**: CSV/JSON input, CSV/JSON output
3. **Composable**: Tools chain together with pipes
4. **Configurable**: Command-line flags for different contexts
5. **Testable**: Clear inputs/outputs make testing easy

### **Example: Building a Custom Tool**

Want to build a tool that finds your most expensive days? Here's how:

```python
#!/usr/bin/env python3
# expensive_days.py - Find days with highest costs

import pandas as pd
import argparse
import sys

def find_expensive_days(data, threshold_percentile=90):
    """Find days above the specified cost percentile"""
    threshold = data['Cost'].quantile(threshold_percentile / 100)
    expensive_days = data[data['Cost'] > threshold].copy()
    expensive_days = expensive_days.sort_values('Cost', ascending=False)
    return expensive_days

def main():
    parser = argparse.ArgumentParser(description='Find expensive cost days')
    parser.add_argument('--threshold-percentile', type=float, default=90,
                       help='Percentile threshold (default: 90)')
    parser.add_argument('--input', help='Input CSV file (default: stdin)')
    parser.add_argument('--output-format', choices=['csv', 'json'], default='csv')
    
    args = parser.parse_args()
    
    # Read data from file or stdin
    if args.input:
        data = pd.read_csv(args.input)
    else:
        data = pd.read_csv(sys.stdin)
    
    # Find expensive days
    expensive_days = find_expensive_days(data, args.threshold_percentile)
    
    # Output results
    if args.output_format == 'json':
        print(expensive_days.to_json(orient='records', date_format='iso'))
    else:
        print(expensive_days.to_csv(index=False))

if __name__ == '__main__':
    main()
```

**Usage:**
```bash
# Find your top 10% most expensive days
python aws/cost_and_usage.py --granularity daily | \
  python expensive_days.py --threshold-percentile 90

# Chain with forecasting to predict expensive days
python expensive_days.py --threshold-percentile 90 | \
  python forecast.py --date-column PeriodStart --value-column Cost
```

### **Observable Dashboard Integration**

Your custom tools can feed directly into Observable dashboards:

```js
// Load data from your custom tool pipeline
const expensiveDays = FileAttachment("data/expensive_days.csv").csv({typed: true});

// Create interactive visualization
Plot.plot({
  marks: [
    Plot.ruleY([0]),
    Plot.barY(expensiveDays, {
      x: "PeriodStart",
      y: "Cost",
      fill: "red",
      tip: true
    })
  ]
})
```

---

## Try It Yourself

**Pick your question, build your pipeline:**

```bash
# Your data + Our tools = Your answer
cat your-costs.csv | python forecast.py --date-column your_date --value-column your_cost
```

**Current Examples:**
```bash
# Pipeline 1: Cost Analysis → Forecasting → Budget Alerts
python aws/cost_and_usage.py --granularity daily --group SERVICE | \
  python forecast.py --date-column PeriodStart --value-column Cost --ensemble | \
  python aws/budget_analysis.py --budget-name "Monthly-Budget" --threshold 80

# Pipeline 2: Anomaly Detection → Forecasting → Optimization
python aws/anomaly_detection_forecast.sh --threshold 20 | \
  python forecast.py --date-column PeriodStart --value-column Cost --method prophet

# Pipeline 3: Custom Data → Forecasting → Dashboard
cat custom-metrics.csv | \
  python forecast.py --date-column date --value-column metric_value --output-format csv | \
  # → Feed into Observable dashboard
```

**Future Examples (when tools are built):**
```bash
# Multi-cloud cost comparison
python aws/cost_and_usage.py --granularity daily | \
  python gcp/cost_and_usage.py --granularity daily | \
  python cloud/cost_comparison.py --normalize-by-region

# Capacity planning with auto-provisioning
python aws/usage_metrics.py --metrics cpu,memory | \
  python forecast.py --date-column timestamp --value-column cpu_utilization | \
  python capacity/capacity_planner.py --target-utilization 70 | \
  python capacity/provisioning_scheduler.py --auto-scale
```

*Ready to build your own FinOps toolkit? [Start here](https://github.com/fcontrepois/finops-toolkit)*

*Want to contribute new tools? Check our [coding standards](https://github.com/fcontrepois/finops-toolkit/blob/main/documentation/CODING_STANDARDS.md) and submit a PR.*

# Appendix

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
