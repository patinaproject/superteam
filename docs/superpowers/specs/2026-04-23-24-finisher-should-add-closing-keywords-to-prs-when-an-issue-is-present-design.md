# Design: Finisher should add closing keywords to PRs when an issue is present [#24](https://github.com/patinaproject/superteam/issues/24)

## Summary

Teach `Finisher` to include a GitHub-recognized closing-keyword line in the generated PR body whenever the current workflow has a real issue number available. Keep the change narrow by updating the PR body contract and the directly relevant `Finisher` wording so the workflow closes linked issues automatically on merge without inventing issue references when none exist.

## Goals

- Add a closing-keyword line such as `Closes #24` to generated PR bodies when the workflow has an issue number and the PR is intended to complete that issue
- Keep the line in a GitHub-recognized format for automatic issue closure on merge
- Require a brief explanation when an issue is present but the PR intentionally does not close it
- Preserve the current behavior when no issue number is present by omitting the issue-closing line entirely
- Keep the change focused on `Finisher` PR-body generation guidance instead of redesigning the broader publish workflow

## Non-Goals

- Introducing a new issue-detection mechanism beyond the issue context already available to the workflow
- Changing PR title rules or acceptance-criteria formatting
- Adding multiple closing lines or supporting arbitrary linked-work item formats outside standard GitHub issue references

## Approaches Considered

### Recommended: add an explicit linked-issue section and Finisher rule

Update the PR body template so it contains a dedicated section or line for the closing keyword, and document that `Finisher` must render it only when an issue number is available. This keeps the contract visible in the exact artifact `Finisher` already uses and gives reviewers a stable place to inspect.

### Alternative: mention closing keywords only in prose guidance

Document the behavior only in `Finisher` instructions without updating the PR template. This is weaker because the desired output shape stays implicit and is easier to omit or render inconsistently.

### Alternative: add the closing keyword to the summary section ad hoc

Allow `Finisher` to place the closing keyword somewhere in `## Summary`. This technically could work, but it hides an important automation contract inside free-form prose and makes future review harder.

## Design

### PR body shape

The generated PR body should include a dedicated linked-issue area near the top of the template so the closing keyword is easy to find and hard to forget. The rendered line should use a standard GitHub keyword format such as:

- `Closes #<issue-number>`

`Closes` should be the canonical keyword for this workflow. It is the most semantically direct match for the intended result: merging the PR closes the linked issue. The design should not vary between `Closes`, `Fixes`, and `Resolves` unless a future repository rule explicitly requires that flexibility.

The design assumes one issue number for the current workflow because that is the repository convention surfaced by the current branch and issue-driven docs. The template should use placeholders that make the issue number explicit while still allowing omission when the workflow has no issue.

### Conditional rendering rule

`Finisher` should render the closing-keyword line only when the workflow has a real issue number available and the PR is intended to complete that issue. If no issue number is present, `Finisher` must omit the line instead of inventing a placeholder, fake reference, or malformed keyword.

If an issue number is present but the PR intentionally does not close that issue, `Finisher` should still make the relationship explicit and explain the omission briefly. The PR body should include a non-closing linked-issue note plus a short explanation such as:

- `Related to #<issue-number>`
- `This PR does not close #<issue-number> because follow-up work remains.`

This rule belongs in the directly relevant `Finisher` support text as well as the PR body template so both the output artifact and the teammate contract say the same thing.

### Intent decision stays narrow

The implementation should not invent a new intent-detection system. `Finisher` should use only explicit workflow context already available at PR-render time:

- if the run is the canonical issue-backed workflow for a single issue and nothing in the current run says the work is partial, render `Closes #<issue-number>`
- if the current run explicitly says the work is partial, follow-up, or otherwise not issue-completing, render the non-closing explanation path instead
- if the workflow has no issue number, omit the issue-reference line entirely

Do not add heuristics that infer intent from commit messages, diff size, acceptance-criteria count, branch wording beyond the issue number, or speculative interpretation of unfinished work. When the workflow lacks explicit non-closing intent, prefer the existing issue-completing default for issue-backed runs rather than adding new decision machinery.

### Scope of repository changes

The repository changes should stay narrow:

- update [`skills/superteam/pr-body-template.md`](/Users/tlmader/.codex/worktrees/d113/superteam/skills/superteam/pr-body-template.md) so the template shows where the closing keyword line goes and makes omission explicit when no issue is present
- update the directly relevant `Finisher` guidance in [`skills/superteam/SKILL.md`](/Users/tlmader/.codex/worktrees/d113/superteam/skills/superteam/SKILL.md) and/or [`skills/superteam/agent-spawn-template.md`](/Users/tlmader/.codex/worktrees/d113/superteam/skills/superteam/agent-spawn-template.md) if needed so the teammate contract explicitly requires that behavior
- inspect mirrored public-facing docs before editing them; only update a mirrored doc if it already describes PR body behavior closely enough that leaving it unchanged would create a contradiction

At the time of implementation, this means checking the repository-owned PR templates and public workflow docs first, then editing only the files that would otherwise disagree with the new `Finisher` contract.

## Acceptance Criteria

- `AC-24-1`: When an issue number is available in the current workflow and the PR is intended to complete that issue, `Finisher` adds a `Closes #<issue-number>` line to the generated PR body.
- `AC-24-2`: The closing-keyword line uses a GitHub-recognized automatic-closing format.
- `AC-24-3`: When an issue number is present but the PR intentionally does not close it, `Finisher` includes a brief explanation instead of silently omitting the issue reference.
- `AC-24-4`: When no issue number is present, `Finisher` omits the issue-reference line and does not invent an issue reference.

## Testing And Verification

- inspect the PR body template to confirm it contains a dedicated closing-keyword line or section tied to an issue-number placeholder
- inspect the relevant `Finisher` contract text to confirm the behavior is conditional on both issue presence and issue-completing intent
- verify the template includes a non-closing issue-reference explanation path for issue-linked partial work
- verify the implementation rules do not introduce new heuristics for inferring intent
- inspect mirrored PR-facing docs and confirm only genuinely contradictory docs were changed
- verify there is no template text that implies a closing-keyword line must be emitted when no issue exists
