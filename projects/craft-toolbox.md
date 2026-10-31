# Craft toolbox (knitting/crochet calculators)

**Status:** exploring
**Started:** 2026-07-02
**Tags:** #vanilla-js #static #crafts #fun

## What is it?
A static web app collecting small, focused calculator tools for knitting/crochet: a gauge calculator, a yarn swap/substitution tool, a yarn matching tool, and room to add more over time.

## Why
Wanted a fun toolbox-style project outside the usual VPS-backend app pattern — genuinely useful for actual craft projects, and each tool is small/self-contained enough to build and ship independently.

## Brainstorming
- Gauge tool: pure numbers-in-numbers-out calculator (input your swatch gauge + pattern's stated gauge, output adjusted stitch/row counts) — no pattern-text parsing/conversion, kept simple for v1
- Yarn swap/matching: match by weight/yardage only for v1 (given a pattern's yarn and a substitute yarn's yardage per skein, calculate skeins needed) — not matching by color or fiber content yet
- Static site, no backend — these are calculation tools, not data-management tools, so there's no server-side logic needed
- Saving data between visits (e.g. a yarn stash) is undecided — if it becomes wanted later, this can still be done client-side via `localStorage` without needing to add a backend
- Toolbox framing means new tools can be added independently over time without restructuring what's already built

## Specs / requirements
- **Frontend:** plain HTML/CSS/JS, static site, no backend
- **Gauge calculator:** inputs (your gauge: stitches/rows per 4in or 10cm; pattern's stated gauge; target measurement), outputs adjusted stitch/row counts
- **Yarn swap/matching tool:** inputs (original yarn's yardage per skein + skeins called for in pattern; substitute yarn's yardage per skein), outputs how many skeins of the substitute are needed
- **Extensibility:** structured so additional calculator tools can be added later without reworking the existing ones

## Open questions
- Whether to save data between visits (e.g. yarn stash, past calculations) — if yes, `localStorage` is the natural fit for a static site
- What other tools belong in the toolbox beyond gauge + yarn swap (e.g. yardage estimator, stash inventory, project tracker)
- Visual/UI direction — single-page with tabs/sections vs. separate pages per tool

## Links
- 
