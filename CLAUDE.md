# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla JS (ES6+). No dependencies, no bundler, no tests, no linter, and not a git repo. README and in-game text are in Spanish; keep UI strings and comments in Spanish for consistency.

## Running

Open `index.html` in a browser, or serve locally: `npx serve .` (http://localhost:3000).

## Architecture

All logic lives in `game.js` (loaded by `index.html`, which hosts an 800x600 `<canvas id="canvas">`). It is a single script organized in banner-commented sections: Input, Utils, Bullet, Asteroid, Ship, Particle, game state, Update, Draw, main loop.

- **Game loop**: `loop(ts)` calls `update(dt)` then `draw()` each `requestAnimationFrame`. `dt` is in seconds and clamped to 0.05 s. All speeds/timers are per-second values scaled by `dt`.
- **State machine**: global `state` is `'playing' | 'dead' | 'gameover'`. `dead` is a 2 s respawn pause (asteroids and particles keep moving); `gameover` waits for Space to call `initGame()`. Game state (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`) is module-level `let` globals, not encapsulated.
- **Input**: `keys[code]` holds held state; `pressed(code)` is a consume-once "just pressed" check (used for Space shooting/restart, so holding Space does not auto-fire).
- **World**: toroidal. Every entity wraps position with `wrap(v, max)` using `W`/`H`.
- **Entities**: classes with `update(dt)`, `draw()`, and a `dead` flag. Dead entities are removed by filtering arrays after updates, not by splicing mid-iteration.
- **Asteroids**: size 1/2/3 indexes parallel arrays `RADII`, `SPEEDS`, `POINTS` (index 0 is an unused placeholder). `split()` returns two asteroids of `size - 1`. Smaller asteroids give more points (size 3 = 20, 1 = 100). Level N spawns `3 + N` large asteroids outside a 130 px safe radius from center.
- **Collision**: circle checks via `dist()`. Ship vs asteroid uses `ship.radius + a.radius * 0.82` and is skipped while `ship.invincible > 0` (3 s after any `Ship.reset()`, shown as blinking).
- `nextLevel()` triggers when `asteroids.length === 0`, and resets the ship (including invincibility) and clears bullets and particles.
- README lists "power-ups" and a "shooting star" asteroid; neither is implemented in `game.js`.
