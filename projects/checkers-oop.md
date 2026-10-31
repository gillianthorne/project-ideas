# Checkers (OOP practice)

**Status:** exploring
**Started:** 2026-06-30
**Tags:** #learning #python #oop #game

## What is it?
A checkers game, built as a project specifically to relearn classes/objects (haven't done much OOP in years). Local single-player/two-player to start, with online synchronous play as a stretch goal.

## Why
Wanted a project to relearn OOP concepts without an "in over my head" feeling — board games have natural class boundaries (Board, Piece, Player, Game) and checkers has a smaller move-validation surface than chess, so the rules logic doesn't swallow the OOP practice. Games in general appeal more than simulations or tools for this one.

## Brainstorming
- Considered blackjack and a text-based RPG first — neither felt fun enough to stay motivated through
- Considered chess, but checkers keeps things simpler while still allowing a clean Piece/King polymorphism example (king overrides valid_moves() to allow backward moves)
- Python chosen over Java/C#/JS — already comfortable with it from FastAPI work, so syntax won't compete with the OOP concepts for attention
- Could treat this as part of a small ladder (tic-tac-toe → Connect 4 → checkers) if extra warmups are wanted before diving in, but checkers alone is a reasonable starting scope
- **Stretch goal: online synchronous play** — a genuinely separate skill (state sync over network, likely WebSockets) from the OOP practice itself. Designing `Game` with a clean state export (`to_dict()`/`from_dict()` or similar) now means multiplayer later is "send that state over a WebSocket" rather than a rewrite. FastAPI has built-in WebSocket support, so this could live on the VPS alongside the other apps eventually.

## Specs / requirements
- **Language:** Python
- **Scope (v1):** local two-player checkers, graphical via a local web app — FastAPI backend serving game state as JSON, HTML5 canvas + JS frontend for rendering the board and handling click/move input. Chosen over Pygame (didn't like the game-loop/sprite style) and Tkinter, since checkers is turn-based rather than real-time, and this doubles as groundwork for the multiplayer stretch goal.
- **Stretch goal (v2):** online synchronous play via WebSockets (FastAPI has built-in support) — natural extension of the v1 web app rather than a rewrite

### Class design
- `Board` — 8x8 grid, tracks piece positions, knows how to render itself
- `Piece` — color, position, valid_moves() — base class for a regular piece
- `King` — subclass of `Piece`, overrides valid_moves() to allow backward movement
- `Player` — color, name, tracks captured pieces
- `Game` — owns a `Board` + two `Player`s, manages turn order, move validation/execution, win condition; state export methods (to_dict()/from_dict()) to set up for the multiplayer stretch goal

## Open questions
- Standard checkers rules vs. forced-jump/tournament rules?
- If pursuing the multiplayer stretch goal: how much of the VPS FastAPI/WebSocket setup to reuse vs. treat as a separate mini-project?

## Links
- 
