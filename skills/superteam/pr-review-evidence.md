# PR review evidence workflow

Use this workflow when a Superteam orchestration must produce recommendations
grounded in recent pull requests and review evidence.

## Flow

1. Resolve repository and time window from the operator prompt. If no range is
   supplied, use a recent merged-PR window.
2. Collect recent merged PR metadata: PR number, title, author, merge date,
   merge SHA, changed-file summary, linked issues, labels, and review-decision
   summary.
3. Collect direct review evidence when available: review comments, review
   bodies, review-thread discussions, requested changes, approvals, bot review
   findings, and resolution state.
4. Assemble an evidence ledger for every collected signal.
5. Activate the fallback ladder when direct review evidence is incomplete or
   unavailable.
6. Render recommendations with per-item attribution fields.

## Evidence ledger schema

Record each signal with:

- `tier`
- `source`
- `locator`
- `summary`
- `availability_status`

## Tier taxonomy

- `direct-review`: highest-confidence source; review comments and thread text.
- `pr-metadata`: contextual support from merged PR surfaces.
- `fallback-proxy`: commit history, commit messages, local diffs, issue text,
  and repository docs only when the workflow must continue without direct
  review evidence.

## Fallback ladder and confidence rules

Fallback is mandatory and explicit when any of these occur:

- review comments are missing
- permissions are missing
- connector access is unavailable
- evidence tooling fails
- zero recent PRs match the requested window

When fallback activates:

- set `fallback_used` to `true`
- disclose the missing or failed source in `source_summary`
- downgrade confidence language to match available evidence tier

When direct review evidence is present, keep `fallback_used` as `false` unless
mixed evidence required proxy support.

## Recommendation output contract

Each recommendation item includes:

- `evidence_tier`
- `source_summary`
- `confidence`
- `fallback_used`

Example:

```yaml
recommendation: Standardize reviewer-facing migration checklists for schema PRs.
evidence_tier: direct-review
source_summary: |
  Direct review comments on PRs #418 and #423 flagged missing rollback notes;
  PR metadata from #410 shows repeat incidents. No connector fallback used.
confidence: high
fallback_used: false
```
