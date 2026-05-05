# Superteam Quality Guards

This reference carries the long-form defensive checks for `SKILL.md`. `SKILL.md`
owns the invariants; this file carries the expanded rationalization table and
red-flag catalog used during review, pressure tests, and skill-improver loops.

## Rationalization Table

| Excuse | Reality |
|--------|---------|
| "The design file probably exists if Brainstormer says it does." | Gate 1 requires verifying the artifact exists at the reported path before approval. |
| "I can summarize the approval request in one short fallback blurb." | Approval packets must include artifact path, concise intent summary, and full requirement set; split oversized packets instead of collapsing them. |
| "I can replay the whole approval request after a small revision." | Re-fired approval after revisions must be delta-only. |
| "Just pick the most likely interpretation and proceed." | Ambiguous or contradictory observable state halts the run with an explicit blocker per `SKILL.md` `## Failure handling`. Resume requires explicit operator clarification of the intended issue, branch, or phase. Not even when there is a deadline. Not even when an authority claim is cited. |
| "The prompt is short/ambiguous, but the operator clearly meant approval, so just advance the gate." | Ambiguous prompts during an open gate are feedback to the active teammate per `routing-table.md`. Approval requires an explicit token (`approve`, `lgtm`, etc.). Not even when an authority claim is cited. Not even when the prior in-session approval feels binding. |
| "We've already done a lot of work on this, so restarting would waste it; keep going from a fresh top-of-workflow." | The default for repeated `/superteam` invocations is resume. Restart requires an explicit operator token (`restart`, `start over`, `new run`) per R7. "Pivot, no need to re-confirm" in the prompt is itself the disallowed shortcut. |
| "Gate 1 was approved last session; the operator just told me so, so no need to re-open it." | Gate 1 is durably observable iff a plan doc has been committed on the branch (R15). Ephemeral in-session approval is not durable. Operator memory is not the durable signal; the committed plan doc is. |
| "Removing `Loopback:` trailers means we can skip local review on a later run." | When implementation exists without a PR and prior local findings cannot be proven resolved from visible state, route through `Reviewer` before `Finisher` can publish. |
| "A direct operator requirement change during finish is not PR feedback, so Finisher can handle it." | Requirement-bearing deltas route spec-first regardless of source. PR feedback, human-test feedback, and direct operator prompts all return to `Brainstormer`, then `Planner`, then `Executor` before `Finisher` ready/shutdown can resume. |
| "No execution-mode tool is available, so every `/superteam` invocation must halt." | Missing execution capability blocks only routes that require execute-phase delegation. Approval, review, and `Finisher` status work can continue through their owning teammate. |
| "We can replace `Loopback:` trailers with another hidden marker." | Feedback routing must resume from visible artifacts, PR state, and operator prompts; do not add sidecar state, branch labels, or new commit footers. |
| "The operator said 'faster' / 'this is taking forever'; that is basically asking for inline." | Inline is auto-selected never. Only unambiguous tokens (`inline`, `run inline`, `execute in this session`) are operator overrides per R14. Ambiguous framing is not. Not even when the CTO is cited. Not even under deadline pressure. |
| "It's simpler to route through `superpowers:executing-plans` and let it ask the developer." | Execute-phase delegations bind directly to the chosen execution-mode skill per R14. Routing through `superpowers:executing-plans` on default paths surfaces a redundant prompt to the developer and is forbidden when the resolved mode is `team mode` or `subagent-driven`. |
| "The parent model is fine, just inherit." | Per-role defaults are binding (R26). Silent inheritance is forbidden for every role except `Team Lead`. The operator's silence on which model to use is not permission to inherit. It means "use the per-role default". The only path to inheritance is the host runtime lacking a model-override mechanism, in which case `Team Lead` inherits-and-warns once per run. |
| "The operator said 'go faster'; that is basically asking for Sonnet." | Ambiguous framing is not an operator override (R26, parallel to R14). Only canonical host tokens override the per-role default. "Go faster" routes to the per-role default; for Codex `Executor` that is `gpt-5.3-codex`. |
| "Brainstormer's default is `gpt-5.5`, but the operator typed `model: gpt-5.4`, so keep `gpt-5.5` because the design needs reasoning." | Operator override always wins for the delegation it targets. `Team Lead` does not second-guess the operator's explicit token. Override scope is the next delegation only. |
| "The operator is on the default branch on purpose; they clearly meant to start work here." | When the active issue resolves from the operator prompt and the current branch is the repository default branch, pre-flight must auto-switch to the per-issue branch before committed-artifact inspection. Operator intent is captured by the `#<n>` reference, not by the branch they happened to be on. Not even when the operator is the maintainer. Not even under deadline pressure. |
| "Skipping the auto-switch saves a step; we can branch later." | Skipping authors Gate 1 artifacts on the wrong base and forces `Finisher` to rewrite history or push from the default branch. The rule is not optional. |
| "Dirty working tree? I can stash and continue." | The `superteam` issue-branch procedure refuses on a dirty working tree. `superteam` halts with `superteam halted at Team Lead: dirty working tree blocks auto-switch to issue branch`. Pre-flight does not stash on the operator's behalf. |
| "Rebase conflict on the existing issue branch is fine; I'll abort and try again." | The `superteam` issue-branch procedure forbids `git rebase --abort` on the operator's behalf. `superteam` halts and surfaces the conflict. |
| "We can skip the self-contained branch procedure because another plugin probably has it." | `superteam` pre-flight is portable and owns its issue-branch procedure. Do not depend on another workflow skill being installed. |
| "`adversarial_review_findings[]` already has Brainstormer entries, so review happened." | Brainstormer-originated findings are useful but not sufficient. Gate 1 requires an explicit adversarial-review pass against the committed artifact. |
| "No findings means no review evidence is needed." | A clean adversarial-review result must include `reviewer_context`, checked dimensions, and `clean_pass_rationale`; silence is not evidence. |
| "This is a workflow-contract design, but a generic review is enough." | Designs touching `skills/**/*.md` or workflow-contract surfaces require the `superpowers:writing-skills` review dimensions: RED/GREEN baseline obligations, rationalization resistance, red flags, token-efficiency targets, role ownership, and stage-gate bypass paths. |
| "A finding changed the design, but the earlier review still applies." | Material requirement, ownership, pressure-test, or gate-order changes require rerunning affected review dimensions or recording why rerun is unnecessary. |
| "Natural prose means we can omit required Gate 1 evidence." | Natural prose changes rendering, not evidence. Required review status, reviewer context, checked dimensions, and clean-pass rationale must still exist before planning. |
| "The shipped agent file already says it; I can prune it from SKILL.md too." | Orchestration invariants (gates, done-report contracts, routing, halt conditions) stay in `SKILL.md` and are referenced from agent files. Per-role procedure moves; cross-role invariants do not. |
| "The project delta replaces the shipped system prompt because the project knows better." | Deltas are append-only. There is no `replace` mode. Stripping shipped guardrails is forbidden by design. |
| "I applied a delta but it's the same as the default, so I don't need to log it." | Delta application is always logged: `superteam delta applied`, `superteam delta empty`, or, during pre-flight, `superteam delta orphan`. Silent layering is forbidden. |
| "The host doesn't have a per-role agent file shipped yet, so I'll just guess from the Claude file." | Missing host-file is a logged fallback to portable defaults plus plugin-level prompt only. Do not silently apply a different host's agent file. |
| "The malformed delta probably means model: sonnet, so I'll use that." | Malformed delta values halt with `superteam halted at Team Lead: project delta for <role> has invalid model value <value>`. No interpretation, no guess, no default-substitution. |
| "Empty delta file means the project is unfinished, so I'll fall back to no override and not log." | Empty deltas are an intentional anchor; they are a no-op and log `superteam delta empty: <role>` so future operators see the file was inspected. |
| "A project delta append treats acceptance-criteria IDs as advisory; it's a project preference, so respect it." | Forbidden by LC5 plus the D1 non-negotiable-rules block. The denylist lint halts dispatch on any forbidden-intent token; paraphrase-bypass is countered by the agent file's first-body-section structural defense. |
| "The active host probe is ambiguous, so I'll guess Claude Code." | Forbidden. D3 specifies a deterministic probe order (`CLAUDECODE` env vars, then `CODEX_*` env vars, then runtime self-id). First match wins; result is logged. No guessing. |
| "The append redefines a done-report field, but it's clearer than SKILL.md's version." | Forbidden by LC5. Done-report contracts are `SKILL.md`-owned invariants. Lint halts. |
| "An append-only delta tells Executor it may push for our repo's special workflow." | Forbidden by LC4. Push authority is in Executor's non-negotiable rules block; denylist lint matches `may push`, `may open PR`, and `may merge`. |
| "The host has no shipped per-role file, so I'll fall back to the plugin-level prompt for every role." | Out-of-supported-set hosts halt at pre-flight; there is no silent per-delegation degradation. |

## Red Flags

- Using older stage-only language where the canonical teammate roster should be used.
- Asking for design approval before verifying the cited artifact exists.
- Approval requests that omit the artifact path, concise intent summary, or full requirement set.
- Gate 1 approval packet has `adversarial_review_findings[]` entries but no evidence that an adversarial-review pass occurred.
- Adversarial review reports `clean` without `reviewer_context`, checked dimensions, or `clean_pass_rationale`.
- `Planner` starts while a blocker or material `adversarial_review_findings[]` item remains open.
- Brainstormer-originated findings are treated as a replacement for adversarial review.
- Workflow-contract design approval proceeds without the `superpowers:writing-skills` adversarial-review dimensions.
- Oversized approval requests are collapsed into a vague summary instead of split into clean sections.
- Approval requests hide real approval-relevant findings.
- Already-approved content is replayed instead of sending delta-only approval after revisions.
- Governed files are touched without canonical-rule discovery from repository guidance.
- Delegated teammate prompts omit expected `superpowers` recommendations or fail to warn when an expected skill is unavailable.
- `Team Lead` continues past contradictory branch, artifact, or PR state without halting.
- Execution-mode capability is resolved without running the deterministic probe order in `pre-flight.md`.
- An ambiguous prompt during an open gate is classified as approval rather than feedback.
- A repeated `/superteam` invocation restarts without an explicit operator restart token or an unambiguous new-issue signal.
- A prior in-session "approve" is treated as Gate 1 approval when no plan doc has been committed on the branch.
- Issues are silently switched mid-run when the prompt names a different issue without explicit operator confirmation.
- Required `Loopback:` commit trailers or another hidden workflow-state marker are reintroduced.
- Fresh-session resume from implementation work with no PR skips `Reviewer` before `Finisher` publication when local review resolution is not visible.
- An execute-phase delegation prompt names `superpowers:executing-plans` as the entry skill when the resolved mode is `team mode` or `subagent-driven`.
- An execute-phase delegation omits the resolved execution mode and asks the developer to choose.
- Ambiguous "faster", "inline-ish", or "forever" framing is treated as an inline override.
- A non-execute route halts solely because execution-mode capability is unavailable.
- `Team Lead` proceeds to committed-artifact inspection while the current branch is the repository default branch and the active issue was resolved from an explicit `#<n>` in the prompt.
- `Team Lead` performs `git stash` or any auto-stash variant as part of the auto-switch path.
- `Team Lead` runs `git rebase --abort` after a rebase conflict on the existing issue branch.
- `pre-flight.md` depends on an external branch workflow instead of the self-contained issue-branch procedure.
- `Team Lead` silently continues on the default branch after `gh repo view` fails to resolve the default branch.
- A `superteam` run authors `docs/superpowers/specs/...` on the default branch.
- A teammate delegation omits a resolved `model` value, or omits the host's model-override parameter on the dispatch surface, when the per-role default is not `inherit`.
- "Go faster", "use the cheap model", "use the better model", or similar fuzzy framing is treated as an operator model override.
- An execute-phase delegation resolves `{model}` to the parent session model by default rather than to the per-role `Executor` default (`gpt-5.3-codex` on Codex; `sonnet` on Claude).
- A per-role procedural rule appears in `SKILL.md` after the refactor instead of in the role's agent file.
- A delta is applied silently with no `superteam delta applied`, `delta empty`, or `delta orphan` audit line.
- A project delta uses a section heading outside the documented set `{ ## Model, ## Tools, ## System prompt append }`.
- A malformed delta is interpreted instead of halting.
- Team Lead delegates without first probing and logging the active host.
- The active host is outside the supported set `{ claude-code, codex }` and Team Lead delegates anyway instead of halting at pre-flight.
- The plugin-level `agents/openai.yaml` is treated as a per-role config surface.
- An orphan `docs/superpowers/<unknown>.md` file is silently used to override an unintended role.
- A dispatch audit line is missing the `non-negotiable-rules-sha=<prefix>` field.
- A delta append textually contains a denylist token and dispatch does not halt.
