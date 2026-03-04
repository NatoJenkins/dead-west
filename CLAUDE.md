# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Games

No build step or server is required. Open either file directly in a browser:

- `game.html` — Dead West (Wild West top-down shooter)
- `index.html` — Tic-Tac-Toe

For quick local testing via CLI: `start game.html` (Windows) or `open game.html` (macOS).

## Git Workflow

After every change, commit and push immediately so work is never lost and any mistake can be reverted:

```bash
git add <changed-file>
git commit -m "Short description of what changed"
git push
```

Never batch unrelated changes into one commit. Each logical change gets its own commit. This ensures `git revert` or `git checkout HEAD~1 -- <file>` can restore any previous working state cleanly.

## Architecture

Both games are **single-file, zero-dependency HTML5 applications** — all CSS, JS, and markup in one `.html` file. No npm, no bundler, no external libraries.

### game.html — Dead West

Canvas-based game rendered at 900×650px on a single `<canvas id="c">`.

**State machine** (`STATES`): `MENU → PLAYING ↔ PAUSED`, `PLAYING → LEVEL_COMPLETE → PLAYING`, `PLAYING → BOSS_INTRO → PLAYING`, `PLAYING → GAME_OVER`.

**Class hierarchy** (all in global scope):
- `Game` — owns all game state: `player`, `enemies[]`, `bullets[]`, `particles[]`, `boss`, `level`, `score`. Drives the update/draw loop and input handling.
- `Player` — WASD/arrow movement, mouse-aimed dual revolvers, power-up state.
- `Enemy` — five types (`grunt`, `outlaw`, `sniper`, `brute`, `dynamite`), each configured via `ENEMY_CONFIGS`. AI behavior varies per type (chase, keep-distance, arc-lob).
- `Boss` — spawns every 5th level. Three phases with different attack patterns (spiral, shotgun spread, rapid + aimed). Phase transitions trigger screen shake.
- `Bullet` — shared by player and all enemies; `isExplosive` flag adds gravity arc.
- `PowerUp` — five types (`heal`, `speed`, `rapidfire`, `shield`, `damage`) with timed durations in `POWERUP_DURATION`.
- `Particle` / `Tumbleweed` — visual effects only.

**Game loop**: `requestAnimationFrame` → `update(dt)` → `draw()`. Delta time (`dt`) is capped to prevent spiral-of-death on tab blur.

**Persistence**: High score saved to `localStorage` key `'deadWestHS'`.

**Enemy scaling**: `pickEnemyType()` changes the spawn pool based on `this.level`. Boss levels are multiples of 5 (`isBossLevel()`). Enemy count per level = `level * 3 + 2`.

### index.html — Tic-Tac-Toe

DOM-based (no canvas). AI uses unmodified **minimax** (perfect play). Two modes: PvP and vs AI. Scores persist in JS variables only (reset on page reload).
