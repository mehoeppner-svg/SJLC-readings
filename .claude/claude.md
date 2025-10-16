# SJLC Gospel Project - Claude Context

## Project Overview
Python program to process Bible verse references into HTML files with images, audio, and navigation for daily devotional reading.

## Critical Path Information

### Project Locations
- **Working Directory (Git Repo)**: `c:\pyApps\SJLC-readings`
- **Actual Project Code**: `c:\pyApps\sjlc-private\`
  - Backend/CLI code: `c:\pyApps\sjlc-private\src\`
  - Frontend/GUI code: `c:\pyApps\sjlc-private\gui\`
  - Configuration: `c:\pyApps\sjlc-private\config\`
  - Utilities: `c:\pyApps\sjlc-private\utils\`
  - Templates: `c:\pyApps\sjlc-private\gui\templates\`
  - Routes: `c:\pyApps\sjlc-private\gui\routes\`

### Key Project Structure
```
c:\pyApps\sjlc-private\
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
