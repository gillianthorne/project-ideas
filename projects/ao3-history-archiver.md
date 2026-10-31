# AO3 history archiver

**Status:** idea (planned start: ~late November 2026)
**Started:** 2026-10-29
**Tags:** #nodejs #sqlite #scraper #ao3 #side-project

## What is it?
A tool that logs in to AO3, walks through your reading history, downloads every fic as an EPUB, and writes its metadata to a SQLite database. Meant as a personal offline archive of everything you've read.

## Why
AO3 is currently down for an outage (expected 2-4 weeks). Wanted a way to have everything already read saved locally for situations like this, rather than being locked out of it.

## Decisions so far
| Question | Decision |
|---|---|
| File format | EPUB and HTML (HTML added so a v3 in-app reader has something to render) |
| Run mode | Full run first, then incremental re-runs |
| Language | Node.js |
| Database | SQLite |
| Login | Username/password in a git-ignored config or env file |
| Where it runs | Locally first, maybe VPS later |
| Sharing | Yes, designed so a friend can run it with her own AO3 login |
| Authors, fandoms, warnings, categories | Each is many-to-many with fics (a fic can have several, and each can appear on many fics), so each gets its own table plus a linking table |
| Tag storage | Relationships, characters, and additional tags each get their own separate table (three tag tables, not one shared tags table), and each is many-to-many with fics via its own linking table |
| Rating | Stored on the fic itself |
| Fics primary key | The AO3 work id itself (matches the EPUB file naming) |
| Series | A series table, with a series id and an index (position in the series) on the fic, so one series per fic |
| Column naming | Plain names (e.g. `name`, `description`), not prefixed |
| Schema pattern | Plural table names, linking tables named after the pair (e.g. fic_authors) with a composite primary key and ON DELETE CASCADE on both sides |
| Chapters | Chapters posted so far plus the planned total (empty when AO3 shows "?") |
| Extra dates | Also store when a fic was first downloaded and when it was last re-downloaded/checked |
| Series table columns | Name, is_complete, and total |
| EPUB path | Stored on the fic |
| Authors identified by | Both pseud and username are stored |
| Series primary key | The AO3 series id |
| Authors table shape | Two tables: accounts (usernames) and pseuds linked to an account |
| Accounts key | Auto-increment id, with the username as a changeable column, so an AO3 username change only edits one cell (this replaced keying accounts by username) |
| Rename detection | The scraper flags a possible rename in the end-of-run report (e.g. a new account whose pseud matches an existing one); fixing it is done by hand |
| Account username | UNIQUE |
| Lookup tables | Fandoms, warnings, categories, relationships, characters, and additional tags each get an auto-increment id and a UNIQUE name |
| Fandom links | Characters and relationships each link to fandoms through a linking table (character_fandoms, relationship_fandoms), so one can belong to several fandoms |
| Canonical tag grouping | Possibly a v1.1 side project: capture the full grouping (meta tags and sub tags) on a separate git branch alongside the app. Not part of v1 |
| Fic links | Seven linking tables (fic_pseuds, fic_fandoms, fic_warnings, fic_categories, fic_relationships, fic_characters, fic_additional_tags). Authors' credit order is not kept; the relationship, character, and additional tag links store a position number to keep AO3's listed order |
| Pseuds key | Auto-increment id, plus the account id and the pseud name (both required); the same pseud name can exist under different accounts |
| Series defaults and deletion | is_complete defaults to 0 (not complete); deleting a series keeps its fics and just empties their series id; a pseud name can't appear twice under one account |
| Series required columns | Name and is_complete are required; total is NULL for ongoing series so it displays as "/?" |
| Fic author link | Fics link to pseuds (the credited name); each pseud links to its account |
| Anonymous and orphaned works | Stored as normal entries (an "Anonymous" pseud, an "orphan_account" username), no special cases |
| Resume marker contents | Replaced the "page number only" idea, since a page number from the front shifts as you read more. Runs go oldest first, so progress is counted from the end of the history: the page from the end plus the last fic finished, with fallbacks in case that fic was re-read and moved (the fic finished before it, its last-visited date, and a fics-from-the-end count) |
| Resume marker table | One table (run_progress), not one per run type: every run is incremental, because even the first run will need to stop and resume over several days. Rows are kept after a run finishes so the next run knows where to start |
| Run types | run_progress has one row per run_type, with run_type as the key: 'full_check' (walking the history; update checks never touch it) and 'update_check' (overwritten every time an update check runs) |
| Update check | A periodic run that re-checks unfinished works only: chapters posted below chapters planned, or planned is empty ('?'). No separate table for this; it compares against the columns already on fics |
| Request delay | Adaptive: starts conservative and slows down after any refusal from AO3 |
| Starting delay | 8 seconds between requests (web searches suggest AO3's limit is around 5-6 seconds, so this starts above it); still adaptive, and tuned by testing on a small batch |
| On "Retry later" | Retry a few times with growing delays, then save progress (the resume marker) and take the long break below |
| After retries run out | The run saves progress, sleeps for a long break (e.g. hours), then resumes itself, since a full run is expected to take multiple days |
| Failures treated differently from "Retry later" | A single fic's page being gone or forbidden (already logged in unavailable_fics, run continues), login expired or rejected mid-run, and the site being down or in an outage (server errors) |
| Login expired mid-run | Try logging in again once; if that fails, save progress and stop with a clear message (no long break) |
| Site down or server errors | Hard stop: save progress and exit, with no retries and no long break or self-resume |
| Long break length | A config setting; the default is picked later |
| Delay speeding back up | No: the delay never speeds back up on its own. Instead the scraper records the delay it was using and how many fics it got at that delay, so you can compare which delays work better |
| Delay stats | A table in the SQLite database (delay_stats): requests made, refusals, and time spent at a delay. Every request counts, so an EPUB download and an HTML download are two. Each time the delay info changes a new row is started, so the same delay can appear on several rows. Each row also records when the stretch started. The delay is stored in whole seconds |
| Retry count | 5 retries on a refused request, then stop and save progress |
| Retry wait | Start with the wait time AO3 gives plus a buffer of about 3 seconds; if none is given, start at 1 minute. Double the wait on each further retry |
| Slowdown after a refusal | The normal delay between requests doubles |
| Run cap | No cap; a run goes until the history is done |
| Visit history | Latest values only (last visited, visit count); no per-run snapshots |
| Extra AO3 details | None beyond the metadata listed above (no language, kudos, hits, bookmarks, or comment counts) |
| v1 scope | Scraper and database only |
| v2 direction | A desktop app to browse and filter the archive |
| v2 toolkit | Electron (JavaScript, HTML, CSS) |
| Opening a fic in v2 | Just show its details and file location; no built-in reader |
| v1 scope | The downloader and database only. Browsing/filtering and searching are a separate later project |
| EPUB path type | Relative to the configured download folder, so it also works on a friend's machine |
| Date format | ISO 8601 text (e.g. 2026-10-30 14:55:00) for every date column |
| Summary format | The HTML exactly as AO3 shows it (keeps italics, links, paragraphs); the v2 app will need to render it |
| Rating format | Text, exactly as AO3 words it (e.g. "Teen And Up Audiences") |
| Required fics columns | Title, word count, chapters posted, AO3 created date, first downloaded, and last checked (plus the work id primary key) must always have a value. EPUB path and HTML path used to be required but are now optional (see "Download order and failures") |
| Download order and failures | For each fic, the EPUB and HTML are fetched together, back to back (which goes first doesn't matter). If one fails after retries, the fic is still saved with its metadata and the file that worked; the failed file's path stays empty. Both paths are optional so fewer requests are wasted and AO3 sees less scraping |
| Logging missing files | A fic with a missing file is also logged in unavailable_fics as download_failed, purely for record keeping (how much fails, and how often) |
| Going back for missing files | Its own periodic pass, like the update check, with its own run_type ('file_retry') in run_progress; overwritten each time it runs |
| Finding fics with a missing file | A query on fics for an empty epub_path or html_path. No separate has_epub / has_html table, to avoid data getting out of sync. Every fic gets a fics row even if nothing was downloaded |
| Chapters planned | Optional, empty when AO3 shows "?" |
| Series id and position | Optional, since most fics aren't in a series |
| HTML files | Stored in a separate folder from the EPUBs (one folder for EPUBs, one for HTML), both named by work id; the fics table has an HTML path column, optional like the EPUB path |
| Optional fics columns | Rating, summary, AO3 last-updated date, last visited, and visit count can be empty |
| Missing required data | Skip the fic and record it in the unavailable fics table with the reason |
| Unavailable fics table | One row per work: auto-increment id with the AO3 work id as a UNIQUE column. Repeated failures update the row (fail count, last fail) rather than adding rows |
| Unavailable fics reason | A fixed set (deleted, locked, missing_data, download_failed) plus a free-text details column; both describe the latest failure |
| Unavailable fics dates | first_fail, last_fail, and a resolved_at that stays empty until the fic downloads successfully. Resolved rows are kept; if a resolved fic fails again, resolved_at is cleared and the count keeps going |
| EPUB file naming | AO3 work id only |
| Detecting updates | Re-download if either the chapter count or the last-updated date changed |
| Unavailable fics (deleted, private, locked) | Skip them, record them in their own separate table with a reason, and print a list at the end of the run |
| Resuming an interrupted run | Save a progress marker (e.g. history page number) in a small table in the same SQLite database, and resume from it |

## Metadata to store
- **Core:** title, rating, word count, chapters posted so far and planned total (empty when AO3 shows "?")
- **Series:** series name, whether it's complete, and its total, plus the fic's position within it
- **Many-to-many with fics:** authors (both pseud and username), fandoms, warnings, categories (M/M, F/M, etc.)
- **Tags (each in its own table, many-to-many with fics):** relationships, characters, additional tags
- **Summary text**
- **History info:** last visited, visit count
- **Dates:** AO3 created and last updated (needed for the incremental behaviour below), plus when it was first downloaded and when it was last re-downloaded/checked

## Incremental behaviour
- Re-runs should overwrite the stored file for a fic that has the same AO3 id and now has more chapters, and update its metadata
- Created and last-updated dates are stored so it's possible to tell whether a fic has changed since it was last downloaded

## Later (separate project, not v1)
- A small app of its own to browse and filter the archive (by fandom, relationship, rating, word count, etc.)
- Searching summaries and tags

## v3 (idea only)
- A fic reader built into the desktop app, rendering the downloaded HTML files
- This is why HTML is downloaded alongside the EPUB in v1

## v2 (scoped, not built yet)
- A desktop app (Electron) to browse and filter the archive, reading from the SQLite database and the stored EPUBs
- Selecting a fic shows its details and where its EPUB file is; there's no built-in reader
- v1 already stores what this needs: fic metadata, authors (pseuds and accounts), fandoms, warnings, categories, the three tag types, series, dates, summary, and the EPUB path
- Other ideas that came up but weren't chosen for v2 so far: searching inside fics' text, "updated since I last read it" tracking, reading stats over time, and personal ratings or notes. If personal ratings or notes are ever added, they should live in their own table so incremental runs don't overwrite them
- Reading stats over time would have needed per-run snapshots of visit count and last-visited, which v1 deliberately doesn't keep

## Sharing with a friend
- Each person runs it on their own machine with their own git-ignored config file holding their own AO3 login, so no credentials are ever shared
- Her downloads and database are completely separate from yours, and her requests come from her own account and IP
- Implications: download folder and database location must be configurable (no hard-coded paths), a setup guide is needed (install Node, create the config file, run it), and error messages should be friendly (e.g. wrong password)

## Things to keep in mind
- AO3 offers official per-work downloads (EPUB, HTML, PDF, MOBI), which is gentler than scraping the reader page and gives EPUBs directly
- Downloading both EPUB and HTML means two download requests per fic, which roughly doubles the request count and makes the delay and retry settings matter more
- History is behind a login, so credentials must stay out of git
- AO3 rate-limits aggressive requests ("Retry later" errors), so a big history means a slow crawl with delays and the ability to resume
- Worth reading AO3's terms regarding automated access before building
- I couldn't find AO3's exact rate limits documented anywhere reliable, so the starting delay should be conservative and tuned by testing on a small batch
- SQLite doesn't enforce foreign keys (so ON DELETE CASCADE does nothing) unless `PRAGMA foreign_keys = ON` is set on each connection

## Open questions
- Canonical/meta tags (v1.1 idea): scraping each tag's page to record what it falls under (e.g. a character-has-PTSD tag falling under a disability grouping). Check a tag page once AO3 is back to see what it shows. Idea so far: look in the database before fetching a tag so only new tags cost a request. Whether it runs in the main run or a separate pass is undecided until it's clear how it works
- Fics that get updated on AO3 but that you don't re-read don't move in your history, so a marker based on history position alone won't notice them. Decided: a periodic update_check pass re-checks unfinished works only (see the Decisions table). Still open: how often it runs and how it is started. The first run is expected to take multiple days (over 100 history pages, possibly a decade of reading)
- How exactly the scraper decides something is a "possible rename" for the end-of-run report
- Where the EPUB files live on disk (naming is decided: AO3 work id)
- Rate-limit specifics still open: the default length of the long break (it will be a config setting), and which other settings are configurable for your friend's copy. Starting delay (8 seconds), retry count (5), retry wait (AO3's own plus about 3 seconds, else 1 minute, doubling), and the slowdown (double) are decided
- v2 desktop app: which SQLite library it uses to read the database, which filters and views you want first, and whether your friend gets it too (which would mean packaging it as an installer)
- Sharing: how to package it for your friend (git repo she clones, zip, etc.) and how much setup help the README should give

## Links
- AO3: https://archiveofourown.org/
