---
name: application-followup-nudges
description: "Mon & Thu 8am: find stale job applications and draft friendly follow-up pings (drafts only)."
# suggested cron: 0 8 * * 1,4
---

Scan {YOUR_NAME}'s job applications for staleness and draft friendly follow-up nudges. Run autonomously. Drafts only — never send, apply, or message anyone. Timezone: {TIMEZONE}.

STEP 1 — Load voice/context:
- {CONTEXT_DIR}/writing-voice.md (writing voice — informal, direct, human, no AI filler, no em dashes)
- {CONTEXT_DIR}/about-me.md and work-preferences.md
- {CONTEXT_DIR}/current-focus.md

STEP 2 — Read the tracker. Open the jobs tab (gid={TRACKER_TAB_GID}) of the job search tracker (Google Sheet id {TRACKER_SHEET_ID}) via the Claude in Chrome extension; fetch the gviz CSV in-page (/gviz/tq?tqx=out:csv&gid={TRACKER_TAB_GID}&cachebuster, credentials include, cache no-store). Columns A-Q: A Job Title, B Link, C Company, D Team, E Leveling, F Location, G Date Applied, H Status, I Hiring Manager, J Recruiter, K Internal Reference, L Claude Notes, M LinkedIn Connections, N Referral status, O Resume, P Cover Letter, Q Outreach Note.

STEP 3 — Identify stale items. Honor the no-go rule: only consider rows whose column H Status is "Applied" or "Interviews" (ignore anything else — "Not Interested", "Passed on me", etc. are closed). Flag as stale:
- Status "Applied" with a Date Applied (col G) 7+ days ago and no sign of movement.
- Status "Interviews" where the last step looks 5+ days stale.
Use today's date to compute gaps.

STEP 4 — For each stale role, draft a short, warm follow-up in {YOUR_NAME}'s voice. Pick the right recipient and channel:
- If there's a named recruiter (col J), hiring manager (col I), or internal reference (col K), draft a brief email-style note to that person.
- Otherwise draft a LinkedIn message to a referral contact listed in col M, or a recruiter to find.
Keep each note 3-5 sentences, specific to the role, no AI filler, no em dashes. A 2nd-degree LinkedIn note should be ~300 chars.

STEP 5 — Deliver. Where the follow-up is an email to a known address, create a Gmail draft (Gmail MCP create_draft) — never send. Compile everything into a dated file {PROJECTS_DIR}/followup-nudges-YYYY-MM-DD.md: a table of stale roles (company, role, days since last activity, recommended action, who to contact) followed by each drafted note.

STEP 6 — Finish with a 2-3 sentence summary: how many stale, and the single most urgent follow-up.
