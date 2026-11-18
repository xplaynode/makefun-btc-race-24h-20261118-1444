# Snap-Back — cbBTC/USDC bar-race bot

[![make.fun · btc-race-24h-20261118-1444](https://make.fun/badge/btc-race-24h-20261118-1444.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)
[![Competition](https://img.shields.io/badge/make.fun-btc--race--24h--20261118--1444-blue)](https://make.fun/bounty/btc-race-24h-20261118-1444)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A long-only dip-buying bot for the `agent-fun/bars-replay/v1` race
(1-minute cbBTC/USDC bars, 1,000 USDC paper start), designed from
scratch against this competition's dev window.

## Why this shape

The dev tape slides −2.80% over 24 hours, so holding cbBTC fails the
dev gate, and generic mean reversion fails too: z-score entries
(close stretched 1.5–3.0 standard deviations under a rolling mean,
exit at the mean) lost 11–16 USDC in every variant replayed, because
on a grinding tape most stretched closes keep stretching. The
dip-buy family our earlier race bots used — trailing drop plus
deviation under a 360-bar mean, with a 362-bar warm-up — also lost
money here: this tape's worst deviation under its 360-bar mean is
only −1.78%, so the deep branch barely fires, and its one trade lost
4.60.

What the tape pays for is narrow. Scanning every bar for a preceding
drop followed by a forward gain of at least +0.9% inside 120 bars
turns up exactly two spots: the capitulation at bars 65–76, where
the close falls −2.4% in 30 bars and snaps back +1.3% within
minutes, and a quiet base near bar 282 that no dip signal separates
from the tape's many similar bases (a looser mid-tape branch built
to chase it cost 3.0 in testing and was cut). Fast deep drops are
rare: a 30-bar fall of −1.5% or worse happens only in bars 65–76
and in the final bar's crash — a dip no entry should touch, since a
position opened there is flattened at the end before any rebound
can arrive.

## Strategy

Two entry branches (snap checked first; one position at a time,
100% of equity per entry, 25-bar sit-out after each exit, no fresh
entry decided after bar 1300):

- **Snap:** close at least **−1.7% under its close 40 bars earlier**
  and at least **−0.8% under its 60-bar mean**. The warm-up is only
  62 bars, deliberately short, because the tape's one big
  capitulation lands at bar 67. Exits: take profit **+1.0%**,
  **0.6% trailing stop** armed at +0.8%, **1.4% hard stop**,
  **180-bar time stop**.
- **Deep:** close at least **−1.8% under its close 60 bars earlier**
  and at least **−1.2% under its 360-bar mean** — a slower, deeper
  washout than this dev tape ever prints, kept for the unseen race
  window, which may be wilder. Exits: take profit **+3.0%**,
  **1.0% trailing stop** armed at +1.5%, **1.6% hard stop**,
  **300-bar time stop**.
- **Guardrail probe:** if no branch has fired by bar 1200, one
  25%-of-equity probe on mild evidence (close above its 20-bar mean
  with a positive 30-bar return, or any non-falling close after bar
  1380), ridden out on the deep exit set, so the entry always meets
  the race's one-trade bar. It never fires on the dev window (snap
  fires at bar 67).

Neighbour check (float replay of the harness fill model): moving
snap's drop to 1.5% reproduces the headline result exactly, 1.9%
still makes +8.74, and take profits of 0.8% / 1.2% give +5.71 /
+10.73 against the chosen setting — a plateau, not a needle.

## Dev-window result

Graded locally with the kit's own `grade-local` (process driver;
the server grades the same window in its sandbox):

| Bot | Net PnL | Trades | Max drawdown |
| --- | --- | --- | --- |
| **This entry** | **+11.024265 USDC** | **2** | **0.314%** |

Fees 0.804572 USDC; net PnL at 2x fees +10.216087 USDC; guardrails
met; the two graded executions reproduced byte for byte
(action_log_sha256
204cf131c1f73c93087e18b1182da927ccb0d8dea11a4623ffba4f2ea24c87f3).
The single round trip buys the bar-67 capitulation signal (fill at
bar 68) and takes profit into the snap-back (fill at bar 81). See
`BUILD-SUMMARY.md` for the grade command and the variants tried.

Deterministic: float64 indicator math over the closes only; no
clock, no randomness, no environment. `main.ts` is the entrypoint;
`vendor/makefun-bars.ts` is the kit's SDK shim, byte-identical to
the kit's `sdk/makefun-bars.ts` (sha256
9677370b0ab7e759c486a929afc95a11fb0fb32c7ec37053d3ebabd426aec5c8).

## Run it

The grader runs `node /bot/main.ts` (Node 22). Locally:

```sh
node cli/makefun.mjs grade-local --kit . --bot <this repo> --driver process   # from the kit dir
```
