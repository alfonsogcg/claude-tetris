# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No package.json, build, lint, or tests. README is in Spanish; UI strings are Spanish too.

## Run

Open `index.html` directly, or serve statically: `python -m http.server 8000`.

## Architecture

Three files, all logic in `game.js` (global script, no modules, `'use strict'`):

- `index.html` — DOM ids that `game.js` looks up at load (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`). Renaming an id breaks the game.
- `style.css` — dark theme; overlay visibility toggled via `.hidden` class.
- `game.js` — module-level mutable state (`board`, `current`, `next`, `score`, ...) reset by `init()`. Flow: `loop` (rAF, accumulates `dropAccum` vs `dropInterval`) → `lockPiece` → `merge` + `clearLines` + `spawn`. `spawn` calls `endGame` if the new piece collides.

Non-obvious details:
- Board cells and `PIECES` matrices store color indices 1–7 that index `COLORS`; 0 = empty. Piece type = its color index.
- Rotation (`rotateCW`) works on the matrix; `tryRotate` tries horizontal kicks `[0,-1,1,-2,2]` only (no SRS).
- Pause/game-over both cancel the rAF loop; unpausing restarts it via `togglePause`. `init()` doubles as restart.
- Changing `COLS`/`ROWS`/`BLOCK` requires updating the `<canvas id="board">` width/height in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`). The next-piece canvas is hardcoded 120×120 with block size 30 in `drawNext`.
- Fall speed: `max(100, 1000 - (level-1)*90)` ms; level = `floor(lines/10)+1`. Line scores `[0,100,300,500,800] * level`; soft drop +1/row, hard drop +2/row.
