# SJLC Daily Bible Readings - Claude Context (V2)

## Project Overview
Python/Flask application for managing daily Bible reading content with verse references, AI-generated images, and audio. Generates HTML pages for Google Sites embedding. V2 redesign focuses on simplifying the data model (calendar years + collections), migrating to SQLite database, and modernizing the admin interface.

**Purpose:** Help St. John Lutheran Church congregation engage with daily Bible readings through visual and audio content.

**Users:** ~10 daily (potential to grow to hundreds), church admin manages content generation.

---

## Code Modification Rules

**Can modify directly (no approval needed):**
- `sjlc-private/gui/` - Flask admin interface (templates, routes, static files)
- `sjlc-private/gui/testing/` - Admin GUI prototypes (standalone HTML files)
- Any `.html`, `.css`, `.js` files in GUI directories

**Require approval before modification:**
- `sjlc-private/src/` - Backend Python code (core business logic)
- `sjlc-private/config/` - Configuration files
- `sjlc-private/utils/` - Utility modules
- Any `.py` files outside `gui/` folder

**Approval process:** Present complete design plan first, wait for approval, then implement.

---

## Repository Structure

### Two Separate Repositories

**1. Public Repository: `SJLC-readings`** (c:\pyApps\SJLC-readings)
- **Purpose:** Published HTML content served via GitHub Pages
- **URL:** https://mehoeppner-svg.github.io/SJLC-readings
- **Contents:**
  - `years/{year}/content/{date}_reading.html` - Daily HTML files
  - `daily_content/{year}_daily_content.json` - Content index for fast loading
  - `.claude/claude.md` - This documentation file

**2. Private Repository: `sjlc-private`** (c:\pyApps\sjlc-private)
- **Purpose:** Backend code and admin interface (NOT published)
- **Contents:**
  - `src/` - Backend Python classes
  - `gui/` - Flask admin application
  - `config/` - API keys, credentials, settings
  - `data/` - Database, backups
  - `utils/` - Helper modules
  - `logs/` - Application logs

---

## V2 Data Model

### Core Entities

**1. Readings** (Daily Bible passages)
```python
{
    "date": "2025-09-08",           # YYYY-MM-DD format
    "reference": "Genesis 1:1-13"   # Bible reference only
}
```
- **Simple flat structure** - No embedded themes or project flags
- **One reading per day** - Sundays typically omitted
- **~352 readings per year**

**2. Collections** (Organizational containers like "Gospel Project", "Advent 2024", "Lent 2025")
```python
{
    "name": "Gospel Project",
    "color": "#72abbf",             # Hex color for UI display
    "weeks": [                       # Note: "weeks" = flexible date ranges (not just 7 days)
        {
            "start": "2025-09-07",  # Range start date
            "end": "2025-09-13",    # Range end date
            "theme": "Creation"     # Theme for this date range
        },
        {
            "start": "2025-03-05",  # Single day (Ash Wednesday)
            "end": "2025-03-05",
            "theme": "Ashes & Repentance"
        },
        ...
    ]
}
```
- **Independent from readings** - Stored separately, linked by date matching only
- **Date-based matching** - If reading date falls within a collection's date range, it belongs to that collection
- **One collection per reading** - Each reading can only match ONE collection (no cross-collection overlaps)
- **Flexible date ranges** - Can be 1 day, 7 days, or any length (not restricted to weeks)
- **Visual indicators** - 4px colored left borders on calendar days, colored dots in lists

**3. Content Status** (Tracks generated content and caches Drive file IDs)
```python
{
    "date": "2025-09-08",
    "drive_image_id": "1abc...xyz",     # Cached Google Drive file ID (prevents API calls)
    "github_html_exists": true,          # Whether HTML file exists
    "last_generated": "2025-01-15T10:30:00"
}
```
- **Cache Drive file IDs** - Eliminates API calls when displaying content
- **Status tracking** - Complete (image + HTML), Partial (only one), Missing (neither)

### Database: SQLite

**Why SQLite:**
- ✅ Zero cost (no server hosting fees)
- ✅ Perfect scale (~7,000 readings over 20 years = tiny dataset)
- ✅ ACID transactions (no race conditions)
- ✅ Fast queries with indexes
- ✅ Easy backup (single .db file)
- ✅ No maintenance required

**Schema:**
```sql
readings (id, date, reference, created_at, updated_at)
collections (id, name, color, created_at)
collection_weeks (id, collection_id, start_date, end_date, theme)
content_status (id, date, drive_image_id, github_html_exists, last_generated)
drive_folders (year, folder_id, last_scanned)
```

**Location:** `sjlc-private/data/sjlc_readings.db`

---

## Project Structure (Detailed)

```
c:/pyApps/sjlc-private/           # Private repository
├── src/                          # Backend Python (REQUIRE APPROVAL)
│   ├── database_manager.py      # SQLite operations (NEW V2)
│   ├── readings_manager.py      # Readings CRUD (NEW V2)
│   ├── collections_manager.py   # Collections CRUD (NEW V2)
│   ├── day_processor.py         # Content generation orchestration (UPDATED V2)
│   ├── bible_manager.py         # ESV API integration (unchanged)
│   ├── image_manager.py         # OpenAI DALL-E integration (unchanged)
│   ├── content_manager.py       # HTML assembly (UPDATED V2)
│   ├── drive_manager.py         # Google Drive operations (SIMPLIFIED V2)
│   └── github_manager.py        # GitHub filesystem operations (unchanged)
├── gui/                          # Flask admin app (CAN MODIFY)
│   ├── web_app.py               # Flask app initialization
│   ├── routes/                  # API endpoints (WILL BE UPDATED V2)
│   │   ├── dashboard.py
│   │   ├── readings.py          # Replaces bookmarks.py (NEW V2)
│   │   ├── collections.py       # NEW V2
│   │   ├── content.py
│   │   ├── processing.py
│   │   ├── maintenance.py       # NEW V2
│   │   └── settings.py
│   ├── templates/               # Jinja2 templates (WILL BE REBUILT V2)
│   │   ├── base.html
│   │   ├── dashboard.html
│   │   ├── readings_manager.html
│   │   ├── content_manager.html
│   │   ├── maintenance.html
│   │   └── settings.html
│   ├── static/                  # CSS/JS assets
│   └── testing/                 # V2 prototypes (standalone HTML with dummy data)
│       ├── admin-dashboard.html          # ✅ Complete
│       ├── readings-manager.html         # ✅ Complete (4 views)
│       ├── content-manager.html          # ✅ Complete
│       ├── header.html                   # ✅ Complete
│       ├── maintenance.html              # ✅ Complete
│       └── settings.html                 # ✅ Complete
├── config/                       # Configuration (REQUIRE APPROVAL)
│   ├── config.py                # Settings, API keys
│   ├── googleDriveAPI.json      # Drive credentials
│   └── token.json               # OAuth token
├── data/                         # Data storage
│   ├── sjlc_readings.db         # SQLite database (NEW V2)
│   └── static_files/            # JS/CSS templates for Google Sites
├── utils/                        # Utilities (NEW V2)
│   ├── date_helpers.py          # Date parsing/formatting
│   ├── api_helpers.py           # Retry logic, rate limiting
│   └── file_helpers.py          # Safe file I/O
├── logs/                         # Application logs
│   ├── errors.log
│   ├── processing.log
│   └── system.log
└── .env                          # Environment variables (API keys)

c:/pyApps/SJLC-readings/          # Public repository
├── years/                        # Generated HTML content
│   └── {year}/
│       └── content/
│           └── {date}_reading.html
├── daily_content/                # JSON indexes
│   └── {year}_daily_content.json
└── .claude/
    └── claude.md                 # This file
```

---

## External Services Integration

### 1. ESV Bible API
**Purpose:** Fetch Bible text with audio, footnotes, cross-references
**Endpoint:** `https://api.esv.org/v3/passage/html/`
**Auth:** Bearer token (ESV_API_KEY in .env)
**Rate Limit:** 1-second delay between calls
**Manager:** `bible_manager.py`

**Returns:** HTML with:
- Bible passage text with verse numbers
- Audio player (MP3 link converted to `<audio>` element)
- Footnotes and cross-references
- Copyright attribution

### 2. OpenAI API
**Purpose:** Image generation via DALL-E 3, prompt refinement via ChatGPT
**Two-step process:**
1. **ChatGPT (gpt-4o-mini):** Refine user prompt into DALL-E optimized prompt
2. **DALL-E 3:** Generate image (1792x1024 → resized to 896x512 WebP)

**Auth:** Bearer token (OPENAI_API_KEY in .env)
**Rate Limit:** 2-second delay between calls, 3 retry attempts
**Manager:** `image_manager.py`

**Quad Images:** Generate 4 style variations (Realistic, Artistic, Watercolor, Minimalist) for user selection

### 3. Google Drive
**Purpose:** Image storage and hosting
**Folder Structure:**
```
DRIVE_FOLDERS.YEARS (root)
└── {year}/
    └── content/
        └── {date}_image.webp
```

**Auth:** OAuth 2.0 (credentials.json → token.json)
**Operations:** Upload, delete, list files, get file IDs
**Manager:** `drive_manager.py`
**Thread Safety:** Single `_api_lock` for ALL Drive operations (critical)

**Caching:** File IDs cached in `content_status` table to eliminate API calls during display

### 4. GitHub Pages
**Purpose:** HTML content hosting (public access)
**URL:** `https://mehoeppner-svg.github.io/SJLC-readings`
**Workflow:**
1. Generate HTML locally via `github_manager.py`
2. Save to local Git repository (filesystem operations only)
3. User manually commits and pushes via VS Code/CLI
4. GitHub Pages automatically serves updated content

**No API:** Direct filesystem operations, no automated deployment

---

## Documentation Structure

This file (claude.md) provides **architectural overview only**. For detailed information, consult these files:

### Core Documentation Files

**1. `SJLC-readings/.claude/claude.md`** (this file)
- **Purpose:** Quick architectural reference for new Claude sessions
- **When to read:** Always read this first (5 min scan)
- **Contents:** Project overview, data model, repository structure, external services, V1→V2 changes
- **Updates:** Only when architecture changes (rare)

**2. `sjlc-private/backend.md`**
- **Purpose:** Complete backend technical specification
- **When to read:** When working on backend code (managers, database, APIs)
- **Contents:** Database schema, manager architecture, code consolidation, processing flows, external API integration, threading
- **Updates:** When backend implementation details change

**3. `sjlc-private/gui/admin.md`**
- **Purpose:** Complete admin GUI specification
- **When to read:** When working on frontend (templates, routes, UI/UX)
- **Contents:** Design system, page specifications, data flow (API contracts), component library, V1→V2 UI changes
- **Updates:** When UI specifications change

**4. `sjlc-private/v2-todo.md`**
- **Purpose:** Implementation plan and progress tracking
- **When to read:** To see current status, next steps, what's completed
- **Contents:** Phase-by-phase plan, task checkboxes, changelog, current blockers
- **Updates:** After completing each task, mark as done and update status

### How to Use These Files

**Starting a new session:**
1. Read `claude.md` (this file) - 5 min architectural overview
2. Read `v2-todo.md` - See current status and next task
3. Read `backend.md` or `admin.md` - Depending on task type
4. Begin work

**During work:**
- Reference `backend.md` or `admin.md` for detailed specs
- Update `v2-todo.md` after completing tasks
- Only update `claude.md`, `backend.md`, `admin.md` if architecture/specs change

---

## Major V1 → V2 Changes

### Data Model
| Aspect | V1 | V2 |
|--------|----|----|
| **Year Model** | Project Year (2024-2025) | Calendar Year (2024, 2025) |
| **Terminology** | Bookmarks | Readings |
| **Structure** | Week-based with embedded themes | Flat readings + separate collections |
| **Storage** | JSON files (`*_bookmarks.json`) | SQLite database |
| **Theme Storage** | In week object with days | In collection weeks (separate table) |
| **Project Tracking** | Boolean flag on weeks | Implicit by date range matching |
| **Caching** | JSON cache files | Database table with Drive file IDs |

### Backend Architecture
| Component | V1 | V2 |
|-----------|----|----|
| **Data Access** | `bookmarks_manager.py` (JSON parsing) | `database_manager.py` (SQLite queries) |
| **Readings** | Part of bookmarks manager | Separate `readings_manager.py` |
| **Collections** | Not a concept (themes in weeks) | Separate `collections_manager.py` |
| **Caching** | `cache_manager.py` (JSON files) | Database `content_status` table |
| **Code Duplication** | Date/API/file logic scattered | Consolidated in `utils/` modules |

### Admin Interface
| Feature | V1 | V2 |
|---------|----|----|
| **Navigation** | Separate pages, tree view | Modal-based workflows, 4 main views |
| **Readings View** | Tree (week → day) + Calendar | Calendar + List + Collections + Year |
| **Content Generation** | Multi-page wizard | Single 3-screen modal |
| **Quad Images** | Separate generator page | Integrated in generation workflow |
| **Maintenance** | Part of Settings | Separate section with database sync |
| **Project Selector** | Dropdown badge (2024-2025) | Simple year dropdown (2024) |
| **Collections** | Not visible (embedded in data) | Dedicated CRUD interface with color coding |

### User Experience
| Aspect | V1 | V2 |
|--------|----|----|
| **Year Navigation** | September-to-August (confusing) | January-to-December (intuitive) |
| **Theme Discovery** | Hidden in week structure | Visual collection indicators (colors, legend) |
| **Status Display** | Basic complete/incomplete | Color-coded borders (green/orange/red) |
| **Bulk Actions** | Limited selection options | Checkbox selection syncs across views |
| **Image Selection** | Navigate away to separate page | Inline approval modals auto-open |
| **Progress Tracking** | Separate page with refresh | Real-time threaded progress in modal |

---

## File Location Quick Reference

### Backend Code
- **Database:** `sjlc-private/data/sjlc_readings.db`
- **Managers:** `sjlc-private/src/*_manager.py`
- **Utils:** `sjlc-private/utils/*.py`
- **Config:** `sjlc-private/config/config.py` and `.env`

### Frontend Code
- **Flask App:** `sjlc-private/gui/web_app.py`
- **Routes:** `sjlc-private/gui/routes/*.py`
- **Templates:** `sjlc-private/gui/templates/*.html`
- **Prototypes:** `sjlc-private/gui/testing/*.html` (V2 standalone HTML)

### Published Content
- **HTML:** `SJLC-readings/years/{year}/content/{date}_reading.html`
- **JSON Index:** `SJLC-readings/daily_content/{year}_daily_content.json`

### Logs
- **Errors:** `sjlc-private/logs/errors.log`
- **Processing:** `sjlc-private/logs/processing.log`
- **System:** `sjlc-private/logs/system.log`

### Documentation
- **Architecture:** `SJLC-readings/.claude/claude.md` (this file)
- **Backend Details:** `sjlc-private/backend.md`
- **Admin GUI Details:** `sjlc-private/gui/admin.md`

---

## Key Technical Rules

### Logging
- **Always use logger, never print():**
  ```python
  logger.info("Informational message")
  logger.success("Success message")
  logger.warning("Warning message")
  logger.error("Error message")
  ```
- **Log files:** errors.log (errors only), processing.log (generation progress), system.log (all events)

### File Encoding
- **All file operations use UTF-8** (critical for Windows compatibility)
- **Example:** `with open(path, 'r', encoding='utf-8') as f:`
- **Applies to:** JSON files, HTML files, log files

### Thread Safety
- **Drive Manager:** Single `_api_lock` for ALL Google Drive API operations
- **Database:** SQLite handles locking automatically (serialized writes)
- **Progress Tracking:** Thread-safe callbacks for UI updates

### API Rate Limiting
- **ESV:** 1-second delay between calls
- **OpenAI:** 2-second delay, 3 retry attempts with exponential backoff
- **Drive:** Retry with backoff (1, 2, 4 seconds) on errors

---

## Design Principles (V2)

### Code Organization
1. **Single Responsibility:** Each manager does ONE thing (readings, collections, images, etc.)
2. **Separate Concerns:** Data (managers) ≠ Presentation (Flask routes) ≠ UI (templates)
3. **No Duplication:** Common logic in utils modules (date, API, file helpers)
4. **Easy Maintenance:** Files < 300 lines, clear naming, comprehensive docstrings

### Data Flow
1. **User Action** (Frontend) → Flask Route → Manager → Database/API
2. **Response** Database/API → Manager → Flask Route → Template → User
3. **Caching Layer:** Drive file IDs cached in database to minimize API calls

### Error Handling
1. **Fail gracefully:** Try/except with specific exceptions
2. **Log all errors:** Use logger.error() with context
3. **User-friendly messages:** Return clear error messages to frontend
4. **Retry on transient errors:** API helpers with exponential backoff

---

## Development Workflow

### For Backend Changes (Requires Approval)
1. Present design plan in chat
2. Wait for explicit approval
3. Make changes to `src/` or `config/`
4. Test thoroughly
5. Commit with descriptive message

### For Frontend Changes (Can Modify)
1. Edit files in `gui/` directly
2. Test in browser
3. Commit when working

### For Database Changes
1. Update schema in `database_manager.py`
2. Create migration script if needed
3. Test with backup data
4. Update documentation

---

## Critical: File Location Rules

**IMPORTANT:** Always use absolute paths. Never guess repository locations.

### Backend Code (sjlc-private repo ONLY)
```
c:\pyApps\sjlc-private\
├── src\*.py                    # Python managers
├── gui\web_app_v2.py          # Flask app
├── gui\routes\*.py            # API routes
├── gui\templates\*.html       # Jinja2 templates
├── gui\static\                # CSS/JS/images
├── config\*                   # Configuration files
├── utils\*.py                 # Helper modules
└── data\sjlc_readings.db      # SQLite database
```

### Published Content (SJLC-readings repo ONLY)
```
c:\pyApps\SJLC-readings\
├── years\{year}\content\*.html    # Daily HTML files
├── daily_content\*.json           # JSON indexes
└── .claude\*.md                   # Documentation
```

### V1 Backups (Reference only, don't modify)
```
c:\pyApps\sjlc-private\
├── src\backups\*              # Old V1 managers
├── gui\routes\backups\*       # Old V1 routes
├── gui\templates\backups\*    # Old V1 templates
└── gui\web_app_v1.py          # Old Flask app (renamed)
```

**Never look in SJLC-readings repo for backend Python code!**

---

## Quick Start for New Claude Sessions

1. **Read this file first** (5 min) - Architectural overview
2. **Read `v2-todo.md`** - Current status and next task
3. **Read `backend.md` or `admin.md`** - Detailed specs for your task
4. **Begin work** - Reference docs as needed

---

## Notes

- **V2 implementation in progress** - See `v2-todo.md` for current status
- **Two repos:** Keep public (HTML) and private (code) separate
- **Database first:** SQLite foundation, then managers, then UI
- **Collections are key:** Separate from readings, matched by date ranges
- **Cache Drive IDs:** Store in database to eliminate API calls
- **Calendar years only:** No more Sept-Aug project year confusion

---

**Last Updated:** 2025-01-02
**Status:** V2 Implementation Starting - See v2-todo.md
