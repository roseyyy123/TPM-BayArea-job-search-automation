# Search queries (template)

The daily run reads this file for where and how to sweep. Keep it current; every fix you discover belongs here so the next run doesn't rediscover it.

## Workday tenants
One row per target employer on Workday. A wrong site token returns 422 or 404 and looks like an outage.

| Company | tenant | host | site | Last verified |
|---|---|---|---|---|
| Example Co | exampleco | wd5 | ExampleCareers | YYYY-MM-DD |

## Not on Workday
Employers you checked that publish elsewhere, and where (Greenhouse, Ashby, SmartRecruiters, Eightfold, their own site).

## Other ATS endpoints
Per-employer notes for Ashby, SmartRecruiters, Eightfold and any custom careers API: the working URL, page size, and anything that silently returns zero.

## Browser-only careers sites
For each site that can't be swept from code: the search URL with your target offices, how long to wait for it to render, how to pull job IDs from the page, and the list of queries to run every time.

## Keyword queries
The title queries the sweep runs, e.g. "director technical program manager", "head of technical program management", "senior director program management".

## Sources cut
Boards you stopped sweeping, when, and why. Don't reinstate one without evidence.
