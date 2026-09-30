# Wheeler — open items

Single self-contained `index.html`, house style from `../PROJECT-HANDOFF.md`. Storage: one
`localStorage` key, `wheel.v1`, schema 1. Footer version 0.9 (pre-release). Live at https://mrholdingsworth.github.io/Wheeler/ (repo
`mrholdingsworth/Wheeler`, Pages from `main` / root).

## To do

- **Edit an event in place.** Today a mistyped fill is fixed by undoing it and entering it again.
  That's fine for the latest event but awkward for one buried under later events. It needs an
  edit modal per history row that reuses the same validation.
- **Rolls as one action.** A roll today is a buyback plus a new sale: two entries. A single "Roll"
  on an open leg (close + reopen, same date) would halve the typing. Decide first whether a roll's
  net credit should be shown as one line.
- **Live marks.** Calc has Finnhub. Here the last price is typed by hand per position.

## Decided

- **Buying power counts open credit, as a broker does** (changed 2026-09-29, at Steve's request
  via the selector). A put's net credit lands when it sells and offsets its own collateral:
  `BP = value − shares at cost − put collateral − long puts + net credit on open shorts`. It
  still isn't *P/L*, so account value only moves when the leg closes. Max put strike is therefore
  `(BP − fee) / 100 + the credit per share`. The first version left open credit out entirely. That
  made BP go negative right after following the selector's advice, so it was reversed.
- **Account value is closed P/L only**, to the penny. Shares count at cost. "Value at marks" shows
  up only when a position has a typed last price.
- **Assignment realizes the put's premium** into the account. The same premium comes off the share
  basis, so the adjusted basis already includes it. When the shares go, share P/L is measured
  against average *cost*, not basis, so nothing is counted twice. The position total equals
  −(cost − option P/L − share P/L) at the end, and the ledger test in the build session checked
  this against hand-computed figures.
- **Assignment and exercise fees ride on the share trade.** They go into share cost or come off
  proceeds, not into the option's P/L.
- **A put closed without assignment closes its position.** Next week's put is a new position. The
  exception is "Bought back, and bought shares", which keeps the put's credit in the basis. You
  can also sell a new put *into* an open position (averaging down) from section 03 or from the
  position card.
- **Annualized is simple, with a one-week floor.** A Monday-to-Friday put ties up the cash until
  the next Friday cycle, so four days is priced as seven. ROE's denominator is peak capital: put
  collateral plus shares at cost, at the position's high-water mark.
- **Optimizer.** It maximizes total net credit (per week-equivalent when expiries differ), which
  is the same as return on the *whole* capital figure. It never favors a higher % on a smaller
  slice. The constraint is `Σ(collateral − net credit) ≤ capital`: $10,000 can carry $10,100 of
  puts paying $113.70 (Steve's ABC + QUP case). With one expiry it's exact: a DP over collateral in
  the strikes' common step, keeping the most credit at each level. With mixed expiries it's a DP
  over cash needed, rounded up, so it can't overspend. At build time it was checked against
  brute force on 300 random lists. Rows below the weekly minimum are excluded. The result box
  states returns on the whole capital figure.
- **A fixed "Capital to deploy" shrinks as you sell** from section 03, by the cash each sale used.
  Blank means "all of buying power" and follows BP on its own.
- **No start date.** Dropped at Steve's request. The equity chart starts at the first event.
- **Named Wheeler** (repo name). The storage key stays `wheel.v1`: it's invisible, and renaming it
  would strand data already entered on the live site.
- **Bring in existing shares.** "Premium already collected" lowers basis but not account P/L,
  because it's already inside the starting value.
