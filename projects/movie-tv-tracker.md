# Movie & TV watch tracker

**Status:** exploring
**Started:** 2026-06-30
**Tags:** #vps #fastapi #trakt #side-project

## What is it?
A single-page app that recreates an old movie-tracking spreadsheet — total watch time, movie count, busiest day of week, etc. — but with Trakt API autofill on add, plus markdown reviews/notes per watch, and TV episode support.

## Why
Loved this as a spreadsheet years ago. Want it back as a proper app, sitting on vm666 alongside the other small projects, with auto-filled metadata instead of manual entry.

## Brainstorming
- Switched from OMDb to the Trakt API — Trakt is built specifically for movie/TV *watch history* (not just metadata), so it natively supports proper season/episode-level TV data, which OMDb only handled awkwardly via the `series_id` workaround
- Trakt uses OAuth 2.0, not a simple API key — requires registering a free app at trakt.tv for a Client ID + Client Secret
- Authorization Code flow is the right OAuth fit here — this is a web app accessed via browser from any device, not a CLI/limited-input tool, so Device Code flow (meant for TVs/CLI without a convenient browser) doesn't apply. Requires registering a `redirect_uri` pointing at the VPS domain, and a FastAPI route to handle the callback. Still a one-time personal authorization, not a multi-user login flow.
- Access tokens expire after 7 days; refresh tokens are single-use, so the app needs to store the new access/refresh token pair every time it refreshes rather than reusing a static credential
- Public/search endpoints only need the API key (Client ID); anything tied to personal history/ratings needs the OAuth access token too
- Trakt's terms require attribution/branding compliance and reasonable API caching — avoid excessive repeated calls for the same data
- Rewatches = new `POST /sync/history` entries on Trakt, not overwrites, so stats count them properly — this is Trakt's own behavior, no need to replicate it locally
- Stats are computed from Trakt's `GET /sync/history` responses (paginated, filterable by date range/type) — total minutes, count, day-of-week breakdown, busiest month, average rating — rather than local SQL queries

## Specs / requirements
- **Frontend:** plain HTML/CSS/JS SPA, served as static files via nginx
- **Backend:** FastAPI (thin layer over the Trakt API — auth/token handling, proxying requests, computing stats from returned data — not a data owner)
- **Auth (app):** single user, hashed password (bcrypt/argon2) + signed session cookie or JWT for accessing your own app — separate from the Trakt OAuth tokens
- **Movie/TV data + watch history:** Trakt API is the source of truth. Logging a watch = `POST /sync/history`. Stats/history = `GET /sync/history` (paginated, filterable by type and date range) — day-of-week/total-time/etc. stats computed from what Trakt returns, not from a local table
- **Notes/reviews:** Trakt's Sync API supports private notes attached to history items/movies/shows/episodes — worth using instead of a local notes column, pending confirming exactly how that endpoint behaves
- **Local database:** likely not needed for the core feature set, since Trakt already stores everything above. Only a candidate if something ends up needing local caching (e.g. reducing API calls) or a feature Trakt doesn't support
- **Credential storage:** Client ID/Secret + current access/refresh token pair need secure storage (env var/config, not hardcoded), with refresh logic that overwrites both tokens on every refresh

### Schema sketch
No local schema needed for v1, pending confirmation that Trakt's notes endpoint covers reviews adequately. If a local DB ends up needed (e.g. for caching), revisit then rather than building it preemptively.

## Open questions
- Confirm how Trakt's private notes endpoint works in practice (scope, length limits, whether it supports markdown) before relying on it for reviews
- Any stats beyond the classics (total time, count, day-of-week) worth tracking — longest streak? most-watched genre?
- Episode-level vs. season-level granularity for TV stats?
- Whether any local caching/storage ends up necessary once building starts (e.g. to avoid excessive Trakt API calls, per their "cache responsibly" terms)

## Links
- Trakt API docs: https://docs.trakt.tv/docs/getting-started
- Trakt app registration: https://app.trakt.tv/settings/apps/api/new
