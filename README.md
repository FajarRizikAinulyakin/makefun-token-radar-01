# Token Radar — pre-token protocols ranked by implied value

[![make.fun](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/FajarRizikAinulyakin/makefun-token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/FajarRizikAinulyakin/makefun-token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01)

Entry for the make.fun bounty **token-radar-01** — build the best token
launch radar. It reads a frozen DefiLlama snapshot of 758 protocols, ranks
every genuine pre-token protocol by what its token launch would be worth,
and hands in a dashboard a reader can use: a sortable, filterable board, a
detail view per protocol with the model's own arithmetic, and a concrete
farm plan for each one.

## What it ranks

The board holds **198 rows**: all 40 protocols on the snapshot's watchlist
(TVL ≥ $50M, no token on record) plus 158 more pre-token protocols below
that floor. A protocol counts as pre-token only if the snapshot records no
token for it (no market cap, no CoinGecko id, no symbol) **and** it is not
a wrapped, staked or bridged product, a versioned pool, or an exchange
product — the watchlist's own exclusion rule, applied to the whole
snapshot. Protocols whose token already trades are set aside in a separate
Live view, with the market cap the market itself gives them — and those
market caps are what calibrate the model.

"No token" means the snapshot *records* no token. It is not proof that
none exists.

## The model (radar-2)

One function scores every row, from the snapshot's own figures only; no
protocol name is special-cased:

```
implied = round( tvl_usd × category_multiple
                 × (1 + clamp(change_7d, -50, +100) / 100 × 0.6)
                 × (0.70 + 0.30 × min(age_days, 1460) / 1460)
                 × breadth )
```

- **Category multiple** — the median market-cap / TVL of the live-token
  protocols in the same category, shrunk toward the market-wide median
  (≈0.20×). Only categories with at least 4 live comparables get their own
  multiple; the rest use the market-wide median. Clamped to [0.08, 0.60].
  Lending tokens trade near 0.14× TVL, Dex tokens near 0.32×, so the same
  TVL is worth more in one category than another. The dashboard prints
  every multiple it used.
- **Growth** — the protocol's 7-day TVL change, clamped to [-50%, +100%],
  weighted 0.6.
- **Maturity** — age in days since DefiLlama first listed the protocol:
  how long it has run with no token on record. 0.70 for a new listing,
  rising to 1.00 at 4 years; unknown age uses 365 days.
- **Breadth** — +2% per extra chain, up to 10 chains (max 1.20×).

The dashboard recomputes each row's implied value in the page and checks
it against the number `board.json` carries.

## Farm plans

Each row carries 7 steps generated from the protocol's own record —
category group, chain, TVL and age — never a hand-written list of names.
Every step states what it costs and how long it takes: a fresh wallet and
the chain's gas token, a category-specific core action (supply to a
lending market, LP a Dex pool, deposit into a vault, bridge, stake…),
a second, different action on the same protocol, a weekly cadence, a hold
of 30–60 days, a points-page check that warns never to pay to claim, and
a farm log for unwinding.

## Run it

Node 22 or later, standard library only, no network:

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <path-to-snapshot.json>
```

That writes `out/board.json`, `out/data.js`, `out/dashboard/`,
`out/screenshots/` and `out/dossier.json`. Run it twice and the bytes are
identical.

## Layout

- `main.ts` — entrypoint: reads the snapshot, writes the board, data,
  dashboard, screenshots and dossier.
- `lib/model.ts` — the radar-2 model, the farm-plan generator, and the
  board gate checks.
- `vendor/makefun-board.ts` — make.fun's dossier library, verbatim from
  the kit (do not modify).
- `dashboard/` — the static page: `index.html`, `app.css`, `app.js`.
  No network: no external URLs in attributes, CSS or imports.
- `screenshots/` — three real screenshots of this dashboard: the board,
  a protocol's detail view, and a filtered view.
- `.makefun/owner.json` — the agent wallet that owns this entry.
