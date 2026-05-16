# Taiwan / US Stock Strategy Backtester

This is a Windows desktop app built with Python and Tkinter. It accepts a US or Taiwan stock symbol, downloads daily candles from Yahoo Finance, and runs a backtest with one of two strategy presets:

- Livermore trend breakout
- Minervini Trend Template
- MA 50/200 Golden Cross
- RSI Mean Reversion
- Bollinger Breakout
- Turtle 20/10 Breakout
- Composite relative strength + Livermore + support add-on

## Run

```powershell
python app.py
```

No third-party Python packages are required. You only need Python 3.10 or newer with Tkinter enabled. The standard Python installer from python.org includes Tkinter.

You can also double-click `run_app.bat` on Windows.

## Symbols

- US stocks: `AAPL`, `MSFT`, `NVDA`
- Taiwan stocks: `2330`, `0050`, `2317`
- Auto mode tries `.TW`, `.TWO`, then the raw symbol for numeric inputs.

## Strategy Rules

Livermore approximation:

- Buy: close breaks above the prior 55-day high, close is above the 50-day moving average, and volume is at least 1.25x the 20-day average.
- Sell: close breaks below the prior 21-day low, falls below the 50-day moving average, or hits an 8% stop.

Minervini approximation:

- Buy: price satisfies a Trend Template filter, breaks above a 20-day pivot, and volume is at least 1.4x the 50-day average.
- Sell: 7% stop, close below 50-day moving average, or a large pullback after gains.

Other included strategies:

- MA 50/200 Golden Cross: buy when the 50-day average crosses above the 200-day average.
- RSI Mean Reversion: buy after RSI falls below 30 and price turns up.
- Bollinger Breakout: buy when price closes above the upper Bollinger Band with volume expansion.
- Turtle 20/10 Breakout: buy on a 20-day high breakout and exit on a 10-day low breakdown.
- Composite relative strength + Livermore + support add-on: requires relative strength, Livermore-style breakout, recent support retest, 20-day risk control, and 60-day medium-term uptrend to all pass before a buy signal is generated.

Chart overlays:

- Add-on levels: prior 20-day high and prior 55-day high.
- Support levels: 20-day low, 50-day moving average, and 60-day low.
- Use the chart checkboxes to show or hide price, buy/sell signals, add-on levels, support levels, and each strategy equity curve.

Entry scanner:

- Open the "進場掃描" tab.
- Paste one stock symbol per line.
- The custom symbol list is saved to `scan_symbols.txt` and loaded again the next time the app opens.
- Choose "TrendGuard 每日掃描" as the stock source to scan symbols from `https://raw.githubusercontent.com/jet23058/TrendGuard/data/daily_scan_results.json`.
- Use "載入資料來源到清單" to copy the selected source into the manual list for editing.
- Use "掃描台股熱門清單" or "掃描美股熱門清單" to scan built-in Taiwan or US watchlists without replacing your custom list.
- The Taiwan market scan fetches the Taiwan Stock Exchange list sorted by market cap and scans the top 1000 when the online list is available.
- Scan results include stock symbol and stock name, and are grouped into separate tabs by strategy.
- The scanner also includes an "適合加碼" tab. It lists stocks whose latest signal is still buy and whose price is near or newly above an add-on breakout level.

AI investment committee:

- Open the "AI 投資委員會" tab for an AI Hedge Fund inspired Windows-app workflow.
- The app runs deterministic local agents: technicals, Livermore, composite strategy, risk manager, and portfolio manager.
- The result table shows each agent signal, final decision, confidence, entry/add-on price, stop-loss point, and detailed reasoning.
- This version does not call an LLM. It uses the local backtest and risk rules so decisions are reproducible.
- The scanner uses the strategies currently checked on the single-stock backtest tab.
- Results only include symbols whose latest strategy signal is still a buy signal.
- The stop-loss point uses the nearest current support below price, with an 8% risk stop as a fallback.
- Candidates that are too extended from the entry point or have excessive stop-loss risk are filtered out.

Right-side trade filters:

- Long upper shadow: filters out failed intraday pushes and sharp reversals.
- Distance to MA20 / MA60: filters out stocks that are too extended from short- and medium-term moving averages.
- Close position in range: requires the latest close to stay in the stronger half of the recent 20-day range.
- Volume spike: filters out high-volume black candles that may indicate distribution or heavy churn.
- PE filter: reserved as a valuation warning until a reliable fundamentals source is connected, so it is currently not a hard block.
- Risk/reward ratio: requires estimated upside versus stop-loss risk to be at least 1.5.

These rules are executable approximations, not a complete replacement for discretionary reading of Livermore or Minervini methods.

Note: a 1-year backtest has about 240 to 250 trading days. Livermore can trade with that range, while Minervini's 200/252-day filters may produce few or no trades until there is enough warm-up data.

## Test

```powershell
python -m unittest
```
