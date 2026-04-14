# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Games

No build step or server required. Open any HTML file directly in a browser:

```bash
open shooter.html
open tictactoe.html
```

## Repository Structure

All games are self-contained single HTML files — CSS and JS are embedded inline. No dependencies, no package manager, no bundler.

## shooter.html Architecture

The script is organized into lettered sections (A–P) in order of dependency:

| Section | Purpose |
|---|---|
| A. `CONFIG` (`C`) | All tuning constants (speeds, radii, cooldowns, color palette). Edit here first when balancing. |
| B. Canvas setup | Logical resolution is **480×270**. CSS scales it up to fill the window. All game coordinates are in logical space; mouse coords must be divided by `C.SCALE`. |
| C. Audio | `beep()` drives all sound via Web Audio API oscillators. `SFX.*` are named presets. Audio context is lazy-initialized on first click (browser autoplay policy). |
| D. `Input` | Keyboard state map (`Input.k`) + mouse position (`Input.mx/my`) + `Input.mdown`. `Input.move()` returns a normalized direction vector. |
| E. `Parts` (Particles) | Single pooled array. `Parts.emit()` is the generic emitter; named presets (`muzzle`, `spark`, `death`, `popup`) are convenience wrappers. |
| F. `Shake` | Screen shake via `ctx.translate`. Call `Shake.trigger(intensity, duration)`. Always paired with `Shake.apply(ctx)` / `Shake.restore(ctx)` around the render. |
| G. Sprites | Pixel-art sprites are **2D arrays of palette indices** rendered with `fillRect` via the `spr()` helper. Each logical "pixel" = `ps` units (default 2). `drawPlayer()` and `drawEnemy()` are the top-level draw calls. |
| H. `Player` | Movement, aim angle, walk animation (4 frames), shoot cooldown, invincibility frames. |
| I. `Bullet` | Travels at `C.BSPEED` px/s. `owner` is `'p'` (player) or `'e'` (enemy). Carries a short positional trail array for rendering. |
| J. `Enemy` | Type-driven via `ECFG`. Behaviors: `chase`, `zz` (zigzag), `boss` (orbit + phase 2 at 50% HP). Enemies optionally return a `Bullet` from `update()` when their shoot timer fires. |
| K. `Waves` | Builds a timed spawn queue via `Waves.build(level)`. Boss wave every 5th level. Wave ends when spawn queue is empty **and** `enemies` array is empty. |
| L. Collision | `resolveAll()` handles player-bullets→enemies, enemy-bullets→player, and enemy-contact→player. Circle-circle only. |
| M–N. State machine | Three states: `ST.MENU`, `ST.PLAY`, `ST.OVER`. `newGame()` resets all world state and starts wave 1. |
| O. Render | Each state has its own `render*()` function. `drawCursor()` must be called last in every screen to keep the crosshair on top. |
| P. Boot | `clamp()`, `rnd()`, hi-score localStorage helpers, click dispatcher, fixed-timestep game loop (`FDT = 1000/60`). |

## Key Conventions

- **Coordinate space:** always logical (480×270). Never use `canvas.width`/`canvas.height` for game logic.
- **Delta time:** `update(dt)` receives seconds. Velocities are in px/s (e.g. `C.P_SPEED = 115` px/s).
- **Adding a new enemy type:** add an entry to `ECFG`, define sprite frames + palette arrays, add a branch in `drawEnemy()`, and reference the type in `Waves.build()`.
- **Adding a new sound:** add a method to `SFX` calling `beep(freq, dur, type, vol, freqEndOrNull)`.
- **Color palette:** all colors live in `C.COL`. Do not hardcode colors elsewhere.

## Git Workflow

**Commit and push to GitHub after every meaningful unit of work.** This is a hard requirement — never leave the repo in an uncommitted state at the end of a task. The goal is that the GitHub remote always reflects the latest working state so work can be recovered or reverted at any point.

### Rules
- Commit after each logical change (feature added, bug fixed, tuning tweak applied). Do not batch unrelated changes into one commit.
- Always push immediately after committing — a local-only commit is not a saved version.
- Write commit messages that describe **what changed and why**, not just "update file".

### Commands
```bash
git add <file>          # stage only the changed file(s), never git add -A blindly
git commit -m "verb: concise description of the change"
git push
```

### Good commit message examples
```
fix: crosshair now visible on menu and game-over screens
feat: add fast enemy zigzag behavior starting wave 2
balance: reduce boss shoot interval in phase 2 from 900ms to 450ms
refactor: extract drawCursor() helper used by all render states
```

Remote: `https://github.com/NWT77/pixel-assault`
