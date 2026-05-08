# Plan: Add a first-class PR review evidence workflow for orchestrators [#81](https://github.com/patinaproject/superteam/issues/81)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking. Tasks have stable IDs (`T1`, `T2.3`, ...) so Executor can report them in `completed_task_ids[]`.

**Goal:** Ship a reusable Superteam PR review evidence workflow that collects recent PR and review signals in a disciplined order, exposes fallback use when direct review evidence is missing, and makes every recommendation traceable by evidence tier.

**Architecture:** Keep the runtime contract small in `skills/superteam/SKILL.md`: trigger, ownership boundary, output contract, and pointer to a support reference. Put the longer collection ladder, evidence ledger, fallback rules, and examples in a new `skills/superteam/pr-review-evidence.md` support file. Extend Team Lead routing cues and quality guards around that shared contract, and prove the behavior with explicit RED-before-GREEN pressure-test evidence rather than documentation-only walkthroughs.

**Tech Stack:** Markdown workflow-contract files, Codex role YAML, repo-local pressure-test docs, `rg`, `sed`, `git`, `pnpm lint:md`, and the Superteam fallback `superpowers:writing-skills` review discipline when the primary skill-improver loop is unavailable.

---

## Planning Basis

| Input | Source | Why it matters |
|---|---|---|
| Approved design | `docs/superpowers/specs/2026-05-08-81-add-a-first-class-pr-review-evidence-workflow-for-orchestrators-design.md` @ `fa58e4d9fb912534d8f04c70d83422bf3603d2d3` | Defines AC-81-1 through AC-81-4, R1 through R7, PT-81-1 through PT-81-5, and the ownership boundary between Team Lead and Finisher. |
| Repo contract | `AGENTS.md` | Controls plan naming, commit format, markdown lint expectations, and the requirement to report missing verification evidence instead of overclaiming readiness. |
| Shipped skill surfaces | `skills/superteam/SKILL.md`, `skills/superteam/quality-guards.md`, `skills/superteam/agents/team-lead.openai.yaml`, `skills/superteam/agents/finisher.openai.yaml` | These are the actual installed workflow-contract surfaces the design expects to change or verify. |
| Reusable pressure-test surface | `docs/superpowers/pressure-tests/superteam-orchestration-contract.md` | Existing home for portable orchestration pressure tests; issue #81 should extend this instead of inventing a one-off format. |
| Quality gate | `docs/skill-improver-quality-gate.md` | Required review-evidence path for `skills/superteam/**` and adjacent workflow-contract changes. |

## Scope And Constraints

- This is a shipped skill/workflow-contract change, not a docs-only cleanup; the implementation commit must use a release-triggering type.
- Preserve the design's split of responsibility: `Team Lead` routes analysis prompts into the shared evidence contract; `Finisher` keeps ownership of live external PR feedback during normal issue-to-PR runs.
- Do not invent repo-specific recommendation heuristics, scoring, or a required GitHub integration.
- Do not equate commit history, commit messages, or local diffs with reviewer feedback. Those remain `fallback-proxy` evidence only.
- Do not silently skip RED/GREEN pressure tests. If a scenario cannot run, record the exact blocker and treat missing evidence as residual risk or a publish blocker.
- No AC-to-file:line mapping tables in this plan. AC coverage is mapped to workstreams and verification tasks only.
- Keep host-parity scope narrow: this issue needs the Team Lead routing cue on both shipped host surfaces (`skills/superteam/agents/team-lead.openai.yaml` and `skills/superteam/.claude/agents/team-lead.md`). Do not broaden the plan into unrelated Claude-host rewrites beyond that cue.

## Acceptance Criteria Mapping

| AC | Plan coverage |
|---|---|
| `AC-81-1` | WS2 adds the primary collection path and shared support reference; WS3 adds pressure-test scenarios and verification greps that prove the path is documented. |
| `AC-81-2` | WS2 defines the fallback ladder and explicit `fallback_used` disclosure; WS4 reruns no-direct-evidence and connector-failure scenarios to prove the downgrade is visible. |
| `AC-81-3` | WS2 adds the per-recommendation attribution contract (`evidence_tier`, `source_summary`, `confidence`, `fallback_used`); WS4 verifies the fields are still present in the shipped contract. |
| `AC-81-4` | WS2 creates the reusable named workflow reference; WS3 adds reuse pressure tests so future orchestrations are expected to call the shared contract instead of inventing new evidence taxonomies. |

## Files In Scope

| Path | Change | Notes |
|---|---|---|
| `docs/superpowers/baselines/2026-05-08-81-pr-review-evidence-red-phase-baseline.md` | Create, then append GREEN evidence later | Canonical RED/GREEN evidence record for PT-81-1 through PT-81-5. RED capture must land before implementation edits. |
| `skills/superteam/SKILL.md` | Modify | Add the compact workflow trigger, ownership rule, output contract, and support-file pointer. |
| `skills/superteam/pr-review-evidence.md` | Create | Holds the detailed evidence ladder, ledger schema, fallback rules, and recommendation examples. |
| `skills/superteam/agents/team-lead.openai.yaml` | Modify | Add or tighten the routing cue for evidence-grounded PR/review analysis requests. |
| `skills/superteam/.claude/agents/team-lead.md` | Modify | Add the matching narrow Team Lead routing cue on the Claude-host parity surface. |
| `skills/superteam/agents/finisher.openai.yaml` | Modify only if needed | Change only if the current wording is insufficient to preserve Finisher ownership after the new Team Lead route lands. |
| `skills/superteam/quality-guards.md` | Modify | Add rationalization closures and red flags for silent fallback, commit-history overclaiming, and source laundering. |
| `docs/superpowers/pressure-tests/superteam-orchestration-contract.md` | Modify | Add the reusable PT-81-1 through PT-81-5 scenarios and expected rule surface. |

## Workstreams

| WS | Title | Depends on | Task IDs |
|---|---|---|---|
| `WS1` | RED baseline pressure-test evidence before implementation edits | none | `T1.0`-`T1.3` |
| `WS2` | Add the reusable PR review evidence workflow contract | `WS1` | `T2.1`-`T2.6` |
| `WS3` | Extend quality guards and reusable pressure-test surfaces | `WS2` | `T3.1`-`T3.4` |
| `WS4` | GREEN reruns, review evidence, and handoff proof | `WS3` | `T4.1`-`T4.5` |

## Task List

### WS1 - RED baseline pressure-test evidence before implementation edits

The design's R7 is mandatory: capture actual pre-change failure, rationalization, or source-laundering behavior before editing the shipped contract. The RED evidence lives in one baseline artifact so GREEN reruns can append directly below the original failure record.

**Files:**

- Create: `docs/superpowers/baselines/2026-05-08-81-pr-review-evidence-red-phase-baseline.md`

#### T1.0: Confirm clean starting point

- [ ] **Step 1: Verify branch state before authoring baseline evidence.**
  Run:

  ```bash
  git rev-parse --abbrev-ref HEAD
  git status --porcelain
  ```

  Expected: the issue branch is checked out and the working tree is clean before the baseline file is added.

- [ ] **Step 2: Confirm the approved design commit is the reference point.**
  Run:

  ```bash
  git log --oneline -1 docs/superpowers/specs/2026-05-08-81-add-a-first-class-pr-review-evidence-workflow-for-orchestrators-design.md
  ```

  Expected: HEAD for the design artifact is `fa58e4d`.

#### T1.1: Record RED evidence for PT-81-1 through PT-81-5

- [ ] **Step 1: Author the baseline document skeleton.**
  Use these top-level sections in `docs/superpowers/baselines/2026-05-08-81-pr-review-evidence-red-phase-baseline.md`:
  `# RED-phase baseline: Add a first-class PR review evidence workflow for orchestrators [#81](...)`,
  `## Harness`,
  `## PT-81-1` through `## PT-81-5`,
  `## Captured rationalizations`,
  `## GREEN-phase verification`,
  and `## Residual blockers`.

- [ ] **Step 2: Run the five RED scenarios against the current contract and capture the actual result, not an imagined one.**
  Use one prompt or walkthrough per design pressure test:
  - `PT-81-1`: recent merged PRs with review comments and thread resolutions available
  - `PT-81-2`: recent merged PRs exist but review comments are absent or inaccessible
  - `PT-81-3`: GitHub evidence source fails while local git history is still available
  - `PT-81-4`: a future orchestration asks for recommendations grounded in recent PRs and reviews
  - `PT-81-5`: commit history implies one theme but direct review comments imply another

  For each scenario, record:
  - the exact prompt or walkthrough framing used
  - what the current shipped contract did
  - the failure signal or rationalization that proves the current contract is insufficient
  - whether the scenario was runnable in the current environment

- [ ] **Step 3: If a scenario cannot run, disclose the precise blocker in the RED artifact.**
  Accepted blockers include missing permissions, absent connectors, or missing underlying data. "I can infer the outcome from the design" is not an accepted substitute.

#### T1.2: Commit RED evidence before touching shipped contract files

- [ ] **Step 1: Stage only the RED baseline artifact.**
  Run:

  ```bash
  git add docs/superpowers/baselines/2026-05-08-81-pr-review-evidence-red-phase-baseline.md
  git diff --cached --name-only
  ```

  Expected: only the baseline file is staged.

- [ ] **Step 2: Commit the RED baseline.**
  Run:

  ```bash
  git commit -m "docs: #81 capture PR review evidence RED baseline"
  ```

  Expected: the RED evidence exists in history before any contract edit.

#### T1.3: Guard the sequencing

- [ ] **Step 1: Re-check the last commit before implementation starts.**
  Run:

  ```bash
  git log --oneline -1
  ```

  Expected: the most recent commit is the RED baseline commit from `T1.2`.

### WS2 - Add the reusable PR review evidence workflow contract

Keep `SKILL.md` compact and invariant-focused. Put the long-form ladder in a support file so future orchestrations can reference one named contract without bloating the main skill text.

**Files:**

- Modify: `skills/superteam/SKILL.md`
- Create: `skills/superteam/pr-review-evidence.md`
- Modify: `skills/superteam/agents/team-lead.openai.yaml`
- Modify: `skills/superteam/.claude/agents/team-lead.md`
- Modify: `skills/superteam/agents/finisher.openai.yaml` only if required by `T2.5`

#### T2.1: Add the shared trigger and ownership contract to `SKILL.md`

- [ ] **Step 1: Add a compact named workflow section in `skills/superteam/SKILL.md`.**
  The section should tell Team Lead when to route into the workflow: prompts asking for recommendations or analysis grounded in recent PRs, review comments, or review decisions.

- [ ] **Step 2: Encode the ownership boundary in the same section.**
  State explicitly that this contract is for analysis tasks; live external PR feedback, replies, and follow-through during normal issue-to-PR runs remain Finisher-owned.

- [ ] **Step 3: Add the output contract to `SKILL.md`.**
  Require every recommendation produced through this workflow to include:
  `evidence_tier`,
  `source_summary`,
  `confidence`,
  and `fallback_used`.
  Mixed evidence must name the strongest available tier and any fallback support.

#### T2.2: Create `skills/superteam/pr-review-evidence.md`

- [ ] **Step 1: Define the evidence ladder in order.**
  The support file must cover:
  1. repository and time-window resolution
  2. recent merged PR metadata collection
  3. direct review evidence collection
  4. evidence ledger assembly
  5. fallback ladder activation
  6. recommendation rendering with attribution

- [ ] **Step 2: Define the ledger schema.**
  Every collected signal should carry:
  `tier`,
  `source`,
  `locator`,
  `summary`,
  and `availability_status`.

- [ ] **Step 3: Define the tier taxonomy exactly once.**
  `direct-review` is highest confidence,
  `pr-metadata` is contextual support,
  `fallback-proxy` covers commit history, commit messages, local diffs, issue text, and repo docs only when the workflow must continue without direct review evidence.

- [ ] **Step 4: Add fallback disclosure and confidence rules.**
  Missing review comments, missing permissions, missing connectors, tool failures, and zero matching recent PRs must all force visible fallback disclosure and a downgraded confidence statement.

- [ ] **Step 5: Add one reusable recommendation example.**
  Show a recommendation payload that includes the required attribution fields and names whether fallback support was used.

#### T2.3: Teach Team Lead to route into the shared contract

- [ ] **Step 1: Update `skills/superteam/agents/team-lead.openai.yaml`.**
  Add a small routing cue that tells Team Lead to invoke the shared PR review evidence workflow when the operator asks for evidence-grounded PR/review analysis, rather than inventing a fresh collection method.

- [ ] **Step 2: Update `skills/superteam/.claude/agents/team-lead.md` with the same narrow routing cue.**
  Keep the Claude-host wording aligned with the Codex cue so both shipped Team Lead surfaces route evidence-grounded PR/review analysis through the shared workflow by name.

- [ ] **Step 3: Keep the cue narrow on both hosts.**
  Do not broaden it into ordinary issue execution, CI triage, or generic code review routing.

#### T2.4: Keep the reusable contract discoverable and compact

- [ ] **Step 1: Add a pointer from `SKILL.md` to `skills/superteam/pr-review-evidence.md`.**
  The pointer should make it obvious that future orchestrations can reference the shared workflow by name.

- [ ] **Step 2: Keep `SKILL.md` invariant-heavy, not tutorial-heavy.**
  Any long examples, evidence ladders, or scenario walkthroughs belong in the support file, not in the main skill contract.

#### T2.5: Decide whether `finisher.openai.yaml` needs a wording change

- [ ] **Step 1: Inspect the landed Team Lead wording against current Finisher ownership rules.**
  If the Team Lead cue could be read as taking live PR feedback away from Finisher, add the smallest possible clarification in `skills/superteam/agents/finisher.openai.yaml`.

- [ ] **Step 2: If no change is needed, record that explicitly in Executor's done report.**
  "No Finisher file edit needed because existing rule 6 already preserves ownership" is acceptable evidence; silent omission is not.

#### T2.6: Commit the workflow contract changes

- [ ] **Step 1: Stage the shared contract changes.**
  Run:

  ```bash
  git add skills/superteam/SKILL.md \
    skills/superteam/pr-review-evidence.md \
    skills/superteam/agents/team-lead.openai.yaml \
    skills/superteam/.claude/agents/team-lead.md \
    skills/superteam/agents/finisher.openai.yaml
  git diff --cached --name-only
  ```

  Expected: only files changed by WS2 are staged; omit `finisher.openai.yaml` if `T2.5` found no change was needed.

- [ ] **Step 2: Commit the shipped behavior change.**
  Run:

  ```bash
  git commit -m "feat: #81 add PR review evidence workflow"
  ```

  Expected: the main workflow contract ships under a release-triggering commit type.

### WS3 - Extend quality guards and reusable pressure-test surfaces

This workstream makes the new contract harder to rationalize away and gives Reviewer durable scenarios to rerun.

**Files:**

- Modify: `skills/superteam/quality-guards.md`
- Modify: `docs/superpowers/pressure-tests/superteam-orchestration-contract.md`

#### T3.1: Add rationalization closures to `quality-guards.md`

- [ ] **Step 1: Add a rationalization row for silent fallback.**
  Close the excuse that PR metadata or commit history can quietly stand in for review evidence.

- [ ] **Step 2: Add a rationalization row for source laundering.**
  Close the excuse that commit messages can be described as if reviewers said the same thing.

- [ ] **Step 3: Add a rationalization row for ownership drift.**
  Close the excuse that an analysis workflow can absorb live PR feedback handling that belongs to Finisher.

- [ ] **Step 4: Add matching red flags.**
  Add red flags for missing `fallback_used`, recommendations without tier/source/confidence labeling, and outputs that follow commit-history framing while direct review comments are available.

#### T3.2: Add reusable pressure-test scenarios

- [ ] **Step 1: Extend `docs/superpowers/pressure-tests/superteam-orchestration-contract.md` with PT-81-1 through PT-81-5.**
  Keep the scenarios portable and documentation-backed, consistent with the rest of the file's style.

- [ ] **Step 2: For each scenario, include required halt or reroute behavior and the rule surface.**
  The scenarios must make it obvious which contract clause is being exercised, especially for fallback disclosure and direct-review precedence.

#### T3.3: Add implementation-time verification greps

- [ ] **Step 1: Add a grep bundle to Executor notes or done-report evidence.**
  Use commands that prove the key contract terms are present:

  ```bash
  rg -n "direct-review|pr-metadata|fallback-proxy|fallback_used|source_summary|confidence|availability_status" \
    skills/superteam/SKILL.md \
    skills/superteam/pr-review-evidence.md \
    docs/superpowers/pressure-tests/superteam-orchestration-contract.md
  ```

- [ ] **Step 2: Add an ownership grep.**
  Run:

  ```bash
  rg -n "Finisher|live external PR feedback|analysis tasks" \
    skills/superteam/SKILL.md \
    skills/superteam/agents/team-lead.openai.yaml \
    skills/superteam/agents/finisher.openai.yaml
  ```

  Expected: the shared contract is analysis-only and Finisher ownership remains explicit.

- [ ] **Step 3: Add focused Team Lead parity verification.**
  Run:

  ```bash
  rg -n "PR review evidence workflow|evidence-grounded PR/review analysis|recent PRs|reviews" \
    skills/superteam/agents/team-lead.openai.yaml \
    skills/superteam/.claude/agents/team-lead.md
  ```

  Expected: both shipped Team Lead surfaces contain the narrow routing cue for the shared workflow.

#### T3.4: Fold follow-up edits into the shipped behavior commit if no new behavior changed after review

- [ ] **Step 1: If WS3 only refined the just-landed workflow contract, amend or add a follow-up `feat:` commit consistently.**
  Do not leave these quality-guard and pressure-test surface changes as unstaged local state.

### WS4 - GREEN reruns, review evidence, and handoff proof

GREEN evidence is mandatory. Documentation-only walkthroughs do not close R7.

**Files:**

- Modify: `docs/superpowers/baselines/2026-05-08-81-pr-review-evidence-red-phase-baseline.md`
- Review: changed workflow-contract files

#### T4.1: Append GREEN reruns for PT-81-1 through PT-81-5

- [ ] **Step 1: Re-run the exact five scenarios from WS1 against the updated contract.**
  Append the results under `## GREEN-phase verification` in the baseline file.

- [ ] **Step 2: For each scenario, record the observed GREEN behavior.**
  Confirm:
  - `PT-81-1`: direct review evidence is gathered and recommendations land as `direct-review`
  - `PT-81-2`: missing direct review evidence is disclosed and recommendations downgrade to `pr-metadata`
  - `PT-81-3`: collection failure is disclosed and any continued answer is labeled `fallback-proxy`
  - `PT-81-4`: the future orchestration reuses the shared workflow instead of inventing a new taxonomy
  - `PT-81-5`: direct review evidence outranks commit-history framing

- [ ] **Step 3: Record remaining red flags or blockers explicitly.**
  If any scenario still rationalizes, or cannot run, call that out under `## Residual blockers` instead of claiming readiness.

#### T4.2: Run verification commands

- [ ] **Step 1: Re-run the grep bundle from `T3.3`.**
  Expected: all required contract terms are present in the shipped files.

- [ ] **Step 2: Run markdown lint.**
  Run:

  ```bash
  pnpm lint:md
  ```

  Expected: `Summary: 0 error(s)`.

#### T4.3: Run the Superteam skill review gate

- [ ] **Step 1: Probe primary-mode availability per `docs/skill-improver-quality-gate.md`.**
  Run:

  ```bash
  ls "$HOME/.claude/plugins/cache/trailofbits/skill-improver"/*/skills/skill-improver/SKILL.md 2>/dev/null
  ls "$HOME/.claude/plugins/cache"/**/skills/plugin-dev/skill-reviewer 2>/dev/null
  ```

- [ ] **Step 2: If both probes succeed, run the primary skill-improver loop and record the required completion evidence.**

- [ ] **Step 3: If either probe fails, run the fallback review.**
  The fallback review must explicitly check:
  RED/GREEN baseline obligation,
  rationalization resistance,
  red flags,
  token-efficiency targets,
  role ownership,
  and stage-gate bypass paths.

  Record the reviewer, dimensions checked, and the fact that primary-mode tooling was unavailable.

#### T4.4: Prepare Reviewer and Finisher evidence requirements

- [ ] **Step 1: Reviewer handoff must include `completed_task_ids[]`, the RED/GREEN baseline path, grep results, lint result, and skill-improver evidence mode.**

- [ ] **Step 2: Finisher PR body must include:**
  - `Closes #81`
  - `### AC-81-1` through `### AC-81-4` with verification steps under each AC
  - the RED/GREEN baseline artifact path
  - the skill-improver quality gate block required by `docs/skill-improver-quality-gate.md`
  - an explicit note whenever a pressure test or review loop could not run

- [ ] **Step 3: Do not describe the work as production-ready if PT evidence or review-gate evidence is missing.**

#### T4.5: Final implementation commit and handoff

- [ ] **Step 1: Stage the GREEN baseline append and any last workflow-contract fixes.**
  Run:

  ```bash
  git add docs/superpowers/baselines/2026-05-08-81-pr-review-evidence-red-phase-baseline.md \
    skills/superteam/SKILL.md \
    skills/superteam/pr-review-evidence.md \
    skills/superteam/quality-guards.md \
    skills/superteam/agents/team-lead.openai.yaml \
    skills/superteam/agents/finisher.openai.yaml \
    docs/superpowers/pressure-tests/superteam-orchestration-contract.md
  git diff --cached --name-only
  ```

- [ ] **Step 2: Commit any post-review or GREEN-evidence follow-up not already committed.**
  Use:

  ```bash
  git commit -m "feat: #81 finalize PR review evidence workflow"
  ```

  only if there are still unstaged or uncommitted shipped changes after the earlier `feat:` commit. If WS2/WS3/WS4 work was already folded into a clean `feat:` history, do not create an empty ritual commit.

## Verification Commands

Run these during execution and cite the ones that produced final evidence:

```bash
git rev-parse --abbrev-ref HEAD
git status --porcelain
git log --oneline -1 docs/superpowers/specs/2026-05-08-81-add-a-first-class-pr-review-evidence-workflow-for-orchestrators-design.md
rg -n "direct-review|pr-metadata|fallback-proxy|fallback_used|source_summary|confidence|availability_status" \
  skills/superteam/SKILL.md \
  skills/superteam/pr-review-evidence.md \
  docs/superpowers/pressure-tests/superteam-orchestration-contract.md
rg -n "Finisher|live external PR feedback|analysis tasks" \
  skills/superteam/SKILL.md \
  skills/superteam/agents/team-lead.openai.yaml \
  skills/superteam/agents/finisher.openai.yaml
rg -n "PR review evidence workflow|evidence-grounded PR/review analysis|recent PRs|reviews" \
  skills/superteam/agents/team-lead.openai.yaml \
  skills/superteam/.claude/agents/team-lead.md
pnpm lint:md
```

## Review Evidence Requirements

- RED baseline evidence must be committed before any shipped contract edit.
- GREEN reruns must reuse the same PT-81-1 through PT-81-5 scenarios; switching to easier scenarios does not satisfy R7.
- Documentation-only walkthroughs are insufficient when the design required pressure-test or subagent evidence.
- For `skills/superteam/**` and adjacent workflow-contract changes, run `docs/skill-improver-quality-gate.md` in primary mode when available; otherwise run the documented fallback `superpowers:writing-skills` review and disclose the limitation.
- Reviewer should treat missing RED evidence, missing GREEN reruns, silent fallback, or lost ownership boundaries as approval-blocking findings.

## Sequencing And Handoff

1. `WS1` must complete and commit before `WS2` starts.
2. `WS2` and `WS3` may be split into small commits, but the shipped workflow contract must remain coherent at each handoff.
3. `WS4` must append GREEN evidence after the final contract wording lands, not before.
4. Executor hands off only after all artifact changes are committed and the done report names the completed task IDs plus verification evidence.
5. Reviewer routes requirement-changing findings back through Brainstormer/Planner only if the finding alters approved requirements; implementation-level findings loop back to Executor.
6. Finisher owns PR publication and must preserve the AC-81-1 through AC-81-4 verification plus quality-gate evidence in the PR body.

## Self-Review

- Spec coverage: AC-81-1 through AC-81-4, R1 through R7, and PT-81-1 through PT-81-5 all map to explicit workstreams or verification tasks.
- Placeholder scan: no TODO/TBD placeholders remain; every in-scope file and evidence obligation is named directly.
- Constraint check: no AC-to-file:line mapping table was introduced; Finisher ownership and RED-before-GREEN sequencing remain explicit.
