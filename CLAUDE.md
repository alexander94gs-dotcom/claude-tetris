# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris. No build, no dependencies, no package.json. Three files: `index.html`, `style.css`, `game.js`.

## Running

Open `index.html` directly, or serve statically:

```bash
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

No lint/test/build commands exist in this repo.

## Architecture

All game logic lives in `game.js` (~300 lines), organized around a few core data structures and a single RAF loop:

- **Board**: `ROWS × COLS` (20×10) matrix, each cell `0` (empty) or `1–7` (color index of the piece that occupies it).
- **Pieces**: `PIECES` array holds the 7 tetromino shapes as square matrices of color indices; `randomPiece()` picks one and spawns it centered at the top. Rotation is `rotateCW` (transpose + reverse), not a lookup table — there's no distinct rotation-state tracking, just the current shape matrix.
- **Collision** (`collide`): checks board bounds and existing locked cells; used for movement, rotation, and ghost-piece projection.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until one doesn't collide.
- **Game loop** (`loop`, driven by `requestAnimationFrame`): accumulates elapsed time in `dropAccum`; when it exceeds `dropInterval`, advances the piece one row or locks it via `lockPiece` (which merges into the board, clears lines, and spawns the next piece).
- **Line clearing** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows in; awards score from `LINE_SCORES` scaled by `level`, and recomputes `level`/`dropInterval` from total `lines`.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at low alpha.
- **Rendering**: `draw()` clears and redraws the board canvas each frame (grid, locked cells, ghost, current piece); `drawNext()` renders the preview piece on a separate canvas (`next-canvas`).
- **Input**: single `keydown` listener switches on `e.code` for movement/rotation/soft-drop/hard-drop; `KeyP` toggles pause independently of the paused/gameOver guard.

State (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) is held in module-level `let` bindings reset by `init()`, which also wires the restart button and kicks off the RAF loop.

Tunable constants live at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
