# Finance Asset Tracking System (Personal)

A personal Android app for tracking investments and cash holdings across
multiple accounts — giving a single, accurate, up-to-date view of net
worth without relying on manual spreadsheets or logging into multiple
brokerage sites.

---

## Why This Exists

Most brokerage apps only show you what's held with them. If you hold
assets across multiple institutions — or want to track cash alongside
stocks — you end up juggling multiple logins or maintaining a spreadsheet
that's always slightly out of date.

This app solves that by being the one place where everything lives:
stocks, cash, multiple currencies, multiple portfolios — updated
automatically where possible, and easy to enter manually where not.

---

## What It Does

- **Tracks stocks and cash accounts** across as many portfolios as you
  need (e.g. personal vs. joint, or by goal/strategy).
- **Pulls stock prices automatically** while the app is open, using
  Google Finance or Yahoo Finance — configurable per portfolio.
- **Shows net worth and allocation** at a glance on a dashboard — the
  grand total net worth is shown together with its breakdowns on the
  same screen: how much each portfolio contributes to the total, and
  an allocation breakdown by asset type, account, or currency.
- **Tracks cost basis per lot** using the specific-lot method — when
  you sell, you choose exactly which purchase lot(s) the sale draws
  from, giving you full control over realized gain/loss.
- **Handles partial sells and stock splits** correctly, preserving a
  full audit trail of every change.
- **Supports multiple currencies** with automatic conversion to USD for
  net worth totals.
- **Records historical snapshots** so you can see how your holdings
  changed over the last 3, 6, or 12 months.
- **Imports and exports via CSV** for bulk data entry or backup.
- **Supports full device migration** — export everything to a single
  file and restore it on a new phone.

---

## What It Doesn't Do

This is a tracking tool, not a trading platform. It will not:

- Execute trades or connect to brokerage accounts automatically.
- Sync with bank accounts (deposits and withdrawals are entered manually).
- Provide tax reports or compliance features.
- Run on iOS, web, or desktop (Android only, for now).
- Store or sync data to a server — everything stays on your device.

---

## How It Works (Non-Technical Overview)

All data is stored locally on your Android phone — there is no account
to create, no server, and no subscription. Prices are fetched
periodically while the app is open (roughly every 30–60 seconds) from
publicly available finance endpoints. If the app is offline or a price
can't be fetched, the last known price is shown with a clear "stale"
indicator so you always know what you're looking at.

The transaction ledger (every buy, sell, split, and cash movement) is
encrypted on the device and protected against accidental changes —
edits are tracked as amendments rather than silent overwrites, and
deletions require confirmation.

Exporting your full data (for moving to a new phone) requires biometric
authentication before the file is generated.

---

## Key Concepts

| Term | What it means here |
|---|---|
| **Portfolio** | A named group of assets and cash accounts (e.g. "Personal", "Joint") |
| **Lot** | A single purchase of a stock — tracked separately for cost-basis purposes |
| **Specific-lot method** | When selling, you choose which purchase(s) the sale draws from |
| **Cost basis** | What you originally paid for a lot, used to calculate gain/loss |
| **Stale price** | The last successfully fetched price, shown when live data isn't available |
| **Stock split** | When a company adjusts its share count (e.g. 2-for-1); the app adjusts your holdings accordingly |
| **Device migration** | A full export of all your data to move to a new phone |

---

## Project Status

This is a personal/small-team tool currently in active development (v1 / MVP).
It is not yet available on the Play Store. The initial version targets
Android 14 (API 34) and above.

---

## Tech Stack (for collaborators)

- **Language:** Kotlin
- **UI:** Jetpack Compose + Navigation Compose
- **Storage:** Room / SQLite (local only, encrypted via SQLCipher or
  Jetpack Security)
- **Price data:** Google Finance / Yahoo Finance (unofficial endpoints,
  pluggable interface)
- **FX rates:** exchangerate-api.com (free tier), ECB daily rates as fallback
- **Min SDK:** 34 (Android 14)

---

## Open Questions / Known Gaps

A few things are still being decided before or during development:

- How API keys are stored securely on-device
- How often historical snapshots are taken (e.g. daily via WorkManager,
  or only on app open)
- How fractional shares from a reverse stock split are handled

See `CLAUDE.md` or `copilot-instructions.md` for the full list of settled and open decisions.

---

## Contributing

This is primarily a personal project, but contributions, suggestions, and
issue reports are welcome. Please read `CLAUDE.md` or `copilot-instructions.md` before contributing —
it contains all settled design decisions and working conventions that
should be followed to keep the codebase consistent.
