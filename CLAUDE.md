# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 写代码时必须遵守的准则
多思考，思考之后再写，三思而后行。
尽量追求改动最小，能用一行代码解决的东西不要用两行；
可读性要好；
如果要加新功能，尽量将不同功能的代码模块化保存在不同文件，这样让大模型读取也方便索引

## How to run

Open `index.html` in a browser. No build step, no dependencies.

## Architecture

Single-file vanilla Tetris: `index.html` (~480 lines) — HTML canvas, inline CSS, inline JS.

**Core data:**
- `board` — 20×10 2D array, `0` = empty, color string = locked block
- `current` — active piece `{ shape (2D bool matrix), color, x, y }`
- `PIECES` — 7 Tetromino definitions with colors; rotation is computed via `rotateCW()` (transpose + reverse rows)
- Piece spawns at `y=0`, so it can start partially above the board — collision at `by < 0` during lock = game over

**Key flow:**
1. `tick(now)` (rAF loop) → gravity timer → `lockPiece()` when piece can't move down
2. `lockPiece()` → writes piece to board → `clearLines()` detects full rows → if full: starts 300ms flash animation; if not: `spawnPiece()` immediately
3. `finishClear()` → splices full rows, prepends empty rows, scores, then `spawnPiece()`

**Animation gating:** During line-clear animation (`clearingRows.length > 0`), the game loop skips gravity, ghost piece, and current piece rendering. `hardDrop()` guards with `!current` so it's also blocked during animation (current is null).

**Row removal bug pattern:** When removing multiple rows from the board, splice all rows first in descending order, THEN unshift empty rows. Interleaving splice+unshift corrupts remaining indices because unshift shifts everything up.
