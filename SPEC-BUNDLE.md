# Spec Package: TradeSync Pro
### Trading Journal + Psychology Dashboard for Retail Traders

**Spec Version:** 1.0
**Date:** 2026-04-05
**Author:** Neo Builder Agent
**Status:** PRE-GATE (not yet approved for implementation)

---

## 0) Interpretation Snapshot

TradeSync Pro is a trading journal SaaS that differentiates from competitors by treating emotional state as a first-class data field alongside price and size. Unlike TraderVue or EdgeCleat (which focus on trade analytics), TradeSync surfaces the **tilt-to-loss correlation** — helping traders see when their psychology is costing them money. MVP is a single HTML file with localStorage; production version adds cloud sync, multi-device, and subscription billing. The core loop: log a trade → get psychology feedback → build self-awareness → trade better.

**In scope:** Trade logging, P&L calculation, tilt tracking, psychology flagging, weekly stats, CSV export, dark theme, mobile-responsive.
**Out of scope:** Cloud sync, multi-device, broker integrations, TradingView webhook import, subscription billing, social features, backtesting.

**Target user:** Retail trader (equities, crypto, forex) who journals manually or not at all.

---

## 1) PRD.md — Product Requirements

```
REQ-001: Trade Entry
  Statement: "System MUST allow user to log a trade with: ticker symbol, 
             direction (long/short), entry price, exit price, position size, 
             trade date, and emotional tilt (integer 1-5)."
  Rationale: Core data capture — incomplete entry = useless journal
  Priority: P0
  Acceptance: Form accepts all 7 fields; all fields validated; trade saved 
             to localStorage within 500ms of submit

REQ-002: P&L Calculation
  Statement: "System MUST calculate P&L as: (exit_price - entry_price) × size 
             × direction_multiplier, where direction_multiplier = +1 for long, 
             -1 for short."
  Rationale: Incorrect P&L = broken journal — must match industry standard
  Priority: P0
  Acceptance: (105-100)×10×1 = +$50 long; (100-105)×10×-1 = +$50 short; 
              result stored with trade record and displayed to 2 decimal places

REQ-003: Dashboard Statistics
  Statement: "System MUST display: total trade count, win rate (%, wins/total), 
             total P&L (sum), average win (mean of positive P&L), average loss 
             (mean of negative P&L), largest single win, largest single loss."
  Rationale: These 7 stats are the minimum viable dashboard for a trading journal
  Priority: P0
  Acceptance: Stats update within 500ms of new trade; empty state shows "--" 
              for avg/percentage stats; 0 trades shows 0% win rate

REQ-004: Psychology Flag
  Statement: "System MUST flag any trade where tilt ≥ 4 AND P&L < 0 as a 
             'TILT TRADE' with red visual treatment."
  Rationale: Core differentiator — surfacing tilt-driven losses is the product's 
             primary value
  Priority: P0
  Acceptance: Flag appears within 1 second of trade save; red border + 
              TILT badge visible on trade card; flagged trades appear in 
              dedicated filter view

REQ-005: Tilt Scale
  Statement: "System MUST present tilt as a 1-5 ordinal scale with labels:
             1=Calm, 2=Focused, 3=Neutral, 4=Tense, 5=Tilting/Revenge."
  Rationale: Standardized emotional vocabulary helps traders build awareness
  Priority: P1
  Acceptance: All 5 buttons visible; selected button shows filled state with 
             correct color; 1-3 use warm colors, 4-5 use red spectrum

REQ-006: Trade Journal List
  Statement: "System MUST display all logged trades in reverse-chronological 
             order, showing: direction badge, ticker, date, entry→exit range, 
             size, P&L, tilt level."
  Rationale: Journal view is the primary interface after entry
  Priority: P0
  Acceptance: Newest trade at top; trades persist after refresh; scrollable 
              list with 20+ trades; empty state message when no trades exist

REQ-007: Weekly P&L Chart
  Statement: "System MUST display a bar chart showing P&L per calendar day for 
             the 7 days ending today."
  Rationale: Traders think in sessions/days; weekly view prevents recency bias
  Priority: P1
  Acceptance: One bar per day (Mon-Sun or last-7-calendar-days); green bars 
              for profit days, red for loss days; bars sized proportionally; 
              day labels below; P&L value above each bar

REQ-008: CSV Export
  Statement: "System MUST export all trades as a CSV file with headers:
             Date, Ticker, Direction, Entry, Exit, Size, P&L, Tilt."
  Rationale: Portability is essential for serious traders who use multiple tools
  Priority: P1
  Acceptance: CSV downloads within 1 second of export button click; file 
             named 'tradesync-export-YYYY-MM-DD.csv'; valid CSV opens in Excel/Sheets

REQ-009: Data Persistence
  Statement: "System MUST persist all trades in localStorage under key 
             'tradesync_trades' as a JSON array."
  Rationale: localStorage is the MVP persistence layer; survives browser refresh
  Priority: P0
  Acceptance: Trades survive page close/reopen; max localStorage ~5MB; 
              graceful handling if localStorage is full (error toast shown)

REQ-010: Mobile Responsiveness
  Statement: "System MUST be fully functional on screens ≥320px wide, 
             with single-column layout below 600px."
  Rationale: Traders use phones to log trades immediately after exits
  Priority: P1
  Acceptance: Form fields stack vertically on mobile; stat cards wrap to 
              2-column grid; trade list items reflow; no horizontal scroll

REQ-011: Dark Theme
  Statement: "System MUST use dark theme exclusively, with background #0a0e17 
             and accent colors as defined in UI spec."
  Rationale: Traders prefer dark UIs for extended screen time; reduces eye strain
  Priority: P1
  Acceptance: All text readable (contrast ratio ≥4.5:1); no white backgrounds 
              anywhere; chart colors visible on dark background

REQ-012: Delete Trade
  Statement: "System MUST allow user to delete any single trade with 
             confirmation before removal."
  Rationale: Mistakes happen; users must be able to correct entries
  Priority: P1
  Acceptance: Delete button visible on each trade card; confirmation dialog 
              shown before removal; trade removed immediately after confirm; 
              stats update after deletion
```

---

## 2) USER_FLOWS.md

```
FLOW-001: Log a Trade
  Entry: User clicks "Log Trade" tab or form is visible on page load
  Steps:
    1. User types ticker symbol → auto-uppercased, max 10 chars
    2. User selects direction (Long selected by default)
    3. User enters entry price (number, 2 decimal precision)
    4. User enters exit price (number, 2 decimal precision)
    5. User enters position size (number, no limit)
    6. User selects date (date picker, defaults to today)
    7. User selects tilt level (1-5 button group, required)
    8. System calculates and displays P&L preview in real-time
    9. User clicks "Add Trade" button
  System Responses:
    - Valid: Save to localStorage, show success toast "Trade logged ✅", 
             clear form, update dashboard stats, add trade to top of journal
    - Invalid ticker: Show inline error "Ticker required"
    - Missing price/size: Show inline error on empty fields
    - Missing tilt: Show inline error "Select tilt level"
    - localStorage full: Show error toast "Storage full — export and clear old trades"
  UI States:
    - Loading: N/A (form is synchronous)
    - Empty: Form ready, journal shows empty state message
    - Error: Inline field errors, form values preserved
    - Success: Toast + cleared form + updated stats/journal
  Instrumentation: trade_logged event with {ticker, direction, pnl, tilt}

FLOW-002: View Dashboard
  Entry: Page load or tab switch to Dashboard
  Steps:
    1. System reads all trades from localStorage
    2. System calculates all 7 statistics
    3. System renders stat cards with values
    4. System renders weekly chart (last 7 days)
  System Responses:
    - With trades: Display populated stats and chart
    - Empty: Show "--" for calculated stats, empty chart with message
  UI States:
    - Loading: Skeleton cards with subtle pulse (200ms max — localStorage is fast)
    - Populated: All 7 stat cards + weekly chart
    - Error: Show "Unable to load stats" with retry button

FLOW-003: Review Journal
  Entry: Journal tab click or scroll to journal section
  Steps:
    1. System reads all trades from localStorage
    2. System sorts by date descending
    3. System renders trade cards with tilt flags
    4. User scrolls through list
  System Responses:
    - Normal trades: Standard card styling
    - Tilt trades (tilt≥4 AND pnl<0): Red border, TILT badge
    - All trades show delete button (hidden until hover on desktop, 
      always visible on mobile)
  UI States:
    - Loading: 3 skeleton cards
    - Empty: "No trades yet — log your first trade above"
    - Populated: Scrollable list, newest first
    - Filter active: "Showing tilt trades only (X of Y)"
  Instrumentation: journal_viewed event with {count, tilt_trade_count}

FLOW-004: Delete a Trade
  Entry: User clicks delete button on a trade card
  Steps:
    1. System shows confirmation modal: "Delete [TICKER] trade? 
       This cannot be undone."
    2. User clicks Confirm or Cancel
  System Responses:
    - Confirm: Remove from localStorage, update stats, remove from journal, 
               show toast "Trade deleted"
    - Cancel: Close modal, no changes
  UI States:
    - Modal: Centered overlay with blur backdrop, Confirm (red) + Cancel buttons
  Instrumentation: trade_deleted event with {ticker, pnl, tilt}

FLOW-005: Export CSV
  Entry: User clicks "Export CSV" button
  Steps:
    1. System reads all trades from localStorage
    2. System generates CSV string with headers
    3. System triggers browser download
  System Responses:
    - Success: File downloads, show toast "Exported X trades"
    - Empty: Show toast "No trades to export"
    - Error: Show toast "Export failed — try again"
  UI States:
    - Button: Shows "Exporting..." during generation (should be <1s)
  Instrumentation: csv_exported event with {trade_count}

FLOW-006: Filter Tilt Trades
  Entry: User clicks "Tilt Trades" filter tab
  Steps:
    1. System reads all trades
    2. System filters to tilt trades (tilt≥4 AND pnl<0)
    3. System renders filtered list with count badge
  System Responses:
    - Matches found: Show filtered list with count "X tilt trades"
    - No matches: Show "No tilt trades yet 🎉"
  UI States:
    - Filter active: Tab highlighted, clear filter button visible
```

---

## 3) UI_SPEC.md

```
SCREEN-001: Dashboard View (Default/Home)
  Description: Main screen showing stats cards + weekly chart
  Components:
    - Header with app name + tagline
    - 6-stat grid (Win Rate, Total P&L, Avg Win, Avg Loss, Largest Win, Largest Loss)
    - Weekly P&L bar chart
    - Quick-add form (collapsible on mobile)
    - Navigation: Dashboard | Journal
  States:
    - Empty: Stat values show "--", chart shows "No data yet"
    - Populated: All values calculated
    - Loading: Skeleton pulse on cards

SCREEN-002: Journal View
  Description: Full trade log with filtering
  Components:
    - Filter tabs: All | Tilt Trades
    - Trade card list (scrollable)
    - Empty state per filter
  States:
    - Empty all: "No trades yet — log your first trade"
    - Empty tilt: "No tilt trades yet 🎉"
    - Populated: Trade cards with full details

SCREEN-003: Trade Entry Form
  Description: Inline form for logging new trades
  Components:
    - Ticker input (text, uppercase)
    - Direction dropdown (Long/Short)
    - Entry price input (number)
    - Exit price input (number)
    - Position size input (number)
    - Date picker (defaults today)
    - Tilt selector (5-button row)
    - P&L preview (calculated live)
    - Submit button
  States:
    - Default: All fields empty/defaulted
    - Filled: Values present, P&L preview shown
    - Error: Inline errors below invalid fields
    - Submitting: Button disabled, shows "Adding..."
    - Success: Toast, form cleared

COMP-001: Stat Card
  Description: Single metric display card
  Props: label (string), value (number|string), trend (up|down|neutral)
  States: positive (green value), negative (red value), neutral (white), empty (--)

COMP-002: Trade Card
  Description: Individual trade display
  Props: trade object (ticker, direction, entry, exit, size, pnl, tilt, date)
  States:
    - Normal: Standard styling
    - Tilt Flagged: Red border (#ff4757), TILT badge, warning icon
    - Hover: Show delete button
    - Mobile: Delete always visible

COMP-003: Tilt Button Group
  Description: 5-button tilt selector
  Props: selected (1-5)
  States per button:
    - Unselected: Muted background, white text
    - Selected 1-3: Warm colors (yellow→orange)
    - Selected 4-5: Red spectrum
  Labels: "1=Calm", "2=Focused", "3=Neutral", "4=Tense", "5=Tilting"

COMP-004: Weekly Chart
  Description: 7-bar P&L bar chart
  Props: data (array of {day, pnl})
  States:
    - Empty: "No data for last 7 days"
    - Populated: Proportional green/red bars
    - All profit: All green
    - All loss: All red

COMP-005: Toast Notification
  Description: Temporary feedback message
  Props: message (string), type (success|error|info)
  Animation: Slide up from bottom-right, auto-dismiss 3s
  Types:
    - Success: Green accent (#00d9a5)
    - Error: Red accent (#ff4757)
    - Info: Blue accent (#4f7cff)

COPY STRINGS (user-facing):
- App name: "TradeSync"
- Tagline: "Trade smarter. Journal better."
- Tilt labels: "Calm | Focused | Neutral | Tense | Tilting"
- Filter tabs: "All Trades" | "Tilt Trades"
- Empty journal: "No trades yet — log your first trade above"
- Empty tilt: "No tilt trades yet 🎉"
- Export button: "Export CSV"
- Delete confirm: "Delete {TICKER} trade? This cannot be undone."
- Toast trade logged: "Trade logged ✅"
- Toast deleted: "Trade deleted"
- Toast exported: "Exported {n} trades"
```

---

## 4) DOMAIN_MODEL.md

```
ENTITY: Trade
  Attributes:
    - id: string (UUID v4, auto-generated)
    - ticker: string (1-10 uppercase chars)
    - direction: enum ("long" | "short")
    - entryPrice: number (USD, ≥0)
    - exitPrice: number (USD, ≥0)
    - size: number (≥0, unit depends on asset class)
    - date: string (ISO 8601 date)
    - tilt: integer (1-5)
    - pnl: number (calculated, USD)
    - createdAt: string (ISO 8601 datetime)
  Derived:
    - pnl = (exitPrice - entryPrice) × size × direction_multiplier
      where direction_multiplier = 1 for long, -1 for short
    - isTiltTrade = tilt ≥ 4 AND pnl < 0
    - isWin = pnl > 0
    - isLoss = pnl < 0

ENTITY: DashboardStats (derived, not persisted)
  Attributes:
    - tradeCount: integer
    - winCount: integer
    - lossCount: integer
    - winRate: number (winCount / tradeCount × 100, or null if no trades)
    - totalPnl: number
    - avgWin: number (or null if no wins)
    - avgLoss: number (or null if no losses)
    - largestWin: number (or null if no wins)
    - largestLoss: number (or null if no losses)

ENTITY: WeeklyPnl (derived, not persisted)
  Attributes:
    - days: array of {date: string, pnl: number} (last 7 calendar days)

INV-001: "ticker must be non-empty string, max 10 chars, uppercase only"
INV-002: "entryPrice and exitPrice must be ≥ 0"
INV-003: "size must be > 0"
INV-004: "tilt must be integer 1-5 inclusive"
INV-005: "date must be valid ISO 8601 date, not in future"
INV-006: "pnl calculation must be deterministic and match formula exactly"
INV-007: "isTiltTrade must be (tilt >= 4 AND pnl < 0)"
INV-008: "Trade ID must be globally unique (UUID)"
INV-009: "All trades in localStorage array must be valid per INV-001 through INV-008"

RELATIONSHIPS:
- DashboardStats derived from: zero or more Trade entities
- WeeklyPnl derived from: zero or more Trade entities
- No foreign key relationships (single-entity localStorage MVP)
```

---

## 5) DATA_MODEL.md

```
STORAGE: localStorage
  Key: "tradesync_trades"
  Format: JSON array of Trade objects

SCHEMA:
{
  "trades": [
    {
      "id": "string (UUID v4)",
      "ticker": "string",
      "direction": "long" | "short",
      "entryPrice": "number",
      "exitPrice": "number",
      "size": "number",
      "date": "string (ISO 8601 date)",
      "tilt": "number (1-5)",
      "pnl": "number",
      "createdAt": "string (ISO 8601 datetime)"
    }
  ]
}

VALIDATION RULES (enforced at write time):
- trades must be an array
- each trade must pass INV-001 through INV-009
- invalid trades are rejected (not silently dropped)

MIGRATION PLAN:
- v1.0: Single array at "tradesync_trades"
- Future v2.0: Add "tradesync_settings" key for user preferences
- Migration: Merge new keys without destroying existing trades

DATA RETENTION:
- Trades retained until user deletes them manually
- No TTL in MVP (localStorage limit is natural ceiling ~5MB / ~1000 trades)

BACKUP/EXPORT:
- CSV export provides portable backup
- No automatic cloud backup in MVP

ERROR HANDLING:
- localStorage unavailable: Show error toast, disable all save/delete operations
- localStorage quota exceeded: Show error toast "Storage full", prevent new saves
- Corrupt JSON: Show error, offer to reset data (with confirmation)
```

---

## 6) STATE_MACHINE.md

```
STATE: AppState
  Values: "loading" | "ready" | "error"

STATE: FormState
  Values: "idle" | "validating" | "submitting" | "success" | "error"
  Transitions:
    idle → validating: (on input change, real-time validation)
    validating → error: (validation errors found)
    validating → idle: (errors cleared)
    validating → submitting: (user clicks submit, no errors)
    submitting → success: (localStorage write complete)
    submitting → error: (localStorage write failed)
    success → idle: (after 1.5s delay)
    error → idle: (user modifies any field)

STATE: TradeCard
  Values: "normal" | "tilt-flagged" | "deleting"
  Transitions:
    normal → tilt-flagged: (trade saved with tilt≥4 AND pnl<0)
    tilt-flagged → normal: (trade edited to tilt<4 OR pnl≥0 — not MVP scope)
    normal/tilt-flagged → deleting: (delete button clicked)
    deleting → (removed from DOM): (user confirms delete)

STATE: FilterState
  Values: "all" | "tilt-only"
  Transitions:
    all → tilt-only: (user clicks Tilt Trades tab)
    tilt-only → all: (user clicks All Trades tab)
    any → all: (journal re-renders after trade delete if tilt-only and count=0)

STATE: ToastState
  Values: "hidden" | "visible"
  Transitions:
    hidden → visible: (any toast triggered)
    visible → hidden: (3 second auto-dismiss OR new toast replaces current)

INVALID TRANSITIONS:
- FormState.submitting → idle without save: Shows error toast instead
- Delete on already-deleted trade: No-op (element already removed)
- Export with empty array: Shows info toast "No trades to export"
```

---

## 7) NON_FUNCTIONAL.md

```
PERFORMANCE:
- First contentful paint: < 1 second on 3G
- localStorage read (all trades): < 50ms for 1000 trades
- localStorage write (one trade): < 50ms
- Form validation: < 16ms (within single frame)
- Stats calculation (1000 trades): < 100ms
- CSV generation (1000 trades): < 500ms

RELIABILITY:
- localStorage read failure: Show error state with retry
- localStorage write failure: Show error toast, preserve form data
- Corrupt data: Offer reset with explicit confirmation
- No network dependency: Works fully offline after initial load

AVAILABILITY:
- 100% availability offline (no server required for MVP)
- Google Fonts loaded with fallback to system sans-serif

SECURITY:
- No authentication (single-device MVP)
- No PII beyond ticker/price data (not sensitive)
- No data transmission (all local)
- XSS risk: All user inputs treated as text (no HTML rendering)
- CSP: No inline scripts in production build

STORAGE LIMITS:
- localStorage: ~5-10MB typical
- Max trades estimate: ~5,000 trades before hitting limit
- Trade record size estimate: ~200 bytes JSON
- Export: No limit (generates in memory, streams to download)

BROWSER SUPPORT:
- Modern browsers (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- Mobile: iOS Safari 14+, Chrome Android 90+
```

---

## 8) OBSERVABILITY.md

```
LOGGING (console.log for MVP — production would use analytics):
Events logged:
  - trade_logged: {ticker, direction, pnl, tilt, trade_count}
  - trade_deleted: {ticker, pnl, tilt, remaining_count}
  - csv_exported: {trade_count}
  - app_loaded: {trade_count}
  - filter_changed: {filter_type, result_count}

METRICS (in-memory counters for MVP):
- total_trades_logged: counter
- total_trades_deleted: counter
- tilt_trades_flagged: gauge (current tilt trade count)
- export_count: counter
- storage_usage_bytes: gauge (estimated from JSON string length)

ERROR REPORTING:
- localStorage errors: Caught and surfaced as user-facing toast
- Validation errors: Inline field-level messages (not console)
- Uncaught exceptions: window.onerror handler → error toast + console

DASHBOARDS (not built in MVP — spec for production):
- Daily trade count line chart
- P&L running total
- Tilt trade ratio over time
- Most traded tickers
```

---

## 9) TEST_PLAN.md

```
TEST-001: Log valid long trade
  Covers: REQ-001, REQ-002, REQ-009
  Setup: Fresh localStorage, page loaded
  Steps:
    1. Enter ticker "AAPL"
    2. Select "Long"
    3. Enter entry price 150.00
    4. Enter exit price 155.00
    5. Enter size 100
    6. Select tilt 2
    7. Click "Add Trade"
  Expected: Toast "Trade logged ✅", trade appears in journal with 
            P&L +$500.00, stats update (win rate 100%, total +$500)

TEST-002: Log valid short trade
  Covers: REQ-001, REQ-002, REQ-009
  Setup: Empty journal
  Steps: Same as TEST-001 but direction=Short, entry=155, exit=150, size=100
  Expected: P&L +$500.00 (short profit when price drops)

TEST-003: P&L calculation accuracy
  Covers: REQ-002
  Setup: Empty journal
  Test cases:
    - Long: entry=100, exit=110, size=50 → +$500
    - Short: entry=100, exit=90, size=50 → +$500
    - Long: entry=100, exit=90, size=50 → -$500
    - Short: entry=100, exit=110, size=50 → -$500
    - Long: entry=50.25, exit=51.75, size=200 → +$300.00
  Expected: All exact to 2 decimal places

TEST-004: Psychology flag — tilt trade detected
  Covers: REQ-004
  Setup: Empty journal
  Steps: Log trade with tilt=4, pnl=-$200 (long: entry=100, exit=98, size=100)
  Expected: Trade card has red border, TILT badge visible, included in tilt filter

TEST-005: Psychology flag — not flagged when profitable
  Covers: REQ-004
  Setup: Empty journal
  Steps: Log trade with tilt=5, pnl=+$100
  Expected: No red border, no TILT badge, NOT included in tilt filter

TEST-006: Psychology flag — not flagged at tilt=3
  Covers: REQ-004
  Setup: Empty journal
  Steps: Log trade with tilt=3, pnl=-$200
  Expected: Normal card styling, NOT in tilt filter

TEST-007: Tilt scale — all 5 levels selectable
  Covers: REQ-005
  Setup: Form ready
  Steps: Click each tilt button 1-5, verify selection state and color
  Expected: Each button independently selectable; colors change per level

TEST-008: Dashboard stats — empty state
  Covers: REQ-003
  Setup: localStorage empty
  Steps: Load page, check dashboard
  Expected: Win Rate "--", Total P&L "$0.00", Avg Win "--", 
            Avg Loss "--", Largest Win "--", Largest Loss "--"

TEST-009: Dashboard stats — mixed trades
  Covers: REQ-003
  Setup: 4 trades (3 wins: +$100, +$200, +$150; 1 loss: -$80)
  Steps: Load page, observe stats
  Expected: Trade count=4, Win rate=75%, Total P&L=+$370, 
            Avg Win=+$150, Avg Loss=-$80, Largest Win=+$200, Largest Loss=-$80

TEST-010: Weekly chart — no data
  Covers: REQ-007
  Setup: Empty localStorage
  Steps: Load page, check chart area
  Expected: Empty chart area with "No data" message

TEST-011: Weekly chart — 7 days with mixed results
  Covers: REQ-007
  Setup: Add trades for 3 profitable days and 2 losing days in last 7
  Steps: Load page, check chart
  Expected: 5 bars visible (days with no trades may be omitted or show $0 bar)

TEST-012: CSV export — all fields present
  Covers: REQ-008
  Setup: Add 3 trades
  Steps: Click Export CSV, open downloaded file
  Expected: Headers: Date, Ticker, Direction, Entry, Exit, Size, P&L, Tilt
           3 data rows with correct values

TEST-013: CSV export — empty journal
  Covers: REQ-008
  Setup: Empty localStorage
  Steps: Click Export CSV
  Expected: Info toast "No trades to export"

TEST-014: Delete trade — confirmation required
  Covers: REQ-012
  Setup: Add 1 trade
  Steps: Click delete on trade, check modal
  Expected: Modal appears with "Delete TICKER trade? This cannot be undone."
           Confirm and Cancel buttons

TEST-015: Delete trade — confirm removes from storage
  Covers: REQ-012, REQ-009
  Setup: Add 2 trades, delete one
  Steps: Confirm delete
  Expected: Deleted trade gone from journal and localStorage; 
            other trade still present; stats update

TEST-016: Delete trade — cancel preserves
  Covers: REQ-012
  Setup: Add 1 trade
  Steps: Click delete, then Cancel
  Expected: Trade still in journal and localStorage; modal closed

TEST-017: Data persistence after refresh
  Covers: REQ-009
  Setup: Add 3 trades, verify stats show them
  Steps: Close browser tab, reopen, load page
  Expected: All 3 trades present, stats same as before

TEST-018: Form validation — missing ticker
  Covers: REQ-001
  Setup: Form ready
  Steps: Leave ticker empty, fill all other fields, select tilt, submit
  Expected: Inline error "Ticker required" below ticker field

TEST-019: Form validation — missing tilt
  Covers: REQ-001
  Setup: Form ready
  Steps: Fill all required fields, leave tilt unselected, submit
  Expected: Inline error "Select tilt level"

TEST-020: Form uppercase ticker
  Covers: REQ-001
  Setup: Empty ticker field
  Steps: Type "aapl" (lowercase)
  Expected: Field shows "AAPL" (uppercase)

TEST-021: Mobile layout — narrow screen
  Covers: REQ-010
  Setup: Browser viewport 375px wide
  Steps: Load page, observe layout
  Expected: Single column layout; stat cards 2-column grid; 
            form fields stack; no horizontal scroll

TEST-022: Tilt filter shows only tilt trades
  Covers: REQ-004, FLOW-006
  Setup: 3 normal trades, 2 tilt trades
  Steps: Click "Tilt Trades" filter
  Expected: Only 2 tilt trades shown; count badge shows "2"
```

---

## 10) ROLL_OUT.md

```
FEATURE FLAGS: N/A (MVP single file, no staged rollout)

RELEASE PLAN:
Phase 1 (MVP Launch):
  - Single HTML file deployed to Netlify
  - GitHub Pages alternative deployment
  - Share link with target users (trading communities)

Phase 2 (Post-MVP if traction):
  - Add cloud sync (Supabase auth + Postgres)
  - Multi-device support
  - Subscription billing ($9/mo)
  - TradingView webhook import

MIGRATION: N/A (v1.0, no previous version)

ROLLBACK:
  - Previous version: Replace index.html with prior version
  - localStorage is forward-compatible: new fields ignored by old code
  - No database migrations in MVP

RELEASE DAY MONITORING:
  - Check Netlify deploy succeeded
  - Manual test of all 22 test cases
  - Verify Google Fonts loading
  - Check mobile layout on real device (if available)

RISKS:
  - Google Fonts CDN failure → fallback to system fonts (Outfit → system sans)
  - localStorage full → show warning, prompt CSV export
  - Browser compatibility → tested on Chrome/Firefox/Safari latest

RUNBOOK:
  Symptom: Stats show wrong values
    Action: Clear localStorage, re-add trades from backup CSV
  
  Symptom: Page blank
    Action: Check browser console; likely font/CSS load failure → hard refresh
  
  Symptom: CSV export downloads empty file
    Action: Refresh page, try again; check localStorage has trades
  
  Symptom: Can't add trades
    Action: Check localStorage not full; export CSV to backup; 
            clear old trades if needed
```

---

## 11) OPEN_DECISIONS.md

```
ASSUMPTION-001: Currency assumption
  What: We assume all prices/P&L are in USD
  Why: Simplest MVP; no multi-currency support needed initially
  Risk: Non-USD traders (forex, international stocks) find it less useful
  Default: USD only; no currency selector in MVP

ASSUMPTION-002: Asset class assumption
  What: We assume equity/crypto-style linear P&L (not options, futures)
  Why: Simplest P&L formula covers most retail use cases
  Risk: Options traders need more complex P&L (multi-leg, IV)
  Default: Single-leg linear P&L only

ASSUMPTION-003: Single user assumption
  What: No authentication; all data on one device
  Why: localStorage is single-origin; auth adds backend complexity
  Risk: Users on multiple devices need to re-enter data
  Default: Single device; cloud sync is Phase 2

ASSUMPTION-004: No TradingView integration
  What: Manual trade entry only; no broker or TradingView import
  Why: Broker integrations require OAuth, compliance review; too complex for MVP
  Risk: Power users who want auto-import may not adopt
  Default: Manual only; webhook import is Phase 2

ASSUMPTION-005: Tilt scale labels are approximate
  What: Tilt labels (Calm/Focused/Neutral/Tense/Tilting) are approximate descriptors
  Why: Emotional states are subjective; no single right answer
  Risk: Users disagree with label placement (e.g., "Focused" = tilt 4 for some)
  Default: Fixed labels; future could allow custom labels

DECISION-001: Weekly chart — calendar days or trading days?
  Options: Calendar days (Mon-Sun) vs last-7-trading-days
  Recommendation: Last 7 calendar days (simpler, includes weekends for crypto traders)
  Impact: Changes bar chart data significantly for stock traders
  Blocking: No — default to calendar days, easily changed

DECISION-002: CSV date format?
  Options: ISO 8601 (YYYY-MM-DD) vs US (MM/DD/YYYY) vs Excel-friendly (MM-DD-YYYY)
  Recommendation: ISO 8601 (YYYY-MM-DD) — universal, sorts correctly in Excel
  Impact: Minor UX preference
  Blocking: No

BLOCKING ITEMS: None — all assumptions resolved with defaults
```

---

## 12) Spec Quality Checklist

```
[✓] No ambiguous words without metrics
[✓] All requirements (REQ-001 through REQ-012) have measurable acceptance criteria
[✓] All requirements are atomic (no multi-part "and