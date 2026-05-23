---
name: sde-tracker
description: |
  Edit, query, and analyze the SDE (Skydive Events / SUB-events GmbH) time tracker at github.com/merlinkraemer/sde-timetracker (sde-time-tracker.md).
  Use when the user mentions: sde, skydive, sub-events, log a session, time tracker, hours this month, overtime, banked hours, stundenzettel hours, work session, or quick-log syntax like "sde 14:00-18:30".
  Companion skill `stundenzettel` handles monthly XLSX filling — this skill handles the markdown tracker itself.
---

# SDE Time Tracker Skill

Operating manual for the SDE work-hours tracker.

## File location

- **Repo:** `github.com/merlinkraemer/sde-timetracker`
- **Local clone:** `/Users/merlinkraemer/local/2-Areas/SAV/SubEvents/Stundenzettel/clawd`
- **Data file:** `sde-time-tracker.md` in repo root
- **Companion files:** `stundenzettel-skill.md` (XLSX-fill skill, separate concern)

Also edited by **openclaw** via Telegram (topic: SDE time tracking). GitHub is the single source of truth — both systems pull before editing and push after.

## Ground rule: GitHub is truth

Before any edit, **always**:

```bash
cd /Users/merlinkraemer/local/2-Areas/SAV/SubEvents/Stundenzettel/clawd
git pull --rebase
```

If `git pull --rebase` produces a conflict: **stop**. Tell the user, show the conflicting hunks, ask how to resolve. Never auto-resolve — openclaw may have just pushed a session from Telegram.

After every edit:

```bash
git add sde-time-tracker.md
git commit -m "<see commit conventions below>"
git push origin main
```

If `git push` fails: tell the user immediately. Don't leave local commits unpushed.

## Default behavior

- **Silent confirmation** for logging: pull → edit → commit → push → reply one line: `Logged. <date> <start>-<end> (<hours>). ✓ pushed.`
- **Auto-recompute month totals** on every write (Month Total, Month Overtime, Cumulative).
- **Never commit without pushing.** Each edit = one full pull-edit-commit-push cycle.
- **Read-only queries** still `git pull --rebase` first so you don't answer from stale data.

## Quick-log syntax

```
sde <start>-<end> [- note]
sde <start>-<end> <date> [- note]
```

Examples:
- `sde 14:00-18:30` → today, 14:00–18:30, computed hours, no note
- `sde 09:00-13:15 - deployed auth fix` → today, with note
- `sde 10:00-16:00 2026-05-20 - backfill` → specific date
- `start sde` → opens an incomplete session (End = "-")
- `stop sde - deployed X` → closes today's open session, sets End to now, adds note

### Date handling

- Default date: today (use the actual current date, not a guess).
- Sessions crossing midnight: End < Start → append `*` to End (e.g., `01:32*`) and add 24h when computing hours.

### Hours format

- Decimal hours with `h` suffix: `4h`, `6.37h`, `9.5h`.
- Round to 2 decimals.
- Incomplete sessions: `-` in the Hours column.

## Tracker format (don't drift)

`sde-time-tracker.md` structure:

```markdown
# SDE Time Tracker

[intro / usage]

## Cumulative Overtime Balance
**As of <Month YYYY>: <X>h banked**

## <Month YYYY>

| Date | Start | End | Hours | Note |
|------|-------|-----|-------|------|
| YYYY-MM-DD | HH:MM | HH:MM | X.YYh | optional note |
...

**Month Total:** <sum>h / 30h cap
**Month Overtime:** <total - 30>h (only if > 30)
**Cumulative:** <prior cumulative + this month's overtime>h

## Notes
...
```

**Preserve this structure.** Don't rename headers, don't reorder, don't remove the totals lines.

## When adding a session

1. `git pull --rebase`
2. Find the right `## <Month YYYY>` section. If the month doesn't exist yet, **create it** in chronological order (before `## Notes`).
3. Insert the row in date order within that month's table.
4. Compute hours: `(end - start) % 24`, round to 2 decimals. Use `*` on end time if it crossed midnight.
5. Recompute that month's `Month Total`, `Month Overtime` (max(0, total - 30)), and `Cumulative`.
6. If this is the first session of a new month, update prior month's `Cumulative` only if it was wrong — otherwise leave history alone.
7. Commit + push.

## When closing an incomplete session (`stop sde`)

1. Find the row with `End = -` and `Hours = -` for today (or most recent if none today — confirm with user first).
2. Set End to current time.
3. Compute and write Hours.
4. Append note if provided.
5. Recompute month totals.
6. Commit + push.

## When fixing a session

Ask the user to identify the row by date + start time. Then:
- Edit in place; don't delete and re-add (preserves git blame and openclaw audit trail).
- Recompute month totals.
- Commit with message describing what changed.

## Read-only queries

- `hours today` → sum today's session hours.
- `hours this month` / `month total` → read the `Month Total` line.
- `overtime` / `banked` → read the latest `Cumulative` value.
- `last session` → show the most recent row.
- `open session` / `am i tracking` → find any row with End = `-`.

All read-only ops: `git pull --rebase` first to ensure fresh data, then read, then reply. No commit/push.

## Cumulative overtime tracking

- 30h/month cap → 556€ / 18.50€/h.
- Anything over 30h in a month adds to the cumulative bank.
- The `## Cumulative Overtime Balance` header at the top shows the **rolling total as of the last fully-counted month**. Update it when a month closes (defer to monthly XLSX-fill workflow if unsure).
- Per-month `Cumulative:` lines show the running total *after* that month.

## Commit message conventions

Match the existing openclaw style (prose, not conventional commits):

- Single session: `Add session <MonDD> <HH:MM>-<HH:MM> (<X.Yh>) - "<original user request>"`
- Multiple sessions in one go: `Add sessions <MonDD> (<X.Yh> + <Y.Zh>) - "<original user request>"`
- Close incomplete session: `Close session <MonDD> <HH:MM>-<HH:MM> (<X.Yh>)`
- Fix: `Fix session <MonDD>: <what changed>`
- New month: include `(new month <Month YYYY>)` in the message.

When Claude initiated the log (not Telegram), the trailing quoted user request is optional — use a brief description instead.

## When to ask, don't guess

Stop and ask before:
- Closing an open session if there are multiple candidates or it's from a different day.
- Adding a session that would push the month past 30h without acknowledgement (mention it: "this puts April at 34.5h — adds 4.5h to overtime bank").
- Editing or deleting a session from a closed/already-billed month (months that have already been written to XLSX).
- Resolving a git merge conflict (always stop).
- Restructuring tracker sections.

## Never do

- Silently change a number.
- Reorder rows within a month (date order only).
- Delete a session — fix in place, or mark with a note.
- Push without pulling first.
- Commit without pushing.
- Force-push, amend pushed commits, or skip hooks.
- Modify months that have already been filled into XLSX timesheets unless explicitly asked.

## Cross-skill handoff

For monthly **XLSX timesheet generation** (running `fill_stundenzettel.py`, committing the XLSX), defer to the `stundenzettel` skill. This skill only manages `sde-time-tracker.md`.

## Special commands

| User says | You do |
|-----------|--------|
| `sde <start>-<end> [- note]` | Quick-log a session (today). Pull → add → totals → push. |
| `start sde` | Open incomplete session (now → `-`). Pull → add → push. |
| `stop sde [- note]` | Close today's open session. Pull → close → totals → push. |
| `hours today` | Pull, sum today's rows, reply one line. |
| `month total` / `hours this month` | Pull, read `Month Total`, reply one line. |
| `overtime` / `banked` | Pull, read latest `Cumulative`, reply one line. |
| `last session` | Pull, show most recent row. |
| `am i tracking` | Pull, check for any End = `-` row. |
| Anything ambiguous | Ask one clarifying question, then proceed. |
