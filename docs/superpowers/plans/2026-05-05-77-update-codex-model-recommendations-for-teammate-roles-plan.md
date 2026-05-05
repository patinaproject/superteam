# Plan: Update Codex model recommendations for teammate roles [#77](https://github.com/patinaproject/superteam/issues/77)

## Planning basis

- Issue: [#77](https://github.com/patinaproject/superteam/issues/77)
- Approved design: [2026-05-05-77-update-codex-model-recommendations-for-teammate-roles-design.md](/Users/tlmader/.codex/worktrees/b1e8/superteam/docs/superpowers/specs/2026-05-05-77-update-codex-model-recommendations-for-teammate-roles-design.md)
- Approved design commit: `830b42d724fc00f8e99d5869d2ae524270f6ae62`
- Planner scope: this plan covers only the implementation surfaces named in the approved design. No code or docs outside that surface should change unless execution finds a direct contradiction that blocks an AC.

## Scope and constraints

- Primary goal: update Codex-hosted Superteam model defaults and host-aware model-selection guidance without weakening Claude-host guidance.
- Keep `Team Lead` at `inherit`.
- Update only Codex per-role defaults under `skills/superteam/agents/*.openai.yaml`.
- Treat `gpt-5.3-codex-spark` as override-only for `Executor` and `Finisher`.
- Reject Spark from shipped defaults, project deltas, and host-wide default enums.
- Preserve Claude guidance in `skills/superteam/.claude/agents/*.md` unless execution finds a concrete inconsistency that blocks AC-77-5.
- Because this issue changes `skills/**/*.md` and workflow-contract surfaces, readiness claims require `superpowers:writing-skills` review dimensions, the named pressure tests, and the `skill-improver` quality gate evidence before handoff to Finisher.

## Acceptance criteria mapping

| AC | Implementation target |
|---|---|
| `AC-77-1` | Replace Claude-era Codex defaults in `skills/superteam/agents/*.openai.yaml` and update shared contract text so Codex non-`Team Lead` delegations resolve to Codex-native defaults. |
| `AC-77-2` | Set Codex per-role YAML defaults to the issue-approved table and verify by direct file inspection plus targeted `rg`. |
| `AC-77-3` | Update host-aware override grammar and loophole language so exact Codex model tokens override one targeted delegation only, then revert to defaults. |
| `AC-77-4` | Document `gpt-5.3-codex-spark` only as an explicit transient override for trivial `Executor` or `Finisher` delegations, and explicitly reject it from defaults and project deltas. |
| `AC-77-5` | Keep Claude-facing files and guidance valid by splitting host vocabularies instead of replacing Claude tokens globally. |

## Files in scope

- Modify `skills/superteam/SKILL.md`
- Modify `skills/superteam/project-deltas.md`
- Modify `skills/superteam/agents/brainstormer.openai.yaml`
- Modify `skills/superteam/agents/planner.openai.yaml`
- Modify `skills/superteam/agents/executor.openai.yaml`
- Modify `skills/superteam/agents/reviewer.openai.yaml`
- Modify `skills/superteam/agents/finisher.openai.yaml`
- Inspect only `skills/superteam/agents/team-lead.openai.yaml`
- Inspect only `skills/superteam/.claude/agents/*.md`

## Workstreams

### WS-77-1: Codex per-role defaults

**AC linkage:** `AC-77-1`, `AC-77-2`, `AC-77-5`

#### Files to modify

- `skills/superteam/agents/brainstormer.openai.yaml`
- `skills/superteam/agents/planner.openai.yaml`
- `skills/superteam/agents/executor.openai.yaml`
- `skills/superteam/agents/reviewer.openai.yaml`
- `skills/superteam/agents/finisher.openai.yaml`

#### Task IDs

- `T77-1` Change `brainstormer.openai.yaml` from `model: opus` to `model: gpt-5.5`.
- `T77-2` Change `planner.openai.yaml` from `model: opus` to `model: gpt-5.4`.
- `T77-3` Change `executor.openai.yaml` from `model: sonnet` to `model: gpt-5.3-codex`.
- `T77-4` Change `reviewer.openai.yaml` from `model: opus` to `model: gpt-5.5`.
- `T77-5` Change `finisher.openai.yaml` from `model: sonnet` to `model: gpt-5.4-mini`.
- `T77-6` Confirm `team-lead.openai.yaml` remains `model: inherit` and does not gain stale prose that contradicts the new Codex contract.
- `T77-7` Inspect `skills/superteam/.claude/agents/*.md` to confirm Claude role metadata remains unchanged and still valid for `AC-77-5`.

#### RED verification

- `rg -n '^model: (opus|sonnet|haiku)$' skills/superteam/agents/*.openai.yaml`
- Expected before changes: matches for non-`Team Lead` Codex role files.

#### GREEN verification

- `rg -n '^model: (gpt-5\\.5|gpt-5\\.4|gpt-5\\.3-codex|gpt-5\\.4-mini|inherit)$' skills/superteam/agents/*.openai.yaml`
- `rg -n '^model: (opus|sonnet|haiku)$' skills/superteam/agents/*.openai.yaml`
- Expected after changes: first command shows the approved Codex mapping plus `inherit`; second command returns no matches.

### WS-77-2: Shared model-selection contract and override grammar

**AC linkage:** `AC-77-1`, `AC-77-3`, `AC-77-4`, `AC-77-5`

#### Files to modify

- `skills/superteam/SKILL.md`
- `skills/superteam/project-deltas.md`

#### Task IDs

- `T77-8` Update `skills/superteam/SKILL.md` `## Model selection` so per-role defaults are explicitly host-specific rather than globally limited to `opus`, `sonnet`, or `haiku`.
- `T77-9` Update `skills/superteam/SKILL.md` `### Operator override grammar` to include supported Codex model tokens and preserve the exact-token-only rule.
- `T77-10` Update `skills/superteam/SKILL.md` `## Project deltas (Team Lead lookup)` schema and closed-enum prose to describe host-aware legal values.
- `T77-11` Update rationalization, loophole, and red-flag language in `skills/superteam/SKILL.md` so it rejects silent inheritance and ambiguous downshifts for Codex the same way it currently rejects them for Claude.
- `T77-12` Update `skills/superteam/project-deltas.md` `resolve_role_config` pseudocode so `allowed_model_values` is host-aware and Spark is excluded from delta-legal values.
- `T77-13` Update `skills/superteam/project-deltas.md` grammar examples to include supported Codex exact tokens and targeted examples such as `model: gpt-5.3-codex-spark for finisher`.
- `T77-14` Update `skills/superteam/project-deltas.md` loophole-closure text so Spark remains override-only, one-delegation, and limited to `Executor` or `Finisher`.
- `T77-15` Ensure every shared reference to non-`Team Lead` explicit model binding is host-aware and no longer says all non-`Team Lead` roles must resolve only to `opus`, `sonnet`, or `haiku`.

#### Explicit Spark rule

- Spark is valid only as an exact operator override token for the next `Executor` or `Finisher` delegation.
- Spark is invalid as:
  - a shipped role default
  - a project-delta `## Model` value
  - a host-wide `allowed_model_values` member
  - a `Reviewer`, `Planner`, `Brainstormer`, or `Team Lead` override target
  - an inferred response to phrases like "go cheap", "go faster", or "tiny task"

#### RED verification

- `rg -n "every other delegation MUST resolve to .*|<one of: opus \\| sonnet \\| haiku \\| inherit>|\\{ opus, sonnet, haiku, inherit \\}|parsed\\.model not in \\{\"opus\", \"sonnet\", \"haiku\", \"inherit\"\\}" skills/superteam/SKILL.md skills/superteam/project-deltas.md`
- `rg -n "gpt-5\\.3-codex-spark" skills/superteam/SKILL.md skills/superteam/project-deltas.md`
- Expected before changes: Claude-only enum text is present; Spark is absent or insufficiently constrained.

#### GREEN verification

- `rg -n "allowed_model_values|gpt-5\\.5|gpt-5\\.4|gpt-5\\.3-codex|gpt-5\\.4-mini|gpt-5\\.3-codex-spark" skills/superteam/SKILL.md skills/superteam/project-deltas.md`
- `rg -n "override-only|Executor|Finisher|invalid model value" skills/superteam/SKILL.md skills/superteam/project-deltas.md`
- `rg -n "every other delegation MUST resolve to .*|\\{ opus, sonnet, haiku, inherit \\}" skills/superteam/SKILL.md`
- Expected after changes: host-aware token language and Spark restrictions are present; the old shared Claude-only wording is removed from Codex-applicable sections.

### WS-77-3: Review evidence and readiness gates

**AC linkage:** `AC-77-3`, `AC-77-4`, `AC-77-5`

#### Files to modify

- None required for this workstream unless execution needs to add clarifying verification notes inside the touched files.

#### Task IDs

- `T77-16` Run the design pressure tests as implementation-review walkthroughs against the final edited files, not as confidence-only claims.
- `T77-17` Invoke `superpowers:writing-skills` during review because the change touches `skills/**/*.md` and workflow-contract surfaces.
- `T77-18` Capture evidence for the required writing-skills dimensions: RED/GREEN baseline obligations, rationalization resistance, red flags, token-efficiency targets, role ownership, and stage-gate bypass paths.
- `T77-19` Run the `skill-improver` quality gate because the change touches `skills/superteam/**`, and record completion evidence before Finisher handoff.
- `T77-20` Do not claim readiness if any pressure test, writing-skills dimension, or skill-improver evidence is missing.

## Pressure tests to execute during review

- `PT-77-1` Codex default resolution
  - Prove Codex role defaults come from `agents/*.openai.yaml` and use `gpt-*` values for every non-`Team Lead` role.
- `PT-77-2` Claude preservation
  - Prove `.claude/agents/*.md` remain the Claude-owned default surface and are not silently rewritten to Codex model IDs.
- `PT-77-3` Codex override scope
  - Prove `model: gpt-5.3-codex-spark for finisher` applies only to the next targeted `Finisher` delegation and does not persist.
- `PT-77-4` Ambiguous downshift pressure
  - Prove phrases such as `go cheap` or `tiny Executor task` remain non-overrides.
- `PT-77-5` Project delta validation
  - Prove host-aware validation accepts supported Codex delta values, still accepts supported Claude delta values on Claude, and rejects Spark in deltas.
- `PT-77-5A` Spark wrong-role override rejection
  - Prove `model: gpt-5.3-codex-spark for reviewer` is rejected and does not silently map to another cheap model.
- `PT-77-6` Writing-skills review dimensions
  - Prove the review explicitly checked RED/GREEN baseline obligations, rationalization resistance, red flags, token-efficiency, role ownership, and stage-gate bypass paths.

## Validation commands

Run these exact commands during implementation and review:

```bash
sed -n '1,220p' skills/superteam/agents/brainstormer.openai.yaml
sed -n '1,220p' skills/superteam/agents/planner.openai.yaml
sed -n '1,220p' skills/superteam/agents/executor.openai.yaml
sed -n '1,220p' skills/superteam/agents/reviewer.openai.yaml
sed -n '1,220p' skills/superteam/agents/finisher.openai.yaml
sed -n '80,220p' skills/superteam/SKILL.md
sed -n '1,260p' skills/superteam/project-deltas.md
rg -n '^model: (gpt-5\.5|gpt-5\.4|gpt-5\.3-codex|gpt-5\.4-mini|inherit)$' skills/superteam/agents/*.openai.yaml
rg -n '^model: (opus|sonnet|haiku)$' skills/superteam/agents/*.openai.yaml
rg -n 'gpt-5\.3-codex-spark|allowed_model_values|override-only|invalid model value' skills/superteam/SKILL.md skills/superteam/project-deltas.md
rg -n 'skill-improver|writing-skills|RED|GREEN|rationalization|stage-gate bypass' skills/superteam/SKILL.md skills/superteam/project-deltas.md skills/superteam/agents/reviewer.openai.yaml
pnpm lint:md
git diff -- skills/superteam/SKILL.md skills/superteam/project-deltas.md skills/superteam/agents/*.openai.yaml
```

## Sequencing and handoff

1. Complete `WS-77-1` first so the Codex role metadata matches the issue-approved table.
2. Complete `WS-77-2` next so the shared contract explains and validates the new defaults correctly.
3. Finish with `WS-77-3` review evidence before any readiness claim or Finisher handoff.
4. Keep commits scoped to the implementation surface above; do not fold unrelated cleanup into this issue.

## Expected implementation output

- Codex role YAML defaults match the issue table exactly.
- Shared docs describe host-specific model vocabularies without weakening Claude guidance.
- Spark is documented as an explicit transient override for trivial `Executor` or `Finisher` delegations only.
- Project-delta validation is host-aware and rejects Spark from `allowed_model_values`.
- Reviewer evidence includes writing-skills pressure-test coverage and `skill-improver` completion evidence before readiness claims.
