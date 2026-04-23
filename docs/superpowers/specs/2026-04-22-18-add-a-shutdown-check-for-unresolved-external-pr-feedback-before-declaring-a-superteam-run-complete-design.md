# Design: Add a shutdown check for unresolved external PR feedback before declaring a superteam run complete [#18](https://github.com/patinaproject/superteam/issues/18)

## Summary

Harden the `superteam` finish and shutdown contract so a run cannot present itself as complete merely because a PR exists or because one snapshot of publish-state looks healthy. `Finisher` must stay active after PR publication until the post-publish workflow is actually stable on the latest pushed state or until an explicit blocker is reported. That includes mergeability, required checks, PR metadata requirements, and final external-feedback handling, not just unresolved inline review threads.

The design focuses on the failure mode surfaced on April 23, 2026: `superteam` could create or update a PR, report a status snapshot, and then stop too early even though `Finisher`-owned work was still active. The fix stays narrow. It does not redesign the teammate roster or the overall `superteam` flow. Instead, it strengthens the existing `Finisher` loop and shutdown path so PR publication is treated as a milestone rather than workflow completion.

## Goals

- Prevent `superteam` from presenting a run as complete while `Finisher`-owned publish-state follow-through is still active on the latest PR state
- Prevent `superteam` from stopping at PR publication plus one status snapshot when mergeability, CI, PR metadata correction, or external feedback handling still requires follow-through
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

PR publication is a milestone, not the end of the workflow. `Finisher` remains active after the PR exists and after any individual status snapshot until the publish-state follow-through is stable enough to hand off cleanly or an explicit blocker is reported.

If required shutdown checks are not clear, the run must not present itself as complete. Instead, it either:

- routes the feedback back through the expected `Finisher`-owned handling path and re-checks, or
- halts and explicitly prompts the operator when the system cannot determine the correct state or cannot safely proceed

This makes the contract match the intended operator trust model: "complete" means external publish-state checks passed, not merely that implementation work and CI reached a good-looking state.

### Required Finisher Shutdown Checks

Before successful shutdown, `Finisher` must perform an explicit post-latest-push verification sequence for the active PR:

1. verify the active PR and current branch state being evaluated
2. verify current publish-state blockers for the latest pushed state, including mergeability, required checks, and PR metadata requirements discovered from repository rules
3. check unresolved inline review threads on the latest PR head
4. check recent blocking external PR feedback on the latest pushed state
5. if blocking work remains, continue the `Finisher`-owned handling loop and re-check rather than stopping at a status snapshot
6. record the final unresolved external-feedback counts for the latest pushed state, including unresolved inline review threads and unresolved top-level blocking reviewer or bot comments
7. treat any nonzero unresolved blocking-feedback count as a blocker
8. only declare successful shutdown when the re-check confirms that no blocking unresolved external feedback or other `Finisher`-owned publish-state blockers remain

For this issue, "blocking external PR feedback" should be interpreted narrowly and concretely. It includes only:

- unresolved inline review threads on the latest PR head
- unresolved review comments or bot findings posted after the latest push that request a code change, verification rerun, follow-up response, or other concrete corrective action before the PR is ready
- unresolved top-level reviewer or bot comments on the latest pushed state when they contain still-applicable findings or requested corrective action, even if there are no inline comments for those findings

It does not include general discussion, acknowledgements, or informational comments that do not ask for a change and do not affect PR readiness.

`Finisher` may dedupe a top-level comment from the final unresolved count only when it is explicitly functioning as a summary of specific inline findings that were already audited on the latest pushed state. If a top-level comment contains standalone findings, mixed findings, or findings with no matching inline thread, `Finisher` must count it as its own blocking finding source and account for it explicitly.

If `Finisher` cannot tell from the available PR state whether a bot or reviewer comment is blocking, the run must treat that uncertainty as a failed shutdown check and prompt the operator instead of guessing.

This definition keeps the shutdown rule enforceable without turning `Finisher` into a speculative classifier for every possible PR comment. It also makes the final shutdown report auditable: `Finisher` should report the unresolved blocking-feedback counts it observed on the latest pushed state rather than implying that the count was probably zero.

Partial success signals are insufficient on their own. PR creation, resolved merge conflicts, green CI, or restored mergeability may all be real milestones, but none of them alone means the workflow is complete until the final publish-state sweep is clear.

### Loopback And Escalation Behavior

When shutdown checks find unresolved feedback, `Finisher` remains the owner of the next step. The workflow should not silently end and should not push ownership back onto `Reviewer`.

The required behaviors are:

- implementation-level external feedback routes through the expected loopback handling path before shutdown
- requirement-bearing feedback continues to route through the existing spec-first path
- unresolved feedback that cannot be safely classified, matched to current branch state, or handled must block shutdown and be surfaced to the operator explicitly
- unresolved publish-state blockers such as metadata violations, pending or failing required checks, or ambiguous branch-caused vs baseline CI failures must remain inside `Finisher` until they are resolved or reported explicitly
- nonzero unresolved blocking-feedback counts must be reported explicitly and treated as blockers rather than as advisory context
- top-level finding comments may only be excluded from the final unresolved count when `Finisher` has explicitly verified that they are summaries of already-audited inline findings on the current head

If the run cannot determine whether feedback still applies to the latest state, cannot determine whether a thread is resolved on the current head, or otherwise cannot safely evaluate shutdown readiness, it must stop and prompt the operator instead of claiming completion.

When CI is failing or unstable after the latest push, `Finisher` should inspect enough evidence to distinguish branch-caused failures from likely baseline or unrelated failures when possible. If that distinction still cannot be made safely, the run must halt with an explicit blocker and prompt the operator instead of silently treating the workflow as done.

### Explicit Blocker Reporting

When shutdown cannot proceed, the run should report the blocker explicitly rather than ending in an implicit or misleading success state. The blocker report should clearly indicate that shutdown did not occur because required external-feedback checks failed or could not be completed.

This is a narrow strengthening of the existing failure handling contract. The important behavioral change is that ambiguous or unresolved publish-state feedback is no longer compatible with a success-style completion message.

### Scope Of Repository Changes

The repository changes should stay tightly coupled to the shutdown problem:

- update `skills/superteam/SKILL.md` so `Finisher` remains active after PR publication, shutdown is clearly success-only, and operator escalation is required when checks cannot be completed
- update only directly relevant `Finisher`-owned prompt or template language if it currently allows completion to be reported before shutdown checks truly pass
- update repository-local pressure tests to cover the exact failure mode and the new halt behavior

The change should avoid broad wording cleanup outside the shutdown and external-feedback path unless a directly related line would otherwise undermine the fix.

## Testing And Verification

- inspect the updated `superteam` skill contract for success-only shutdown wording
- verify the `Finisher` contract makes PR publication a milestone rather than completion
- verify the `Finisher` contract requires continued follow-through for mergeability, required checks, PR metadata, and external feedback handling
- verify `Finisher` shutdown instructions require checking unresolved inline review threads after the latest push
- verify `Finisher` shutdown instructions require checking recent blocking external PR feedback after the latest push
- verify runs with unresolved external feedback cannot present a successful completion state
- verify runs with active publish-state blockers cannot stop at a status snapshot
- verify unresolved implementation-level feedback routes back through the expected loopback handling path before shutdown
- verify failing checks are reported with branch-caused vs likely baseline distinction when known
- verify `Finisher` reports final unresolved blocking-feedback counts for the latest pushed state and treats nonzero counts as blockers
- verify top-level finding comments without inline companions still count as blocking findings
- verify top-level summary comments are only deduped when they explicitly summarize already-audited inline findings on the latest pushed state
- verify indeterminate shutdown state causes an explicit operator prompt instead of silent success
- review pressure-test coverage for the April 22, 2026 failure mode: CI green, PR published, unresolved review threads still open

## Acceptance Criteria

- AC-18-1: Given a `superteam` run that has published a PR, when unresolved inline review threads still exist on the latest PR head, then the run does not present itself as complete
- AC-18-2: Given external PR feedback that is implementation-level, when `Finisher` reaches shutdown, then that feedback is explicitly routed back through the expected loopback path before completion
- AC-18-3: Given a PR with no unresolved inline review threads and no recent blocking bot or reviewer findings on the latest pushed state, when `Finisher` performs shutdown checks, then the run may complete normally
- AC-18-4: Given unresolved external feedback remains or the shutdown state cannot be determined safely, when the run stops, then it reports the blocker explicitly and prompts the operator instead of silently ending in a success state
- AC-18-5: Given a PR has been created or updated, when mergeability, required checks, PR metadata requirements, or final publish-state follow-through still require `Finisher` action, then the run stays in the `Finisher` loop instead of treating publication plus a status snapshot as completion
- AC-18-6: Given CI or publish-state blockers remain after the latest push, when `Finisher` reports status, then it distinguishes branch-caused blockers from likely baseline or unrelated failures when possible and otherwise reports an explicit blocker for operator review
- AC-18-7: Given `Finisher` performs the final publish-state sweep on the latest pushed PR state, when unresolved inline threads or unresolved top-level blocking reviewer or bot comments remain, then it reports the final unresolved counts explicitly and treats any nonzero count as a blocker
- AC-18-8: Given a top-level reviewer or bot comment contains still-applicable findings on the latest pushed state, when those findings are not fully represented by already-audited inline threads, then `Finisher` counts that top-level comment as a separate blocking finding source rather than deduping it away

## Implementation Notes

- Preserve the existing teammate roster and overall `superteam` flow
- Keep the behavioral change centered on `Finisher` post-publish follow-through and shutdown
- Prefer precise contract language over adding broad new workflow machinery
- Treat PR publication as a milestone and shutdown as the final success signal, not merely the end of activity
