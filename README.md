# brave-search-sync

A Python utility for managing and synchronizing search engines across Brave Browser profiles.

**Project Home**: [https://github.com/mcgroarty/brave-search-sync](https://github.com/mcgroarty/brave-search-sync)

## Current Status

Mostly AI-coded, but humam-reviewed. Tested on macOS. I'll check Linux and Windows soon. Open an issue if you test before me and it fails.

## TL;DR: Too Long, Didn't Read. Twitter broke my attention span.

This tool syncs custom search engine shortcuts across all your Brave browser profiles. Close Brave, run `./brave-search-sync -cs` to sync everything, then reopen Brave and visit `brave://settings/search` in each profile to activate the newly added search engines. Use `-c` on its own for a dry run preview. It's good to backup your Brave config first, just in case. The brave config dir location is shown in `-h`. Run `./brave-search-sync --smoke-test` to check if your system is ready.

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

- **Cross-platform support**: Works on macOS, Linux, Windows, and Windows/Cygwin
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
3. **Comparison**: Identifies the most recent version of each shortcut keyword
4. **Synchronization**: Copies missing shortcuts to profiles that don't have them
5. **Validation**: Ensures case-insensitive keyword matching and prevents duplicates

**Note**: After synchronization, newly added search engines will appear in the "Other search engines" section of each profile's search settings (`brave://settings/search`). You need to visit this page in each profile to activate them for use.

## Installation

No installation required - this is a standalone Python script.

**Requirements:**

- Python 3.6+ (uses pathlib and f-strings)
- Standard library only (no external dependencies)
- Brave browser installed and run at least once

## Usage

### Basic Commands

```bash
# List all Brave browser profiles
./brave-search-sync -p

# Show search engine shortcuts for each profile
./brave-search-sync -s

# Show combined search engines from all profiles
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
- **Windows**: `~/AppData/Local/BraveSoftware/Brave-Browser`

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
- Built-in vs custom search engine identification
- Case-insensitive keyword matching

### Profile Discovery

Intelligently discovers profiles through:

1. **Primary source**: Local State file for profile names
2. **Fallback**: Individual profile Preferences files
3. **Directory scanning**: Finds Default, Profile 1-N, and System Profile directories

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.

## Warnings

⚠️ **Always backup your Brave browser configuration before using this tool.**

⚠️ **Close Brave completely before running this tool.**

⚠️ **Use at your own risk - this tool modifies Brave's configuration files.**

## Backup Locations

- **macOS**: `~/Library/Application Support/BraveSoftware/Brave-Browser`
- **Linux**: `~/.config/BraveSoftware/Brave-Browser`  
- **Windows**: `~/AppData/Local/BraveSoftware/Brave-Browser`
