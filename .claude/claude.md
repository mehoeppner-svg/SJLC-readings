# SJLC Daily Bible Readings - Claude Context (V3)

## Project Overview
Python/Flask application for managing daily Bible reading content with verse references, AI-generated images, and audio. Generates HTML pages for Google Sites embedding. V3 simplifies the admin interface to 3 streamlined pages (Import, Collections, Content) with a month calendar as the primary view.

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

## Workflow & Task Approval

**CRITICAL: Only do what is explicitly approved.**

1. **Present plan for ONE task at a time** - Don't bundle multiple tasks together
2. **Wait for explicit approval** - User must approve the plan before implementation
3. **Stay within approved scope** - Only implement what was approved, nothing more
4. **Ask before proceeding to next task** - After completing approved work, ask if user wants to proceed to the next task

**Example workflow:**
- User: "Let's start with task 2.2"
- Claude: *presents plan for task 2.2 only*
- User: *approves*
- Claude: *implements task 2.2*
- Claude: "Task 2.2 complete. Ready to proceed to task 2.3?"
- User: *approves next task*

**Do NOT:**
- Automatically continue to the next task after completing one
- Bundle multiple tasks into a single approval request (unless user explicitly asks)
- Assume approval for related tasks

---

## Repository Structure

### Two Separate Repositories

**IMPORTANT:** Repository locations are configured in `.env` file variables:
- `GITHUB_PUBLIC_REPO` - Path to public SJLC-readings repository
- `GITHUB_PRIVATE_REPO` - Path to private sjlc-private repository

**1. Public Repository: `SJLC-readings`** (path from `GITHUB_PUBLIC_REPO` in .env)
- **Purpose:** Published HTML content served via GitHub Pages
- **URL:** https://mehoeppner-svg.github.io/SJLC-readings
- **Contents:**
  - `years/{year}/content/{date}_reading.html` - Daily HTML files
  - `daily_content/{year}_daily_content.json` - Content index for fast loading
  - `.claude/claude.md` - This documentation file

**2. Private Repository: `sjlc-private`** (path from `GITHUB_PRIVATE_REPO` in .env)
- **Purpose:** Backend code and admin interface (NOT published)
- **Contents:**
  - `src/` - Backend Python classes
  - `gui/` - Flask admin application
  - `config/` - API keys, credentials, settings
  - `data/` - Database, backups
  - `utils/` - Helper modules
  - `logs/` - Application logs

---

## V3 Data Model

### Core Entities

**1. Readings** (Daily Bible passages)
```python
{
    "date": "2025-09-08",           # YYYY-MM-DD format
    "reference": "Genesis 1:1-13",  # Bible reference
    "collection_id": 1,             # FK to collections (nullable)
    "theme": "Creation"             # Theme for this reading (nullable)
}
```
- **Flat structure** - Collection and theme stored directly on reading
- **One reading per day** - Sundays typically omitted
- **~352 readings per year**

**2. Collections** (Simple tags like "Gospel Project", "Advent 2024", "Lent 2025")
```python
{
    "id": 1,
    "name": "Gospel Project",
    "color": "#72abbf"              # Hex color for UI display
}
```
- **Simple tags** - Just id, name, color (no date ranges)
- **Linked to readings** - Via collection_id foreign key
- **Usage tracking** - Count readings using each collection
- **Visual indicators** - 4px colored left borders on calendar days

**3. Content Status** (Tracks generated content and caches Drive file IDs)
```python
{
    "date": "2025-09-08",
    "drive_image_id": "1abc...xyz",     # Cached Google Drive file ID
    "github_html_exists": true,          # Whether HTML file exists
    "devotional_exists": false,          # Future content type
    "commentary_exists": false,          # Future content type
    "verse_card_exists": false,          # Future content type
    "last_generated": "2025-01-15T10:30:00"
}
```
- **Cache Drive file IDs** - Eliminates API calls when displaying content
- **Status tracking** - Complete (image + HTML), Partial (only one), Missing (neither)
- **Expandable** - Additional content types can be added

### Database: SQLite

**Why SQLite:**
- Zero cost (no server hosting fees)
- Perfect scale (~7,000 readings over 20 years = tiny dataset)
- ACID transactions (no race conditions)
- Fast queries with indexes
- Easy backup (single .db file)
- No maintenance required

**Schema:**
```sql
readings (id, date, reference, collection_id, theme, created_at, updated_at)
collections (id, name, color, created_at)
content_status (id, date, drive_image_id, github_html_exists, devotional_exists, commentary_exists, verse_card_exists, last_generated)
drive_folders (year, folder_id, last_scanned)
```

**Location:** `sjlc-private/data/sjlc_readings.db`

---

## Project Structure (Detailed)

**Note:** Paths shown are examples. Actual locations from `.env` variables:
- Private repo: `$GITHUB_PRIVATE_REPO`
- Public repo: `$GITHUB_PUBLIC_REPO`

```
$GITHUB_PRIVATE_REPO/             # Private repository (from .env)
├── src/                          # Backend Python (REQUIRE APPROVAL)
│   ├── database_manager.py      # SQLite operations
│   ├── readings_manager.py      # Readings CRUD (with collection_id, theme)
│   ├── collections_manager.py   # Collections CRUD (simplified for V3)
│   ├── day_processor.py         # Content generation orchestration
│   ├── bible_manager.py         # ESV API integration
│   ├── image_manager.py         # OpenAI DALL-E integration
│   ├── content_manager.py       # HTML assembly + status tracking
│   ├── drive_manager.py         # Google Drive operations
│   └── github_manager.py        # GitHub filesystem operations
├── gui/                          # Flask admin app (CAN MODIFY)
│   ├── web_app.py               # Flask app initialization
│   ├── routes/                  # API endpoints
│   │   ├── dashboard.py
│   │   ├── import_bp.py         # Import readings (V3)
│   │   ├── collections.py       # Simplified collections CRUD (V3)
│   │   ├── content.py           # Calendar + day popup (V3)
│   │   └── settings.py
│   ├── templates/               # Jinja2 templates
│   │   ├── base.html
│   │   ├── dashboard.html
│   │   ├── import.html          # V3 import page
│   │   ├── collections_v3.html  # V3 simplified collections
│   │   ├── content_v3.html      # V3 calendar + day popup
│   │   └── settings.html
│   └── static/                  # CSS/JS assets
│       ├── css/
│       └── js/
├── config/                       # Configuration (REQUIRE APPROVAL)
│   ├── config.py                # Settings, API keys
│   ├── googleDriveAPI.json      # Drive credentials
│   └── token.json               # OAuth token
├── data/                         # Data storage
│   ├── sjlc_readings.db         # SQLite database
│   └── static_files/            # JS/CSS templates for Google Sites
├── utils/                        # Utilities
│   ├── date_helpers.py          # Date parsing/formatting
│   ├── api_helpers.py           # Retry logic, rate limiting
│   └── file_helpers.py          # Safe file I/O
├── logs/                         # Application logs
│   ├── errors.log
│   ├── processing.log
│   └── system.log
└── .env                          # Environment variables

$GITHUB_PUBLIC_REPO/              # Public repository (from .env)
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
    └── {date}_image.webp
```

**Auth:** OAuth 2.0 (credentials.json → token.json)
**Operations:** Upload, delete, list files, get file IDs
**Manager:** `drive_manager.py`
**Thread Safety:** Single `_api_lock` for ALL Drive operations (critical)

**Caching:** File IDs cached in `content_status` table to eliminate API calls during display

**Note:** Images stored directly in year folder (e.g., "2025/"), not in "content" subfolder

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
- **Contents:** Project overview, data model, repository structure, external services
- **Updates:** Only when architecture changes (rare)

**2. `sjlc-private/backend.md`**
- **Purpose:** Complete backend technical specification
- **When to read:** When working on backend code (managers, database, APIs)
- **Contents:** Database schema, manager architecture, processing flows, external API integration, threading
- **Updates:** When backend implementation details change

**3. `sjlc-private/gui/admin.md`**
- **Purpose:** Complete admin GUI specification
- **When to read:** When working on frontend (templates, routes, UI/UX)
- **Contents:** Design system, page specifications, data flow (API contracts), component library
- **Updates:** When UI specifications change

**4. `sjlc-private/v3-todo.md`**
- **Purpose:** Implementation plan and progress tracking
- **When to read:** To see current status, next steps, what's completed
- **Contents:** Phase-by-phase plan, task checkboxes, changelog, current blockers
- **Updates:** After completing each task, mark as done and update status

### How to Use These Files

**Starting a new session:**
1. Read `claude.md` (this file) - 5 min architectural overview
2. Read `v3-todo.md` - See current status and next task
3. Read `backend.md` or `admin.md` - Depending on task type
4. Begin work

**During work:**
- Reference `backend.md` or `admin.md` for detailed specs
- Update `v3-todo.md` after completing tasks
- Only update `claude.md`, `backend.md`, `admin.md` if architecture/specs change

---

## Major V2 → V3 Changes

### Data Model
| Aspect | V2 | V3 |
|--------|----|----|
| **Collections** | Separate table with date ranges | Simple tags (id, name, color only) |
| **Date Ranges** | `collection_weeks` table | Removed - no date range matching |
| **Theme Storage** | In collection_weeks table | Directly on readings table |
| **Collection Link** | Date-based matching | FK `readings.collection_id` |
| **Content Status** | 3 fields | 6 fields (added devotional, commentary, verse_card) |

### Admin Interface
| Feature | V2 | V3 |
|---------|----|----|
| **Navigation** | Dashboard, Readings, Collections, Content, Settings | Dashboard, Import, Collections, Content, Settings |
| **Readings Manager** | 4 views (Calendar, List, Collections, Year) | Removed - replaced by Import page |
| **Collections Manager** | 4 views with date range management | Simple CRUD table (no date ranges) |
| **Content Manager** | 2 views (Calendar, List) | Month calendar + day popup modal |
| **Primary Interface** | Multiple views per manager | Single calendar view with popup for all CRUD |
| **Add Readings** | Modal in Readings Manager | Import page (file upload) or day popup |
| **Edit Readings** | Bulk edit modal | Day popup (single reading at a time) |

### Backend Changes
| Component | V2 | V3 |
|-----------|----|----|
| **readings_manager** | date, reference only | date, reference, collection_id, theme |
| **collections_manager** | Full CRUD + date range methods | Simplified CRUD + usage count |
| **collection_weeks table** | Exists | Dropped |

### User Experience
| Aspect | V2 | V3 |
|--------|----|----|
| **Workflow** | Navigate between manager views | Single calendar, click day for all operations |
| **Add Reading** | Open modal, fill form | Click empty day, fill popup, save |
| **Edit Reading** | Select, click edit, bulk modal | Click day, edit in popup, save |
| **Assign Collection** | Date range overlaps | Direct dropdown in day popup |
| **Create Collection** | Collections Manager | Inline in day popup ("+ New Collection") |

---

## File Location Quick Reference

**Note:** Use `$GITHUB_PRIVATE_REPO` and `$GITHUB_PUBLIC_REPO` environment variables from `.env` file.

### Backend Code
- **Database:** `$GITHUB_PRIVATE_REPO/data/sjlc_readings.db`
- **Managers:** `$GITHUB_PRIVATE_REPO/src/*_manager.py`
- **Utils:** `$GITHUB_PRIVATE_REPO/utils/*.py`
- **Config:** `$GITHUB_PRIVATE_REPO/config/config.py` and `.env`

### Frontend Code
- **Flask App:** `$GITHUB_PRIVATE_REPO/gui/web_app.py`
- **Routes:** `$GITHUB_PRIVATE_REPO/gui/routes/*.py`
- **Templates:** `$GITHUB_PRIVATE_REPO/gui/templates/*.html`

### Published Content
- **HTML:** `$GITHUB_PUBLIC_REPO/years/{year}/content/{date}_reading.html`
- **JSON Index:** `$GITHUB_PUBLIC_REPO/daily_content/{year}_daily_content.json`

### Logs
- **Errors:** `$GITHUB_PRIVATE_REPO/logs/errors.log`
- **Processing:** `$GITHUB_PRIVATE_REPO/logs/processing.log`
- **System:** `$GITHUB_PRIVATE_REPO/logs/system.log`

### Documentation
- **Architecture:** `$GITHUB_PUBLIC_REPO/.claude/claude.md` (this file)
- **Backend Details:** `$GITHUB_PRIVATE_REPO/backend.md`
- **Admin GUI Details:** `$GITHUB_PRIVATE_REPO/gui/admin.md`

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

### Error Display
- **NEVER use flash messages** (banners that push content down)
- **Preferred:** Inline error messages within modals/forms
- **Alternative:** Alert pop-ups for critical errors
- **Example:** Use `showError()` function to display inline errors in modal forms

---

## Design Principles

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

**IMPORTANT:** Repository paths are configured in `.env` file:
- `GITHUB_PUBLIC_REPO` - Path to public SJLC-readings repository
- `GITHUB_PRIVATE_REPO` - Path to private sjlc-private repository

Always use these environment variables. Never hardcode paths.

### Backend Code (sjlc-private repo from GITHUB_PRIVATE_REPO)
```
$GITHUB_PRIVATE_REPO/
├── src\*.py                    # Python managers
├── gui\web_app.py             # Flask app
├── gui\routes\*.py            # API routes
├── gui\templates\*.html       # Jinja2 templates
├── gui\static\                # CSS/JS/images
├── config\*                   # Configuration files
├── utils\*.py                 # Helper modules
└── data\sjlc_readings.db      # SQLite database
```

### Published Content (SJLC-readings repo from GITHUB_PUBLIC_REPO)
```
$GITHUB_PUBLIC_REPO/
├── years\{year}\content\*.html    # Daily HTML files
├── daily_content\*.json           # JSON indexes
└── .claude\*.md                   # Documentation
```

### Archive (Reference only, don't modify)
```
$GITHUB_PRIVATE_REPO/gui/archive/
├── v1\                        # V1 backup code
└── v2\                        # V2 backup code
```

**Never look in SJLC-readings repo for backend Python code!**

---

## Quick Start for New Claude Sessions

1. **Read this file first** (5 min) - Architectural overview
2. **Read `v3-todo.md`** - Current status and next task
3. **Read `backend.md` or `admin.md`** - Detailed specs for your task
4. **Begin work** - Reference docs as needed

---

## Notes

- **V3 implementation in progress** - See `v3-todo.md` for current status
- **Two repos:** Keep public (HTML) and private (code) separate
- **Simple collections:** Tags only, no date ranges
- **Calendar-first:** Month calendar is primary interface
- **Day popup:** All CRUD operations in one modal
- **Cache Drive IDs:** Store in database to eliminate API calls
- **Calendar years only:** January-December (no Sept-Aug confusion)

---

**Last Updated:** 2025-01-19
**Status:** V3 Implementation Starting - See v3-todo.md
