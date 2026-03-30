---
name: stundenzettel
description: |
  Fill the SUB-events (SDE) timesheet XLSX from the clawd time tracker.
  Pulls latest hours from git, runs fill_stundenzettel.py, generates the monthly XLSX.
  Use when: user says 'stundenzettel', 'timesheet', 'fill timesheet', 'generate stundenzettel',
  'fill hours', 'work hours', 'sde hours', 'sde timesheet', 'monthly hours', 'stundenzettel ausfüllen'.
  Triggers: 'stundenzettel', 'timesheet', 'fill timesheet', 'sde hours', 'work hours',
  'monthly hours', 'stundenzettel ausfüllen', 'generate stundenzettel'.
---

# Stundenzettel (SDE Timesheet)

Fill the SUB-events GmbH timesheet XLSX from clawd's time tracker markdown.

## Steps

### 1. Sync tracker data to local clone

```bash
cd /root/sde-timetracker && cp sde-time-tracker.md clawd/sde-time-tracker.md
```

### 2. Run the fill script

Default (last complete month):
```bash
cd /root/sde-timetracker && .venv/bin/python3 fill_stundenzettel.py
```

Specific month:
```bash
cd /root/sde-timetracker && .venv/bin/python3 fill_stundenzettel.py --month YYYY-MM
```

Dry run (preview):
```bash
cd /root/sde-timetracker && .venv/bin/python3 fill_stundenzettel.py --month YYYY-MM --dry-run
```

### 3. Push results to GitHub

```bash
cd /root/sde-timetracker && git add -A && git commit -m 'Generate Stundenzettel YYYY-MM' && git push
```

### 4. Report results

Tell the user:
- How many sessions were found
- Any skipped incomplete sessions (user may need to close them first)
- The output file path in the repo

## Key Details

- Script reads `clawd/sde-time-tracker.md` (symlinked copy) and writes to the latest `Stundenzettel-*-Merlin.xlsx`
- Template: `template-stundenzettel.xlsx` — copy to `2026/Stundenzettel-{Tab}-Merlin.xlsx` before first run of a new month
- Multiple sessions on the same day get separate rows (duplicate date rows)
- Sessions with no end time or hours="-" are skipped (incomplete/cancelled)
- Output: `2026/Stundenzettel-{MonthDE}{YY}-Merlin.xlsx` (German month abbreviations: Jan, Feb, Mär, Apr, Mai, Jun, Jul, Aug, Sep, Okt, Nov, Dez)
- Pay: 556€/month, 18.50€/h, 30h cap
- Überstunden (overtime) tab is updated automatically

## Setup (one-time, done)

```bash
cd /root/sde-timetracker
python3 -m venv .venv && .venv/bin/pip install openpyxl
mkdir -p clawd  # symlink for tracker data
```

## If the script fails

- No XLSX found: copy template → `cp template-stundenzettel.xlsx 2026/Stundenzettel-{Tab}-Merlin.xlsx`
- Month not in markdown: user needs to log hours via tracker first
- Missing venv: `cd /root/sde-timetracker && python3 -m venv .venv && .venv/bin/pip install openpyxl`
