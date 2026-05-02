# Poind Simulation Rules

## Current Scope

The current scope is rules definition only. Gameplay code is not implemented yet.

The first playable model is a territory simulation:

- no individual body organisms
- no direct combat
- no pathfinding agents inside the territory
- no ML/learning agents until deterministic scripted baselines exist

## World

- The world is a bounded square grid.
- A grid cell is either empty or owned by exactly one colony.
- The first prototype starts with 20 colonies by default.
- Each colony starts by placing one owned block at a valid start position.
- Start positions must be unique.

## Placement

A colony may place a block only when:

- the target cell is empty
- the target cell is inside the world
- the target cell is one of the 8 neighbors around the colony's current growth origin
- the colony can pay the placement cost once energy exists

After placement:

- the new block becomes the colony's current growth origin
- local capture checks run around the new block
- no other owned cells are overwritten

## Growth Origin

The first growth-origin model is strict:

```text
growth_origin = last_successfully_placed_block
```

If all 8 neighboring cells around the growth origin are occupied or out of bounds, the colony has no legal placement in the first model.

Multiple active tips, frontier lists, or choosing any owned block are later extensions and need a decision update.

## Capture

After placing a block, inspect the neighboring empty regions around the new block.

A region is captured when:

- it is made only of empty cells
- it does not touch the map edge
- all occupied boundary cells around it belong to the placing colony

Captured cells:

- become owned by the placing colony
- do not overwrite enemy cells
- do not automatically become the growth origin in the first model

Default connectivity:

- placement checks 8 directions
- capture flood fill moves through empty cells in 4 directions

## Resources And Energy

Resources and energy are not implemented yet.

Planned first rule:

- colonies collect energy from owned resource cells
- placing a block costs energy
- capture can secure resource cells if they were empty resource tiles
- larger territory may later add upkeep or administrative cost

## Agents And Strategy

The first strategy layer should use scripted baseline agents before learning agents.

Potential actions:

- choose a legal neighbor from the current growth origin
- prefer nearby resources
- prefer enclosing moves
- prefer blocking opponents
- save energy once energy exists

Learning agents should only be introduced after:

- placement rules are deterministic and tested
- capture rules are deterministic and tested
- scripted baselines exist for comparison
- observations and actions are documented

## Non-Goals For The First Playable Model

- individual moving units
- body-part simulation
- projectile combat
- direct attacks on owned cells
- overwriting enemy territory
- global frontier scanning for ordinary growth
- unbounded maps
- visual effects that own simulation state
