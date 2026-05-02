# Poind Architecture

Poind starts as a Godot 4.6 territory simulation. The first architecture goal is a small deterministic core that can later support visual labs, scripted colony strategies, and eventually learning agents.

## Layer Order

```text
ui / debug / scenes
        ^
rendering
        ^
runtime
        ^
sim
        ^
core
```

Rules:

- `core/` contains reusable deterministic math: grid coordinates, neighbor directions, bounds, and flood-fill helpers. It must not depend on Godot scenes or nodes.
- `sim/` owns world occupancy, colony state, placement, capture, resources, energy, and tick order.
- `runtime/` builds read-only snapshots and derived view models for renderers, HUD, and debug overlays.
- `rendering/` draws snapshots. It never reads or mutates live simulation state directly.
- `scenes/` compose nodes, resources, input, lab workflow, and tick driving.
- `ui/` presents controls and state. It does not own simulation truth.
- `debug/` observes and instruments. Debug tools do not write simulation truth except through explicit sim APIs used by normal gameplay.

## Simulation Truth

Simulation truth is:

- grid size and coordinate system
- cell occupancy and owner id
- colony state, energy, resources, active growth origin, and strategy state
- placement history needed by deterministic replay
- capture results

Simulation truth is not:

- rendered colors, sprites, outlines, particles, labels, camera position, or HUD layout
- debug overlays
- editor-only previews

## Placement Contract

The planned public placement API is:

```text
TerritorySimulationService.place_block(colony_id, coord)
```

The service must validate:

- target coordinate is inside the world
- target cell is empty
- target cell is adjacent to the colony's current growth origin in the first playable model
- the colony can pay the placement cost once energy exists

The service then:

- writes occupancy
- updates colony state
- triggers local enclosure checks around the newly placed block
- records any capture result

Direct mutation of world-grid ownership outside the simulation service is forbidden.

## World Grid Contract

The world grid stores occupancy by coordinate:

```text
empty | owner_id
```

A cell has at most one owner. The first playable model has no overwrite attacks and no stacked blocks. Conflict happens through blocking, racing, enclosing, and resource control.

Ordinary expansion must not scan all owned cells. It starts from the colony's current growth origin and checks only local neighbors. A later switch to multiple active growth tips or frontier lists needs an ADR.

## Capture Contract

Capture is caused only by placement. After a block is placed, the simulation inspects empty neighbor regions around that block.

A candidate empty region is captured when:

- it does not reach the map edge
- every occupied boundary cell around the region belongs to the capturing colony
- the region contains only empty cells

Captured cells become owned by the capturing colony. Enemy-owned cells are never overwritten by capture in the first playable model.

The first default is:

- placement adjacency uses 8 neighbors
- capture flood fill uses 4-way empty-cell connectivity

This means diagonal contact can close a corner. Changing this rule needs an ADR because it strongly changes strategy.

## Runtime Snapshot Contract

Renderers, HUD, and debug overlays consume snapshots rather than live simulation objects.

Snapshots may include:

- grid dimensions
- visible occupied cells with owner ids
- colony colors and basic metrics
- last placement and capture events
- resource and energy display data

Snapshots must not expose mutable simulation state.

## Lab Contract

The first lab scene should be a deterministic sandbox:

- bounded square grid
- fixed random seed support
- 20 colonies by default once gameplay starts
- step, pause, resume, reset, and seed controls
- optional debug overlays for growth origin, placement choices, and captured regions

The lab is a composition root. It may drive ticks locally until multiple scenes or a larger runtime require a shared time service.

## Asset And Plugin Contract

External plugins live in `addons/` and are connected through adapter scripts. They may improve rendering or debugging, but they do not own simulation state.

Any external asset, plugin, adapter, or autoload needs an ADR in `docs/DECISIONS.md`.
