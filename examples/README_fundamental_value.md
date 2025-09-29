# Fundamental Value Strategy

## Overview

The **Fundamental Value Strategy** is a quantitative value investing approach that uses fundamental financial metrics to identify undervalued stocks and make buy/sell decisions. This strategy implements Benjamin Graham's intrinsic value calculation (Graham Number) to determine when stocks are trading below their fair value.

## How the Strategy Works

### Core Philosophy

This strategy is based on **value investing principles** popularized by Benjamin Graham and Warren Buffett. It seeks to:

1. **Identify undervalued stocks** by comparing market price to intrinsic value
2. **Buy when price < intrinsic value** (with optional margin of safety)
3. **Sell when price ≥ intrinsic value**
4. **Diversify across multiple value opportunities**

### Intrinsic Value Calculation

The strategy calculates intrinsic value using the **Graham Number formula**:

```
Intrinsic Value = √(22.5 × EPS × Book Value per Share)
```

Where:
- **EPS (Earnings Per Share)**: Company's earnings divided by shares outstanding
- **Book Value per Share**: Company's net worth divided by shares outstanding
- **22.5**: Graham's multiplier representing a P/E ratio of 15 and P/B ratio of 1.5

### Decision Logic

#### Buy Signal
- **Condition**: `Market Price < Intrinsic Value × (1 - Margin of Safety)`
- **Action**: Purchase shares up to target allocation
- **Allocation**: Equal weight across all buy signals (or custom allocation)

#### Sell Signal  
- **Condition**: `Market Price ≥ Intrinsic Value`
- **Action**: Sell entire position
- **Rationale**: Stock is no longer undervalued

#### Hold Signal
- **Condition**: `Market Price ≥ Intrinsic Value × (1 - Margin of Safety)` AND `Market Price < Intrinsic Value`
- **Action**: No trading (maintain current position)

### Key Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `margin_of_safety` | 0.0 | Safety buffer (0-99%) applied to intrinsic value |
| `target_allocation` | Auto | Portfolio allocation per symbol (auto = 1/N tickers) |
| `fundamental_refresh_hours` | 24 | How often to refresh fundamental data |

### Data Sources

- **Price Data**: Lumibot's configured data source (Yahoo Finance for backtesting)
- **Fundamental Data**: Yahoo Finance via `yfinance` package
- **Metrics Used**: EPS, P/E ratio, book value, dividend yield, current price

## Configuration

### Ticker Configuration

The strategy uses a YAML configuration file to specify which stocks to analyze:

```yaml
# config/fundamental_value.yaml
tickers:
  - AAPL
  - NVDA  
  - MSFT
  - GOOGL
  - AMZN
```

You can customize this file or provide a different path via the `--config` parameter.

### Strategy Parameters

```python
parameters = {
    "config_path": "path/to/your/config.yaml",  # Optional
    "margin_of_safety": 0.2,                   # 20% safety buffer
    "target_allocation": 0.1,                  # 10% per stock
    "fundamental_refresh_hours": 24,           # Daily refresh
}
```

## Usage Examples

### Backtesting

#### Basic Backtest (2024)
```bash
python examples/run_fundamental_value.py --backtest --start 2024-01-01 --end 2024-12-31
```

#### Backtest with Custom Parameters
```bash
python examples/run_fundamental_value.py \
    --backtest \
    --start 2023-01-01 \
    --end 2023-12-31 \
    --margin 0.2 \
    --config lumibot/example_strategies/config/fundamental_value.yaml
```

#### Multi-Year Backtest
```bash
python examples/run_fundamental_value.py \
    --backtest \
    --start 2020-01-01 \
    --end 2023-12-31 \
    --allocation 0.15
```

### Live Trading

#### Paper Trading (Alpaca)
```bash
python examples/run_fundamental_value.py \
    --live \
    --allocation 0.1 \
    --margin 0.15
```

#### Live Trading with Custom Config
```bash
python examples/run_fundamental_value.py \
    --live \
    --config /path/to/my/watchlist.yaml \
    --allocation 0.08 \
    --margin 0.25
```

## Setup Requirements

### 1. Environment Setup
```bash
# Activate your conda environment
conda activate trade

# Ensure Lumibot is installed
pip install -e .
```

### 2. Backtesting Setup
No additional setup required - uses Yahoo Finance data automatically.

### 3. Live Trading Setup (Alpaca)

Create a `.env` file in the project root:
```bash
# .env
ALPACA_API_KEY=your_alpaca_api_key
ALPACA_SECRET_KEY=your_alpaca_secret_key
ALPACA_PAPER=True  # Set to False for live trading
```

Or set environment variables:
```bash
export ALPACA_API_KEY="your_api_key"
export ALPACA_SECRET_KEY="your_secret_key"
export ALPACA_PAPER="True"
```

## Strategy Performance

### Expected Behavior

- **Bull Markets**: May underperform growth stocks as value stocks may lag
- **Bear Markets**: Often outperforms due to focus on undervalued, fundamentally sound companies
- **Sideways Markets**: Can generate alpha through mean reversion

### Risk Management

1. **Diversification**: Equal weight across multiple value opportunities
2. **Margin of Safety**: Optional buffer against calculation errors
3. **Fundamental Focus**: Only trades companies with solid financial metrics
4. **Position Sizing**: Limits exposure per stock based on target allocation

### Limitations

- **Data Dependency**: Relies on accurate fundamental data from Yahoo Finance
- **Lag**: Fundamental data may be delayed (quarterly reports)
- **Market Conditions**: May underperform in strong growth markets
- **Calculation Simplicity**: Uses simplified Graham Number vs. complex DCF models

## Customization

### Adding More Metrics
Extend the `FundamentalSnapshot` dataclass to include additional metrics:
```python
@dataclass
class FundamentalSnapshot:
    # ... existing fields ...
    debt_to_equity: Optional[float]
    return_on_equity: Optional[float]
    price_to_sales: Optional[float]
```

### Custom Intrinsic Value Formula
Modify the `_calculate_intrinsic_value` method:
```python
def _calculate_intrinsic_value(self, eps, book_value, revenue_growth):
    # Your custom calculation here
    return custom_value
```

### Dynamic Allocation
Override `_compute_target_allocation` for sophisticated position sizing:
```python
def _compute_target_allocation(self, symbol):
    # Risk-based or volatility-based sizing
    return calculated_allocation
```

## Troubleshooting

### Common Issues

1. **"No tickers configured"**
   - Check your YAML config file exists and has valid ticker symbols
   - Verify file permissions

2. **"Missing fundamental data"**
   - Yahoo Finance may be rate-limited
   - Some tickers may not have complete fundamental data
   - Check internet connection

3. **"ALPACA_CONFIG is not set"**
   - Set up your Alpaca credentials in `.env` file
   - Or configure environment variables

### Performance Optimization

- **Reduce `fundamental_refresh_hours`** for more frequent updates (increases API calls)
- **Increase cache TTL** for less frequent updates (may miss recent data)
- **Limit ticker list** to reduce computation and API usage

## Example Output

```
Tracking 3 ticker(s) from fundamental_value.yaml: AAPL, NVDA, MSFT
AAPL | price=150.25 | intrinsic=165.30 | eps=6.13 | pe=24.5 | book=3.61
NVDA | price=475.80 | intrinsic=420.15 | eps=4.44 | pe=107.2 | book=12.43
MSFT | price=330.45 | intrinsic=355.20 | eps=11.06 | pe=29.9 | book=18.22

Buy signals: AAPL (price < intrinsic)
Sell signals: NVDA (price > intrinsic)  
Hold signals: MSFT (within margin)
```

## Further Reading

- [Benjamin Graham's "The Intelligent Investor"](https://en.wikipedia.org/wiki/The_Intelligent_Investor)
- [Graham Number Calculation](https://www.investopedia.com/terms/g/graham-number.asp)
- [Lumibot Documentation](http://lumibot.lumiwealth.com/)
- [Alpaca Trading API](https://alpaca.markets/docs/)
