# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Classic Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no build step, no bundler, no package.json, no test suite.

## Running the game

There is no build/lint/test tooling. To run:

```bash
start index.html       # Windows: open directly in a browser
# or serve it locally (recommended so canvas/assets behave like production):
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

Then verify changes by opening the page in a browser and playing — there are no automated tests.

## Architecture

Three files, no modules/imports — everything in `game.js` runs in global scope against elements looked up by ID from `index.html`.

- `index.html` — DOM shell: `<canvas id="board">` (300×600, i.e. `COLS×BLOCK` by `ROWS×BLOCK`), a side panel (`#score`, `#lines`, `#level`, `#next-canvas`), and a shared `#overlay` used for both PAUSE and GAME OVER.
- `style.css` — dark/retro visual theme only; no layout logic depends on it.
- `game.js` — all game logic, in one file:
  - **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece type filled it.
  - **Pieces**: `PIECES` are square matrices (see the array in `game.js`); `rotateCW` rotates via transpose + row-reverse.
  - **Collision**: `collide(shape, ox, oy)` checks board bounds and overlap with locked cells.
  - **Wall kicks**: `tryRotate()` rotates then retries at x-offsets `[0, -1, 1, -2, 2]` before giving up.
  - **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulates elapsed time in `dropAccum`, and drops the piece one row (or locks it) once `dropAccum >= dropInterval`.
  - **Locking/clearing**: `lockPiece()` → `merge()` writes the piece into `board`, then `clearLines()` sweeps bottom-to-top removing full rows and unshifting empty ones.
  - **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` × `level`; hard drop adds 2 pts/row, soft drop 1 pt/row. Level increments every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
  - **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2`.
  - All game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) lives in module-level `let` bindings reset by `init()`.
  - Input is a single `keydown` listener switching on `e.code` (arrows, `KeyX` rotate, `Space` hard drop, `KeyP` pause); the restart button calls `init()` directly.

If you change `COLS`, `ROWS`, or `BLOCK` in `game.js`, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS×BLOCK` and `ROWS×BLOCK`).

The README (in Spanish) documents controls, scoring rules, and tunable constants in more detail — check it before re-deriving that information.
