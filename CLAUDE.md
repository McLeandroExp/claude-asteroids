# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

There is no build, lint, or test tooling: no `package.json`, bundler, or test suite. The game is plain HTML5 Canvas + JavaScript.

- Run: open `index.html` directly in a browser, or serve the folder with `npx serve .` and visit `http://localhost:3000` (the port the README documents).
- Tests / lint: none exist. Verify changes by running the game in a browser.

## Architecture

The whole game is two files:

- `index.html` — an 800×600 `<canvas id="canvas">` plus a `<script src="game.js">` at the end of `<body>`. No modules; `game.js` runs as a classic script.
- `game.js` (~420 lines) — all logic, rendering, and input, in one `'use strict'` file with global mutable state.

### Main loop and state

`loop(ts)` drives everything via `requestAnimationFrame`: it computes `dt` in seconds (clamped to 0.05 s), calls `update(dt)`, then `draw()`. There is no fixed timestep and no separate render state.

Global game state: `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, and `state` (`'playing' | 'dead' | 'gameover'`) with `deadTimer` for the respawn delay. `initGame()` resets all of it; `nextLevel()` resets only bullets/particles/ship and respawns asteroids.

### Update flow

`update(dt)` branches on `state`:

- `gameover`: only particles update; Space calls `initGame()`.
- `dead`: asteroids keep drifting while `deadTimer` counts down, then `ship.reset()` and state returns to `playing`.
- `playing`: fire → update ship/bullets/asteroids/particles → bullet-vs-asteroid collisions → ship-vs-asteroid collision (skipped while `ship.invincible > 0`) → if no asteroids remain, `nextLevel()`.

### Input

`keydown`/`keyup` maintain `keys` (held state). `justPressed` is an edge flag consumed by `pressed(code)`, which returns it once and clears it. Use `pressed()` for one-shot actions (fire, restart) and `keys[...]` for continuous ones (rotate, thrust). The arrow keys and Space have `preventDefault` applied.

### Asteroids

Sizes are 1 (small) to 3 (large), and `RADII`, `SPEEDS`, `POINTS` are indexed by size. Destroying an asteroid awards `POINTS[size]` and calls `split()`, which returns two asteroids of `size - 1` at the same position (none for size 1). Each new asteroid gets a random direction and a randomized polygon, and is drawn as an outline.

### Rendering

Everything is vector-drawn with `ctx` path calls inside `save()`/`restore()` — there are no sprites or image assets. `draw()` paints in this order: particles, asteroids, bullets, ship, HUD, and the GAME OVER overlay. Canvas size is fixed at `W = 800`, `H = 600`.

### Physics and space

Ship, bullets, and asteroids wrap around the edges (`wrap()`, a toroidal playfield); particles do not. Collisions are circle-distance checks. Ship-vs-asteroid uses `ship.radius + asteroid.radius * 0.82`. New levels spawn `3 + level` large asteroids at least 130 px from the center.

## Notes

- All player-facing text is in Spanish (HUD, overlay, README). Keep new UI strings consistent with that.
- `README.md` is partly stale: it still mentions power-ups and a shooting-star asteroid, which were removed (commit `13e713f`). Trust `game.js` over the README for game behavior.
