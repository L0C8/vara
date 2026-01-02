# vara design doc

## Vision & Goals
- Build a fast, responsive “boomer shooter” with semi-true 3D geometry (rooms with height, ramps, stairs) while keeping sprites for characters/props.
- Deliver crunchy, low-latency combat on modest hardware; prioritize tight controls over visual complexity.
- Target a vertical slice: one map, one enemy type, one hitscan weapon, pickups, doors, HUD/audio, and a settings menu with resolution options (default 640x480).

## Pillars
- Immediate response: steady 60+ FPS at 640x480; low input latency; readable feedback.
- Clarity: distinct silhouettes, high-contrast sprites, consistent collision using rectangular shapes for actors.
- Extensibility: clean module seams (rendering, gameplay, input, audio, UI, assets) so new enemies/items/maps are incremental additions.
- Tool-first: custom mapmaker (exports OBJ + metadata) plus hot-reload for rapid iteration.

## Tech Targets
- Language/Platform: C++17, desktop (Linux/macOS/Windows).
- Framework: raylib for windowing, input, audio, 3D rendering, and billboards.
- Rendering: 3D geometry with raylib meshes for level, billboards for dynamic sprites; software-light shading kept simple (vertex lights or single directional light).
- Audio: raylib audio or miniaudio via raylib; stereo panning by angle/distance.
- Build: CMake; CI runs unit tests plus headless asset/map validation.

## Scope / Non-Goals (v1)
- In-scope: 3D level geometry (axis-aligned rooms, ramps, stairs), doors, pickups, sprite enemies, hitscan/projectile weapons, distance-based shading.
- Out-of-scope (v1): skeletal animation, complex PBR, network multiplayer, scripting languages; no physics beyond simple collisions.

## Engine Architecture
- Core: Platform init, main loop, timing, logging, memory helpers.
- Renderer: Level mesh loading/drawing, billboard system for sprites, depth-aware sprite sorting, simple lighting, resolution scaling/output.
- Assets: Loader for maps, meshes, textures, sprite sheets, audio; optional packer.
- Gameplay: World state (player, entities, items), collision, damage/combat, door/trigger logic, pickups, HUD state.
- AI: Per-enemy finite state machine (idle/alert/chase/attack/search) with LOS checks and simple navigation (grid/waypoint).
- Audio: Sound registry, channel management, 2D spatialization.
- Input: Key bindings, per-frame snapshot (pressed/held), mouse look/turn.
- UI: HUD, message feed, settings menu (resolution, sensitivity, volume).
- Tooling: mapmaker app (exports OBJ mesh + metadata), CLI map validator, hot-reload.

## Game Loop (per frame)
1) Poll input → build `InputState`.
2) Update: player movement, collisions, doors/triggers, AI, projectiles, combat resolution, audio events; fixed or clamped timestep.
3) Render: draw level geometry; draw billboards for enemies/props/projectiles; draw HUD/UI; present at selected resolution (with scaling).
4) Mix audio and submit frame.

## World Representation
- Level geometry: 3D mesh constructed from grid/brush data; stored as sectors/rooms with floor/ceiling heights; exported to raylib mesh or loaded from glTF/OBJ with metadata.
- Collision: Axis-aligned rectangular prisms for walls/geometry; actors use rectangular footprints (rect extruded in Y) for collisions; projectiles use spheres/rays.
- Cells/Rooms: For AI and triggers, maintain a coarse grid or waypoint graph for navigation and “hearing” logic.
- Doors: Modeled as animated meshes or moving quads; track open fraction and collision blocking.

## Rendering Pipeline
- Framebuffer/resolution: Render at selected internal resolution (640x480 default; others selectable in settings) and scale to window using nearest or integer scaling.
- Level: Draw opaque geometry first; simple vertex colors or single directional light; optional lightmaps baked per level.
- Billboards: Use raylib billboards for enemies, items, and effects; sort by depth; support simple animation frames and directional variants (front/side/back) if provided.
- Particles: Optional lightweight billboard particles for impacts/muzzle flashes.
- HUD/UI: Screen-space sprites/text drawn after 3D pass.

## Gameplay Systems
- Player: WASD + mouse/keyboard turn; sprint optional; collision uses rectangular footprint and step-up on low obstacles.
- Weapons: Hitscan (ray test vs level + actors) and projectile types; weapon states (idle, firing, cooldown); muzzle flashes and sounds.
- Enemies: Billboard visuals with rectangular collision; FSM with LOS/hearing; melee or ranged variants; telegraph attacks via animation/sfx.
- Pickups: Health, armor, ammo, keys; optional respawn flag.
- Doors/Triggers: Use key to interact; triggers on use/enter/kill-all; can open doors, spawn enemies, toggle blockers, play sounds.
- Difficulty scaling: Adjust enemy damage/aggro, resource availability.

## Physics/Collision
- 3D, simplified: sweep tests for rectangular actors against static colliders; slide along surfaces; step height threshold to allow stairs.
- Projectiles: Ray/sphere sweep per tick; spawn impact effects; respect level collision and actors.
- Hitscan: Ray tests against level BSP/mesh and actor bounds; apply damage to closest hit.

## Audio
- Channel-based playback; panning/attenuation by angle/distance.
- Priority: weapon/enemy cues override ambience when limited.

## UI/UX
- HUD: health, armor, ammo, weapon icon, keys, crosshair.
- Messages: pickups, objectives.
- Menus: start/pause/options; options include resolution list, fullscreen toggle, mouse sensitivity, audio volumes.

## Tooling & Pipeline
- Mapmaker: Custom editor (top-down/orthographic) to paint sectors/brushes, set floor/ceiling heights, ramps, doors, triggers, waypoints, and place entities. Exports:
  - `level.obj`: render mesh with named groups per sector/material.
  - `level.meta.json`: sectors (heights/material ids), collision volumes (AABBs), doors (open dir/speed), triggers (volumes + actions), entities (type/pos/facing/params), waypoints/nav graph, materials list.
- Validator: `map-validate <mapfile>` checks geometry integrity, collisions, entity references, trigger wiring.
- Asset bundling: Convert textures/sprites to GPU-ready formats; optional pack file for release.
- Hot-reload: In dev mode, reload textures/maps on file change; reset to nearest spawn to continue testing.

## Debugging/Instrumentation
- Overlays: FPS, frame time histogram, player pos/angle, nav grid/waypoints.
- Logging: AI transitions, collision rejects, trigger activations.

## Performance Targets
- 60 FPS at 640x480 on mid-tier CPU/GPU with typical scene load.
- Keep draw calls low via batched level mesh and grouped billboards; minimize allocations (use pools).

## Save/Load & Config
- Save: player state, door/trigger state, enemy states, RNG seed.
- Config: keybinds, mouse sensitivity, audio volumes, resolution, fullscreen, difficulty.

## Security/Robustness
- Validate map/entity indices; clamp coordinates; reject oversized assets; clear error logs when failing load.
- Deterministic update ordering to aid reproducibility.

## Milestones
1) Bootstrap: window, input, fixed timestep, clear color; render a test cube level and one billboard.
2) Movement/collision: player controller with rectangular collision, stairs/ramps.
3) Combat: hitscan weapon, one enemy FSM, pickups, HUD basics.
4) Rendering polish: animated billboards, lighting pass, impact particles.
5) Audio pass: SFX, music, panning.
6) Triggers/doors: interactions, keys, spawn triggers.
7) Settings: resolution/fullscreen menu; save/load config.
8) Tooling: mapmaker MVP (export/import), map validator, hot-reload, perf cleanup.
