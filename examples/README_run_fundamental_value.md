# Fundamental Value Strategy Runner

This guide explains what the `FundamentalValueStrategy` does and how to execute it
through the helper script `examples/run_fundamental_value.py` for both
backtesting and live (paper) trading.

## Strategy Overview

`FundamentalValueStrategy` is a Graham-style value investing example that fuses
Lumibot's lifecycle hooks with Yahoo Finance fundamentals retrieved via
`yfinance`. Key characteristics:

- **Ticker universe** – Symbols are listed in
  `lumibot/example_strategies/config/fundamental_value.yaml`. You can supply an
  alternate YAML file with the `--config` flag; each entry should just be a
  ticker symbol.
- **Fundamental fetch** – For every symbol, the strategy downloads updated
  fundamentals using `yfinance.Ticker.get_info()`. The results are cached for 24
  hours by default (`fundamental_refresh_hours` parameter) to limit repeated API
  calls.
- **Metrics tracked** – EPS (trailing or forward), P/E ratio, book value,
  dividend yield, and the latest market price. Market price is sourced from the
  active Lumibot data feed when possible, falling back to Yahoo's pricing fields.
- **Intrinsic value calculation** – Uses the classic Graham number: `sqrt(22.5 *
  EPS * Book Value)`. A margin of safety can be applied (defaults to 0%).
- **Signals** – Per the original requirement, *buy* when intrinsic value is
  lower than the current market price; otherwise *sell*. When buying, the
  strategy targets an equal-weight allocation (or `target_allocation` parameter
  if supplied). Sell signals liquidate the entire position for that symbol.
- **Order sizing** – Buys attempt to reach the target value per symbol given the
  latest portfolio value while respecting available cash.

## Running the Strategy

All examples assume Lumibot is installed in editable mode (e.g. `pip install -e
.`) and dependencies such as `yfinance` and `PyYAML` are available in your
environment.

### Backtesting

Use Yahoo historical data via the helper script:

```bash
python examples/run_fundamental_value.py --backtest \
    --config lumibot/example_strategies/config/fundamental_value.yaml \
    --start 2023-01-01 \
    --end 2023-12-31 \
    --margin 0.2
```

Important flags:

- `--config`: Optional path to your YAML watch list. Omit to use the bundled
  file.
- `--start` / `--end`: Backtest window (defaults to the 2023 calendar year if
  not provided).
- `--margin`: Margin of safety (0.0–1.0). A value of `0.2` discounts intrinsic
  value by 20%.
- `--allocation`: Target fraction of the portfolio to allocate to each symbol
  that triggers a buy. Defaults to equal weighting.

The script prints the backtest summary returned by Lumibot. Additional reports
(such as tearsheets) can be enabled by modifying the script or strategy.

### Live / Paper Trading

To run live or on Alpaca paper accounts:

```bash
python examples/run_fundamental_value.py --live \
    --config lumibot/example_strategies/config/fundamental_value.yaml \
    --allocation 0.35
```

Requirements:

1. Populate valid Alpaca credentials in `lumibot/credentials/ALPACA_CONFIG` or
   via environment variables according to Lumibot's documentation.
2. Ensure the account supports every ticker in your YAML watch list.
3. Keep the process running; press `Ctrl+C` to stop.

The script instantiates an Alpaca broker, builds the strategy with any supplied
parameters, and calls `run_live()`. Positions and orders will appear in your
Alpaca dashboard during execution.

## Customization Tips

- **Watch list** – Duplicate the default YAML, edit the ticker list, and point
  `--config` to your custom file.
- **Refresh cadence** – Adjust `fundamental_refresh_hours` via the `parameters`
  dictionary inside the script if you need more frequent or less frequent data
  refreshes.
- **Signal logic tweaks** – Edit `lumibot/example_strategies/fundamental_value.py`
  if you want to flip the buy/sell rule or incorporate additional filters.

With these steps you can evaluate the example strategy quickly and adapt it to
your own preferences.
