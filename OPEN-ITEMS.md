# Wheel — open items

Single self-contained `index.html`, house style from `../PROJECT-HANDOFF.md`. Storage: one
`localStorage` key, `wheel.v1`, schema 1. Footer version 0.9 (pre-release). Not deployed yet.

## To do

- **Edit an event in place.** Today a mistyped fill is fixed by undoing it and entering it again.
  That's fine for the latest event but awkward for one buried under later events. It needs an
  edit modal per history row that reuses the same validation.
- **Rolls as one action.** A roll today is a buyback plus a new sale: two entries. A single "Roll"
  on an open leg (close + reopen, same date) would halve the typing. Decide first whether a roll's
  net credit should be shown as one line.
- **Live marks.** Calc has Finnhub. Here the last price is typed by hand per position.
- **Deploy.** GitHub Pages like Calc and Books, if wanted. Needs a repo name.

## Decided

- **Buying power is conservative.** Premium on a short option that's still open is left out until
  that option closes. That's why BP right after a sale is `value − collateral − fees`, not
  `+ credit`. It's the spec's "new cash ready to be used, once it is closed".
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
- **Optimizer objective** is net credit (after per-contract fees) per week-equivalent, under the
  capital constraint. With one expiry it reduces to the largest total net credit. Rows below the
  weekly minimum are excluded, not just flagged. It's an exact bounded knapsack, coarsened (never
  overspending) only when capital divided by the strikes' common step exceeds 200k units.
- **Bring in existing shares.** "Premium already collected" lowers basis but not account P/L,
  because it's already inside the starting value.
