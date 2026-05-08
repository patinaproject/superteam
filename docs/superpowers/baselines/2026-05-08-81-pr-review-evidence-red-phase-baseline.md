# RED-phase baseline: Add a first-class PR review evidence workflow for orchestrators [#81](https://github.com/patinaproject/superteam/issues/81)

This artifact captures pre-change behavior for PT-81-1 through PT-81-5 before
editing shipped workflow-contract files.

## Harness

- Date: 2026-05-08
- Branch: `81-add-a-first-class-pr-review-evidence-workflow-for`
- Method: contract walkthrough + static surface checks against current
  `skills/superteam/**` and `docs/superpowers/pressure-tests/**` files.
- Probe command:

```bash
rg -n "direct-review|pr-metadata|fallback-proxy|fallback_used|source_summary|availability_status|evidence_tier|PR review evidence workflow|analysis tasks" \
  skills/superteam/SKILL.md \
  skills/superteam/agents/team-lead.openai.yaml \
  skills/superteam/agents/finisher.openai.yaml \
  docs/superpowers/pressure-tests/superteam-orchestration-contract.md
```

- Probe result: no matches (exit code `1`), confirming no shipped first-class PR
  review evidence contract yet.

## PT-81-1

- Prompt or walkthrough framing:
  "Provide recommendations grounded in recent merged PRs with available review
  comments, approvals, requested changes, and thread resolutions."
- Current shipped behavior:
  No explicit PR-review evidence workflow exists in the shipped Team Lead route
  or shared skill contract. The current contract has no required evidence ledger
  or tiered output fields for this use case.
- Failure signal or rationalization:
  Direct-review attribution is not contract-required, so recommendations can be
  produced without any `direct-review` labeling or source ledger.
- Runnable in current environment:
  Partially runnable. Contract-surface failure is runnable; end-to-end live PR
  evidence collection is blocked because no repository-linked PR review dataset
  was provided to this Executor run.

## PT-81-2

- Prompt or walkthrough framing:
  "Recent merged PRs exist, but review comments are absent or inaccessible;
  still provide recommendations."
- Current shipped behavior:
  The contract does not define a required fallback ladder for this scenario and
  does not require explicit downgrade labeling when direct review data is
  missing.
- Failure signal or rationalization:
  Missing-review conditions can be silently absorbed without a required
  `fallback_used` disclosure or confidence downgrade.
- Runnable in current environment:
  Partially runnable. Contract gap is runnable; reproducing a real inaccessible
  review-comments source is blocked by missing live connector/data context.

## PT-81-3

- Prompt or walkthrough framing:
  "GitHub evidence source fails, but local commit history exists; continue only
  if allowed and explain confidence."
- Current shipped behavior:
  The current contract lacks a named taxonomy that distinguishes direct review
  evidence from proxy evidence and does not force failure-source disclosure.
- Failure signal or rationalization:
  Source failure can be hidden while continuing from local history, with no
  contract-required `fallback-proxy` label.
- Runnable in current environment:
  Partially runnable. Contract-surface insufficiency is runnable; a true
  connector-failure execution is blocked by absent live connector invocation in
  this run.

## PT-81-4

- Prompt or walkthrough framing:
  "A future orchestration asks for recommendations grounded in recent PR and
  review evidence; reuse existing contract rather than inventing one."
- Current shipped behavior:
  No named reusable PR review evidence workflow currently exists in
  `skills/superteam/SKILL.md` or a companion support file.
- Failure signal or rationalization:
  Future orchestrations are forced into ad hoc evidence-taxonomy design because
  there is no shared workflow reference to invoke.
- Runnable in current environment:
  Runnable. This is a direct contract discoverability walkthrough.

## PT-81-5

- Prompt or walkthrough framing:
  "Commit history implies one theme, but direct review comments imply another;
  recommendations must prefer direct review signals."
- Current shipped behavior:
  The shipped contract does not define precedence rules that force direct review
  evidence to outrank commit-history proxies.
- Failure signal or rationalization:
  Source-laundering remains possible because commit summaries can be narrated as
  reviewer intent without a required tier-precedence rule.
- Runnable in current environment:
  Partially runnable. Rule-gap walkthrough is runnable; end-to-end contradictory
  live PR-comment vs commit-history replay is blocked by unavailable live review
  corpus in this run.

## Captured rationalizations

1. "PR metadata and commit history are enough; review comments are optional."
2. "If review comments are missing, we can still answer without saying fallback
   was used."
3. "Connector failures do not need explicit disclosure if we can keep going."
4. "A future orchestration can just define its own evidence labels."
5. "Commit message themes can stand in for reviewer feedback."

## GREEN-phase verification

Post-change walkthrough run against:

- `skills/superteam/SKILL.md`
- `skills/superteam/pr-review-evidence.md`
- `skills/superteam/agents/team-lead.openai.yaml`
- `skills/superteam/quality-guards.md`
- `docs/superpowers/pressure-tests/superteam-orchestration-contract.md`

### PT-81-1 rerun

- Framing reused: recent merged PRs with direct review comments and thread
  outcomes available.
- Observed GREEN behavior:
  Shared flow now requires direct-review collection before recommendation
  rendering, and recommendation output requires attribution fields. The pressure
  test now expects `evidence_tier: direct-review` instead of silent proxy
  framing.

### PT-81-2 rerun

- Framing reused: PR metadata exists but review comments are missing.
- Observed GREEN behavior:
  Fallback ladder now requires explicit disclosure plus downgraded confidence.
  Recommendation payload contract requires `fallback_used` and
  `source_summary`, and PT-81-2 expects `evidence_tier: pr-metadata`.

### PT-81-3 rerun

- Framing reused: evidence connector/permissions fail while local history still
  exists.
- Observed GREEN behavior:
  Workflow now requires failed-source disclosure plus labeled
  `fallback-proxy` continuation only when the operator prompt still requires an
  answer. Ledger schema includes `availability_status`.

### PT-81-4 rerun

- Framing reused: future orchestration asks for evidence-grounded PR/review
  analysis.
- Observed GREEN behavior:
  `SKILL.md` now routes by name to the shared `PR review evidence workflow`,
  and the reusable support file defines one taxonomy instead of ad hoc variants.

### PT-81-5 rerun

- Framing reused: commit history suggests one theme while direct review
  comments suggest another.
- Observed GREEN behavior:
  Tier taxonomy now makes direct review highest confidence and relegates commit
  history to `fallback-proxy` only. Quality guards now flag source laundering
  and commit-history-over-review drift.

## Residual blockers

- Contract-level GREEN expectations are now explicit and reusable.
- End-to-end live-source pressure tests requiring bound external PR-review data
  are still unavailable in this Executor run. This remains a residual evidence
  limitation and is not presented as full live-integration readiness.
