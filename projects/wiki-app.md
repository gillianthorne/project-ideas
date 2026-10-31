# Wiki app (from scratch)

**Status:** exploring
**Started:** 2026-07-02
**Tags:** #fastapi #wiki #accounts #side-project

## What is it?
A self-hosted wiki app built from scratch (no Wiki.js or similar), with markdown pages, free-form wiki-style linking ([[Page Name]]), page revision history with restore and diff, and role-based accounts (admin, editor, viewer). Login-only access, not exposed to the outside world beyond that.

## Why
Want a wiki tailored to personal/small-group needs without adopting an existing wiki platform. Multiple real users from the start, not just a single-user tool like most of the other projects.

## Brainstorming
- **Storage decision:** database-stored content with a revisions table, not git-backed markdown files. Revision history is simpler as a table (`page_revisions` with page_id, revision_number, content, edited_by, timestamp) than replicating git functionality via shelling out or a library. Permissions also tie in naturally when content and accounts live in the same database, rather than syncing a git repo against a separate permissions DB. Content is still authored/stored as markdown text either way — this is a storage decision, not a format decision.
- Concurrent edits: a DB-row approach (edit lock or last-write-wins) is simpler to reason about than git merge conflicts, appropriate for a small (2-5 person) wiki
- Revision history needs both restore (roll a page back to an old version) and diff (compare two versions) — not just passive viewing
- Free-form wiki-style linking ([[Page Name]]) means parsing markdown for link syntax and resolving/creating pages on the fly, rather than a rigid nested-category structure
- Roles: admin/editor/viewer confirmed as the three tiers. Whether per-page permission overrides are needed (e.g. a viewer-only page within an otherwise editor-accessible wiki) is still undecided — affects whether permissions are a simple role column vs. a separate per-page permissions table
- "Locked to the outside world" = login only (username/password), not network-level restriction (no VPN/IP allowlist requirement)

## Specs / requirements
- **Backend:** FastAPI + SQLAlchemy, MySQL
- **Frontend:** plain HTML/CSS/JS, markdown rendering client-side or server-side (TBD), diff view for comparing revisions
- **Accounts:** multi-user from the start, three roles (admin, editor, viewer), login-only access (no anonymous viewing)
- **Pages:** markdown content, free-form [[Page Name]] linking
- **Revisions:** every edit creates a new revision row; support viewing history, diffing between any two revisions, and restoring an old revision as the new current version

### Schema sketch
- `users` — id, username, password_hash, role (admin/editor/viewer), created_at
- `pages` — id, title, slug, current_revision_id (FK), locked_by (FK to users, nullable), locked_at (nullable), created_at
- `page_revisions` — id, page_id (FK), revision_number, content (markdown), edited_by (FK to users), created_at
- `page_links` — page_id (FK), linked_page_id (FK, nullable if target doesn't exist yet), link_text — for resolving/tracking [[Page Name]] links, including links to not-yet-created pages
- `page_permissions` — page_id (FK), user_id (FK), permission_level (view/edit) — overrides the user's global role for this specific page; absence of a row means the global role applies

## Key decisions (resolved)

| Question | Decision |
|---|---|
| Per-page permission overrides | Needed — global role is a default, per-page overrides possible |
| Markdown rendering | Server-side (Python) — centralizes sanitization |
| Links to non-existent pages | Prompt before creating (not silent auto-create) |
| Edit conflict handling | Hard lock — one editor at a time per page |
| Search (v1) | Title/link-based only, no full-text search |

## Links
- 
