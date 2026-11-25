---
name: application-followup-nudges
description: "Mon & Thu 8am: find stale applications, half-finished applications and closing deadlines, and draft friendly follow-up notes (drafts only)."
# suggested cron: 0 8 * * 1,4
---

Scan {YOUR_NAME}'s job applications for staleness and draft friendly follow-up nudges. Run autonomously. Drafts only: never send, apply, or message anyone. Timezone: {TIMEZONE}.

STEP 1 — Load voice and context:
- {CONTEXT_DIR}/writing-voice.md (writing voice: informal, direct, human, no AI filler, no em dashes)
- {CONTEXT_DIR}/about-me.md and {CONTEXT_DIR}/work-preferences.md
- {CONTEXT_DIR}/current-focus.md

STEP 2 — Read the pipeline from the Notion tracker. Query the tracker data source `{TRACKER_DB_ID}` with `notion-query-data-sources`. Relevant properties: Role (title), Company, Status, Job URL, Level, Shape, Location, Comp Min, Comp Max, Deadline, Next Action, Warm Path, Notes, `date:Date Applied:start`, `date:First Seen:start`, Source.

Status is a closed vocabulary. Use the live list recorded in `current-focus.md` and match it exactly. The suggested set is: **Applying (Reviewed), Ready to Apply, Applied, Recruiter Screen, Interviewing, Offer, Accepted, Rejected, Not Interested, Passed on me, Closed.** A status name that isn't in the live vocabulary silently matches nothing; an older version of this file listed values like "Phone Screen" and "Interviews" and quietly missed every row they were meant to catch. `Phase` is a formula property and is not queryable in SQL mode, so filter on Status.

STEP 2.5 — Read the latest recruiter communication on each candidacy: email through the Gmail MCP, and LinkedIn messages through the Claude in Chrome extension.

STEP 3 — Identify what needs a push. Three separate buckets, reported separately:

**a) Stale applications.** Rows with Status "Applied", "Recruiter Screen", "Interviewing" or "Offer". Flag as stale:
- Status "Applied" with Date Applied 7+ days ago and no sign of movement.
- Any interviewing or screening status where the last step looks 5+ days stale.
Use today's date to compute gaps. Where Date Applied is empty but the status implies submission, say so rather than guessing.

**b) Stalled mid-application. Status "Applying (Reviewed)".** These were started but never submitted, and nothing else in the system chases them. They are not stale applications and they are not unstarted leads; they are half-finished work. List every one with how long it has sat and what the Next Action says. Before this bucket existed, half a dozen half-finished applications sat unnoticed.

**c) Closing deadlines.** Any row with a Deadline inside the next 10 days whose Status is still "Ready to Apply" or "Applying (Reviewed)". A closing req with no application against it is more urgent than a stale follow-up. A Next Action that opens with `NOT VERIFIED` means comp or role shape is unconfirmed, so the real action is confirming, not applying.

STEP 4 — For each stale role, draft a short, warm follow-up in {YOUR_NAME}'s voice. Pick the recipient and channel from the Warm Path property and the Notes: a named recruiter or hiring manager gets an email-style note; otherwise draft a LinkedIn message to a referral contact. Keep each note 3-5 sentences, specific to the role, no AI filler, no em dashes. A 2nd-degree LinkedIn note should be ~300 characters.

**Read the Warm Path property before choosing a contact.** It records dead routes, including people who never replied and people you have asked not to contact again. Do not draft to a retired route. If the only listed path is retired, say so and suggest an alternative rather than reusing it.

STEP 5 — Deliver. Where the follow-up is an email to a known address, create a Gmail draft (Gmail MCP create_draft); never send. Compile everything into {PROJECTS_DIR}/followup-nudges-YYYY-MM-DD.md: a table of stale roles (company, role, days since last activity, recommended action, who to contact), then the stalled mid-application list, then the closing-deadline list, then each drafted note.

STEP 6 — Optionally update the tracker row's Next Action property to reflect the follow-up you drafted. Do not change Status; that is {YOUR_NAME}'s to set. Append to Notes and Warm Path rather than replacing them; they are edited by hand and carry real recruiter detail.

STEP 7 — Finish with a 2-3 sentence summary: how many stale, how many stalled mid-application, any closing deadlines, and the single most urgent follow-up.
