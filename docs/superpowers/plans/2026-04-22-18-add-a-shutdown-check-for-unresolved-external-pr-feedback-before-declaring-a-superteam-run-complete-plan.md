# Plan: Add a shutdown check for unresolved external PR feedback before declaring a superteam run complete [#18](https://github.com/patinaproject/superteam/issues/18)

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make `superteam` keep `Finisher` alive through post-publish follow-through and block completion until mergeability, required checks, PR metadata requirements, unresolved inline review threads, and other blocking external PR feedback are handled on the latest pushed state, or the operator is prompted explicitly when the state cannot be determined safely.

**Architecture:** Tighten the contract at the source of truth in `skills/superteam/SKILL.md`, mirror the broader post-publish `Finisher` loop in the directly relevant `Finisher` support text, then add pressure-test coverage for early-stop, status-snapshot, and operator-prompt failure modes. Keep the change documentation-focused and avoid broad workflow rewrites.

**Tech Stack:** Markdown docs, repository-local workflow assets, `pnpm sync:plugin`, `rg`, `sed`, `find`

---

## File Structure

- `docs/superpowers/specs/2026-04-22-18-add-a-shutdown-check-for-unresolved-external-pr-feedback-before-declaring-a-superteam-run-complete-design.md`
  - Approved design doc and acceptance source for this issue.
- `docs/superpowers/plans/2026-04-22-18-add-a-shutdown-check-for-unresolved-external-pr-feedback-before-declaring-a-superteam-run-complete-plan.md`
  - This implementation plan.
- `skills/superteam/SKILL.md`
  - Canonical `superteam` workflow contract; primary source of shutdown behavior.
- `skills/superteam/agent-spawn-template.md`
  - Directly relevant `Finisher` teammate prompt surface that must mirror shutdown blocking and operator-escalation language if needed.
- `docs/superpowers/pressure-tests/superteam-orchestration-contract.md`
  - Repo-local pressure tests that verify the workflow halts or reroutes for the shutdown failure mode.
- `plugins/superteam/skills/superteam/`
  - Packaged copy refreshed via `pnpm sync:plugin` after source skill changes.

### Task 1: Tighten the canonical shutdown contract

**Files:**
- Modify: `skills/superteam/SKILL.md`

- [ ] **Step 1: Inspect the current shutdown and external-feedback wording**

Run: `rg -n "Shutdown|shutdown|bot findings|review threads|operator|complete" skills/superteam/SKILL.md`
Expected: locate the existing shutdown checklist and nearby `Finisher` language that governs completion.

- [ ] **Step 2: Write the minimal contract change in the source skill**

Update `skills/superteam/SKILL.md` so it states all of the following in one consistent shutdown path:

```md
## Shutdown

Shutdown is a success-only action. Do not shut down or present the run as complete unless every required shutdown check passes on the latest pushed PR state.

Before shutdown:

1. Verify the active PR and the current branch state after the latest push.
2. Check unresolved inline review threads on the latest PR head.
3. Check for recent blocking external PR feedback on the latest pushed state.
4. Treat the following as blocking:
   - unresolved inline review threads on the latest PR head
   - unresolved reviewer or bot feedback posted after the latest push that requests a code change, verification rerun, follow-up response, or other concrete corrective action before the PR is ready
5. If blocking feedback exists, dispatch `Finisher`-owned feedback handling and re-check.
6. If the state cannot be determined safely, prompt the operator instead of guessing.
7. Only request shutdown when all required shutdown checks pass. Otherwise halt with an explicit blocker.
```

The edit should preserve the existing ownership model: `Finisher` owns external feedback, requirement-bearing feedback still routes through spec then plan then execution, and ambiguous state blocks success.

- [ ] **Step 3: Review the updated skill text in context**

Run: `sed -n '1,260p' skills/superteam/SKILL.md`
Expected: the shutdown section reads cleanly, uses `blocking external PR feedback` consistently, and does not weaken existing routing rules.

### Task 2: Mirror the shutdown rule in the directly relevant Finisher prompt surface

**Files:**
- Modify: `skills/superteam/agent-spawn-template.md`

- [ ] **Step 1: Inspect the existing Finisher role-specific prompt block**

Run: `sed -n '110,220p' skills/superteam/agent-spawn-template.md`
Expected: locate the `Finisher` block and confirm whether shutdown-specific blocking language is currently missing or too soft.

- [ ] **Step 2: Add explicit shutdown-success and operator-escalation instructions if needed**

Update the `Finisher` block so it includes wording equivalent to:

```text
Shutdown is success-only. Do not report completion or request shutdown until you have checked the active PR after the latest push for unresolved inline review threads and other blocking external PR feedback.
Treat unresolved inline review threads and unresolved post-latest-push reviewer or bot feedback requesting concrete corrective action as blocking.
If blocking feedback exists, handle it through the Finisher-owned path and re-check.
If you cannot determine whether shutdown checks pass safely, prompt the operator and report the blocker instead of claiming completion.
```

Keep the existing done-report contract intact while making the shutdown guard hard to miss.

- [ ] **Step 3: Re-read the Finisher block for consistency with the source skill**

Run: `sed -n '1,240p' skills/superteam/agent-spawn-template.md`
Expected: `Finisher` guidance matches the canonical skill contract and still preserves branch-state-aware comment handling.

### Task 3: Add pressure-test coverage for the exact failure mode and fallback

**Files:**
- Modify: `docs/superpowers/pressure-tests/superteam-orchestration-contract.md`

- [ ] **Step 1: Inspect the existing shutdown pressure test**

Run: `rg -n "Shutdown attempted|shutdown" docs/superpowers/pressure-tests/superteam-orchestration-contract.md`
Expected: find the current shutdown test that covers unresolved threads or bot findings.

- [ ] **Step 2: Expand the pressure-test wording to cover success-only shutdown and operator prompting**

Update the shutdown-related pressure tests so they explicitly cover:

```md
## Shutdown attempted with unresolved threads or blocking external PR feedback

- Starting condition: The workflow tries to shut down after publishing a PR, but unresolved inline review threads still exist on the latest PR head, or unresolved post-latest-push reviewer or bot feedback still requests concrete corrective action.
- Required halt or reroute behavior: Do not shut down or present the run as complete. Dispatch Finisher-owned follow-through, re-check, and only allow shutdown after the blocking items are cleared.
- Rule surface: The Finisher shutdown checklist should treat unresolved inline threads and blocking external PR feedback as shutdown blockers.

## Shutdown attempted when the external-feedback state cannot be determined safely

- Starting condition: The workflow cannot tell whether review threads or recent reviewer/bot findings still block the latest pushed state.
- Required halt or reroute behavior: Do not guess and do not present success. Halt with an explicit blocker and prompt the operator.
- Rule surface: The shutdown contract should require operator escalation when shutdown readiness cannot be determined safely.
```

- [ ] **Step 3: Re-read the pressure-test section in context**

Run: `sed -n '1,260p' docs/superpowers/pressure-tests/superteam-orchestration-contract.md`
Expected: the repo-local pressure tests now exercise both the original miss and the indeterminate-state fallback.

### Task 4: Refresh the packaged plugin copy and verify the repository state

**Files:**
- Modify via sync: `plugins/superteam/skills/superteam/SKILL.md`
- Modify via sync: `plugins/superteam/skills/superteam/agent-spawn-template.md`

- [ ] **Step 1: Sync the packaged plugin copy**

Run: `pnpm sync:plugin`
Expected: the plugin copy under `plugins/superteam/skills/superteam/` updates to match the source skill assets.

- [ ] **Step 2: Verify the packaged files reflect the source changes**

Run: `rg -n "success-only|blocking external PR feedback|prompt the operator" skills/superteam/SKILL.md skills/superteam/agent-spawn-template.md plugins/superteam/skills/superteam/SKILL.md plugins/superteam/skills/superteam/agent-spawn-template.md`
Expected: matching shutdown language appears in both source and packaged copies.

- [ ] **Step 3: Verify repo-local documentation coverage**

Run: `rg -n "blocking external PR feedback|prompt the operator|Do not shut down|Do not report completion" docs/superpowers/pressure-tests/superteam-orchestration-contract.md skills/superteam/SKILL.md skills/superteam/agent-spawn-template.md`
Expected: all core shutdown requirements are represented in the canonical skill, the Finisher prompt surface, and the pressure tests.

### Task 5: Final verification and commit

**Files:**
- Modify: `docs/superpowers/specs/2026-04-22-18-add-a-shutdown-check-for-unresolved-external-pr-feedback-before-declaring-a-superteam-run-complete-design.md`
- Modify: `docs/superpowers/plans/2026-04-22-18-add-a-shutdown-check-for-unresolved-external-pr-feedback-before-declaring-a-superteam-run-complete-plan.md`
- Modify: `skills/superteam/SKILL.md`
- Modify: `skills/superteam/agent-spawn-template.md`
- Modify: `docs/superpowers/pressure-tests/superteam-orchestration-contract.md`
- Modify via sync: `plugins/superteam/skills/superteam/SKILL.md`
- Modify via sync: `plugins/superteam/skills/superteam/agent-spawn-template.md`

- [ ] **Step 1: Review the exact changed files**

Run: `git diff -- docs/superpowers/specs/2026-04-22-18-add-a-shutdown-check-for-unresolved-external-pr-feedback-before-declaring-a-superteam-run-complete-design.md docs/superpowers/plans/2026-04-22-18-add-a-shutdown-check-for-unresolved-external-pr-feedback-before-declaring-a-superteam-run-complete-plan.md skills/superteam/SKILL.md skills/superteam/agent-spawn-template.md docs/superpowers/pressure-tests/superteam-orchestration-contract.md plugins/superteam/skills/superteam/SKILL.md plugins/superteam/skills/superteam/agent-spawn-template.md`
Expected: only the planned shutdown-focused changes appear.

- [ ] **Step 2: Confirm working tree state**

Run: `git status --short`
Expected: only the issue-18 spec, plan, source skill files, pressure tests, and synced plugin files are modified.

- [ ] **Step 3: Commit the completed issue work**

Run:

```bash
git add docs/superpowers/specs/2026-04-22-18-add-a-shutdown-check-for-unresolved-external-pr-feedback-before-declaring-a-superteam-run-complete-design.md \
        docs/superpowers/plans/2026-04-22-18-add-a-shutdown-check-for-unresolved-external-pr-feedback-before-declaring-a-superteam-run-complete-plan.md \
        skills/superteam/SKILL.md \
        skills/superteam/agent-spawn-template.md \
        docs/superpowers/pressure-tests/superteam-orchestration-contract.md \
        plugins/superteam/skills/superteam/SKILL.md \
        plugins/superteam/skills/superteam/agent-spawn-template.md
git commit -m "docs: #18 harden shutdown feedback checks"
```

Expected: one clean commit captures the approved spec tightening, implementation plan, source workflow changes, pressure tests, and synced packaged copy.

## Self-Review

- Spec coverage: Task 1 covers success-only shutdown and blocking checks, Task 2 mirrors the rule in the directly relevant `Finisher` prompt surface, Task 3 covers the documented failure modes, and Task 4 verifies the packaged plugin copy. AC-18-1 through AC-18-4 each map to at least one explicit task.
- Placeholder scan: No `TODO`, `TBD`, or vague “handle later” instructions remain; every task lists exact files and concrete commands.
- Type consistency: The plan consistently uses `blocking external PR feedback`, `latest pushed state`, `prompt the operator`, and `success-only shutdown` across tasks.
