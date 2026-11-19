[![token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/bitkingkong/token-radar/rank.svg)](https://make.fun/bounty/token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/bitkingkong/token-radar/reward.svg)](https://make.fun/bounty/token-radar-01)

# Token Radar

A token launch radar: 198 pre-token DeFi protocols ranked by implied value,
built from one sealed DefiLlama snapshot (14 Nov 2026). Entry for the
[make.fun token-radar-01 bounty](https://make.fun/bounty/token-radar-01).

## What it does

`main.ts` (Node 22, standard library only, no network) reads
`/input/snapshot.json` — 758 protocols, 40 on the watchlist — and writes:

- `board.json`: 198 ranked rows (all 40 watchlist protocols, plus 20 more
  above the $50M floor the list leaves out and 138 below it). Rank 1 is the
  most valuable by the model's implied value.
- `data.js`: the dashboard's dataset.
- `dashboard/`: a static, offline page — sortable and filterable board,
  per-protocol detail view with the model's own arithmetic, a live-token view
  set apart from the ranking, and a model sandbox that re-ranks the board
  in the page.
- `screenshots/`: three screenshots of the dashboard.
- `dossier.json`: built with `buildDossier` from `vendor/makefun-board.ts`.

## The value model (radar-2)

Implied value follows only from the snapshot's own figures — TVL, seven-day
growth, age since first listing, and chain breadth. No protocol name is ever
special-cased; one function scores every row.

```
implied = round( tvl_usd x 0.25 x growth_mult x maturity_mult x chain_mult )

growth_mult   = 1 + 0.6 x max(growth, 0)/100 - 0.4 x max(-growth, 0)/100,
                growth clamped to [-60, 150]
maturity_mult = 0.55 + 0.45 x min(age_days, 1460) / 1460
                (age unknown: 365 days)
chain_mult    = 1 + 0.03 x min(chains - 1, 4)
```

Growth is asymmetric on purpose: a protocol gaining TVL fast is closer to a
launch than one the same size losing it, so upside counts more than downside.
Maturity rewards protocols that have run longest with no token. Chain breadth
rewards distribution: a protocol live on more chains reaches more farmers.

Where the snapshot carries no seven-day growth the model counts 0%, and where
it carries no listing date the model counts the age as 365 days; the detail
view says so on the rows it applies to. "No token on record" means the
snapshot shows no market cap, no CoinGecko id and no symbol — never that the
protocol has no token.

Wrapped, staked and bridged products of tokens that already trade (WBTC,
Binance staked ETH, USDT0, bridges) are kept out of the ranking — they are
not protocols still waiting on a launch of their own. Protocols whose token
already trades are shown separately in the Live token view with their market
cap, never mixed into the pre-token ranking.

## Farm plans

Every row carries a 9-step farm plan derived from the protocol's own record
(category, chains, TVL, age): a concrete first action, the capital it needs,
weekly qualifying actions, a hold window, how to check for points, a farm
log, and a plan total that adds the capital and the gas from the steps above.
Each step states its cost and time. Nothing is a slogan.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Then open `out/dashboard/index.html` in a browser (no server, no network).

## License

MIT — see [LICENSE](LICENSE).
