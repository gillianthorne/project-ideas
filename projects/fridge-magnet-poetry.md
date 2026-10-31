# Fridge magnet poetry

**Status:** exploring
**Started:** 2026-06-30
**Tags:** #vanilla-js #fun #static

## What is it?
A webpage that recreates fridge magnet poetry — draggable word tiles on a board, arranged into little poems, exportable as an image.

## Why
Recently figured out drag-and-drop for rearranging images and want to apply it to something fun and low-stakes. No need to save/reopen arrangements later — export to image is the end state each time.

## Brainstorming
- Word bank: start from an existing public magnetic-poetry word list, then add personal/relevant words on top
- Kept genuinely simple — no backend, no database. Words live in a plain comma-separated list in a JS/text file, edited directly when adding new words
- **scatterWords():** scatters word tiles randomly around the screen on load, keeping the center clear as open space for arranging words. Basic math checks — e.g. reject/reroll a random position if it falls inside a center exclusion zone (bounding box or radius check), otherwise place it
- **Rotation on drop:** each tile gets a small random rotation (approx -10deg to +10deg from its baseline/unrotated angle) applied when dropped, for a more natural scattered-magnet look
- Export via html2canvas — screenshots a DOM element client-side as an image, no backend rendering needed, single script include, fits the vanilla-JS constraint
- No arrangement-saving needed — simplifies the backend down to basically just "serve the word list"
- Board is a styled div "fridge" background, word tiles are absolutely-positioned draggable elements — reuses the drag mechanic already worked out in a recent experiment

## Specs / requirements
- **Frontend:** plain HTML/CSS/JS, draggable word tiles, export button (html2canvas)
- **Word storage:** a single `words.js` (or `.txt`) file with a comma-separated word list, loaded client-side — no backend, no database
- **Adding words:** just edit the file directly and reload the page
- **scatterWords()** — places word tiles at random positions on load/reset, avoiding a center exclusion zone so there's open space to build a poem
- **Rotation on drop** — on drop, apply a random rotation between approx -10deg and +10deg relative to the tile's baseline angle

### Word list format
```js
const words = "the,quick,brown,fox,jumps,love,forever,whisper,...".split(",");
```

## Open questions
- Which public magnetic-poetry word list to seed from?
- Any styling direction for the "fridge" (literal fridge look vs. plain minimal board)?
- Word tile styling — literal magnet look (rounded, slight shadow) vs. plain text tiles?
- Center exclusion zone for scatterWords() — fixed rectangle vs. radius-based, and how big relative to screen size?

## Links
- html2canvas: https://html2canvas.hertzen.com/
