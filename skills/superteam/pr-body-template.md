# PR body template

`Finisher` renders this when opening or updating a PR so acceptance criteria, review state, and verification stay anchored to the latest pushed branch state.

If the project has its own PR template or explicit PR-body rules, satisfy that project-owned template first. Use this template as the fallback/default shape when no project-specific surface overrides it.

````markdown
## Summary
- <1-3 bullets>

## Linked issue
- `Closes #<issue-number>` when this PR is intended to complete the issue
- Otherwise: `Related to #<issue-number>` plus one short explanation of why this PR does not close it yet
- Omit this section entirely when no issue number is present

## Branch state
- Latest pushed branch state: `<branch>@<sha>`
- Review state: <no open findings | open findings summarized below>
- CI state: <passing | failing | pending>

## Acceptance criteria
### AC-<issue-number>-1
Short outcome summary for this acceptance criterion.
- [ ] Verification: `path/to/test.spec.ts > should ...` — ✅ verified

### AC-<issue-number>-2
Short outcome summary for this acceptance criterion.
- [ ] Verification: manual verification — <reason>

## Test plan
- [ ] All required ACs above verified on the latest pushed branch state
- [ ] Verification reflects the current branch state, not superseded local work

## Review follow-up
- Local reviewer state: <none | summarized findings>
- External feedback state: <no open review threads | summarized open items>
- Branch-state-aware next action: <none | respond/update/re-route>

## Known CI state

Only include this section when CI is still red and the operator has explicitly chosen to proceed.
````

## Status rules

- If a project-owned PR template exists, treat it as authoritative and use this template as fallback/default guidance rather than as an override.
- Use `Closes #<issue-number>` as the canonical closing-keyword line for issue-completing runs.
- If the issue is related but not complete, keep a non-closing issue reference and explain the omission briefly.
- If no issue number is present, omit the linked-issue section instead of inventing a placeholder.
- Use the summary line under each `### AC-<issue-number>-<n>` heading for the outcome.
- Use checkboxes only for verification steps beneath the AC heading.
- Keep the acceptance criteria and verification synchronized with the latest pushed branch state.
- Make review-state notes easy for `Finisher` to update after new pushes, reviewer findings, or CI changes.
