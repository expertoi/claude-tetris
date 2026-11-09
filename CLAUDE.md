# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A block-stacking puzzle game in vanilla JavaScript + HTML5 Canvas. No dependencies, no `package.json`, no build step, no linter, no test suite. User-facing text (UI labels, overlay messages, README) is in Spanish — keep new UI strings in Spanish.

## Running

Open `index.html` directly, or serve the directory statically (preferred):

```bash
python -m http.server 8000   # then open http://localhost:8000
npx serve .
```

There are no automated tests; verify changes by playing in the browser.

## Architecture

Three files: `index.html` (DOM: `#board` canvas, side panel with `#score`/`#lines`/`#level`/`#next-canvas`, and a single `#overlay` reused for both pause and game over), `style.css` (dark theme), and `game.js` (all logic, a single non-module script with global state).

Key points in `game.js`:

- **State is module-level `let` globals** (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `lastTime`, `dropAccum`, `dropInterval`, `animId`), all (re)initialized in `init()`. `init()` is also the restart handler.
- **Board** is a `ROWS × COLS` matrix of color indices (`0` = empty, `1–7` = piece type). Piece shapes in `PIECES` store their own type index as the cell value, so the same number indexes `COLORS` and `PIECES`.
- **Rotation**: `rotateCW` (transpose + reverse) and `tryRotate` with simple horizontal kicks `[0, -1, 1, -2, 2]` — not SRS.
- **Piece lifecycle**: `lockPiece()` → `merge()` → `clearLines()` (scoring, level, `dropInterval`) → `spawn()` (promotes `next` to `current`; calls `endGame()` if the new piece collides immediately).
- **Loop**: `requestAnimationFrame`-driven `loop(ts)` accumulates `dt` into `dropAccum` and applies gravity when it exceeds `dropInterval`; `draw()` redraws everything each frame (grid, board, ghost at alpha 0.2, current piece). `drawNext()` is only called from `spawn()`.
- **Scoring**: `LINE_SCORES[cleared] * level`, +1 per soft-drop row, +2 per hard-drop row. Level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 − (level − 1) × 90)`.

## Coupling to watch

- Changing `COLS`, `ROWS`, or `BLOCK` requires updating the `#board` canvas `width`/`height` in `index.html` (`COLS×BLOCK` × `ROWS×BLOCK`). The next-piece preview assumes a 4×4 grid of 30px cells on a 120×120 canvas.
- Pause/resume works by cancelling and restarting the rAF loop and resetting `lastTime`; any new timed behavior must respect that.

## Known issues in current code

- Resuming from pause (`togglePause`) does not re-add the `hidden` class to `#overlay`, so the pause overlay stays visible.
- `endGame()` cancels `animId`, but when it's triggered from inside `loop()` (gravity → `lockPiece` → `spawn`), `loop` schedules a new frame right afterward, so the loop keeps running after game over.
