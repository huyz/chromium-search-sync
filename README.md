# chromium-search-sync

A Python utility for managing and synchronizing search engines across Chromium-based browser profiles (Brave and Chrome).

**Project Home**: [https://github.com/mcgroarty/chromium-search-sync](https://github.com/mcgroarty/chromium-search-sync)

This project is not affiliated with Brave Software or Google.

## TL;DR: Too Long, Didn't Read. Twitter broke my attention span.

This tool syncs custom search engine shortcuts across all your Chromium-based browser profiles (Brave and Chrome).

**Quick Start:**

1. Clone this repository or download the `chromium-search-sync` script
2. Close all Chromium browsers completely (Brave and Chrome)
3. Run `./chromium-search-sync --smoke-test` to verify system compatibility
4. Run `./chromium-search-sync -c` for a dry run preview of what will be synced
5. Run `./chromium-search-sync -cs` to sync everything
6. Reopen your browsers and visit `chrome://settings/search` (Chrome) or `brave://settings/search` (Brave) in each profile to activate the newly added search engines

**Safety:** Always backup your browser configs first (locations shown in help with `-h`). The tool won't run if browsers are open and includes built-in protections against deleting essential search engines.

## Current Status

**Tested on:** macOS and Linux

**Windows Support:** Requires Cygwin environment (native Windows not supported)

**Development:** Mostly AI-coded, but extensively human-reviewed and tested

**Bug Reports:** Include `--smoke-test` output when reporting issues on [GitHub](https://github.com/mcgroarty/chromium-search-sync/issues)

## Overview

**chromium-search-sync** is a cross-platform command-line tool that helps you manage custom search engines across multiple Chromium-based browser profiles (Brave and Chrome). If you use multiple profiles across different browsers and want to keep your search engine shortcuts synchronized, this tool automates the process by reading from and writing to the browsers' SQLite databases.

## Purpose

When you have multiple Chromium-based browser profiles (Brave and Chrome across personal, work, hobby, etc.), manually keeping custom search engine shortcuts in sync becomes tedious. This tool solves that problem by:

- **Discovering all profiles** automatically across different browsers and operating systems
- **Reading search engines** from each profile's database
- **Synchronizing shortcuts** by copying the newest version to all profiles
- **Managing shortcuts** through command-line operations

## Key Features

### Core Functionality

- **Cross-platform support**: Works on macOS, Linux, and Windows via Cygwin (Cygwin required on Windows)
- **Multi-browser support**: Automatically detects and manages both Brave and Chrome browsers
- **Profile discovery**: Automatically finds all browser profiles (Default, Profile 1-N, System Profile)
- **Search engine listing**: View shortcuts for individual profiles or combined across all profiles
- **Intelligent syncing**: Copies the most recent version of each shortcut to all profiles
- **Cross-browser compatibility**: Filters out browser-specific schemes (brave://, chrome://) to prevent conflicts
- **Safe deletion**: Remove shortcuts from all profiles with built-in protections
- **System validation**: Comprehensive smoke tests to ensure safe operation

### Safety Features

The tool includes multiple safety mechanisms to protect your data:

- **Process Detection**: Automatically detects if Brave or Chrome is running and refuses to proceed if either is
- **Permission Validation**: Checks write permissions before attempting any modifications
- **Built-in Search Engine Protection**: Prevents deletion of essential search engines (Google, DuckDuckGo, Brave Search)
- **Database Integrity**: Uses atomic transactions to prevent database corruption
- **Backup Reminders**: Consistently reminds users to backup their configurations
- **Comprehensive Testing**: The `--smoke-test` option validates system compatibility before use
- **Error handling**: Graceful handling of database locks and permission issues

## How It Works

### Database Structure

Both Brave and Chrome store search engines in SQLite databases located in each profile's `Web Data` file. The tool directly accesses the `keywords` table which contains:

- **short_name**: Display name (e.g., "Google", "DuckDuckGo")
- **keyword**: Shortcut keyword (e.g., ":g", ":d", "csr")  
- **url**: Search URL template with `{searchTerms}` placeholder
- **last_modified**: WebKit timestamp for version comparison
- **prepopulate_id**: Identifies built-in vs custom search engines

### Synchronization Logic

1. **Discovery**: Finds all Brave and Chrome profiles on the system
2. **Collection**: Reads search engines from each profile's database
3. **Filtering**: Excludes browser-specific schemes (brave://, chrome://) that don't work across browsers
4. **Comparison**: Identifies the most recent version of each shortcut keyword (case-insensitive)
5. **Synchronization**: Copies missing shortcuts to profiles that don't have them
6. **Validation**: Ensures case-insensitive keyword matching and prevents duplicates

The tool uses the `last_modified` timestamp to determine which version of a search engine is newest when the same keyword exists in multiple profiles.

**Note**: After synchronization, newly added search engines will appear in the "Other search engines" section of each profile's search settings (`chrome://settings/search` for Chrome or `brave://settings/search` for Brave). You need to visit this page in each profile to activate them for use.

## Installation

Clone this repository or download the standalone `chromium-search-sync` script. No further installation is required.

**Requirements:**

- Python 3.6+ (uses pathlib and f-strings)
- Standard library only (no external dependencies)
- Brave and/or Chrome browser installed and run at least once

**File Permissions:**

The script requires write permissions to the browsers' configuration directories to modify search engine databases. The script will check permissions during the smoke test and before making any changes.

**Making the Script Executable:**

```bash
chmod +x chromium-search-sync
```

**Alternative Execution:**

If you prefer not to make it executable, you can run it with Python directly:

```bash
python3 chromium-search-sync [options]
```

## Usage

### Command-Line Options

The script supports the following command-line options:

| Option | Long Form | Description |
|--------|-----------|-------------|
| `-p` | `--profiles` | List all Brave and Chrome browser profiles with their names and directories |
| `-s` | `--search-shortcuts` | Show search engine shortcuts for each profile |
| `-c` | `--combined` | Show combined search engines from all profiles, displaying all versions when shortcuts differ (excludes browser-specific schemes) |
| `-cs` | `--combined-sync` | Sync newest search engines to all profiles (case-insensitive keyword matching) - **CAUTION: Modifies browser configurations** |
| `-d SHORTCUT` | `--delete SHORTCUT` | Delete search engine with specified shortcut from all profiles - **CAUTION: Modifies browser configurations** |
| | `--smoke-test` | Run system validation checks to ensure the tool can operate safely |
| `-h` | `--help` | Show help message and exit |

### Basic Commands

**List all profiles:**

```bash
./chromium-search-sync -p
```

**Show search shortcuts for each profile:**

```bash
./chromium-search-sync -s
```

**Show combined search engines (dry run preview):**

```bash
./chromium-search-sync -c
```

**Sync newest search engines to all profiles:**

```bash
./chromium-search-sync -cs
```

**Delete a specific search engine shortcut:**

```bash
./chromium-search-sync -d :shortcut
```

**Run system validation checks:**

```bash
./chromium-search-sync --smoke-test
```

**Show help:**

```bash
./chromium-search-sync -h
```

### Example Workflow

1. **Check your profiles:**

   ```bash
   ./chromium-search-sync -p
   ```

2. **See what search engines you have:**

   ```bash
   ./chromium-search-sync -s
   ```

3. **View the combined list (most recent versions):**

   ```bash
   ./chromium-search-sync -c
   ```

4. **Sync everything:**

   ```bash
   ./chromium-search-sync -cs
   ```

5. **Activate new search engines:**
   After synchronization, you need to visit the search settings page in each profile to activate the newly added search engines:
   - Chrome: `chrome://settings/search`
   - Brave: `brave://settings/search`
   
   They will appear in the "Other search engines" section and can be activated by clicking on them.

## Important Notes

### Database Locking

- **Always close all browsers completely** before running this tool (both Brave and Chrome)
- The script will check for running browser processes and refuse to run if any are detected
- Database files are locked while browsers are running, preventing safe modifications

### Search Engine Activation

After running `-cs` (sync), newly added search engines will appear in the "Other search engines" section of each profile's search settings. You must visit the search settings page in each profile to activate them for use in the address bar:

- Chrome: `chrome://settings/search`
- Brave: `brave://settings/search`

### Case-Insensitive Matching

The tool uses case-insensitive keyword matching during synchronization to prevent duplicates. For example, `:G` and `:g` are considered the same shortcut.

### Browser-Specific Scheme Filtering

The tool automatically filters out browser-specific schemes during synchronization and when displaying combined results:

- **brave://** schemes (like bookmarks, history, settings) are excluded from Chrome profiles
- **chrome://** schemes are excluded from Brave profiles
- This prevents broken shortcuts and ensures only cross-browser compatible search engines are synced

Examples of filtered schemes:

- `@bookmarks` with `brave://bookmarks/` or `chrome://bookmarks/`
- `@history` with `brave://history/` or `chrome://history/`
- `@settings` with `brave://settings/` or `chrome://settings/`

These browser-specific shortcuts remain functional within their respective browsers but are not synced across different browser types.

### Built-in Search Engine Protection

The deletion feature (`-d`) includes built-in protections against removing essential search engines that have `prepopulate_id > 0` (like Google, DuckDuckGo, Brave Search).

## Design Philosophy

### Single-Purpose Functions

Each function has a clear, single responsibility:

- `detect_os()`: OS detection and path resolution
- `get_profile_names_from_local_state()`: Profile name extraction
- `get_search_shortcuts_with_metadata()`: Database reading
- `sync_search_engines_to_all_profiles()`: Synchronization logic

### Cross-Platform Compatibility

The tool centralizes OS-specific logic in `detect_os()` and uses `pathlib` for cross-platform path handling, supporting:

- **macOS**:
  - Brave: `~/Library/Application Support/BraveSoftware/Brave-Browser`
  - Chrome: `~/Library/Application Support/Google/Chrome`
- **Linux**:
  - Brave: `~/.config/BraveSoftware/Brave-Browser`
  - Chrome: `~/.config/google-chrome`
- **Windows (Cygwin required)**:
  - Brave: `~/AppData/Local/BraveSoftware/Brave-Browser`
  - Chrome: `~/AppData/Local/Google/Chrome/User Data`

### Error Handling

Comprehensive error handling ensures the tool degrades gracefully:

- Database connection failures
- Permission errors
- Missing files or directories
- Process conflicts

### Safety First

Multiple safety mechanisms protect user data:

- Process detection prevents database corruption
- Built-in search engine protection
- Backup reminders
- Comprehensive validation through smoke tests

## Technical Details

### Database Schema

The tool works with Brave's `keywords` table structure, understanding:

- WebKit timestamp format (microseconds since January 1, 1601)
- Search URL templates with `{searchTerms}` placeholders
- Built-in vs custom search engine identification via `prepopulate_id`
- Case-insensitive keyword matching
- Common `prepopulate_id` values: 1 (Google), 501 (DuckDuckGo), 550 (Brave Search)

### Profile Discovery

Intelligently discovers profiles through:

1. **Primary source**: Local State file for profile names
2. **Fallback**: Individual profile Preferences files
3. **Directory scanning**: Finds Default, Profile 1-N, and System Profile directories

### Process Detection

The tool uses the `ps` command to detect running browser processes (both Brave and Chrome) and will refuse to run if any are active. This prevents database corruption and ensures safe operation.

### Smoke Test Features

The `--smoke-test` option performs comprehensive validation:

- OS detection and path validation
- Browser directory structure verification
- Profile discovery testing
- SQLite database schema validation
- Write permission checks
- Command-line tool availability

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.

## Warnings

⚠️ **Always backup your browser configurations before using this tool.**

⚠️ **Close all browsers completely before running this tool.**

⚠️ **Use at your own risk - this tool modifies browser configuration files.**

⚠️ **The tool will check for running browser processes and refuse to run if any are detected.**

## Backup Locations

- **macOS**:
  - Brave: `~/Library/Application Support/BraveSoftware/Brave-Browser`
  - Chrome: `~/Library/Application Support/Google/Chrome`
- **Linux**:
  - Brave: `~/.config/BraveSoftware/Brave-Browser`
  - Chrome: `~/.config/google-chrome`
- **Windows (Cygwin required)**:
  - Brave: `~/AppData/Local/BraveSoftware/Brave-Browser`
  - Chrome: `~/AppData/Local/Google/Chrome/User Data`

## Troubleshooting

### "Browser is currently running" Error

If you get this error, make sure to:

1. Quit all browsers normally (Cmd+Q on macOS, Ctrl+Q on Linux/Windows)
2. Wait a few seconds for all processes to terminate
3. Or use: `pkill -f brave` or `pkill -f chrome` to force quit
4. Run the script again

### Permission Denied Errors

Run the smoke test first to check permissions:

```bash
./chromium-search-sync --smoke-test
```

Ensure you have write permissions to the browser configuration directories.

### Database Locked Errors

This usually means a browser is still running. Make sure all browser processes are closed before running the tool.

### No Profiles Found

Make sure:

1. Brave and/or Chrome is installed and has been run at least once
2. You have the correct browser installation (not Beta or Nightly versions)
3. Your OS is supported (macOS, Linux, Windows)

## Contributing

Feel free to submit issues, feature requests, or pull requests on the [GitHub repository](https://github.com/mcgroarty/chromium-search-sync).
