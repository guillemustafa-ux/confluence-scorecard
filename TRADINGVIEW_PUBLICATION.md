# Confluence Scorecard: Signals Graded Against a Base Rate

**DRAFT: not published.** Nothing below has been posted to TradingView. Publishing is a separate decision.

Description to paste as is (between the rules):

---
WHAT IT DOES
An all-in-one confluence overlay with a built-in test of its own signals. Seven votes, each +1, 0 or -1, are read at the close of every bar: EMA stack, higher-timeframe trend, ADX with the leading DI, squeeze momentum, above-average volume signed by the candle, a recent regular RSI divergence, and the close inside the 0.5-0.786 retracement band of the current swing. Their sum is the score, from -7 to +7. BUY prints when the score reaches the minimum (4 by default) from below, SELL when it reaches minus the minimum from above. Each signal (each confirmed one, with the optional safeguard on) opens a hypothetical trade with three targets and a stop, and a scorecard counts how often each level came first. Context layers, each with its own switch: four moving averages, a Bollinger area, trend candles, ADX dots, auto Fibonacci, floor pivots, swing support and resistance on the chart and on a higher timeframe with role reversal, a multi-timeframe trend card and a volume profile.

WHAT IS ORIGINAL HERE
Confluence tools stack indicators and print signals, but agreeing indicators say nothing about whether the next move reaches a target. Three choices make this one checkable. First, nothing waits on a forming bar: votes are read, and signals, dots, candle colors and levels are set, only at the close, and every higher-timeframe value comes from the last completed bar of that timeframe (expression offset by one bar, lookahead on). Swing levels are drawn back at their pivot, and their tooltips say how many bars later they were confirmed. Second, every BUY or SELL label lists its seven votes by name and value in its tooltip, so a signal can be read, not just trusted. Third, the scorecard compares the signals with a base rate: the same TP1, stop and horizon applied to every closed bar, long and short. Long, short and all signals each get a TP1 rate with a Wilson 95% interval, the edge over the base rate in points, and the verdict "evidence" only when the lower bound of the interval clears the base rate. The components are combined because the engine needs them: the moving averages, the ADX, the Fibonacci swing and the higher timeframe feed four of the votes, and the layers put those inputs on the chart.

HOW IT IS MEASURED
- Entry: the close of the signal bar, or of the confirmation bar with the optional safeguard (a later close beyond the signal bar's high or low within 3 bars).
- Levels: a percent of the entry (targets 1, 2 and 3%, stop 1.5%) or ATR multiples.
- Each bar after the entry: an open beyond the stop is a stop; targets reached by the open count first; a stop and a further target inside one bar count as a stop and are tallied as ambiguous; after H bars (50) the trade times out.
- Base rate: one long and one short entry at every closed bar after the warm-up, TP1 only, same rules.
- Counters move on closed bars only; open trades are not counted. Before costs.

HOW TO USE IT
1. Keep the defaults first and read N: under about 30 trades in a row the verdict will read "no evidence".
2. Compare each signal row's TP1 with the base-rate row of the same side, then read "Edge vs base".
3. Hover a label to see which votes made the signal.
4. Change one setting at a time (minimum score, safeguard, ATR mode) and watch whether the edge holds. Trying settings until one reads "evidence" is a search the interval does not account for.

HOW TO READ THE TABLE
Header: symbol, timeframe, levels, horizon, safeguard, before costs.
Long, short and all signals: N, TP1, TP2 and TP3 reached before the stop, Stop, Timeouts, Ambiguous, TP1 95% CI, Edge vs base, Verdict.
Base rate, long and short: the same columns for TP1 only.
Footer: open trades not counted yet.

LIMITATIONS
- The volume profile spreads each bar's volume over its range: an approximation from OHLCV bars, not tick data.
- Swing levels appear 10 bars after their pivot (10 higher-timeframe bars on the higher timeframe); RSI divergences 5 bars after theirs.
- Percent targets depend on the asset and the timeframe; the ATR mode scales them to volatility.
- Consecutive base-rate trades share bars and signals can overlap, so the interval is optimistic when signals cluster.
- No fills, spread, commission or slippage. Only loaded bars count. Use standard candles. Counts, not a trading rule. Not financial advice.

The chart shows BINANCE:BTCUSDT on the 1 hour timeframe with the default settings. The scorecard is in the bottom right, the trend card in the top right.
---

## Publishing checklist

- [ ] Decide whether to publish at all, and when: the account's quota of public scripts per 30 days applies, and a publication cannot be withdrawn after 15 minutes.
- [ ] Compile `confluence_scorecard.pine` and `confluence_scorecard_oscillator.pine` in the Pine Editor with no errors or warnings.
- [ ] Set the chart: BINANCE:BTCUSDT, 1 hour, default inputs, scorecard and trend card visible.
- [ ] Check the identities: in each signal row TP3 + Stop + Timeouts / N = 100% up to rounding, and TP1 >= TP2 >= TP3; in each base-rate row TP1 + Stop + Timeouts / N = 100%; All N = Long N + Short N; Ambiguous never above the number of stops.
- [ ] Hover three signal labels: the seven votes add up to the score in the label.
- [ ] Control: set TP1 to 0.01%. TP1 must read close to 100% in every row. Set it back to 1.
- [ ] Control: set the minimum score to 7. Signals become rare; the base-rate rows do not change. Set it back to 4.
- [ ] Bar Replay across a few signals: labels, dots, candle colors and levels appear on the same bars as before the replay.
- [ ] The description publishes only the overlay. The oscillator is a second script: publish it separately, or leave it out of the publication.
- [ ] Check the title is at most 60 characters (it is 56).
- [ ] Publish as open source, with the MIT license text if the form allows it.
- [ ] Check that the description contains no links, contact info, or performance claims, and is between 3,000 and 4,600 characters.
- [ ] In Final touches, open Show more and fill the tags BEFORE pressing Publish; the field takes 5 tags at most, and the Tags field has to be clicked inside before each paste. Take the tags from the suggestion list and never a category name. Suggested tags (one word each, to check against the list): confluence, baserate, squeeze, supportresistance, scorecard.
- [ ] Before pressing Publish, add the script to the chart from the editor and read the tables on screen: the snapshot shows what is on the chart at that moment. Make sure the chart has no drawings: a stray paste becomes a text drawing in the snapshot.
- [ ] Add the TradingView script URL to `README.md` after publishing.
