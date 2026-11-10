# Dustline Rally

A single-file 3D off-road rally racing game built with three.js, using the Rally Supercar model.

- Full-physics driving (suspension, tyre slip, gearbox, damage) with driver assists for the player
- 1-5 AI opponents, 1-5 laps, three difficulty levels
- Keyboard and touch controls, procedural engine sound, dust and tyre marks, minimap

## Run

three.js loads from a CDN, so serve the folder over HTTP and open `index.html`:

```bash
python3 -m http.server 8000
```

## Controls

W/S throttle and brake, A/D or arrows steer, Space handbrake, R reset to road, M mute.
