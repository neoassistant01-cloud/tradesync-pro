# TradeSync Pro — Implementation Tickets
### Generated from SPEC-BUNDLE.md

---

## TICKET-001: Build TradeSync MVP — Core Application
**Priority:** P0
**Estimated Time:** 45-60 minutes
**Assignee:** Frontend Builder Agent

### Context
Full spec at `/opt/openclaw/mvp-projects/trading-journal-sync/SPEC-BUNDLE.md`. Read it fully before writing any code.

### Deliverable
A single production-ready `dist/index.html` file at:
`/opt/openclaw/mvp-projects/trading-journal-sync/dist/index.html`

### Requirements Checklist

**Form & Data Capture (REQ-001, REQ-002, REQ-005):**
- [ ] Ticker input: text, auto-uppercase, max 10 chars
- [ ] Direction dropdown: Long (default) / Short
- [ ] Entry price: number, 2 decimal precision
- [ ] Exit price: number, 2 decimal precision
- [ ] Position size: number, no limit
- [ ] Date picker: defaults to today, no future dates
- [ ] Tilt selector: 5-button group, labels = Calm/Focused/Neutral/Tense/Tilting
- [ ] Colors: 1-3 warm (yellow→orange), 4-5 red spectrum
- [ ] Live P&L preview as user types

**Dashboard Stats (REQ-003):**
- [ ] Win Rate % (wins/total, show "--" if no trades)
- [ ] Total P&L (green if positive, red if negative)
- [ ] Average Win (mean of positive P&L, "--" if no wins)
- [ ] Average Loss (mean of negative P&L, "--" if no losses)
- [ ] Largest Win (max positive P&L)
- [ ] Largest Loss (max negative, most negative)
- [ ] All stats update within 500ms of new trade

**Psychology Flag (REQ-004):**
- [ ] Auto-flag: tilt ≥ 4 AND P&L < 0
- [ ] Visual: red border (#ff4757) + "TILT" badge on trade card
- [ ] Filter tab "Tilt Trades" shows only flagged trades

**Journal (REQ-006, REQ-012, FLOW-003, FLOW-004):**
- [ ] Reverse chronological order (newest first)
- [ ] Each card shows: direction badge, ticker, date, entry→exit, size, P&L, tilt
- [ ] Hover reveals delete button (desktop); always visible (mobile)
- [ ] Delete shows confirmation modal before removal
- [ ] Toast "Trade deleted" after confirm

**Weekly Chart (REQ-007):**
- [ ] 7 bars, last 7 calendar days
- [ ] Green bars = profit days, red = loss days
- [ ] Day label below each bar, P&L value above

**CSV Export (REQ-008, FLOW-005):**
- [ ] Headers: Date, Ticker, Direction, Entry, Exit, Size, P&L, Tilt
- [ ] Filename: tradesync-export-YYYY-MM-DD.csv
- [ ] Toast "Exported X trades" on success; "No trades to export" if empty

**Data Persistence (REQ-009):**
- [ ] Key: `tradesync_trades` in localStorage
- [ ] JSON array of trade objects
- [ ] Graceful error: "Storage full" toast if quota exceeded
- [ ] Graceful error: "Unable to load" if localStorage unavailable

**UI/Design (REQ-010, REQ-011):**
- [ ] Dark theme: bg #0a0e17, cards #161d2e, borders #2a3548
- [ ] Fonts: Outfit (headings/UI), JetBrains Mono (numbers)
- [ ] Mobile: single column <600px, 2-col stat grid
- [ ] Toast notifications: bottom-right, slide up, auto-dismiss 3s

**Test All Flows:**
- [ ] FLOW-001: Log a trade (valid entry → saved + toast)
- [ ] FLOW-001 error: missing ticker → inline error
- [ ] FLOW-001 error: missing tilt → inline error
- [ ] FLOW-002: Dashboard renders with stats
- [ ] FLOW-003: Journal shows all trades
- [ ] FLOW-004: Delete with confirmation
- [ ] FLOW-005: Export CSV downloads file
- [ ] FLOW-006: Tilt filter shows only tilt trades

**Empty States:**
- [ ] No trades: stats show "--", journal shows "No trades yet"
- [ ] No tilt trades: filter shows "No tilt trades yet 🎉"

### Technical Requirements
- Single HTML file, no build step
- Google Fonts loaded via CDN (Outfit, JetBrains Mono)
- Vanilla JS, no frameworks
- All state in localStorage
- No external JS dependencies except fonts

### Quality Bar
- Zero placeholder code
- All 22 test cases from TEST_PLAN pass mentally (test your code against them)
- Works on Chrome, Firefox, Safari latest
- No console errors

---

## TICKET-002: Quality Assurance — Test & Verify
**Priority:** P0
**Estimated Time:** 15-20 minutes
**Assignee:** QA Agent

### Context
TradeSync MVP built at `/opt/openclaw/mvp-projects/trading-journal-sync/dist/index.html`. 
Full spec at SPEC-BUNDLE.md. Test plan at TEST_PLAN.md.

### Deliverable
Execute manual test cases from TEST_PLAN.md and document results.
Write findings to `/opt/openclaw/mvp-projects/trading-journal-sync/TEST-RESULTS.md`.

### Test Cases to Execute

**TEST-001 through TEST-022** (from TEST_PLAN.md):

For each test:
1. Describe the setup
2. Perform the steps
3. Record actual result
4. Pass/Fail against expected

**Critical tests (must all pass before ticket close):**
- TEST-003: P&L calculation accuracy (5 sub-cases)
- TEST-004: Tilt trade flag (tilt=4, loss → flagged)
- TEST-005: NOT flagged (tilt=5, profit → not flagged)
- TEST-006: NOT flagged (tilt=3, loss → not flagged)
- TEST-009: Dashboard stats with 4 mixed trades
- TEST-012: CSV export all fields
- TEST-015: Delete confirm removes trade
- TEST-017: Data persists after "refresh" (clear and reload)

**Report format** (TEST-RESULTS.md):
```
## Test Results Summary
Total: 22 tests
Passed: X
Failed: Y

## Failed Tests (if any)
[detail each failure with reproduction steps]

## Test Detail Log
[each test with pass/fail + notes]
```

---

## TICKET-003: Package & Deploy
**Priority:** P1
**Estimated Time:** 5-10 minutes
**Assignee:** DevOps Agent

### Context
TradeSync MVP complete and tested. Need deploy package + deployment.

### Deliverable
1. Deploy package: `/opt/openclaw/deploy-packages/tradesync-pro-deploy.zip`
   - Contains: `dist/index.html` (the single file)

2. Deploy to: Netlify (or GitHub Pages as fallback)

### Steps
1. Create zip from `dist/index.html` only (or with a simple README)
2. Attempt Netlify deploy via UI token if available
3. If Netlify blocked (no token), push to GitHub Pages:
   - Create/update `tradesync-pro` repo on GitHub
   - Push dist/index.html as `index.html`
   - Enable GitHub Pages
4. Report live URL

### Fallback
If all cloud deploys fail, document the manual deploy steps in README at:
`/opt/openclaw/mvp-projects/trading-journal-sync/README.md`

---

## Ticket Handoff Notes

### For Frontend Builder
- Read SPEC-BUNDLE.md first — especially REQ-001 through REQ-012 and all FLOW-### entries
- The existing `dist/index.html` is a reference implementation (MVP-quality)
- New build should be equivalent or better — the spec is the source of truth
- Pay special attention to P&L formula: (exit-entry)×size×direction_multiplier
- Tilt flag logic: tilt ≥ 4 AND pnl < 0

### For QA Agent  
- The test plan has 22 tests — execute all
- P&L calculation has 5 sub-cases — test each
- Focus on the "critical" tests first

### For DevOps
- Single HTML file deploy is trivial
- Check if Netlify token is in env before falling back to GitHub Pages
