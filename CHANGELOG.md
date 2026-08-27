# Changelog

## 2026-08-27 — Claude Opus 5 revision

Tuned and hardened against roughly two months of daily production runs. Every change below came from a failure that actually happened.

### Breaking

- **Tracker moved from Google Sheets to a Notion database.** The Sheet and its companion `seen_jobs.json` are retired. Placeholders change: `{TRACKER_SHEET_ID}` and `{TRACKER_TAB_GID}` → `{TRACKER_DB_ID}`.
- **Added an append-only dedup ledger** (`ledger.jsonl` + `ledger-index.json`) alongside the tracker. The tracker holds only roles worth acting on; the ledger holds everything ever evaluated, including rejects, so a dropped role is never re-surfaced as net-new.
- **The two stores must be reconciled in both directions at the start of every run.** The ledger→tracker direction was missing initially and cost a week: dozens of researched, comp-verified roles had no pipeline row and were invisible to the run.

### New rules

- **Geography is now a hard drop keyed on work arrangement, not headquarters.** Onsite/hybrid must be in a target metro; fully remote is fine anywhere; **"remote eligible" is not remote.** Previously a flag, which meant a well-matched req in the wrong metro consumed a pick slot every run.
- **Rejected reqs block at req level, not company level.** A company that rejects you isn't banned; the specific req is, permanently, including reposts. Match reposts on normalized company+title, not req ID — a title reworded from "Technical Program Manager, X" to "Research Program Manager, X" on the same team at the same level is the same req.
- **A rejection in the ledger or registry outranks the tracker's silence.** Never overturn a rejection-skip because no tracker row exists.
- **Added a `Closed` status** distinct from `Not Interested`: the posting URL no longer resolves to the original req. No judgement about the candidate, and no registry entry. Filing closed reqs as "Not Interested" understates how many expire before you act.
- **Added a six-rung failure-escalation ladder** for boards that stop responding, with an explicit HTTP-code triage table. See below for why.

### Fixes that came from real breakage

- **Pagination caps.** One board's API caps at 10 postings per response regardless of the `num` parameter, while reporting the true total in `count`. The old loop requested 100, got 10, saw `len < 100`, stopped, and **reported success while reading about 2% of the board for weeks.** Now: step by the observed page size and assert `fetched == count`.
- **HTTP codes mean different things and conflating them hides real gaps.** `401/404/422` is a config bug (fix the call, don't retry on a schedule); `403` is blocked (try headers, then the browser); **`429` means the endpoint is correct and reachable** — never log it as dead; `200 with 0 rows` is worse than an error because it looks like a clean sweep.
- **Wrong endpoint path recorded as a refusal.** One employer was logged as "refusing" for two weeks. The path was simply wrong. One browser network trace found the right one in minutes. This is why rung 2 of the ladder exists.
- **Front-end ATS ≠ backing ATS.** An employer was recorded as "not a Workday tenant" because its search layer runs on a different platform. Its Apply button pointed at Workday the whole time. **Check where Apply points.**
- **A missing `User-Agent` header made an entire ATS return zero jobs silently**, indistinguishable from a clean sweep.
- **Ambiguous URLs corrupt dedup.** One aggregator search URL collapsed dozens of distinct roles onto a single key, so real finds were silently skipped. The index rebuild now drops any normalized URL mapping to 2+ distinct company+title keys.
- **Index rebuilds must merge, never overwrite.** A clean rebuild once produced ~900 company+title keys against the prior index's ~1,000. Roughly 100 keys existed only in the index and could not be reproduced from the ledger; several were known no-gos. A bare overwrite would have made all of them re-surfaceable.
- **Liveness annotations must not go in the ledger `title` field.** Doing so changes the dedup key and forks one role into several ledger identities.
- **Web search returns dead reqs.** A fallback surfaced a Director role with a real posted band; the employer's own page said the post no longer exists. Every fallback hit now requires a liveness check.
- **Titles lie about level in both directions.** A req titled "Sr. Staff" whose body said "manager of a lean team of 3–4" was dropped by a Director+ title filter. Read the body before ranking on the title.

### Contacts

- **Prefer 1st-degree connections who previously worked with you.** Search pattern: company filter + 1st-degree filter + past-employer name as keyword. This routinely beats a 2nd-degree contact with a better title.
- **Record dead routes** — non-repliers and people you've asked not to contact — in the tracker's Warm Path field, and never resurrect them.
- **Re-verify contacts older than a few weeks.** A referral plan built on someone who has since left is worse than no plan, because it looks researched.
- **Comp questions go to the recruiter**, never the hiring manager or a peer contact.

### Reporting

- Run files now report **geography drops and rejected-req drops on their own lines, with counts**, so it stays visible whether new filters cut the right things.
- Runs must state **whether any role arrived from outside the system entirely.** If roles keep arriving through personal alerts rather than the sweep, company coverage is too narrow and the fix is more tokens, not fewer.
- **Interviews in flight outrank new applications for prep hours**, and the run says so when it's true. A pipeline generating new applications while live threads go cold is optimising the wrong number.
