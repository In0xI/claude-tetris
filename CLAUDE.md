# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript (ES6+), HTML5 Canvas, and CSS. No dependencies, no build step, no `package.json`. Just three files: `index.html`, `style.css`, `game.js`.

## Running the game

There is no build/test/lint tooling. To run it, just open `index.html` in a browser, or serve it with any static server, e.g.:

```bash
python3 -m http.server 8000
# or
npx serve .
```

Then open `http://localhost:8000`.

## Architecture

All game logic lives in `game.js` (~300 lines, single file, no modules). Key pieces:

- **Board model**: `board` is a `ROWS × COLS` matrix (20×10). Each cell is `0` (empty) or an integer 1–7 identifying which piece color occupies it (indexes into `COLORS`/`PIECES`).
- **Pieces**: the 7 tetrominoes are hardcoded as square matrices in `PIECES`. Rotation (`rotateCW`) is a generic transpose + row-reverse, not per-piece rotation tables.
- **Collision** (`collide`): checks a shape against board bounds and already-locked cells given an offset.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` and takes the first that doesn't collide.
- **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time (`dropAccum`) and advances the piece down one row once `dropInterval` is exceeded; otherwise calls `lockPiece()`.
- **Locking/clearing** (`lockPiece` → `merge` + `clearLines` + `spawn`): `clearLines` scans bottom-up, splicing out full rows and unshifting empty ones at the top.
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/cell, soft drop 1 pt/row. Level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2`.
- **Rendering**: all Canvas 2D — `draw()` renders the grid, locked board, ghost piece, and current piece onto `#board`; `drawNext()` renders the next-piece preview onto a separate small canvas (`#next-canvas`).
- **Game over**: triggered in `spawn()` when a freshly spawned piece immediately collides; shows the overlay via `endGame()`.
- All game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing vars) is held in module-level `let` bindings, reset by `init()`.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell pixel size), `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `width`/`height` attributes of `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).

## Conventions

- Comments in `README.md` and code are in Spanish; keep that consistent if adding docs/comments in this repo.
- No semicolon-less style — code uses semicolons and `'use strict'`.
- Keep it dependency-free: don't introduce a bundler, framework, or `package.json` unless explicitly asked.
