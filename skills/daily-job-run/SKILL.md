---
name: daily-job-run
description: Weekday job-board sweep for senior/exec TPM roles. Ranks against your criteria, verifies comp on the employer's own site, drafts outreach and application material, updates your tracker and dedup ledger. Drafts only — never applies, submits, or messages anyone.
schedule: weekdays 08:00 {TIMEZONE}
model: Claude Opus 5
---

# Daily job run

Run the job search end to end. Produce a dated run file, update the tracker, and append to the dedup ledger. Work autonomously; never apply, submit forms, or message anyone. **Drafts only, for review.**

> **Changed since the previous published version.** The tracker moved from a Google Sheet to a Notion database, an append-only dedup ledger was added, geography became a hard drop keyed on work arrangement, rejected reqs became req-level rather than company-level blocks, and a failure-escalation ladder was added for boards that stop responding. Later revisions added a canonical title normalizer, stricter rules for the rejected-req registry, and more ATS endpoint lessons. See `CHANGELOG.md`.

---

## State lives in two places and nowhere else

- **The tracker** — a Notion database, `{TRACKER_DB_ID}`. The live pipeline. Only roles worth acting on live here.
- **`{CONTEXT_DIR}/ledger.jsonl`** — append-only, one JSON object per line, every posting ever fetched or evaluated *including rejects*. Never rewrite it, only append. Companion `{CONTEXT_DIR}/ledger-index.json` holds `{"urls":{normalized_url:status},"cts":{normalized_company|title:status}}` for O(1) dedup. **Read the index, not the ledger.**

Normalization used by the index. This is the canonical normalizer; match it exactly or dedup silently breaks:

- **URL:** lowercase, strip scheme and `www.`, strip query string, strip trailing slash.
- **company:** lowercase, remove every non-alphanumeric character.
- **title:** **first expand abbreviations on the raw string, word-boundary, case-insensitive**: `\bsr\.?\b` → senior, `\bdir\.?\b` → director, `\bmgr\.?\b` → manager, `\btpm\b` and `\btpgm\b` → technical program management, `\bepm\b` → engineering program management, `\bvp\b` → vice president. **Then** remove every non-alphanumeric character. Join `company|title`.
- **Never expand `pm`.** It is ambiguous between product manager and program manager, and expanding it would merge genuinely different roles.

Without the abbreviation step, "Sr. Director, TPM" and "Senior Director, Technical Program Management" are two different keys for one req. Introducing the normalizer and migrating the index to it once cut the index-only keys by about two thirds. **Record the expected carried-forward count after a migration (see Step 8). If it climbs back toward the old number, the canonical normalizer is not being applied at runtime.** Keep the migration spec and a rollback copy in `{CONTEXT_DIR}`.

---

# PHASE A — cheap. Text search and public APIs only. No browser.

## STEP 0 — Load state

1. Read `{CONTEXT_DIR}/ledger-index.json`.
2. Query the tracker for all rows: Role, Company, Status, Job URL, Level, Shape, Comp Min, Comp Max, Deadline, Next Action.
3. **Reconcile tracker → ledger.** Any tracker row whose Job URL or normalized company+title is absent from the index gets appended now with `status: "from-tracker"` and `reason` set to its tracker Status. This is the direction that stops a hand-added role being re-surfaced as net-new. Regenerate the index after appending.
4. **Reconcile ledger → tracker.** Query the ledger for `status: "shortlisted"` rows with no tracker row. These are unactioned work, not new finds. Surface them under **Orphaned shortlists** and treat them as candidates alongside the sweep. If there are none, say so in one line.
5. **Check for duplicate tracker rows.** Two rows sharing a normalized Job URL is a real failure mode: a row hand-added after recruiter contact collides with a daily-run row for the same req. Report duplicates, recommend which to keep, **do not create a third.**
6. Read only the **Recommendation** and **Deadlines** sections of the previous run file. Drop reasons are already in the ledger's `reason` field.
7. **Exclusion set** = everything in the index + every tracker row + the no-list + the rejected-req registry.

## STEP 1 — Load context

Read and follow exactly: `{CONTEXT_DIR}/current-focus.md`, `{CONTEXT_DIR}/about-me.md`, `{CONTEXT_DIR}/work-preferences.md`, `{CONTEXT_DIR}/writing-voice.md`, `{CONTEXT_DIR}/search-queries.md`.

## STEP 2 — Sweep the boards

**Read this framing first, because it was got wrong once already.** The sweep has two jobs and they fail in different ways:

- **Depth** — checking known target companies thoroughly. ATS APIs do this well and cheaply.
- **Discovery** — finding a role at a company nobody thought to list. **A token-list sweep cannot do this by construction.**

The lesson: a Director-level req at a mid-size company arrived via an email alert, not the sweep. That company publishes on Greenhouse — a board swept on every run. Its token had simply never been on the list. **Board coverage was fine. Company coverage was not.**

### 2a. Depth — ATS sweep

- **Workday** — `POST /wday/cxs/{tenant}/{site}/jobs` with a JSON body. The req-detail endpoint at the returned `externalPath` carries the comp band and posting date. Maintain an explicit tenant/host/site table in `search-queries.md`; a wrong site token returns 422 or 404 and looks like an outage.
  - **The req-detail path is `/wday/cxs/{tenant}/{site}{externalPath}`.** Inserting `/job` before `externalPath` returns 406 and looks like a refusal.
  - When reporting a Workday failure, distinguish "wrong site token" from "tenant refused." Record which employers turned out **not** to be Workday tenants, and where they publish instead, so the next run doesn't rediscover it.
- **Greenhouse** — sweep **every** token in `{CONTEXT_DIR}/greenhouse-tokens.json`. **Do not trim to "tokens that produced picks."** One token is one HTTP call and the whole sweep runs in seconds. An extra token costs nothing; a missing one costs a role.
  - **An `SSL: UNEXPECTED_EOF_WHILE_READING` on a Greenhouse token is usually a transient flake, not a dead board.** Several tokens have hit it in one run and all recovered on retry. Retry before reporting any Greenhouse failure.
- **Ashby** — `api.ashbyhq.com/posting-api/job-board/{token}?includeCompensation=true`. **Requires a `User-Agent` header**; without it every board returns zero jobs silently rather than erroring. The compensation field makes targeted per-token calls worthwhile even if a full sweep isn't.
- **SmartRecruiters** — `api.smartrecruiters.com/v1/companies/{company}/postings?limit=100&offset=N`.
- **Phenom** — `POST /widgets` with a JSON body. Some employers run Phenom purely as a front-end search layer while the ATS behind Apply is Workday. **Check where the Apply button points before concluding an employer is not on Workday.**
- **Eightfold** — two paths exist in the wild: `/api/apply/v2/jobs` and `/api/pcsx/search`. **Derive which one a given employer uses before treating it as swept.** Response shape differs too: postings may sit at the top level or nested under `data.positions`, with the total at `data.count`. Repeat filter parameters rather than comma-joining them.
  - **Facet filters such as seniority or career discipline can return HTTP 200 with `count: 0` from code.** Those facets may exist only on the logged-in personalized surface. Query by keyword and location, step `start` by the observed page size, and filter titles client-side.
  - **Check that the detail URL you record resolves to the same req.** An older detail-URL form on one employer's site redirected to an unrelated posting. Use the form the site itself links to today.

**Named target companies stay swept at full depth every run.** A single rejection must never quietly shrink a company's sweep; the rejected-req rule in Step 3 handles the one req.

> **Paginated APIs lie about completeness.** One employer's endpoint caps at 10 postings per response regardless of the `num` parameter while reporting the true total in `count`. A loop requesting 100, receiving 10, and stopping because `len < 100` read about **2% of that board for weeks while reporting success.** Step the offset by the *observed* page size until the response is empty, and assert `fetched == count`.

### 2b. Token discovery — weekly, not daily

Generate 100–200 candidate Greenhouse tokens for target-metro mid-size companies not already listed (company name lowercased, punctuation stripped, is right most of the time) and probe each against `boards-api.greenhouse.io`. Append the valid ones. **This is the only mechanism in Phase A that can find a company nobody named.**

**Screen every validated token against the no-list before adding it.** A valid token with real postings can still belong to a no-list company or one of its subsidiaries, and once it is on the list it gets swept every day. Record when discovery last ran and when it is next due.

### 2c. Discovery — browser channels, because no aggregator API works

Aggregators block server-side requests aggressively; 403 after roughly one request is typical. **That block applies to code, not to a logged-in browser.** Move aggregator work to Phase B. Note the trap: aggregator *keyword search* is close to useless at senior levels; the *personalized feed* is the part that works.

### 2d. When a source fails: escalate, do not skip

**A failing query is a task, not a status line.** One board reported "failed" for two weeks; the path was simply wrong and one browser network trace found it in minutes.

1. **Diagnose the code before retrying.**
   - `401 / 404 / 422` — wrong tenant, site token, or path. A config bug. **Fix the call; do not retry on a schedule.**
   - `403` — blocked. Try headers first (`User-Agent`, `Accept: application/json` have each silently fixed a whole board), then the browser.
   - `429` — rate limited. The endpoint is correct and reachable. Back off. **Never record a 429 as a dead endpoint.**
   - `200 with 0 rows` — worse than an error, because it looks like a clean sweep. Treat a persistent zero as a failure.
2. **Re-derive the endpoint from the browser.** Open the careers page in the browser, read the network requests, find the XHR returning the job list. Highest-yield step. Check the *response shape* too, not just the URL.
3. **Identify the ATS from its fingerprint** and use that platform's standard API.
4. **Fall back to web search** against the employer's own careers domain. **Label anything found this way `WEB-SEARCH FALLBACK, not a complete sweep`.** ⚠️ **Search indexes hold dead reqs — liveness-check every hit on the employer's site.** A band in a search snippet proves the req once existed, not that it is open.
5. **Browser page-read** as the last resort. Expensive; shortlist only.
6. **If all five fail, name the rung that failed and why.** "Untested, not absent" is an acceptable outcome. "Failed" on its own is not.

**A pasted job link is a first-class input.** If the user hands you a posting, verify it on the employer's own ATS and add that employer's token to `greenhouse-tokens.json` (or its tenant to `search-queries.md`) so the sweep catches them from then on.

Skip anything already in the exclusion set before fetching it. Pre-filter on titles and snippets.

> **Track discovery separately from depth.** Each run file should say whether Phase A found anything the browser pass would not have, **and** whether any role arrived from outside the system entirely. The second number is the one that matters: if roles keep arriving through personal alerts, company coverage is too narrow and the fix is more tokens, not fewer.

## STEP 3 — Filter, rank, cut to 3–5

**Three hard drops, applied BEFORE ranking. A role failing any of them is not a near-miss and gets no line in the picks table.**

### 1. Company-level no-list
Maintained in `current-focus.md`. Permanent, not re-litigated per run.

### 2. Geography — keys off work arrangement, not headquarters
- **Onsite or hybrid must sit in a target metro.** Anchored anywhere else is a **hard drop**, not a flagged near-miss.
- **Fully remote is acceptable anywhere.**
- **"Remote eligible" is not remote.** Many reqs say it while naming a required office metro. Read the live posting.
- **Expand any truncated location control before deciding.** A listing that shows one city "+1 location" often hides the location that decides the call.
- If the arrangement genuinely cannot be determined, surface it tagged `ARRANGEMENT UNVERIFIED` rather than guessing.

### 3. Rejected-req registry — REQ-level, not company-level
- A company that rejected you is **not** banned. The specific req is, permanently, along with any repost.
- **Match reposts on normalized company+title, not req ID.** Reqs get pulled and reposted under fresh IDs; the title is the stable part. A title reworded from e.g. "Technical Program Manager, X" to "Research Program Manager, X" on the same team at the same level **is the same req** — drop it.
- **Reposts can come with fresh-looking comp bands.** A rejected req once came back through an aggregator feed with the highest band the pipeline had ever seen. **A spectacular band is not evidence that a req is new.**
- Other roles at the same company are fair game when **team, level, or domain is materially different.**
- **A rejection in the registry or the ledger outranks the tracker's silence.** Never overturn a rejection-skip because no tracker row exists.
- **Only a tracker Status of `Passed on me` or `Rejected` may create a registry row.** A no-list drop is not a rejection. Getting this wrong once kept a perfectly good role blocked for weeks. If a registry entry's own note explains it away, check the ledger before trusting it.
- **Surface registry drops explicitly rather than dropping them silently**, especially when the role will reach you through your own channels or the comp is extraordinary. You should see the reasoning, not an unexplained absence.
- Warm contacts collected on a rejected req **stay usable** for other reqs at that company.

### Ranking

**Level fit first.** Director / Head / VP / Principal-manager highest. Flag senior-IC seats **"likely over-leveled"** unless comp clears the floor **and** it is hands-on work in your target domain. **Always label IC vs manager.**

⚠️ **Titles lie about level in both directions.** A req titled "Sr. Staff" whose body says "manager of a lean team of 3–4" is a manager seat, and a title-only filter will drop it. Read the body before ranking on the title.

**Comp is a flag, not a filter.** Note the gap against `{COMP_TARGET}` and `{COMP_FLOOR}`. Tag **"confirm comp before investing"** when no band is posted. One-line over-qualification/comp note per pick. **Where several reqs at one company share an identical posted band, use the minimum qualifications to separate the levels, not the comp.**

**Watch for adjacent-req collisions.** Two open applications to near-identical seats at one company read badly, and it is worse when one of them is a live interview loop. Flag it rather than filing both silently.

Orphaned shortlists compete on the same criteria as fresh finds. A role already researched and comp-verified but never actioned usually beats a new one — the work is done and the only thing missing is the ask.

**Do not trust a stale assessment over the live posting.** A req once dismissed as the "weakest thread" turned out on re-read to describe a materially different and stronger role.

**If fewer than 3 roles genuinely clear, report fewer and say why. Do not pad the list.** A run that reports zero picks and names the real bottleneck is more useful than one that pads. **When several comp-verified roles are already sitting at `Ready to Apply` with no action taken, say so. That, not supply, is usually the constraint.**

---

# PHASE B — expensive. Browser, shortlist only.

## STEP 4 — Verify

Open each shortlisted posting **on the employer's own career site** and verify the comp band against the live page. **Never carry a band over from a search snippet or an aggregator.** A role whose req cannot be found on the employer's own site is a near-miss, not a pick.

Capture per role: title, company, team, level, IC/manager, location, **work arrangement and required office metro**, verified comp, deadline, direct link, why it fits, watch-outs.

**Verify the arrangement at the same time as the band.** A role that passed the geography screen on a snippet and turns out hybrid in a third metro gets dropped here, not carried into the picks.

Some career sites are client-rendered and return an empty shell to a plain fetch. Use the browser for those.

**Some employers can only be swept in the browser.** A careers site rendered through GraphQL with signed tokens may return errors to code after one or two requests. Record the working method for each such site in `search-queries.md`: the search URL with your target offices, how long to wait for the page to render, and how to pull job IDs from the links on the page. Then:

- **Run several queries every time** (for example "director technical program manager", "head of technical program management", "senior director program management"). **A single query is not a sweep;** one query has been seen to omit live Director reqs that another query returned.
- **Never mark a req `Closed` because one query stopped returning it.** Check it by its job ID first.

Confirm liveness and req ID for any orphaned shortlist being promoted — reqs get reposted under new IDs, which restarts the clock.

**Aggregator pass, browser only, in this order:**

1. **Personalized feed** — the surface that generates your alert emails, driven by your saved profile. The only aggregator view worth reading.
2. **Saved / applied tabs** — roles you saved or applied to outside the system. Anything not in the tracker is an orphan: liveness-check it and give it a row.
3. **Keyword search is a trap.** At senior levels it returns retail management and unrelated IT roles, and an exact-phrase search returned *zero* results while that exact req was live at a company already on the swept list. Do not use it to conclude anything is or is not out there.

For any aggregator role that clears the bar, verify the band on the employer's own ATS and **add that employer's token to the Greenhouse list** so the sweep catches them from then on.

## STEP 5 — Contacts

Per pick, people search filtered to that company, 1st + 2nd degree. Surface referral paths and recruiters with mutual-connection counts. Draft one outreach note per role in your voice: ~300 characters for a 2nd-degree connect request, a normal message for 1st-degree.

**A 1st-degree connection inside the company outranks any 2nd-degree intro chain; lead with it.** **Prefer 1st-degree connections who previously worked with you.** Filtering to 1st degree and cross-referencing prior employers routinely beats a 2nd-degree contact with a better title. Useful search pattern: **company filter + 1st-degree filter + past-employer name as keyword.**

⚠️ **Check the tracker's Warm Path field before proposing any route.** It records dead routes, non-repliers, and people you've asked not to contact. **Never resurrect a retired route.**

⚠️ **Re-verify contacts recorded more than a few weeks ago.** People change jobs. A referral plan built on someone who has left is worse than no plan, because it looks researched.

**Comp questions go to the recruiter, never to the hiring manager or a peer contact.**

## STEP 6 — Materials for the strongest pick

Tailored resume summary plus reworded bullets, and a short cover letter in your voice per `writing-voice.md`. Use `{IC_RESUME_PATH}` for senior-IC roles and `{LEADERSHIP_RESUME_PATH}` for Director+.

**If a cover letter for the pick already exists, say so and note what needs updating rather than writing a second one.**

**Do not assert a qualification that is not established in your own records; flag the gap instead.** And if two of your own sources disagree on a number (say, the story bank and the resume quote different results for the same project), do not quote it anywhere until the conflict is resolved.

---

# OUTPUT AND STATE

## STEP 7 — Write the run file

`{PROJECTS_DIR}/daily-job-run-YYYY-MM-DD.md`: headline, how the run was done, what did not run and the consequence, orphaned shortlists, de-dup and drops with reasons, ranked picks, near-misses, drafts, contacts and outreach, recommended first move.

**Report geography drops and rejected-req drops on their own lines, with counts**, so it stays visible whether the filters are cutting the right things.

## STEP 8 — Update the tracker, then the ledger

**Tracker:** create one page for **every role this run marks `shortlisted`**, not only net-new picks. If a shortlisted role already has a row, update it rather than creating a second.

Put long-form material in the **page body**, not a Notes cell: why it fits, watch-outs, outreach drafts, tailored bullets, cover letter.

**Tracker properties.** Create the Notion database with these properties; every skill in this repo reads them:

| Property | Type and values |
|---|---|
| Role | Title |
| Company | Text |
| Status | Select; see the vocabulary below. Set new rows to "Ready to Apply". |
| Job URL | URL |
| Level | Select: VP, Sr Director, Director, Sr Manager, Principal IC, Staff IC |
| Shape | Select: Manager, IC, Unconfirmed |
| Location | Text, including the work arrangement and any required office metro |
| Comp Min, Comp Max | Number |
| Comp Verified | Checkbox. Tick it **only** when the band was read off the employer's own page. |
| Deadline, First Seen, Date Applied | Date |
| Source | Select: daily-run, manual, referral, recruiter inbound |
| Next Action | Text. Open it with `NOT VERIFIED` when comp, role shape or liveness is unconfirmed. |
| Warm Path | Text. Referral routes, including dead ones. |
| Notes | Text. Short notes only; long-form material goes in the page body. |
| Phase | Formula from Status (see below) |

Status is a closed vocabulary. **The API cannot add options, but a human can add them in the UI** — so record the live vocabulary in `current-focus.md` and re-verify when a run sees an unfamiliar value. Suggested set: **Applying (Reviewed), Ready to Apply, Applied, Recruiter Screen, Interviewing, Offer, Accepted, Rejected, Not Interested, Passed on me, Closed.** Suggested semantics for the ones that end a thread:

| Status | Means |
|---|---|
| `Closed` | The posting URL no longer resolves to the original req. **No judgement about you implied, and no registry entry.** |
| `Not Interested` | Live req, you declined. A judgement about fit. |
| `Passed on me` | The req rejected you. **Feeds the rejected-req registry.** |
| `Rejected` | Rejection after a formal application or interview. **Also feeds the rejected-req registry.** |

When a row moves to `Passed on me` or `Rejected`, add the req to the registry in `current-focus.md` in the same run.

**There is no backlog state by default.** A role that is live but has unverified comp or role shape still sits at "Ready to Apply" — open `Next Action` with `NOT VERIFIED` so the deadline sweep doesn't read it as ready to send.

**`Phase` is a formula property. Never set it.** It derives from Status: Applying = Ready to Apply or Applying (Reviewed); Interviewing = Applied, Recruiter Screen, Interviewing or Offer; Closed = the rest. Formula properties are not queryable in SQL, so filter on Status.

**Additive only for rows you own.** Never downgrade a hand-set Status. Notes and Warm Path carry real recruiter and interview detail — **append, never replace.** **If the user overrides a recommendation, record that they did and keep the underlying caution visible rather than deleting it.**

### Ledger — mandatory. The run is not complete without it.

Append one line for every posting this run fetched, evaluated, shortlisted or skipped, **including everything dropped** on comp, level, role shape, geography, the registry or the no-list.

Schema: `{key, company, title, url, first_seen, status, reason, note}`. `status` ∈ `shortlisted | near-miss | skipped | new | dead | from-tracker | sweep-record`. Put the specific detail in `reason` (free text), **never in `status`**.

⚠️ **Never put liveness annotations in `title`.** That changes the dedup key and forks one role into several ledger identities. Liveness goes in `note`.

### Rebuild the index by MERGING, never by overwriting

> **This is the single most dangerous step in the run.** A clean rebuild from the ledger once produced ~900 company+title keys against the prior index's ~1,000: roughly **100 keys existed only in the index and could not be reproduced from any line in the ledger.** Several were known no-gos. A bare overwrite makes all of them re-surfaceable as net-new.

Safe procedure:

1. Copy the current index to a `PRERUN` file.
2. Rebuild `urls` and `cts` from the ledger, **using the canonical normalizer above.**
3. Apply the **ambiguous-URL drop to the `urls` map only** — drop any normalized URL mapping to 2+ distinct company+title keys. (Without this, one aggregator search URL can collapse dozens of roles onto a single key and dedup silently skips real finds.)
4. Union the prior index's keys back in for anything the rebuild did not produce, skipping URLs the ambiguity rule just dropped.
5. **When keys collide, keep the status that blocks hardest.** `shortlisted`, `near-miss`, `dead` or a rejection beats a bare `skipped`.
6. **Report the carried-forward count every time it is non-zero, and compare it with the expected count** recorded after the last normalizer migration.

**An unexplained carried-forward count is a symptom, not a resolved issue.** It means either lines were deleted from the ledger at some point, or the index has been accumulating keys from a source the ledger never recorded. It deserves an explicit audit rather than another merge.

Then confirm three things: the line count rose, no duplicate URL or company+title was introduced, and **no row with `status: "shortlisted"` lacks a tracker row.**

## STEP 9 — Close

Finish with 3–4 sentences naming the single best opportunity and the recommended first move — usually a warm intro before applying.

**Interviews already in flight outrank new applications for prep hours. Say so when it is true.** A pipeline that keeps generating new applications while live threads go cold is optimising the wrong number. **Name any unsent follow-up drafts too;** they tend to be the standing bottleneck.

Surface any tracker row whose Deadline is within 10 days and whose Status is still "Ready to Apply".

---

## If something is unavailable

**Do not silently skip a step.** If the browser is down, the tracker API fails, or the ledger cannot be read, complete what you can and state plainly in the run file which step did not run and what the consequence is.

**A run that reports a gap is useful. A run that hides one corrupts the dedup state.**
