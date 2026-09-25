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
Stock/ETF prices are fetched on demand — the user triggers a refresh
(app open / manual pull-to-refresh), anytime during the day or market
session — via Yahoo Finance (using the unofficial `yfinance`-style
access, since Yahoo has no official public API). This is "current price
as of last refresh," not a continuous live tick stream. Cash balances
and holdings are entered manually, and updated whenever the user
triggers it — no fixed schedule, no reminders required for v1. All
values are in a single currency.

A simple manual export/import (to a file) is included in v1 as a backup
safety net, since there's no cloud backup by default.

The app is fully self-contained on the device: both the UI and the local
data store (holdings, cash balances, history) run on the phone, with no
remote backend or server-side account. This keeps setup simple and
minimizes the data's exposure, at the cost of no automatic cross-device
sync or cloud backup for v1 (data is lost if the device is lost, unless a
manual export/import feature is added later).

A future version may add an in-app web-style dashboard view — opened only on
the same device (e.g. a locally served page or embedded webview), not a
separate computer/browser — plus automatic account syncing if an
API/aggregator becomes available. Both are explicitly out of scope for
v1. Note: this same-device web view does not require cross-device sync
or a remote backend, and does not change the "on-device only" constraint
above. A dashboard accessible from a separate computer/browser would
require a remote backend and is a distinct, larger feature not currently
planned.

## Affected users and systems
- User: me only (single user, personal use — no multi-user/auth-sharing
  needed; access control is just securing the device/app itself)
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
  continuously — freshness is "as of last refresh"
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
None outstanding. All open questions from the initial draft have been
resolved (market data provider, update cadence, backup/export) and are
reflected in Proposed outcome and Constraints above.
