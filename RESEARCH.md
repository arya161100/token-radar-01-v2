# Token Radar radar-3 — research note

How v2's board is built, what was tested, what was rejected, and where it can still be wrong.
All figures below are recomputed by `main.ts` from the sealed snapshot on every run
(`out/data.js` carries the same validation block); nothing here is asserted from memory.

## 1. What changed vs v1 (radar-2)

v1 scored every row with a hand-picked formula: implied = 0.30 × TVL × growth × maturity
× breadth × category premium. None of its constants came from data. v2 replaces the whole
formula with a regression fit on the snapshot's own traded-token comps, validates it out of
sample, and reports the validation honestly — including the parts that did not work.

Same subjects as v1: all 198 rankable pre-token protocols (40 watchlist + 158 early finds),
same farm-plan shape (6 steps per row). Different values, different order, different story.

## 2. Data

- Snapshot: DefiLlama `https://api.llama.fi/protocols`, fetched 2026-11-14T01:18:33Z, 758
  protocols, watchlist floor $50,000,000, 40 protocols on the watchlist.
- Comps: the 76 traded protocols with market cap and TVL above zero, not an exchange, in core
  DeFi categories (Dexs, Lending, Yield, Yield Aggregator, Derivatives, Perpetuals, Options,
  CDP, RWA, Liquid Staking, Liquid Restaking, Restaking, Leveraged Farming, Basis Trading,
  Liquidity Manager, Farm). Exchanges are excluded: a CEX token is not priced off protocol TVL.
  57 of the 76 comps carry a listing date, which is what the time split uses.
- "Pre-token" means the record carries no token (no market cap, no CoinGecko id, no symbol).
  It is never proof that a token does not exist: a protocol on the board may still have a token
  its DefiLlama record does not carry.

## 3. The fitted model

```
log(market cap) = b0 + b1 * log(TVL) + b2 * growth
```

Ordinary least squares, fit at run time from the snapshot (no coefficient is hardcoded).
`growth` is the seven-day TVL change in per cent, clamped to [-50, +50] and divided by 100;
missing change is 0. Fitted coefficients on this snapshot:

| coefficient | value |
| --- | --- |
| b0 (intercept) | 4.489 |
| b1 (log TVL) | 0.659 |
| b2 (growth) | 2.513 |

Reading: the slope on log TVL is well below 1 — a protocol ten times larger is worth about
4.6× more, not 10× more, so implied value per dollar of TVL falls with size. That is the
model's main claim, and it is why the biggest rows here are worth far less than a fixed
multiple of TVL would say (v1's top row was $796M; v2's is $156M). Growth is capped at
±50% because traded comps' seven-day changes run from −40% to +27%: beyond ±50% the model
holds the effect flat rather than extrapolating a straight line to a wild week (one board
row's raw change is +4,448%).

## 4. Validation (out of sample, honestly)

| check | result |
| --- | --- |
| In-sample R² (log market cap, 76 comps) | 0.272 |
| In-sample median \|log error\| | 1.071 |
| 5-fold cross-validation R² (folds by slug order) | 0.238 |
| 5-fold median \|log error\| / OOS multiple | 1.079 / ×2.94 |
| Time split R² (train 28 older listings, test 29 newer) | 0.301 |
| Time-split median \|log error\| / OOS multiple | 0.800 / ×2.22 |

The honest error bar: a typical implied value is within a factor of about ×3 of the
traded-token relation. Implied values are order-of-magnitude estimates; the board's ranking
is more stable than its levels (see bootstrap below).

### Bootstrap (200 refits, seeded, deterministic)

| coefficient | mean | sd | 90% interval |
| --- | --- | --- | --- |
| intercept | 4.34 | 1.75 | [1.10, 6.93] |
| log(TVL) | 0.67 | 0.10 | [0.52, 0.85] |
| growth | 2.26 | 2.49 | [−1.71, 5.70] |

The log-TVL slope is the stable part of the model. The growth coefficient is imprecisely
estimated — its 90% interval covers zero — which is exactly why its effect is capped at
±50% instead of being left to run.

Top-10 rank stability under the 200 refits (share of refits the row stays in the top 10,
median bootstrap rank):

| rank | protocol | stay | median rank |
| --- | --- | --- | --- |
| 1 | sentora-curator | 1.00 | 1 |
| 2 | steakhouse-financial | 1.00 | 2 |
| 3 | gauntlet | 1.00 | 3 |
| 4 | veda | 0.93 | 4 |
| 5 | usdd | 0.95 | 5 |
| 6 | upshift | 0.93 | 6 |
| 7 | figure-markets-prime | 0.58 | 9 |
| 8 | nonce-capital | 0.67 | 8 |
| 9 | cian-yield-layer | 0.40 | 11 |
| 10 | shift-defi | 0.48 | 13 |

The top 6 are stable; places 7–10 shuffle. The exact order below the top 6 should not be
over-read.

## 5. Ablation — what was tried and rejected

Every row below was scored by the same 5-fold / time-split protocol as the chosen model.

| specification | 5-fold R² | time-split R² | kept? |
| --- | --- | --- | --- |
| log(TVL) only | 0.236 | 0.214 | No — baseline |
| **log(TVL) + growth (chosen)** | **0.238** | **0.301** | **Yes** |
| + log age (and age-missing flag) | 0.217 | 0.068 | No |
| + log chain count | 0.197 | 0.293 | No |
| + category-group dummies | 0.199 | 0.369 | No |

- **Age:** adding it lowers both splits, and its fitted coefficient is negative. Among traded
  tokens, older listings are worth less per TVL, not more — the comps give no support to an
  age premium for waiting without a token. The model gives age no weight; age is shown as
  context (days since first listing).
- **Chain count:** lowers the 5-fold split. Breadth does not price traded tokens here.
- **Category dummies:** lift in-sample R² to 0.40 but lower the 5-fold split to 0.199, and
  shrunk group offsets do not generalise to the board's non-core categories (nested 5-fold on
  all 119 traded protocols: R² 0.05, multiple ×4.0). The model gives category no weight either.

## 6. The board (top 10)

| rank | protocol | TVL | 7d | age (days) | implied value | farm net | verdict |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | sentora-curator | $2,758M | +2.4% | 407 | $156.0M | −$55.03 | SKIP |
| 2 | steakhouse-financial | $2,754M | −2.2% | 543 | $139.0M | −$1.30 | WATCH |
| 3 | gauntlet | $1,578M | +5.6% | 543 | $117.2M | −$0.63 | WATCH |
| 4 | veda | $1,563M | −8.3% | 632 | $82.1M | −$2.64 | WATCH |
| 5 | usdd | $1,056M | −2.8% | 611 | $72.8M | −$5.68 | SKIP |
| 6 | upshift | $499M | +8.3% | 710 | $58.7M | −$54.20 | SKIP |
| 7 | figure-markets-prime | $191M | +18.0% | 474 | $39.8M | +$0.01 | WATCH |
| 8 | nonce-capital | $514M | −9.7% | 45 | $38.0M | −$87.43 | SKIP |
| 9 | cian-yield-layer | $280M | +4.7% | 726 | $36.7M | −$53.96 | SKIP |
| 10 | shift-defi | $48M | +4,448% → capped +50% | 45 | $35.6M | −$81.39 | SKIP |

Median implied value across the 198 rows is $5.6M. Seven of v1's top 10 remain in v2's
top 10, but the levels are 3–5× lower and the order changed: the fitted sublinear TVL
slope, not the category premium, now decides the board.

## 7. Farm economics (netted to dollars)

Per row, from the row's own figures plus stated scenario assumptions (a scenario, not a
prediction):

- Capital by TVL tier: $250 (TVL ≥ $5B), $150 (≥ $1B), $100 (≥ $100M), else $50.
- Hold: 60 days if listed < 180 days, 45 days if < 365 days, else 30 days.
- Gas per transaction (stated estimates, Nov 2026): Ethereum $8.00, Bitcoin $5.00,
  Tron $1.00, Solana $0.05, Base / Arbitrum / Optimism / Unichain / BNB Chain / Binance
  $0.30, Polygon $0.10, Avalanche $0.50, every other chain $0.50. Transactions: 2 setup +
  1 per week of hold.
- Opportunity cost: capital × 5% annual × hold / 365.
- Airdrop if launched: implied value × 7.5% farmer allocation × (capital / (10% × TVL)).
- Expected value: airdrop-if-launched × 25% launch probability.
- Net = expected value − gas − opportunity cost. Verdict: FARM if net ≥ $10, WATCH if
  −$5 ≤ net < $10, SKIP if net < −$5.

Distribution on this snapshot: **1 FARM, 132 WATCH, 65 SKIP**. Median net −$1.91 (mean
−$20.77 — the mean is dragged down by Ethereum rows paying $56 of gas). Best net:
manifest-trade +$11.79 (Solana,
small TVL, high implied/TVL). The pattern is structural, not a bug: at a $50–250 position
the pro-rata share of a launch is a few dollars, while a week of Ethereum transactions
costs $56 — so the top rows by implied value are mostly SKIP, and the only FARM sits on a
cheap chain with a small TVL. A reader who farms on Ethereum at this size loses money in
the base case; the board says so instead of dressing every row up as a good farm.

Sensitivity: verdicts move with gas. At the earlier $2.00 blanket estimate for unlisted
chains the split was 1 FARM / 53 WATCH / 144 SKIP; at the $0.50 blanket used here it is
1 / 132 / 65. The net-dollar figure per row is the stable output; the verdict band is a
summary of it. The farmer-share (10% of TVL) and launch-probability (25%) assumptions move
the expected value linearly and are stated so a reader can re-do the arithmetic.

## 8. Limitations (read before trusting the board)

1. **The relation is weak.** TVL explains about a quarter of the variance in log market
   cap out of sample. Most of what makes a token valuable — team, tokenomics, timing,
   narrative — is not in this snapshot, and no model built from it can see it.
2. **Levels are uncertain, ranks are stabler.** Treat implied values as order-of-magnitude
   (±×3 typical error); treat the top-6 order as fairly solid and places 7–10 as fluid.
3. **Growth is noisy.** Its coefficient's interval covers zero; the ±50% cap is a guardrail,
   not a finding.
4. **Age gets no weight.** If waiting without a token is truly worth a premium, this
   snapshot's traded comps cannot show it, and the model does not claim it.
5. **The farm scenario is a scenario.** A 25% launch chance, a 7.5% farmer allocation and
   farmers holding 10% of TVL are round assumptions. Real airdrops reward activity and
   points, not pro-rata TVL, and the net figures inherit that simplification.
6. **"Records no token" is not "has no token."** Verify a protocol has no token before
   farming it; the snapshot is a DefiLlama record, not a fact about the world.
7. **No hindsight fitting.** The model is fit once, on traded comps, and applied uniformly.
   Nothing in it was tuned to make any particular pre-token protocol rank higher.

## 9. Reproduction

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Node 22, standard library only, no network. Two runs produce byte-identical
`board.json`, `data.js` and `dossier.json` (verified by sha256). The dashboard is a static
page (`dashboard/`, reads `out/data.js`); the three screenshots in `screenshots/` are the
board, a detail view and a filtered view, taken at 1440×900 from the built dashboard.
