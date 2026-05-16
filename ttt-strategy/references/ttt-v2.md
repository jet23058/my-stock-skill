# TTT v2 Reference

TTT means Taiwan Trend Tracking Strategy. Current app naming: 台股右側趨勢系統 v2.

## Goal

TTT is designed to:

- Catch right-side trend moves in Taiwan stocks.
- Avoid overreacting to single-day MA20 breaks.
- Keep a core position in strong Taiwan stocks.
- Reduce liquidity and distribution risk from limit-up, limit-down, disposition-like, and high-volatility behavior.
- Provide consistent conclusions for single-symbol query, scanner, and screenshot analysis.

## User-Facing Result Fields

Always expose:

- 是否適合進場
- 加碼時機
- Strategy Version
- Engine
- Exit Mode
- Position style
- Strategy conclusion in Traditional Chinese

Recommended version block:

```text
Strategy Version: TTT v2.0 | Engine: Position-Based | Exit Mode: Close Confirm | Position: Core/Mobile
```

## Data And Indicators

Required price data:

- open
- high
- low
- close
- volume

Derived indicators:

- MA5
- MA10
- MA20
- MA60
- 20-day volume average
- 5-day volume average
- prior 10-day high
- prior 20-day high
- 20-day low
- 10-day low
- turnover
- 20MA distance
- 20-day volume ratio
- 5-day volume ratio

Stable close policy:

- During Taiwan market intraday hours, scanner should prefer the previous stable trading day's close.
- Do not let live opening or intraday fluctuation make scanner results drift unless the app explicitly supports intraday mode.

## Stock Name Rule

Taiwan stock names must be Traditional Chinese.

Resolution order:

1. Memory cache
2. Local cache file
3. TWSE / TPEx official mapping
4. Fallback table
5. Symbol only as last resort

Normalize symbols before lookup:

```python
2330.TW -> 2330
6147.TWO -> 6147
2330 -> 2330
```

## Position Model

TTT is position-based:

```text
if no position:
    evaluate entry
else:
    manage current position
```

Do not create duplicate trades while already holding a position. Later signals become add/reduce/hold warnings.

Position stages:

| Stage | Purpose | Size |
| --- | --- | --- |
| Initial breakout | Establish position | 30% |
| Confirmation | Add after follow-through | +30% |
| Main trend | Add during strong profit trend | +40% |

Core/Mobile idea:

- Core position is not sold lightly.
- Mobile position can be reduced for risk control.

## Entry Logic

Initial entry is market-structure-first.

Core requirements:

```text
MA5 > MA10 > MA20
close > recent breakout / prior 20-day high
not weak close
turnover >= minimum daily turnover
```

Weak close examples:

- Breakout candle closes near the low.
- Long upper shadow.
- Fails to hold the breakout area.

Volume is supporting evidence, not the only rule.

Avoid pure "volume ratio only" decisions. In Taiwan stocks, limit-up locking, disposition-like behavior, and liquidity gaps can make volume ratio misleading.

## Add Logic

Add only when already holding and trend confirms.

Confirmation add:

```text
limit-up locked / re-locked
or close > prior 10-day high and close is strong
```

Main-trend add:

```text
profit >= 20%
and close is not weak
```

加碼 output should separate:

- 可加碼觀察
- 等待加碼
- 不適合加碼

Typical "not add" reasons:

- Not currently holding under strategy.
- Price is too far above MA20.
- Weak close or distribution risk.
- No clear continuation/relock structure.

## Reduce And Exit Logic

TTT does not sell all only because price dips below MA20 once.

Layered risk control:

1. First MA20 close break:
   - Reduce 30%.
   - Observe whether price can recover.

2. Fail to recover after 3 days:
   - Reduce more.

3. Long failure plus structure break:
   - Exit.

Exit is close-confirm based:

```text
close-confirmed weakness -> next open execution where applicable
```

Danger signals:

- Limit-down locked / liquidity death
- Massive volume plus breakdown
- Cannot recover structure low
- Weak rebound after breakdown

High profit protection:

- Profit >= 60% and below MA10 with high-risk structure: reduce.
- Profit >= 100% and below max(MA10, 10-day low): exit.

## Latest Decision Logic

The shared latest decision should answer:

- Is it suitable for entry now?
- Is it suitable for add now?
- What are the reasons?
- What is the latest market structure?
- What is the current distance to MA20?
- What is 20-day and 5-day volume ratio?
- What is turnover?

Single-symbol query, scanner, and screenshot analysis must call the same helper or equivalent shared logic.

## Scanner Requirements

Scanner should:

- Use Taiwan top-N universe with daily cache.
- Use stable close data.
- Return only rows evaluated by the same TTT decision logic.
- Display Traditional Chinese stock names.
- Allow filtering by add status and volume-ratio threshold.
- Preserve original strategy ranking unless user sorts a column.

Sortable scanner columns:

- 5MA
- 20MA
- 60MA
- 20日量比
- 5日量比
- 價量
- 成交金額
- 20MA 乖離

Click cycle:

```text
ascending -> descending -> original strategy order
```

## Screenshot Analysis Requirements

Screenshot analysis should:

- Support pasted images and multiple images.
- Store image groups for re-analysis.
- Store all extracted symbols in its own history page.
- Display results in-page, not only popup.
- Use the same TTT latest decision as scanner and single-symbol query.
- Use the same LLM provider/model/API key settings as the investment committee module when configured.

If LLM fails:

- Show a clear Traditional Chinese error.
- Avoid blocking the UI indefinitely.
- Preserve any symbols already extracted or analyzed.

## Debug Explain

A debug explanation should be able to answer:

- Why did the strategy buy?
- Why did it not buy?
- Why did it add or not add?
- Why did it reduce or exit?
- Which guard blocked a signal?
- Which data date and close price were used?
- Whether stable close policy affected the scan.

## Regression Examples

Use examples as behavioral anchors:

- 6217: should catch at least a meaningful part of the main trend and use high-profit protection.
- 2337: should avoid repeated whipsaw entries after the main move.
- 3016: should avoid weak rebound entries and late overheated chase entries.
- High-volatility Taiwan stocks: should account for limit-up/limit-down and liquidity behavior.

