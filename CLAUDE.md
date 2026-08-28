# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no build step, no package.json.

## Running / testing

There is no build, lint, or test tooling. To run the game:

```bash
open index.html                # macOS, direct file open
python3 -m http.server 8000    # or any static server, then visit localhost:8000
```

There are no automated tests. Verify changes by opening the game in a browser and playing it (movement, rotation, soft/hard drop, line clears, pause, game over/restart).

## Architecture

The entire game lives in three files with no module system — everything in `game.js` is global scope, loaded directly by `index.html` via `<script src="game.js">`.

- `index.html` — DOM structure: the main `#board` canvas (300×600, i.e. `COLS×BLOCK` by `ROWS×BLOCK`), a `#next-canvas` preview, HUD spans (`#score`, `#lines`, `#level`), and a shared `#overlay` used for both pause and game-over states.
- `style.css` — dark/retro arcade visual theme only; no layout logic that `game.js` depends on beyond element IDs.
- `game.js` — all game logic, in this rough top-to-bottom order: constants → board/piece helpers → collision/rotation → line clearing/scoring → rendering → game loop → input handling → `init()`.

Key mechanics to know before modifying logic:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or an index 1–7 into `COLORS`/`PIECES` identifying which piece color occupies it.
- **Pieces**: each of the 7 tetrominoes is a square matrix in `PIECES`. Rotation (`rotateCW`) is a matrix transpose + column reversal, not a lookup table of rotation states.
- **Wall kicks**: `tryRotate()` rotates then tries horizontal offsets `[0, -1, 1, -2, 2]` via `collide()` until one doesn't collide, else the rotation is discarded. This is a simplified kick table, not the SRS spec.
- **Collision** (`collide`): checks board bounds and existing locked cells; used for movement, rotation, and ghost-piece projection — reuse it rather than writing new bounds checks.
- **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time in `dropAccum` and advances the piece one row once `dropAccum >= dropInterval`. Pause/resume works by cancelling/restarting this rAF chain (`animId`), not by gating logic inside the loop.
- **Scoring/leveling**: line clears use `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop scores 2 pts/row, soft drop 1 pt/row. Level increments every 10 lines and recomputes `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece**: `ghostY()` projects the current piece straight down via `collide()` and is drawn at `globalAlpha = 0.2` before the real piece.
- **State**: nearly all mutable game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing vars) lives in module-level `let` bindings reset by `init()` — there is no state container object.

If changing `COLS`, `ROWS`, or `BLOCK`, also update the `#board` canvas `width`/`height` attributes in `index.html` to match (`COLS×BLOCK`, `ROWS×BLOCK`).
