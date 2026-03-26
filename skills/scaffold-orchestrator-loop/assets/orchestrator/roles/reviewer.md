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
- When the round finalizes, write `review-record.json` with the active `roadmap_id`, `roadmap_revision`, and `roadmap_dir`.

## Boundaries
- Do not fix implementation directly.
- Do not skip checks because the round looks small.
- Do not merge changes.
