# Design: Brainstormer commits write `Loopback:` in body prose, not as a real git trailer [#57](https://github.com/patinaproject/superteam/issues/57)

## Intent

Prevent artifact-producing teammates from reporting loopback handoffs as done
when their `Loopback:` line is body prose instead of a parseable git trailer.
The fix should make correct placement the default path, make parseability
evidence part of the handoff contract, and make malformed loopback commits a
pre-flight contract violation rather than a quiet state-loss bug.

This is a workflow-contract and skill-surface change, so the implementation must
follow `superpowers:writing-skills`: capture the current RED behavior before
editing the contract, close the rationalization that "near the bottom is close
enough," and verify GREEN behavior with trailer-parser evidence instead of
confidence language.

## Problem

The current loopback contract says `Loopback:` must be a git trailer, but the
spawn guidance does not make the final contiguous trailer block hard to get
right. During active loopbacks, a teammate can produce a message shaped like:

```text
fix: #57 update design

Body paragraph.

Loopback: spec-level

Co-Authored-By: teammate <noreply@example.com>
```

Git treats only the final contiguous `Key: value` paragraph as the trailer
block. In that shape, `Co-Authored-By:` is parseable but `Loopback:` is body
text. `git log --format='%(trailers:only,key=Loopback)'` returns nothing, so
fresh pre-flight cannot recover the active loopback class.

The cost shows up downstream: `Team Lead` must detect the miss after the fact,
rewrite commits with `git interpret-trailers`, and risk rebase and hook churn.
The right fix is upstream prevention plus early detection, not more salvage
logic.

## Decision

Tighten the contract in three places.

1. `agent-spawn-template.md` should show the exact final trailer block shape for
   active loopback commits and require `git interpret-trailers` for adding
   `Loopback:`. The example must keep `Co-Authored-By:` and `Loopback:` in the
   same final trailer block with no blank line between them.
2. Artifact-producing teammate contracts should require parseability evidence
   before done reports during an active loopback. Brainstormer, Planner, and
   Executor must run the parser check on each loopback-originated commit they
   hand off and report either the `git log %(trailers)` output or an explicit
   parseable confirmation.
3. `pre-flight.md` and `loopback-trailers.md` should define malformed
   loopback-looking body prose as a contract violation when it appears in a
   branch-only active-issue commit without a parseable `Loopback:` trailer.
   Pre-flight should surface the violation and prevent the affected teammate
   from claiming done until the commit is fixed.

The recovery algorithm remains pure `git log` for valid trailers. The new
malformation check is a guardrail around commits that mention a loopback line
for the active issue but fail the parser check. It must not invent a new state
store, silently infer loopback state from body prose, or rewrite history.

## Surfaces

### `skills/superteam/agent-spawn-template.md`

Add loopback-specific commit guidance to the hard rules or shared prompt block:

- When a delegation is inside an active loopback, commit messages must place
  `Loopback: <class>` or `Loopback: resolved` in the final contiguous trailer
  block.
- Prefer a mechanical construction with `git interpret-trailers --trailer
  "Loopback: <class>"` rather than hand-typing the footer.
- If another trailer such as `Co-Authored-By:` is present, `Loopback:` belongs
  in the same final trailer block with no blank line between trailers.
- Include a compact worked example:

```text
<subject>

<body paragraph>

Co-Authored-By: teammate <noreply@example.com>
Loopback: spec-level
```

The example should not include a blank line between the two trailers.

### `skills/superteam/SKILL.md`

Update Brainstormer, Planner, and Executor contracts:

- During an active loopback, any loopback-originated commit they hand off must
  include a parseable `Loopback:` trailer.
- Their done report must include the parser evidence or explicit parseable
  confirmation for loopback-originated commits.
- They must refuse to report done if
  `git log -1 --format='%(trailers:only,key=Loopback)'` is empty for the commit
  they just produced in the loopback.

Update Team Lead and pre-flight guidance:

- If branch-only commits for the active issue contain a body line that looks
  like `Loopback: spec-level`, `Loopback: plan-level`,
  `Loopback: implementation-level`, or `Loopback: resolved`, but
  `%(trailers:key=Loopback)` is empty for that commit, halt with an explicit
  contract violation instead of recovering state from body prose.
- The halt should route the fix back to the teammate that authored or handed off
  the affected commit when that can be determined from the phase/loopback
  context; otherwise report the commit SHA and require operator intervention.

Add a rationalization-table row:

| Excuse | Reality |
|--------|---------|
| "I put `Loopback:` near the bottom; that's basically a trailer." | Git's trailer parser only recognizes the final contiguous `Key: value` block. A blank line before another trailer leaves `Loopback:` in body prose, invisible to `git log %(trailers)` and to loopback-class recovery. |

Add a red flag:

- Brainstormer, Planner, or Executor commits during an active loopback and
  `git log -1 --format='%(trailers:only,key=Loopback)'` is empty.

### `skills/superteam/loopback-trailers.md`

Clarify that a prose line matching `Loopback: <value>` is not a weak fallback
signal. It is invalid evidence. Recovery must ignore it for state derivation,
and pre-flight should surface it as a malformed loopback contract violation
when it appears in an active-issue branch-only commit.

### `skills/superteam/pre-flight.md`

Extend loopback-class recovery with a malformed-trailer guard:

- Scope the scan to the same branch-only active-issue range used by loopback
  recovery.
- For commits whose subject references the active issue, detect body prose lines
  that exactly match the loopback grammar while parser output for
  `key=Loopback` is empty.
- Halt with a clear message, for example:
  `superteam halted at Team Lead: malformed Loopback line in body prose on <sha>; fix the commit trailer before resuming`.

This guard should run before normal routing so status, CI, publish, and "is it
done" prompts cannot fall through to a stale or missing loopback class.

### `docs/superpowers/pressure-tests/superteam-orchestration-contract.md`

Add pressure tests covering:

- A teammate commit whose `Loopback:` line is separated from
  `Co-Authored-By:` by a blank line. Required behavior: parser check fails,
  teammate cannot report done, and pre-flight halts if the malformed commit is
  already present.
- A teammate commit built with `git interpret-trailers` where `Loopback:` and
  `Co-Authored-By:` are in one final trailer block. Required behavior:
  `git log %(trailers:only,key=Loopback)` returns the expected class and the
  handoff may continue.
- A pre-flight scan over branch-only commits where body prose contains a
  loopback-looking line but parser output is empty. Required behavior: halt
  rather than recovering loopback state from the body line.

### RED-phase baseline artifact

Before editing the workflow contract, add a baseline artifact under
`docs/superpowers/baselines/` that records the observed failure:

- a bad commit message with a blank line between `Loopback:` and
  `Co-Authored-By:`
- the parser output showing no `Loopback:` trailer
- the rationalization an agent could use to claim the footer was present
- the target GREEN parser output expected after the contract change

## Acceptance Criteria

### AC-57-1

Given `Brainstormer` is dispatched during an active spec-level loopback, when it
commits the design artifact and reports done, then
`git log -1 --format='%(trailers:only,key=Loopback)'` on that commit returns
`Loopback: spec-level`.

- [ ] Capture the RED baseline showing the current bad shape returns empty
      parser output.
- [ ] Verify the updated Brainstormer contract requires parseable trailer
      evidence before done.

### AC-57-2

Given any artifact-producing teammate emits a commit during an active loopback,
when they report done, then their done report includes either parser output or
an explicit confirmation that the trailer is parseable by git's trailer parser.

- [ ] Verify Brainstormer, Planner, and Executor done-report contracts include
      loopback trailer parseability evidence.
- [ ] Verify the agent-spawn template carries the same expectation into
      delegated work.

### AC-57-3

Given the agent-spawn template is updated, when a fresh agent reads it, then the
worked example shows `Loopback:` and `Co-Authored-By:` in the same trailer block
with no blank line between them.

- [ ] Inspect `skills/superteam/agent-spawn-template.md` and confirm the worked
      example has one final contiguous trailer block.

### AC-57-4

Given `Team Lead` reviews an active loopback's commits during pre-flight, when
any active-issue branch-only commit contains a loopback-looking body line but
`git log %(trailers)` does not include the expected `Loopback:` key, then the
workflow surfaces the contract violation and prevents the affected teammate from
claiming done without a trailer fix.

- [ ] Verify `pre-flight.md` documents the malformed-trailer halt.
- [ ] Verify the pressure-test walkthrough covers malformed body prose and
      parseable trailer success.

## Requirements

- Keep loopback state recoverable from git trailers, not sidecar files or chat
  memory.
- Do not recover state from malformed body prose. Body prose is evidence of a
  contract violation, not an alternate parser.
- Use `git interpret-trailers` as the preferred mechanical path for adding
  `Loopback:`.
- Keep `Loopback:` adjacent to other trailers in the final trailer block when
  multiple trailers are present.
- Require Brainstormer, Planner, and Executor to verify parseability before
  loopback done reports.
- Extend pre-flight to detect malformed active-issue loopback body prose before
  routing.
- Add pressure tests and a RED baseline before claiming the workflow-contract
  change is ready.
- Preserve existing loopback grammar and `resolved` precedence from #41.

## Non-Goals

- Do not change the `Loopback:` grammar.
- Do not add `Team Lead` history-rewrite or salvage automation.
- Do not introduce a sidecar state file.
- Do not require every non-loopback commit to run trailer checks.
- Do not alter #41's rule that `Loopback: resolved` wins on a commit that also
  carries class trailers.

## Writing-Skills Review Dimensions

- RED baseline obligation: required before contract edits.
- GREEN verification path: parser output and pressure-test walkthroughs, not
  confidence language.
- Rationalization resistance: explicitly closes "near the bottom is close
  enough."
- Red flags: parser output empty after loopback teammate commit.
- Token efficiency: add compact rules and examples only where the agent needs
  them at dispatch or handoff time.
- Role ownership: teammates verify their own loopback commit before done; Team
  Lead halts malformed committed state during pre-flight; Reviewer later reruns
  pressure-test walkthroughs for the workflow-contract change.
- Stage-gate bypass paths: malformed trailers block handoff and pre-flight
  routing, including status and publish prompts.

## Open Questions

None. The issue describes the failing shape, the parser command, and the desired
prevention path clearly enough for planning.
