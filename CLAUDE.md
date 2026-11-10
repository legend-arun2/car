# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Dustline Rally: a single-file 3D off-road racing game (three.js). The whole game is `index.html` (HTML, CSS and one `<script type="module">`). There is no build step, no package manager, no linter and no test suite in the repo. `.nojekyll` and `README.md` are the only other files.

- Live site: GitHub Pages from `main` / root, https://legend-arun2.github.io/car/ . Every push to `main` triggers a "pages build and deployment" run.
- three.js `0.184.0` loads from unpkg through an import map with integrity hashes (`index.html` near line 150). Fonts load from Google Fonts. There is no local copy of three.js, so the game needs internet.

## Running it

Serve the folder over HTTP (opening the file directly can break module loading) and open `index.html`:

```bash
python3 -m http.server 8000
```

## Testing

There are no committed tests. Behaviour has been verified with throwaway headless Playwright scripts that drive the page through the `window.__dbg` hook (last line of the script: `OBST, simulate, game, Track, Car, heightAt, THREE, startRace, showMenu, keys, touch, renderer, camera, scene, dust, skids`). Useful patterns:

- Step the simulation without rendering: `d.startRace()`, then `d.simulate(1/60)` in a loop (about 4 s of steps passes the countdown). Real-time rendering is far too slow under software GL (about 4 fps), so never wait on `requestAnimationFrame` for physics tests.
- Drive the player by setting `d.keys.KeyW/KeyA/KeyS/KeyD/Space` before each `simulate` call. Set `game.player.auto = true` to let the AI drive the player car.
- For a local test copy, replace the `<script type="importmap">` block with one that maps `three` to a local `node_modules/three/build/three.module.js` (the unpkg integrity hashes will not match a local file). Chromium needs `--use-gl=swiftshader --enable-unsafe-swiftshader`.
- To test turning, call `d.OBST.clear()` first, otherwise trees at the road edge cause crashes that look like instability.
- Useful metrics: heading change after holding full lock for 2 s at 100/150/190 km/h, and a crude keyboard-style driver around the lap (distance, time off road, spins).

## Architecture (all in `index.html`, in this order)

Sections are marked with `/* ===== ... ===== */` comments:

1. **Track + terrain**: a closed Catmull-Rom spline sampled into `Track` (`P` points, `T` tangents, `R` right vectors, `K`/`Ks` curvature, `S` distance, `vProf` safe-speed profile). `heightAt(x, z)`, `distAt` and `normalAt` read a height grid built from the track plus noise; road height is interpolated between centre-line samples (stepping it makes the car hop at speed). `surfaceAt` classifies road, loose edge and grass. Trees and rocks go into the `OBST` spatial hash for collisions.
2. **Car model**: `buildCar(color, number)` builds the Rally Supercar from primitives and merges its meshes by material (the original has about 330 meshes per car). Wheels are named groups driven by the physics.
3. **Vehicle physics (`class Car`)**: a custom rigid body stepped at a fixed 240 Hz (`DT`) inside the main loop. Four raycast suspension wheels, a Pacejka-style tyre model with surface-dependent grip, a gearbox and torque curve, body-vs-ground and sphere-vs-tree impulses, and visual dents. **The same `Car.step` drives the player and the AI**, so a physics change affects both and must be checked on both.
4. **Driver assists**: `assistLvl` (0 Pro, 1 Normal, 2 Easy) applies to the human player only (`ai` and `auto` cars use level 0 inside `step`). Easy and Normal turn the steering input into a requested turn rate (`AY_EASY`/`AY_NORMAL`) and push the car toward it with a yaw assist, add grip that builds with speed, bleed off sideways velocity, and apply turn-in scrub. Pro uses direct wheel-angle steering. Tuning constants sit near the top of the vehicle physics section.
5. **AI (`aiControl`)**: follows the centre line with a lane offset, pure-pursuit steering plus a curvature feed-forward, and speed from `Track.vProf` scaled by the difficulty `skill`. If you change grip or aero, re-run an AI-only race: the AI lap times (about 103-107 s on Pro difficulty) are the regression baseline.
6. **Game layer**: `game` object (state machine `menu` / `countdown` / `racing` / `finished`), grid setup, lap and position tracking via `locate()`, car-car collisions, chase camera, HUD and minimap, the racing line mesh (`updateLine`), dust and skid-mark effects, procedural Web Audio (`Sfx`), keyboard and touch input, menu wiring.
7. `frame()` calls `simulate(dt)` (physics and race logic), then `updateLine()` and `updateCamera()`, then renders. Menu choices (assist level, racing line) are saved in `localStorage` under `dl.assist` and `dl.line`, always through the try/catch `store` wrapper.

## Conventions and gotchas

- Coordinate system: car forward is local +x, up is +y, and local +z is the car's right. A right turn is therefore a negative yaw rate about the up axis. Getting this sign wrong inverts a stability control.
- Forces added to `F` after the `this.v.addScaledVector(F, dt / MASS)` line in `Car.step` have no effect; assists applied after that point change `v` directly.
- The game is also published as a private claude.ai artifact (https://claude.ai/artifact/KJpdMtR6Ldx55Cag9SvSb8). That copy is `index.html` without its first three lines (the doctype and the charset and viewport metas, which the artifact wrapper adds). Republish it after meaningful changes.
- Do not change the three.js version without updating both import-map integrity hashes.
