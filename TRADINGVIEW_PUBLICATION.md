# Confluence Scorecard: Signals Graded Against a Base Rate

**Published 2026-11-12** as a public open-source script: https://www.tradingview.com/script/oqAIpNH6-Confluence-Scorecard-Signals-Graded-Against-a-Base-Rate/

Description as published (between the rules):

---
WHAT IT DOES
An all-in-one confluence overlay with a built-in test of its own signals. Seven votes, each +1, 0 or -1, are read at the close of every bar: EMA stack (20/55/200), higher-timeframe trend, ADX at or above 25 with the leading DI, squeeze momentum (the linear-regression momentum of LazyBear's open-source Squeeze Momentum Indicator, read for sign and slope only, with no squeeze state), volume above 1.2 times its 20-bar average signed by the candle, a recent regular RSI divergence, and the close inside the 0.5-0.786 retracement band of the 144-bar swing. Their sum is the score, from -7 to +7. BUY prints when the score reaches the minimum (4 by default) from below, SELL when it reaches minus the minimum from above; by default they alternate (no second BUY before a SELL, and the reverse). Each signal (each confirmed one, with the optional safeguard on) opens a hypothetical trade with three targets and a stop, and a scorecard counts how often each target was reached before the stop. Context layers, each with its own switch: four moving averages, a Bollinger area, trend candles, ADX dots, auto Fibonacci, floor pivots, swing support and resistance on the chart and on a higher timeframe with role reversal (drawn within 5 ATR of price), a multi-timeframe trend card and a volume profile.

WHAT IS ORIGINAL HERE
Confluence tools stack indicators and print signals, but agreeing indicators say nothing about whether the next move reaches a target. Three choices make this one checkable. First, no vote, signal or level depends on a forming bar: votes are read, and signals, dots, candle colors and levels are set, only at the close, and every higher-timeframe value comes from the last completed bar of that timeframe (expression offset by one bar, lookahead on). Swing levels are drawn back at their pivot, and their tooltips say how many bars later they were confirmed. Second, every BUY or SELL label lists its seven votes by name and value in its tooltip. Third, the scorecard compares the signals with a base rate: the same TP1, stop and horizon applied to every closed bar, long and short. Long, short and all signals each get a TP1 rate with a Wilson 95% interval, the difference from the base rate in percentage points, and a verdict: "better" when the interval sits above the base rate, "worse" when below, "no evidence" otherwise.

WHY THESE COMPONENTS ARE TOGETHER
The moving averages, trend candles, ADX dots, auto Fibonacci and trend card draw the inputs of votes 1, 2, 3 and 7, so each signal can be checked against what produced it. The Bollinger area, floor pivots, swing levels and volume profile do not vote and do not change the scorecard; they are context for where a signal's targets and stop sit relative to nearby levels.

HOW IT IS MEASURED
- Entry: the close of the signal bar, or of the confirmation bar with the optional safeguard (a close beyond the signal bar's high or low within 3 bars while the score still reaches the minimum minus one).
- Levels: ATR multiples by default (1, 2 and 3 x ATR(14), stop 1.5) or a percent of the entry.
- Each bar after the entry: an open at or beyond the stop is a stop; targets reached by the open count first; a stop and a further target inside one bar count as a stop and are tallied as ambiguous; after H bars (50) the trade times out.
- Base rate: one long and one short entry at every closed bar after the warm-up, TP1 only, same rules.
- Counters move on closed bars only; open trades are not counted. Before costs.

HOW TO USE IT
1. Keep the defaults first and read N: under about 30 trades per row the interval is wide, and the verdict usually reads "no evidence" unless the gap to the base rate is large, either way.
2. Compare each signal row's TP1 with the base-rate row of the same side, then read "Edge vs base". The All row uses both base rates, weighted by its own mix of longs and shorts.
3. Change one setting at a time (minimum score, safeguard, ATR mode) and watch whether the edge holds. Trying settings until one reads "better" is a search the interval does not account for.

HOW TO READ THE TABLE
Title: hover it for the symbol, timeframe, levels, horizon, counting rules and the number of open trades.
Long, short and all signals: N, TP1, TP2 and TP3 reached before the stop, Stop, Timeouts, Ambiguous, TP1 95% CI, Edge vs base, Verdict.
Base rate, long and short: N, TP1, Stop, Timeouts, Ambiguous and the TP1 interval; TP2 and TP3 read "-", and Edge and Verdict are empty.

LIMITATIONS
- The volume profile spreads each bar's volume over its range: an approximation from OHLCV bars, not tick data.
- Swing levels appear 10 bars after their pivot (on the higher timeframe, 10 of its bars plus one chart bar); RSI divergences 5 bars after theirs.
- Consecutive base-rate trades share bars and signals can overlap, so the interval is narrower than it should be when signals cluster.
- No fills, spread, commission or slippage. Only loaded bars count. Use standard candles. Counts, not a trading rule. Not financial advice.

The chart shows BINANCE:BTCUSDT on the 1 hour timeframe with the default settings (the Bollinger area and floor pivots are off by default): three EMAs, trend candles, ADX dots, Fibonacci lines, swing levels (solid: chart timeframe; dashed: higher timeframe) and the volume profile. The scorecard is in the bottom right, the trend card in the top right.
---

## Publishing checklist

- [ ] Decide whether to publish at all, and when: the account's quota of public scripts per 30 days applies, and a publication cannot be withdrawn after 15 minutes.
- [ ] Compile `confluence_scorecard.pine` in the Pine Editor with no errors or warnings.
- [ ] Set the chart: BINANCE:BTCUSDT, 1 hour, default inputs, scorecard and trend card visible.
- [ ] Check the identities: in each signal row TP3 + Stop + Timeouts / N = 100% up to rounding, and TP1 >= TP2 >= TP3; in each base-rate row TP1 + Stop + Timeouts / N = 100%; All N = Long N + Short N; Ambiguous never above the number of stops.
- [ ] Hover three signal labels: the seven votes add up to the score in the label.
- [ ] Control: set TP1 x ATR to 0.01. TP1 must read close to 100% in every row. Set it back to 1.
- [ ] Control: set the minimum score to 7. Signals become rare; the base-rate rows do not change. Set it back to 4.
- [ ] Bar Replay across a few signals: labels, dots, candle colors and levels appear on the same bars as before the replay.
- [x] The description publishes only the overlay. The oscillator stays out of the TradingView publication (decided 2026-11-12); it lives in the repository only.
- [ ] Check the title is at most 60 characters (it is 56).
- [ ] Publish as open source, with the MIT license text if the form allows it.
- [ ] Check that the description contains no links, contact info, or performance claims, and credits LazyBear for the squeeze momentum formula.
- [ ] In Final touches, open Show more and fill the tags BEFORE pressing Publish; the field takes 5 tags at most, and the Tags field has to be clicked inside before each paste. Take the tags from the suggestion list and never a category name. Suggested tags (one word each, to check against the list): confluence, baserate, supportresistance, scorecard, adx. Do not use `squeeze`: the overlay reads no squeeze state.
- [ ] Before pressing Publish, add the script to the chart from the editor and read the tables on screen: the snapshot shows what is on the chart at that moment. Make sure the chart has no drawings: a stray paste becomes a text drawing in the snapshot.
- [ ] Add the TradingView script URL to `README.md` after publishing.
