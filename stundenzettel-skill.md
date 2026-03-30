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

### 1. Pull latest time data

```bash
cd /Users/merlinkramer/local/2-Areas/SAV/SubEvents/Stundenzettel/clawd && git pull
```

### 2. Run the fill script

Default (last complete month):
```bash
cd /Users/merlinkramer/local/2-Areas/SAV/SubEvents/Stundenzettel && .venv/bin/python3 fill_stundenzettel.py
```

Specific month:
```bash
cd /Users/merlinkramer/local/2-Areas/SAV/SubEvents/Stundenzettel && .venv/bin/python3 fill_stundenzettel.py --month YYYY-MM
```

Dry run (preview):
```bash
cd /Users/merlinkramer/local/2-Areas/SAV/SubEvents/Stundenzettel && .venv/bin/python3 fill_stundenzettel.py --month YYYY-MM --dry-run
```

### 3. Report results

Tell the user:
- How many sessions were found
- Any skipped incomplete sessions (user may need to close them in Clawd first)
- The output file path

## Key Details

- Script reads `clawd/sde-time-tracker.md` and writes to the latest `Stundenzettel-*-Merlin.xlsx`
- Multiple sessions on the same day get separate rows (duplicate date rows)
- Sessions with no end time or hours="-" are skipped (incomplete/cancelled)
- Output: `2026/Stundenzettel-{MonthDE}{YY}-Merlin.xlsx` (German month abbreviations: Jan, Feb, Mär, Apr, Mai, Jun, Jul, Aug, Sep, Okt, Nov, Dez)
- Pay: 556€/month, 18.50€/h, 30h cap
- Überstunden (overtime) tab is updated automatically

## If the script fails

- Missing venv: `cd /Users/merlinkramer/local/2-Areas/SAV/SubEvents/Stundenzettel && python3 -m venv .venv && .venv/bin/pip install openpyxl`
- No XLSX found: check `2026/` directory for existing Stundenzettel file
- Month not in markdown: user needs to log hours via Clawd first, or the month header format changed
