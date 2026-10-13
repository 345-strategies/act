# ACT: Alignment-Colored Trend

ACT is a TradingView indicator (Pine Script v6) that colors candles by trend direction, then shades them by how much of the market agrees: how far up a ladder of timeframes, and across how many related indexes, a TheStrat breakout is confirmed. A faint candle has a direction and little behind it. A solid one has its higher timeframes and its sister indexes breaking the same way.

It is display only: no orders, no entry signals, no strategy. It shows how much of the market agrees with a direction, and leaves the trade to you.

## Reading the candles

| Candle | Meaning |
|---|---|
| Green / red | Bull / bear trend. Price is above (below) both Direction timeframe opens |
| Faint to solid | How much confirms it, 0 to 11 points (see Shade) |
| Lime / pink | Peak: a Direction-level breakout with both State timeframes breaking the same way |
| Blue / purple | Counter-trend: the two Direction timeframes disagree, and a State 2 breakout sets the direction |
| Grey | No trend: no direction, or the Direction timeframes are conflicting, both inside, or failing |

Green and red always mean bull and bear, never good and bad.

**Shade** adds up:

- **Cascade:** the highest ladder timeframe at which any related index holds a break of its prior bar in the trend direction. Direction 1 = 7, State 2 = 3, State 1 = 1.
- **State 2:** +2 breaking with the trend, +1 outside bar with it, -1 against or failing.
- **State 1:** +1 breaking with the trend, -1 against.
- **Counter-trend bonus:** +1 when a counter-trend direction has a cascade at State 2 or higher.

**Grade** is the shade plus index sync on the chart bar (all three indexes break their prior bar the same way = 4 points, two = 2): A+ at 10, A 7, B 5, C 3, D 1. Grades are a readout in the dashboard and the detailed alert; they never change the paint. Sync moves at chart-bar speed, so it stays out of the paint to keep the candles from strobing.

## Ladders

Every ladder is five steps of the Strat timeframe list, `1m 5m 15m 30m 1H 4H D W M Q Y`, each with one job:

| Ladder | Trigger | State 1, 2 | Direction 1, 2 | Auto on charts |
|---|---|---|---|---|
| Scalp | 15m | 30m, 1H | 4H, D | under 1 minute |
| Day Trade | 30m | 1H, 4H | D, W | 1 minute to 1H |
| Swing | 4H | D, W | M, Q | above 1H to 1D |
| Position | D | W, M | Q, Y | above 1D to 1W |

- **Trigger:** a break of the prior Trigger bar's high or low, behind a ladder-timeframe break, turns the stance from watch to active (dashboard and detailed alert).
- **State 1 and 2:** add or subtract shade.
- **Direction 1 and 2:** set the trend.

A ladder runs on charts up to its State 1 timeframe. Day Trade covers 1-minute through 1-hour charts. When the chart is above the Trigger (Day Trade on a 1H chart), the Trigger reads the prior chart bar instead, and the State and Direction reading behind the colors is unchanged.

## Changing chart timeframes

**Colors belong to the ladder, not the chart.** Inside one ladder, a different chart timeframe samples the same reading more or less often: 1m, 5m and 1H charts all show the Day Trade reading. Small differences remain. Larger bars touch prior highs and lows more easily.

Crossing into another band in Auto switches the ladder, and the colors change with it. The **ladder badge** (bottom right by default) always says which reading you're looking at:

`ACT · Day Trade   1H  4H  D  W   auto`

Each timeframe in the badge is green while price is above its open and red while below, so the Direction pair explains the candle color at a glance. Hover any cell for its job. A `≈` means the chart timeframe doesn't divide the ladder evenly (a 45-minute chart, or monthly timeframes built from weekly bars), so some ranges are approximate.

**Lock a ladder** under Timeframes > Ladder to keep one reading across charts. On a chart above the locked ladder's State 1 timeframe, ACT stands down instead of quietly switching: no paint, no dots, and an orange badge (`ACT: Day Trade needs a chart at 1H or lower`).

## Indexes

ACT reads three related markets, picked from the chart symbol by Auto: ES, NQ, RTY and YM futures; SPY, QQQ, IWM and DIA; or BTC, ETH, SOL and XRP on Coinbase. The chart's own ticker is left out. Any three symbols can be set under Custom.

## Panels and alerts

- **Alignment dots** (off by default, bottom of the chart): one per Trigger, State 1, State 2 and Direction 1 timeframe. Bright = the chart and all three indexes hold beyond that timeframe's prior high (bull) or low (bear); dim = three of the four.
- **Dashboard** (off by default): ladder, trend and its source, stance and why, grade, the Trigger bar, the indexes, and each ladder timeframe's bar type.
- **Alerts:** `ACT: Trend On` fires when candles go from grey to colored, and `ACT: Trend Off` the other way, both on bar close. Turn on Detailed Alert Message for one `alert()` that carries direction, source, grade, stance and ladder.

## No repaint

Closed bars never change. Higher timeframes come from completed bars (`[1]` with `lookahead_on`) and period opens. Every forming higher-timeframe range is rebuilt from chart bars, never read off the served forming bar. Trend On/Off alerts fire on bar close.

## Limits

- Charts above 1W have no ladder.
- Extended-hours charts, and Custom index sets, build calendar timeframes from loaded chart bars. They need enough history loaded to cover two full periods of the highest timeframe before those readings are right.
- The leftmost bars of any chart are approximate until each higher timeframe completes its first period.
- Non-index symbols (single stocks, forex) are compared against the index ETFs.

## License

Mozilla Public License 2.0.
