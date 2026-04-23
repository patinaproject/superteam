# Plan: Finisher should add closing keywords to PRs when an issue is present [#24](https://github.com/patinaproject/superteam/issues/24)

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make `Finisher` render `Closes #<issue-number>` in PR bodies for issue-completing runs, require a brief explanation for issue-linked non-closing runs, and keep the implementation narrow without adding new intent-detection heuristics.

**Architecture:** Update the canonical PR body template first so the desired output shape is explicit. Then mirror the same narrow rule in the directly relevant `Finisher` contract surfaces. Finally, inspect PR-facing mirrored docs and only update any file whose wording would otherwise contradict the new contract.

**Tech Stack:** Markdown docs, repository workflow assets, `rg`, `sed`

---

## File Structure

- `docs/superpowers/specs/2026-04-23-24-finisher-should-add-closing-keywords-to-prs-when-an-issue-is-present-design.md`
  - Approved design doc and source of acceptance criteria.
- `docs/superpowers/plans/2026-04-23-24-finisher-should-add-closing-keywords-to-prs-when-an-issue-is-present-plan.md`
  - This implementation plan.
- `skills/superteam/pr-body-template.md`
  - Canonical PR body output contract for `Finisher`.
- `skills/superteam/SKILL.md`
  - Canonical `superteam` workflow contract; update only if the `Finisher` PR-body rule needs to be explicit here.
- `skills/superteam/agent-spawn-template.md`
  - Directly relevant `Finisher` prompt surface; mirror the narrow PR-body rule if needed.
- `.github/pull_request_template.md`
  - Repository PR-facing template to inspect for contradiction before deciding whether any mirrored doc change is necessary.

### Task 1: Make the PR body template encode the new issue-closing contract

**Files:**
- Modify: `skills/superteam/pr-body-template.md`

- [ ] **Step 1: Inspect the current template and locate the top-of-body placement**

Run: `sed -n '1,220p' skills/superteam/pr-body-template.md`
Expected: identify where a linked-issue line can be added near the top without disturbing the existing acceptance-criteria structure.

- [ ] **Step 2: Add the minimal template change**

Update `skills/superteam/pr-body-template.md` so it includes explicit issue-linking guidance near the top, equivalent to:

```md
## Linked Issue
- `Closes #<issue-number>` when this PR is intended to complete the issue
- Otherwise: `Related to #<issue-number>` plus one short explanation of why the PR does not close it yet
- Omit this section entirely when no issue number is present
```

Keep the rest of the PR body structure intact.

- [ ] **Step 3: Re-read the template for clarity**

Run: `sed -n '1,240p' skills/superteam/pr-body-template.md`
Expected: the issue-linking contract is easy to find, uses `Closes` as the canonical closing keyword, and does not imply that a fake issue should be invented.

### Task 2: Mirror the same narrow rule in the directly relevant Finisher contract

**Files:**
- Modify: `skills/superteam/SKILL.md`
- Modify: `skills/superteam/agent-spawn-template.md`

- [ ] **Step 1: Inspect the current Finisher wording**

Run: `rg -n "PR body|pull request body|Closes|Related to|issue number|issue is present" skills/superteam/SKILL.md skills/superteam/agent-spawn-template.md`
Expected: confirm whether the rule is absent or too implicit in the `Finisher` contract surfaces.

- [ ] **Step 2: Add the minimal wording needed for contract parity**

Update only the directly relevant `Finisher` text so it states all of the following:

```md
When a real issue number is available and the run is issue-completing, render `Closes #<issue-number>` in the PR body.
When the issue is related but not complete, render a non-closing issue reference and a brief explanation.
When no issue number is present, omit the issue-reference line entirely.
Do not invent a new intent-detection system or infer intent from weak heuristics such as commit wording, diff size, or acceptance-criteria count.
```

Keep the broader publish-state workflow unchanged.

- [ ] **Step 3: Re-read the Finisher wording in context**

Run: `sed -n '180,260p' skills/superteam/SKILL.md && sed -n '130,220p' skills/superteam/agent-spawn-template.md`
Expected: the PR-body rule is explicit, narrow, and consistent with the approved design.

### Task 3: Inspect mirrored docs and update only if needed

**Files:**
- Inspect: `.github/pull_request_template.md`
- Modify only if contradictory: `.github/pull_request_template.md`

- [ ] **Step 1: Inspect the mirrored PR-facing doc**

Run: `sed -n '1,220p' .github/pull_request_template.md`
Expected: determine whether the repo PR template already permits the new linked-issue contract or would contradict it.

- [ ] **Step 2: Make no change unless contradiction is real**

If the mirrored template would contradict the new contract, update it minimally so it allows the same three-way behavior:

```md
- `Closes #<issue-number>` for issue-completing PRs
- `Related to #<issue-number>` plus a short explanation for non-closing issue-linked PRs
- no issue-reference line when no issue exists
```

If there is no contradiction, leave the file untouched.

- [ ] **Step 3: Verify the final changed scope stays narrow**

Run: `git diff -- skills/superteam/pr-body-template.md skills/superteam/SKILL.md skills/superteam/agent-spawn-template.md .github/pull_request_template.md`
Expected: only the PR-body contract and directly relevant mirrored wording changed, with no unrelated workflow edits.

### Task 4: Verify acceptance coverage for the final doc changes

**Files:**
- Verify: `skills/superteam/pr-body-template.md`
- Verify: `skills/superteam/SKILL.md`
- Verify: `skills/superteam/agent-spawn-template.md`
- Verify: `.github/pull_request_template.md`

- [ ] **Step 1: Confirm the canonical closing keyword path**

Run: `rg -n "Closes #<issue-number>|Closes #|Related to #<issue-number>|Related to #" skills/superteam/pr-body-template.md skills/superteam/SKILL.md skills/superteam/agent-spawn-template.md .github/pull_request_template.md`
Expected: the canonical `Closes` path is present where needed, and the non-closing path is present only where relevant.

- [ ] **Step 2: Confirm heuristic-free intent handling**

Run: `rg -n "heuristic|commit wording|diff size|acceptance-criteria count|invent" skills/superteam/SKILL.md skills/superteam/agent-spawn-template.md`
Expected: the contract explicitly avoids new intent-detection heuristics.

- [ ] **Step 3: Confirm acceptance-criteria coverage**

Run: `sed -n '1,220p' docs/superpowers/specs/2026-04-23-24-finisher-should-add-closing-keywords-to-prs-when-an-issue-is-present-design.md`
Expected: final edits cover `AC-24-1` through `AC-24-4` without broadening scope.
