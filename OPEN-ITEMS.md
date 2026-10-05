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
  still isn't *P/L*, so account value only moves when the leg closes. The first version left open
  credit out entirely. That made BP go negative right after following the selector's advice, so
  it was reversed.
- **No max-put-strike box.** Removed at Steve's request, since the selector covers it. The formula
  is still `(BP − fee) / 100 + credit`, if it's ever wanted back as a one-liner.
- **Expiry is always a Friday dropdown of exactly 8 choices**, in the selector, section 03 and the
  position forms. In 03 and the forms the list starts at the trade date's Friday, so a back-dated
  sale gets back-dated choices. A stored expiry that has dropped off the list replaces the
  furthest Friday rather than adding a 9th. Checked across 400 consecutive start dates.
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
  brute force on 300 random lists. The result box leads with the gain as a % of account value, and
  adds the % on capital-to-deploy when that figure differs.
- **No per-put return floor** (removed 2026-10-05, at Steve's request). The old "min weekly
  return %" setting gated each put on its own yield, which could leave cash idle that a low-yield
  put would have put to work. The goal is total account gain, so every put competes on the dollars
  it adds. Rows are excluded only for: expired, fees ≥ credit, too big. They're checked in that
  order, so "too big" is never hidden behind a yield label.
- **Buy-to-close targets** (2026-10-05). Every open short option shows a "Close at" limit for today
  and a schedule for each session to expiry. Time is counted in trading sessions (a Mon→Fri put has
  5). With h = target weekly return (an account setting, default 1%; it does *not* filter the
  selector) and K = strike for a put or adjusted basis for a call, the target is the lower of:
  - remaining: `h·K·(T−t)/5 − 2·fee`. The premium still to come earns under h, so redeploying wins.
    Fees count the close and the reopen.
  - floor: `C − fees − h·K·max(t,1)/5`. Closing still banks h for the sessions held.

  The result is rounded down to the cent. "Hold" when neither leaves a positive price, or h is 0.
  For credits paying more than h, the remaining rule binds and always sits (1 − h/ρ) ahead of the
  straight-line pace. For credits paying less than h, the floor binds. Hand-checked: a $50 put at
  $0.60 gives $0.48 / .38 / .28 / .18 / .08 Mon→Fri, and at $0.40 gives .28 / .28 / .18 / .08 /
  hold.

  Open questions it doesn't model: tick size (some chains only fill in $0.05 steps), holidays (only
  weekends are skipped), and whether mid-week redeployment at h is actually available. That last
  one is the assumption the remaining rule rests on.
- **Max per symbol %** (2026-10-05, for diversification in larger accounts). Optional; blank means
  no cap. The cap is `maxAlloc% × account value`. A symbol's exposure is its shares at cost plus
  collateral on its open short puts, across every open position on that symbol. Calls add nothing.
  - **Selector:** each symbol gets `room = cap − exposure`. A row whose single contract won't fit
    shows "at cap". Rows on the same symbol share the room, so the optimizer is now a grouped
    (multiple-choice) knapsack: each group holds the contract mixes that fit the symbol's room.
    It was brute-force checked on 300 random lists with mixed caps. The result box names each
    symbol the cap touched: shut out (already held, or a single contract over the cap) or held back.
  - **Sell a Put and the position "Sell a put" form:** an amber flag, not a block.
  - **Position cards:** a "% of acct" pill, amber when over the cap.

  Steve's examples: 100 ABC at $50 against a 25% cap on $10k filters ABC out. At $10 a share, puts
  up to $15 are eligible and not above.
- **One shared put list, run per account** (2026-10-05, after Steve's first live week showed the
  same puts being typed into every account). The list is `root.cands`. Old per-account lists are
  merged into it on load, deduplicated by symbol, strike, credit and expiry. The selector runs it
  once per account against that account's buying power or capital-to-deploy, fees and per-symbol
  cap. The active account is shown first.
  - **Picks column:** one chip per account (take N / pass / sold N / too big / at cap), named when
    there's more than one account.
  - **Capital to deploy:** a box per account, now inside its result block.
  - **Sell:** switches to that account before prefilling. A recorded sale no longer deletes the
    row. It adds to `c.sold[accountId]`, which comes off that account's max qty, so other accounts
    still see the row.
  - **Net/contract:** uses the active account's fee; it's the only per-account column.
- **CSV import** fills the list. Prices come from the CSV itself. Steve ruled out a quote API, so
  there's no Finnhub here, deliberately. It finds Symbol/Ticker and Price/Last headers; with no
  header, column 1 is the symbol and the first number after it is the price. It handles comma,
  semicolon or tab, quotes, and `$`.
  - **Filter:** a ticker is left out when no account could hold one contract at the bottom of the
    strike band, `price × (1 − band%)` × 100, against `min(capital, cap room)`. Band is a shared
    setting, default 10%, for puts just outside 40 delta. The credit offset is ignored because
    it's unknown at import.
  - **Title lines before the header** (scanner exports open with "Watchlist Scanner", "Results")
    are skipped. Rows narrower than the table's usual width are ignored, and the header row is
    searched for in the first 15 rows. Header words are never taken as tickers. Tested on Steve's
    real export: 32 added, 33 left out, all priced, for a $10k account.
  - **Already-listed tickers** get their price updated, not duplicated. A re-import re-checks
    size and removes oversized rows that were never filled in. It also sweeps unfilled rows named
    after header words. Tickers with no price are
    added unchecked and flagged.
  - **The strike box** placeholder shows the band floor. "Remove unfilled" clears rows left blank.
- **Selector picks are listed best first** by weekly return on collateral (ties by net), numbered,
  so the first one is the one to sell first. A dollar ordering would just rank the high strikes first.
- **Collapse all / Expand all** on Open Positions. It's one button showing whichever action
  applies.
- **A fixed "Capital to deploy" shrinks as you sell** from section 03, by the cash each sale used.
  Blank means "all of buying power" and follows BP on its own.
- **No start date.** Dropped at Steve's request. The equity chart starts at the first event.
- **Named Wheeler** (repo name). The storage key stays `wheel.v1`: it's invisible, and renaming it
  would strand data already entered on the live site.
- **Bring in existing shares** takes the unrecorded put as contracts, credit per share, buyback
  debit per share, and total fees. That covers "sold a put, bought it back deep in the money, then
  bought shares". Its net always comes off the basis. A select decides whether it also counts in
  account P/L: "already in my starting value" (the default, and how older data is read) or
  "happened since" (counted on the share date). Stored as `carry`, `carryIn`, `carryDate`,
  `carryParts`.
