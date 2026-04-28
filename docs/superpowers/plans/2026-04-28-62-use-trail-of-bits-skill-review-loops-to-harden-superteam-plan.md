# Plan: Use Trail of Bits skill review loops to harden Superteam [#62](https://github.com/patinaproject/superteam/issues/62)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a repeatable, evidence-producing Trail of Bits-inspired skill review loop to Superteam's workflow-contract surfaces.

**Architecture:** This is a workflow-contract documentation change. Add one concise Superteam-owned review-loop guide, link it from the canonical skill, require the loop in relevant delegated role prompts, and extend pressure tests so the new discipline has RED/GREEN coverage without making Trail of Bits plugins a shipped dependency.

**Tech Stack:** Markdown, GitHub CLI, `markdownlint-cli2`, repo-local `pnpm` scripts, Git commit-message hooks.

---

## Source Documents

- Issue: `#62`
- Design: `docs/superpowers/specs/2026-04-28-62-use-trail-of-bits-skill-review-loops-to-harden-superteam-design.md`
- Core contract: `skills/superteam/SKILL.md`
- Delegation prompts: `skills/superteam/agent-spawn-template.md`
- Pressure tests: `docs/superpowers/pressure-tests/superteam-orchestration-contract.md`
- Trail of Bits workflow design skill: `workflow-skill-design/designing-workflow-skills`
- Trail of Bits workflow reviewer agent: `workflow-skill-design/workflow-skill-reviewer`
- Trail of Bits skill improver skill: `skill-improver/skill-improver`
- Repo rules: `AGENTS.md`

## File Structure

- Create `skills/superteam/skill-quality-review.md`: concise, repo-owned review-loop contract adapted from Trail of Bits workflow-skill reviewer and skill-improver guidance.
- Create `docs/superpowers/reviews/2026-04-28-62-trail-of-bits-skill-review.md`: durable review evidence for this issue, including RED baseline, findings, minor finding evaluation, and conflict disposition.
- Modify `skills/superteam/SKILL.md`: link the review-loop guide, require it for skill/workflow-contract changes, and add rationalization and red-flag coverage.
- Modify `skills/superteam/agent-spawn-template.md`: make Executor and Reviewer prompts carry the new review-loop guidance for workflow-contract changes.
- Modify `docs/superpowers/pressure-tests/superteam-orchestration-contract.md`: add scenarios proving the loop cannot be skipped, findings cannot be vague, minor findings cannot be applied blindly, and local contracts win over external guidance.
- Modify `.github/pull_request_template.md` only if execution discovers the template cannot express AC-62 evidence clearly. The current template already supports AC evidence, so this is expected to remain unchanged.

## RED Baseline Evidence

Before editing workflow surfaces, capture the unchanged state from the current branch after the design commit.

- [ ] **Step 1: Confirm no Superteam-owned skill-quality review guide exists**

Run:

```bash
test ! -f skills/superteam/skill-quality-review.md
```

Expected RED evidence: exit code 0, proving no repeatable Superteam-owned Trail of Bits adaptation guide exists.

- [ ] **Step 2: Confirm current Superteam references do not name Trail of Bits review loops**

Run:

```bash
rg -n "Trail of Bits|workflow-skill-reviewer|skill-improver|skill-quality-review" skills/superteam docs/superpowers/pressure-tests/superteam-orchestration-contract.md
```

Expected RED evidence: no output and exit code 1, proving the current workflow does not name the requested review/improvement loop.

- [ ] **Step 3: Confirm pressure tests lack the requested evidence contract**

Run:

```bash
rg -n "minor findings|severity.*disposition|skill-quality review|Trail of Bits" docs/superpowers/pressure-tests/superteam-orchestration-contract.md
```

Expected RED evidence: no output and exit code 1, proving the pressure-test suite does not yet cover severity/disposition, minor finding evaluation, or Trail of Bits adaptation.

Record these command outputs in `docs/superpowers/reviews/2026-04-28-62-trail-of-bits-skill-review.md`.

## Task 1: Add Durable Review Evidence

**Files:**

- Create: `docs/superpowers/reviews/2026-04-28-62-trail-of-bits-skill-review.md`

- [ ] **Step 1: Create the review evidence file**

Create the file with this content:

```markdown
# Review: Trail of Bits skill loop hardening [#62](https://github.com/patinaproject/superteam/issues/62)

## Purpose

Record the adapted Trail of Bits workflow-skill review and skill-improver evidence for Superteam issue #62.

## RED Baseline

| Check | Command | Expected RED result | Observed |
| --- | --- | --- | --- |
| Missing local review guide | `test ! -f skills/superteam/skill-quality-review.md` | Exit code 0 | RED confirmed before implementation: exit code 0, no stdout |
| Missing Trail of Bits loop references | `rg -n "Trail of Bits\|workflow-skill-reviewer\|skill-improver\|skill-quality-review" skills/superteam docs/superpowers/pressure-tests/superteam-orchestration-contract.md` | Exit code 1, no output | RED confirmed before implementation: exit code 1, no stdout |
| Missing pressure tests | `rg -n "minor findings\|severity.*disposition\|skill-quality review\|Trail of Bits" docs/superpowers/pressure-tests/superteam-orchestration-contract.md` | Exit code 1, no output | RED confirmed before implementation: exit code 1, no stdout |

## Adapted Review Sources

- Trail of Bits `workflow-skill-reviewer`: structural analysis, workflow pattern analysis, content quality, tool assignment, and anti-pattern scan.
- Trail of Bits `skill-improver`: critical and major findings must be fixed or explicitly dispositioned; minor findings are evaluated before applying.
- Superteam local contract: review evidence feeds Brainstormer, Planner, Executor, Reviewer, and Finisher; it does not replace them.

## Findings

| ID | Severity | Surface | Finding | Disposition | Verification |
| --- | --- | --- | --- | --- | --- |
| F-62-1 | major | `skills/superteam` | No repo-owned adaptation guide currently tells Superteam reviewers how to run or record the Trail of Bits workflow-skill review loop. | Fix by adding `skills/superteam/skill-quality-review.md`. | `rg -n "skill-quality-review|Trail of Bits" skills/superteam` |
| F-62-2 | major | `skills/superteam/SKILL.md` and `skills/superteam/agent-spawn-template.md` | Skill/workflow-contract changes do not yet require severity, disposition, and minor-finding evaluation from the adapted loop. | Fix by linking the review guide and updating role guidance. | `rg -n "severity|disposition|minor findings" skills/superteam/SKILL.md skills/superteam/agent-spawn-template.md` |
| F-62-3 | major | `docs/superpowers/pressure-tests/superteam-orchestration-contract.md` | Existing pressure tests do not fail the shortcuts introduced by this issue. | Fix by adding pressure tests for skipped loop, vague findings, blind minor fixes, and local-contract conflicts. | `rg -n "Trail of Bits|minor findings|local contract" docs/superpowers/pressure-tests/superteam-orchestration-contract.md` |

## Minor Finding Evaluation

No minor findings are accepted by default. During execution, add one row per minor finding discovered by the adapted review loop.

| ID | Surface | Finding | Evaluation | Decision |
| --- | --- | --- | --- | --- |

## Conflict Disposition

Trail of Bits guidance is advisory when it conflicts with Superteam's local contracts. Superteam preserves committed design and plan artifacts, visible-state resume, Reviewer before Finisher, latest-head Finisher shutdown, and the prohibition on `Loopback:` trailers or hidden workflow-state markers.

## Completion Marker

The adapted skill-improver loop is complete only when no critical or major finding remains open, and all minor findings have been evaluated for usefulness.
```

- [ ] **Step 2: Verify baseline observations are specific**

Read the `## RED Baseline` table and confirm each row contains an observed exit code.

- [ ] **Step 3: Verify RED observations are present**

Run:

```bash
rg -n "RED confirmed before implementation" docs/superpowers/reviews/2026-04-28-62-trail-of-bits-skill-review.md
```

Expected: three matches in the `## RED Baseline` table.

## Task 2: Add the Superteam Skill-Quality Review Guide

**Files:**

- Create: `skills/superteam/skill-quality-review.md`

- [ ] **Step 1: Create the guide**

Create `skills/superteam/skill-quality-review.md` with this content:

```markdown
# Skill Quality Review

Use this guide when a Superteam run changes `skills/superteam/SKILL.md`, `skills/superteam/*.md`, `docs/superpowers/pressure-tests/*.md`, or another repository-owned workflow-contract surface.

The guide adapts Trail of Bits workflow-skill review and skill-improver discipline into Superteam's local workflow. It is a review method, not a runtime dependency.

## Review Sequence

1. Confirm the changed surfaces and read the governing repository rules.
2. Apply the workflow-skill review categories:
   - frontmatter and trigger description quality
   - workflow pattern and phase/gate clarity
   - progressive disclosure and reference size
   - file-reference integrity
   - tool or agent portability
   - anti-patterns, especially reference dumps, hidden state, and unbounded delegation
3. Categorize findings:
   - `critical`: blocks loading, references missing files, or makes the workflow impossible to execute
   - `major`: materially weakens triggering, gates, role ownership, verification, or handoff safety
   - `minor`: polish, style, or optional clarity improvement
4. Fix or explicitly disposition every critical and major finding.
5. Evaluate every minor finding before applying it. Apply only when it improves execution reliability, evidence quality, or operator clarity.
6. Record evidence in an issue-specific review artifact or PR acceptance-criteria section.

## Required Evidence

Each finding needs:

- severity: `critical`, `major`, or `minor`
- affected surface
- finding summary
- disposition: `fixed`, `not applicable`, `deferred with rationale`, or `accepted minor`
- verification command, pressure-test scenario, or review evidence

## Local Contract Priority

When Trail of Bits guidance conflicts with Superteam's local contracts, keep the local contract and record the decision. Local contracts include committed design and plan artifacts, visible-state resume, no hidden routing markers, Reviewer before Finisher, and latest-head Finisher shutdown.

## Completion Criteria

The adapted loop is complete when:

- no critical or major finding remains open
- every minor finding has been evaluated for usefulness
- pressure tests cover any new workflow discipline rule
- the PR body maps acceptance criteria to review evidence that actually ran
```

- [ ] **Step 2: Verify the guide covers required terms**

Run:

```bash
rg -n "critical|major|minor|disposition|Local Contract Priority|Completion Criteria" skills/superteam/skill-quality-review.md
```

Expected: matches in the categorization, evidence, local contract, and completion sections.

## Task 3: Wire the Guide into Superteam Contracts

**Files:**

- Modify: `skills/superteam/SKILL.md`
- Modify: `skills/superteam/agent-spawn-template.md`

- [ ] **Step 1: Add a skill-quality review section to `SKILL.md`**

Insert this section after `## Canonical rule discovery`:

```markdown
## Skill-quality review loop

When a run changes `skills/superteam/SKILL.md`, adjacent `skills/superteam/*.md` workflow-contract files, or repo-owned pressure tests for this workflow, use `skill-quality-review.md` as the local adaptation of Trail of Bits workflow-skill review and skill-improver discipline.

This loop feeds Superteam's normal teammate workflow. It does not replace `Brainstormer`, `Planner`, `Executor`, `Reviewer`, or `Finisher`.

Required outcomes:

- Record each finding with severity, affected surface, disposition, and verification evidence.
- Fix or explicitly disposition every critical and major finding before publish.
- Evaluate minor findings for usefulness before applying them.
- Preserve local contracts when external skill-review guidance conflicts with Superteam.
- Keep review evidence in durable artifacts, PR acceptance criteria, or other inspectable records rather than volatile chat context.
```

- [ ] **Step 2: Add Reviewer contract guidance**

In `### Reviewer`, after the existing `superpowers:writing-skills` recommendation, add:

```markdown
- For Superteam workflow-contract changes, apply `skill-quality-review.md` before publish and verify critical/major findings are fixed or dispositioned.
- Evaluate minor findings for usefulness instead of applying them blindly.
- Treat the Trail of Bits-inspired review loop as evidence for local review, not as a replacement for local review ownership.
```

- [ ] **Step 3: Add Executor contract guidance**

In `### Executor`, after the existing skill recommendations, add:

```markdown
- When implementing a workflow-contract change, produce or update the issue-specific review evidence required by `skill-quality-review.md`.
```

- [ ] **Step 4: Add rationalization rows**

Append these rows to `## Rationalization table`:

```markdown
| "The Trail of Bits loop ran, so Reviewer and Finisher can be skipped." | The loop is evidence for Superteam's local stages, not a replacement for Reviewer or latest-head Finisher follow-through. |
| "A minor skill-review finding came from a reviewer, so we should apply it automatically." | Minor findings can be false positives or style preferences. Evaluate usefulness before applying. |
| "The upstream plugin is unavailable, so we can omit the evidence." | `skill-quality-review.md` is the repo-owned adaptation. Use it and record any upstream-tool limitation. |
```

- [ ] **Step 5: Add red flags**

Append these bullets to `## Red flags`:

```markdown
- Superteam workflow-contract changes publish without `skill-quality-review.md` evidence.
- Critical or major skill-review findings remain open without a blocker or explicit disposition.
- Minor skill-review findings are applied automatically without usefulness evaluation.
- Trail of Bits guidance overrides a local Superteam contract without a recorded conflict disposition.
```

- [ ] **Step 6: Link the guide in Supporting files**

Append this bullet to `## Supporting files`:

```markdown
- [skill-quality-review.md](./skill-quality-review.md): local Trail of Bits-inspired skill review and improvement loop
```

- [ ] **Step 7: Update `agent-spawn-template.md` Reviewer block**

In the `### Reviewer` role-specific block, after the existing `superpowers:writing-skills` recommendation, add:

```text
For Superteam workflow-contract changes, apply `skills/superteam/skill-quality-review.md` before publish. Verify critical and major findings are fixed or explicitly dispositioned, evaluate minor findings for usefulness, and keep evidence durable in an issue-specific review artifact, PR acceptance criteria, or another inspectable record.
```

- [ ] **Step 8: Update `agent-spawn-template.md` Executor block**

In the `### Executor` role-specific block, after the existing `superpowers:writing-skills` recommendation, add:

```text
When implementation changes Superteam workflow-contract surfaces, produce or update the issue-specific review evidence required by `skills/superteam/skill-quality-review.md`.
```

- [ ] **Step 9: Verify contract wiring**

Run:

```bash
rg -n "skill-quality-review|critical.*major|minor findings|Trail of Bits" skills/superteam/SKILL.md skills/superteam/agent-spawn-template.md
```

Expected: matches in the new skill-quality review section, Reviewer and Executor guidance, rationalization table, red flags, supporting files, and delegated prompt blocks.

## Task 4: Add Pressure-Test Coverage

**Files:**

- Modify: `docs/superpowers/pressure-tests/superteam-orchestration-contract.md`

- [ ] **Step 1: Add skipped-loop pressure test**

Append this scenario:

```markdown
## Superteam workflow-contract change skips Trail of Bits-inspired review loop

- Starting condition: A run changes `skills/superteam/SKILL.md`, adjacent `skills/superteam/*.md`, or Superteam pressure tests, but no `skill-quality-review.md` evidence exists in a durable artifact, PR acceptance criteria, or other inspectable record.
- Required halt or reroute behavior: Halt publish readiness and route back through local `Reviewer` until the adapted review loop has run or the blocker is explicitly reported.
- Rule surface: Superteam workflow-contract changes require the repo-owned Trail of Bits-inspired skill-quality review loop before publish.
```

- [ ] **Step 2: Add vague-finding pressure test**

Append this scenario:

```markdown
## Skill-quality review findings lack severity or disposition

- Starting condition: A skill-quality review says the Superteam skill was reviewed, but findings lack severity, affected surface, disposition, or verification evidence.
- Required halt or reroute behavior: Halt the handoff and require the review evidence to classify each finding before publish.
- Rule surface: `skill-quality-review.md` requires severity, affected surface, disposition, and verification evidence for every finding.
```

- [ ] **Step 3: Add minor-finding pressure test**

Append this scenario:

```markdown
## Minor skill-review findings applied blindly

- Starting condition: A reviewer reports minor skill-quality findings and Executor applies them without evaluating whether they improve execution reliability, evidence quality, or operator clarity.
- Required halt or reroute behavior: Halt or reroute the change until each minor finding is evaluated and either accepted with rationale or rejected as not useful.
- Rule surface: The adapted skill-improver loop requires minor findings to be evaluated before implementation.
```

- [ ] **Step 4: Add local-contract conflict pressure test**

Append this scenario:

```markdown
## External skill-review guidance conflicts with Superteam local contracts

- Starting condition: Trail of Bits-inspired guidance suggests a change that would bypass committed design/plan artifacts, visible-state resume, local Reviewer ownership, latest-head Finisher shutdown, or the prohibition on hidden routing markers.
- Required halt or reroute behavior: Preserve the Superteam local contract and record the conflict disposition before publish.
- Rule surface: `skill-quality-review.md` treats Trail of Bits guidance as an adapted review method, not a higher-priority runtime contract.
```

- [ ] **Step 5: Verify pressure tests**

Run:

```bash
rg -n "Trail of Bits-inspired|severity or disposition|Minor skill-review findings|local contracts" docs/superpowers/pressure-tests/superteam-orchestration-contract.md
```

Expected: matches for all four new scenarios.

## Task 5: Verify, Commit, and Hand Off to Reviewer

**Files:**

- All files changed in Tasks 1-4

- [ ] **Step 1: Verify forbidden hidden-state trailers were not revived**

Run:

```bash
git diff -- docs/superpowers/reviews/2026-04-28-62-trail-of-bits-skill-review.md skills/superteam/skill-quality-review.md skills/superteam/SKILL.md skills/superteam/agent-spawn-template.md docs/superpowers/pressure-tests/superteam-orchestration-contract.md | rg "Loopback: spec-level|Loopback: plan-level|Loopback: implementation-level"
```

Expected: no output and exit code 1.

- [ ] **Step 2: Verify targeted AC evidence**

Run:

```bash
rg -n "F-62-|critical|major|minor|disposition|skill-quality-review|Trail of Bits-inspired" docs/superpowers/reviews/2026-04-28-62-trail-of-bits-skill-review.md skills/superteam docs/superpowers/pressure-tests/superteam-orchestration-contract.md
```

Expected: matches in the review artifact, skill-quality guide, Superteam contract, delegated prompts, and pressure tests.

- [ ] **Step 3: Run Markdown lint**

Run:

```bash
pnpm lint:md
```

Expected: exit code 0 and `Summary: 0 error(s)`.

- [ ] **Step 4: Verify plugin versions**

Run:

```bash
node scripts/check-plugin-versions.mjs
```

Expected: `check-plugin-versions: all manifests at 1.3.0`.

- [ ] **Step 5: Commit implementation**

Run:

```bash
git status --short
git add docs/superpowers/reviews/2026-04-28-62-trail-of-bits-skill-review.md skills/superteam/skill-quality-review.md skills/superteam/SKILL.md skills/superteam/agent-spawn-template.md docs/superpowers/pressure-tests/superteam-orchestration-contract.md
git commit -m "feat: #62 add superteam skill review loop"
```

Expected: commit succeeds after pre-commit lint and version checks.

- [ ] **Step 6: Hand off to Reviewer**

Reviewer must use `superpowers:requesting-code-review`, `superpowers:receiving-code-review` when interpreting any disputed finding, and `superpowers:writing-skills` for the changed skill/workflow-contract surfaces. Reviewer must also apply `skills/superteam/skill-quality-review.md` and rerun the relevant pressure-test walkthrough before Finisher publishes.

## Plan Self-Review

- Spec coverage: Requirements 1-9 map to Tasks 1-5. AC-62-1 through AC-62-5 map to the review artifact, skill-quality guide, contract wiring, pressure tests, and PR evidence requirements.
- Placeholder scan: Task 1 includes concrete RED observations from the pre-implementation state.
- Type and naming consistency: The new guide is consistently named `skills/superteam/skill-quality-review.md`; the durable review artifact is consistently named `docs/superpowers/reviews/2026-04-28-62-trail-of-bits-skill-review.md`.
- Plan adjustment: Task 5 checks the current diff for forbidden `Loopback:` trailer revival so existing historical pressure-test text does not create a false failure.
- Blockers: none known.
