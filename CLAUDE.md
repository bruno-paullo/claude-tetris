# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A Tetris-style game in vanilla JavaScript + HTML5 Canvas. No dependencies, no `package.json`, no build step, no linter, no test suite. The README and all user-facing UI text are in Spanish; keep new UI strings in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder statically (preferred):

```bash
python -m http.server 8000   # then open http://localhost:8000
npx serve .
```

There are no automated tests; verify changes by playing the game in the browser.

## Architecture

Three files: `index.html` (DOM: `#board` canvas, side panel with `#score`/`#lines`/`#level`, `#next-canvas`, and a shared `#overlay` used for both PAUSA and GAME OVER), `style.css`, and `game.js`, which holds all logic as top-level functions over module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, …). `game.js` is loaded as a classic script at the end of `<body>` and grabs DOM elements by id at load time.

Key conventions in `game.js`:

- **Cell values double as piece type and color index.** `PIECES[t]` matrices are filled with the value `t`, and `COLORS[t]` is its color; index 0 is empty/null. `board` stores these same values once a piece is merged. Types 1–7 are the standard tetrominoes; 8–10 are pentominoes (+, U, Y), 11 is the 1×1 single, 12 the hollow 3×3. Adding a piece type means extending both arrays and the selection logic in `randomPiece()`.
- **Piece selection:** `randomPiece()` returns the single if `rewardPending` (set by `clearLines()` on a 4-line clear, reset in `init()`); otherwise it rolls `HOLLOW_CHANCE` / `PENTO_CHANCE` (picking from `PENTOMINOES`) before falling back to a uniform standard piece. `makePiece(type)` builds a piece at the spawn position. All matrices must stay square and ≤4×4 for `drawNext()`.
- **Pieces are square matrices** rotated by `rotateCW` (transpose + reverse); `tryRotate` applies simple horizontal wall kicks `[0, -1, 1, -2, 2]` (not SRS).
- **`collide(shape, x, y)` is the single source of truth** for movement, rotation, ghost projection (`ghostY`), drops, and game-over detection in `spawn()`. Cells with `y < 0` are allowed (above the board).
- **Lock pipeline:** `lockPiece()` → `merge()` → `clearLines()` (updates lines/score/level/`dropInterval`) → `spawn()` (promotes `next`, detects game over, redraws next preview).
- **Game loop:** `loop()` runs on `requestAnimationFrame`, accumulates `dt` into `dropAccum`, and auto-drops when it exceeds `dropInterval`. Pause/game-over work by `cancelAnimationFrame(animId)`; `init()` fully resets state and is also the restart button handler.
- **Scoring:** `LINE_SCORES[cleared] * level`; hard drop +2/cell, soft drop +1/row. Level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)`.

## Coupled values

- `COLS × BLOCK` and `ROWS × BLOCK` must match the `#board` canvas `width`/`height` in `index.html` (300×600).
- `drawNext()` assumes a 4×4 grid of 30px cells, matching the 120×120 `#next-canvas`.
- The controls list in `index.html` and the README controls table should stay in sync with the `keydown` handler.
