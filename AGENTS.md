# Poind Agent Rules

## Always Read First

Before non-trivial changes, read:

1. `AGENTS.md`
2. `docs/STATUS.md`
3. `docs/NEXT_STEPS.md`
4. `docs/ARCHITEKTUR.md` when changing architecture, layer rules, ownership, services, snapshots, rendering, scenes, or autoloads
5. `docs/SIM_RULES.md` when changing grid rules, placement, capture, energy, resources, agents, tick order, or non-goals
6. `docs/FINDINGS.md` when reviewing, debugging, or fixing known issues

If documents conflict, use this authority order:

```text
ARCHITEKTUR > DECISIONS > SIM_RULES > STATUS > NEXT_STEPS > AGENTS > README
```

## Start Protocol

Before every non-trivial change:

- check `git status --short`
- read the required docs from "Always Read First"
- name the goal of the change
- name the affected layer or file group
- name the main assumption and risk
- name the intended validation path before editing

## Slice Discipline

- Work in coherent slices that are as large as possible and as small as needed.
- Prefer one thematic slice with a clear validation gate over artificial micro-slices.
- Do not bundle unrelated gameplay, rendering, AI, and tooling changes just to reduce slice count.
- Every slice needs a goal, scope boundary, validation path, and explicit non-goals.
- Architecture and documentation slices are valid slices. They should still leave the repo in a clearer state.

## Architecture Invariants

These rules are non-negotiable. A change that conflicts with them needs an ADR in `docs/DECISIONS.md` before implementation.

- Simulation data, runtime snapshots, rendering, scenes, UI, and debug stay separated by layer.
- A territory block is data, not a Godot node.
- The world grid owns occupancy truth.
- A grid cell can have at most one owner.
- Blocks are placed only on empty cells.
- Direct mutation of grid ownership is forbidden outside the simulation service.
- The planned public placement API is `TerritorySimulationService.place_block(colony_id, coord)`.
- Rendering, scenes, UI, and debug must not create or mutate simulation truth.
- Placement expansion in the first playable model is local to the colony's current growth origin. Do not scan a whole territory frontier unless an ADR changes that rule.
- Enclosure/capture checks are triggered by a newly placed block and inspect only local neighbor regions around that placement.
- Tuning parameters belong in inspector-editable `Resource` or `@export` fields once code exists. Avoid hidden hardcoded balance constants in core simulation files.

## Validation

After every code change, match validation depth to the changed layer:

- `core/`: run deterministic headless tests for grid math, neighbor lookup, and flood fill helpers.
- `sim/`: run headless tests for placement legality, occupancy, colony state, capture, resources, and tick order.
- `runtime/` and `rendering/`: run snapshot validation and a visual lab smoke check when available.
- `scenes/`, `ui/`, and `debug/`: run or hand off a manual lab check for controls, overlays, and visible state.
- `docs/`: check links, authority order, status/next-step consistency, and whether ADRs match the rules.

Rules:

- Run the smallest validation that is meaningful for the affected layer, not the smallest validation that is convenient.
- Never invent test results or repo facts.
- If validation was not run, state exactly why and which gate remains open.

## Documentation Sync

After every relevant repo change, check whether documentation must be updated:

- `docs/STATUS.md` for the real current state
- `docs/NEXT_STEPS.md` for the next concrete work block or open gate
- `docs/FINDINGS.md` for new bugs, review findings, debug observations, or planned corrections
- `docs/DECISIONS.md` when a Decision Trigger below applies
- `docs/ARCHITEKTUR.md` for changed layer rules, ownership, services, or autoloads
- `docs/SIM_RULES.md` for changed simulation rules, tick order, non-goals, or balancing contracts

If code changed but docs did not, explain why no documentation update was needed.

## Decision Triggers

Update `docs/DECISIONS.md` when a change introduces or changes:

- architecture or layer authority
- world-grid ownership or placement authority
- public APIs used across layers
- capture/enclosure rules
- colony strategy model, agent observation space, or action set
- resources, energy, upkeep, scoring, or win/loss rules
- rendering snapshot contracts
- external assets, plugins, adapters, autoloads, or asset version upgrades
- test strategy or runner framework
- performance gates, misses, or regressions
- slice direction, scope boundaries, or non-goals

## Review Handoff

Every implementation completion report ends with exactly one review stance:

- `No cross-agent review needed` for trivial single-file docs, typos, or mechanical fixes.
- A concrete cross-agent review request when another agent should evaluate the change.

A cross-agent review is recommended when the change:

- touches architecture, layer boundaries, or simulation authority
- introduces or changes a slice plan or scope boundary
- changes core simulation data structures, capture rules, or snapshots
- adds external assets, plugins, autoloads, or adapter boundaries
- misses a performance gate or exposes unclear tradeoffs

Frame the request as a concrete focus, not just "please review".

Examples:

- `Codex: verify that all territory placement still goes through TerritorySimulationService.place_block().`
- `GPT: review whether the current slice creates useful validation gates without artificial micro-slicing.`
- `Claude Code: verify that capture rules do not scan whole territories or overwrite enemy-owned cells.`

## External Evaluation Intake

When the user provides an external analysis, evaluation, review, or cross-agent critique, treat it as structured implementation input, not background context.

This protocol runs alongside Documentation Sync and Completion Protocol. Per-finding evidence belongs in the completion report. Findings that should remain open go into `docs/FINDINGS.md`. Findings that change architecture, public APIs, simulation truth, capture rules, strategy contracts, assets, adapters, or validation strategy also update the relevant docs and tests.

Required process:

- Extract every concrete finding into an internal checklist before editing.
- Preserve reviewer priority labels such as `P0`, `P1`, and `P2`.
- For each finding, choose one explicit action: `implement`, `document as open finding`, or `reject with reason`.
- Do not merge several findings into a vague "addressed review feedback" bucket.
- P0 items block further implementation until fixed, unless the user explicitly overrides them.
- P1 items should be fixed before the next slice unless explicitly deferred by the user.
- P2 items may be documented as future work, but if the user asks to implement all P2 items, implement them or state the exact blocker.
- If a finding changes architecture, public APIs, simulation truth, capture rules, strategy contracts, assets, adapters, or validation strategy, update the relevant docs and tests.
- After implementation, report evidence per finding group: changed files, validation run, searches run, and remaining open items.

Guardrail:

- Do not claim an external review was implemented just because the general intent was followed. Each actionable point needs a traceable code, test, documentation, or explicit deferral outcome.

## Completion Protocol

Every implementation completion report includes:

- goal handled
- changed file groups
- validation run
- documentation sync result or reason it was not needed
- known risks or follow-ups
- review handoff stance from "Review Handoff"
- commit title suggestion in its own copyable block
- commit description suggestion in a separate copyable block

Commit title format:

```text
type(scope): imperative title
```

Allowed commit types:

```text
feat, fix, perf, refactor, docs, test, chore
```

Common scopes:

```text
core, sim, runtime, rendering, ui, scenes, debug, docs, tests, tools, planning
```

Do not auto-commit, push, or open PRs unless explicitly requested.

## Anti-Patterns

- Do not hide simulation truth in `rendering/`, `scenes/`, `ui/`, or `debug/`.
- Do not mutate grid ownership outside the simulation service.
- Do not scan every owned block for ordinary expansion in the first playable model.
- Do not let visual nodes become territory blocks.
- Do not introduce assets, plugins, adapter boundaries, or autoloads without documenting the decision.
- Do not start ML/learning agents before the deterministic rules and scripted baseline agents are testable.
- Do not invent validation results.
- Do not auto-commit, push, or open PRs without explicit user approval.
