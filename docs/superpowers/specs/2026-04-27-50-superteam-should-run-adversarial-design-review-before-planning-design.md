# Design: Superteam should run adversarial design review before planning [#50](https://github.com/patinaproject/superteam/issues/50)

## Intent

Add a distinct adversarial design-review gate between `Brainstormer` design
authoring and `Planner` planning. The gate should complement
`superpowers:brainstorming`: keep Brainstormer's collaborative design work and
existing `concerns[]` reporting, but require an explicit review-owned attack on
the committed design artifact before Gate 1 approval can advance.

The change should borrow the useful parts of BMad and gstack without importing
either workflow wholesale:

- a BMad-like posture that assumes material issues may exist and requires either
  actionable findings or a clean-pass rationale
- a gstack-like gate before planning, with fresh-context or specialist review
  when the host runtime can provide it
- Superpowers-native reuse of `superpowers:writing-skills`,
  `superpowers:requesting-code-review`, `superpowers:receiving-code-review`,
  `superpowers:writing-plans`, and `superpowers:verification-before-completion`

## Problem Framing

Today `Brainstormer` owns the design artifact and reports `concerns[]` in the
Gate 1 approval packet. That catches approval-relevant uncertainty, but it is
still author-local: the same teammate who authored the design decides which
concerns to surface. Superteam can therefore proceed to planning with a coherent
design that has not been attacked for missing requirements, weak assumptions,
stage-gate bypasses, or plan-readiness gaps.

The problem is sharpest for `skills/**/*.md` and workflow-contract changes.
Those designs can sound complete while still missing RED/GREEN pressure-test
obligations, rationalization closures, red flags, token-efficiency limits, or
role-specific ownership rules. Once such gaps reach `Planner`, the plan either
has to invent design authority or encode a weak contract into implementation
tasks.

## Requirements

- Preserve `Brainstormer` as the owner of the design artifact.
- Keep `concerns[]` in the approval packet as Brainstormer-owned
  approval-relevant concerns.
- Add a separate adversarial-review result that is review-owned and attacks the
  committed design artifact.
- Require the adversarial review before Gate 1 approval can advance to
  `Planner`.
- Require material findings to be dispositioned before planning.
- Require clean reviews to explain what was checked and why no material findings
  remain.
- Prefer fresh-context or parallel specialist review when the runtime supports
  it and the scope is large or workflow-critical.
- Keep a portable fallback for runtimes without fresh-context review support.
- For `skills/**/*.md` and workflow-contract surfaces, require the stricter
  `superpowers:writing-skills` review dimensions.

## Design

### Gate 1 sequence

Gate 1 should become a two-part approval gate:

1. `Brainstormer` writes and commits the design artifact.
2. `Brainstormer` runs the existing spec self-review for placeholders,
   contradictions, scope, and ambiguity.
3. `Team Lead` verifies the design artifact exists at the reported path.
4. `Team Lead` runs or dispatches adversarial design review against the
   committed artifact.
5. `Brainstormer` dispositions any material adversarial findings.
6. If the design materially changes, `Team Lead` reruns the relevant
   adversarial review dimensions or records why rerun is unnecessary.
7. `Team Lead` presents the Gate 1 approval packet with both `concerns[]` and
   the adversarial-review result.
8. Only explicit operator approval advances to `Planner`.

The adversarial-review gate is not a replacement for user approval. It gives
the operator a sharper packet to approve.

If self-review or adversarial findings change the design after the initial
commit, `Brainstormer` must commit the revised design before Gate 1 approval is
presented. Downstream teammates rely on committed branch state, so a clean
review of uncommitted changes is not enough for handoff.

### Review ownership

`Team Lead` owns enforcing the gate and selecting the review path.
`Brainstormer` owns design changes that result from findings. The adversarial
review itself may be performed by:

- a fresh-context subagent, preferred when available for workflow-critical or
  broad designs
- parallel specialist passes when dimensions are naturally separable
- a same-thread compact pass when no fresh-context runtime is available

The same-thread path is a portability fallback, not a claim that fresh-context
review is unnecessary.

The review result is Team Lead-owned gate evidence, not a new teammate role.
`Brainstormer` may report how findings were addressed, but `Team Lead` remains
responsible for verifying that the review result exists and is fit for approval.

### Review output

The Gate 1 packet should add an adversarial-review block beside the existing
Brainstormer fields:

```markdown
Adversarial review: <clean | findings dispositioned | blocked>
Reviewer context: <fresh subagent | parallel specialists | same-thread fallback>
Findings:
- <severity> <artifact section/path> - <finding>
  Disposition: <fixed in design | deferred/out of scope | rejected with rationale>
Clean-pass rationale: <required when no material findings remain>
```

`concerns[]` remains Brainstormer-owned. Adversarial findings are review-owned.
The two surfaces should not be collapsed into one generic "concerns" list.

### Review dimensions

All adversarial design reviews should check:

- requirement completeness and acceptance-criteria clarity
- weak or unstated assumptions
- simpler or safer alternatives
- failure modes and recovery paths
- teammate ownership and stage-gate boundaries
- plan readiness for `superpowers:writing-plans`
- whether the approval packet would let an operator make an informed decision

For `skills/**/*.md` and workflow-contract changes, the review must also check:

- RED-phase baseline obligation
- GREEN verification path
- rationalization-table coverage
- red-flag coverage
- loophole closure for authority, deadline, stale-context, partial-artifact, and
  "spirit not letter" evasions
- token-efficiency targets and trigger-only skill descriptions
- stage-gate bypass paths
- role ownership for finding intake, remediation, and publish follow-through

### Findings disposition

Material findings must be dispositioned before planning:

- `fixed in design`: the design changed and the finding is resolved
- `deferred/out of scope`: the design records why the issue is intentionally not
  part of this work
- `rejected with rationale`: the finding is technically evaluated and rejected

Dispositions should follow the spirit of `superpowers:receiving-code-review`:
understand the finding, verify it against the repository, evaluate whether it is
correct for this workflow, then act or push back with rationale.

### Rerun rule

When a finding causes a material design change, rerun the relevant review
dimensions before Gate 1 approval. Tiny wording fixes can record why no rerun is
needed, but requirement, ownership, pressure-test, or gate-order changes require
a rerun.

### Scope control

The gate should be proportional:

- small documentation or product changes: one compact adversarial pass
- workflow-contract or skill changes: strict `superpowers:writing-skills`
  dimensions
- large or high-risk designs: prefer fresh-context or parallel specialist review
  when available

The contract should explicitly reject endless review loops. After material
findings are fixed or dispositioned and no new material findings remain, the
gate can advance to operator approval.

## Surface Changes

Update `skills/superteam/SKILL.md`:

- Gate 1 requirements include adversarial design review before approval can
  advance to `Planner`.
- `Team Lead` owns enforcing the review gate and including the review block in
  approval packets.
- `Brainstormer` done reports include `adversarial_findings_addressed[]` when
  findings caused design changes, or an explicit empty result when no design
  changes were required by review.
- Team Lead approval packets include `adversarial_review_status`,
  `adversarial_findings[]`, and `clean_pass_rationale` when applicable.
- Rationalization table and red flags close bypasses such as "concerns[] is the
  same as review" and "same-thread review is always enough."

Update `skills/superteam/agent-spawn-template.md`:

- Team Lead approval packets require the adversarial-review block.
- Brainstormer done reports include finding-addressed state for design changes
  made in response to adversarial review.
- Brainstormer prompts state that `concerns[]` does not satisfy adversarial
  review.

Update `docs/superpowers/pressure-tests/superteam-orchestration-contract.md`:

- Add pressure tests for skipped adversarial review, collapsed review/concerns,
  clean pass without rationale, material findings without disposition, workflow
  surfaces reviewed without `writing-skills` dimensions, material design changes
  without rerun, and runtime support treated as a hard dependency.

## Loophole-Closure Language

The workflow must close these rationalizations:

| Excuse | Reality |
|--------|---------|
| "`concerns[]` already covers review." | `concerns[]` is Brainstormer-owned. Adversarial design review is a separate review-owned attack on the committed artifact. |
| "The operator can catch design gaps during approval." | Gate 1 should present an already-attacked design; operator approval is not a substitute for required review. |
| "No findings means no review block is needed." | A clean pass must include the checked dimensions and a rationale for why no material findings remain. |
| "Same-thread review is good enough even for workflow contracts." | Fresh-context review is preferred for workflow-critical or broad designs when available; same-thread review is only the portable fallback. |
| "A finding was fixed, so the original review still applies." | Material design changes require rerunning the affected review dimensions or recording why rerun is unnecessary. |
| "Adversarial review can become an endless loop." | Once material findings are fixed or dispositioned and no new material findings remain, the gate advances to operator approval. |

## Red Flags

- Gate 1 approval packet has `concerns[]` but no adversarial-review block.
- Adversarial review reports "clean" without naming checked dimensions.
- Material findings are listed without dispositions.
- `Planner` starts from a design with unresolved material adversarial findings.
- Skill or workflow-contract designs skip RED/GREEN pressure-test obligations.
- Requirement, ownership, pressure-test, or gate-order changes land after review
  without rerunning affected review dimensions.
- Runtime fresh-context support is treated as mandatory for all runs, or ignored
  when available for high-risk designs.

## RED-Phase Baseline Obligation

Before implementing the rule, capture evidence that the current contract allows
Gate 1 to proceed without a distinct adversarial design-review result. The
baseline may be live behavioral evidence or inspection-based evidence, but it
must show all of the following against the unchanged contract:

1. Gate 1 approval requires artifact path, intent summary, requirements,
   `concerns[]`, and `Remaining concerns: None`, but not adversarial review.
2. `Brainstormer` done reports require design path, AC IDs, requirements,
   `concerns[]`, and handoff SHA, but not review findings or clean-pass
   rationale.
3. Existing pressure tests cover approval packet shape and `concerns[]`, but do
   not fail a packet that omits adversarial review.

The plan should record the baseline evidence before editing workflow surfaces.
GREEN verification should then show the same scenarios now require the
adversarial-review block and halt or reroute when it is missing.

## Adversarial Review of This Design

This section applies the proposed strategy to this design before planning. The
review context is `same-thread fallback` because the issue is still in
Brainstormer design drafting and no committed design artifact existed when the
operator required the review. That is intentionally weaker than the desired
fresh-context path, so the Gate 1 packet should report it plainly.

Review dimensions checked:

- requirement completeness and acceptance-criteria clarity
- ownership boundaries between `Team Lead`, `Brainstormer`, `Planner`, and
  local `Reviewer`
- plan readiness for `superpowers:writing-plans`
- workflow-contract pressure-test obligations from `superpowers:writing-skills`
- loopholes around `concerns[]`, uncommitted artifacts, runtime dependency, and
  endless review loops

Findings:

- High `## Gate 1 sequence` - The original sequence committed the design before
  spec self-review and did not state that review-driven edits must be committed
  before approval. That could let a clean review describe uncommitted state.
  Disposition: fixed in design by requiring revised design commits before Gate
  1 approval.
- High `## Surface Changes` - The original done-report proposal put
  `adversarial_review_status` on `Brainstormer`, which blurred the review-owned
  gate evidence with Brainstormer-owned remediation. Disposition: fixed in
  design by keeping review status on the Team Lead approval packet and limiting
  Brainstormer to findings-addressed state.
- Medium `## Review ownership` - The draft said review-owned, but did not say
  whether this creates a new teammate or reuses existing ownership. Disposition:
  fixed in design by naming the review result as Team Lead-owned gate evidence,
  not a new teammate role.
- Medium `## RED-Phase Baseline Obligation` - The design explains future
  baseline evidence, but this first draft had no way to apply the new rule to
  itself before a committed artifact existed. Disposition: fixed in design by
  recording this same-thread fallback review and requiring the Gate 1 packet to
  report that weaker context.
- Low `## Scope control` - The design rejects endless review loops, but an
  implementation plan could still over-specify repeated review. Disposition:
  deferred to planning; the plan should include one bounded rerun rule and avoid
  open-ended specialist loops.

Self-analysis:

- What works: the proposed strategy caught ownership and artifact-state bugs
  that normal `concerns[]` reporting could easily miss. It forced traceable
  dispositions and made the review context visible instead of pretending a
  same-thread review has fresh-context strength.
- What does not work yet: same-thread review is useful for bootstrapping, but it
  is not the target quality bar for broad or workflow-critical designs. A later
  implementation should prefer a fresh-context pass when the runtime supports
  it.
- Potential gap: requiring review before approval adds another gate that could
  slow small changes. The proportionality rule and bounded rerun rule are
  load-bearing and should be kept concise in implementation.
- Potential gap: review findings can become a second requirements source if
  dispositions are vague. The implementation should require requirement-bearing
  findings to update the design first, then planning, preserving the spec-first
  loop.

## Acceptance Criteria

### AC-50-1

**Given** `Brainstormer` has authored and committed a design artifact,
**When** `Team Lead` prepares Gate 1 approval,
**Then** a distinct adversarial design-review result is required before
planning can start.

### AC-50-2

**Given** the adversarial review finds material issues,
**When** Gate 1 approval is presented,
**Then** those findings are surfaced separately from `concerns[]` and each
finding has a disposition before `Planner` is invoked.

### AC-50-3

**Given** the adversarial review reports no material issues,
**When** Gate 1 approval is presented,
**Then** the packet includes an explicit clean-pass rationale rather than
silently omitting review findings.

### AC-50-4

**Given** a design touches `skills/**/*.md` or a workflow-contract surface,
**When** adversarial review runs,
**Then** it checks loophole closure, rationalization resistance, red-flag
prompts, token-efficiency targets, RED/GREEN pressure-test obligations, role
ownership, and stage-gate bypass paths.

### AC-50-5

**Given** a design is revised to address adversarial findings,
**When** the changes are material,
**Then** the workflow either reruns the relevant adversarial review dimensions
or records why a rerun is unnecessary.

### AC-50-6

**Given** the host runtime supports suitable fresh-context or parallel
specialist review,
**When** the design review scope is large or workflow-critical,
**Then** Superteam prefers that capability without making it a portability
requirement.

### AC-50-7

**Given** the implementation changes workflow-contract guidance,
**When** verification runs,
**Then** the relevant pressure-test walkthroughs cover skipped review, collapsed
review/concerns, missing clean-pass rationale, missing finding dispositions,
missing workflow-contract dimensions, missing rerun after material changes, and
runtime support overreach.

## Out of Scope

- Importing BMad or gstack as dependencies.
- Replacing `superpowers:brainstorming`.
- Moving implementation review into the design stage.
- Changing the canonical teammate roster.
- Requiring subagents in runtimes that do not provide them.
