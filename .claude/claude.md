# SJLC Daily Bible Readings - Claude Context

## Project Overview
Python/Flask application to manage daily Bible reading content with verse references, images, audio, and optional collections (Gospel Project, Advent, etc.). Generates HTML pages for Google Sites embedding.

## Code Modification Rules

**Can modify directly (no approval needed):**
- `sjlc-private/gui/` - Flask frontend (templates, routes, static files)
- `sjlc-private/testing/` - Admin GUI prototypes (standalone HTML files)
- Any `.html`, `.css`, `.js` files

**Require approval before modification:**
- `sjlc-private/src/` - Backend Python code
- `sjlc-private/config/` - Configuration files
- `sjlc-private/utils/` - Utility modules
- Any `.py` files outside `gui/` folder

**Approval process:** Present complete design plan first, wait for approval, then implement.

## Project Structure

```
sjlc-private/
├── src/                    # Backend (day_processor, content_manager, bible_manager, etc.)
├── gui/                    # Flask app (routes/, templates/)
├── testing/                # Admin GUI V2 prototypes (standalone HTML with inline CSS/JS)
│   ├── admin-dashboard.html
│   ├── readings-manager.html    # ✅ Calendar/List/Collections/Year views complete
│   ├── content-manager.html     # TODO - Next
│   ├── maintenance.html
│   └── settings.html
├── config/                 # config.py, credentials.json
├── utils/                  # logger.py, backup_configs.py
├── logs/                   # errors.log, processing.log, system.log
└── .env                    # API keys (ESV_API_KEY, OPENAI_API_KEY, FLASK_SECRET_KEY)
```

## Data Model (V2 - Collections-Based)

### Daily Readings
- **Core entity:** Date + Bible verse reference only
- **No embedded metadata:** No theme, no project flags
- **Example:** `{ date: '2024-11-01', reference: 'Genesis 7:1-24' }`

### Collections
- **Definition:** Named projects with weekly themes (e.g., "Gospel Project", "Advent 2024", "Psalms Series")
- **Structure:** `{ name, color (hex), weeks: [{start, end, theme}] }`
- **Relationship:** Implicit by date range (if reading date falls within collection week, it's part of that collection)
- **Visual:** Each collection has custom color shown as:
  - 4px colored left border on calendar days
  - Colored dot (●) + name in list view
  - Legend bars below navigation

### Special Values
- **"General"**: Readings NOT in any collection (filter value: `__general__`)
- **"All Readings"**: Default filter showing everything

## Admin GUI V2 Architecture

### Design System
- **Colors:** Teal accent (#72abbf), purple header gradient (#667eea → #764ba2)
- **Typography:** Gotham sans-serif (body), Sentinel/Georgia serif (headings)
- **Responsive:** 768px breakpoint (mobile: single column, desktop: multi-column)
- **Components:** White cards, 6-8px border-radius, subtle shadows, icon-only action buttons

### Readings Manager (readings-manager.html)
**Status:** ✅ Complete (4 views)

#### 1. Calendar View
- Month-at-a-time grid with ‹ [Month ▼] [Year ▼] › navigation
- Days show: date number, verse reference, 4px colored left border (by collection)
- Click day to toggle selection (bulk edit/delete via action buttons)
- Collection legend below navigation

#### 2. List View
- Sortable table: Date, Reference (Bible book canonical order), Collection
- Filters: Month, Year, Collection (All/General/specific)
- Search by date or reference
- Pagination (10/30/50/100 per page)
- Bulk selection syncs with calendar view

#### 3. Collections View
- Collection selector dropdown with color picker
- Weeks table (start date, end date, theme)
- Bulk edit/delete weeks with select-all checkbox
- Rename/Delete collection buttons
- Color updates re-render all views in real-time

#### 4. Year Overview
- 12 mini-month calendars in grid (responsive)
- Year dropdown with ‹ › arrows
- Collection legend (shows collections in current year)
- Day click → compact floating popup (date, collection, reference, edit icon)
- View-only (no bulk selection)
- Popup closes on: ESC, click outside, click same day again, click different day

### Content Manager (TODO - Next up)
- View content status (HTML ✅/❌, Image ✅/❌)
- Generate/regenerate/delete content
- Preview modal with rendered HTML + image thumbnail
- 3-screen generation workflow (confirm → image approval → final processing)

### Common Patterns
- **Selection state:** Global `selectedReadings` and `selectedWeeks` Sets
- **Action buttons:** Fade in/out using opacity transitions (icon-only: ✕ ✏️ 🗑️)
- **Bible book sorting:** 66-book lookup table (Genesis=1, Revelation=66)
- **Date filtering:** Parse YYYY-MM-DD strings, compare month/year values

## Key Technical Rules

### Backend
1. **Logging:** Always use `logger.info/error/success()`, never `print()`
2. **Encoding:** All file ops use `encoding='utf-8'` (Windows compatibility)
3. **API Keys:** Stored in `.env`, loaded via `python-dotenv` in config.py
4. **Backups:** Auto-backup to Google Drive (2-file rotation) after settings changes

### Frontend
1. **Google Sites embeds:** No `<html>/<head>/<body>` tags, inline CSS/JS only
2. **Admin GUI prototypes:** Standalone HTML with dummy data (Phase 1), Flask templates later (Phase 2)
3. **Button states:** Disable during async operations, show loading text
4. **Modal positioning:** Click-outside to close, ESC key support

## File Locations
- **Google Sites templates:** `sjlc-private/testing/test-*.html`
- **Admin GUI prototypes:** `sjlc-private/testing/admin-*.html` and `*-manager.html`
- **Backend classes:** `sjlc-private/src/*.py`
- **Configuration:** `sjlc-private/config/config.py` and `.env`

## Current Status (Nov 1, 2025)
- ✅ **Readings Manager:** All 4 views complete (Calendar, List, Collections, Year)
- ⏳ **Content Manager:** Next priority
- 📋 **Future:** Maintenance page, Settings page, backend integration
