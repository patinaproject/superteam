# Design: Superteam: optimize for repeated /superteam invocations and make workflow state more durable [#39](https://github.com/patinaproject/superteam/issues/39)

## Context

Developers using `superteam` have converged on a usage pattern the skill was
not built for: they prefix nearly every instruction with `/superteam` and rely
on the skill to figure out which phase they are in and route the prompt to the
correct teammate. Today `skills/superteam/SKILL.md` has no in-skill memory
between invocations. Phase is re-derived on every call from committed
artifacts (`docs/superpowers/specs/`, `docs/superpowers/plans/`), branch
state, and PR state. There is no canonical, machine-readable pointer to the
current phase, the active teammate, the open approval gate, the in-flight
loopback class, or the last handoff SHA.

Symptoms (from issue #39 and observed behavior):

- Repeated `/superteam <new instruction>` calls mid-flow re-enter from the top
  of `Team Lead` orchestration and may mis-route, restart phases, or silently
  skip approval gates (e.g. Gate 1).
- A new instruction arriving mid-`Brainstormer`-approval is sometimes treated
  as a fresh design request rather than as feedback against the pending
  approval packet.
- A new instruction arriving mid-`Executor` may restart planning rather than
  routing as a plan-level or implementation-level loopback.
- Post-PR `Finisher` monitoring loses context across invocations, so a fresh
  `/superteam` call can be confused about whether the workflow is `triage`,
  `monitoring`, `ready`, or `blocked`.
- An ambiguous prompt during a loopback can be reinterpreted as a new top-of-
  workflow request because the skill has no signal of what loopback is in
  flight.

## Intent

Change `skills/superteam/SKILL.md` (and one supporting artifact convention)
so that:

1. Every `/superteam` invocation **must first detect the current phase** from
   durable signals before doing anything else.
2. Detection is anchored on a small, committed, machine-readable
   **workflow-state file** under `docs/superpowers/state/<issue>.md`, with
   committed artifacts, branch state, and PR state as cross-checks.
3. The default behavior on a repeated invocation is **resume and route**, not
   **restart**.
4. New prompts arriving mid-phase are classified explicitly as **resume**,
   **feedback for the active teammate / open gate**, **explicit loopback**,
   or **new top-of-workflow request**, with the routing for each case
   spelled out in `SKILL.md`.
5. When the state file and reality disagree, the workflow halts with an
   explicit blocker rather than guessing.

This is a workflow-contract change. It touches a single skill
(`skills/superteam/SKILL.md`), introduces one new artifact convention
(`docs/superpowers/state/<issue>.md`), and modifies `Team Lead` and
`Finisher` contracts. No new teammate is added.

## Requirements

R1. `skills/superteam/SKILL.md` must define a **phase-detection pre-flight**
that runs at the top of every `/superteam` invocation, before routing.

R2. The pre-flight must consult, in order:

  (a) the workflow-state file at `docs/superpowers/state/<issue>.md` if
      present;
  (b) committed design artifacts under `docs/superpowers/specs/` and plan
      artifacts under `docs/superpowers/plans/` for the active issue;
  (c) branch state (current branch name, last commit author / message /
      trailers);
  (d) PR state for the active branch (exists, open/closed/merged, latest
      pushed head, required-check status).

R3. The workflow-state file must exist as a **committed** artifact. It is
created by `Team Lead` on first entry into a `superteam` run for an issue
and is updated by the teammate that owns the next state transition. Schema:

- `issue`: GitHub issue number
- `issue_url`: full URL
- `branch`: branch name in use
- `phase`: one of `brainstorm | plan | execute | review | finish | halted`
- `active_teammate`: one of `Team Lead | Brainstormer | Planner | Executor | Reviewer | Finisher`
- `open_gate`: e.g. `Gate 1: Brainstormer approval` or `none`
- `open_loopback`: one of `none | implementation-level | plan-level | spec-level`
- `last_handoff_sha`: commit SHA of the most recent committed handoff artifact
- `finisher_state`: one of `none | triage | monitoring | ready | blocked`
- `last_updated_utc`: ISO-8601 timestamp
- `notes`: free-form short string for the most recent transition reason

R4. `SKILL.md` must define the **state-file lifecycle**:

- Created by `Team Lead` on first `/superteam` for an issue, before
  delegating to `Brainstormer`.
- Updated by the teammate that performs a state-changing action,
  committed in the same commit as the artifact handoff when one exists,
  or in a dedicated state-update commit when there is no other artifact
  change (e.g. transition to `Finisher` `monitoring`).
- Never edited by hand to bypass a gate; gates are still authoritative.
- Treated as advisory when it disagrees with reality (see R8).

R5. `SKILL.md` must add an explicit **routing table** for repeated
invocations. For each `(detected_phase, prompt_classification)` pair, the
table must specify the teammate to route to and the action (resume,
deliver-as-feedback, open-loopback, or new-run). Required rows:

- phase=brainstorm, gate=Gate 1 open, prompt looks like feedback ->
  deliver to `Brainstormer` as delta-only revision; do not restart.
- phase=brainstorm, gate=Gate 1 open, prompt looks like approval ->
  fire Gate 1 approval and route to `Planner`.
- phase=execute, prompt looks like requirement change -> route through
  `Brainstormer` (`spec-level` loopback) per existing loopback rules.
- phase=execute, prompt looks like task adjustment that preserves
  requirements -> route to `Planner` (`plan-level` loopback).
- phase=execute, prompt looks like an implementation question -> route
  to `Executor`.
- phase=finish, finisher_state in {triage, monitoring, blocked}, prompt
  is a status check -> route to `Finisher`; do not restart.
- phase=finish, prompt is requirement-bearing PR feedback -> route
  through `Brainstormer` per existing external-feedback rules.
- phase=halted, prompt is anything -> show the halt reason and require
  explicit operator instruction before resuming.
- any phase, prompt is unambiguously a new top-of-workflow request for a
  different issue -> require explicit operator confirmation before
  starting a new run.

R6. `SKILL.md` must define a **prompt-classification heuristic** with a bias
toward "treat as feedback for the active teammate / open gate" when the
prompt is ambiguous and a phase is in flight. Ambiguous prompts must not
silently start a new phase.

R7. The default for repeated `/superteam` invocations must be **resume**.
"Restart" requires either an explicit operator instruction or an
unambiguous new-issue signal.

R8. **State-vs-reality disagreement** must halt the run with an explicit
blocker per existing `Failure handling` rules. Examples that must halt:

- state file says `phase=plan` but no design doc exists at the
  canonical path.
- state file says `phase=finish` but no PR exists for the branch.
- state file's `branch` does not match the current branch.
- state file is missing for an issue that already has committed design
  or plan artifacts on the active branch.

Recovery is operator-driven: the operator can instruct `Team Lead` to
  reconcile the state file in a dedicated state-update commit before any
  teammate work resumes.

R9. `Team Lead` contract must be extended to:

- Run the phase-detection pre-flight before any routing decision.
- Create or update the workflow-state file as part of every state
  transition it performs.
- Treat the state file as advisory and committed artifacts + PR state
  as authoritative when reconciling.

R10. `Finisher` contract must be extended to update `finisher_state` in the
workflow-state file on every transition between `triage`, `monitoring`,
`ready`, and `blocked`, in a dedicated state-update commit when no other
artifact change is being committed.

R11. The done-report contracts for `Brainstormer`, `Planner`, and
`Executor` must include the post-handoff `state_file_sha` (the commit SHA
that updated the workflow-state file) in addition to the existing
`handoff_commit_sha`. The two SHAs may be the same when the state-file
update rides in the same commit as the artifact handoff.

R12. The new `docs/superpowers/state/` directory must be referenced from
`AGENTS.md` only if implementation discovers it is needed for repo-wide
discoverability. The design does not require an `AGENTS.md` change; the
state file convention is internal to the `superteam` workflow.

R13. All edits to `skills/superteam/SKILL.md` must go through
`superpowers:writing-skills` with pressure-test walkthroughs covering at
least the canonical cases enumerated in the Pressure Tests section below.

## Approach

### Phase-detection pre-flight

At the top of every `/superteam` invocation, before any teammate
delegation, `Team Lead` runs a deterministic detection sequence:

1. Resolve the active issue from the prompt, branch name, or operator.
2. Look for `docs/superpowers/state/<issue>.md`. If present, parse it.
3. Cross-check against committed artifacts:
   - design doc presence and SHA at the canonical specs path
   - plan doc presence and SHA at the canonical plans path
   - latest commit on branch (author, message, trailers)
4. Cross-check against PR state for the branch.
5. Reconcile:
   - if state file is absent and artifacts exist, halt with a
     `state-file-missing` blocker and ask the operator to bootstrap.
   - if state file disagrees with artifacts/PR, halt with a
     `state-vs-reality-mismatch` blocker.
   - if state file is consistent, treat its `phase`, `active_teammate`,
     `open_gate`, `open_loopback`, and `finisher_state` as the routing
     basis.
6. Classify the incoming prompt under R6.
7. Route per the table in R5.

### Workflow-state file format

Markdown with a single fenced YAML frontmatter block, then a short
human-readable transition log. Markdown is chosen because the repository
already lints markdown via `markdownlint-cli2`, and it stays human-
reviewable in PRs.

```markdown
---
issue: 39
issue_url: https://github.com/patinaproject/superteam/issues/39
branch: 39-superteam-optimize-for-repeated-superteam-invocations-and-make-workflow-state-more-durable
phase: brainstorm
active_teammate: Brainstormer
open_gate: "Gate 1: Brainstormer approval"
open_loopback: none
last_handoff_sha: <sha>
finisher_state: none
last_updated_utc: 2026-04-26T00:00:00Z
notes: design doc committed; awaiting Gate 1 approval
---

# State: #39

## Transitions

- 2026-04-26T00:00:00Z Team Lead -> Brainstormer (delegate design)
- 2026-04-26T00:30:00Z Brainstormer -> Team Lead (design doc committed, awaiting Gate 1)
```

### Prompt classification heuristic

The classifier is a small bulleted decision list in `SKILL.md`:

- If `open_gate` is set and the prompt does not contain an explicit
  approve/reject token (e.g. `approve`, `reject`, `lgtm`, `request changes`),
  treat as feedback for the gate's owning teammate.
- If `phase=execute` and prompt mentions changing requirements,
  acceptance criteria, or "what we are building", classify as
  `spec-level` loopback.
- If `phase=execute` and prompt mentions changing tasks, sequencing, or
  workstreams without changing requirements, classify as `plan-level`
  loopback.
- If `phase=execute` and prompt is a question about implementation,
  classify as implementation work for `Executor`.
- If `phase=finish` and prompt is a status, "is it done", "check CI"
  type prompt, route to `Finisher` with the existing latest-head sweep.
- If the prompt names a different issue number explicitly, require
  operator confirmation before starting a new run.
- Otherwise, treat the prompt as feedback for the active teammate.

### Resume vs restart

The default is resume. Restart requires one of:

- explicit operator instruction (e.g. `restart`, `start over`, `new run`)
- prompt clearly references a different issue number than the active
  state file
- state file is `phase=halted` and operator explicitly resumes with a
  new direction

### Approval gates remain authoritative

Adding a state file does not weaken Gate 1. The state file's
`open_gate` field reflects gate state but does not satisfy the gate.
Approval still requires the existing approval packet (artifact path,
intent summary, full requirement set, `concerns[]`).

## Pressure Tests

The following walkthroughs must pass during `superpowers:writing-skills`
review of any change to `skills/superteam/SKILL.md` produced from this
design.

PT-1. Mid-Brainstormer-approval feedback. State file shows
`phase=brainstorm`, `open_gate=Gate 1`. Operator runs
`/superteam tighten the schema, drop the notes field`. Expected:
classified as feedback, delivered to `Brainstormer` as delta-only
revision; design doc not duplicated; gate stays open until explicit
approval.

PT-2. Mid-Executor requirement change. State file shows
`phase=execute`. Operator runs `/superteam we also need to support
unkeyed state files for non-issue runs`. Expected: classified as
`spec-level` loopback, routed back through `Brainstormer` first per
existing loopback rules.

PT-3. Mid-Executor task adjustment. State file shows `phase=execute`.
Operator runs `/superteam split the SKILL.md edits into two commits`.
Expected: classified as `plan-level` loopback, routed to `Planner`.

PT-4. Post-PR Finisher monitoring. State file shows `phase=finish`,
`finisher_state=monitoring`. Operator runs `/superteam where are we`.
Expected: routed to `Finisher`; `Finisher` runs the latest-head sweep
and reports state; no restart, no new spec, no new plan.

PT-5. Ambiguous prompt during loopback. State file shows `phase=execute`,
`open_loopback=plan-level`. Operator runs `/superteam ok`. Expected:
treated as feedback for the active teammate (`Planner`), not as a
top-of-workflow request.

PT-6. State-vs-reality mismatch. State file says `phase=plan`,
`last_handoff_sha=<x>`, but no design doc exists at the canonical
specs path. Expected: halt with
`superteam halted at Team Lead: state-vs-reality-mismatch (no design doc at canonical path)`.

PT-7. New issue mid-run. State file shows `phase=execute` for issue
number 39. Operator runs `/superteam #41 something different`. Expected:
require explicit operator confirmation before starting a new run; do
not silently switch.

PT-8. Bootstrapped repo with no state file. Operator runs
`/superteam` for an issue that already has a committed design doc on
the branch but no state file. Expected: halt with
`superteam halted at Team Lead: state-file-missing` and ask the
operator to bootstrap the state file from the existing artifacts in a
dedicated state-update commit.

## Out of Scope

- Replacing `superteam` with a stateful agent harness (explicitly
  rejected in the issue's Alternatives).
- Auto-classifying prompts via an LLM call inside the skill; the
  classifier is intentionally a small deterministic checklist so
  behavior is predictable.
- Cross-issue state aggregation or a global `state/` index file.
- Changes to `AGENTS.md` (the state file convention is internal to
  `superteam`; promote to `AGENTS.md` only if a future need arises).
- Changes to the canonical roster, gates, or the loopback class set.
  This design strictly adds detection, durability, and routing on top
  of the existing contract.

## Recommended skills for implementation

- `superpowers:writing-skills` (mandatory; this work edits
  `skills/superteam/SKILL.md`).
- `superpowers:writing-plans` (Planner phase).
- `superpowers:test-driven-development` (Executor phase, ATDD against
  the pressure-test walkthroughs above).
- `superpowers:verification-before-completion` (Executor pre-handoff).
- `superpowers:requesting-code-review` and
  `superpowers:receiving-code-review` (Reviewer / Finisher).

## Acceptance criteria

### AC-39-1

`skills/superteam/SKILL.md` adds a phase-detection pre-flight that runs
at the top of every `/superteam` invocation and consults, in order, the
workflow-state file, committed design and plan artifacts, branch state,
and PR state.

### AC-39-2

`skills/superteam/SKILL.md` defines a committed workflow-state file at
`docs/superpowers/state/<issue>.md` with the schema in R3 and the
lifecycle in R4.

### AC-39-3

`skills/superteam/SKILL.md` adds a routing table covering the
`(detected_phase, prompt_classification)` pairs enumerated in R5,
including explicit handling for resume, deliver-as-feedback,
open-loopback, and new-run cases.

### AC-39-4

`skills/superteam/SKILL.md` defines a prompt-classification heuristic
that defaults to "treat as feedback for the active teammate / open
gate" when a phase is in flight and the prompt is ambiguous, and
defaults repeated invocations to resume rather than restart.

### AC-39-5

`skills/superteam/SKILL.md` requires `Team Lead` to halt with an
explicit blocker when the state file and reality disagree (state file
missing while artifacts exist, state file referencing a different
branch, state file claiming a phase whose required artifact is
missing, or state file claiming `phase=finish` with no PR).

### AC-39-6

`Team Lead` contract is extended to create or update the workflow-state
file on every state transition it performs, and `Finisher` contract is
extended to update `finisher_state` on every transition between
`triage`, `monitoring`, `ready`, and `blocked`, in a dedicated
state-update commit when no other artifact change is being committed.

### AC-39-7

The done-report contracts for `Brainstormer`, `Planner`, and
`Executor` are extended to include `state_file_sha` alongside the
existing `handoff_commit_sha`.

### AC-39-8

The eight pressure-test walkthroughs (PT-1 through PT-8) pass during
`superpowers:writing-skills` review of the resulting `SKILL.md` change,
and the Reviewer reports pass/fail per workflow-contract rules.
