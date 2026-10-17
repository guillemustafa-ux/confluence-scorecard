# Confluence Scorecard

Open-source TradingView indicator pair (Pine Script v6). The overlay combines moving averages, trend candles, ADX dots, an auto Fibonacci retracement, floor pivots, swing support and resistance on two timeframes, a multi-timeframe trend card and a volume profile with a seven-vote signal engine. Its BUY and SELL signals wait for the bar to close, show every vote behind them, and are graded in a built-in scorecard against the base rate of the same take-profit and stop rule applied to every closed bar after the warm-up. A companion pane shows squeeze momentum, the squeeze state and momentum divergences.

Not published on TradingView. The draft description is in `TRADINGVIEW_PUBLICATION.md`.

## What makes it different

1. **Closed bars only.** Signals, trades, ADX dots, candle colors, support and resistance levels and the scorecard's counts change only on closed bars, and the engine reads its votes only on closed bars. Every `request.security` call offsets its expression by one bar and uses `lookahead = barmerge.lookahead_on`, so for a timeframe at or above the chart's it returns the last completed bar and never a bar that is still forming (a row or a higher timeframe set below the chart's is requested but not used). A swing level is drawn back from its pivot bar on the bar that confirms it, and its dot's tooltip says how many bars later that was.
2. **Every signal label shows its votes.** A signal label reads `BUY 5/7` or `SELL 5/7`, and its tooltip lists the seven votes by name and value, with the readings behind them (ADX and DI values, the volume ratio, how many bars ago a divergence confirmed).
3. **A scorecard against a base rate.** For each signal the script follows a hypothetical trade: did TP1, TP2, TP3 or the stop come first within H bars? The TP1 rate, with a Wilson 95% interval, is compared with the same TP1, stop and horizon applied to every closed bar after the warm-up, as a long and as a short entry. The verdict reads "better" only when the lower bound of the interval is above that base rate, "worse" only when the upper bound is below it, and "no evidence" otherwise. All results are before costs.

The point of the scorecard: a confluence of indicators says that several readings agree, not that the next move is more likely to reach a target. The base rate answers "how often does this target come before this stop anyway, from any bar?", so the signals have to beat the chart they run on, not 50%.

## Signal engine

Seven votes, each +1, 0 or -1; the score is their sum, from -7 to +7. The fast, mid and slow EMAs are `ta.ema(close, length)` with the lengths of MA 1, MA 2 and MA 3 (defaults 20, 55 and 200), whatever type the MA rows draw.

| # | Vote | +1 | -1 | Defaults |
| --- | --- | --- | --- | --- |
| 1 | EMA stack | fast > mid > slow | fast < mid < slow | 20, 55, 200 |
| 2 | Higher-timeframe trend | on the last completed bar of the higher timeframe: fast EMA > mid EMA and close > mid EMA | fast EMA < mid EMA and close < mid EMA | automatic timeframe (see below) |
| 3 | ADX | ADX >= threshold and +DI > -DI | ADX >= threshold and -DI > +DI | `ta.dmi(14, 14)`, threshold 25 |
| 4 | Squeeze momentum | momentum > 0 and above the previous bar's | momentum < 0 and below the previous bar's | length 20 |
| 5 | Volume | volume > 1.2 x SMA(volume, 20) and close > open | volume > 1.2 x SMA(volume, 20) and close < open | 20, 1.2 |
| 6 | RSI divergence | a regular bullish divergence confirmed in the window | a regular bearish divergence confirmed in the window | RSI 14, pivots 5/5, 5 to 60 bars apart, window 10 |
| 7 | Fibonacci band | up swing and the close inside the 0.5-0.786 retracement band | down swing and the close inside the band | 144 bars |

Details:

- **Vote 2.** The higher timeframe is the input in the Higher timeframe group. Empty (default) means automatic: a chart timeframe up to 15 minutes uses 60, up to 60 minutes uses 240, below 1 day uses D, below 1 week uses W, below 1 month uses M, and anything longer uses 12M. The vote is 0 when the higher timeframe is not above the chart timeframe. S/R #2 uses the same timeframe.
- **Vote 4.** Momentum = `ta.linreg(close - avg(avg(ta.highest(high, 20), ta.lowest(low, 20)), ta.sma(close, 20)), 20, 0)`: the linear regression of the close minus the average of the 20-bar midpoint and the 20-bar SMA. The oscillator pane plots the same formula.
- **Vote 5.** On a symbol with no volume the comparison fails and the vote is 0.
- **Vote 6.** Taken from `rsi-divergence-vs-random`: pivots of `ta.rsi(close, 14)` with `ta.pivothigh` and `ta.pivotlow`, 5 bars to the left and 5 to the right, so a pivot confirms 5 bars after its bar. When 5 to 60 bars lie strictly between a pivot and the previous pivot of the same kind, price lower low (`low[5]` below the previous pivot's low) with an RSI higher low is bullish, and price higher high with an RSI lower high is bearish. The previous pivot is replaced either way. A divergence votes on the bar that confirms it and the next 9 bars (window 10). When both kinds fall in the window, the more recent one votes; two on the same bar give 0.
- **Vote 7.** The swing is the auto Fibonacci swing: the highest high and the lowest low of the last 144 bars, the current one included. It is an up swing when the highest high is on a more recent bar than the lowest low, a down swing in the opposite case, and no swing (vote 0) when one bar holds both. Up swing: +1 when `high - 0.786 x range <= close <= high - 0.5 x range`. Down swing: -1 when `low + 0.5 x range <= close <= low + 0.786 x range`.
- **Warm-up.** Signals and the base rate start on the first bar where the bar index is at least the longest of the three EMA lengths and the ADX, the momentum, the RSI and the ATR all have values.

## Signals

- On a closed bar after the warm-up: **BUY** when the score is at or above the minimum score (default 4) and the previous bar's score was below it; **SELL** when the score is at or below minus the minimum and the previous bar's score was above it. The previous bar's score is its value at its close.
- **Alternate signals** (default on): no BUY while the last signal was a BUY and no SELL while it was a SELL. A crossing blocked this way is not stored; the next BUY needs a new crossing after a SELL.
- The label sits below the bar for a BUY and above it for a SELL. `BUY 5/7` means a score of +5; `SELL 5/7` a score of -5. The tooltip starts with the score and the bar's time and lists the seven votes.
- The oldest labels beyond **Labels kept** (default 100, at most 150) are deleted. The scorecard counts every signal either way.

### Safeguard (default off)

A signal becomes pending. It is confirmed by the first closed bar among the next M bars (default 3) that closes above the signal bar's high (BUY) or below its low (SELL) while the score is still at or above minimum - 1 (BUY) or at or below -(minimum - 1) (SELL). With labels shown, a small check mark appears on the confirmation bar. A pending signal lapses after M bars, and a new BUY or SELL replaces it. The signal label's tooltip gets one more line with the outcome: confirmed n bars later, not confirmed within M bars, or replaced by the next signal.

With the safeguard on, a trade opens only at the close of a confirmation bar, a signal that is not confirmed opens no trade, and the scorecard's title adds "safeguard on" (its tooltip says "safeguard on, confirmed signals only").

## Trades

- **Entry.** The close of the signal bar, or of the confirmation bar with the safeguard on.
- **Levels.** ATR mode (default): TPk = entry + mk x ATR and stop = entry - s x ATR for a long, mirrored for a short, with `ta.atr(14)` read on the entry bar; defaults 1, 2, 3 and stop 1.5. Percent mode: TPk = entry x (1 + pk / 100) and stop = entry x (1 - s / 100) for a long, mirrored for a short; its inputs default to TP1 1.0%, TP2 2.0%, TP3 3.0%, stop 1.5%. Keep TP1 < TP2 < TP3. ATR is the default because percent levels do not scale across timeframes: on BINANCE:BTCUSDT 1 minute every signal trade timed out before reaching 1% or 1.5% (and 26,762 of 27,331 base-rate trades), while on 1 day 5 of 9 signal trades ended on a bar that held both a target and the stop.
- **Order on each closed bar**, starting with the bar after the entry:
  1. An open at or beyond the stop: the trade ends at the stop.
  2. Targets the open already reached are credited, since the open is the bar's first price; TP3 ends the trade.
  3. Further targets reached by the high (long) or the low (short), and the stop reached by the other extreme. When both happen on one bar, the order inside the bar is unknown: the bar counts as a stop, the further targets are not credited, and the trade is flagged ambiguous. A stop alone ends the trade at the stop; targets alone are credited, and TP3 ends the trade.
  4. A trade that has reached neither the stop nor TP3 ends as a timeout on the H-th bar after the entry (default H = 50).
- Trades overlap: a new signal opens a new trade while earlier ones are still open. The next opposite signal does not end a trade; it only stops its drawing.
- **Drawing** (default on): an entry line (dotted gray), TP1 to TP3 (dashed teal) and the stop (dashed red) from the entry bar, extended on every closed bar until the trade ends or the next opposite BUY or SELL signal, whichever comes first. While the lines extend, marks `TP1`, `TP2`, `TP3` and `SL` appear on the bar that reaches each level; `SL*` marks an ambiguous stop. Only the last 20 trades (at most 30) keep their lines and marks.

## Scorecard

Bottom right by default (the Position input offers eight table positions; top right is left out because the trend card sits there). A short title row ("Scorecard · before costs"), one row of column titles, three signal rows and two base-rate rows. Hover the title for the symbol, the timeframe, the levels, the horizon, the safeguard state, the counting rules and the trades not counted yet; that text lives in the tooltip so that no long line widens the columns.

| Column | Meaning |
| --- | --- |
| N | Trades that have ended: at the stop, at TP3 or by timeout. Open trades are not counted; the title's tooltip gives their number. |
| TP1, TP2, TP3 | Share of N that reached the target before the stop. A trade that reached TP1 and was stopped later counts in TP1 and in Stop. |
| Stop | Share of N that ended at the stop. |
| Timeouts | Number of trades that ended by the horizon. |
| Ambiguous | Number of stops on a bar that also reached a further target. They are part of Stop. |
| TP1 95% CI | Wilson score interval for the TP1 rate. |
| Edge vs base | TP1 rate minus the base rate, in percentage points. |
| Verdict | "better" (green) when N > 0 and the lower bound of the TP1 interval is above the base rate; "worse" (red) when N > 0 and the upper bound is below it, meaning the signals reach TP1 before the stop less often than an entry on any bar; "no evidence" otherwise, including N = 0. |

Rows:

- **Long signals** and **Short signals** are compared with the long and the short base rate.
- **All signals** pools both sides and is compared with the base rate weighted by the signals' own mix: `(nLong x baseLong + nShort x baseShort) / (nLong + nShort)`.
- **Base rate, long, every bar** and **Base rate, short, every bar**: from the first bar after the warm-up, every closed bar opens a long and a short trade at its close with the same TP1, stop, horizon and bar rules, TP1 only. A base-rate trade ends at TP1, at the stop or by timeout. These rows show N, the TP1 rate, Stop, Timeouts, Ambiguous and the interval; TP2 and TP3 read "-".

Statistics:

- Wilson interval with z = 1.96, p = k / n: center `(p + z^2 / (2n)) / (1 + z^2 / n)`, half width `z x sqrt(p (1 - p) / n + z^2 / (4 n^2)) / (1 + z^2 / n)`. It stays inside [0, 1] and is usable at small n, where the normal interval is not.
- The base rate is treated as a fixed number. With one entry per bar and per side it has far more trades than the signals, so its own error is small next to the signals' interval; it is not zero, and the windows of consecutive base-rate trades overlap.

Identities that must hold on any chart (rates rounded to one decimal):

- Signal rows: TP3 + Stop + Timeouts / N = 100%; TP1 >= TP2 >= TP3 with ascending targets; Ambiguous <= the number of stops.
- Base-rate rows: TP1 + Stop + Timeouts / N = 100%.
- All signals N = Long N + Short N, and every count in the All row is the sum of the long and the short counts.

## Context layers

| Layer | Rule | Defaults |
| --- | --- | --- |
| Moving averages | Four rows, each with a switch, a type (EMA, SMA, WMA, RMA, HMA, VWMA), a length and a color. MA 1 to MA 3 lengths are also the engine's fast, mid and slow EMAs; MA 4 is display only. | EMA 20 on, EMA 55 on, EMA 200 on, EMA 100 off |
| Bollinger area | Filled band between `SMA(close, 20) - 2 x stdev(close, 20)` and `SMA(close, 20) + 2 x stdev(close, 20)`. | off |
| Trend candles | Bull color (teal) when the fast EMA is above the mid EMA and the close is above the mid EMA, bear color (red) in the mirror case, gray otherwise. Full color when the ADX is at or above the threshold, 60% transparent below it. Colored once the bar has closed; the forming bar keeps the chart's own colors. | on, ADX 14/14, threshold 25 |
| ADX dots | On the closed bar where the ADX is at or above the threshold and the previous bar's ADX was below it: a teal dot under the bar when +DI > -DI, a red dot over the bar when -DI > +DI, no dot when they are equal. | on |
| Auto Fibonacci | The swing of vote 7 over the last 144 closed bars. Level r sits at `end - r x (end - start)`, where end is the more recent extreme: 0, 0.236, 0.382, 0.5, 0.618, 0.786 and 1, plus 1.272 and 1.618 beyond the start when enabled. Dotted lines from the start of the swing, extended right, with a label per level (ratio and price) at the left end of its line, on the start bar of the swing, so the space right of the last bar stays free for the volume profile; the 0.5 and 0.786 lines are orange because they bound vote 7's band. The label tooltip names the two bars of the swing. | on, 144, extensions off |
| Floor pivots | `P = (H + L + C) / 3`, `R1 = 2P - L`, `S1 = 2P - H`, `R2 = P + (H - L)`, `S2 = P - (H - L)`, `R3 = H + 2 (P - L)`, `S3 = L - 2 (H - P)`, from the high, low and close of the previous completed bar of the pivot timeframe. The lines break on the bar where P changes, so that bar has no point. Hidden unless the pivot timeframe is above the chart timeframe. | off, D |
| S/R #1 | Swing pivots `ta.pivothigh(high, 10, 10)` and `ta.pivotlow(low, 10, 10)` on the chart timeframe. A pivot high becomes a resistance and a pivot low a support: a line from the pivot bar extended right, created on the bar that confirms the pivot, 10 bars later. It starts unbroken, and that role always matches price at birth: the confirming bar is the last of the 10 bars whose highs stay at or below the pivot high (lows at or above the pivot low), so its close cannot be beyond the level. Breaks are checked from the next closed bar on: a close above a resistance flips it to support and a close below a support flips it to resistance; a second break removes the level. At most 8 levels born from pivot highs and 8 from pivot lows; the oldest beyond that is deleted. A small dot in the line's color at the pivot carries the tooltip: price, pivot lengths, "confirmed 10 bars after its bar" and the time of the confirming bar, then the current role. The distance filter below decides which of these levels are drawn. | on, gray |
| S/R #2 | The same rule on the higher timeframe: pivots computed on its bars, breaks checked with the close of each completed higher-timeframe bar, once per completed bar, on the first closed chart bar after it. A new level starts unbroken for the same reason: the close of the higher-timeframe bar that confirms it cannot be beyond it. Its breaks are checked from the next completed higher-timeframe bar on. At most 4 levels from pivot highs and 4 from pivot lows, dashed. Not drawn unless the higher timeframe is above the chart timeframe. | on, automatic timeframe |
| Color by role | Off: every level in the line color. On: supports teal, resistances red. | off |
| Distance filter | Both S/R sets, evaluated on the last bar only: a level farther than 5 x ATR(14) from the last close (the previous bar's close and ATR while the last bar is forming) is not drawn; its line and dot are deleted. Its price, role and breaks are kept and checked as usual, and it is drawn again from its pivot when price comes back within range. 0 turns the filter off, and every level is drawn while the ATR has no value. A display choice only: levels are not votes, so the signals and the scorecard do not depend on it. Far levels extending right otherwise stretch the chart's price scale (observed on an XAUUSD 1 hour chart). | 5.0 |
| Trend card | Top right. Per row timeframe, from its last completed bar: Up when the fast EMA is above the mid EMA and the close is above the mid EMA, Down in the mirror case, Flat otherwise; then the ADX with "strong" (at or above the threshold) or "weak". A row below the chart timeframe reads "below chart". The last row is the score of the last closed bar. | on, 15, 60, 240, D, W |
| Volume profile | Over the last 200 closed bars: 30 rows between the lowest low and the highest high; each bar's volume is spread over the rows its high-low range overlaps, in proportion to the overlap; a bar with no range puts its volume in the row of its close. Boxes to the right of the last bar, the longest 40 bars long. The point of control (POC) is the row with the most volume, with a line from the first bar of the range to the right edge of the profile. The value area starts at the POC and adds the neighboring row with more volume, one row at a time, until it holds 70% of the total; its rows are shaded. Nothing is drawn when the symbol reports no volume. | on, 200, 30, 40, value area on |

The auto Fibonacci and the volume profile are redrawn on the last bar of the chart. While that bar is forming, every series they read is taken one bar back, so they change only when a bar closes.

## Oscillator pane

`confluence_scorecard_oscillator.pine`, a separate indicator in its own pane.

| Element | Rule | Defaults |
| --- | --- | --- |
| Histogram | The momentum of vote 4, as columns. Color: zero or above and rising, dark teal; zero or above and not rising, light teal; below zero and rising, light red; below zero and not rising, dark red. Rising means above the previous bar's value. Transparency `round(70 x (1 - min(ADX / threshold, 1)))`: opaque at or above the threshold, 70% at an ADX of 0, and 70% while the ADX has no value. | length 20, ADX 14/14, threshold 25 |
| Squeeze dots | A dot on the zero line per closed bar: orange while the Bollinger Bands (`SMA(close, 20) +- 2 x stdev`) sit inside the Keltner Channel (`SMA(close, 20) +- 1.5 x SMA(true range, 20)`), blue on the release bar (the first bar not in a squeeze after a bar in one), gray otherwise. | 20, 2.0, 20, 1.5 |
| Zero crosses | An up triangle at the bottom on a closed bar where momentum is above zero and was at or below zero on the previous bar; a down triangle at the top in the mirror case. | |
| Divergences | Momentum pivots with `ta.pivothigh` and `ta.pivotlow`, 5 bars to the left and 5 to the right, paired with the price of their bar. With 5 to 60 bars strictly between two pivots of the same kind: price higher high with momentum lower high is bearish, price lower low with momentum higher low is bullish. A line joins the two momentum pivots and a `Bull` or `Bear` mark sits at the second; its tooltip gives both prices, the pivot bar's time and "confirmed 5 bars after its bar" with the time of the confirming bar. The oldest beyond 50 are deleted. | on, 5/5, 5 to 60, 50 kept |
| Alerts | `alertcondition` for momentum crossing above zero, crossing below zero, and the squeeze release, each true only on a closed bar. | |

The histogram on the forming bar moves with price until the bar closes, like any plot; the dots, the triangles and the divergence marks wait for the close.

## Alerts

- `alertcondition`: BUY, SELL, BUY confirmed and SELL confirmed. The conditions are set on closed bars only; the two confirmed ones can fire only with the safeguard on.
- `alert()`, for an alert set on "Any alert() function call", with `alert.freq_once_per_bar_close`. "Signals only" (default) calls it on the closed bars with a BUY, a SELL or a safeguard confirmation; "Every bar close" on every closed bar. The message carries the symbol, the timeframe, the event, the score and the seven votes, for example `BINANCE:BTCUSDT 60 | BUY | score +5/7 | EMA +1 HTF +1 ADX +1 SQZ +1 VOL 0 DIV 0 FIB +1`.

## What changes after a bar closes, and what does not

- Plots of the forming bar move with price until it closes, like any plot: the moving averages, the Bollinger area and the oscillator histogram. Their values never change after the close.
- Signals, trades, ADX dots, candle colors, S/R levels, safeguard marks, divergence marks, squeeze dots and the scorecard's counts are created or updated on closed bars only.
- A drawing made on a closed bar stays as drawn, with these exceptions, all made on later closed bars by the rules above: a level flips role or is removed after a break; a level beyond the distance filter is not drawn, and is drawn again when price comes back within range; a signal label's tooltip gains the safeguard outcome; trade lines extend until the trade ends or the next opposite signal; drawings beyond the kept counts are deleted; the auto Fibonacci and the volume profile are redrawn when a bar closes.
- All seven `request.security` calls (the floor pivots, the higher-timeframe values behind vote 2 and S/R #2, and the five trend card rows) request `expression[1]` with `lookahead = barmerge.lookahead_on`: the last completed bar of that timeframe, on historical and realtime bars alike.

## Settings

| Group | Input | Default |
| --- | --- | --- |
| Moving averages | MA 1 / MA 2 / MA 3 / MA 4 | EMA 20 on / EMA 55 on / EMA 200 on / EMA 100 off |
| Bollinger area | Show, length, multiplier | off, 20, 2.0 |
| Trend candles and ADX | Trend candles, ADX dots, DI length, ADX smoothing, ADX threshold | on, on, 14, 14, 25 |
| Auto Fibonacci | Show, lookback (2 to 499 bars), 1.272 and 1.618 | on, 144, off |
| Floor pivots | Show, pivot timeframe | off, D |
| Support and resistance | S/R #1, S/R #2, pivot left bars, pivot right bars, color by role, line color, show levels within (ATR) | on, on, 10, 10, off, gray, 5.0 |
| Higher timeframe | Timeframe | empty (automatic) |
| Trend card | Show, rows 1 to 5 | on, 15, 60, 240, D, W |
| Volume profile | Show, bars (10 to 499), rows, width, value area | on, 200, 30, 40, on |
| Signal engine | Minimum score (1 to 7), alternate signals, labels, labels kept, squeeze length, volume length, volume multiple, RSI length, RSI pivot left and right, min and max bars between RSI pivots, divergence window | 4, on, on, 100, 20, 20, 1.2, 14, 5, 5, 5, 60, 10 |
| Safeguard | Use safeguard, confirmation window | off, 3 |
| Take profits and stop | Mode, TP1 / TP2 / TP3 / stop in %, ATR length, TP1 / TP2 / TP3 / stop in ATR, horizon H (1 to 100), draw levels, trades drawn (1 to 30) | ATR multiple, 1 / 2 / 3 / 1.5, 14, 1 / 2 / 3 / 1.5, 50, on, 20 |
| Scorecard | Show, position (eight of the nine table positions; top right is left to the trend card), text size (also used by the trend card) | on, Bottom right, Small |
| Alerts | alert() calls | Signals only |

## How to read the scorecard

1. Read N first. Under about 30 trades per row the interval is wide, and the verdict usually reads "no evidence" unless the gap to the base rate is large, in either direction.
2. Compare the TP1 column with the base-rate row of the same side. The base rate is how often TP1 comes before the stop from any bar on this chart; with TP1 at 1 ATR and the stop at 1.5 ATR it is often above 50%, because the target is closer.
3. Read "Edge vs base": the difference in points. An edge of either sign with "no evidence" means the difference is inside the noise of N trades. "worse" is a result too: on that chart the signals pick entries that reach TP1 before the stop less often than random entries do.
4. Read the Ambiguous column. Ambiguous stops are bars where the order of the stop and a target cannot be known from OHLC; they are counted against both the signals and the base rate. When they are a large share of the stops, the levels are too tight for the timeframe.
5. Change one thing at a time (minimum score, safeguard, TP mode) and watch whether the edge holds. Changing settings until the verdict reads "better" is a search over many tries; the 95% interval does not account for it.

## Check it yourself

1. Add both scripts to BINANCE:BTCUSDT on 1 hour from the Pine Editor and confirm they compile without errors or warnings.
2. Check the identities of the scorecard (see Scorecard).
3. Hover a signal label: the votes must add up to the score in the label, and the score must be at or above the minimum (BUY) or at or below minus the minimum (SELL). The trend card's last row must equal the score of the last closed bar.
4. Control: set TP1 x ATR to 0.01. TP1 must read close to 100% in every signal and base-rate row (a trade can still be stopped by an open beyond the stop, or by a bar that reaches the stop and TP1 together). Set it back to 1.
5. Control: set the minimum score to 7. Signals become rare or disappear, N falls, and the base-rate rows do not change. Set it back to 4.
6. Control: turn the safeguard on. N usually falls, since unconfirmed signals open no trade; the title says "safeguard on"; and each check mark sits 1 to 3 bars after a signal label of the same side.
7. Non-repainting: open Bar Replay, step forward bar by bar across a few signals, and compare with the chart before replay: labels, dots, candle colors and levels must appear on the same bars with the same tooltips.
8. Hand check one S/R level: hover its dot, go to the pivot bar it names, count 10 bars to the right, and confirm that bar's time matches the tooltip.

## Limitations

- **Volume profile.** It spreads each bar's volume evenly over its high-low range. It is an approximation from OHLCV bars, not tick or footprint data, and it differs from a tick-based profile.
- **Pivot lag.** S/R #1 levels appear 10 bars after their pivot, S/R #2 levels 10 higher-timeframe bars after theirs, RSI and momentum divergences 5 bars after theirs. They are drawn back at the pivot, but nothing could have acted on them before the confirmation bar; the tooltips give that bar.
- **Before costs.** No spread, commission, slippage or funding. A small positive edge can disappear after costs.
- **In Percent mode the targets depend on the asset and the timeframe.** 1% is a large move on a forex hourly chart and a small one on a volatile crypto daily chart. The ATR mode, the default, scales the levels to recent volatility; it still assumes the last 14 bars' range is a fair guide to the next H bars.
- **Base-rate definition.** Every closed bar after the warm-up is an entry on both sides, so consecutive base-rate trades share most of their bars and are not independent. The signals' own trades can overlap too, and the Wilson interval treats them as independent, which makes it narrower than it should be when signals cluster.
- **Ambiguous bars count as stops.** This is conservative for the signals and the base rate alike; it can lower both TP1 rates on low timeframes or with tight levels.
- **The votes overlap.** The EMA stack, the higher-timeframe trend and the ADX all measure trend in related ways. The score counts agreements; it is not a probability.
- **Execution time.** The base rate keeps up to 2 x H trades open and steps all of them on every closed bar, so a deeper horizon multiplies that loop. The horizon is capped at 100 bars; near the cap, on a long history, the script can still reach TradingView's total execution-time limit.
- **History depth.** Only loaded bars count, so history depth, symbol and timeframe change every number.
- **Drawing limits.** Only the last 100 signal labels (150 at most), 20 trade drawings (30 at most), 8 + 8 and 4 + 4 levels (drawn only within the distance filter) and 50 divergence marks stay on the chart; the scorecard counts every trade either way.
- **Standard candles only.** Heikin Ashi, Renko and other synthetic bars change the open, high, low and close every rule reads.
- **Not compiled outside TradingView.** Compile both scripts in the Pine Editor before relying on them.
- It reports counts and context, not a trading rule. Not financial advice.

## Left out on purpose

Some all-in-one indicators also offer the following. They are not here, for these reasons:

- **Chart patterns** (triangles, flags, head and shoulders): no definition that can be stated as a rule and checked bar by bar has been published for this kind of tool, and a pattern drawn after the fact cannot be scored without look-ahead.
- **Fibonacci clusters across timeframes**: no definition of what counts as a cluster (tolerance, which swings, which timeframes), so any choice would be invented and untestable against the original.
- **An EMA ribbon**: many EMAs add no vote that the EMA stack does not already give, and they cost plots.

## Pine API notes

What the Pine Script v6 reference and user manual say about the built-ins the scripts rely on, read on 2026-10-16 through the context7 index of the reference (`/websites/tradingview_pine-script-reference_v6`), the user manual (`/websites/tradingview_pine-script-docs`) and the manual's Limitations and User-defined functions pages:

- `barstate.isconfirmed`: "Returns true if the script is calculating the last (closing) update of the current bar." Bar states page: "barstate.isconfirmed is `true` on all historical bars and on the last (closing) update of a realtime bar."
- `request.security`: "The most reliable approach to achieve non-repainting results is to use an `expression` argument that only references past bars (e.g., `close[1]`) while using barmerge.lookahead_on as the `lookahead` value." The `expression` "can also accept tuples, collections, user-defined types, and the outputs of function and method calls."
- Request limits: "Scripts can perform up to 40 unique requests using any combination of `request.*()` function calls, or up to 64 if the user has the Ultimate plan." The overlay makes 7 unique calls. "All the `request.*()` function calls in a script taken together cannot return more than 127 tuple elements"; the overlay returns 30.
- Drawings: "A script can display a maximum of 500 lines, 500 boxes, 500 polylines, and 500 labels on a chart at any given time", and when a script exceeds its limit "the runtime system _deletes_ the _oldest_ drawings of that type". The overlay deletes its own oldest drawings first and keeps at most 423 labels (150 signal labels, 150 check marks, 90 trade marks, 24 level dots, 9 Fibonacci labels), 184 lines and 100 boxes, below the 500 limit, so the runtime does not have to delete a live level.
- Future bars: "A maximum of 500 bars in the future can be referenced." The profile reaches at most 202 bars ahead.
- Tables (manual, Tables page, read on 2026-10-16): "The tooltip parameter accepts a “string” value that defines the tooltip text for the cell", and a cell width of 0, the default, "specifies that the width automatically adjusts based on the cell’s text". The scorecard keeps its long text in the title's tooltip for that reason.
- Support and resistance levels are drawn with `xloc.bar_time` from the pivot bar's open time, so a level redrawn on the last bar by the distance filter never needs a bar index beyond the 500-bar buffers.
- Loops: "The execution time for any loop on any single bar is limited to 500 milliseconds." Total execution: "The limit is 20 seconds for basic accounts, 40 for others."
- `max_bars_back(var, num)`: the runtime sizes buffers on historical bars, but "The system does not continue to resize buffers on realtime bars", and the reference example also sizes `time` because "drawing calculations convert bar indices to `time` values internally".
- Plot counts: "A maximum of 64 plot counts are allowed per script", and `alertcondition()`, `barcolor()` and `plotshape()` count. The overlay uses about 20 and the oscillator about 9.
- `alertcondition(condition, title, message)`: title and message are `const string`. `alert(message, freq)`: `alert.freq_once_per_bar_close` "allows an alert only when a realtime bar closes".
- Moving-average switch: with a `simple string` parameter "the compiler does not raise a warning about the function", "as long as the script evaluates calls to the function on every bar".
- v6 `for` loops "dynamically evaluate their stopping criteria before **each iteration**"; every `for` loop that removes elements takes its bound from a variable set before the loop and runs backwards.
- `if` blocks used as values: "the type of the value returned by all local blocks must match"; in the functions whose last statement is an `if` with an `else`, both branches end with a call that returns nothing.
- Realtime rollback: "if a script calls label.new() to create a label object on the open bar, the system _deletes_ that object during rollback", and changes to objects held in `var` variables are reverted. The last-bar drawings rely on this.
- `ta.pivothigh(source, leftbars, rightbars)` returns "Price of the point or 'NaN'"; the reference example plots it with `offset=-rightBars`, so a pivot is reported `rightbars` bars after its bar. `ta.highestbars` and `ta.lowestbars` return an "Offset to the highest bar" used as `bar_index + offset` in the reference example.
- `timeframe.in_seconds()`: "When the timeframe is "1M" or more, calculations use 2628003 as the number of seconds in one month."

## Files

- `confluence_scorecard.pine`: the overlay (engine, signals, trades, scorecard, context layers).
- `confluence_scorecard_oscillator.pine`: the oscillator pane.
- `TRADINGVIEW_PUBLICATION.md`: draft title, description and checklist. Not published.
- `LICENSE`: MIT.

## License

MIT. See `LICENSE`.
