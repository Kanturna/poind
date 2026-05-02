# Poind Findings

## Open Questions

### F-001: Growth Origin May Dead-End Too Easily

Priority: P1

The strict `last_successfully_placed_block` rule is extremely performant, but colonies can become permanently blocked even when older owned cells still have open space. This is acceptable for the first deterministic prototype, but if starts collapse too often, evaluate multiple active growth tips through an ADR.

### F-002: Capture Connectivity Strongly Changes Strategy

Priority: P1

The current default is 8-neighbor placement and 4-neighbor capture flood fill. This makes diagonal corners count as closed. It is easy to understand visually, but it can make captures more generous. Test both diagonal-corner and leak cases before visual polish.

### F-003: Flood Fill Can Touch Large Open Areas

Priority: P1

Capture checks are local in trigger, but a flood fill from a neighboring empty cell can still traverse a large open map before discovering an edge leak. The first implementation should use visited stamps, skip duplicate regions per placement, and exit immediately when the map edge is reached. If this becomes expensive, consider cached empty-region labels in a later ADR.

### F-004: Captured Territory Is Passive In The First Model

Priority: P2

Captured cells become owned territory but do not automatically become growth origins. This keeps expansion local and simple, but it means a big capture does not directly create new expansion fronts. Revisit after the first playable tests.

### F-005: Agent Learning Needs Scripted Baselines First

Priority: P1

Learning agents should not be added until deterministic scripted strategies exist. Without baselines, it will be hard to tell whether a learned behavior is improving, exploiting an unclear rule, or just reacting to noise.
