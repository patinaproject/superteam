# Design: Add a first-class PR review evidence workflow for orchestrators [#81](https://github.com/patinaproject/superteam/issues/81)

## Intent summary

Add a reusable Superteam orchestration contract for recommendations that must be grounded in recent pull requests and review feedback. The workflow should tell an orchestrator what evidence to collect first, what fallback signals are allowed when direct review evidence is unavailable, and how to label each recommendation so operators can see whether it came from review comments, PR metadata, or lower-confidence fallback evidence.

This is a workflow-contract design because implementation is expected to touch the installed `skills/superteam/**` package and may add a reusable support reference for evidence collection.

## Acceptance criteria

### AC-81-1

Given an orchestration task that asks for recommendations grounded in recent PRs and reviews, when the workflow runs, then it has a documented primary path for collecting PR metadata and review evidence.

### AC-81-2

Given direct review evidence is unavailable, when the workflow falls back, then it uses a documented fallback path and reports that fallback in the final output.

### AC-81-3

Given the workflow produces a recommendation, when an operator reads it, then they can tell whether it came from direct review comments, PR metadata, or fallback evidence.

### AC-81-4

Given a new orchestration is built on top of this capability, when its prompt asks for evidence-grounded PR/review analysis, then the orchestrator can reuse this contract instead of inventing its own collection logic.

## Requirements

- **R1 Primary path**: Define a first-class path that starts from recent merged PRs, then gathers PR metadata, review decisions, review comments, unresolved or resolved review threads when available, bot review findings, linked issue references, and merge SHAs.
- **R2 Evidence tiers**: Classify evidence as `direct-review`, `pr-metadata`, or `fallback-proxy`. Direct review comments and review-thread text are the highest-confidence signal. PR metadata is contextual support. Commit history, commit messages, local git diffs, and issue text are fallback proxies only.
- **R3 Fallback visibility**: When direct review evidence cannot be collected because of missing permissions, absent review comments, tool failure, unavailable connector, or no matching recent PRs, the orchestrator must say which fallback path was used and lower confidence accordingly.
- **R4 Recommendation attribution**: Every final recommendation produced from the workflow must include its evidence tier, evidence source summary, and confidence level. Mixed evidence should name the strongest available tier plus any fallback support.
- **R5 Reusable contract**: Put the reusable collection and reporting contract in the Superteam skill package, not in a one-off design or repo-specific heuristic. Future orchestrations should be able to reference it by name.
- **R6 Role ownership**: `Team Lead` owns routing into the contract and deciding when a prompt asks for evidence-grounded PR/review analysis. `Finisher` remains owner of external PR feedback during normal issue-to-PR runs. This workflow is for analysis tasks and must not steal live PR feedback handling from Finisher.
- **R7 Verification**: Implementation must follow writing-skills RED/GREEN evidence discipline for this skill/workflow-contract change. Before implementation edits, run baseline pressure tests for full evidence, partial evidence, no direct review evidence, and misleading commit-history-only scenarios; capture the actual failure or rationalization. After implementation, rerun the pressure tests, capture GREEN evidence, record any remaining red flags, and explicitly disclose any pressure test that could not be run. Documentation-only walkthroughs are insufficient readiness evidence.

## Proposed architecture

Add a compact `PR review evidence workflow` contract to `skills/superteam/SKILL.md`, backed by a support reference such as `skills/superteam/pr-review-evidence.md` if the details would make `SKILL.md` too long. `SKILL.md` should hold the trigger, ownership rule, and required output contract; the support reference should hold the step-by-step evidence ladder, source tiers, fallback rules, and pressure-test scenarios.

The collection flow should be:

1. Resolve the repository and target window from the operator prompt, with a conservative default such as recent merged PRs if the prompt does not specify a range.
2. Collect recent merged PR metadata: number, title, author, merge date, merge SHA, changed-file summary, linked issue, labels, and review decision summary.
3. Collect direct review evidence where available: review comments, review bodies, review-thread discussions, requested changes, approval comments, bot review findings, and final resolutions.
4. Build an evidence ledger that records each signal with `tier`, `source`, `locator`, `summary`, and `availability_status`.
5. If direct review evidence is incomplete or absent, explicitly activate the fallback ladder: PR metadata first, then commit history and local git evidence, then issue text or repository docs only when the prompt requires continuing.
6. Produce recommendations with per-item attribution: `evidence_tier`, `source_summary`, `confidence`, and `fallback_used`.

Preferred implementation shape:

- Keep `SKILL.md` short and invariant-focused.
- Add a support reference if the evidence ladder or examples would exceed a small section.
- Update Team Lead guidance so prompts asking for recommendations grounded in recent PRs/reviews route through this contract.
- Update Finisher guidance only enough to avoid ownership confusion: live external PR feedback remains Finisher-owned; this workflow is an analysis evidence contract.
- Add pressure-test coverage to `docs/superpowers/pressure-tests/superteam-orchestration-contract.md` or a focused adjacent pressure-test file.

## Affected files

- `skills/superteam/SKILL.md`: new trigger, ownership rule, evidence-tier output contract, and support-file link.
- `skills/superteam/pr-review-evidence.md`: likely new support reference for collection steps, fallback ladder, and examples.
- `skills/superteam/agents/team-lead.openai.yaml`: small prompt cue if Team Lead needs to recognize evidence-grounded PR/review analysis requests.
- `skills/superteam/agents/finisher.openai.yaml`: small ownership clarification only if needed.
- `skills/superteam/quality-guards.md`: rationalization rows and red flags for silent fallback, commit-history overclaiming, and source laundering.
- `docs/superpowers/pressure-tests/superteam-orchestration-contract.md`: reusable pressure scenarios and expected evidence labels.

Claude-host parity files may also need the same role guidance if present in the installed package at implementation time.

## Non-goals

- Do not add repo-specific scoring heuristics or recommendations.
- Do not require a particular GitHub connector, CLI, or API when equivalent evidence can be gathered another way.
- Do not make commit history equivalent to direct review feedback.
- Do not change normal Superteam live PR feedback ownership or Finisher shutdown gates.
- Do not require this workflow for ordinary issue implementation, local code review, or CI triage unless the operator asks for evidence-grounded PR/review analysis.

## Risks and mitigations

| Risk | Mitigation |
|---|---|
| Orchestrator silently degrades to commit history and presents generic advice. | Require `fallback_used` and confidence disclosure in the final output. |
| Commit messages are treated as if they were reviewer feedback. | Use explicit tiers; commit history is always `fallback-proxy`. |
| The new workflow conflicts with Finisher's external feedback ownership. | State that live PR feedback handling remains Finisher-owned; this contract covers analysis tasks. |
| The contract becomes too verbose for frequently loaded skill text. | Keep `SKILL.md` to trigger and invariant language; move examples and ladder details to a support file. |
| A future orchestrator gathers evidence but loses source attribution in the final recommendation. | Require per-recommendation evidence tier, source summary, confidence, and fallback fields. |
| Tooling cannot access review comments in a given repo. | Make unavailability an explicit evidence status, then use the documented fallback ladder. |

## Pressure tests

These are writing-skills pressure tests, not optional documentation walkthroughs. Planner and Executor should preserve the RED/GREEN evidence loop:

1. Before implementation edits, run baseline prompts or subagent pressure tests against the current contract.
2. Capture the actual RED failure, rationalization, or refusal to use the desired evidence discipline.
3. After implementation, rerun the same scenarios against the revised contract.
4. Record GREEN evidence, any remaining rationalizations, and any pressure test that could not be run.
5. Treat documentation-only walkthroughs as insufficient for readiness when pressure-test or subagent evidence was required but unavailable.

### PT-81-1: Full direct review evidence

Scenario: The repository has recent merged PRs with review comments, requested changes, approvals, and thread resolutions.

RED evidence to capture: Current behavior either omits direct-review attribution, treats a PR summary as enough, or gives recommendations without an evidence ledger.

Expected behavior: The orchestrator records direct review evidence, attributes recommendations to `direct-review`, uses PR metadata as context, and does not invoke fallback.

Failure signal: Recommendations cite only commit summaries or omit review-comment attribution.

### PT-81-2: PR metadata present, no review comments

Scenario: Recent merged PRs exist, but review comments are absent or inaccessible.

RED evidence to capture: Current behavior either invents reviewer intent, hides missing review comments, or fails to lower confidence.

Expected behavior: The orchestrator reports direct review evidence as unavailable, falls back to PR metadata, labels recommendations as `pr-metadata`, and lowers confidence.

Failure signal: Output implies reviewers said something specific without direct review evidence.

### PT-81-3: Connector or permission failure

Scenario: The GitHub evidence source fails while local git history is available.

RED evidence to capture: Current behavior silently continues from local git history or treats the connector failure as irrelevant.

Expected behavior: The orchestrator reports the failed source, uses documented fallback proxies only if the operator task can still be answered, labels them `fallback-proxy`, and names the confidence limitation.

Failure signal: Output hides the collection failure or presents local commit evidence as review evidence.

### PT-81-4: New orchestration reuse

Scenario: A future repo-health or coaching orchestration asks for recommendations grounded in recent PRs and reviews.

RED evidence to capture: Current behavior invents an ad hoc collection plan, source taxonomy, or fallback disclosure standard.

Expected behavior: The prompt references the shared PR review evidence workflow and uses its evidence ledger plus output attribution contract instead of inventing new collection logic.

Failure signal: The orchestration defines an incompatible evidence taxonomy or has no fallback disclosure.

### PT-81-5: Misleading commit-history proxy

Scenario: Commit messages suggest one review theme, but available review comments show a different concern.

RED evidence to capture: Current behavior follows the commit-history proxy, overclaims confidence, or fails to prefer direct review comments.

Expected behavior: Direct review evidence wins. Commit history may appear only as supporting `fallback-proxy` or context, not as the primary basis.

Failure signal: The recommendation follows commit-history framing while ignoring direct review comments.

## Adversarial review preparation

Reviewer context: same-thread fallback for the Brainstormer design draft. A fresh Team Lead adversarial design review is still required after this committed artifact before Gate 1 approval can advance.

Checked writing-skills dimensions:

- **RED/GREEN baseline obligations**: Present after revision. R7 and the pressure-test section require baseline pressure-test or subagent evidence before implementation, captured failure/rationalization, rerun GREEN evidence after implementation, remaining red-flag disclosure, and explicit disclosure for any pressure test that could not be run.
- **Rationalization resistance**: Present. The design blocks "commit history is close enough" and "fallback can be silent" rationalizations.
- **Red flags**: Present. Risks and pressure tests name source laundering, missing fallback disclosure, and Finisher ownership confusion.
- **Token-efficiency targets**: Present. Runtime skill text is constrained to a short `SKILL.md` trigger plus a support reference for details.
- **Role ownership**: Present. Team Lead owns analysis routing; Finisher retains live PR feedback handling.
- **Stage-gate bypass paths**: Present. The design calls for pressure tests and visible evidence status rather than confidence-only readiness claims.

## Adversarial review findings

| Source | Severity | Location | Finding | Disposition |
|---|---|---|---|---|
| brainstormer | material | `## Proposed architecture` | The first architecture pass could blur live PR feedback handling with offline evidence-grounded analysis, which would conflict with Finisher ownership. | Resolved by adding R6, the Finisher ownership note, and an explicit non-goal. |
| brainstormer | material | `## Requirements` | The first requirement set said to "use fallback signals" but did not force per-recommendation attribution, allowing source laundering in final advice. | Resolved by adding R4 and the final-output fields. |
| brainstormer | minor | `## Affected files` | The first affected-files list risked over-prescribing agent-file edits before Planner confirms exact surfaces. | Resolved by marking Team Lead and Finisher agent updates as small cues only if needed. |
| adversarial-review | material | `R7`, `## Pressure tests`, `## Adversarial review preparation`, `## Handoff guidance` | The design defined scenarios but did not require writing-skills RED/GREEN baseline evidence, captured rationalizations, GREEN reruns, or insufficient-evidence disclosure; it also left room for documentation-only walkthroughs as readiness evidence. | Resolved by strengthening R7, adding explicit RED/GREEN pressure-test evidence requirements and per-scenario RED evidence prompts, updating the checked RED/GREEN dimension, and revising handoff guidance to label documentation-only walkthroughs as insufficient readiness evidence. |

Brainstormer same-thread review result: `findings dispositioned`.

## Clean pass rationale

No blocker or material findings recorded in this artifact remain open. The draft has explicit AC IDs, a primary evidence path, visible fallback reporting, per-recommendation attribution, reusable contract placement, role ownership boundaries, and writing-skills RED/GREEN pressure-test evidence requirements that close the documentation-walkthrough bypass. Team Lead must still decide whether the revised committed artifact satisfies Gate 1.

## Handoff guidance

Planner should convert this design into a small implementation plan that preserves the contract split: `SKILL.md` owns trigger and invariant language, while a support reference owns the longer evidence ladder if needed. The plan must schedule RED baseline pressure tests before implementation edits, require captured rationalizations or failures, and schedule GREEN reruns after implementation. Executor should avoid adding a heavyweight runtime integration or repo-specific heuristic and must preserve the RED evidence rather than replacing it with confidence language. Reviewer should treat `skills/superteam/**` changes as installable skill changes and run the required writing-skills and Superteam quality gates, including checking that pressure-test evidence exists or that missing tests are explicitly disclosed as blockers or residual risk. Finisher should ensure the PR body maps verification to AC-81-1 through AC-81-4 and labels documentation-only walkthroughs as insufficient readiness evidence when pressure-test or subagent evidence was required but not run.
