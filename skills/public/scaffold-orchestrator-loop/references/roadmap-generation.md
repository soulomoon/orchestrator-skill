# Roadmap Generation

Build the initial roadmap from the goal plus a fast repository survey.

## Inputs

- User goal
- Existing code, docs, tests, and config
- Current branch status
- Constraints discovered in the repo

## Process

1. Survey the repository before drafting tasks.
2. Choose a stable `roadmap_id` that names the control-plane family rather than one transient round.
3. Initialize the first revision as `rev-001`.
4. Identify the smallest meaningful milestones that move the goal forward.
5. Sequence milestones so each item leaves the repo in a coherent state.
6. Record dependencies only when they change ordering.
7. Keep later items coarse; make the next item concrete.

## Roadmap Rules

- Use an ordered list.
- Give each item one clear deliverable.
- Record status with `[pending]`, `[in-progress]`, or `[done]`.
- Include `Depends on:` and `Completion notes:` lines for each item.
- Avoid planning parallel rounds or speculative implementation detail far ahead.
- Prefer 3-7 initial items unless the goal is truly smaller.
- Write the initial roadmap to `orchestrator/roadmaps/<roadmap_id>/rev-001/roadmap.md`.
- Keep a roadmap revision immutable once any round uses it; later semantic updates should create `rev-00N+1` under the same `roadmap_id`.

## First Item Quality Bar

The first actionable roadmap item should be concrete enough that a guider can select it without guessing:

- clear outcome
- obvious verification target
- bounded scope

## Revision Rules

The roadmap is expected to change after accepted rounds. Keep it stable enough to guide execution, but lightweight enough that the guider can update it without rewriting the whole file.
