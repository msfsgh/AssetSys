# Intent: Personal portfolio tracker
Author: [your name]. Status: draft, open questions resolved — ready for review.

## Problem
I have no single place to see my overall financial position. My cash sits
with my bank, and my stocks/ETFs sit with one brokerage. To know my current
net worth or how a position is doing, I have to open multiple apps and do
the math myself. There's no 24/7, at-a-glance view of what I own and what
it's currently worth.

## Proposed outcome
A mobile app, for personal use only, that shows:
- Total net worth at a glance (cash + stock/ETF market value combined)
- A breakdown by holding: each cash balance and each stock/ETF position,
  with current market value
- History/trends over time (e.g. net worth and per-holding value changing
  across past refreshes) — this is a fuller app the user expects to spend
  time in, not just a quick glance, so historical data must be retained
  and viewable, not just the current snapshot
- One cash account, but support for more than one stock/ETF portfolio
  (e.g. grouped separately rather than one flat list across portfolios).
  Each portfolio has a custom user-assigned name (e.g. "Retirement",
  "Brokerage A"), which can be renamed after creation. A portfolio can
  also be deleted after creation. Each portfolio's ledger is fully
  independent/isolated — no transferring a holding between portfolios
  in v1

**Ledger model**
- Both cash and stock/ETF holdings are tracked as transaction ledgers,
  not editable snapshots: cash changes are logged as deposit/withdrawal
  entries, and stock/ETF changes are logged as buy/sell events (ticker,
  quantity, total cost/amount, date — not a per-share price; per-share
  price, if shown, is derived by dividing total by quantity)
- The current cash balance and each stock/ETF position update
  automatically and immediately from the ledger the moment a
  transaction is logged — there is no separate manual balance field to
  keep in sync and no extra step to recalculate
- Every ledger entry has an effective date, and entries of any type
  (buy, sell, split adjustment, direct cash movement) are always
  applied in date order to compute balances/positions and historical
  trends — not the order they were typed in. This means a backdated
  entry (e.g. a split you forgot to log until later) is inserted at its
  correct point in time and recalculates everything from that date
  forward correctly
- Both ledgers are append-only — corrections are made via new
  offsetting transactions, not by editing or deleting past entries, so
  historical trends stay accurate
- A sell transaction that would take a position below zero (more shares
  than currently held) is rejected, not allowed

**Cash ledger entry types**
- The cash ledger has four kinds of entries: two generated automatically
  (a withdrawal from a buy, a deposit from a sell) and two entered
  directly by the user (a direct withdrawal — cash leaving the system
  entirely, e.g. spent or transferred out — and a direct deposit — cash
  entering the system, e.g. a paycheck or bank transfer in)

**Stock/ETF ledger: buy/sell and split adjustment**
- Buy/sell events capture ticker, quantity, total cost/amount, and date
- A third stock/ETF ledger entry type, split adjustment, handles stock
  splits:
  - Adjusts a holding's quantity by a split ratio as of a given date,
    entered as a single multiplier (e.g. 2.0 for a 2-for-1 split, 0.1
    for a 1-for-10 reverse split)
  - No cash impact and no effect on total cost recorded to date — it
    only corrects quantity/share-count going forward, the same way a
    buy or sell would, but without a cash-linked transaction
  - Supports both forward splits (e.g. 2-for-1, increasing share count)
    and reverse splits (e.g. 1-for-10, decreasing share count), at any
    ratio, not just whole-number multiples — which can produce a
    fractional share count (e.g. a 3-for-2 split on an odd number of
    shares), so stock/ETF quantities support fractional/decimal values
    throughout, not just whole shares
  - A split adjustment for a ticker/portfolio with zero position as of
    that date is rejected, the same way an oversell is rejected — you
    can't split a position you don't hold
  - If a split and a buy/sell share the same effective date, the split
    is applied first, so the same-day buy/sell is interpreted in
    post-split terms
  - A wrongly-entered split is corrected the same way as any other
    ledger mistake — via a new offsetting entry, here a split
    adjustment with the reciprocal ratio (e.g. an erroneous 2.0 is
    corrected with 0.5) — never by editing or deleting the original entry
  - Split detection is fully manual for v1: the app does not attempt to
    auto-detect splits from Yahoo Finance or any other source
  - If the same ticker is held in more than one portfolio, a split must
    be logged separately in each portfolio that holds it, consistent
    with portfolios being fully isolated ledgers
  - Stock dividends and spin-offs are different corporate actions (a
    stock dividend issues shares as a dividend rather than adjusting
    existing quantity; a spin-off distributes shares of a different
    company entirely) and are out of scope for v1 — only true splits
    (forward or reverse) are handled

**Buy/sell ↔ cash linkage**
- A stock/ETF buy or sell transaction automatically creates the
  matching cash movement in the same action — buying withdraws the cost
  from the single shared cash account, selling deposits the proceeds
  into it — so cash and portfolio ledgers always stay in sync with one
  entry, not two
- This is the one link between an otherwise-isolated portfolio and the
  cash account: portfolios don't share positions with each other, but
  they all draw from and return to the same cash account
- A buy transaction that costs more than the current cash balance is
  rejected, the same way an oversell is rejected
- A direct cash withdrawal that would take the cash balance below zero
  is rejected, the same way an oversell or an over-budget buy is
  rejected

**Market data & offline behavior**
- Stock/ETF prices are fetched on demand — the user triggers a refresh
  (app open / manual pull-to-refresh), anytime during the day or market
  session — via Yahoo Finance (using the unofficial `yfinance`-style
  access, since Yahoo has no official public API)
- This is "current price as of last refresh," not a continuous live
  tick stream
- When offline or a refresh fails, the app shows the last-known prices,
  clearly marked as stale/offline, rather than blocking the view

**Manual entry & currency**
- Cash balances and holdings are entered manually, and updated whenever
  the user triggers it — no fixed schedule, no reminders required for v1
- All values are in a single currency

**Backup**
- A simple manual export/import (to a file) is included in v1 as a
  backup safety net, since there's no cloud backup by default

**On-device architecture**
- The app is fully self-contained on the device: both the UI and the
  local data store (holdings, cash balances, history) run on the phone,
  with no remote backend or server-side account. This keeps setup
  simple and minimizes the data's exposure, at the cost of no automatic
  cross-device sync or cloud backup for v1 (data is lost if the device
  is lost, unless a manual export/import feature is added later)

**Deferred to a future version**
- Support for stock dividends and spin-offs — v1 handles only true
  stock splits (forward or reverse)
- Transferring a holding between portfolios — v1 treats each
  portfolio's ledger as fully isolated
- Alerts/notifications (e.g. price threshold or portfolio drop alerts)
  — v1 is view-on-demand only, no background monitoring or notifications
- An app-level passcode/biometric unlock (on top of the phone's own
  lock screen) for extra protection of this sensitive data — v1 relies
  on the device's own lock screen only
- An in-app web-style dashboard view — opened only on the same device
  (e.g. a locally served page or embedded webview), not a separate
  computer/browser — plus automatic account syncing if an
  API/aggregator becomes available. Note: this same-device web view
  does not require cross-device sync or a remote backend, and does not
  change the "on-device only" constraint above. A dashboard accessible
  from a separate computer/browser would require a remote backend and
  is a distinct, larger feature not currently planned

## Affected users and systems
- User: me only (single user, personal use — no multi-user/auth-sharing
  needed; access control is just securing the device/app itself)
- Structure: one cash account with its own deposit/withdrawal ledger;
  one or more separate, custom-named stock/ETF portfolios, each holding
  its own transaction ledger; portfolios can be renamed or permanently
  deleted
- New system: a mobile app with an on-device local data store (e.g. a
  local SQLite database) — no remote server or hosted backend
- External dependency: Yahoo Finance, accessed via the unofficial
  `yfinance`-style method (no official Yahoo API exists), called on demand
  (not streamed) — this is a real technical risk since Yahoo can change
  its site/endpoints without notice and break data access; see Constraints
- On-device export/import (backup) of the local data store
- No integration with the bank or brokerage systems in v1 (manual entry only)

## Constraints
- No trading or transaction execution of any kind — read/tracking only
- No brokerage or bank API integration in v1 — all holdings and cash
  balances are entered and updated manually by me, whenever I choose to
  (no fixed schedule or reminders needed for v1)
- Market prices are refreshed on demand (user-triggered), not streamed
  continuously — freshness is "as of last refresh." When offline or a
  refresh fails, the app must show last-known prices marked stale, not
  block the view or show nothing
- The app retains historical data (not just current snapshot) so
  net-worth and per-holding trends over time can be shown, except where
  a whole portfolio is deliberately deleted (see below)
- Both cash and stock/ETF holdings are recorded as append-only
  transaction ledgers (deposits/withdrawals for cash; buy/sell events
  for stock/ETF); balances/positions are always calculated from the
  ledger and update automatically the moment a transaction is logged —
  never directly edited or set as a separate field. Mistakes are
  corrected via new offsetting transactions, never by editing or
  deleting past entries. A sell transaction that would take a position
  below zero is rejected
- The cash ledger supports four entry types: buy-linked withdrawal and
  sell-linked deposit (both auto-generated, see below), plus direct
  withdrawal and direct deposit (entered manually, representing cash
  leaving or entering the system entirely — e.g. spending, a paycheck,
  or a bank transfer)
- Stock/ETF ledgers support a third entry type beyond buy/sell: a split
  adjustment, which corrects a holding's quantity for a stock split (no
  cash impact, no change to previously recorded cost), entered as a
  single multiplier (e.g. 2.0 for 2-for-1, 0.1 for 1-for-10). Both
  forward and reverse splits are supported, at any ratio, which may
  produce fractional share counts — stock/ETF quantities support
  fractional/decimal values throughout. A split for a ticker/portfolio
  with zero position at that date is rejected, the same way an oversell
  is rejected. If a split and a buy/sell share the same effective date,
  the split applies first. A wrongly-entered split is corrected via an
  offsetting split adjustment with the reciprocal ratio, never by
  editing or deleting the original entry. Split detection is fully
  manual for v1 — no auto-detection from Yahoo Finance or elsewhere. A
  split affecting a ticker held in more than one
  portfolio must be logged separately in each. Stock dividends and
  spin-offs are distinct corporate actions and out of scope for v1
- Every ledger entry (any type, either ledger) has an effective date and
  is applied in date order — not entry order — when computing balances,
  positions, and historical trends, so a backdated entry is correctly
  inserted at its point in time and recalculates everything from there
- A stock/ETF buy or sell transaction automatically creates the matching
  cash withdrawal/deposit in the single shared cash account, in the same
  action — portfolios are isolated from each other's positions, but all
  share and draw from the one cash account. A buy that costs more than
  the current cash balance is rejected
- A direct cash withdrawal that would take the cash balance below zero
  is rejected, the same way an oversell or an over-budget buy is rejected
- Portfolios can be renamed after creation. Deleting a portfolio is a
  deliberate, permanent action: it erases that portfolio's entire ledger
  and historical trend data, unlike a within-portfolio correction, which
  must go through an offsetting transaction rather than deletion
- Market data comes from Yahoo Finance via unofficial/undocumented access
  (no official Yahoo API exists, so this is not a supported integration).
  Accepted risk: Yahoo can change its site and silently break data
  access; the app should isolate this dependency (e.g. behind a single
  data-source module) so it can be patched or swapped without a rebuild
  of the rest of the app
- All data (holdings, cash, history) is stored locally on the device only;
  no remote backend or account system in v1. A manual export/import to a
  file is provided as a backup safety net
- This is sensitive personal financial data — must not be exposed publicly
  or synced anywhere by default
- Single currency only for v1 (no FX conversion needed)
- Single brokerage (stocks/ETFs) + single bank (cash) for v1 — not designed
  for multiple institutions yet

## Open questions
None outstanding. Every question raised while drafting this intent —
market data provider, update cadence, backup/export, on-device storage,
the transaction-ledger data model (cash and stock/ETF), portfolio
structure and lifecycle (naming, renaming, deletion), buy/sell↔cash
linkage and rejection rules, offline/stale-price handling, and stock
split handling (ratio format, fractional shares, cross-portfolio
logging, same-date ordering, corrections) — has been resolved and is
reflected in Proposed outcome and Constraints above.
