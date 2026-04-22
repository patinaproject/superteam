# Design: Add a shutdown check for unresolved external PR feedback before declaring a superteam run complete [#18](https://github.com/patinaproject/superteam/issues/18)

## Summary

Harden the `superteam` shutdown contract so a run can only complete successfully after `Finisher` verifies that external PR feedback has been checked on the latest pushed state and no unresolved blocking feedback remains. The design focuses on the exact failure mode from April 22, 2026: a run published a PR, got CI green, and still ended as if complete even though unresolved inline review threads were open.

The fix stays narrow. It does not redesign the teammate roster or the overall `superteam` flow. Instead, it strengthens the existing `Finisher` shutdown path so unresolved inline review threads and recent blocking PR feedback become explicit pre-shutdown checks with clear failure behavior.

## Goals

- Prevent `superteam` from presenting a run as complete while unresolved external PR feedback still exists on the latest PR head
- Make shutdown a success-only action that occurs only after required external-feedback checks pass
- Keep ownership of external PR feedback with `Finisher`
- Require explicit loopback or explicit operator escalation when shutdown checks fail
- Keep the change focused on the current shutdown problem rather than broad workflow redesign

## Non-Goals

- Changing the canonical teammate roster
- Redesigning the full `superteam` orchestration flow
- Retrofitting already completed historical runs
- Building a generalized GitHub review bot outside the existing `superteam` workflow
- Making unrelated changes to other `superpowers` skills

## Design

### Shutdown Becomes Success-Only

The orchestration contract should treat shutdown as a success state, not as a generic end-of-run state. A `superteam` run may only shut down successfully after all required publish-state checks pass on the current PR state after the latest push.

If required shutdown checks are not clear, the run must not present itself as complete. Instead, it either:

- routes the feedback back through the expected `Finisher`-owned handling path and re-checks, or
- halts and explicitly prompts the operator when the system cannot determine the correct state or cannot safely proceed

This makes the contract match the intended operator trust model: "complete" means external publish-state checks passed, not merely that implementation work and CI reached a good-looking state.

### Required Finisher Shutdown Checks

Before successful shutdown, `Finisher` must perform an explicit post-latest-push verification sequence for the active PR:

1. verify the active PR and current branch state being evaluated
2. check unresolved inline review threads on the latest PR head
3. check recent blocking external PR feedback on the latest pushed state
4. if blocking feedback exists, dispatch `Finisher`-owned handling and re-check
5. only declare successful shutdown when the re-check confirms that no blocking unresolved external feedback remains

For this issue, "blocking external PR feedback" should be interpreted narrowly and concretely. It includes only:

- unresolved inline review threads on the latest PR head
- unresolved review comments or bot findings posted after the latest push that request a code change, verification rerun, follow-up response, or other concrete corrective action before the PR is ready

It does not include general discussion, acknowledgements, or informational comments that do not ask for a change and do not affect PR readiness.

If `Finisher` cannot tell from the available PR state whether a bot or reviewer comment is blocking, the run must treat that uncertainty as a failed shutdown check and prompt the operator instead of guessing.

This definition keeps the shutdown rule enforceable without turning `Finisher` into a speculative classifier for every possible PR comment.

### Loopback And Escalation Behavior

When shutdown checks find unresolved feedback, `Finisher` remains the owner of the next step. The workflow should not silently end and should not push ownership back onto `Reviewer`.

The required behaviors are:

- implementation-level external feedback routes through the expected loopback handling path before shutdown
- requirement-bearing feedback continues to route through the existing spec-first path
- unresolved feedback that cannot be safely classified, matched to current branch state, or handled must block shutdown and be surfaced to the operator explicitly

If the run cannot determine whether feedback still applies to the latest state, cannot determine whether a thread is resolved on the current head, or otherwise cannot safely evaluate shutdown readiness, it must stop and prompt the operator instead of claiming completion.

### Explicit Blocker Reporting

When shutdown cannot proceed, the run should report the blocker explicitly rather than ending in an implicit or misleading success state. The blocker report should clearly indicate that shutdown did not occur because required external-feedback checks failed or could not be completed.

This is a narrow strengthening of the existing failure handling contract. The important behavioral change is that ambiguous or unresolved publish-state feedback is no longer compatible with a success-style completion message.

### Scope Of Repository Changes

The repository changes should stay tightly coupled to the shutdown problem:

- update `skills/superteam/SKILL.md` so shutdown is clearly success-only and operator escalation is required when checks cannot be completed
- update only directly relevant `Finisher`-owned prompt or template language if it currently allows completion to be reported before shutdown checks truly pass
- update repository-local pressure tests to cover the exact failure mode and the new halt behavior

The change should avoid broad wording cleanup outside the shutdown and external-feedback path unless a directly related line would otherwise undermine the fix.

## Testing And Verification

- inspect the updated `superteam` skill contract for success-only shutdown wording
- verify `Finisher` shutdown instructions require checking unresolved inline review threads after the latest push
- verify `Finisher` shutdown instructions require checking recent blocking external PR feedback after the latest push
- verify runs with unresolved external feedback cannot present a successful completion state
- verify unresolved implementation-level feedback routes back through the expected loopback handling path before shutdown
- verify indeterminate shutdown state causes an explicit operator prompt instead of silent success
- review pressure-test coverage for the April 22, 2026 failure mode: CI green, PR published, unresolved review threads still open

## Acceptance Criteria

- AC-18-1: Given a `superteam` run that has published a PR, when unresolved inline review threads still exist on the latest PR head, then the run does not present itself as complete
- AC-18-2: Given external PR feedback that is implementation-level, when `Finisher` reaches shutdown, then that feedback is explicitly routed back through the expected loopback path before completion
- AC-18-3: Given a PR with no unresolved inline review threads and no recent blocking bot or reviewer findings on the latest pushed state, when `Finisher` performs shutdown checks, then the run may complete normally
- AC-18-4: Given unresolved external feedback remains or the shutdown state cannot be determined safely, when the run stops, then it reports the blocker explicitly and prompts the operator instead of silently ending in a success state

## Implementation Notes

- Preserve the existing teammate roster and overall `superteam` flow
- Keep the behavioral change centered on `Finisher` shutdown and external-feedback follow-through
- Prefer precise contract language over adding broad new workflow machinery
- Treat shutdown as the final success signal, not merely the end of activity
