---
name: interview-prep-autoprep
description: "Daily 5pm: if an interview is on tomorrow's calendar, build a tailored prep packet (with a technical mock when relevant)."
# suggested cron: 0 17 * * *
---

Check {YOUR_NAME}'s calendar for interviews happening tomorrow and, for each, build a tailored interview prep packet. Run autonomously; never send, apply, or message anyone. Drafts and files only. Timezone: {TIMEZONE}.

STEP 1 — Detect tomorrow's interviews. Use the Google Calendar MCP to list tomorrow's events on the primary calendar ({YOUR_EMAIL}). Identify job interviews (title or description mentions interview, screen, onsite, recruiter, hiring manager, or matches a company in the pipeline). Cross-check against the Notion tracker: query the data source `{TRACKER_DB_ID}` with `notion-query-data-sources` for rows with Status "Interviewing", "Recruiter Screen" or "Offer".

Status is a closed vocabulary. Use the live list recorded in `current-focus.md` and match it exactly; a status name that isn't in it silently matches nothing. The `Phase` formula property is not queryable in SQL mode, so filter on Status.

If there is no interview tomorrow, write a one-line "no interview tomorrow, no packet needed" note and stop.

STEP 1.5 — CHECK FOR AN EXISTING PACKET FIRST. Look in {PROJECTS_DIR}/ for any `interview-prep-*` file matching this company and date, including variants with a role qualifier in the filename (for example `interview-prep-{company}-{role}-YYYY-MM-DD.md`). If one already exists:
- Do not write a second packet.
- Read it, and append only what is genuinely new since it was written: a named interviewer who has since been confirmed, fresh company news, a changed req, or a calendar detail.
- If nothing has changed, say so in one line and stop. A duplicate packet is worse than none, because the candidate then has to work out which is current.

STEP 2 — Load context and voice:
- {CONTEXT_DIR}/about-me.md (your background, signature projects and strengths)
- {CONTEXT_DIR}/current-focus.md and {CONTEXT_DIR}/work-preferences.md
- {CONTEXT_DIR}/writing-voice.md (voice)
- {CONTEXT_DIR}/interview-story-bank.md (a reusable STAR story library; pull from it rather than inventing stories). Honour the **freshness ledger** in that file: it records who has already heard which story. Do not send the candidate into a room with a story that interviewer's panel already heard, unless you flag it and give them a new detail to add.
- Match the structure and depth of prior packets in {PROJECTS_DIR}/ and mirror that format.

STEP 3 — Research each interview via the Claude in Chrome extension (the user is logged in):
- Company: recent news, product and AI strategy, scale, culture signals (Glassdoor), and how it maps to your strengths and comp targets.
- Team and role: pull the live posting from the tracker row's Job URL for responsibilities and level. **Read the live req rather than trusting the row's summary.** A stale one-line assessment in Notes has been wrong before. Some career sites are client-rendered, so a plain fetch returns an empty shell; use the browser for those. The row's Level, Shape, Comp Min, Comp Max, Warm Path and Notes properties, and its page body, carry prior research; read the page body before re-researching from scratch.
- Interviewer(s): if a name is in the calendar event or the tracker row's Warm Path or Notes, look them up on LinkedIn: background, tenure, what they likely probe, mutual connections.

STEP 4 — Build the packet (in {YOUR_NAME}'s voice, no AI filler, no em dashes):
- Snapshot: company, team, role, level, IC vs manager, comp read against the {COMP_TARGET} target and {COMP_FLOOR} floor, logistics from the calendar event.
- Interviewer profile(s) and likely focus.
- Likely questions with tailored talking points and specific stories from the story bank.
- **Gaps.** If a minimum qualification on the req is something the candidate genuinely does not have, say so plainly and give a plan for naming it rather than claiming it, plus the closest honest analogue from their background. Do not paper over it.
- Smart questions for the candidate to ask.
- Watch-outs and positioning (over-leveling, comp gaps, role-shape ambiguity).
- **If there is more than one live thread at the same company**, say so and give a line on how to handle it if asked.
- TECHNICAL MOCK when the interview is technical (system design, data or ML infra, technical screen): 3-5 realistic questions with strong sample answers and a short framework the candidate can use live.

STEP 5 — Deliver. Save each packet to {PROJECTS_DIR}/interview-prep-{company}-YYYY-MM-DD.md, adding a short role qualifier when there is more than one thread at that company. Append a link or short summary to the matching tracker row's page body so the prep lives with the role. Create a Gmail draft (Gmail MCP create_draft) to {YOUR_EMAIL} with subject "Interview prep: {Company} {tomorrow's date}", containing the packet or a tight summary plus the saved file path. Do not send.

STEP 6 — Finish with a one-line summary naming the interview(s) prepped and the top thing for the candidate to nail.
