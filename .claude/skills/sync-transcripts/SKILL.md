# Sync Transcripts

Pull recent call transcripts from Google Drive and store them locally. Compares what exists in the Drive folder against local files and only downloads new transcripts.

## Trigger

User invokes `/sync-transcripts` with no arguments.

## Context Consumed

- Local transcript files in `customer-intelligence/transcripts/` (filenames only, to detect duplicates)
- During the extract-insights step: `strategy/icp.md`, `strategy/personas.md`, `strategy/competitive-landscape.md` (loaded per-transcript by the extract-insights skill)

## MCP Required

This skill requires the **Google Drive** MCP server to be connected. If the MCP tools (`mcp__claude_ai_Google_Drive__search_files`, `mcp__claude_ai_Google_Drive__read_file_content`) are not available, tell the user:

> The Google Drive MCP server is not connected. Connect it in Claude Code settings to use this skill.

## Configuration

- **Google Drive Folder ID:** `1Op5gBFQXRrRXFxZDe1YKL3P3hfvxcL5d`
- **Local destination:** `customer-intelligence/transcripts/`

## Workflow

### Step 1: List Drive folder contents

Search for all files in the configured folder:

```
mcp__claude_ai_Google_Drive__search_files
  query: parentId = '1Op5gBFQXRrRXFxZDe1YKL3P3hfvxcL5d'
  pageSize: 50
  excludeContentSnippets: true
```

If a `nextPageToken` is returned, paginate until all files are retrieved.

### Step 2: List existing local transcripts

Use the Glob tool to list all `.md` files in `customer-intelligence/transcripts/`.

### Step 3: Classify each Drive file

For each Drive file, apply the classification logic from `.claude/skills/classify-transcript/SKILL.md` to determine if it is a sales call:

1. **Title-based fast path**: Check the file title and size against auto-skip and auto-include patterns defined in the classify-transcript skill.
2. **Content-based fallback**: If the title is ambiguous, read the first 50 lines of the file content via `mcp__claude_ai_Google_Drive__read_file_content` and apply the content-based classification signals.
3. **Result**: Each file gets one of: `sales_call`, `not_sales_call`, or `uncertain`.

Files classified as `not_sales_call` are excluded from sync. Files classified as `uncertain` are included but flagged for the user to review.

### Step 4: Parse metadata for sales calls and match against local files

For each file classified as `sales_call` or `uncertain`, extract:
- **Date**: from the title pattern. Titles typically follow formats like:
  - `{Company} and Quotivity {type} - {YYYY/MM/DD HH:MM TZ} - Transcript`
  - `Quotivity {type} for {Person} @ {Company} - {YYYY/MM/DD HH:MM TZ} - Transcript`
  - `{Company} {type} - {YYYY/MM/DD HH:MM TZ} - Transcript`
  - Extract the date portion (`YYYY/MM/DD`) and convert to `YYYY-MM-DD`
- **Company slug**: extract the company name from the title (the non-Quotivity company), lowercase it, replace spaces with hyphens, remove suffixes like "Inc", "LLC", etc.

The local filename convention is: `{YYYY-MM-DD}-{company-slug}.md`

Compare each file against existing local filenames. A file is considered **already synced** if a local file exists with the same date and company slug.

### Step 5: Present summary and confirm

Present the user with a summary:
- Total files in Drive folder
- Skipped (not sales calls) — list with reasons
- Already synced locally
- New sales call transcripts to download
- Uncertain transcripts (if any) — listed separately for user review

Ask the user to confirm before downloading. If no new transcripts are found, tell the user everything is up to date and stop.

### Step 6: Download and save new transcripts

For each confirmed new transcript:

1. Read the file content using `mcp__claude_ai_Google_Drive__read_file_content` with the file's `id`.
2. Save the content to `customer-intelligence/transcripts/{YYYY-MM-DD}-{company-slug}.md` using the Write tool.
3. Preserve the transcript text as-is. Do not reformat, summarize, or edit the content.

### Step 7: Extract insights from new transcripts

After all downloads complete, run `/extract-insights` on each newly downloaded transcript. Process them one at a time in sequence:

1. For each new transcript file, invoke the extract-insights skill with the file path (e.g., `customer-intelligence/transcripts/2026-05-06-acme.md`).
2. Wait for each extraction to complete before starting the next one.
3. Track which transcripts were successfully processed and which failed.

If a transcript fails insight extraction, log the error and continue with the remaining transcripts. Do not abort the entire batch.

### Step 8: Report results

After all downloads and extractions complete, report:
- Number of transcripts downloaded
- Number of transcripts skipped (not sales calls)
- List of new local filenames created
- Number of transcripts with insights extracted successfully
- Any extraction failures (with file paths and error summaries)

## Filename Slug Rules

When generating the company slug from the Drive file title:

1. Identify the company name (the non-Quotivity entity in the title)
2. Remove corporate suffixes: Inc, LLC, Ltd, Corp, Co, GmbH, AG, BV
3. Remove filler words from the title context: "and", "for", "with", "w/"
4. Lowercase the company name
5. Replace spaces with hyphens
6. Remove special characters except hyphens
7. Collapse multiple hyphens into one

Examples:
- "Air Cleaning Specialists Inc and Quotivity Follow Up" -> `air-cleaning-specialists`
- "Quotivity Introduction for Victor Su @ Qblox" -> `qblox`
- "Gov Pilot and Quotivity Sync" -> `govpilot` (note: match existing local convention if a prior file exists)
- "Kneat Requirements Review" -> `kneat`
- "Quotivity Introduction for Mike Collister @ Mike Collister" -> `mike-collister`

**Important:** When a company has been synced before (an older transcript exists locally), match the existing slug exactly. Check local filenames for prior entries with the same company name to ensure consistency.

## Edge Cases

- **Duplicate timestamps**: If two calls happen on the same date with the same company, append a counter: `{date}-{slug}-2.md`
- **Very large transcripts**: The Google Drive `read_file_content` tool may truncate very large files. If content appears truncated, note this to the user and suggest they export the file manually.
