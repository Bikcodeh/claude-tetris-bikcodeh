# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A falling-block puzzle game in vanilla JavaScript + HTML5 Canvas. No dependencies, no `package.json`, no build step, no test suite, no linter. User-facing text (UI labels, README) is in Spanish.

## Running

```bash
open index.html                 # open directly
python3 -m http.server 8000     # or serve statically → http://localhost:8000
```

There are no automated tests; verify changes by playing in the browser.

## Architecture

All game logic is in `game.js` as top-level functions sharing module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`). `init()` resets all of it and is also the restart handler.

- **Board model**: `ROWS × COLS` matrix; `0` = empty, `1–7` = piece type, which also indexes `COLORS` and `PIECES` (both have `null` at index 0). Piece shapes store their type number in their cells, so merging a shape into the board preserves color.
- **Movement/rotation**: every move goes through `collide(shape, x, y)`. `tryRotate()` uses `rotateCW` (transpose + reverse) and tries horizontal kicks `[0, -1, 1, -2, 2]`.
- **Piece lifecycle**: `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next` to `current`; if the new piece collides immediately, `endGame()` runs.
- **Game loop**: `loop(ts)` via `requestAnimationFrame` accumulates elapsed time in `dropAccum` and applies gravity when it exceeds `dropInterval`. Pause/game over call `cancelAnimationFrame(animId)`; resuming resets `lastTime` and calls `loop` directly.
- **Scoring/levels**: `LINE_SCORES[cleared] * level`; hard drop +2/cell, soft drop +1/row. Level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 − (level − 1) × 90)`, recalculated only inside `clearLines()`.
- **Rendering**: `draw()` redraws the whole board every frame (grid → locked cells → ghost at alpha 0.2 → current piece). `drawNext()` renders into the separate `#next-canvas` (4×4 cells of 30px) and is only called from `spawn()`.
- **HUD/overlay**: DOM elements in `index.html` (`#score`, `#lines`, `#level`, `#overlay`) are updated via `updateHUD()` and the overlay's `hidden` class.

## Gotchas

- `COLS`, `ROWS`, `BLOCK` must stay consistent with the `<canvas id="board">` `width`/`height` attributes in `index.html` (`COLS × BLOCK` by `ROWS × BLOCK`).
- Collision allows `y < 0` (cells above the board), but `merge()` doesn't guard against negative rows.
