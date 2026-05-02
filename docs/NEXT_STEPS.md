# Poind Next Steps

## Immediate Gate

Review the Slice 0 docs and confirm the rule defaults:

- send `docs/EVALUATIONSAUFTRAG_CLAUDE.md` to Claude and process the returned findings through `AGENTS.md` External Evaluation Intake
- `docs/PLANUNGSAUFTRAG.md` accurately summarizes the concept and slice plan
- expansion from only the last placed block
- placement checks 8 neighboring cells
- block placement only on empty cells
- capture fills enclosed empty regions only
- capture flood fill uses 4-neighbor connectivity
- enemy-owned cells are never overwritten in the first model

## Proposed Slice 1: Deterministic Core

Goal: build the pure data foundation without visuals.

Scope:

- `core/` grid coordinate helpers
- `sim/` world grid data model
- colony state with `growth_origin`
- placement legality checks
- deterministic seed setup for 20 colonies
- headless tests for occupancy and local neighbor placement

Non-goals:

- rendering
- capture
- resources
- learning agents

Validation:

- headless test for unique starts
- headless test for legal/illegal placement
- headless test that occupied cells cannot be overwritten

## Proposed Slice 2: Capture Algorithm

Goal: implement enclosure capture as a tested deterministic rule.

Scope:

- local neighbor-region inspection after placement
- flood-fill helper with early exit on map edge
- boundary-owner validation
- fill captured empty regions
- tests for closed rings, edge leaks, diagonal corners, and mixed-owner boundaries

Non-goals:

- visual polish
- resources
- strategy learning

## Proposed Slice 3: First Lab View

Goal: make the simulation visible and step-through debuggable.

Scope:

- simple grid renderer
- colony colors
- pause/step/reset controls
- show growth origin and last placement
- optional capture-highlight overlay

Non-goals:

- beauty rendering
- animation
- ML

## Proposed Slice 4: Resources And Energy

Goal: add the first pressure system.

Scope:

- resource tiles
- colony energy
- placement costs
- resource income from owned cells
- simple scripted placement strategies

Non-goals:

- learning agents
- complex economy

## Proposed Slice 5: Strategy Agents

Goal: introduce comparable colony behavior.

Scope:

- baseline random agent
- resource-seeking agent
- enclosure-seeking agent
- blocking agent
- documented observation/action contract for later learning agents

Non-goals:

- neural training loop until scripted baselines are stable

## Current Commit Suggestion

```text
docs(planning): add poind planning and evaluation briefs
```
