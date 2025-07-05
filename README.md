# brave-search-sync

A Python utility for managing and synchronizing search engines across Brave Browser profiles.

**Project Home**: [https://github.com/mcgroarty/brave-search-sync](https://github.com/mcgroarty/brave-search-sync)

## TL;DR: Too Long, Didn't Read. Twitter broke my attention span.

This tool syncs custom search engine shortcuts across all your Brave browser profiles. Close Brave, run `./brave-search-sync -cs` to sync everything, then reopen Brave and visit `brave://settings/search` in each profile to activate the newly added search engines. Use `-c` on its own for a dry run preview. It's good to backup your Brave config first, just in case. The brave config dir location is shown in `-h`. Run `./brave-search-sync --smoke-test` to check if your system is ready.

## Current Status

Tested on macOS and Linux. Windows support requires running this script from the Cygwin environment. Open an issue if you test before me and it fails.

Include the --smoke-test option in [bug reports](https://github.com/mcgroarty/brave-search-sync/issues).

Mostly AI-coded, but human-reviewed.

## Overview

**brave-search-sync** is a cross-platform command-line tool that helps you manage custom search engines across multiple Brave Browser profiles. If you use multiple profiles and want to keep your search engine shortcuts synchronized, this tool automates the process by reading from and writing to Brave's SQLite databases.

## Purpose

When you have multiple Brave Browser profiles (personal, work, hobby, etc.), manually keeping custom search engine shortcuts in sync becomes tedious. This tool solves that problem by:

- **Discovering all profiles** automatically across different operating systems
- **Reading search engines** from each profile's database
- **Synchronizing shortcuts** by copying the newest version to all profiles
- **Managing shortcuts** through command-line operations

## Key Features

### Core Functionality

- **Cross-platform support**: Works on macOS, Linux, and Windows via Cygwin (Cygwin required on Windows)
- **Profile discovery**: Automatically finds all Brave profiles (Default, Profile 1-N, System Profile)
- **Search engine listing**: View shortcuts for individual profiles or combined across all profiles
- **Intelligent syncing**: Copies the most recent version of each shortcut to all profiles
- **Safe deletion**: Remove shortcuts from all profiles with built-in protections
- **System validation**: Comprehensive smoke tests to ensure safe operation

### Safety Features

- **Process detection**: Prevents running while Brave is active to avoid database corruption
- **Backup warnings**: Reminds users to backup their Brave configuration
- **Built-in protections**: Prevents deletion of essential built-in search engines
- **Write permission checks**: Validates file system access before making changes
- **Error handling**: Graceful handling of database locks and permission issues

## How It Works

### Database Structure

Brave stores search engines in SQLite databases located in each profile's `Web Data` file. The tool directly accesses the `keywords` table which contains:

- **short_name**: Display name (e.g., "Google", "DuckDuckGo")
- **keyword**: Shortcut keyword (e.g., ":g", ":d", "csr")  
- **url**: Search URL template with `{searchTerms}` placeholder
- **last_modified**: WebKit timestamp for version comparison
- **prepopulate_id**: Identifies built-in vs custom search engines

### Synchronization Logic

1. **Discovery**: Finds all Brave profiles on the system
2. **Collection**: Reads search engines from each profile's database
3. **Comparison**: Identifies the most recent version of each shortcut keyword (case-insensitive)
4. **Synchronization**: Copies missing shortcuts to profiles that don't have them
5. **Validation**: Ensures case-insensitive keyword matching and prevents duplicates

The tool uses the `last_modified` timestamp to determine which version of a search engine is newest when the same keyword exists in multiple profiles.

**Note**: After synchronization, newly added search engines will appear in the "Other search engines" section of each profile's search settings (`brave://settings/search`). You need to visit this page in each profile to activate them for use.

## Installation

No installation required - this is a standalone Python script.

**Requirements:**

- Python 3.6+ (uses pathlib and f-strings)
- Standard library only (no external dependencies)
- Brave browser installed and run at least once

**File Permissions:**

The script requires write permissions to Brave's configuration directory to modify search engine databases. The script will check permissions during the smoke test and before making any changes.

**Making the Script Executable:**

```bash
chmod +x brave-search-sync
```

**Alternative Execution:**

If you prefer not to make it executable, you can run it with Python directly:

```bash
python3 brave-search-sync [options]
```

## Usage

### Basic Commands

```bash
# List all Brave browser profiles
./brave-search-sync -p

# Show search engine shortcuts for each profile
./brave-search-sync -s

# Show combined search engines from all profiles (most recent version of each shortcut)
./brave-search-sync -c

# Sync newest search engines to all profiles
./brave-search-sync -cs

# Delete a search engine shortcut from all profiles
./brave-search-sync -d :shortcut

# Run system validation checks
./brave-search-sync --smoke-test
```

### Example Workflow

1. **Check your profiles:**

   ```bash
   ./brave-search-sync -p
   ```

2. **See what search engines you have:**

   ```bash
   ./brave-search-sync -s
   ```

3. **View the combined list (most recent versions):**

   ```bash
   ./brave-search-sync -c
   ```

4. **Sync everything:**

   ```bash
   ./brave-search-sync -cs
   ```

5. **Activate new search engines:**
   After synchronization, you need to visit `brave://settings/search` in each profile to activate the newly added search engines. They will appear in the "Other search engines" section and can be activated by clicking on them.

## Important Notes

### Database Locking

- **Always close Brave completely** before running this tool
- The script will check for running Brave processes and refuse to run if any are detected
- Database files are locked while Brave is running, preventing safe modifications

### Search Engine Activation

After running `-cs` (sync), newly added search engines will appear in the "Other search engines" section of each profile's search settings. You must visit `brave://settings/search` in each profile to activate them for use in the address bar.

### Case-Insensitive Matching

The tool uses case-insensitive keyword matching during synchronization to prevent duplicates. For example, `:G` and `:g` are considered the same shortcut.

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

- **macOS**: `~/Library/Application Support/BraveSoftware/Brave-Browser`
- **Linux**: `~/.config/BraveSoftware/Brave-Browser`
- **Windows (Cygwin required)**: `~/AppData/Local/BraveSoftware/Brave-Browser`

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

The tool uses the `ps` command to detect running Brave processes and will refuse to run if Brave is active. This prevents database corruption and ensures safe operation.

### Smoke Test Features

The `--smoke-test` option performs comprehensive validation:

- OS detection and path validation
- Brave directory structure verification
- Profile discovery testing
- SQLite database schema validation
- Write permission checks
- Command-line tool availability

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.

## Warnings

⚠️ **Always backup your Brave browser configuration before using this tool.**

⚠️ **Close Brave completely before running this tool.**

⚠️ **Use at your own risk - this tool modifies Brave's configuration files.**

⚠️ **The tool will check for running Brave processes and refuse to run if any are detected.**

## Backup Locations

- **macOS**: `~/Library/Application Support/BraveSoftware/Brave-Browser`
- **Linux**: `~/.config/BraveSoftware/Brave-Browser`  
- **Windows (Cygwin required)**: `~/AppData/Local/BraveSoftware/Brave-Browser`

## Troubleshooting

### "Brave Browser is currently running" Error

If you get this error, make sure to:

1. Quit Brave normally (Cmd+Q on macOS, Ctrl+Q on Linux/Windows)
2. Wait a few seconds for all processes to terminate
3. Or use: `pkill -f brave` to force quit
4. Run the script again

### Permission Denied Errors

Run the smoke test first to check permissions:

```bash
./brave-search-sync --smoke-test
```

Ensure you have write permissions to the Brave configuration directory.

### Database Locked Errors

This usually means Brave is still running. Make sure all Brave processes are closed before running the tool.

### No Profiles Found

Make sure:

1. Brave is installed and has been run at least once
2. You have the correct Brave installation (not Brave Beta or Brave Nightly)
3. Your OS is supported (macOS, Linux, Windows)
