# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Browser-based retro arcade games — single self-contained HTML files with no build step, no dependencies, and no external assets. Everything (HTML, CSS, JS, sprites) lives in one file per game.

## Running / Testing

Open any `.html` file directly in a browser:

```bash
# Windows
start shooter.html

# Or double-click the file in Explorer
```

There is no build process, bundler, package manager, or test suite. Changes are visible immediately on browser refresh.

## Architecture: shooter.html

The game uses a flat script with sections separated by banner comments (`// ─── Section ───`). Execution order matters — classes are defined before use.

**State machine:** `MENU → PLAYING → LEVEL_COMPLETE → PLAYING` (repeat) or `→ GAME_OVER → MENU`

**Section order in the script:**

1. **Constants** — `PLAYER_SPEED`, `BULLET_SPEED`, `LEVEL_CONFIG[]` (one entry per level, drives all difficulty scaling)
2. **`Input`** — Singleton. Tracks `keys{}`, `mouse{x,y,down,clicked}`. Mouse coords are CSS-scaled to canvas space via `(clientX - rect.left) * (canvas.width / rect.width)`.
3. **`Particles`** — Singleton pool (max 200). Call `Particles.burst()` or `Particles.muzzleFlash()` for effects; `update(dt)` + `draw()` each frame.
4. **`Bullet`** — Owns `owner` (`'player'` or `'enemy'`), `dmg`, `radius`. Marks itself `dead` when out of bounds or lifetime expired.
5. **Enemy hierarchy** — `Enemy` (base) → `Grunt`, `Shooter`, `Tank`, `Boss`. Each subclass overrides `update(dt, player, bullets)` and `draw()`. `Enemy.takeDamage()` calls `die()` which spawns debris particles.
6. **`Player`** — Handles movement, aiming, shooting, invincibility frames, and hit flash. Fires by checking `Input.mouse.down` against `fireCooldown` each frame.
7. **`LevelManager`** — Builds and shuffles a spawn queue from `LEVEL_CONFIG`, drains it via `spawnTimer`. Sets `this.complete = true` when queue is empty and `enemies.length === 0`.
8. **`Game`** — Top-level orchestrator. Owns `bullets[]`, `screenFlash`, and the state machine. Collision is handled here (`circlesOverlap`), not in individual classes.
9. **Helpers** — `circlesOverlap(a, b)` and `drawBackground(tint)` at the bottom.

**Rendering pipeline (back to front):** background grid → enemies → player → bullets → particles → HUD → state overlays → screen flash.

**Delta time:** capped at 50ms (`Math.min(dt, 0.05)`) to prevent tunneling after tab-switch.

## Key Conventions

- **Sprites are pure `fillRect`** — no images, no canvas paths beyond `arc` for bullets. Rotation uses `ctx.save() / ctx.translate() / ctx.rotate() / ctx.restore()`.
- **`LEVEL_CONFIG[]`** is the single source of truth for difficulty. To tune a level, edit its entry — don't scatter magic numbers elsewhere.
- **Dead entities** are marked with a `dead` flag and filtered out at the end of `Game.updatePlaying()`, not removed mid-loop.
- **All coordinates are in canvas space** (800×600). The canvas is CSS-scaled to fit the viewport; always use the scaling formula in `Input` when reading mouse position.

## Git Workflow

After every code change: `git add <file>`, commit with a descriptive message, and `git push`.

```bash
git add shooter.html
git commit -m "feat: short description of what changed"
git push
```

Remote: https://github.com/diethrv/starfall-shooter
