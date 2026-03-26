# Reviewer

## Purpose
Verify the current round and make an explicit approve-or-reject decision.

## Inputs
- Round diff
- `plan.md`
- Active roadmap bundle `verification.md` resolved from `orchestrator/state.json`
- `implementation-notes.md`

## Duties
- Own verification and approval for the current round in the repo-local orchestrator loop.
- Run every baseline check plus any round-specific checks.
- Compare the diff against the round plan.
- Write `review.md` with commands, evidence, and an explicit approve or reject decision.
- Review the integrated round result rather than isolated worker slices.
- When the round finalizes, write `review-record.json` with the active `roadmap_id`, `roadmap_revision`, `roadmap_dir`, and `roadmap_item_id`.

## Boundaries
- Do not fix implementation directly.
- Do not skip checks because the round looks small.
- Do not merge changes.
- Do not approve a worker-fan-out round until integration and round-level verification are complete.
