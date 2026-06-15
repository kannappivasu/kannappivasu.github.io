# BTC 15m Paper Trade Runbook

## Strategy

- Market: BTC futures
- Timeframe: 15m
- Bias filter: 1h RSI trend filter
- Entry style: 15m mean reversion
- Entry type: maker-first limit orders
- Timeout: cancel after 2 candles if not filled
- No market-chasing fallback for the base paper setup

## Exact rule set

- Long only when `1h RSI > 55`, `15m RSI < 28`, `close < lower Bollinger band`, and `ADX < 28`.
- Short only when `1h RSI < 62`, `15m RSI > 72`, `close > upper Bollinger band`, and `ADX < 28`.
- Skip entries when spread is above `0.30%`.
- Skip entries when volatility is above `1.05%`.
- Skip entries after an impulse candle for 2 bars.
- Reject the trade if expected edge cannot cover fees plus slippage buffer.
- Position size is scaled down in high-volatility regimes.
- Stop trading for the day after a `3%` realized daily loss.
- Stop trading for the week after a `7%` realized weekly loss.
- Stop after 3 consecutive losses.

## Daily log

Record these fields every day:

- Date
- Side
- Signal time
- Entry time
- Exit time
- Entry price
- Exit price
- Spread at entry
- Slippage at entry
- Fees paid
- Funding paid
- Gross PnL
- Net PnL
- MAE
- MFE
- Reason for entry
- Reason for exit
- Notes on missed fills or abnormal behavior

## Weekly review

- Total trades
- Win rate
- Average win / average loss
- Max drawdown
- Worst trade
- Consecutive losses
- Missed entries
- Average slippage versus the backtest assumption
- Any regime where losses cluster

## Pause rules

Pause paper trading if any of these happen:

- Slippage is materially worse than the backtest assumption for several trades in a row.
- The strategy hits the daily or weekly loss cap.
- Fill quality is poor enough that maker entries are no longer realistic.
- The live curve is negative while the same period looked positive in the backtest.

## Continue rules

Continue paper trading if:

- Live paper results stay directionally close to the backtest.
- Slippage remains near or below the tested threshold.
- Losses stay bounded and recover normally.
- The strategy keeps trading enough to be judged fairly.

## Go-live gate

Do not go live yet unless all of these are true:

- At least 30 paper trades have been logged.
- Net paper PnL is positive after fees and estimated slippage.
- Average live slippage is close to or better than `0.05%`.
- The drawdown profile is still tolerable.
- No hidden venue issue shows up in fills, funding, or order handling.

