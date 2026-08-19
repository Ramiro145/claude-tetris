# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build process, no package.json.

## Running the game

There is no build/lint/test tooling. To run:

```bash
start index.html       # Windows: open directly in browser
# or serve locally (recommended for consistent behavior):
python3 -m http.server 8000
npx serve .
```

Then open `http://localhost:8000`. There are no automated tests in this repo — verify changes by playing the game in a browser.

## Architecture

Three files, no modules/bundler:

- `index.html` — DOM structure: `<canvas id="board">` (300×600, the main playfield), `<canvas id="next-canvas">` (next-piece preview), score/lines/level panel, and a pause/game-over overlay.
- `style.css` — dark/retro arcade visual theme.
- `game.js` (~300 lines) — all game logic, single file, no classes.

### Core model (`game.js`)

- Board is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- Pieces are defined as square matrices in `PIECES`; rotation is done via transpose + row-reverse (`rotateCW`), not precomputed rotation states.
- `collide(shape, ox, oy)` checks board bounds and overlap with locked cells — used both for movement and rotation.
- `tryRotate()` implements basic wall kicks: on collision, retries the rotated shape shifted ±1 and ±2 columns before giving up.
- `ghostY()` projects the current piece straight down to compute the ghost-piece landing row (drawn at `globalAlpha = 0.2`).

### Game loop

`loop(ts)` runs via `requestAnimationFrame`, accumulating elapsed time in `dropAccum` and dropping the piece one row once `dropAccum >= dropInterval`, then redrawing (grid → locked board → ghost → current piece).

Key derived values:
- Level increases every 10 lines cleared; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- Scoring uses `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by current level; hard drop adds 2 points/cell traveled, soft drop adds 1 point/row.
- A newly spawned piece that immediately collides (`spawn()`) triggers `endGame()`.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell size in px), `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).

## Controls

`←`/`→` move, `↑` or `X` rotate CW, `↓` soft drop, `Space` hard drop, `P` pause.
