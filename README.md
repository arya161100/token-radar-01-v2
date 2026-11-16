# token-radar-01-v2

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/arya161100/token-radar-01-v2/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/arya161100/token-radar-01-v2/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

Token Radar (model radar-3): a ranked board of pre-token DeFi protocols by implied value, where the value is **fitted, not assumed**. The program fits `log(market cap) = 4.489 + 0.659·log(TVL) + 2.513·growth` by OLS on the snapshot's own 76 traded-token comps, validates it out of sample (5-fold R² 0.238, time-split R² 0.301, out-of-sample multiple ×2.9 / ×2.2), checks it under 200 bootstrap refits, and ranks all 198 rankable pre-token protocols (40 on the watchlist, 158 early finds). Every farm plan is netted to dollars under stated scenario assumptions, with a verdict: 1 FARM, 132 WATCH, 65 SKIP — most farms lose money once gas and dilution are counted, and the board says so. Age, chain count and category dummies were fit, tested and rejected (they lower out-of-sample R²); the dashboard's Method tab shows the ablation. Full methodology: [RESEARCH.md](RESEARCH.md).

It reads a sealed DefiLlama snapshot (758 protocols, as of 2026-11-14), and renders a sortable, filterable dashboard with a detail view per protocol and a separate live-token view. Implied value follows from the snapshot's own figures only — TVL and seven-day growth — never from a hardcoded list of names. Live-token protocols are set aside with their market caps, never mixed into the ranking. "Records no token" means the snapshot's record, never proof that a token does not exist.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input kits/token-radar-01-kit/token-radar-01-kit/input/snapshot.json
```

Node 22, standard library only, no network. The program writes `out/board.json`, `out/data.js`, copies `dashboard/` and `screenshots/` to `out/`, and writes `out/dossier.json`. Run it twice: the output bytes are identical (seeded bootstrap, sorted iteration, no clock).

---

<sub>Entered on [make.fun](https://make.fun/?ref=3cjrh3ss&utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards</sub>
