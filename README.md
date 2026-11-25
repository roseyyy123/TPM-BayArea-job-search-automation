# TPM Job Search Automation

Four scheduled Claude agents that run a senior TPM job search for me every day. They find and rank roles, draft tailored materials and outreach, keep my tracker current, nudge stale applications and build interview prep packets. I built them for my own search and hardened them over months of daily runs, so every rule in here came from something that actually broke. Fork it, fill in the placeholders and make it yours.

## Why I built it

A senior job search is really a program: dozens of live threads, lots of sources, and follow-ups that go cold when nobody owns them. I wanted agents doing the legwork every morning while I kept every decision that touches another person. So the agents only ever draft. They never apply, send email or message anyone on my behalf.

## What's included

| Skill | Runs | What it does |
|---|---|---|
| `skills/daily-job-run` | Weekdays 8am | Searches job boards, ranks roles against my criteria, drafts tailored resume and cover letter material plus outreach notes, and updates the tracker |
| `skills/morning-planner` | Daily 6:30am | Builds the day's plan from my calendar, job search to-dos, interview prep and personal tasks, as a Gmail draft |
| `skills/application-followup-nudges` | Mon and Thu 8am | Finds stale applications, half-finished applications and closing deadlines, and drafts friendly follow-up notes |
| `skills/interview-prep-autoprep` | Daily 5pm | If tomorrow has an interview, builds a tailored prep packet, with a technical mock when it fits |

## How it fits together

```
Context folder (who I am, targets, comp rules, voice)
        │
        ▼
daily-job-run ──▶ job boards, read through public ATS APIs and Claude in Chrome
        ├──▶ dedup ledger: every role ever evaluated, rejects included
        ├──▶ Notion tracker: only roles worth acting on
        └──▶ dated run file with drafts: bullets, cover letter, outreach

morning-planner  ◀── calendar, tracker, personal tasks  ──▶ Gmail draft
followup-nudges  ◀── tracker, email, LinkedIn messages  ──▶ Gmail drafts
interview-prep   ◀── calendar, tracker, story bank      ──▶ prep packet
```

## How I know it works

Agents fail quietly. A run that found nothing looks just like a run that broke, so most of my effort went into making failures visible. A few checks that came from real misses (the full list is in [CHANGELOG.md](CHANGELOG.md)):

- **Coverage, not just success.** One board capped every response at 10 jobs while reporting the true total. For weeks the loop read about 2% of that board and reported success. Every run now checks that the jobs fetched match the count the board reports.
- **Zero rows is a warning, not a clean sweep.** A missing User-Agent header made a whole ATS return nothing. A 200 with no rows is now treated as worse than an error.
- **Two stores, reconciled both ways.** The tracker holds roles worth acting on, and an append-only ledger holds everything ever evaluated. Reconciling in only one direction once hid dozens of researched roles for a week.
- **Rebuilds merge, never overwrite.** A clean rebuild of the dedup index would have dropped about 100 known no-go roles and let them come back as new.
- **One req, one key.** A canonical title normalizer expands abbreviations like "Sr." and "TPM" before matching, so a reworded repost can't slip past dedup or the rejected-req registry.
- **Every run shows its work.** Geography drops and rejected-req drops get their own counted lines, and each run says whether any good role arrived from outside the system. If roles keep showing up through personal alerts, the sweep is too narrow.

## A sample run (made-up data)

```
Daily job run: Tuesday, September 15
Read: 14 boards, 212 postings (count check passed on all 14)
Dropped: 31 for geography, 4 rejected reqs, 58 already seen

Top pick: Director, AI Platform Programs at Northwind AI (remote)
Why it fits: leads TPMs across model serving and evals; level and domain match
Watch out: no posted band, so confirm comp before investing time
First move: warm intro through a former teammate (1st degree)
Drafts ready: summary and 5 bullets, cover letter, outreach note
```

## Requirements

- Claude Cowork desktop app with scheduled tasks
- Connectors: Gmail (drafts), Google Calendar and Notion
- Claude in Chrome, signed in to LinkedIn and your job aggregator
- A Notion tracker database, with the properties listed in `skills/daily-job-run/SKILL.md` (Step 8)
- A Context folder holding the files in `templates/`, filled in

The daily run also keeps `greenhouse-tokens.json` (a JSON list of Greenhouse board tokens to sweep) in your Context folder; start it with your target companies and the weekly discovery step adds more. The dedup ledger (`ledger.jsonl` and `ledger-index.json`) is created there on the first run.

## Setup

1. Copy the files in `templates/` into a Context folder Claude can read, and fill them in: who you are, target roles, comp targets, companies to skip, your tracker's status list, where to search, your interview stories and your writing voice.
2. Create your Notion tracker with the properties in `skills/daily-job-run/SKILL.md` (Step 8). Phase is a formula built from Status, so never set it by hand.
3. In each `SKILL.md`, replace the placeholder values listed below. `{TRACKER_DB_ID}` is your Notion tracker's data source ID.
4. In Cowork, ask Claude to create a scheduled task from each `SKILL.md`, using the schedule noted at the top of the file.

## Placeholders

- `{YOUR_NAME}`, `{YOUR_EMAIL}`: you
- `{TIMEZONE}`: for example, America/Los_Angeles
- `{CONTEXT_DIR}`: the folder holding your context files, ledger and token list
- `{PROJECTS_DIR}`: where run files, planners and prep packets get saved
- `{TRACKER_DB_ID}`: your Notion tracker's data source ID
- `{NOTION_TASKS_DB_ID}`: an optional Notion tasks database for the morning planner
- `{TARGET_METRO_1}`, `{TARGET_METRO_2}`: metros where onsite or hybrid roles work for you
- `{COMP_TARGET}`, `{COMP_FLOOR}`: your total comp target and walk-away floor
- `{BASE_MINIMUM}`: a base salary floor, separate from total comp
- `{LEADERSHIP_RESUME_PATH}`, `{IC_RESUME_PATH}`: your two resume versions

## Known gaps

The morning planner still reads the original Google Sheet tracker (`{TRACKER_SHEET_ID}`, `{TRACKER_TAB_GID}`). Moving it to Notion is next.

## Changelog and license

See [CHANGELOG.md](CHANGELOG.md) for what changed and why. MIT license, so use it however you like.
