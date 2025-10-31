# SJLC Readings Project - Current Status

**Last Updated:** October 30, 2024

## Current Phase: Google Sites Template Development

We are building HTML templates for embedding in Google Sites to create a modern, user-friendly daily Bible reading experience.

## Completed Work

### ✅ Template Pages (All Complete)

#### 1. Home Page
- **File:** `c:\pyApps\sjlc-private\testing\test-home-page.html`
- **Status:** ✅ Complete
- **Features:**
  - Hero section with tagline and description
  - 2x2 feature grid (Scripture, Audio, Visual, Devotional)
  - St. John Lutheran Church worship times and location
  - CTA buttons to today's reading
- **Key Decisions:**
  - No logo to avoid appearing church-sponsored
  - Reduced top padding (Google Sites banner has title)
  - "Website" button instead of "Plan Your Visit"

#### 2. Browse Readings Page (formerly "Calendar")
- **File:** `c:\pyApps\sjlc-private\testing\test-browse-page.html`
- **Status:** ✅ Complete
- **Features:**
  - Full-text search across all readings
  - Interactive monthly calendar with color-coded days
  - Month/year dropdowns (current ± 2 years)
  - Dynamic current month display
  - Search results with date, passage, type badge
- **Color Coding:**
  - Teal gradient: Gospel Project readings
  - Purple gradient: Other readings
  - Light gray: No reading
  - Teal border: Today
- **Technical:**
  - Sample data structure ready
  - Smooth scrolling to results
  - Hover tooltips on calendar days

#### 3. Gospel Project Page
- **File:** `c:\pyApps\sjlc-private\testing\test-gospel-page.html`
- **Status:** ✅ Complete
- **Features:**
  - Project description (moved from home page)
  - Visual timeline with 6 milestones
  - Progress bar showing journey completion
  - Collapsible accordion for 100+ themes
  - Status indicators (✓ completed, → current, empty upcoming)
  - Daily reading links within each theme
- **Timeline Milestones:**
  - Creation → Patriarchs → Exodus → Kingdom → Jesus → Church
- **Key Decisions:**
  - Removed stats (3 years, 100+ themes) for accuracy
  - Pulsing animation on current milestone
  - Accordion prevents overwhelming users

#### 4. Daily Reading Page (V2)
- **File:** `c:\pyApps\sjlc-private\testing\test-daily-reading.html`
- **Status:** ✅ Complete (from previous work)
- **Features:**
  - Verse card image (400px × 225px, 16:9)
  - ESV audio player (toggle-able)
  - Settings menu (gear icon) with preferences
  - Share button (Web Share API + clipboard fallback)
  - ESV Bible text with preserved formatting
  - Footnotes & cross-references (modal popups)
  - Commentary & devotional (slide-out panels)

## Design System

### Color Palette
- **Accent:** `#72abbf` (teal)
- **Text:** `#424547` (dark gray)
- **Secondary:** `#5e6266` (medium gray)
- **Tertiary:** `#9a9fa3` (light gray)
- **Background:** `#f8f9fa` (off-white)
- **Completed:** `#4caf50` (green)
- **Current:** `#72abbf` (teal)
- **Upcoming:** `#e0e0e0` (gray)

### Typography
- **Serif:** "Sentinel", "Gentium Plus", Georgia (headings, titles)
- **Sans:** "Gotham", Helvetica, Arial (body text)
- **Bible Text:** 1.1rem with 1.7 line-height

### Responsive Design
- **Breakpoint:** 768px (mobile/tablet)
- **Mobile:** Single columns, stacked buttons, full-width controls
- **Desktop:** Multi-column grids, side-by-side layouts

### Interactions
- Smooth transitions (0.2s-0.3s)
- Hover effects: translateY, scale, box-shadow
- Color shifts on hover

## Technical Constraints

### Google Sites Requirements
- No `<html>`, `<head>`, or `<body>` tags
- All CSS in `<style>` block
- All JavaScript in `<script>` block
- Embedded element only (iframe restrictions)

### Workarounds Implemented
- Scroll anchor for in-page navigation (can't scroll parent window)
- Unicode characters instead of external images
- LocalStorage for user preferences
- Relative positioning for popup menus

## URLs & Links

### Production URLs
- **Main Site:** https://sites.google.com/view/sjlc-gospel-project-readings/
- **Today's Reading:** https://sites.google.com/view/sjlc-gospel-project-readings/today
- **Browse Readings:** https://sites.google.com/view/sjlc-gospel-project-readings/calendar

### Church Info
- **Website:** https://www.stjohnlutheran.com/
- **Livestream:** https://www.stjohnlutheran.com/livestream/
- **Location:** 315 Rosewood Avenue, Boerne, TX 78006
- **Phone:** (830) 249-3651
- **Email:** info@stjohnlutheran.com

## Next Steps (Priority Order)

### 🎯 Immediate: Verse Selection Feature (IN PROGRESS)
**Goal:** Allow users to click verses and copy them without footnotes, verse numbers, or cross-references

**Status:** Ready for implementation - design decisions finalized

**Requirements:**
- Simple toggle selection: Each click selects/deselects a verse
- Support non-consecutive verse selection (no Ctrl/Shift needed)
- Visual indicator: Dotted underline (not background highlight)
- Floating copy button appears when verses selected
- Mobile: Long-press shows copy context menu
- Include verse references in copied text
- Strip all formatting (footnotes, verse numbers, cross-refs)

**Design Decisions (Oct 30, 2024):**
- **Selection behavior**: Single click toggles verse selection on/off
- **Visual style**:
  - Selected: `border-bottom: 2px dotted #72abbf`
  - Hover: Lighter dotted underline
- **Copy format**:
  - Single verse: "Genesis 1:1 - [text]"
  - Consecutive: "Genesis 1:1-3 - [text]"
  - Non-consecutive: "Genesis 1:1, 3, 5 - [text]" (compact format)
- **Mobile**: Long-press shows context menu with "Copy Verse(s)" option
- **Clear selection**: Click outside Bible text OR press ESC key
- **Copy button**: Floating FAB (bottom-right), shows verse count

**Technical Approach:**
- Wrap each verse in `<span class="verse-wrapper" data-verse="1" data-chapter="1" data-book="Genesis">`
- JavaScript click handler for toggle selection
- Text cleaning function strips `<sup>`, verse numbers, footnotes
- Smart reference parser for consecutive vs non-consecutive verses
- Touch event handler for long-press (mobile)
- Clipboard API for copy functionality

### 📋 Backend Integration
- Connect templates to `day_processor.py`
- Populate calendar with actual reading dates
- Load Gospel Project themes from database
- Generate verse cards dynamically
- Fetch commentary/devotional content

### 🎨 Content Generation
- Automate image generation for verse cards
- Integrate ESV API for Bible text
- Set up audio file retrieval
- Build commentary database

### 🔍 Search Implementation
- Backend API for full-text search
- Index Bible passages and themes
- Support for verse references (e.g., "John 3:16")
- Keyword and topic search

### 📱 Testing & Optimization
- Test on various devices and browsers
- Optimize for Google Sites embedding
- Validate accessibility (WCAG)
- Performance testing (page load times)

## Recent Decisions & Changes

1. **Renamed "Calendar" to "Browse Readings"**
   - More accurate for past + future content
   - Includes search functionality, not just calendar

2. **Removed Stats from Gospel Project Page**
   - Can't verify "3 years, 100+ themes, 700+ readings" accuracy
   - Timeline shows progression instead

3. **No Church Logo on Home Page**
   - Avoid appearing church-sponsored
   - Keeps focus on Bible readings

4. **Church Buttons Open in New Tabs**
   - Watch Online and Website buttons use `target="_blank"`
   - Prevents users from losing their place

5. **Dropdowns Right-Aligned on Calendar**
   - Month/year controls on same line as current month display
   - Better use of horizontal space

6. **Verse Selection UX Design (Oct 30, 2024)**
   - Dotted underline for selected verses (cleaner than highlight)
   - Compact reference format for non-consecutive verses
   - Long-press context menu for mobile (native feel)
   - Click outside or ESC to clear selection

## Files & Structure

```
c:\pyApps\SJLC-readings\              # Git repo (documentation)
├── .claude\
│   └── CLAUDE.md                      # Main project context
├── HOME_PAGE_PLAN.md                  # Home page planning doc
└── CURRENT_STATUS.md                  # This file

c:\pyApps\sjlc-private\               # Actual project code
├── src\                              # Backend Python (require approval)
├── gui\                              # Flask frontend (can modify)
├── testing\
│   ├── test-daily-reading.html       # Daily reading template
│   ├── test-home-page.html           # Home page template
│   ├── test-browse-page.html         # Browse/calendar template
│   └── test-gospel-page.html         # Gospel project template
└── config\                           # Configuration (require approval)
```

## Questions & Open Items

**Verse Selection Feature - RESOLVED (Oct 30, 2024):**
- ✅ Non-consecutive verses: YES - simple click toggle
- ✅ Copied verse format: Include references in compact format
- ✅ Visual style: Dotted underline (not highlight)
- ✅ Clear selection: Click outside or ESC key
- ✅ Mobile: Long-press context menu

**Future Considerations:**
- [ ] Should verse selection persist across page reloads?
- [ ] Do we need a "Copy All" button for the entire passage?
- [ ] Should we add keyboard shortcuts (arrow keys to navigate verses)?

## Notes

- All templates use inline CSS/JS for Google Sites compatibility
- Sample data structures are ready for backend integration
- Mobile responsiveness tested on common breakpoints
- Color palette consistent across all pages
- ESV.org typography preserved for Bible text authenticity
