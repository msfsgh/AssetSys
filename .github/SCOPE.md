# Scope — Finance Asset Tracking System (v1 / MVP)

## In Scope

| Feature | Notes |
|---|---|
| Native Android app | Kotlin, Android-only for v1 |
| Local on-device storage | Room/SQLite; no backend, no cross-device sync |
| Multiple portfolios per user | Assets/cash accounts organized by portfolio; per-portfolio settings (e.g. price data source) and per-portfolio or combined reporting |
| Automated stock price updates via API | User-configurable per portfolio: Google Finance or Yahoo Finance (unofficial APIs — pluggable interface recommended) |
| Foreground price polling | Poll every 30–60s while app is open for current price/market value; not background or streaming real-time |
| Automated cash balance updates | From stock buy/sell settlements only |
| Manual entry of assets & cash transactions | Cash deposits/withdrawals; fallback for sources without API access |
| Stock buy/sell transaction entry | Minimal input: date, type, ticker, shares, total cost (fees included); per-share cost basis derived, not entered |
| Cost-basis tracking | Specific-lot method — user picks the lot(s) when selling via a lot-picker (pre-selected oldest-first, user can override), not auto FIFO/LIFO/average |
| Stale price handling | Last-known price shown with a "stale" indicator/timestamp when offline or the price API fails, instead of blocking price-dependent screens |
| Multi-currency support | Store amounts with explicit currency; convert to USD (base currency) via exchangerate-api.com (default) for totals |
| Historical value snapshots | e.g. monthly, for trend tracking |
| Asset categorization & tagging | By type, account, currency, risk category |
| Simple dashboard | Grand total net worth shown together with its breakdowns on one screen: per-portfolio contribution and allocation chart (type/account/currency/risk); not just a total with a drill-in switcher |
| CSV import/export | For bulk entry and backup |
| Full data export/import (device migration) | Single-file export of all portfolios, assets, snapshots, and settings, for moving to a new phone; distinct from CSV import/export |
| Stock split handling | Forward and reverse splits, entered manually by the user (date + ratio); adjusts share count and cost basis for active/open holdings only (closed lots unaffected); tracked as explicit events |
| Transaction ledger protection | Encrypted at rest; immutable/audit-trailed edits; soft-delete with confirmation to prevent accidental data loss |
| Export access control | Full data export (device migration) requires biometric/app-lock authentication before proceeding; exported file itself is plaintext (biometric gate is the protection, not file encryption) — deliberate tradeoff |

## Out of Scope (for v1)

| Feature | Reason |
|---|---|
| Background/streaming real-time market data | Foreground polling (30–60s) is sufficient; true streaming needs a paid provider and adds complexity |
| Automated trading or brokerage execution | Out of scope entirely for this tool |
| Bank account API integration (e.g. Plaid) | Requires backend service; conflicts with local-only storage decision |
| Crypto and other asset types beyond stocks/cash | Future enhancement, not v1 |
| Multi-user permissions/roles | Single user / small team only |
| Tax reporting / compliance features | Separate concern, not core tracking |
| Backend/server component, cross-device sync | Local-only storage decision for v1 |
| iOS / web / desktop clients | Android-only for v1 |

## Success Criteria

- Add or update an asset in under a minute.
- View total net worth and allocation breakdown at a glance.
- View how holdings changed over the last 3, 6, and 12 months.
- No code or direct database access needed for routine data entry.

## Revisiting Scope

If priorities shift (e.g. automatic price feeds become a must-have),
update this file and reflect the change in `copilot-instructions.md`'s "Scope" summary
so both stay in sync.
