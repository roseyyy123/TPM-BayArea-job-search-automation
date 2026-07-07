---
name: morning-planner
description: "Daily 6:30am planner: job-search to-dos, interview prep, calendar, and personal tasks, as a Gmail draft."
# suggested cron: 30 6 * * *
---

Produce {YOUR_NAME}'s daily morning planner and deliver it as a Gmail draft addressed to {YOUR_EMAIL}. Run autonomously; never send email, apply, or message anyone. Timezone: {TIMEZONE}.

STEP 1 — Load context (read and follow):
- {CONTEXT_DIR}/current-focus.md (job-search criteria, comp targets, no-list)
- {CONTEXT_DIR}/about-me.md
- {CONTEXT_DIR}/work-preferences.md
- {CONTEXT_DIR}/writing-voice.md (writing voice — apply to everything written in {YOUR_NAME}'s voice; informal, direct, no AI filler, no em dashes)

STEP 2 — Gather the day's inputs:
a) Calendar: use the Google Calendar MCP. List today's events on the primary calendar ({YOUR_EMAIL}) and any other calendars the user owns. Capture timed commitments (appointments, errands, calls). Also pull tomorrow's events so the planner can flag prep needed.
b) Personal tasks: use the Notion MCP. Read the personal tasks database (id {NOTION_TASKS_DB_ID}). Surface open/incomplete items, especially anything due today or overdue. Note due dates. (Skip this step if Notion isn't connected.)
c) Job search: read the jobs tab (gid={TRACKER_TAB_GID}) of the job search tracker (Google Sheet id {TRACKER_SHEET_ID}) via the Claude in Chrome extension. Fetch the gviz CSV in-page (/gviz/tq?tqx=out:csv&gid={TRACKER_TAB_GID}&cachebuster, credentials include, no-store cache). Identify: rows with Status "Interviews" (active interview processes — what's the next step), rows "Ready to Apply" (queued to apply), and applications that may need follow-up. IMPORTANT no-go rule: ignore any row whose column H Status is not "Applied", "Ready to Apply", or "Interviews" (e.g., "Not Interested", "Passed on me" are closed).
d) Any other open personal/admin threads found in the tasks database or recent context.

STEP 3 — Build the planner. One tight, scannable email in {YOUR_NAME}'s voice. Suggested sections, in this order:
1. Today at a glance (timed calendar items in chronological order, with times).
2. Job search to-dos (interviews in progress + their next step; anything Ready to Apply worth acting on today; follow-ups due).
3. Interview prep needed (if an interview is on the calendar today or tomorrow, name it and the prep to do; point to any prep packet already saved in {PROJECTS_DIR}/).
4. Life & admin (personal tasks: errands, appointments; flag overdue).
5. Other open threads (if any).
Keep it brief and specific. No filler, no AI words, no em dashes. Use times and concrete actions.

STEP 4 — Deliver. Create a Gmail draft (Gmail MCP create_draft) addressed to {YOUR_EMAIL} with subject "Morning planner — {Weekday}, {Month} {D}". Do not send. Also save a copy to {PROJECTS_DIR}/morning-planner-YYYY-MM-DD.md. Note: the Gmail connector only supports drafts, not sending, so the draft is the deliverable.

STEP 5 — Finish with a one-line confirmation of what was drafted and the single most important thing for {YOUR_NAME} to do today.
