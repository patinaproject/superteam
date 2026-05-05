# Design: Update Codex model recommendations for teammate roles [#77](https://github.com/patinaproject/superteam/issues/77)

## Intent summary

Update the `superteam` model-selection contract so Codex-hosted teammate delegations use Codex-native model IDs while Claude-hosted delegations keep the existing Claude guidance. The change should preserve the static per-role policy introduced by [#67](https://github.com/patinaproject/superteam/issues/67): `Team Lead` resolves a model at delegation time, explicit operator overrides are unambiguous and non-persistent, and silent inheritance is forbidden except for `Team Lead` itself or the existing inherit-and-warn capability fallback.

The design separates host vocabularies instead of replacing `opus` / `sonnet` / `haiku` globally. Claude Code continues to use Claude model aliases in `.claude/agents/*.md` and Claude override examples. Codex uses `gpt-*` model IDs in `agents/*.openai.yaml`, Codex override examples, Codex project-delta validation, and any Codex-facing recommendation text.

## Acceptance criteria

- **AC-77-1**: Given a Codex host, when `Team Lead` resolves teammate model defaults, every non-`Team Lead` role uses a Codex-native default instead of `opus`, `sonnet`, or `haiku`.
- **AC-77-2**: Given the `Brainstormer`, `Planner`, `Executor`, `Reviewer`, and `Finisher` Codex role files, when their Codex model metadata is inspected, the defaults match the agreed table in issue #77.
- **AC-77-3**: Given an operator override in a Codex run, when the override names a supported Codex model token, `Team Lead` applies it only to the targeted delegation and subsequent delegations revert to role defaults.
- **AC-77-4**: Given a trivial `Executor` or `Finisher` task, when `gpt-5.3-codex-spark` is available, the docs describe it as an optional downshift path rather than a canonical role.
- **AC-77-5**: Given existing Claude-facing guidance, when this change is complete, Claude model guidance remains valid or is clearly separated from Codex guidance instead of being silently replaced.

## Requirements

- Preserve the #67 invariants: static per-role defaults, binding model resolution at delegation time, explicit override tokens only, one-delegation override scope, no persistent override memory, and inherit-and-warn when the host lacks a model-override surface.
- Keep `Team Lead` as `inherit` on both hosts.
- Set Codex role defaults exactly:
  - `Brainstormer`: `gpt-5.5`
  - `Planner`: `gpt-5.4`
  - `Executor`: `gpt-5.3-codex`
  - `Reviewer`: `gpt-5.5`
  - `Finisher`: `gpt-5.4-mini`
- Document `gpt-5.3-codex-spark` only as an optional explicit, transient operator downshift for trivial `Executor` or `Finisher` delegations. It must not become a shipped teammate role, a default, a host-wide allowed value, a project-delta value, or an automatic inference path.
- Split model vocabularies by active host:
  - Claude Code allowed model tokens: `opus`, `sonnet`, `haiku`, plus `inherit` where the current contract allows it.
  - Codex role and project-delta model tokens: `gpt-5.5`, `gpt-5.4`, `gpt-5.3-codex`, `gpt-5.4-mini`, plus `inherit` where the current contract allows it.
- Treat `gpt-5.3-codex-spark` as a Codex override-only token, accepted only when an exact operator override targets the next `Executor` or `Finisher` delegation. It is not part of the host-wide role/default token set and must be rejected in project deltas.
- Update project-delta model validation so the legal `## Model` values are host-aware rather than Claude-only. A Codex project delta may name a supported Codex role token; a Claude project delta may still name `opus`, `sonnet`, or `haiku`; neither host may use the transient Spark downshift in a project delta.
- Update operator-override grammar examples so Codex examples name exact supported Codex model tokens. Ambiguous phrases such as "use the cheap model", "go faster", or "use the smart model" remain non-overrides on both hosts.
- Keep `.claude/agents/*.md` model metadata unchanged unless implementation finds an actual Claude-specific inconsistency. The Codex work belongs in `agents/*.openai.yaml` and cross-host orchestration docs.
- Update rationalization and red-flag language that currently hard-codes `opus`, `sonnet`, and `haiku` as the only non-inherit possibilities so it covers host-specific model tokens without weakening the binding requirement.
- Verification must include markdown inspection and targeted `rg` checks proving no Codex role YAML still uses Claude-only model aliases and no shared text still says every non-`Team Lead` role must resolve only to `opus`, `sonnet`, or `haiku`.

## Proposed architecture

### Host-aware model defaults

Keep the current "per-role defaults live in the shipped agent file" rule, but clarify that the authoritative file is host-specific:

| Host | Authoritative role files | Default vocabulary |
|---|---|---|
| Claude Code | `skills/superteam/.claude/agents/<role>.md` | `inherit`, `opus`, `sonnet`, `haiku` |
| Codex | `skills/superteam/agents/<role>.openai.yaml` | `inherit`, `gpt-5.5`, `gpt-5.4`, `gpt-5.3-codex`, `gpt-5.4-mini` |

`skills/superteam/agents/openai.yaml` remains plugin-level metadata, not a per-role config surface.

### Codex default policy

| Teammate | Codex default | Rationale |
|---|---|---|
| `Team Lead` | `inherit` | Main session owns routing, gate enforcement, and pre-flight; it should not downshift itself. |
| `Brainstormer` | `gpt-5.5` | Strongest reasoning for ambiguity, requirements, loopholes, and design pressure tests. |
| `Planner` | `gpt-5.4` | Strong structured planning without spending flagship reasoning on every decomposition step. |
| `Executor` | `gpt-5.3-codex` | Codex-specialized implementation and test work after ACs and plan constraints are explicit. |
| `Reviewer` | `gpt-5.5` | Deep adversarial review, especially for `skills/**/*.md` and workflow-contract changes. Operators may explicitly downshift for trivial code-only review. |
| `Finisher` | `gpt-5.4-mini` | Cheaper mechanical follow-through for PR ops, status sweeps, labels, and CI polling. |

### Override grammar

Keep the #67 grammar shape and make the canonical token set host-aware. Spark is the one Codex exception: it is an override-only token, not a role/default/project-delta token.

- Claude examples remain `model: opus`, `model: sonnet`, `model: haiku`, plus `use <model>` and `with <model>`.
- Codex examples add exact tokens such as `model: gpt-5.3-codex`, `use gpt-5.4-mini`, and `with gpt-5.5`.
- Targeted form remains `model: <model> for <role>`, for example `model: gpt-5.3-codex-spark for finisher`.
- `gpt-5.3-codex-spark` is valid only when the exact override token targets `Executor` or `Finisher` for the next delegation. `model: gpt-5.3-codex-spark for reviewer`, untargeted Spark overrides, role defaults, and project deltas using Spark are invalid.
- Matching remains case-insensitive on canonical token forms. No fuzzy intent inference is added.
- Override scope remains one targeted delegation only. Later delegations re-resolve from host-specific role defaults.

### Project deltas

Revise `resolve_role_config` and the surrounding prose so the closed enum is a host-specific closed enum, not an open free-form model field:

```text
allowed_model_values(host):
  claude-code -> {inherit, opus, sonnet, haiku}
  codex -> {inherit, gpt-5.5, gpt-5.4, gpt-5.3-codex, gpt-5.4-mini}
```

`gpt-5.3-codex-spark` deliberately stays outside `allowed_model_values(host)`. It is handled only by the operator-override parser after the target role is known, and only for `Executor` or `Finisher` one-delegation overrides.

The existing invalid-model halt string can stay stable:

```text
superteam halted at Team Lead: project delta for <role> has invalid model value <value>
```

If implementation needs better diagnostics, it may add explanatory prose near the enum, but it should not change the halt string unless the plan explicitly accounts for downstream tests and documentation that expect it.

### Documentation split

`SKILL.md` should describe the cross-host contract and link to `project-deltas.md` for literal grammar examples. `project-deltas.md` should carry literal host-specific token lists, override examples, non-override phrases, and the host-aware enum in pseudocode. This keeps `SKILL.md` from becoming a long model catalog while preserving an inspectable contract.

## Affected files

- `skills/superteam/SKILL.md`: update `## Model selection`, project-delta schema prose, rationalization rows, red flags, and supporting-file descriptions so they are host-aware.
- `skills/superteam/project-deltas.md`: update `resolve_role_config` pseudocode, enum prose, override grammar examples, loophole closure, and any Claude-only examples that are meant to apply to Codex too.
- `skills/superteam/agents/brainstormer.openai.yaml`: set `model: gpt-5.5`.
- `skills/superteam/agents/planner.openai.yaml`: set `model: gpt-5.4`.
- `skills/superteam/agents/executor.openai.yaml`: set `model: gpt-5.3-codex`.
- `skills/superteam/agents/reviewer.openai.yaml`: set `model: gpt-5.5`.
- `skills/superteam/agents/finisher.openai.yaml`: set `model: gpt-5.4-mini`.
- `skills/superteam/agents/team-lead.openai.yaml`: keep `model: inherit`; inspect only to confirm no stale prose contradicts the new Codex contract.
- `skills/superteam/.claude/agents/*.md`: preserve current Claude model metadata unless a direct inconsistency is found.
- Optional docs such as `docs/project-overrides.md`, if present and still describing a Claude-only `## Model` enum, should be updated to show host-aware examples.

## Non-goals

- No dynamic model scoring, automatic complexity inference, automatic downshift, budget tracking, or benchmark harness.
- No runtime model-availability probing beyond the existing model-override capability path.
- No new teammate role for `gpt-5.3-codex-spark`.
- No replacement of Claude guidance with Codex guidance.
- No changes to execution-mode routing, Gate 1 approval, branch management, PR ownership, or done-report contracts except where text references model selection.
- No broad skill refactor while making this change.

## Risks and mitigations

| Risk | Mitigation |
|---|---|
| Codex IDs accidentally replace Claude aliases globally. | Treat `.claude/agents/*.md` as Claude-owned and keep a host-specific token table in shared docs. Verify with targeted `rg` checks. |
| Host-aware enum becomes an open enum and accepts typos. | Keep the enum closed per host and preserve invalid-value halt behavior. |
| `gpt-5.3-codex-spark` becomes an inferred default, project-delta value, or wrong-role override for "small" work. | Keep Spark outside the Codex role/default/project-delta enum and accept it only as an exact, targeted, one-delegation operator override for `Executor` or `Finisher`. |
| Ambiguous budget language becomes a model override. | Preserve the #67 non-override phrase list and add Codex examples without adding fuzzy matching. |
| Planner over-edits unrelated orchestration contracts. | Scope implementation to model-selection prose, Codex role metadata, and project-delta model validation. |
| Reviewer cannot prove skill readiness from prose alone. | Require pressure tests and writing-skills review dimensions before any production-readiness claim for `skills/**/*.md` changes. |

## Pressure tests

### PT-77-1: Codex default resolution

Given active host `codex` and no operator model override, `Team Lead` resolves defaults from `agents/<role>.openai.yaml`. `Brainstormer`, `Planner`, `Executor`, `Reviewer`, and `Finisher` bind the Codex defaults from the issue table. Any non-`Team Lead` Codex delegation resolving to `opus`, `sonnet`, `haiku`, or inherited parent model is a failure, except for the documented inherit-and-warn capability fallback.

### PT-77-2: Claude preservation

Given active host `claude-code`, `Team Lead` still reads `.claude/agents/<role>.md`, accepts Claude override tokens, and binds Claude aliases. Any implementation that forces `gpt-*` tokens into Claude role metadata or claims Claude operators must use Codex model IDs fails AC-77-5.

### PT-77-3: Codex override scope

Given operator text `model: gpt-5.3-codex-spark for finisher`, the next matching `Finisher` delegation binds `gpt-5.3-codex-spark`. The following `Finisher` delegation without an override reverts to `gpt-5.4-mini`. Reusing spark because the previous task was trivial is a failure.

### PT-77-4: Ambiguous downshift pressure

Given operator text "this is just a tiny Executor task, go cheap", `Team Lead` does not infer `gpt-5.3-codex-spark`. It binds `Executor` to `gpt-5.3-codex` unless an exact supported Codex token is present. Treating "go cheap" as an override fails.

### PT-77-5: Project delta validation

Given active host `codex` and a project delta with `## Model` set to `gpt-5.4-mini`, `resolve_role_config` accepts it for supported roles. Given active host `codex` and `## Model` set to `sonnet` or `gpt-5.3-codex-spark`, it halts as an invalid model value for Codex. Given active host `claude-code` and `## Model` set to `sonnet`, it remains valid.

### PT-77-5A: Spark wrong-role override rejection

Given operator text `model: gpt-5.3-codex-spark for reviewer`, `Team Lead` rejects the override as invalid for the targeted role and does not silently apply Spark, infer another cheap model, or persist the requested value. A Reviewer delegation must use `gpt-5.5` unless an exact supported Reviewer override is present.

### PT-77-6: Writing-skills review dimensions

Before handoff to Finisher, Reviewer pressure-tests the implementation against the required `superpowers:writing-skills` dimensions: RED/GREEN baseline obligations, rationalization resistance, red flags, token-efficiency targets, role ownership, and stage-gate bypass paths. Missing this evidence blocks readiness claims for skill/workflow-contract changes.

## Adversarial review preparation

This Brainstormer pass did not run an independent adversarial review. The later review should use fresh context when available and check:

- RED/GREEN baseline: before state is Claude-only Codex metadata and a closed Claude-only enum; after state is host-aware default resolution and host-aware override validation.
- Rationalization resistance: no "parent model is fine", "go cheap means spark", or "Codex can reuse Claude aliases" shortcuts.
- Red flags: Codex YAMLs with `model: opus|sonnet|haiku`, shared docs saying all non-`Team Lead` roles must resolve only to Claude aliases, project deltas accepting Codex model strings on Claude hosts without a host check, or Spark appearing in role defaults, host-wide enums, project-delta values, or non-`Executor` / non-`Finisher` overrides.
- Token efficiency: `SKILL.md` should stay concise and push literal token catalogs into `project-deltas.md`.
- Role ownership: `Team Lead` owns model resolution; role files own shipped defaults; operator owns explicit overrides; Reviewer owns pressure-test verification.
- Stage-gate bypass paths: no model change should bypass Gate 1, execution-mode routing, Reviewer, Finisher, or committed handoff requirements.

## Brainstormer review notes

| Source | Severity | Location | Finding | Disposition |
|---|---|---|---|---|
| brainstormer | material | `skills/superteam/SKILL.md` and `skills/superteam/project-deltas.md` | Current shared contract states every non-`Team Lead` role must resolve only to `opus`, `sonnet`, or `haiku`, which conflicts with Codex-native defaults. | Addressed by requirement to make the enum host-aware rather than globally replacing Claude guidance. |
| brainstormer | material | `skills/superteam/agents/*.openai.yaml` | Current Codex role metadata uses Claude aliases for all non-`Team Lead` roles. | Addressed by AC-77-2 requirements and affected-file mapping. |
| brainstormer | minor | Spark downshift path | Without explicit wording, `gpt-5.3-codex-spark` could become an inferred default for any "small" task. | Addressed by non-goal, override grammar, and pressure tests requiring exact operator token use. |
| adversarial-review | material | `docs/superpowers/specs/2026-05-05-77-update-codex-model-recommendations-for-teammate-roles-design.md:27` | Design scoped `gpt-5.3-codex-spark` to optional explicit downshift for trivial `Executor` or `Finisher` delegations, but later included Spark in the host-wide Codex token set and project-delta enum. That admitted persistent project-delta use and wrong-role use, weakening AC-77-4. | Resolved by removing Spark from Codex role/default/project-delta `allowed_model_values(host)`, defining Spark as override-only after target role resolution, limiting it to one-delegation `Executor` or `Finisher` overrides, and adding pressure tests for project-delta and wrong-role rejection. |

## Handoff guidance

Planner should turn this design into a tightly scoped implementation plan. Executor should update only the Codex model metadata and model-selection contract surfaces necessary to satisfy the ACs. Reviewer should not accept "docs look right" as readiness evidence; because this touches `skills/**/*.md` and workflow-contract files, Reviewer must require the pressure-test walkthrough evidence named above before Finisher publication.
