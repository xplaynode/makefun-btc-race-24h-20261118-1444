# Build summary — btc-race-24h-20261118-1444

Entry: Snap-Back, a two-branch dip buyer (`main.ts`, Node 22,
vendored kit SDK shim under `vendor/`).

## Final local grade (kit grade-local, process driver)

Command, run from the kit directory
(`~/workspace/makefun/btc-race-24h-20261118-1444-kit/`):

```sh
node cli/makefun.mjs grade-local --kit . --bot ~/workspace/makefun/btc-race-24h-20261118-1444-entry --driver process
```

Result on the dev window (1,440 bars):

- Net PnL: **+11.024265 USDC** (return 0.011024265)
- Trades: **2** (one round trip), win rate 1.0, 0 rejected orders
- Max drawdown: **0.003142584** (0.314%)
- Fees: 0.804572 USDC; net PnL at 2x fees +10.216087 USDC
- Guardrails: met (trades, symbols, drawdown)
- Determinism: L1 regrade reproduced byte for byte;
  action_log_sha256
  204cf131c1f73c93087e18b1182da927ccb0d8dea11a4623ffba4f2ea24c87f3
- Note: the local node is v24.22.1 while the server harness runs
  v26.13.1, so the local result_sha256 cannot equal the server's;
  the score figures are the kit's own grader output.

The one round trip: snap signal at bar 67 (close −1.72% over 40
bars, −1.79% under its 60-bar mean), buy fill at bar 68, take-profit
exit decided at bar 80, sell fill at bar 81.

## What was tried on this dev tape (float replay of the fill model)

- Buy-and-hold posture: −2.80% tape drift, fails the gate by
  construction.
- Z-score mean reversion (windows 30–120, stretch 1.5–3.0 sigma,
  exit at the mean): every variant lost 11–16 USDC.
- Previous races' dip-buy parameters (360-bar anchor/deviation,
  362-bar warm-up): −4.60 USDC, one losing trade; the tape's worst
  deviation under its 360-bar mean is only −1.78%.
- Adding a looser mid-tape dip branch (60-bar drop 1.0–1.1%,
  deviation under the 120-bar mean): +8.01 against +11.02 without
  it — its extra trade is a trap, so the branch was cut.
- Neighbours of the final snap settings: drop 1.5% → +11.02 (same
  trade), drop 1.9% → +8.74, take profit 0.8% → +5.71, take profit
  1.2% → +10.73. The chosen point sits on a plateau.

## Revision note

After the first full grade, the exit test was factored out of the
batch handler into its own `exitDue` function (no parameter moved);
the re-grade reproduced the same action log byte for byte.
