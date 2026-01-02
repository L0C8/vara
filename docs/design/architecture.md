# Architecture & Implementation Guide

This document explains how the codebase is organized for the raylib-based boomer shooter, what classes/files exist, and how to extend gameplay (items/enemies) without diving into code yet.

## Planned Files/Directories (paths)
- `vara/CMakeLists.txt`
- `vara/assets/`
- `vara/assets/maps/`
- `vara/assets/textures/`
- `vara/assets/sprites/`
- `vara/assets/audio/`
- `vara/docs/` (design, architecture, mapmaker)
- `vara/src/core/App.cpp`, `App.h`
- `vara/src/core/Config.cpp`, `Config.h`
- `vara/src/core/Clock.cpp`, `Clock.h`
- `vara/src/core/Log.cpp`, `Log.h`
- `vara/src/core/Rng.cpp`, `Rng.h`
- `vara/src/platform/Input.cpp`, `Input.h`
- `vara/src/platform/Window.cpp`, `Window.h`
- `vara/src/render/Renderer3D.cpp`, `Renderer3D.h`
- `vara/src/render/BillboardSystem.cpp`, `BillboardSystem.h`
- `vara/src/render/SpriteAtlas.cpp`, `SpriteAtlas.h`
- `vara/src/render/ScreenScaler.cpp`, `ScreenScaler.h`
- `vara/src/render/DebugDraw.cpp`, `DebugDraw.h`
- `vara/src/world/Level.cpp`, `Level.h`
- `vara/src/world/CollisionWorld.cpp`, `CollisionWorld.h`
- `vara/src/world/TriggerSystem.cpp`, `TriggerSystem.h`
- `vara/src/world/SpawnTable.cpp`, `SpawnTable.h`
- `vara/src/entities/Entity.cpp`, `Entity.h`
- `vara/src/entities/Actor.cpp`, `Actor.h`
- `vara/src/entities/Player.cpp`, `Player.h`
- `vara/src/entities/Enemy.cpp`, `Enemy.h`
- `vara/src/entities/EnemyBrain.cpp`, `EnemyBrain.h`
- `vara/src/entities/Projectile.cpp`, `Projectile.h`
- `vara/src/entities/Item.cpp`, `Item.h`
- `vara/src/entities/EntitySystem.cpp`, `EntitySystem.h`
- `vara/src/entities/EnemyFactory.cpp`, `EnemyFactory.h`
- `vara/src/entities/ItemFactory.cpp`, `ItemFactory.h`
- `vara/src/entities/ProjectileFactory.cpp`, `ProjectileFactory.h`
- `vara/src/gameplay/WeaponSystem.cpp`, `WeaponSystem.h`
- `vara/src/gameplay/DamageSystem.cpp`, `DamageSystem.h`
- `vara/src/gameplay/HUDModel.cpp`, `HUDModel.h`
- `vara/src/gameplay/ObjectiveSystem.cpp`, `ObjectiveSystem.h` (later)
- `vara/src/audio/AudioSystem.cpp`, `AudioSystem.h`
- `vara/src/audio/AudioBus.cpp`, `AudioBus.h` (optional)
- `vara/src/ui/UiRenderer.cpp`, `UiRenderer.h`
- `vara/src/ui/MenuSystem.cpp`, `MenuSystem.h`
- `vara/src/tools/map_validate.cpp`
- `vara/src/tools/mapmaker/` (editor sources and exporters)
- `vara/tests/` (unit/smoke tests)

## Directory Layout (planned)
- `CMakeLists.txt`: build config.
- `assets/`: maps, textures, sprite sheets, audio.
- `src/core/`: app bootstrap (`App`), configuration (`Config`), time (`Clock`), logging (`Log`), random (`Rng`).
- `src/platform/`: raylib init/shutdown, window management, input polling.
- `src/render/`: 3D renderer (`Renderer3D`), level mesh handling, billboard system, materials, post/screen scaling.
- `src/world/`: world state container, level/sector data, collision world, trigger/door logic, save/load helpers.
- `src/entities/`: entity base classes, actor logic, player, enemies, projectiles, items/pickups, factories.
- `src/gameplay/`: weapon system, damage rules, HUD model, progression (keys/objectives), difficulty tuning.
- `src/audio/`: sound registry, playback, panning/attenuation.
- `src/ui/`: HUD drawing, menus (start/pause/settings), messaging.
- `src/tools/`: map validator CLI, mapmaker app (editor UI + exporters).
- `tests/`: unit/smoke tests (map load, config load, collision edge cases).

## Core Loop
`App` owns the main loop:
1) Poll input (platform layer) → build `InputState`.
2) Update (fixed or clamped dt):
   - `World` update (doors/triggers/time events).
   - `EntitySystem` tick (player, enemies, projectiles, items).
   - `WeaponSystem` tick (fire/cooldown/impact).
   - `AudioSystem` commit enqueued sounds.
3) Render:
   - `Renderer3D` draws level mesh.
   - `BillboardSystem` draws sprites (enemies/props/projectiles).
   - `UiRenderer` draws HUD/menus at target resolution.
4) Present and loop until exit.

## Key Systems & Classes

### Core / Platform
- `Config` (src/core/Config.h/.cpp): reads/writes user config (resolution, fullscreen, sensitivity, volumes).
- `Clock` (src/core/Clock.h/.cpp): frame timing; delta clamping.
- `InputState` (src/platform/InputState.h): per-frame buttons/axes; supports rebinding.
- `App` (src/core/App.h/.cpp): orchestrates init, loop, shutdown; owns `World`, `Renderer3D`, `AudioSystem`, `Ui`.

### World & Data
- `Level` (src/world/Level.h/.cpp): geometry data (meshes, sectors, ramps), collision shapes (AABBs), door markers, trigger volumes, waypoints.
- `CollisionWorld` (src/world/CollisionWorld.h/.cpp): queries and sweeps for rectangular actors and projectiles; step-up for stairs.
- `TriggerSystem` (src/world/TriggerSystem.h/.cpp): enter/use/kill-all triggers; actions (open door, spawn enemy, play sound, toggle blocker).
- `SpawnTable` (src/world/SpawnTable.h/.cpp): definitions for entities placed from map data.

### Entities
- `Entity` (base) (src/entities/Entity.h/.cpp): id, type tag, position, bounds, flags, virtual update/draw hooks (or system registration).
- `Actor` (src/entities/Actor.h/.cpp): derives from `Entity`; health/armor, team, movement parameters, state.
- `Player` (src/entities/Player.h/.cpp): input-driven movement, weapon inventory, interact, takes damage, HUD data provider.
- `Enemy` (src/entities/Enemy.h/.cpp): derives from `Actor`; holds `EnemyBrain`.
- `EnemyBrain` (src/entities/EnemyBrain.h/.cpp): FSM (idle/alert/chase/attack/search), LOS/hearing, navigation target; data-driven params.
- `Projectile` (src/entities/Projectile.h/.cpp): moves each tick, collision/hit callbacks, lifetime.
- `Item` (src/entities/Item.h/.cpp): pickup logic (health/armor/ammo/keys); billboard render data.
- `EntitySystem` (src/entities/EntitySystem.h/.cpp): owns collections, spawns/despawns, update order, depth-sort list for billboards.
- `Factory` classes: build entities from data tables (`EnemyFactory`, `ItemFactory`, `ProjectileFactory`).

### Gameplay
- `WeaponSystem` (src/gameplay/WeaponSystem.h/.cpp): weapon states, cooldowns, firing logic; hitscan rays vs level/actors; projectile spawn path.
- `DamageSystem` (src/gameplay/DamageSystem.h/.cpp): apply damage/knockback, death handling.
- `HUDModel` (src/gameplay/HUDModel.h/.cpp): data for UI (health/armor/ammo/keys/messages).
- `ObjectiveSystem` (later): track goals (exit, key doors).

### Rendering
- `Renderer3D` (src/render/Renderer3D.h/.cpp): wraps raylib 3D; camera setup; draws level mesh; applies simple lighting.
- `BillboardSystem` (src/render/BillboardSystem.h/.cpp): collects billboard instances (texture, frame, world pos, size), sorts by depth, renders.
- `SpriteAtlas` (src/render/SpriteAtlas.h/.cpp): loads and indexes sprite sheets/animations.
- `ScreenScaler` (src/render/ScreenScaler.h/.cpp): renders to target resolution (e.g., 640x480) then scales to window; handles fullscreen toggle.
- `DebugDraw` (src/render/DebugDraw.h/.cpp): overlays (colliders, nav).

### Audio
- `AudioSystem` (src/audio/AudioSystem.h/.cpp): loads/owns sounds/music; play with panning/attenuation; channel priorities.
- `AudioBus` (optional): mix groups for sfx/music/ui.

### UI
- `UiRenderer` (src/ui/UiRenderer.h/.cpp): HUD, crosshair, pickup messages.
- `MenuSystem` (src/ui/MenuSystem.h/.cpp): start/pause/settings navigation; resolution list; apply config changes.

### Tooling
- `map-validate` (src/tools/map_validate.cpp): loads map, checks geometry integrity, entity refs, trigger wiring.
- `mapmaker` (src/tools/mapmaker/…): custom editor that outputs `level.obj` + `level.meta.json`. Features: sector/brush editing, height/ramps, door markers, triggers, waypoints, entity placement, material assignment, export validation.

## Data & Formats
- Map: created in mapmaker (sector/brush/grid-based) and exported to mesh + metadata:
  - Geometry: OBJ with groups per sector/material.
  - Metadata: JSON/TOML with sectors (heights/material refs), collision AABBs, doors (dir/speed), triggers (volumes/actions), entities (type/pos/facing/params), waypoints/nav graph, materials list.
- Sprites: sprite sheets with animation definitions (JSON), directional frames optional.
- Config/save: JSON/TOML for user settings; save files store player state, entities, triggers, RNG seed.

## Extending the Game

### Adding a New Enemy
1) Data: add enemy entry in enemy definitions (speed, health, attack type, sounds, sprite set, billboard size).
2) Behavior: configure `EnemyBrain` parameters; add attack handler (hitscan or projectile) if new pattern is needed.
3) Assets: provide sprite sheet frames; update `SpriteAtlas`.
4) Map: place enemy via entity list in map metadata.

Inheritance use: derive from `Enemy` only if behavior/state differs meaningfully (e.g., charging melee vs ranged shooter); otherwise prefer data-driven `EnemyBrain` configs to reduce class sprawl.

### Adding a New Item/Pickup
1) Data: define item type (health/armor/ammo/key/custom) with amount/effects and sprite frame.
2) Factory: register in `ItemFactory` with pickup callback (e.g., add ammo, set key flag, heal).
3) Map: place item in map metadata; validator checks references.

### Adding a New Weapon
1) Data: weapon definition (damage, spread, cooldown, ammo type, muzzle flash sprite/sfx).
2) Logic: hook into `WeaponSystem` for fire/cooldown; choose hitscan or projectile path.
3) UI: add icon/strings; ensure HUD shows ammo.

### Adding a New Map
1) Author geometry, export mesh + metadata (collision, triggers, entities).
2) Run `map-validate`.
3) List map in game manifest; ensure assets are bundled.

## Notes on Inheritance vs Composition
- Use a small inheritance tree: `Entity` → `Actor` → (`Player`/`Enemy`), `Entity` → (`Item`/`Projectile`).
- Prefer data-driven configs for variation; reserve new subclasses for genuinely different lifecycles or state machines.
- Keep systems (movement, weapon, audio, rendering) operating on shared interfaces instead of deep hierarchies.

## Update Order (deterministic)
1) Input sampling → `Player` intent.
2) Movement/collision resolution (player, enemies, projectiles).
3) Triggers/doors.
4) Combat resolution (damage, deaths).
5) State cleanup/spawn.
6) Render lists build (billboards, debug).
7) Render/UI/audio.

## Config & Settings
- `Config` stores resolution list, current choice, fullscreen flag, mouse sensitivity, audio volumes, keybinds.
- `MenuSystem` edits `Config`, applies to `ScreenScaler` and window.

## Testing Targets
- Unit: collision sweeps, LOS tests, hitscan intersection.
- Smoke: load map, spawn entities, run fixed steps headless, ensure no crashes and deterministic results with a seed.
