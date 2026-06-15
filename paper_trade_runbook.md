# BTC 15m Mean Reversion v1 Paper Trade Runbook

## Status

- Active strategy version: `btc_15m_mean_reversion_v1`
- Current status: paper-trade candidate
- Live status: not live-ready
- Benchmark role: current keeper
- Execution adapter: not built yet

The strategy is frozen as `v1` in `data/strategy_v1.json`. Failed research branches stay archived as evidence, but they are not part of the paper-trade plan.

## Frozen v1 Rules

- Market: BTC perpetual futures proxy
- Timeframe: 15m
- Context: 1h trend/RSI filter
- Entry style: 15m Bollinger/RSI mean reversion
- Execution: maker-first limit order
- Limit timeout: cancel after 2 candles if not filled
- No market-chasing fallback in the base paper setup

Long entry:

- `trend_bias_1h >= 0`
- `1h RSI > 55`
- `15m RSI < 28`
- `15m close < lower Bollinger Band(20, 2)`
- `15m ADX < 28`
- volume is positive

Short entry:

- `trend_bias_1h <= 0`
- `1h RSI < 62`
- `15m RSI > 72`
- `15m close > upper Bollinger Band(20, 2)`
- `15m ADX < 28`
- volume is positive

Exit:

- Long exits when `close >= Bollinger midline` or `15m RSI > 45`.
- Short exits when `close <= Bollinger midline` or `15m RSI < 50`.
- Protective model: 2.5% stop, 1.8% target, trailing activates after 1.4% with 0.7% offset.

## Execution Checklist

Before placing a paper order:

- Confirm the signal is from `v1`, not an experimental branch.
- Record the signal row in `data/paper_signal_events_v1.csv`.
- Check spread estimate is at or below `0.30%`.
- Check volatility estimate is at or below `1.05%`.
- Check there is no impulse-candle skip.
- Place maker-first paper limit order at the model limit price.
- Cancel if not filled within 2 candles.
- Record whether the fill was full, partial, missed, or manually rejected.

After a fill:

- Record actual entry price, actual spread, actual slippage, filled fraction, fees, and order timing.
- Track exit condition every 15m candle.
- Record actual exit price, fees, funding, gross PnL, net PnL, MAE, MFE, and notes.
- Mark any execution issue explicitly; do not hide it inside notes.

## Log Files

- Strategy spec: `data/strategy_v1.json`
- Paper status: `data/paper_trade_status.json`
- Live/paper trade log: `data/paper_trade_log.csv`
- Signal event log: `data/paper_signal_events_v1.csv`
- Blank template: `data/paper_trade_log_template.csv`
- Logger script: `research/paper_signal_logger.py`
- Status builder: `research/paper_status_report.py`

Logger command:

```powershell
python .\research\paper_signal_logger.py --latest-only
python .\research\paper_status_report.py
python .\dashboard\build_dashboard.py
```

Use full-history mode only for debugging the logger, not for paper performance:

```powershell
python .\research\paper_signal_logger.py
```

## Daily Review

- New entry signals
- Skipped signals and skip reasons
- Missed or partial fills
- Expected versus actual spread
- Expected versus actual slippage
- Open position status
- Exit conditions
- Net paper PnL after fees, funding, and slippage
- Any order, API, timing, or venue issue

## Weekly Review

- Closed paper trades
- Signals per month pace
- Win rate
- Average win and average loss
- Net paper PnL
- Max paper drawdown
- Current drawdown
- Missed-fill count
- Average actual slippage versus modeled slippage
- Slippage drift versus the `0.05%` stress threshold
- Execution failures

## Pause Rules

Pause paper trading if:

- Average actual slippage is worse than `0.05%`.
- Fill quality is materially worse than the maker-first model.
- The strategy hits the modeled daily or weekly loss guard.
- There are unexplained execution failures.
- Paper drawdown exceeds the modeled range plus a 25% tolerance.
- Signals do not match the frozen `v1` rules.

## Go/No-Go Gate

Do not go live unless all are true:

- At least 30 closed paper trades, or at least 60 calendar days if trade frequency stays rare.
- Net paper PnL is positive after fees, funding, and actual slippage.
- Max paper drawdown is within the modeled range plus tolerance.
- Average actual slippage is no worse than the `0.05%` stress-tested threshold.
- No unexplained execution failures remain open.
- The execution log is complete enough to audit.

Until every gate passes, the correct status remains: paper-trade candidate.
