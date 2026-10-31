# Todo / Kanban app

**Status:** exploring
**Started:** 2026-06-30
**Tags:** #vps #fastapi #mysql #side-project

## What is it?
A kanban-style todo app with multiple boards (boards = categories), customizable columns, tags, deadlines, subtasks, and recurring tasks. Single-user, login-protected.

## Why
Want filterable categories + tags + deadlines + kanban, with the ability to zoom into one board (e.g. focus on a single project/subject for a day) or zoom out to see everything at once across boards — useful for things like school where you're juggling multiple subjects/projects.

## Brainstorming
- Boards *are* categories — a task belongs to exactly one board, no separate category concept needed
- Tags are the cross-cutting axis — freeform, persisted, reusable/autocomplete, many-to-many with tasks
- "Everything" view is just a query across all tasks regardless of board_id, same filter/sort options (tag, deadline) as a single-board view
- Subtasks: self-referencing `parent_task_id` on tasks, shown as a checklist inside the parent card rather than their own kanban card (keeps board uncluttered)
- Recurring tasks: template-based rather than a rule on the task itself — a `recurring_templates` table + cron job (already planned for the VPS) checks daily and spawns new task instances when due
- Considered multi-user with full accounts, but decided against it — added complexity (registration, data isolation, password reset flow) not needed for a personal tool. Reverted to single-user login, same pattern as the movie tracker.
- Drag-and-drop kanban is the trickiest frontend piece — worth using a small library (e.g. SortableJS) rather than hand-rolling drag events

## Specs / requirements
- **Frontend:** plain HTML/CSS/JS SPA, served as static files via nginx
- **Backend:** FastAPI + SQLAlchemy, MySQL (shared `small_projects` schema, tables prefixed `todo_*`)
- **Auth:** single user, hashed password (bcrypt/argon2), session cookie or JWT — no user table, no multi-tenancy
- **Columns:** customizable per board (add/rename/remove/reorder), not fixed

### Schema
- `boards` — id, name, created_at
- `columns` — id, board_id (FK), name, position
- `tasks` — id, board_id (FK), column_id (FK), parent_task_id (FK, nullable), title, description, deadline, recurring_template_id (FK, nullable), created_at, completed_at
- `recurring_templates` — id, board_id (FK), title, description, recurrence_rule, next_run_date
- `tags` — id, name
- `task_tags` — task_id, tag_id (junction)

## Open questions
- Password storage: env var / config vs. a single-row settings table (affects whether you can change it without redeploying)
- Recurrence rule format — plain string ("weekly") vs. something more structured (cron-like) for flexibility later
- Do completed recurring task instances stick around for history/stats, or get archived/deleted after some time?
- Any stats/dashboard view worth adding later (e.g. tasks completed per week, overdue count)?

## Links
- SortableJS (drag-and-drop): https://sortablejs.github.io/Sortable/
