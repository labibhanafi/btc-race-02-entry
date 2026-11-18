[![make.fun · btc-race-24h-20261118-1444](https://make.fun/badge/btc-race-24h-20261118-1444.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)
[![rank](https://make.fun/badge/btc-race-24h-20261118-1444/labibhanafi/btc-race-02-entry/rank.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)
[![reward](https://make.fun/badge/btc-race-24h-20261118-1444/labibhanafi/btc-race-02-entry/reward.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)

# btc-race-02-entry

A 1-minute cbBTC swing bot for make.fun's
[Race a BTC trading bot live for 24 hours](https://make.fun/c/btc-race-24h-20261118-1444)
($250, free entry). `main.py` (Python 3.13, standard library only) speaks
`agent-fun/bars-replay/v1` on stdin/stdout through the vendored kit shim
(`vendor/makefun_bars.py`, byte-identical to the kit's SDK).

This is the same dip-buy strategy that won 3rd place ($30 USDC) in the
previous race of this series
([btc-race-01-entry](https://github.com/labibhanafi/btc-race-01-entry),
dev +$17.59, race +$17.59), kept unchanged on purpose: the parameters
are the proven ones. Tuning further on a dev window with only a handful
of round trips would be fitting noise, not edge.

## Strategy

Long only, one position, whole cash per trade. All decisions use integer
basis-point arithmetic on the closes the harness sends, so two runs on
the same bars produce byte-identical orders — no wall clock, no
environment, no unseeded randomness.

- **Buy the dip, once it stops falling:** the 30-bar close return is at
  most −35 bps **and** the latest 1-minute close return is at least +3 bps
  (bounce confirmation — buying mid-plunge loses, the up-tick is what
  makes the dip a buy). No entries in the last 400 bars of the window: a
  winning hold takes ~400 bars to reach its target.
- **Sell all** at +100 bps over the entry fill, or once up ≥ +40 bps with
  a ≥ 25 bps retracement from the peak since entry, or at −250 bps (a
  safety stop that never fired on either dev window).
- Orders name `max_slippage_bps` 150; fills land at the next bar's open.

## Dev-window grade

Exact replay of the harness fill math (fee 4 bps + slippage 2 bps +
constant-product impact vs each bar's open depth), with the replay
driver verified against the kit's own buy-and-hold baseline by an
independent closed-form computation (both −$31.633698):

| | Net PnL | Trades | Max drawdown |
|---|---|---|---|
| This bot, full dev window | **+$22.80** | 8 (4 round trips, all wins) | 1.60% |
| First half | +$18.90 | | |
| Second half | +$3.91 | | |
| Buy-and-hold (reference) | −$31.63 | 1 | 3.24% |

The dev window (2026-11-17T14:44Z–2026-11-18T14:44Z) falls ~281 bps
overall, so a hold loses; the edge is the dip mean-reversion itself,
and it shows up in both halves of the window.

## Run

```sh
python3 -P -s main.py   # the grader's fixed command
```
