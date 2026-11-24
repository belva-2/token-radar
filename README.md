# Token Radar — pre-token protocols ranked by implied value

A static, offline dashboard that ranks pre-token DeFi protocols by what a token
launch could be worth, from a frozen DefiLlama snapshot. No network requests,
no framework, no tracking.

## What it is

- **The board** — 200 pre-token protocols, strongest implied value first:
  all 40 protocols on the snapshot's watchlist (TVL ≥ $50M, no token on record),
  plus 160 early rows: pre-token protocols below the $50M floor, by TVL.
- **A detail view for every protocol** — its model breakdown, its 7-step farm
  plan, and the risk check for its category.
- **A live-token view** — the 24 largest live-token protocols, shown with their
  market caps as facts, not models. They are also the comparables the ranking
  is priced from.
- **Search, sort and filters** — by name, category, chain, and implied-value
  band, plus a CSV download of the filtered board.

## The method (radar-2)

Implied value is computed from the snapshot's own figures only:

```
implied = round( tvl_usd × category_multiple
                 × (1 + clamp(change_7d, -60, 120) / 100 × 0.5)
                 × (0.6 + min(age_days, 1095) / 1095 × 0.4) )
```

- **Category multiple** — the median market-cap-to-TVL ratio of the
  live-token protocols in the same category (at least 3 comparables),
  clamped to 0.05–0.80. Categories with too few live comparables use the
  global median (0.25) across the snapshot's live-token protocols. A lending
  protocol and a DEX do not trade at the same multiple, and the board does
  not pretend they do.
- **Growth** — the protocol's 7-day TVL change, clamped to −60%…+120%,
  at half weight.
- **Maturity** — age since first listing (0.6–1.0); an unlisted age uses 365 days.
- **Farm plan** — 7 steps per protocol, derived from its category, chains,
  TVL, age and 7-day change: a fresh wallet, the category action, a
  category-specific risk check, weekly qualifying activity, a hold of
  30–60 days, claiming only from the protocol's own published link, and a
  farm log. Each step states its cost and time.

Top of the board: **Steakhouse Financial** at $535M implied value,
followed by **Gauntlet** ($319M) and **Veda** ($306M).

## Data

- Source: DefiLlama `https://api.llama.fi/protocols`, frozen snapshot
  `input/snapshot.json`, fetched **2026-11-14**. The snapshot is read-only
  and is the only data the entry uses.
- 758 protocols in the snapshot; 40 on the watchlist; 342 pre-token in total.
- `board.json` holds all 200 ranked rows with their implied values and farm
  plans; `data.js` is the same board in the dashboard's compact form.

## Limitations

- A high implied value is **not** a prediction that a token will launch, or
  at what price. Many pre-token protocols never launch one.
- The snapshot is frozen at 2026-11-14. TVL, growth and listings have moved since.
- Farming costs gas and carries smart-contract and market risk. Never pay to
  claim an airdrop, and never send funds to an address from a chat message.
- The early radar is pre-token protocols below the $50M floor, by TVL. A
  protocol below the floor is not worse than one on the watchlist — the floor
  is where the house started, not where the radar stops.

## How to use

```sh
# Build the board and the dossier (Node 22+, standard library only)
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <path to snapshot.json>

# Serve the dashboard locally from the built output
cd out && python3 -m http.server 8123
# then open http://localhost:8123/dashboard/index.html
```

Then open `dashboard/index.html`. Deep links work from the URL hash:
`#p=<slug>` opens a protocol's detail view, `#category=RWA` filters by
category, `#band=100M-500M` by implied-value band.

## Layout

- `main.ts` — builds `board.json`, `data.js`, copies `dashboard/` and
  `screenshots/`, writes `dossier.json`.
- `lib/model.ts` — the ranking model, comparables, farm plans, and validation.
- `dashboard/` — the static page (`index.html`, `app.js`, `app.css`).
- `vendor/makefun-board.ts` — the make.fun dossier library, unchanged.
- `screenshots/` — the board, a detail view, and a filtered view.
