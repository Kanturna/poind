# Poind Decisions

## ADR-001: Square Grid Coordinates

Decision: Poind uses a bounded square grid with integer coordinates `(x, y)`.

Reason: The current concept is territory placement on blocks. Square coordinates make local neighbor checks, capture fills, resources, and future agent observations simple.

## ADR-002: Colony-Level Simulation First

Decision: The initial prototype simulates colonies and territories, not individual ship/body organisms.

Reason: The current design direction is collective territory expansion, enclosure, resource control, and strategy emergence. Individual agents may appear later as strategy controllers, not as physical bodies in the first model.

## ADR-003: Local Growth Origin Instead Of Full Frontier Scan

Decision: The first playable model expands from a colony's current growth origin, initially the last placed block.

Reason: This keeps per-growth work bounded to a local neighbor check and avoids scanning large territories.

Rules:

- each colony starts with one block
- the start block is its first growth origin
- after a successful placement, the new block becomes the growth origin
- only the 8 neighbors of that origin are candidates
- if no candidate is empty, the colony is blocked until a later rule changes the origin model

Re-evaluation trigger: if too many colonies dead-end early, add multiple active growth tips through a new ADR.

## ADR-004: Exclusive Occupancy And No Overwrite Placement

Decision: A block can only be placed on an empty cell.

Reason: The first conflict model should be about blocking, racing, enclosing, and map control rather than direct combat.

Rules:

- no stacked blocks
- no direct overwrite attacks
- no capture of enemy-owned cells in the first playable model

## ADR-005: Local Enclosure Capture

Decision: After each placement, the simulation checks only neighbor regions around the newly placed block for enclosed empty space.

Reason: Only the newest block can close a new boundary, so capture checks should be local to that placement.

Default rule:

- inspect empty neighbors around the newly placed block
- flood-fill candidate empty regions
- if a region reaches the map edge, it is not enclosed
- if any occupied boundary cell belongs to another colony, the region is not captured by the current colony
- if the boundary is fully owned by the current colony, fill the empty region for that colony

Connectivity:

- placement uses 8-neighbor adjacency
- capture flood fill uses 4-neighbor empty connectivity

Reason for mixed connectivity: growth can feel flexible and diagonal, while capture still produces clear closed areas where diagonal corners count as closed.

## ADR-006: Layered Simulation And Snapshot Boundary

Decision: Simulation truth stays in `sim/`; renderers and UI consume runtime snapshots.

Reason: Later learning agents, replay, debugging, and performance work need deterministic simulation data that is independent from visuals.

## ADR-007: Slice 0 Is Method And Documentation

Decision: The project starts with architecture and documentation before gameplay implementation.

Reason: The prior Baktorium workflow showed that explicit docs, agent rules, status, decisions, and slice boundaries reduce drift before code complexity grows.

## Open Decision Backlog

- map size and whether maps can wrap
- whether captured cells become passive territory only or can also become future growth origins
- first resource types and whether resources are one-time stockpiles or per-tick production
- whether energy is global per colony or location/supply based
- first scripted strategy action set
- first visual representation for cells, borders, resources, and colonies
