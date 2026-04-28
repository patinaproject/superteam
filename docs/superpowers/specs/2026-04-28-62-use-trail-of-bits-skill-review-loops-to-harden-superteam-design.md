# Design: Use Trail of Bits skill review loops to harden Superteam [#62](https://github.com/patinaproject/superteam/issues/62)

## Intent

Use Trail of Bits workflow-skill review guidance and skill-improver discipline as a repeatable hardening input for Superteam without making Superteam depend on Claude-only runtime behavior or hidden workflow state.

This change should add a repo-owned quality path for reviewing `skills/superteam/SKILL.md` and adjacent workflow-contract files, capturing findings, applying useful fixes, and documenting the review evidence that proves the workflow-contract surface was actually pressure-tested.

## Requirements

1. The implementation must treat `skills/superteam/SKILL.md` and adjacent workflow files as workflow-skill surfaces, not ordinary docs.
2. The review path must apply or adapt Trail of Bits `workflow-skill-reviewer` checks for structure, trigger description quality, workflow pattern fit, content quality, tool assignment, and anti-patterns.
3. The improvement path must apply or adapt Trail of Bits `skill-improver` severity handling: critical and major findings are fixed or explicitly dispositioned; minor findings are evaluated before applying.
4. Review findings must be recorded with severity, affected surface, finding text, disposition, and verification evidence.
5. The design must preserve Superteam's current local contracts: issue-first work, committed design and plan artifacts, visible-state resume, teammate ownership, Gate 1 adversarial review, Reviewer before Finisher, and latest-head publish-state follow-through.
6. The design must not reintroduce `Loopback:` trailers or other hidden persistence markers removed by the current Superteam workflow.
7. The implementation must document the exact review loop or adapted commands used so future Superteam workflow-contract changes can repeat the gate.
8. The implementation must include RED/GREEN evidence for any new workflow discipline rule it adds, or explicitly record why a Trail of Bits review step is advisory rather than a new rule.
9. The PR must include acceptance-criteria verification under the repository PR template and must not claim production readiness without the review loop, pressure tests, and role-specific verification that actually ran.

## Approach

Use a three-layer adaptation rather than vendoring Trail of Bits plugin behavior wholesale.

First, add repository guidance for a Superteam skill-quality review loop. The guidance should name the Trail of Bits sources and translate their expectations into this repo's language: findings, dispositions, evidence, and pressure-test updates. This keeps the quality loop repeatable even when the exact Trail of Bits plugin is not installed in a future runtime.

Second, update Superteam workflow-contract surfaces only where the review loop needs to be enforced. Likely surfaces include `skills/superteam/SKILL.md`, `skills/superteam/agent-spawn-template.md`, and `docs/superpowers/pressure-tests/superteam-orchestration-contract.md`. The implementation should avoid broad refactors and should not turn the operator-facing chat output back into a rigid status template.

Third, run the adapted review loop against Superteam itself. The resulting findings should drive the actual implementation work. Critical and major issues must either be fixed or explicitly rejected as not applicable to Superteam with a repo-specific rationale. Minor findings are accepted only when they improve execution reliability or reviewer evidence.

## Alternatives Considered

### Vendor the Trail of Bits plugins

This would keep the process closest to the upstream tools, but it would make Superteam's runtime story more brittle and could imply Claude-only behavior. This issue should use Trail of Bits as a review methodology, not as a mandatory shipped dependency.

### Only run the existing pressure tests

This preserves today's workflow but does not add the external skill-design lens requested by the issue. The current pressure tests should remain part of verification, but they should be supplemented by Trail of Bits structural review categories.

### Replace Superteam's review contract with skill-improver

This would overfit to one tool. Superteam still needs its teammate-owned local Reviewer and Finisher contracts. The Trail of Bits loop should feed those contracts rather than replace them.

## Acceptance Criteria

### AC-62-1

Given the Superteam skill and adjacent workflow files, when the Trail of Bits workflow-skill review approach is applied or adapted, then the resulting findings are recorded with severity, affected file, and disposition.

### AC-62-2

Given a critical or major finding from that review, when the implementation is complete, then the finding is either fixed in the repository or explicitly documented as not applicable with a repo-specific rationale.

### AC-62-3

Given minor findings from the review, when the implementation is complete, then each minor finding is evaluated for usefulness rather than applied blindly.

### AC-62-4

Given the improvement pass has completed, when a maintainer reviews the PR, then the PR includes the exact verification steps, pressure tests, or review loop evidence that ran.

### AC-62-5

Given Superteam-specific workflow requirements, when Trail of Bits guidance conflicts with local Patina/Superteam contracts, then the local contract remains intact and the decision is documented.

## Workflow-Skill Review Obligations

The implementation must include a RED-phase baseline for any new Superteam discipline rule introduced by this issue. The baseline can be a pressure-test scenario, an adversarial review note, or a documented reviewer failure mode, but it must show what would go wrong without the rule.

The review must explicitly check these writing-skills dimensions before Gate 1 can advance:

- RED/GREEN baseline obligations for new discipline rules.
- Rationalization resistance for shortcuts such as skipping the Trail of Bits loop, applying minor findings blindly, or treating plugin absence as permission to omit evidence.
- Red flags for missing severity, missing disposition, non-repeatable review commands, and hidden state.
- Token-efficiency targets so the Superteam skill does not grow by dumping external reviewer docs inline.
- Role ownership so Brainstormer, Planner, Executor, Reviewer, and Finisher keep their existing responsibilities.
- Stage-gate bypass paths, especially paths that would let Executor publish without Reviewer or Finisher because a skill-improver loop already ran.

## Rationalizations to Reject

| Rationalization | Why it fails |
| --- | --- |
| "The Trail of Bits reviewer is external, so a summary is enough." | Future reviewers need severity, affected file, disposition, and verification evidence to trust the loop. |
| "The skill-improver loop fixed issues, so Superteam Reviewer can be skipped." | The loop is an input to local review, not a replacement for Superteam's Reviewer and Finisher stages. |
| "A minor finding came from a reviewer, so we should apply it automatically." | Minor findings can be false positives or style preferences; each one needs usefulness evaluation. |
| "The plugin is unavailable, so the AC is impossible." | The repo can adapt the workflow-skill review categories and record the limitation rather than silently skipping the quality gate. |
| "A new sidecar marker would make the loop easier to resume." | Superteam v1.3.0 resumes from visible artifacts, PR state, and operator prompts; hidden persistence markers are out of scope. |

## Red Flags

- Findings are recorded without severity or disposition.
- Critical or major findings remain open without an explicit blocker.
- Minor findings are applied mechanically without value evaluation.
- Review evidence names Trail of Bits generally but does not identify the checks or adapted process that ran.
- Implementation changes workflow-contract files without updating or running pressure-test coverage.
- The review loop bypasses Brainstormer, Planner, Executor, Reviewer, or Finisher ownership.
- The Superteam skill gains large copied Trail of Bits reference material instead of concise local guidance and links.
- The implementation reintroduces `Loopback:` trailers or another hidden workflow-state marker.

## Verification Plan

1. Run or adapt the Trail of Bits `workflow-skill-reviewer` process against `skills/superteam`.
2. Run or adapt the Trail of Bits `skill-improver` severity loop until no critical or major finding remains unaddressed.
3. Run the Superteam orchestration pressure tests that cover workflow-contract review, natural operator-facing output, visible-state resume, Reviewer handoff, and Finisher shutdown behavior.
4. Run markdown lint for changed Markdown files.
5. Verify the PR body maps AC-62-1 through AC-62-5 to the review evidence and pressure-test results that actually ran.

## Out of Scope

- Vendoring Trail of Bits plugins into this repository.
- Requiring Claude-only agent tools for Superteam to function.
- Replacing Superteam's local Reviewer and Finisher stages.
- Reintroducing commit trailers, sidecar files, branch labels, or hidden state for workflow routing.
- Broad rewrites of Superteam unrelated to skill-review loop hardening.
