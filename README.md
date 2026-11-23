# DEEP-DIP v2 — btc-race-24h-20261123-0511

Dip-buying bot for make.fun "Race a BTC trading bot live for 24 hours"
(pot $50, 1st $12). Trades cbBTC/USDC on 1-minute bars, $1,000 paper start.

## Strategy

Buy a sharp pullback, sell the rebound behind a trailing stop:

- **BUY** (all-in, one position) when the close is **>= 100 bps below the
  trailing 60-bar high** of closes **and RSI(14) <= 35** (oversold).
- **SELL** the full position on the first of:
  - take-profit **+200 bps** from entry (entry = true average fill price,
    derived from the position's cost basis, fees included),
  - hard stop-loss **-70 bps** from entry,
  - trailing stop **-70 bps** from the best close since entry.
- **Cooldown 15 bars** after any exit before the next buy.
- Max slippage 500 bps on every order; no clock, no environment, no
  randomness — a pure function of the `hello` and the `batch` stream, so two
  runs produce byte-identical `orders` on every `bar`.

## Why these numbers (dev window, 1,440 bars)

- Dev window: $58,956 -> $61,102 (+3.64%), per-bar stdev 8.1 bps, max
  1-minute move ~70 bps. The deepest dip below any trailing 60-bar high was
  only **-181 bps** — the "classic" 220 bps dip trigger from earlier races
  would *never* fire here, so the dip threshold was re-tuned to this window.
- Baselines on the dev window (grade-local): buy-and-hold **+$32.17**,
  momentum **+$34.94**, ema-cross **-$114.83** (76 trades, churn),
  rsi-bollinger **-$9.32**, adx-momentum **-$11.20**, hold-cash **$0.00**.
  The window rewards being long; active trading mostly loses to the ~26 bps
  round-trip cost. DEEP-DIP beats buy-and-hold by buying only the deep dips
  and exiting on the rebound instead of riding every wiggle.
- Trailing-stop ridge on the real grader (6 trades each): 60 bps -> +$26.99,
  65 -> +$33.25, **70 -> +$33.25**, 75 -> +$30.72, 80 -> +$30.72,
  90 -> +$28.89, 100 -> +$24.23. Smooth plateau, not a knife-edge.
- Robustness (independent Python replay of the same rules): full window
  +$40.84, first half +$22.10 (4 trades), second half +$18.38 (2 trades) —
  profitable on each half separately.

## Local dev grade (kit CLI, `--driver process`)

- **Net PnL +33.245884 USDC** (return +3.32%), **6 trades**, win rate 1.000,
  max drawdown **1.08%**, Sharpe 0.045, fees $2.45, **net PnL at 2x fees
  +$30.78**, 0 rejected orders, guardrails met.
- Determinism: two full grade-local runs give the identical
  `action_log_sha256 6302a79e53da8ab76a5b4a344cd30120e02b4941d2d3c948cec489b224743aba`,
  and the L1 regrade reproduces byte for byte.

## Files

- `main.ts` — the bot (Node 22, erasable TypeScript, written fresh in-session)
- `vendor/makefun-bars.ts` — kit SDK shim, byte-for-byte copy of the kit's
- `LICENSE` — MIT (written by `makefun init`)
