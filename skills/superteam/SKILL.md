---
name: superteam
description: This skill should be used when the operator runs `/superteam` or asks to take a GitHub issue from design through implementation, review, and merged-ready PR using the canonical Team Lead, Brainstormer, Planner, Executor, Reviewer, and Finisher teammate roster. Triggers on phrases like "run superteam on #N", "take this issue through the teammate workflow", or "drive #N to PR".
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - TodoRead
  - TodoWrite
---

# superteam

`superteam` is an orchestration skill for running a structured issue workflow across a canonical teammate roster. It uses repository-owned artifacts in `skills/` and `docs/` so the workflow stays portable across repositories and runtimes.

## When to Use

- A GitHub issue needs design, planning, implementation, review, PR publication, and publish-state follow-through.
- An existing `superteam` issue workflow needs to resume from committed artifacts, branch state, or PR state.
- Operator feedback, review findings, CI state, or PR comments need routing to the correct teammate.
- Workflow-contract or skill changes need explicit gates, pressure-test evidence, and role-owned handoffs.

## When NOT to Use

- Simple GitHub issue or PR edits: use `using-github:using-github`.
- Local code changes that do not need the full issue-to-PR workflow: use the relevant `superpowers` implementation or debugging skill directly.
- One-off code review without teammate orchestration: use `superpowers:requesting-code-review` or `superpowers:receiving-code-review`.
- General skill architecture review: use `workflow-skill-design:designing-workflow-skills`.

## Canonical roster

Use teammate names as the primary organizing language across the workflow:

1. `Team Lead`: owns orchestration, delegation, gates, and feedback routing
2. `Brainstormer`: owns the design doc in `docs/superpowers/specs/`
3. `Planner`: owns the implementation plan in `docs/superpowers/plans/`
4. `Executor`: owns ATDD-driven implementation, code, and tests required by the approved plan
5. `Reviewer`: owns local pre-publish review findings
6. `Finisher`: owns publish-state follow-through, CI, and external feedback handling

The workflow may still reference brainstorm, plan, execute, review, and finish phases, but teammate names are the canonical contract language. See [workflow-diagrams.md](./workflow-diagrams.md) for the canonical Mermaid diagrams.

## Pre-flight

### Phase-detection and execution-mode pre-flight

At the top of every `/superteam` invocation, before any teammate delegation, `Team Lead` runs a deterministic detection sequence covering both phase detection and execution-mode capability detection. See `pre-flight.md` in this skill directory for the full algorithm.

Summary of the sequence:

- Resolve the active issue (explicit `#<n>` in prompt, then branch `<n>-<slug>`, then operator).
- Inspect committed artifacts on the branch (design doc, plan doc) at the canonical specs and plans paths.
- Inspect PR state on origin (open / merged / absent).
- Derive the detected phase per the rules below.
- Classify the operator prompt per `routing-table.md`.
- Resolve execution mode per `pre-flight.md` `## Execution-mode capability detection`, then route per `routing-table.md`.
- Probe active host deterministically (env-var families → runtime self-id; first match wins); log `superteam active host: <name> (probe=<source>)`. Out-of-set hosts halt. Probe order in [`project-deltas.md` `## Active-host probe order`](./project-deltas.md#active-host-probe-order).
- Scan `docs/superpowers/*.md` once: any filename whose role-slug does not match a shipped role emits a single `superteam delta orphan: docs/superpowers/<file> does not match any shipped role` warning. Run continues.

Phase derivation rules:

- no design doc, no plan, no PR -> `brainstorm`
- design doc present, no plan doc on branch, no PR -> `brainstorm` (Gate 1 still open per R15)
- plan doc present on branch, no PR -> `execute`
- PR open or merged -> `finish`, with `Finisher` substate derived from PR / CI / review state
- artifacts and PR state cannot be reconciled -> halt per `pre-flight.md` `## Halt conditions`

Ambiguous or contradictory state halts with `superteam halted at Team Lead: <reason>` per `## Failure handling`. Missing execution capability halts only routes that require execute-phase delegation; non-execute routes continue. Runtime capabilities are execution aids, not replacement contracts.

## Execution-mode injection

`Team Lead` probes execution capability in pre-flight and binds every execute-phase delegation to the resolved mode (R14):

- Prefer **team mode** (R17 detection); fall back to **subagent-driven** (`superpowers:subagent-driven-development`). NEVER route through `superpowers:executing-plans` on default paths.
- **Never auto-select inline.** Only an explicit `inline` / `run inline` / `execute in this session` token in the operator prompt may route through `superpowers:executing-plans`.
- Missing capability blocks only execute-phase routes; approval, review, and Finisher status work continue.

Team Lead duties: detect team-mode in pre-flight; bind directly; inject resolved mode into delegation prompt; carry suppression into nested delegations; do not prompt the developer to choose.

Operator override: explicit `inline` (or equivalent) switches mode for that delegation only. Ambiguous framing is NOT an override.

## Model selection

`Team Lead` resolves an explicit per-role model at delegation time and binds it via the host's model-override mechanism. Silent inheritance of the parent session model is forbidden except where noted.

### Per-teammate model defaults

Per-role defaults live in the shipped agent file for the active host (Claude frontmatter under `.claude/agents/*.md`; Codex top-level YAML under `agents/*.openai.yaml`). `inherit` for `Team Lead` is a literal value; every other delegation MUST resolve to an explicit host-supported token. Defaults are deliberately static.

Host token sets:

- `claude-code`: `inherit`, `opus`, `sonnet`, `haiku`
- `codex`: `inherit`, `gpt-5.5`, `gpt-5.4`, `gpt-5.3-codex`, `gpt-5.4-mini`

`gpt-5.3-codex-spark` is not a default token and is not part of either host's role-default set.

### Operator override grammar

The override grammar mirrors R14's discipline: only unambiguous tokens count. The matching rule is the same as R14 inline override — substring match on the canonical token forms only, no fuzzy interpretation.

Canonical override tokens are host-aware and exact (`model: opus`, `model: sonnet`, `model: haiku` on Claude; `model: gpt-5.5`, `model: gpt-5.4`, `model: gpt-5.3-codex`, `model: gpt-5.4-mini` on Codex, plus `use <model>` / `with <model>` aliases). Targeted form: `model: <model> for <role>`. Matching is case-insensitive; token applies to the next teammate delegation only; does NOT persist.

`gpt-5.3-codex-spark` is override-only for exact targeted `Executor` or `Finisher` delegations and never a default, host-wide enum member, or project-delta value.

The full grammar examples and the non-override phrase list live in [`project-deltas.md` `## Model-override grammar examples`](./project-deltas.md#model-override-grammar-examples).

Override scope: operator override always wins for its targeted delegation; subsequent delegations revert to the per-role default. There is no implicit "remember the last override" behavior.

### Binding mechanism

Resolution order per delegation: (1) canonical operator override token targeting this delegation; (2) per-role default from agent file frontmatter; (3) `inherit` → no model parameter passed. Binding surface for Claude Code: `model` parameter on the `Agent` tool call. Other host runtimes use the analogous dispatch parameter.

### Capability fallback

When the host lacks a model-override mechanism, `Team Lead` records `model_override_capability=unavailable` in pre-flight output, proceeds without binding a model, and surfaces a single warning per run (inherit-and-warn, not halt). Model selection is a cost/fit gate; execution mode is a correctness gate — these behave differently when the capability is unavailable.

### Loophole closure

Model selection is binding; per-role defaults are not advisory. Ambiguous framing is NOT an override. Operator silence is NOT permission to inherit. Override always wins for its targeted delegation only; no persistent override memory. The enumerated loophole-closure rules live in [`project-deltas.md` `## Model-override loophole closure`](./project-deltas.md#model-override-loophole-closure).

## Project deltas (Team Lead lookup)

Project-level behavior modifications live at `docs/superpowers/<role>.md` in the consuming project. The role-name slug matches the kebab-case agent filename. Empty or missing means "use shipped default unchanged."

### Delta file schema

A delta file is a Markdown file with up to four optional sections:

```markdown
---
agent: <role>
---

## Model
<host-aware token; see closed model enum below>

## Tools
allow:
  - <Tool>
deny:
  - <Tool>

## System prompt append
<free-form Markdown appended verbatim after the shipped system prompt>
```

### Closed model enum

`## Model` is a host-aware closed enum: `{inherit, opus, sonnet, haiku}` on `claude-code`; `{inherit, gpt-5.5, gpt-5.4, gpt-5.3-codex, gpt-5.4-mini}` on `codex`. `inherit` resolves to "use the shipped default model for this role." For non-`team-lead` roles, a delta of `inherit` is allowed but logs `superteam delta inherit-redundant: <role>`. Invalid values halt (see halt strings below). `gpt-5.3-codex-spark` is invalid in project deltas.

### Precedence

1. **Shipped default** (agent file frontmatter).
2. **Project delta** (`docs/superpowers/<role>.md` in the consuming repo).
3. **Operator-prompt R26 override** — model layer only. Tool allow/deny and system-prompt-append layers are NOT operator-prompt-overridable.

### Append-only system prompt (LC2)

System-prompt deltas are append-only by design. There is no `replace` mode. A project cannot redact shipped guardrails via a delta. The `## Model` and `## System prompt append` layers are enforced on both supported hosts; the `## Tools` allow/deny layer is enforced on Claude Code and is currently a parity target on Codex (see [`project-deltas.md` `## Host-enforcement asymmetry`](./project-deltas.md#host-enforcement-asymmetry)).

### Closed denylist (LC5)

Team Lead lints every system-prompt-append against a closed denylist of forbidden-intent tokens; matches halt dispatch. The literal token list lives in [`project-deltas.md` `## Forbidden-append denylist (LC5)`](./project-deltas.md#forbidden-append-denylist-lc5).

### `resolve_role_config` algorithm (D5)

Team Lead executes the `resolve_role_config` algorithm at delegation time to merge shipped + delta + operator-prompt config, lint LC5, compute the non-negotiable-rules SHA-256 prefix (LC4), and emit the audit line. The algorithm body lives in [`project-deltas.md` `## resolve_role_config algorithm`](./project-deltas.md#resolve_role_config-algorithm).

### Audit-log strings (N9, LC3)

Team Lead emits one of the documented audit lines for every delegation (chat-first, stderr fallback per N9). The literal format strings live in [`project-deltas.md` `## Audit-log strings`](./project-deltas.md#audit-log-strings-team-lead-emits-these-verbatim). Every applied-line carries `non-negotiable-rules-sha=<8-char-prefix>` (LC4).

### Halt strings

On any of the documented failure modes (invalid model value, agent-disagreement, missing frontmatter, denylist match, unsupported host) Team Lead emits a verbatim halt string; the literal strings live in [`project-deltas.md` `## Halt strings`](./project-deltas.md#halt-strings-team-lead-emits-these-verbatim).

### Deterministic active-host probe (D3)

Pre-flight probes the active host deterministically (env-var families first, runtime self-id last; first match wins). The literal probe order lives in [`project-deltas.md` `## Active-host probe order`](./project-deltas.md#active-host-probe-order).

### Empty / frontmatter-only / body-only cases (N6)

- Zero-byte or all-whitespace → `superteam delta empty: <role>` (no-op; logged).
- Valid frontmatter `agent: <role>` but no body sections → `superteam delta empty: <role>` (no-op; logged). Frontmatter-only is a legitimate anchor file.
- Body sections present but no frontmatter (or frontmatter missing required `agent:` field) → halt with missing-frontmatter blocker string above.

## Canonical rule discovery

Before touching governed files, read root contributor guidance (`AGENTS.md` when present) and any local docs that govern the files; treat repository guidance as authoritative over remembered shortcuts. If canonical guidance cannot be found, halt and surface the blocker instead of guessing.

## Artifact handoff authority

Handoffs depending on uncommitted artifact changes are incomplete unless the run halts explicitly with a blocker. Trust committed branch state; downstream teammates rely on inspectable commits, not uncommitted local changes.

## Operator-facing output

Superteam chat output should satisfy workflow invariants rather than render a fixed status-report template by default. Teammates should write the shortest natural response that makes the current state, requested operator action, active blocker, or next step clear.

Structured bullets and headings are allowed when they help the operator act. They are not mandatory report shells.

Separate durable workflow data from chat rendering: keep required evidence in durable artifacts and PR surfaces, not volatile agent context; surface active blockers, findings, and next steps clearly; do not dump every durable field or enumerate closed findings unless they affect the current operator decision.

## Gate 1: Brainstormer approval

Advancement from `Brainstormer` to `Planner` requires explicit approval of the design artifact. Silence, ambiguity, or partial replies are non-approval.

Before asking for approval:

1. Verify the design artifact exists at the exact reported path.
2. Return the exact artifact path under review.
3. Include a concise intent summary of what the artifact changes or decides.
4. Include the full requirement set currently under review.
5. Include `adversarial_review_findings[]` as the single approval-finding surface, including Brainstormer-originated concerns and adversarial-review findings.
6. Preserve finding provenance with `source: brainstormer | adversarial-review`.
7. Require an explicit adversarial-review result before approval can advance: `clean`, `findings dispositioned`, or `blocked`.
8. Include `reviewer_context`: `fresh subagent`, `parallel specialists`, or `same-thread fallback`.
9. Include `clean_pass_rationale` with checked dimensions when no blocker or material findings remain.
10. Halt approval when any blocker or material finding is still open.

The evidence above is required gate data, not a required chat template. The operator-facing approval request should read naturally and focus on the decision being requested. It may summarize a clean review as no approval-blocking findings remaining instead of replaying closed or dispositioned findings.

Before Gate 1 approval is presented, `Team Lead` must run or dispatch an adversarial design review against the committed design artifact. Fresh-context or parallel specialist review is preferred for workflow-critical or broad designs when the runtime supports it; same-thread review is the portable fallback. Brainstormer-originated findings alone do not satisfy this gate.

For designs that touch `skills/**/*.md` or any workflow-contract surface, the adversarial review must check the `superpowers:writing-skills` dimensions: RED/GREEN baseline obligations, rationalization resistance, red flags, token-efficiency targets, role ownership, and stage-gate bypass paths.

If adversarial review changes the design, `Brainstormer` must commit the revised artifact before approval. Material requirement, ownership, pressure-test, or gate-order changes require rerunning the affected review dimensions or recording why rerun is unnecessary.

If the approval packet is too large to present cleanly, split it into multiple approval requests or sections. Do not collapse it into a vague fallback summary.

If revisions are requested after an approval pass, re-fire approval with delta-only content: include only changed sections or decisions, only requirements changed by those deltas; keep already-approved content authoritative unless it changed.

## Routing table

Every `/superteam` invocation, after pre-flight, routes via an explicit `(detected_phase, prompt_classification)` table. See `routing-table.md` in this skill directory for the complete table, prompt-classification heuristic, resume-not-restart default, and Gate 1 durability rule.

Headline behaviors:

- Default for repeated `/superteam` invocations is **resume**, not restart (R7).
- Ambiguous prompts during an open gate or in-flight phase are classified as **feedback for the active teammate**, never as silent phase advance (R6).
- **Gate 1 is durably observable iff a plan doc has been committed on the branch.** Prior in-session "approve" without a committed plan doc is treated as not-yet-approved on subsequent invocations (R15).

## Teammate contracts

Per-role contracts ship in host-native agent files. SKILL.md owns the orchestration contracts (gates, routing, halt conditions, done-report fields, model-selection grammar) that cross all roles. Each role ships parity files at `agents/<role>.openai.yaml` for Codex; the canonical Claude paths are linked below.

- `Team Lead` — see [.claude/agents/team-lead.md](./.claude/agents/team-lead.md).
- `Brainstormer` — see [.claude/agents/brainstormer.md](./.claude/agents/brainstormer.md).
- `Planner` — see [.claude/agents/planner.md](./.claude/agents/planner.md).
- `Executor` — see [.claude/agents/executor.md](./.claude/agents/executor.md).
- `Reviewer` — see [.claude/agents/reviewer.md](./.claude/agents/reviewer.md).
- `Finisher` — see [.claude/agents/finisher.md](./.claude/agents/finisher.md).

## Missing skill warnings

When `Team Lead` delegates work, the prompt must explicitly recommend the expected `superpowers` skills for that role when relevant. If an expected skill is unavailable in the current environment, say so explicitly in the delegated prompt so both the operator and teammate can see the gap.

Do not silently omit expected skill guidance.

## Done-report contracts

Artifact-producing teammate done reports must anchor on committed handoff state rather than uncommitted workspace state.

### Brainstormer done report

- `design_doc_path`: exact path to the written design doc
- `ac_ids[]`: ordered list of active AC IDs
- `intent_summary`: concise summary of what the artifact changes or decides
- `requirements[]`: full requirement set currently under review
- `adversarial_review_status`: `clean` | `findings dispositioned` | `blocked`
- `reviewer_context`: `fresh subagent` | `parallel specialists` | `same-thread fallback`
- `adversarial_review_findings[]`: findings relevant to approval, with `source`, `severity`, `location`, `finding`, and `disposition`
- `clean_pass_rationale`: required with checked dimensions when no blocker or material findings remain
- `handoff_commit_sha`: commit containing the design artifact used for approval and planning

### Planner done report

- `plan_path`: exact path to the written implementation plan
- `workstreams[]`: short summary of planned batches or workstreams
- `blockers[]`: any blockers preventing execution, or an explicit empty result when none exist
- `handoff_commit_sha`: commit containing the approved implementation plan used for execution

### Executor done report

- `completed_task_ids[]`: explicit task IDs completed in this batch
- `completion_evidence[]`: concrete evidence per completed task
- `head_sha`: current HEAD SHA for the committed implementation and test state being handed to `Reviewer`
- `verification[]`: verification commands and outcomes

## Review and feedback routing

Feedback classifications must be explicit:

1. `implementation-level` findings route to `Executor`
2. `plan-level` findings route to `Planner`
3. `spec-level` findings route to `Brainstormer`

Requirement-bearing feedback does not route straight to implementation. It returns to `Brainstormer`, then to `Planner`, and only then back to `Executor`. This applies to PR feedback, human-test feedback, and direct operator prompts in any detected phase, including `finish`.

Implementation-detail deltas that preserve requirements, ownership, and acceptance intent may route directly to `Planner`.

Feedback routing is same-run state unless the finding is captured in visible durable state such as a committed artifact, plan update, implementation commit, or PR comment. Do not use commit trailers, sidecar files, branch labels, or other hidden markers to persist feedback routing state across sessions.

When a later run resumes with committed implementation work and no PR, and prior local pre-publish findings cannot be proven resolved from visible state, route through `Reviewer` before `Finisher` can publish. `Reviewer` reruns or reconstructs the local pre-publish review from visible artifacts and classifies any remaining findings before routing.

Review interpretation happens at the intake point: `Reviewer` classifies local pre-publish findings; `Finisher` classifies external post-publish PR feedback; `Brainstormer`, `Planner`, and `Executor` own remediation after routing, not primary review intake.

## External feedback ownership

External PR comments, review threads, bot findings, and other repository feedback remain owned by `Finisher`, even when local `Reviewer` findings already exist.

Before resolving comments tied to a prior branch state: verify current state matches the state the comment referred to; do not respond as if nothing changed; re-route requirement-bearing feedback through the spec-first path.

## Quality guards

The expanded rationalization table and red-flag catalog live in
[quality-guards.md](./quality-guards.md). Keep `SKILL.md` focused on the
orchestration contract; use the reference during review, pressure tests, and
skill-improver loops.

High-risk summary:

- Do not advance Gate 1 without verified artifact, full requirements, and
  adversarial-review evidence.
- Do not treat ambiguous prompts as approval, inline execution, model override,
  restart, or issue switch.
- Do not route requirement-bearing feedback straight to implementation.
- Do not silently inherit teammate models, skip execution-mode binding, or
  ignore active-host/model-override probes.
- Do not continue past contradictory branch, artifact, PR, dirty-worktree, or
  unsupported-host state.
- Do not replace visible workflow state with hidden trailers, sidecar files,
  branch labels, or other invisible markers.
- Do not let project deltas replace shipped guardrails, redefine done reports,
  bypass AC IDs, or grant push/PR authority.
- Do not publish without Reviewer and Finisher success on the latest head.

## Shutdown

Finisher owns shutdown; no run is complete until the shutdown contract returns success on the latest pushed head. The shutdown checklist lives in [.claude/agents/finisher.md](./.claude/agents/finisher.md). Team Lead enforces shutdown as an orchestration gate: a run that has not produced a Finisher success-only shutdown is not complete, regardless of in-session signals.

## Failure handling

Any unsatisfied gate or failed teammate contract should halt the run and report:

`superteam halted at <teammate or gate>: <reason>`

Do not silently continue past failed checks, missing artifacts, ambiguous repository state, or unresolved publish-state feedback.

## Success criteria

A successful run routes from observable state, preserves committed handoffs, publishes a PR, and either reaches latest-head shutdown readiness or halts with an explicit blocker.

## Supporting files

- [.claude/agents/](./.claude/agents/): shipped per-role Claude Code subagent files.
- [agents/](./agents/): plugin metadata (`openai.yaml`) and per-role Codex parity files (`<role>.openai.yaml`).
- `docs/superpowers/<role>.md` in the consuming repo: project override surface (see `## Project deltas (Team Lead lookup)`).
- `docs/project-overrides.md` in this repo: operator-facing authoring guide for project delta files (schema, examples, edge-case table).
- [project-deltas.md](./project-deltas.md): Team Lead supporting reference — literal denylist tokens, halt/audit-log format strings, active-host probe order, and the `resolve_role_config` algorithm body. SKILL.md names every rule; this file carries the literal bodies.
- [quality-guards.md](./quality-guards.md): expanded rationalization table and red-flag catalog used for review and pressure tests.
- [pre-flight.md](./pre-flight.md): phase-detection sequence, execution-mode capability detection, halt conditions
- [routing-table.md](./routing-table.md): phase x prompt-class routing, classification heuristic, resume vs restart, Gate 1 durability
- [workflow-diagrams.md](./workflow-diagrams.md): canonical chronological and orchestration diagrams
