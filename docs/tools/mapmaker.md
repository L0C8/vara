# Mapmaker Design

Custom level editor for the game. Outputs standard OBJ geometry plus a metadata file (`.meta.json`) to drive gameplay (doors, triggers, entities, waypoints).

## Goals
- Fast authoring of blocky/semi-true 3D levels with stairs/ramps and consistent collisions.
- Moving platforms (elevators/lifts) authorable as first-class objects.
- Logic authoring: player spawns, enemy spawners, triggers, keys/doors.
- Asset portability: exported geometry is plain OBJ; metadata is JSON (or TOML) so it is diffable and scriptable.
- Tight validation: catch bad geometry, missing materials, dangling triggers, and entity refs on save.
- Round-trip: game can hot-reload exports; editor can re-open its own files.

## Editing Model
- Y-up, grid-snapped coordinates (configurable grid size).
- Sector/brush based: axis-aligned walls; per-sector floor/ceiling heights; sloped ramps as quads.
- Materials: per-surface/material id assignments; names match game material table (includes special `water`).
- Doors: marked edges/sectors with open direction, speed, and open amount defaults.
- Triggers: volumes or sector tags with actions (open door, spawn enemy, toggle blocker, play sound).
- Waypoints/nav: points with links for enemy navigation; auto-link by proximity with manual overrides.
- Logic layer: player spawn(s), enemy spawners (triggerable), triggers, key/door links, exits/objectives.
- Entities: placements for enemies (with params), items, key doors, exits.
- Movers: moving platforms with start/end (or waypoint) path, speed, dwell, loop/ping-pong; optional trigger gating.
- Water tiles: placeable slabs tagged with `water` material; size/snapping configurable.
- Preview: lightweight 3D preview camera; optional “test spawn” to launch the game on the current map.

## Export Format
- Geometry (`level.obj`):
  - Triangulated mesh, Y-up, no transforms.
  - Groups per sector/material for easy binding.
  - Optional smoothing groups off; normals generated on load if absent.
- Metadata (`level.meta.json`):
  - `sectors`: id, floor/ceiling height, material ids, slope/ramp info.
  - `colliders`: AABBs for walls/geometry used for physics.
- `doors`: id, sector ids or edge refs, open dir/speed, initial state.
  - `triggers`: id, volume (box), type (enter/use/kill-all), actions (open door, spawn, toggle, sound).
  - `logic`: player spawns, enemy spawners (link to enemy defs), key/door bindings, objectives/exits.
  - `entities`: type, position, facing, params (enemy variant, item amount, key id).
  - `waypoints`: nodes + links; optional cost/flags.
  - `movingPlatforms`: id, path (start/end or waypoint list), speed, dwell time per stop, loop/ping-pong flag, trigger/key gating.
  - `waterTiles`: positions/sizes; material id assumed `water`; optional depth/effect flags.
- `materials`: ids → texture names.
- `spawn`: player start(s).
- `version`: schema version for compatibility.

## Validation (on save and via CLI)
- Geometry manifoldness: no inverted normals, overlapping sectors, zero-area faces.
- Colliders align with geometry within tolerance.
- All materials referenced exist in material table.
- Triggers/actions reference valid doors/entities.
- Entity definitions exist in game data.
- Waypoints graph connectivity (warn if isolated).

## UI/UX Sketch
- Top bar: file (new/open/save/export), undo/redo, grid snap toggle.
- Left panel: tools (select, add sector, add ramp, place door, place trigger, place entity, waypoint).
- Right panel: properties of selection (heights, material, door params, trigger actions, entity params).
- Viewport: orthographic top-down editing with optional perspective preview.
- Status: validation errors/warnings, coordinates, grid size.

## Tech Notes
- Built with raylib + raygui for UI rendering and input; uses same math types as the game to reduce conversion issues.
- Exporters share code with runtime loaders to guarantee compatibility.
- CLI mode to convert/edit: `mapmaker --export level.mapproj --out level` (produces obj + meta).

## Workflow
1) Author geometry and placements in mapmaker project format (`.mapproj`).
2) Export → `level.obj` + `level.meta.json`.
3) Run `map-validate level.meta.json` (or built-in validator on save).
4) Game loads OBJ + meta; hot-reload during development.

## Future Enhancements
- Stamp/prefab library for reusable room chunks.
- Heightfield/floor painting for faster ramps.
- In-editor lighting preview (if lightmaps added).
- Multiplayer collaboration (deferred).
