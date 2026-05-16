---
name: ttt-strategy
description: Use when working on the Taiwan Trend Tracking Strategy (TTT), a Taiwan stock right-side trend system for backtesting, scanners, screenshot stock analysis, entry/add/hold advice, or UI/debug explanations in the Windows stock app. Applies when the user mentions TTT, 台股右側趨勢系統, 台股低頻趨勢, 適合進場, 加碼時機, TTT 掃描, or TTT 回測.
---

# TTT Strategy Skill

Use this skill when changing, explaining, testing, or extending the TTT strategy in the Windows stock app.

## Core Workflow

1. Keep single-symbol backtest, scanner, and screenshot analysis consistent.
   - All three surfaces should use the same latest-decision logic.
   - If scanner and single-symbol results disagree, inspect the shared decision helper first.

2. Preserve the strategy identity in results.
   - Strategy Version: `TTT v2.0`
   - Engine: `Position-Based`
   - Exit Mode: `Close Confirm`
   - Position style: `Core/Mobile`

3. Use Traditional Chinese for user-facing stock names, labels, reasons, and summaries.
   - Resolve Taiwan stock names through the app's stock-name resolver.
   - Do not fall back to English names unless no official/local Chinese mapping exists.

4. Treat TTT as market-structure-first.
   - Do not reduce it to a fixed MA20 or volume-ratio-only strategy.
   - Consider limit-up/limit-down behavior, strong close/weak close, turnover, volatility, and liquidity.

5. Before editing implementation, read the current code paths:
   - `stock_backtester/strategies.py` for `run_ttt_backtest`.
   - `app.py` for `_ttt_latest_decision`, TTT scanner, TTT screenshot analysis, and UI rendering.
   - `tests/test_strategies.py` for regression coverage.

## Strategy Reference

For detailed rules, read `references/ttt-v2.md`.

Use the reference when implementing or reviewing:

- Entry rules
- Add rules
- Reduce/exit rules
- Scanner sorting/filtering
- "是否適合進場" and "加碼時機" output
- Debug explain text
- Portfolio/scanner consistency
- Regression tests for examples such as 6217, 2337, 3016, and high-volatility Taiwan stocks

## Validation Checklist

After edits, verify:

- Single-symbol query, scanner, and screenshot analysis use the same TTT decision.
- Scanner rows use close data from the stable close policy, not live intraday noise.
- User-facing text is Traditional Chinese.
- The app does not freeze during long scans or LLM screenshot analysis.
- Strategy result shows version/engine/exit mode.
- Sorting/filtering changes do not mutate the original scanner result order unless explicitly requested.

