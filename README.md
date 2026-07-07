# TPM Bay Area Job Search Automation

Four scheduled tasks (skills) for [Claude Cowork](https://claude.com) that automate a senior/exec TPM job search: a daily job-board run, a morning planner, application follow-up nudges, and automatic interview prep. Fork it, fill in the placeholders, and adapt it to your own search.

## What's included

| Skill | Schedule | What it does |
|---|---|---|
| `skills/daily-job-run` | Weekdays 8am | Searches job boards, ranks roles against your criteria, drafts tailored resume/cover-letter material and outreach notes, updates your tracker |
| `skills/morning-planner` | Daily 6:30am | Builds a daily planner (calendar, job-search to-dos, interview prep, personal tasks) as a Gmail draft |
| `skills/application-followup-nudges` | Mon & Thu 8am | Finds stale applications in your tracker and drafts friendly follow-up notes |
| `skills/interview-prep-autoprep` | Daily 5pm | If an interview is on tomorrow's calendar, builds a tailored prep packet (with a technical mock when relevant) |

All four are **drafts-only by design**: they never apply, send email, or message anyone on your behalf.

## Requirements

- Claude Cowork (desktop app) with scheduled tasks
- Claude in Chrome extension, logged into LinkedIn and Google Sheets
- Connectors: Gmail (drafts), Google Calendar, and optionally Notion (personal tasks)
- A job-application tracker as a Google Sheet (column layout described in each skill)
- A `Context/` folder with the files in `templates/` filled out

## Setup

1. Copy the files in `templates/` into a `Context/` folder somewhere Claude can read, and fill them in (who you are, target roles, comp targets, companies to avoid, writing voice).
2. Create your tracker Google Sheet with the columns listed in `skills/application-followup-nudges/SKILL.md` (A–Q).
3. In each `SKILL.md`, replace the `{PLACEHOLDER}` values: `{CONTEXT_DIR}`, `{PROJECTS_DIR}`, `{YOUR_EMAIL}`, `{TRACKER_SHEET_ID}`, `{TRACKER_TAB_GID}`, `{NOTION_TASKS_DB_ID}` (optional), resume paths, timezone, and target locations/levels.
4. In Cowork, ask Claude to create a scheduled task from each SKILL.md with the cron schedule noted in the file.

## Placeholders used

- `{YOUR_NAME}`, `{YOUR_EMAIL}` — you
- `{CONTEXT_DIR}` — folder holding your context files (about-me, current-focus, work-preferences, writing-voice)
- `{PROJECTS_DIR}` — folder where output files (run files, planners, prep packets) get saved
- `{TRACKER_SHEET_ID}`, `{TRACKER_TAB_GID}` — your Google Sheet tracker
- `{NOTION_TASKS_DB_ID}` — optional Notion tasks database
- `{COMP_TARGET}`, `{COMP_FLOOR}` — your total-comp target and walk-away floor
- `{LEADERSHIP_RESUME_PATH}`, `{IC_RESUME_PATH}` — resume variants
- `{TIMEZONE}` — e.g., America/Los_Angeles

## License

MIT — use it however you like.
