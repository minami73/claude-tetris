# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

No build, no install, no test suite, no linter. Vanilla HTML/CSS/JS, zero dependencies.

Run the game:
```bash
open index.html        # macOS
xdg-open index.html    # Linux
start index.html       # Windows
```

Or serve locally (needed if testing anything that requires an HTTP origin):
```bash
python3 -m http.server 8000
# or
npx serve .
```
Then open `http://localhost:8000`.

To verify a change works, open `index.html` in a browser and play — there is no automated test to run instead.

## Architecture

Three files, all logic lives in `game.js` (~300 lines, single script, no modules):

- `index.html` — DOM shell: `<canvas id="board">` (300×600, the 10×20 grid at `BLOCK=30`px/cell), a side panel (`score`/`lines`/`level` spans, `next-canvas` preview), and an `#overlay` div reused for both PAUSE and GAME OVER states.
- `style.css` — dark/retro arcade visuals only.
- `game.js` — state, game loop, rendering. No classes; module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, ...) hold all game state.

Key mechanics in `game.js`:
- **Board**: `ROWS × COLS` matrix, each cell `0` (empty) or `1–7` (piece color index into `COLORS`/`PIECES`).
- **Pieces**: square matrices in `PIECES`. Rotation (`rotateCW`) is a transpose, not a lookup table of rotation states.
- **Collision** (`collide`) and **wall kicks** (`tryRotate`, tries offsets `[0,-1,1,-2,2]`) are the two functions that gate all piece movement — touch both together when changing movement rules.
- **Game loop** (`loop`): `requestAnimationFrame`-driven, accumulates `dt` and drops the piece when `dropAccum >= dropInterval`. `togglePause` cancels/restarts this loop rather than gating inside it.
- **Line clear / scoring** (`clearLines`): classic table `LINE_SCORES = [0,100,300,500,800] * level`; level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece**: `ghostY()` projects the drop position; drawn via the same `drawBlock` with `alpha=0.2`.

Tunable constants sit at the top of `game.js` (`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`). If `COLS`/`ROWS`/`BLOCK` change, the `<canvas id="board">` `width`/`height` in `index.html` must be updated to match (`COLS×BLOCK` by `ROWS×BLOCK`) — they are not computed dynamically.
