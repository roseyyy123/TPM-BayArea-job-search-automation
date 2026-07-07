---
name: interview-prep-autoprep
description: "Daily 5pm: if an interview is on tomorrow's calendar, build a tailored prep packet (with technical mock when relevant)."
# suggested cron: 0 17 * * *
---

Check {YOUR_NAME}'s calendar for interviews happening tomorrow and, for each, build a tailored interview prep packet. Run autonomously; never send, apply, or message anyone. Drafts/files only. Timezone: {TIMEZONE}.

STEP 1 — Detect tomorrow's interviews. Use the Google Calendar MCP to list tomorrow's events on the primary calendar ({YOUR_EMAIL}). Identify any that are job interviews (title/description mentions interview, screen, onsite, recruiter, hiring manager, or matches a company in the tracker). Also cross-check the jobs tab (gid={TRACKER_TAB_GID}) of the job search tracker (Google Sheet id {TRACKER_SHEET_ID}, read via Claude in Chrome gviz CSV) for rows with Status "Interviews". If there is no interview tomorrow, write a one-line "no interview tomorrow — no packet needed" note and stop.

STEP 2 — Load context and voice:
- {CONTEXT_DIR}/about-me.md (your background, signature projects, and strengths)
- {CONTEXT_DIR}/current-focus.md and work-preferences.md
- {CONTEXT_DIR}/writing-voice.md (voice)
- Match the structure and depth of a prior prep packet the user liked. Look in {PROJECTS_DIR}/ for prior interview-prep files and mirror that format.

STEP 3 — Research each interview via the Claude in Chrome extension (user is logged in):
- Company: recent news, product/AI strategy, scale, culture signals (Glassdoor), and how it maps to your strengths and comp targets.
- Team/role: pull the live job posting (link in tracker col B) for responsibilities and level.
- Interviewer(s): if a name is in the calendar event or tracker (col I Hiring Manager / col J Recruiter), look them up on LinkedIn — background, tenure, what they likely probe, and any mutual connections.

STEP 4 — Build the packet (in {YOUR_NAME}'s voice, no AI filler, no em dashes):
- Snapshot: company, team, role, level, comp read vs the {COMP_TARGET} target / {COMP_FLOOR} floor, interview logistics from the calendar event.
- Interviewer profile(s) and what each likely focuses on.
- Likely questions with tailored talking points and specific stories from the user's background (pulled from about-me.md).
- Smart questions for the candidate to ask.
- Watch-outs / positioning (e.g., over-leveling, comp gaps).
- TECHNICAL MOCK: when the interview is technical (system design, data/ML infra, technical screen), include a mock section: 3-5 realistic technical questions for the role with strong sample answers and a short framework the candidate can use live.

STEP 5 — Deliver. Save each packet to {PROJECTS_DIR}/interview-prep-{company}-YYYY-MM-DD.md. Also create a Gmail draft (Gmail MCP create_draft) to {YOUR_EMAIL} with subject "Interview prep — {Company} {tomorrow's date}" containing the packet or a tight summary plus the saved file path. Do not send (the Gmail connector only supports drafts).

STEP 6 — Finish with a one-line summary naming the interview(s) prepped and the top thing for the candidate to nail.
