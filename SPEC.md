# TradeSync - Trading Journal & Psychology Dashboard

## Project Overview
- **Project Name:** TradeSync
- **Type:** Single-page web application (MVP)
- **Core Functionality:** A trading journal that tracks emotional state alongside trade entries, helping retail traders identify tilt-driven losses
- **Target Users:** Retail traders (especially TradingView users)

## UI/UX Specification

### Layout Structure
- **Header:** Project title and tagline
- **Dashboard Grid:** 6 stat cards (Win Rate, Total P&L, Avg Win, Avg Loss, Largest Win, Largest Loss)
- **Main Content:** Two-column layout on desktop (form left, journal right), stacked on mobile

### Responsive Breakpoints
- Desktop: > 900px (two-column grid)
- Mobile: < 600px (stacked layout, 2-column stats)

### Visual Design

#### Color Palette
- Background Primary: `#0a0e17` (deep navy black)
- Background Secondary: `#131a2a` (dark navy)
- Background Card: `#161d2e` (card surface)
- Border: `#2a3548` (subtle borders)
- Text Primary: `#e8ecf4` (off-white)
- Text Secondary: `#8892a4` (muted gray)
- Text Muted: `#5c6578` (very muted)
- Accent Green: `#00d9a5` (profit/winning)
- Accent Red: `#ff4757` (loss/tilt warning)
- Accent Blue: `#4f7cff` (interactive elements)
- Accent Yellow: `#ffc107` (medium tilt)
- Accent Orange: `#ff7b4a` (medium-high tilt)

#### Typography
- Headings/UI: 'Outfit' (modern sans-serif)
- Numbers/Data: 'JetBrains Mono' (monospace)
- Base size: 16px
- Headings: 1.1rem - 1.8rem

#### Spacing
- Container padding: 20px
- Card padding: 24px
- Grid gap: 16px - 24px
- Form gaps: 12px - 16px

### Components

#### Stat Cards
- Border radius: 12px
- Hover: subtle lift with shadow
- Value colors: green for positive, red for negative

#### Trade Entry Form
- Dark input fields with subtle borders
- Focus state: blue border
- Tilt selector: 5 button row, color-coded when selected (1-3 yellow/orange, 4-5 red)
- Submit button: full-width blue

#### Trade List Items
- Grid layout with direction badge, ticker, details, P&L, tilt badge, delete button
- Tilt-flag trades (tilt >= 4 AND P&L < 0): red border + warning badge
- Scrollable container with custom scrollbar

#### Weekly Chart
- Bar chart showing P&L per day for last 7 days
- Green bars for profit, red for loss
- Day labels below, values above bars

#### Toast Notifications
- Bottom-right positioned
- Slide-up animation
- Auto-dismiss after 3 seconds

## Functionality Specification

### Core Features

1. **Trade Entry Form**
   - Ticker symbol (text input, auto-uppercase)
   - Direction (Long/Short dropdown)
   - Entry price (number)
   - Exit price (number)
   - Position size (number)
   - Trade date (date picker, defaults to today)
   - Emotional tilt (1-5 scale via button selector)
   - P&L calculated: (exit - entry) * size * (1 for long, -1 for short)

2. **Trade Journal Log**
   - Chronological list (newest first)
   - Shows: direction, ticker, date, entry→exit, size, P&L, tilt level
   - Delete button per trade

3. **Dashboard Stats**
   - Win Rate: wins / total * 100%
   - Total P&L: sum of all P&L
   - Average Win: mean of positive P&L trades
   - Average Loss: mean of negative P&L trades
   - Largest Win: max positive P&L
   - Largest Loss: max negative P&L (most negative)

4. **Psychology Flags**
   - Auto-detect "tilt trades": tilt >= 4 AND P&L < 0
   - Highlight with red border and warning badge

5. **Weekly Summary**
   - Bar chart of P&L for each of last 7 days
   - Updates dynamically as trades are added

6. **Export CSV**
   - Download all trades as CSV file
   - Columns: Date, Ticker, Direction, Entry, Exit, Size, P&L, Tilt

### Data Storage
- localStorage with key: `tradesync_trades`
- JSON array of trade objects
- Persists across browser sessions

### Edge Cases
- Empty state: show friendly message when no trades
- No wins/losses yet: show "--" for avg calculations
- Delete confirmation before removing trades

## Acceptance Criteria

1. ✓ Single HTML file with no external dependencies except Google Fonts
2. ✓ All 6 stat cards display correct calculated values
3. ✓ Trade form adds new entries to localStorage
4. ✓ Trades display in journal with correct formatting
5. ✓ Tilt trades (tilt >= 4 + loss) show red highlight + warning
6. ✓ Weekly chart renders last 7 days of P&L
7. ✓ CSV export downloads valid file
8. ✓ Dark theme applied throughout
9. ✓ Mobile responsive layout works at 600px and below
10. ✓ Data persists after page refresh
