# Finance Asset Tracking System (Personal)

## Project Goal

Build a lightweight system to track investments, cash, and securities across
accounts — giving a single, accurate, up-to-date view of net worth and
holdings without relying on manual spreadsheets or scattered brokerage
logins.

**Primary user:** Just me / a small team (not a multi-tenant product).

**Platform:** Native Android app (Kotlin). Android-only, no iOS/web/desktop
target for v1.

## Core Objectives

1. **Centralize visibility** — see all assets (cash and stocks) in one
   place, regardless of which institution holds them.
2. **Track value over time** — record historical snapshots to see
   growth/decline trends, not just a current-state number.
3. **Categorize holdings** — group by asset type, account, currency, or
   risk category for easier analysis.
4. **Support multiple portfolios** — organize assets and cash accounts
   into more than one portfolio (e.g. personal vs. joint, or by goal/
   strategy), each trackable and reportable on individually or combined.
5. **Automate updates via API** — pull prices and, where possible, account
   balances automatically rather than relying on manual entry. Manual entry
   and CSV import remain available as a fallback/supplement.
6. **Handle stock splits** — correctly adjust share count and cost basis
   for both forward splits (e.g. 2-for-1) and reverse splits (e.g. 1-for-10),
   without distorting historical value snapshots taken before the split.
7. **Support basic reporting** — net worth summary, allocation breakdown
   (e.g. % stocks vs cash), and gain/loss over a period.
8. **Support multiple currencies** — hold and report on assets across
   different currencies, with conversion to a base currency for net worth
   totals.

## Scope

See `SCOPE.md` for the detailed in-scope / out-of-scope table for v1 (MVP).

Short version:
- **In scope (v1):** native Android app, automated price/balance updates via
  API, manual asset entry & CSV import as fallback, historical snapshots,
  tagging, multi-currency support, simple dashboard.
- **Out of scope (v1):** automated trading, multi-user roles/permissions,
  tax reporting, iOS/web/desktop clients.

## Success Criteria

- Add or update an asset in under a minute.
- See total net worth and allocation breakdown at a glance.
- View how holdings changed over the last 3 / 6 / 12 months.
- No direct database/code editing required for routine data entry.

## Build Order (v1)

Recommended phase ordering for implementation. Each phase should be usable
and testable on its own before moving to the next — this is not a strict
waterfall, but later phases depend on earlier ones being stable.

1. **Phase 0 — Foundation.** Room/SQLite data model (Portfolio, Asset, Lot,
   Transaction, CashAccount) with multi-currency and multi-portfolio
   support built in from the start — not bolted on later. Basic `Migration`
   class established even for schema v1. Single-activity Compose shell with
   Navigation Compose and the four bottom-nav tabs (screens can be empty).

2. **Phase 1 — Manual walking skeleton.** Portfolio Management; Cash
   Account Detail with manual deposit/withdrawal only (no buy/sell
   settlement yet); Add/Edit Transaction (5-field buy/sell form) without
   the lot-picker (single-lot buys only); Asset Detail; Dashboard showing
   grand total + per-portfolio breakdown computed from manually-entered
   data, using cost basis as value (no live prices yet). This is the
   smallest end-to-end loop that satisfies the "add an asset in under a
   minute" success criterion and proves dashboard aggregation, fully
   offline, before price-fetching is introduced.

3. **Phase 2 — Cost-basis correctness (sells).** Lot Picker step in the
   sell flow (oldest-first default, user override); partial-sell lot
   splitting (closed + remaining records); subtraction-method rounding;
   overselling validation; cross-currency sells. This is the highest-risk
   financial logic in the app — get it right and unit-tested in isolation
   before live prices are layered on top, since cost-basis drift is much
   harder to spot than a stale-price bug.

4. **Phase 3 — Live data.** Pluggable price-provider interface
   (Google/Yahoo Finance) with foreground polling (30–60s); stale-price
   indicator and timestamp; offline handling; FX rate integration
   (exchangerate-api.com + ECB fallback); allocation breakdown chart on
   the Dashboard (now meaningful with live values). Depends on Phase 1's
   dashboard and Phase 2's accurate holdings. This is the highest
   external-risk area (unofficial APIs) — keep it behind the interface so
   a broken endpoint doesn't take down the rest of the app.

5. **Phase 4 — Corporate actions.** Split Entry screen (manual date +
   ratio); split adjustment logic for open lots; amendment/audit-trail
   entries for splits interacting with partial sells. **Blocked on
   resolving the open fractional-share-handling decision (see Open
   Decisions) — resolve before starting this phase.** Touches the same lot
   records as Phase 2, so it's safer to build once that logic is stable.

6. **Phase 5 — Protection & portability.** Transaction ledger encryption
   at rest (SQLCipher / Jetpack Security); immutable/amendment-based edit
   model with soft-delete and confirmation; CSV Import/Export; Full Data
   Export/Import (device migration) gated behind BiometricPrompt;
   historical snapshots. **Blocked on resolving the open snapshot
   frequency/trigger decision (see Open Decisions) before implementing the
   snapshot part of this phase.** Placed after core functionality is
   correct rather than first, since retrofitting encryption onto a schema
   that's still actively changing is wasted work.

7. **Phase 6 — Reporting & polish.** Reports tab (gain/loss over a period,
   allocation breakdown, per-portfolio or combined); Settings tab
   (provider choice, polling interval, API key management); a performance
   pass at realistic data volumes (hundreds/thousands of lots and
   transactions).

**Note:** Phase 2 (cost-basis) is deliberately sequenced before Phase 3
(live prices) — it's pure logic that can be unit-tested without any
external dependency, and it's the part most likely to have subtle bugs.
Phase 5 (encryption) is deliberately sequenced after functional
correctness rather than first; if development-time peace of mind about
data-at-rest matters more than avoiding schema churn, this tradeoff can be
revisited and encryption moved earlier — check with the user before doing
so.

## Decisions (settled)

- **Price/balance updates:** Fully automated via API (prices, and account
  balances where the source supports it). Manual/CSV entry available as
  fallback for sources without API access.
- **Delivery form:** Native Android app (Kotlin). No iOS/web/desktop
  client for v1.
- **Currency:** Multi-currency support required, with conversion to a
  base currency for net worth totals.
- **Tech stack:** Kotlin, native Android. Backend/storage/DB choice still
  open — see below.
- **Storage:** Local-only, on-device (Room/SQLite). No backend service,
  no cross-device sync for v1.
- **Asset types (v1):** Public stocks and cash accounts. Crypto and other
  asset types are future enhancements, not v1.
- **Portfolios:** Multiple portfolios per user are supported (e.g.
  personal vs. joint, or by goal/strategy). Stock price provider and
  other settings are configurable per portfolio. Reporting should support
  both per-portfolio views and a combined/total view across portfolios.
- **Dashboard content:** The Dashboard must show the grand total net
  worth (summed across all portfolios and cash accounts, converted to
  USD) together with its breakdowns on the same screen — a
  per-portfolio breakdown (how much each portfolio contributes to the
  total) and an allocation breakdown (by asset type, account, currency,
  risk category). These breakdowns are displayed alongside the grand
  total, not hidden behind a switcher that only shows one portfolio at
  a time. A portfolio switcher/list can still exist for jumping to a
  single Portfolio Detail screen, but it supplements the combined view
  rather than replacing it.
- **Device migration:** Support moving the app (with all data) to a new
  Android phone via a full data export/import file — a single file
  containing all portfolios, assets, cash accounts, historical snapshots,
  and settings. Separate from the CSV import/export feature (which is for
  bulk data entry, not full backups). The exported file can be saved
  anywhere the user chooses (e.g. via Android's Storage Access Framework —
  local storage, Google Drive, email, etc.); this does not require a
  backend or ongoing sync, and doesn't conflict with the local-only
  storage decision.
- **Stock splits:** Must support forward splits (e.g. 2-for-1) and
  reverse splits (e.g. 1-for-10). When a split occurs, share count and
  cost-basis-per-share must be adjusted so total position value is
  unaffected at the moment of the split. Only currently active/open
  holdings are adjusted — already-closed (sold) lots stay as originally
  recorded and are not retroactively adjusted. Historical snapshots
  recorded before the split must remain accurate as of the time they were
  taken (i.e. don't retroactively rewrite pre-split history using
  post-split share counts) — the split-adjustment should apply going
  forward, with the historical record preserved as originally captured,
  unless the price data source itself provides split-adjusted historical
  prices. **Split entry:** manual only — the user enters split events
  (date, ratio) themselves; the app does not attempt to auto-detect
  splits from the price data source.
- **Transaction ledger protection:** The stock transaction ledger
  (buys, sells, splits, cash settlements, etc.) must be:
  - **Encrypted at rest** on the device (not stored as plaintext
    SQLite/Room data).
  - **Immutable/audit-trailed** — edits to a transaction create a new
    record or amendment entry rather than silently overwriting the
    original; the history of changes must be reconstructable.
  - **Protected from accidental data loss** — deleting a transaction
    requires explicit user confirmation, and deletions should be
    soft-deletes (recoverable) rather than immediate hard-deletes, where
    practical.
- **Export access control:** Performing a full data export (device
  migration file, see above) requires app lock / biometric
  authentication (e.g. Android BiometricPrompt) before it proceeds. This
  applies specifically to the export action, not general app access —
  whether the app should also require unlock just to open/view data is a
  separate, not-yet-requested decision.
- **Export file format:** The exported file itself is plaintext (not
  encrypted) — the biometric gate on the export action is considered
  sufficient protection. This is a deliberate tradeoff, not an oversight:
  once exported, the file is protected only by wherever the user chooses
  to store it (device storage, Google Drive, email, etc.), not by the
  app. Do not "fix" this into an encrypted export without checking with
  the user first.
- **Cash account handling:** Balance changes from stock buy/sell
  settlements are calculated automatically. Deposits/withdrawals are
  entered manually (no bank account API integration — out of scope given
  local-only storage; bank APIs like Plaid typically require a backend).
- **Stock price data source:** User-configurable per portfolio, choice of
  Google Finance or Yahoo Finance endpoints. **Risk:** neither has an
  official/stable public API — both are unofficial endpoints that can
  change or break without notice. Build with a pluggable data-source
  interface so the app can swap providers without a rewrite.
- **Price refresh rate:** Poll periodically while the app is open/in the
  foreground (e.g. every 30–60 seconds) for current prices and market
  value. Not true streaming/websocket real-time, and does not poll in
  the background when the app is closed (see snapshot frequency, still
  open, below). Polling interval should be configurable/throttleable in
  case a provider starts rate-limiting.
- **Base currency:** USD.
- **FX rate source:** exchangerate-api.com (free tier) as default, with
  ECB daily rates as a fallback. Should be swappable like the stock price
  source.
- **Stock buy/sell transaction input:** User provides only these fields
  per transaction — transaction date, type (buy/sell), stock ticker,
  number of shares, and total cost (inclusive of fees). No separate
  fee/commission field, no per-share price field — per-share cost basis
  is derived as `total cost / number of shares`. Total cost is entered
  in the currency of the cash account the transaction settles against
  (not necessarily USD), consistent with multi-currency support; it is
  converted to the base currency (USD) only for net worth/reporting
  totals. **Sell transactions additionally require a lot-picker step**
  (see Cost-basis method below) — the user selects which open lot(s)
  the sale draws from; this is a separate UI step, not a sixth core
  field.
- **Cost-basis method:** Specific lot. When selling, the user explicitly
  picks which open lot(s) the sale is drawn from (via a lot-picker step
  in the sell flow), rather than the app auto-selecting via FIFO/LIFO/
  average cost. Realized gain/loss for the sale is computed from the
  cost basis of the selected lot(s); the remaining share count/cost
  basis of those lots is reduced accordingly. **Lot-picker default:**
  the picker pre-selects open lots oldest-first (FIFO ordering) as a
  starting point, but the user can override and choose different lot(s)
  before confirming the sale — the pre-selection is a convenience
  default, not an auto-applied FIFO calculation.
- **Price data unavailable (offline or API failure):** Show the
  last-known price with a visible "stale" indicator and timestamp,
  rather than blocking price-dependent views (dashboard, net worth,
  allocation). Applies whenever the device is offline or the configured
  price provider (Google/Yahoo Finance) fails to respond.
- **Partial sell handling:**
  - **Lot splitting:** When a lot is partially sold, it is split into
    two records: a closed record for the sold portion (shares sold,
    realized cost basis, realized gain/loss) and a remaining-open
    record for the unsold portion (remaining shares, remaining cost
    basis). The original lot is not mutated in place — this preserves
    the immutable/audit-trailed ledger requirement.
  - **Multi-lot sells:** If a single sell transaction draws from
    multiple lots (via the lot-picker), it is stored as one sell
    transaction referencing multiple lot allocations (one allocation
    row per lot drawn from), not silently split into separate sell
    transactions.
  - **Cost-basis rounding (subtraction method):** Per-share cost basis
    is tracked internally at high precision and used only to compute
    the *realized* cost basis of the sold portion, rounded to the
    currency's minor unit (e.g. cents). The *remaining* lot's cost
    basis is then derived by subtracting the realized (rounded) amount
    from the lot's prior total cost basis — never by multiplying a
    rounded per-share figure back out. This guarantees the sold and
    remaining portions always sum exactly to the original total cost
    basis, with no cumulative rounding drift across repeated partial
    sells of the same lot.
  - **Cross-currency sells:** A lot may be sold into a cash account
    with a different currency than the one it was bought in. However,
    the sell transaction's total-proceeds amount must be entered in
    the *settling* cash account's currency (same rule as buys — no
    mixing currencies within a single transaction). Realized gain/loss
    is computed by converting both the original cost basis (in the
    buy's currency, at the buy date's rate) and the sale proceeds (in
    the sell's currency, at the sell date's rate) to USD for reporting.
  - **Splits + partial sells:** If a lot has already been adjusted by a
    stock split, that split adjustment is recorded as its own amendment
    entry on the lot before any partial-sell split is applied, so the
    audit trail shows the split adjustment and the sell-driven lot split
    as distinct, ordered events.
  - **Overselling validation:** Before confirming a sell, the app must
    verify that the total shares selected across all chosen lots equals
    (or does not exceed) the requested sell quantity, and that no
    individual lot allocation exceeds that lot's remaining open shares.
    If the sell quantity exceeds available holding quantity, the UI
    must block confirmation and show an explicit error stating the sell
    quantity exceeds the held quantity.

- **Minimum / target SDK:** minSdk = 34 (Android 14), targetSdk = 34
  (bump target as new Android versions release). Chosen because this is
  a personal/small-team app with no need to support older devices —
  avoids legacy compatibility shims and lets the app use the newest
  Jetpack Compose, BiometricPrompt, and Security APIs without fallback
  paths.
- **Schema / database migration strategy:** Use Room's built-in
  migration system — every schema change ships with an explicit
  `Migration` class (old version → new version); destructive migrations
  (`fallbackToDestructiveMigration`) are not used once the app has real
  user data, since that would silently wipe the local database. The
  full data export/import (device migration) file includes its own
  schema/format version number; on import, if the file's version is
  older than the current app's schema, the app runs the same import
  data through equivalent migration/transform logic before loading it,
  so restoring an old export into a newer app version doesn't fail or
  silently drop fields.
- **Duplicate ticker handling:** Adding a new stock transaction for a
  ticker that already exists as an asset within the same portfolio
  merges into the existing asset — it adds a new lot to that asset
  rather than creating a duplicate asset record. (The same ticker can
  still exist as a separate asset in a *different* portfolio, since
  assets belong to a portfolio.)
- **Other data validation rules:**
  - Share count and total cost on a buy/sell transaction must be
    strictly greater than zero; the form blocks submission with an
    inline error otherwise.
  - A transaction's total-cost/proceeds amount must be in the same
    currency as its settling cash account (enforced at entry — the
    currency field is derived from the selected account, not freely
    chosen).
  - Overselling is blocked per the partial-sell decision above (UI
    error if requested sell quantity exceeds remaining held shares
    across selected lots).
  - Ticker symbols are normalized to uppercase and checked for a valid
    format (alphanumeric, reasonable length) on entry; actual existence
    of the ticker is only confirmed later when a price fetch for it
    succeeds or fails (not blocked at entry time, since the app should
    still work offline/manually per the stale-price decision).
- **Screen list & navigation:** Single-activity app built with Jetpack
  Compose + Navigation Compose. Bottom navigation with top-level tabs —
  **Dashboard**, **Portfolios**, **Reports**, **Settings** — plus
  detail screens pushed on top of the nav stack:
  - **Dashboard** — grand total net worth (combined across all
    portfolios) shown together with its breakdowns on one screen: a
    per-portfolio breakdown (each portfolio's contribution to the
    total, not just a switcher to drill into one at a time) and an
    allocation chart (by asset type, account, currency, risk category).
    A portfolio switcher/list is still present for navigating to a
    single Portfolio Detail screen, but it's in addition to — not a
    replacement for — seeing all portfolios' contributions at a glance.
  - **Portfolio Detail** — holdings + cash accounts for one portfolio,
    snapshot trend chart.
  - **Asset Detail** — a ticker's open/closed lots, current price with
    stale indicator, transaction history for that asset.
  - **Add/Edit Transaction** — the 5-field buy/sell form; sell flow
    continues into the **Lot Picker** step before confirming.
  - **Cash Account Detail** — balance, deposit/withdrawal entry.
  - **Split Entry** — manual stock split entry (date + ratio).
  - **Reports** — gain/loss over a period, allocation breakdown,
    per-portfolio or combined.
  - **Settings** — per-portfolio price-provider choice; global base
    currency, FX source, polling interval, API key management.
  - **CSV Import/Export** and **Full Data Export/Import** — the latter
    gated behind BiometricPrompt per the export access control decision.
  - **Portfolio Management** — create/edit/delete a portfolio.

## Open Decisions

These are not yet finalized — check with the user before assuming an
answer, and update this file once decided:

- [ ] How are API keys (if any are needed) stored securely on-device
      (e.g. Android Keystore vs. encrypted shared prefs)?
- [ ] Snapshot frequency and trigger (e.g. daily background job via
      WorkManager, or only on app open)?
- [ ] How should fractional shares from a reverse split be handled (e.g.
      round down + track a manual cash-in-lieu entry, or allow fractional
      shares to persist)? Decide before implementing reverse-split logic.

## Working Conventions

(Fill these in as they're established during development.)

- Language/platform: Kotlin, native Android (min SDK: 34 / Android 14,
  target SDK: 34)
- Build/test/lint commands: _TBD_ (likely Gradle-based once project is
  scaffolded)
- Code style / naming conventions: _TBD_ (default to standard Kotlin/
  Android conventions unless specified otherwise)
- Directory structure: _TBD_

## Notes for GitHub Copilot

- This is a personal/small-team tool, not a commercial product — prefer
  simple, maintainable solutions over enterprise-grade complexity.
- Build for automated API-driven updates as the primary path (per
  Decisions above), with manual/CSV entry as a fallback — not the other
  way around.
- Multi-currency handling should be considered from the data-model stage
  (e.g. store amounts with an explicit currency, don't bolt it on later).
- Multi-portfolio support should be considered from the data-model stage
  too (e.g. assets and cash accounts belong to a portfolio; a user can
  have several). Don't assume a single implicit portfolio anywhere in
  the schema or UI.
- Device migration (full export/import) should serialize the entire
  local database (all portfolios, assets, snapshots, settings) into one
  file the user can save and later restore from on a new device. Keep
  this distinct from the CSV import/export feature. A single JSON or
  SQLite-file-based export is a reasonable default — pick whatever is
  easiest to keep in sync with schema changes over time.
- Stock split handling: apply split ratio to share count and cost basis
  at the time of the split so total position value is unchanged by the
  adjustment itself. Only adjust currently active/open holdings — leave
  already-closed (sold) lots as originally recorded, don't retroactively
  rewrite their history. Track split events explicitly (date, ratio)
  rather than silently overwriting share count, so the audit trail is
  visible and historical snapshots can be reconciled correctly.
  Fractional-share handling for reverse splits is an open decision
  (see above) — don't assume rounding behavior without checking.
- Transaction ledger data (buys, sells, splits, cash settlements) needs
  encryption at rest (e.g. SQLCipher for Room/SQLite, or Android's
  EncryptedFile/Jetpack Security library), an append-only or
  amendment-based edit model (no silent overwrites — preserve prior
  values), and soft-delete with confirmation prompts for any deletion.
  This applies to the transaction ledger specifically; other data (e.g.
  settings, price cache) doesn't need the same rigor unless stated.
- The full data export (device migration file) must be gated behind
  biometric/app-lock authentication (BiometricPrompt) before it's
  generated. Per the Decisions above, the exported file itself is
  plaintext (not re-encrypted) — the biometric gate is the intended
  protection, not the file's contents. Don't add file-level encryption
  to the export without checking with the user, since that was an
  explicit tradeoff.
- Stock price providers (Google/Yahoo Finance) are unofficial and can
  break. Build the price-fetching layer behind an interface/abstraction
  so a broken or changed endpoint doesn't require a rewrite, and so the
  user's per-portfolio provider choice is easy to honor.
- Prices should poll periodically (e.g. every 30–60s) while the app is in
  the foreground, not on every frame and not as background/streaming
  real-time. Make the interval easy to adjust in case a provider
  rate-limits or blocks frequent polling — this is an unofficial API risk,
  not a guarantee.
- Cash accounts: only stock buy/sell settlements should adjust balances
  automatically. Deposits/withdrawals are user-entered; do not attempt
  bank account API integration (conflicts with local-only storage
  decision).
- Stock buy/sell entry form should collect exactly five fields: date,
  type (buy/sell), ticker, shares, total cost (fees included). Derive
  per-share cost basis by dividing total cost by shares — don't add a
  separate fee or per-share-price input. The total cost amount is in the
  settling cash account's currency; use that account's currency for the
  ledger entry and the buy/sell cash settlement, converting to USD only
  for reporting.
- Cost basis is tracked per lot (specific-lot method) — don't collapse
  positions into a single average cost. A sell transaction must let the
  user pick which open lot(s) to sell from (a lot-picker UI step, not
  one of the five core fields); realized gain/loss and the lot's
  remaining shares/cost basis derive from that selection. Don't default
  to FIFO/LIFO auto-selection — the lot-picker should pre-select open
  lots oldest-first as a convenience default, but the user must be able
  to change the selection before confirming the sale.
- When price data can't be fetched (offline, or the Google/Yahoo Finance
  endpoint fails), show the last successfully fetched price with a
  visible "stale" indicator and last-updated timestamp rather than
  blocking the dashboard or other price-dependent screens.
- Do not add multi-user auth/permissions unless explicitly requested —
  it's out of scope for v1.
- Do not introduce a backend/server component unless the "Storage"
  decision above is revisited — v1 is local-only, on-device.
- When a decision under "Open Decisions" becomes settled, move it up into
  "Decisions (settled)" so future sessions have the current answer.
- Partial sells split a lot into a closed (sold) record and a remaining
  (open) record rather than mutating the original lot in place. Use the
  subtraction method for cost-basis rounding: round the realized (sold)
  cost basis to the currency's minor unit, then derive the remaining
  lot's cost basis by subtracting that rounded amount from the prior
  total — never by rounding a per-share figure and multiplying back out.
  This avoids cumulative rounding drift across repeated partial sells.
  Multi-lot sells are one sell transaction with multiple lot-allocation
  rows, not separate transactions per lot. Cross-currency sells (lot
  bought in one currency, sold into a different cash account/currency)
  are allowed, but each transaction's amount must still match its own
  settling account's currency — convert to USD only for gain/loss
  reporting. Always validate sell quantity against remaining holdings
  before confirming a sale; show a UI error (not a silent clamp) if the
  requested sell quantity exceeds what's held.
- Target minSdk/targetSdk 34 (Android 14) — no legacy compatibility
  shims needed; use the latest Compose, BiometricPrompt, and Security
  library APIs directly.
- Every Room schema change needs an explicit `Migration` class; never
  ship `fallbackToDestructiveMigration` once real data exists. Include a
  schema/format version field in the device-migration export file and
  run imported data through the equivalent migration path if it's older
  than the current schema.
  Adding a transaction for a ticker already held in the same portfolio
  merges into that asset as a new lot — don't create duplicate asset
  records per ticker per portfolio (a different portfolio can still
  hold the same ticker as its own separate asset).
- Enforce on entry: shares and total cost must be > 0; a transaction's
  currency must match its settling cash account's currency; overselling
  is blocked with an explicit UI error (see partial-sell decisions).
  Ticker format is validated on entry, but real existence is only
  confirmed via a later price-fetch attempt, not blocked at entry.
- Build as a single-activity Compose app with Navigation Compose, bottom
  nav tabs for Dashboard / Portfolios / Reports / Settings, and the
  screen set listed under "Screen list & navigation" above. Follow that
  screen list rather than inventing a different structure.
- The Dashboard is the one screen where the grand total net worth and
  its breakdowns all appear together: don't design it as "total number,
  then tap into a portfolio to see its share." Show the per-portfolio
  contributions and the type/account/currency/risk allocation chart
  right there alongside the grand total. A switcher for navigating to a
  single Portfolio Detail screen is fine in addition to this, but the
  combined breakdown view is the primary Dashboard content.
