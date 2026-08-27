# TPM Bay Area Job Search Automation

Four scheduled tasks (skills) for [Claude Cowork](https://claude.com) that automate a senior/exec TPM job search: a daily job-board run, a morning planner, application follow-up nudges, and automatic interview prep. Fork it, fill in the placeholders, and adapt it to your own search.

# What's changed
Latest revision: 2026-08-27, tuned for Claude Opus 5. Hardened against ~2 months of daily production runs — every rule in here came from something that actually broke. See `CHANGELOG.md`. The headline changes: the tracker moved from Sheets to Notion, an append-only dedup ledger was added, geography became a hard drop keyed on work arrangement rather than headquarters, and rejected reqs now block at req level rather than company level.

## What's included

| Skill | Schedule | What it does | Last Revised |
|---|---|---|---|
| `skills/daily-job-run` | Weekdays 8am | Searches job boards, ranks roles against your criteria, drafts tailored resume/cover-letter material and outreach notes, updates your tracker | 2026-08-27 (Opus 5)|
| `skills/morning-planner` | Daily 6:30am | Builds a daily planner (calendar, job-search to-dos, interview prep, personal tasks) as a Gmail draft | unchanged| 
| `skills/application-followup-nudges` | Mon & Thu 8am | Finds stale applications in your tracker and drafts friendly follow-up notes | unchanged — but still references the Google Sheet tracker | 
| `skills/interview-prep-autoprep` | Daily 5pm | If an interview is on tomorrow's calendar, builds a tailored prep packet (with a technical mock when relevant) | unchanged | 

All four are **drafts-only by design**: they never apply, send email, or message anyone on your behalf.

## Requirements

- Claude Cowork (desktop app) with scheduled tasks
- Connectors: Gmail (drafts), Google Calendar, and Notion (personal tasks)
- A job-application tracker as a Notion database (property layout described in `skills/daily-job-run/SKILL.md`)
- An append-only dedup ledger (`ledger.jsonl` + `ledger-index.json`) in your `Context/` folder — created automatically on first run
- Claude in Chrome extension, logged into your LinkedIn and your job aggregator of choice
- A `Context/` folder with the files in `templates/` filled out

## Setup

1. Copy the files in `templates/` into a `Context/` folder somewhere Claude can read, and fill them in (who you are, target roles, comp targets, companies to avoid, writing voice).
2. Create your tracker Notion database with the properties listed in `skills/daily-job-run/SKILL.md`. Note that `Phase` is a formula property derived from `Status` — never set it by hand.
3. In each `SKILL.md`, replace the `{PLACEHOLDER}` values: `{CONTEXT_DIR}`, `{PROJECTS_DIR}`, `{YOUR_EMAIL}`, `{TRACKER_SHEET_ID}`, `{TRACKER_TAB_GID}`, `{NOTION_TASKS_DB_ID}` (optional), resume paths, timezone, and target locations/levels.
4. In Cowork, ask Claude to create a scheduled task from each `SKILL.md` with the cron schedule noted in the file.

## Placeholders used

- `{YOUR_NAME}`, `{YOUR_EMAIL}` — you
- `{CONTEXT_DIR}` — folder holding your context files (about-me, current-focus, work-preferences, writing-voice)
- `{PROJECTS_DIR}` — folder where output files (run files, planners, prep packets) get saved
- `{TRACKER_DB_ID}` — your Notion tracker database
- `{TARGET_METRO_1}`, `{TARGET_METRO_2}` — metros where onsite/hybrid roles are acceptable
- `{BASE_MINIMUM}` — base-salary floor, separate from total comp
- `{TARGET_LEVEL}` — e.g. "Principal/Director+ TPM"
- `{NO_LIST}` — companies excluded at company level
- `{NOTION_TASKS_DB_ID}` — optional Notion tasks database
- `{COMP_TARGET}`, `{COMP_FLOOR}` — your total-comp target and walk-away floor
- `{LEADERSHIP_RESUME_PATH}`, `{IC_RESUME_PATH}` — resume variants
- `{TIMEZONE}` — e.g., America/Los_Angeles

## License

MIT — use it however you like.
