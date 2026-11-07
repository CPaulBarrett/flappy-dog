# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Flappy Dog 3D: a single-file browser game (`index.html`). There is no build step, package manager, linter or test suite. three.js (0.160.0) is loaded as an ES module from the jsDelivr CDN via an import map, so the page needs network access.

## Running

Serve the folder and open it in a browser (module scripts are more reliable over HTTP than `file://`):

```
python3 -m http.server 8000   # then open http://localhost:8000/
```

Port 8000 may already be taken by a long-running server on this machine that serves this folder.

## Architecture

All code is one inline `<script type="module">` in `index.html`. The key design point: **gameplay runs in the original 2D logical plane, and three.js only renders it.**

- **Logic space** is 400×600 with y pointing down (`W`, `H`). `dog`, `cols`, `GROUND`, gravity, collision and scoring are all in these units, unchanged from the original 2D canvas game. The helpers `toX`/`toY` map logic coordinates to world coordinates (centered, y up), and the play lane is the plane `z = 0`.
- **Collision is 2D only.** The dog's hitbox is tested against column x-ranges and the gap in logic space. Column depth (`COL_DEPTH`) and the dog's voxel thickness are purely visual.
- **`update(dt)`** advances the state machine (`'ready'` → `'play'` → `'over'`) and the physics. **`draw(dt)`** syncs three.js objects from the logic state, scrolls the ground dashes and parallax blocks, moves the camera, renders, then updates the HTML HUD.
- **Columns:** each logical column `{x, top, passed}` owns two meshes (`top3`, `bot3`). `spawn()` creates them, and they must be removed from the scene whenever a column is dropped (in `update` and `clearCols`). `fillCols()` keeps columns spawned out to `SPAWN_X` at exactly `SPACING` apart. `SPAWN_X` is further right than the visible frame because the angled perspective camera sees past the logical right edge.
- **Dog:** built from the pixel-art strings `SPR_HEAD`/`LEGS_A`/`LEGS_B` via `buildVoxels`, which makes one `InstancedMesh` per color. `voxelsFor()` decides which depth layers each sprite pixel occupies (legs and ears on both sides, eyes on the outer layers, tail centered). The two leg groups alternate visibility for the run animation.
- **HUD:** HTML `<div>`s overlaid on the canvas. Positions come from the original canvas y-values converted to percentages (`hudTop`), and font sizes use `cqh` units so they scale with the 2:3 frame.
- **Sound** is synthesized with the Web Audio API (`tone`/`sfx`). There are no asset files.
- The best score is stored in `localStorage` under `flappyDogBest`, with reads and writes wrapped in try/catch.

## Repo

Pushed to the private GitHub repo `CPaulBarrett/flappy-dog` (branch `main`).
