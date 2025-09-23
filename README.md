# FinOps Toolkit UI 📊

A comprehensive Financial Operations (FinOps) toolkit built with [Observable Framework](https://observablehq.com/framework/) for time series forecasting, cost analysis, and financial data visualization.

## 🚀 Features

- **Time Series Forecasting**: Multiple forecasting algorithms including SMA, Exponential Smoothing, Holt-Winters, ARIMA, SARIMA, Theta, Prophet, and NeuralProphet
- **Interactive Dashboards**: Real-time data visualization with Observable's powerful plotting capabilities
- **Cost Analysis**: Synthetic and real-world cost data analysis with spike detection
- **Ensemble Methods**: Combines multiple forecasting models for improved accuracy
- **Multiple Scenarios**: Flat baseline, growth patterns, and spike analysis
- **Responsive Design**: Mobile-friendly interface with adaptive layouts

## 🛠️ Quick Start

### Prerequisites

- Node.js 18 or higher
- npm or yarn package manager

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd finops-toolkit-ui
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Start the development server**
   ```bash
   npm run dev
   ```

4. **Open your browser**
   Visit [http://localhost:3000](http://localhost:3000) to view the application

## 📁 Project Structure

```
finops-toolkit-ui/
├── src/
│   ├── components/           # Reusable JavaScript modules
│   │   └── timeline.js      # Timeline visualization component
│   ├── data/                 # Data files and outputs
│   │   ├── input/           # Input CSV files for analysis
│   │   └── out/             # Generated forecasts and visualizations
│   └── index.md             # Main dashboard page
├── dist/                     # Built application (generated)
├── observablehq.config.js   # Framework configuration
├── package.json             # Dependencies and scripts
└── README.md                # This file
```

## 🎯 Use Cases

### 1. Cost Forecasting
- Predict future cloud costs based on historical spending patterns
- Identify cost trends and anomalies
- Plan budgets with confidence using multiple forecasting methods

### 2. Anomaly Detection
- Detect unusual spending spikes in cost data
- Analyze patterns in cost fluctuations
- Understand the impact of irregular events on budgets

### 3. Growth Analysis
- Track and forecast cost growth trends
- Model different growth scenarios
- Plan for scaling costs over time

### 4. Model Comparison
- Compare accuracy of different forecasting algorithms
- Use ensemble methods for improved predictions
- Select the best model for your specific use case

## 🔧 Available Scripts

| Command | Description |
|---------|-------------|
| `npm install` | Install or reinstall dependencies |
| `npm run dev` | Start local preview server |
| `npm run build` | Build static site to `./dist` |
| `npm run deploy` | Deploy to Observable platform |
| `npm run clean` | Clear local data loader cache |
| `npm run observable` | Run Observable CLI commands |

## 📊 Forecasting Algorithms

The toolkit includes several forecasting methods:

- **SMA (Simple Moving Average)**: Stable baseline forecasting
- **ES (Exponential Smoothing)**: Weighted recent data emphasis
- **Holt-Winters**: Level, trend, and seasonality tracking
- **ARIMA**: Statistical autocorrelation modeling
- **SARIMA**: ARIMA with seasonal components
- **Theta**: Level and trend line blending
- **Prophet**: Facebook's flexible trend modeling
- **NeuralProphet**: Neural network-based forecasting
- **Ensemble**: Average of all available models

## 🎨 Customization

### Adding New Data Sources
1. Place CSV files in `src/data/input/`
2. Update the data loading code in your pages
3. Ensure proper date and value column formatting

### Creating New Visualizations
1. Add new components in `src/components/`
2. Import and use in your Markdown pages
3. Leverage Observable's Plot library for charts

### Modifying Forecasting Models
1. Update the forecasting pipeline
2. Add new model outputs to the data processing
3. Update visualization functions to include new models

## 🚀 Deployment

### Local Development
```bash
npm run dev
```

### Production Build
```bash
npm run build
```

### Deploy to Observable
```bash
npm run deploy
```

## 📈 Data Format

The toolkit expects CSV files with the following structure:

```csv
PeriodStart,Cost
2023-01-01,100.50
2023-01-02,102.30
...
```

- **PeriodStart**: Date column (ISO format recommended)
- **Cost**: Numeric value column for analysis

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🔗 Resources

- [Observable Framework Documentation](https://observablehq.com/framework/)
- [D3.js Plot Library](https://observablehq.com/plot/)
- [FinOps Foundation](https://www.finops.org/)
- [Time Series Forecasting Best Practices](https://observablehq.com/@observablehq/time-series-forecasting)

## 📞 Support

For questions, issues, or contributions, please:
- Open an issue on GitHub
- Check the Observable Framework documentation
- Review the example data and visualizations in the app

---

**Built with ❤️ using Observable Framework**
