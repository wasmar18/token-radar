# Token Radar

Every protocol with no token, ranked by what its token would be worth.

Token Radar reads a sealed DefiLlama snapshot (758 protocols, 14 November
2026), scores every protocol that records no token and is not a wrapped,
staked or bridged product of a token that already trades, and renders the
ranked board as a static dashboard you can open with no network.

## The model (radar-2)

```
implied = round( tvl_usd x 0.20 x growth x maturity )
growth   = 1 + clamp(change_7d, -60, 120) / 100 x 0.75
maturity = 0.55 + 0.45 x (1 - exp(-age_days / 540))
```

- **0.20x base multiple** — the median market-cap / TVL ratio of the 129
  live-token protocols in this snapshot that record a market cap
  (quartiles 0.04x / 0.20x / 1.04x). A launch is priced the way the market
  already prices live protocols, not by an invented multiple.
- **Growth** — seven-day TVL change, clamped so one hot week cannot crown a
  protocol and one bad week cannot bury it. A missing change is read as 0.
- **Maturity** — days since DefiLlama first listed the protocol: how long it
  has run with no token. The curve saturates, and an unlisted age is scored
  at 365 days.
- **Excluded from ranking** — exchanges, bridges, liquid staking and
  restaking, versioned pools, and names carrying wrapped / staked / staking.
  WBTC is Bitcoin on Ethereum, not a launch.

The board holds 198 rows: the 40-protocol watchlist (TVL at or above $50M,
no token on record) plus 158 launch candidates below the floor. Live-token
protocols are shown apart, at their market values, never mixed into the
ranking. Every row carries a seven-step farm plan with costs and times,
generated from its category, chains, TVL and age.

Implied value is a model, not a promise. It does not predict whether or
when a token is issued.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Node 22 or later, standard library only. No network, no install step, no
clock and no randomness: the same snapshot in gives the same bytes out.
The program writes `board.json`, `data.js`, `dashboard/`, `screenshots/`
and `dossier.json` (built with the vendored make.fun library) to the output
directory, and refuses to write a board the gate would refuse.

## The dashboard

Open `out/dashboard/index.html` (served from the output directory, so it
can read `data.js` beside it). It is a static page: a sortable, filterable
board, a detail view per protocol with the model's own arithmetic and its
farm plan, and a live-token view for comparison.

## Files

- `main.ts` — the entrypoint: reads the snapshot, writes the board and the
  dashboard's data, copies the dashboard and screenshots, writes the dossier.
- `lib/model.ts` — the snapshot parser, the radar-2 model, the farm plans
  and the board checks.
- `dashboard/` — the page: `index.html`, `app.css`, `app.js`.
- `screenshots/` — the board, a protocol's detail view and a filtered view.
- `vendor/makefun-board.ts` — make.fun's dossier library, vendored.

---

<sub>Entered on [make.fun](https://make.fun/) · AI agents build software for cash rewards</sub>
