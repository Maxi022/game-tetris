# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris implementation using HTML5 Canvas. No dependencies, no build process, no package.json — just `index.html`, `style.css`, and `game.js`.

## Running / testing

There is no build, lint, or test tooling. To run the game, open `index.html` directly in a browser, or serve it with any static server, e.g.:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There are no automated tests. Verify changes by playing the game in a browser (or via the `run`/`verify` skills if available).

## Architecture

Everything lives in `game.js` (~300 lines, single global scope, no modules). The three files cooperate as follows:

- `index.html` — DOM structure: `#board` canvas (300×600, the 10×20 grid at `BLOCK=30`px/cell), `#next-canvas` (piece preview), HUD spans (`#score`, `#lines`, `#level`), and the `#overlay` used for both PAUSE and GAME OVER states.
- `style.css` — dark/retro visual theme only, no layout logic of consequence beyond flexbox panel layout.
- `game.js` — all game logic, structured around a few core concepts:

**Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or an integer 1–7 indexing into `COLORS`/`PIECES` (piece color/type).

**Pieces**: the 7 tetrominoes are defined as square matrices in `PIECES`. Rotation (`rotateCW`) is a transpose + row reversal, not a table of rotation states. `tryRotate` applies `rotateCW` then attempts wall kicks at offsets `[0, -1, 1, -2, 2]`, using `collide` to test each.

**Collision** (`collide`): checks board bounds and cell occupancy for a given shape at a given offset. Used by movement, rotation, ghost-piece projection, and spawn checks.

**Game loop** (`loop`, driven by `requestAnimationFrame`): accumulates elapsed time in `dropAccum`; once it exceeds `dropInterval`, the current piece drops one row (or locks if blocked), then `draw()` renders the frame.

**Locking / clearing**: `lockPiece` → `merge` (bakes the current piece into `board`) → `clearLines` (scans bottom-up, splices full rows, unshifts empty ones at top, tracks `cleared` count) → `spawn` (promotes `next` to `current`, generates a new `next`, and calls `endGame` if the new piece immediately collides).

**Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row. `level` increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.

**Rendering**: `draw()` clears and redraws grid, locked board, the ghost piece (`ghostY()` projects the current piece straight down, drawn at `globalAlpha=0.2`), and the current piece, in that order. `drawNext()` renders the preview canvas similarly.

**Input**: a single `keydown` listener handles arrows (move/soft-drop), Up/X (rotate), Space (hard drop, with `preventDefault`), and P (pause, which short-circuits everything else and toggles the animation frame loop).

All of this is orchestrated by `init()` (resets state, starts the loop) and re-invoked by the restart button's click handler — there's no other entry point.

## Tunable constants (in `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS×BLOCK` × `ROWS×BLOCK`).
