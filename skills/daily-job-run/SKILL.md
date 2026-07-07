---
name: daily-job-run
description: "Weekday 8am exec-TPM job search: ranked roles, tailored materials, contacts, tracker update."
# suggested cron: 0 8 * * 1-5
---

Run {YOUR_NAME}'s daily job search. Produce a dated run file and update the tracker. Work autonomously; never apply, submit forms, or message anyone — drafts only for {YOUR_NAME} to review.

STEP 1 — Load context. Read and follow exactly:
- {CONTEXT_DIR}/current-focus.md (target criteria, comp target of {COMP_TARGET} TC / {COMP_FLOOR} floor, no-list, level-fit + over-qualification rules, board list, contacts approach)
- {CONTEXT_DIR}/about-me.md
- {CONTEXT_DIR}/work-preferences.md (drafts before sending; deliverable formats)
- {CONTEXT_DIR}/writing-voice.md (writing voice — apply to anything written in {YOUR_NAME}'s voice)
Skim recent {PROJECTS_DIR}/daily-job-run-*.md and the job search tracker Google Sheet so you never repeat roles already surfaced, applied to, or tracked.

STEP 2 — Search boards via the Claude in Chrome extension (user is logged in). Check LinkedIn "Jobs where you'd be a top applicant" FIRST, then LinkedIn search, Indeed, Greenhouse, Lever, AshbyHQ, Workday, Built In SF, Wellfound, Levels.fyi, VC portfolio boards (General Catalyst/a16z/Sequoia/Greylock), SmartRecruiters, iCIMS, and direct career pages for your target companies (listed in current-focus.md). Focus: {TARGET_LOCATION} (remote OK), {TARGET_LEVELS} TPM. Most boards are JavaScript-rendered, so render them in Chrome rather than a plain fetch.

STEP 3 — Filter and rank per current-focus.md. Exclude anything already in the tracker/prior runs and anything on the no-list (companies you've ruled out, defined in current-focus.md). Rank by level fit first (Director/Head/VP/Principal-manager highest; senior-IC roles flagged "likely over-leveled" unless comp clears the floor AND the work matches your hands-on specialty). Comp is flag-not-filter: verify the posted band on the live posting, flag the gap vs {COMP_TARGET} target/{COMP_FLOOR} floor, and tag "confirm comp before investing" when no band is posted. Add a one-line over-qualification/comp note per pick.

STEP 4 — For the top 3-5 roles capture: title, company, team, level, location, verified comp, direct link, why it fits, watch-outs. For the single strongest pick, draft a tailored resume summary + reworded bullets and a short cover letter in {YOUR_NAME}'s voice (follow writing-voice.md). Use the IC resume variant ({IC_RESUME_PATH}) for senior-IC roles; the standard leadership resume ({LEADERSHIP_RESUME_PATH}) for Director+.

STEP 5 — Contacts. For each pick, use Claude in Chrome on LinkedIn (people search filtered by current company, network 1st+2nd) to surface referral contacts and recruiters with mutual-connection counts. Draft a short LinkedIn outreach note per role in {YOUR_NAME}'s voice (connect-request length ~300 chars for 2nd-degree; a normal message for 1st-degree).

STEP 6 — Write the dated run file to {PROJECTS_DIR}/daily-job-run-YYYY-MM-DD.md (today's date).

STEP 7 — Update the tracker. Append each new role as a row in the jobs tab of the tracker (Google Sheet id {TRACKER_SHEET_ID}, tab gid {TRACKER_TAB_GID}) via the Chrome extension (no Sheets write API is connected). Columns A-Q in order: Job Title, Link, Company, Team, Leveling, Location, Date Applied (blank), Status (Ready to Apply), Hiring Manager (blank), Recruiter, Internal Reference, Claude Notes (tier + over-leveled/comp note), LinkedIn Connections, Referral status, Link to Resume, Link to CL, Outreach Note (LinkedIn). Append below the last data row (find it with the Name Box + cmd+Up; data is additive only — never edit or delete existing rows).

STEP 8 — Finish with a 3-4 sentence summary naming the single best opportunity and the recommended first move (usually a warm intro before applying). Reminder: drafts only — do not apply, submit, or message on {YOUR_NAME}'s behalf.
