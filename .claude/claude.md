# SJLC Gospel Project - Claude Context

## Project Overview
Python program to process Bible verse references into HTML files with images, audio, and navigation for daily devotional reading.

## Code Modification Restrictions

**Files you CAN directly modify (without approval):**
- `c:\pyApps\sjlc-private\gui\` - Flask frontend code (templates, static files, routes)
- Any `.html`, `.css`, `.js` files anywhere in the project
- Test files (like `test-modal-slideout.html`)

**Files you CANNOT directly modify (require approval process):**
- `c:\pyApps\sjlc-private\src\` - Backend/CLI Python code
- `c:\pyApps\sjlc-private\config\` - Configuration files
- `c:\pyApps\sjlc-private\utils\` - Utility Python code
- Any `.py` files outside the `gui/` folder

**Approval Process for Backend/Src Changes:**
1. **First, propose the design** - Present goals, new functions/classes needed, architectural changes
2. **Wait for user approval** - Don't make ANY code changes until approved
3. **Once approved**, then implement the changes
4. **DO NOT** make incremental code change requests - present the full design plan first

**Example (CORRECT):**
```
"I need to modify the backend to support X feature. Here's my plan:
- Add new method `get_verse_metadata()` to bible_manager.py
- Create new class `MetadataCache` in cache_manager.py
- Update day_processor.py to call these new methods
Do you approve this design?"
```

**Example (INCORRECT):**
```
"Let me modify bible_manager.py to add this method..."
[makes change]
"Now let me update cache_manager.py..."
[makes another change]
```

## Critical Path Information

### Project Locations
**IMPORTANT:** Always use workspace-relative paths (not absolute Windows paths). The base directory varies by computer:
- Computer 1: `c:\pyApps\`
- Computer 2: `c:\Users\comal\Downloads\hoeppner\`

**Workspace Structure:**
- **Documentation Repo**: `SJLC-readings/` (this repo)
- **Actual Project Code**: `sjlc-private/`
  - Backend/CLI code: `sjlc-private/src/`
  - Frontend/GUI code: `sjlc-private/gui/`
  - Configuration: `sjlc-private/config/`
  - Utilities: `sjlc-private/utils/`
  - Templates: `sjlc-private/testing/` (**Google Sites embed templates**)
  - Flask Templates: `sjlc-private/gui/templates/`
  - Routes: `sjlc-private/gui/routes/`

### Key Project Structure
```
sjlc-private/
├── config/
│   ├── config.py           # Main configuration (reads from .env)
│   └── credentials.json    # Google Drive API credentials
├── src/                    # Backend/CLI components
│   ├── day_processor.py
│   ├── content_manager.py
│   ├── cache_manager.py
│   ├── bible_manager.py
│   ├── image_manager.py
│   ├── bookmarks_manager.py
│   ├── github_manager.py
│   └── drive_manager.py
├── gui/                    # Frontend Flask app
│   ├── routes/
│   │   ├── settings.py     # Settings management
│   │   ├── processing.py   # Content processing
│   │   └── content.py      # Content management
│   └── templates/
│       └── settings.html
├── utils/                  # Shared utilities
│   ├── logger.py          # Centralized logging system
│   └── backup_configs.py  # Google Drive config backup
├── logs/                   # Log files
│   ├── errors.log
│   ├── processing.log
│   └── system.log
├── .env                    # Environment variables (API keys)
└── .env.example           # Template for .env
```

## Key Architectural Decisions

### 1. Environment Variables & Configuration
- **API keys stored in `.env` file** (not system environment variables)
  - Reason: Portability across multiple computers
  - Keys: `ESV_API_KEY`, `OPENAI_API_KEY`, `FLASK_SECRET_KEY`
- **config.py reads from .env** using `python-dotenv`
- **Never commit .env to git** (use `.env.example` as template)
- **config.py IS safe to commit** (no hardcoded secrets)

### 2. File Encoding
- **ALL file operations must use `encoding='utf-8'`**
  - Reason: Prevents `'charmap' codec can't decode byte` errors on Windows
  - Applies to: Reading/writing .env, config.py, JSON files, logs, etc.

### 3. Logging System
- **Never use `print()` statements** - always use logger
- **Centralized logging** via `utils/logger.py`
- **Three log files**:
  - `logs/errors.log` - Error messages only
  - `logs/processing.log` - Content generation workflow
  - `logs/system.log` - All log messages
- **Logger methods**:
  - `logger.error(message)` - Errors
  - `logger.info(message)` - General info
  - `logger.success(message)` - Success messages
  - `logger.warning(message)` - Warnings
  - `logger.processing(message, level='INFO')` - Processing workflow

### 4. Google Drive Backup
- **Backup folder ID**: `1xpos_Rpl_zbKlKLZtlHiHuUBYIU0MdHw`
- **2-file rotation system** (current + .bak):
  1. Delete old `.bak` file
  2. Rename current file to `.bak`
  3. Upload new current file
- **Automatic backup** after every settings save
- **Files backed up**:
  - `config.py`
  - `credentials.json`
  - `.env`
  - `secrets.json`
  - `token.json`

### 5. Flask Web Application
- **Blueprint prefix**: `/settings` for all settings routes
- **Example route**: `@settings_bp.route('/api/api-keys', methods=['POST'])`
  - Accessible at: `/settings/api/api-keys`
- **Helper functions** to avoid code duplication:
  - `_perform_backup_and_get_message()`
  - `_read_env_file(env_path)`
  - `_write_env_file(env_path, env_dict)`
  - `_update_config_py(updates)`

## Common Patterns to Follow

### 1. Adding New Settings Routes
```python
@settings_bp.route('/api/new-endpoint', methods=['POST'])
def new_setting():
    try:
        data = request.get_json()

        # Your logic here

        # Backup after changes
        backup_msg = _perform_backup_and_get_message()

        # Log the action
        logger.success(f"Setting updated: {details}")

        return jsonify({
            'success': True,
            'message': f'Setting saved! {backup_msg}'
        })
    except Exception as e:
        logger.error(f"Error in new_setting: {str(e)}")
        return jsonify({'success': False, 'message': str(e)}), 500
```

### 2. Reading/Writing Files
```python
# ALWAYS use UTF-8 encoding
with open(file_path, 'r', encoding='utf-8') as f:
    content = f.read()

with open(file_path, 'w', encoding='utf-8') as f:
    f.write(content)
```

### 3. Using Logger Instead of Print
```python
# BAD
print("Processing started")
print(f"Error: {error}")

# GOOD
logger.info("Processing started")
logger.error(f"Error: {error}")
```

### 4. Updating .env Files
```python
# Use helper function
env_dict = _read_env_file(env_path)
env_dict['NEW_KEY'] = 'new_value'
_write_env_file(env_path, env_dict)
```

## UI/UX Patterns

### Button Styles
- Use `!important` to override external CSS
- Loading states: disable button and change text to "Processing..."
```javascript
async function saveSettings() {
    const saveBtn = event.target;
    const originalText = saveBtn.textContent;
    saveBtn.disabled = true;
    saveBtn.textContent = 'Processing...';
    try {
        // Save logic
    } finally {
        saveBtn.disabled = false;
        saveBtn.textContent = originalText;
    }
}
```

## Important Notes

1. **No emojis in terminal output** (causes encoding errors on Windows console)
   - OK to use emojis in web UI
   - Use ASCII markers in terminal: `[*]`, `[+]`, `[!]`, `[-]`, `[^]`

2. **DRY Principle**: Extract duplicated code into helper functions
   - Especially important in settings routes

3. **Error Handling**: Always wrap routes in try/except and log errors

4. **Testing**: Run `python -m py_compile <file>` to check for syntax errors

5. **Google Drive API**: Uses `credentials.json` for OAuth2 authentication

## Recent Major Changes

- Converted all `print()` statements to `logger.*()` calls across entire project
- Implemented centralized logging system with three log files
- Created Google Drive backup with 2-file rotation
- Refactored settings routes to use helper functions (31% code reduction)
- Fixed all charmap encoding errors by standardizing on UTF-8
- Made API keys portable via .env files

## V2 Daily Reading Page Template (Google Sites Embed)

### Overview
Created a modern, feature-rich daily reading page template designed for embedding in Google Sites. Template provides ESV Bible text with interactive features, verse cards, audio player, and optional content panels.

**File Location**: `sjlc-private/testing/test-daily-reading.html`

### Key Features
1. **Verse Card Image** (400px × 225px, 16:9 ratio)
   - Overlaid verse text and reference
   - Dark overlay filter (brightness 0.7)
   - Copy verse card button
   - Toggle-able via settings

2. **ESV Audio Player**
   - Integrated ESV audio with native HTML5 controls
   - Compact design (32px height on desktop, 30px on mobile)
   - Toggle-able via settings

3. **Settings Menu** (Gear Icon ⚙️)
   - **Position**: Top-right of white Bible content box
   - **Popup Direction**: Upward (above gear icon)
   - **Options**: Verse Card, Audio Player, Footnotes, Cross-References
   - **Persistence**: LocalStorage preferences across sessions

4. **Share Button** (Curved Arrow ↱)
   - **Position**: Above Bible content box, aligned right
   - Uses native Web Share API when available
   - Fallback: Copy URL to clipboard
   - Unicode icon (no external image dependencies)

5. **ESV Bible Text Styling**
   - Preserved ESV.org typography and formatting
   - Serif font (Sentinel/Georgia) at 1.1rem
   - Chapter numbers: Large drop cap style (3.2rem)
   - Verse numbers: Small superscript gray text (0.7rem)
   - Footnotes: Brown/orange `[1]` with hover popup
   - Cross-references: Blue italic `(a)` with hover popup

6. **Commentary & Devotional Panels**
   - Gradient-styled buttons (purple for commentary, pink for devotional)
   - Side panel overlay (500px max width, 90% on mobile)
   - Smooth slide-in animation
   - Auto-scroll to top of page when opened (using scroll anchor workaround)

7. **Responsive Design**
   - Desktop: 400px verse card, full features
   - Mobile (<768px): 360px verse card, stacked layout
   - Settings menu and modals adapt to screen size

### Technical Implementation

#### Google Sites Constraints
- **No `<html>`, `<head>`, or `<body>` tags** (embedded element only)
- **Inline CSS and JavaScript** (all styles in `<style>`, all scripts in `<script>`)
- **Cross-origin iframe restrictions** (can't scroll parent window)

#### Workarounds Implemented
1. **Scroll Issue**: Created invisible anchor div at top (`scroll-top-anchor`) and use `scrollIntoView()` for in-page scrolling when commentary/devotional opens
2. **Share Icon**: Use Unicode character `↱` instead of external image (Google Sites blocks some external resources)
3. **Settings Positioning**: Wrapper div provides positioning context for popup menu above gear icon

#### CSS Architecture
- **CSS Variables**: Consistent colors, fonts, spacing (`:root` vars like `--bible-text-color`, `--accent-color`)
- **ESV Preservation**: Matches ESV.org official styles for authenticity
- **Flexbox & Position**: Settings wrapper, share button, modals all use strategic positioning
- **Transitions**: Smooth animations (0.2s-0.3s) for hovers, panel slides, modal popups

#### JavaScript Features
- **LocalStorage Preferences**: `gospel_showImage`, `gospel_showAudio`, etc.
- **Modal Positioning**: Calculates position relative to clicked element (footnote/crossref)
- **Settings Menu**: Toggle visibility, click-outside-to-close detection
- **Side Panel**: Overlay + slide-in animation with ESC key support

### Design Decisions

1. **Settings Menu Pops Up (Not Down)**
   - Prevents blocking Bible text below
   - Uses `bottom: calc(100% + 0.5rem)` relative to wrapper

2. **Verse Card 400px Width**
   - Balances visibility with page layout
   - 16:9 aspect ratio = 225px height
   - Mobile scales to 360px

3. **Share Button Above (Not Inline)**
   - Keeps white Bible box clean
   - Positioned at `top: -2.5rem` with 3rem container margin

4. **No Rotation/Background on Settings Hover**
   - User preference: Simple color change only
   - Border and icon color shift to accent teal

5. **Footnotes Section Toggle-able**
   - Hide inline markers AND bottom section together
   - Maintains clean reading experience

### File Organization
```
sjlc-private/testing/
├── test-daily-reading.html    # V2 daily reading template (Google Sites embed) + VERSE SELECTION
├── test-home-page.html        # Home page template (Google Sites embed)
├── test-browse-page.html      # Browse/Archive page with calendar & search (Google Sites embed)
└── test-gospel-page.html      # Gospel Project page with timeline (Google Sites embed)
```

### Verse Selection Feature (V2 Daily Reading Page)

**Status:** ✅ Complete (October 31, 2024)

**Purpose:** Allow users to click individual verses and copy them without footnotes, verse numbers, or cross-references for clean text sharing.

#### Key Features Implemented

1. **Click-to-Select System**
   - Simple click toggles verse selection on/off (no Ctrl/Shift required)
   - Visual indicator: Dotted teal underline + subtle background
   - Hover state: Lighter dotted underline
   - Supports non-consecutive selection (e.g., verses 1, 3, 5)

2. **Floating Action Button (FAB)**
   - **Position:** Top-right of Bible content, left of gear icon (absolute position)
   - **Layout:** Horizontal with X button and copy button (📋)
   - **Verse Count Badge:** Red circular badge shows number of selected verses
   - **Auto-hide:** Only visible when verses are selected
   - **Styles:** Matches gear icon appearance (white background, compact)

3. **Copy Functionality**
   - **Click:** Click FAB copy button to copy verses
   - **Keyboard:** Ctrl+C (or Cmd+C on Mac) copies selected verses
   - **Visual Confirmation:** Button changes to checkmark (✓) for 1.5 seconds
   - **Button State:** Disabled during copy to prevent double-clicks
   - **Auto-clear:** Selection automatically clears after 1.5 second confirmation
   - **Toast Notification:** "Verses copied to clipboard!" message appears

4. **Smart Reference Formatting**
   - **Single verse:** `Genesis 1:14 - [text]`
   - **Consecutive verses:** `Genesis 1:14-16 - [text]`
   - **Non-consecutive verses:** `Genesis 1:14, 16, 18 - [text]` (compact format)
   - All 66 Bible books supported in lookup table

5. **Text Cleaning**
   - Removes footnote markers (`[1]`, `[2]`)
   - Removes verse numbers
   - Strips cross-reference markers
   - Removes HTML tags
   - Normalizes whitespace

6. **Clear Selection Methods**
   - **X Button:** Click X on FAB to clear
   - **ESC Key:** Press Escape to clear
   - **Click Outside:** Click outside Bible content (except modals)
   - **Auto-clear:** After successful copy (1.5s delay with checkmark confirmation)

7. **Modal Integration**
   - Clicking footnotes/cross-references does NOT clear selections
   - Opening/closing commentary/devotional panels preserves selections
   - Click-outside detection excludes all modal overlays

8. **Mobile Support**
   - Long-press verse for 500ms to select (with vibration feedback)
   - Responsive FAB sizing (smaller on mobile)
   - Touch event handling with move detection
   - Full-width toast notifications on mobile

#### Technical Implementation

**CSS Classes:**
- `.verse-wrapper` - Wraps each verse text
- `.verse-wrapper.selected` - Selected verse styling
- `.copy-fab` - Floating action button container (absolute positioned)
- `.copy-button` - Copy button with clipboard icon
- `.clear-selection-button` - X button
- `.verse-count-badge` - Red badge showing count
- `.verse-toast` - Success notification

**JavaScript:**
- `VerseSelector` class with ~350 lines of code
- Dynamically wraps verses on page load
- Parses ESV verse IDs: `v[bookNum][chapterNum][verseNum]-1`
- Clipboard API with fallback for older browsers
- Event delegation for click handling
- Keyboard shortcut: Ctrl+C / Cmd+C

**UX Decision (Oct 31, 2024):**
After copy, the button shows a checkmark for 1.5 seconds, then automatically clears the selection. This provides clear completion feedback and a clean slate for the next selection.

## Google Sites Template Pages (2024)

Created modern, responsive HTML templates for embedding in Google Sites (no `<html>/<head>/<body>` tags, inline CSS/JS only).

### 1. Home Page ([test-home-page.html](c:\pyApps\sjlc-private\testing\test-home-page.html))

**Purpose:** Landing page for Daily Bible Readings site

**Sections:**
- **Hero**: Tagline "Know Jesus…Make Jesus Known" + description + CTA buttons
- **Features Grid** (2x2): Scripture, Audio, Visual, Devotional
- **Worship Info**: St. John Lutheran Church service times + location
- **Final CTA**: Button to today's reading

**Key Features:**
- No duplicate header (Google Sites banner has title)
- Reduced top padding for compact layout
- Church buttons open in new tabs
- Fully responsive mobile design

**Design Decisions:**
- Removed church logo to avoid appearing church-sponsored
- Changed "Browse All Readings" from "Calendar"
- Button text: "Website" instead of "Plan Your Visit"

### 2. Browse Readings Page ([test-browse-page.html](c:\pyApps\sjlc-private\testing\test-browse-page.html))

**Purpose:** Search and browse all readings by date or keyword (renamed from "Calendar" to "Browse Readings")

**Sections:**
- **Search**: Full-text search box with Enter key support
- **Calendar**: Interactive month/year calendar with color-coded days
  - Gospel Project: Teal gradient
  - Other Reading: Purple gradient
  - No Reading: Light gray
  - Today: Bold teal border
- **Results**: Dynamic search results or empty state

**Key Features:**
- Month/year dropdowns (current ± 2 years)
- Current month display updates dynamically
- Clickable calendar days navigate to readings
- Hover tooltips show reading info
- Legend explains color coding
- Mobile-responsive grid

**Technical:**
- Sample data structure ready for backend integration
- Smooth scrolling to results
- Click day to navigate to reading page

### 3. Gospel Project Page ([test-gospel-page.html](c:\pyApps\sjlc-private\testing\test-gospel-page.html))

**Purpose:** Display 3-year Gospel Project journey with themes and progression

**Sections:**
- **Hero**: Project description (what was removed from home page)
- **Visual Timeline**: Horizontal progress bar showing journey
  - 6 major milestones: Creation → Patriarchs → Exodus → Kingdom → Jesus → Church
  - Color-coded dots (green=completed, teal=current, gray=upcoming)
  - Pulsing animation on current milestone
  - Progress percentage display
- **Collapsible Themes List**: All 100+ themes in chronological order
  - Status indicator (✓ completed, → current, empty upcoming)
  - Click to expand/collapse daily readings
  - Each day links to reading page

**Key Features:**
- Scalable to 100+ themes without overwhelming page
- Accordion keeps only one theme expanded at a time
- Day links have smooth hover effects (shift right, teal background)
- Shows progression through Scripture visually
- Fully responsive for mobile

**Design Decisions:**
- Removed stats section (3 years, 100+ themes, 700+ readings) for accuracy concerns
- Timeline provides big-picture context
- Accordion allows detailed exploration without clutter

### Common Design Patterns Across All Pages

**Color Palette:**
- Accent: `#72abbf` (teal)
- Text: `#424547` (dark gray)
- Secondary: `#5e6266` (medium gray)
- Tertiary: `#9a9fa3` (light gray)
- Background: `#f8f9fa` (off-white)

**Typography:**
- Serif: "Sentinel", Georgia (headings, titles)
- Sans: "Gotham", Helvetica (body text)

**Interactions:**
- Smooth hover effects (0.2s-0.3s transitions)
- Transform animations (translateY, scale)
- Box shadow depth changes
- Color shifts on hover

**Responsive Breakpoint:** 768px
- Mobile: Single column layouts, stacked buttons, full-width controls
- Desktop: Multi-column grids, side-by-side layouts

### Verse Selection Feature (Oct 31, 2024) ✅
**Status:** COMPLETE

**What it does:**
- Users can click any verse to select it (dotted teal underline appears)
- Click again to deselect
- Supports non-consecutive verse selection (e.g., verses 1, 3, 5)
- Floating copy button (FAB) appears at bottom-right when verses are selected
- Shows verse count badge on FAB
- X button on FAB clears all selections
- Copies verses with smart references:
  - Single: "Genesis 1:1 - text"
  - Consecutive: "Genesis 1:1-3 - text"
  - Non-consecutive: "Genesis 1:1, 3, 5 - text"
- Strips all footnotes, verse numbers, and cross-references from copied text
- Toast notification confirms successful copy
- ESC key or click-outside to clear selections
- Mobile long-press (500ms) to select with vibration feedback

**Technical Implementation:**
- JavaScript dynamically wraps verses on page load (no HTML changes needed)
- All 66 Bible books included in lookup table
- Clipboard API with fallback for older browsers
- ~160 lines of CSS, ~310 lines of JavaScript
- Z-index: 10002-10003 (above side panel at 10000-10001)

### Next Steps (Not Yet Implemented)
- Integration with backend day_processor.py to generate actual daily content
- Dynamic data population from Bible API and database
- Automated image generation for verse cards
- Commentary/devotional content fetching from external sources
