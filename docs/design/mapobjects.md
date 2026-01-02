# Map Objects Catalog

Reference for placeable/map-authored objects and their intended behavior/metadata.

## Core Geometry
- Floors: flat surfaces per sector; set floor height, material id
- Ceilings: flat surfaces per sector; set ceiling height, material id
- Walls: vertical faces (axis-aligned); material id; blocking collider by default.
- Doors: wall segment tagged with open direction/speed, initial state; uses door material; collider blocks until open.
- Stairs/Ramps: stepped or sloped connections between heights; marked as walkable; inherit material.
- Ladders: vertical climbable surface; direction vector; climb speed; collider blocks passthrough unless flagged open.


## Special Block
- Water block (resizable): flat quad/slab with `materialId: water`; rendered with animated/scrolling shader; collision selectable (non-blocking wade vs shallow slowdown). Optionally stores depth for movement penalty and sound.

## Metadata Fields (common)
- `materialId`: binds to texture/shader.
- `collider`: blocking/non-blocking; shape (AABB/slab).
- `flags`: walkable, climbable, slippery, damaging, etc.
- `fx`: optional shader/effect tag (used by water).
