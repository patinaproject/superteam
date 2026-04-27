# Design: Align superteam to the bootstrap baseline [#47](https://github.com/patinaproject/superteam/issues/47)

## Intent

Realign this repository with the current Patina Project bootstrap baseline without
rewriting surfaces that already match the local `superteam` contract. The work
should make future issue filing, release automation, and baseline audits more
predictable.

## Requirements

- Preserve repository-specific `superteam` wording where it is an intentional
  specialization of the generic bootstrap templates.
- Add the missing label source-of-truth document required by the current
  bootstrap audit flow.
- Bring release-reserved label metadata into the documented baseline shape
  without applying or removing those labels from issues or pull requests.
- Keep GitHub repository settings checks explicit and evidence-backed.
- Validate all Markdown and plugin-version checks before publish.
- Use acceptance criteria IDs in the `AC-47-<n>` format.

## Baseline Findings

The repository already has the main agent-plugin baseline shape:

- dual plugin manifests in `.claude-plugin/` and `.codex-plugin/`
- Claude, Cursor, Windsurf, and Copilot agent surfaces
- PNPM, Husky, commitlint, markdownlint, and plugin-version scripts
- release-please configuration and Patina release workflow
- `docs/superpowers/specs/` and `docs/superpowers/plans/`

Verified drift:

- `.github/LABELS.md` is missing, so agents must infer label meaning from
  remote labels alone.
- Remote `autorelease: pending` and `autorelease: tagged` labels exist with
  color `ededed`, but their descriptions are empty.
- `RELEASING.md` and `.github/workflows/release.yml` have older wording about
  manual dispatch compared with the latest observed bootstrap guidance.
- `README.md` still shows a Codex CLI install command pinned to
  `patinaproject/superteam@v0.1.0`, even though the repo is at `1.0.0`.
- `.github/pull_request_template.md` references `docs/ac-traceability.md`, but
  that file is not present and `AGENTS.md` now defines AC ID format directly.

Verified non-drift:

- Default workflow permissions are `write`.
- Tag rulesets returned no required-signature blockers.
- Merge settings match the expected squash-only bootstrap settings.
- Core issue templates, lint workflows, actionlint config, markdownlint config,
  and plugin manifest versions are aligned.

## Design

### Repository files

Add `.github/LABELS.md` as the durable, reviewable label policy. The file should
include an alphabetized `## Labels` table for the standard labels and a
tool-managed release subsection for `autorelease: pending` and
`autorelease: tagged`. It should point readers back to `AGENTS.md` and
`gh label list --json name,description` for the runtime inventory.

Refresh release documentation only where it still describes a manual-dispatch
release path. The repo should describe release-please as running on pushes to
`main`, opening or updating the release PR after regular merges, and cutting the
release after the release PR merge.

Refresh install and contribution guidance where it points to stale or missing
baseline references. The README should not pin new Codex CLI installs to an old
plugin version, and the PR template should point AC authors at the existing
`AGENTS.md` rule rather than a missing traceability document.

Avoid broad template replacement. Existing repo-specific examples and
`superteam` wording should remain unless they conflict with the baseline checks.

### Remote metadata

Update the two release-reserved GitHub labels so both have non-empty
descriptions documenting that Release Please owns them. Do not apply or remove
the labels from issues or PRs.

Do not change repository merge settings because the checked values already
match the baseline. Release immutability is UI-only and should be reported as a
manual follow-up if it cannot be verified through the available APIs.

### Verification

Run the bootstrap-relevant local checks after file changes:

- `pnpm lint:md`
- `node scripts/check-plugin-versions.mjs`
- commitlint positive and negative samples

Also re-check the remote release labels after editing them. For workflow or
release-doc changes, inspect the changed files directly and rely on existing
workflow lint CI after publication.

## Acceptance Criteria

- **AC-47-1**: Given contributors or agents need to choose labels, when they
  inspect the repository, then `.github/LABELS.md` documents the canonical
  labels and release-managed labels in the bootstrap format.
- **AC-47-2**: Given Release Please owns `autorelease: pending` and
  `autorelease: tagged`, when remote label metadata is queried, then both labels
  have non-empty descriptions that explain the reservation.
- **AC-47-3**: Given the release workflow runs from pushes to `main`, when a
  contributor reads `RELEASING.md` or `.github/workflows/release.yml`, then the
  documented flow no longer depends on a manual dispatch path.
- **AC-47-4**: Given the repo is being realigned rather than re-scaffolded, when
  contributors read install or PR guidance, then README and PR-template
  references do not point them to stale plugin versions or missing docs.
- **AC-47-5**: Given the repo is being realigned rather than re-scaffolded, when
  implementation completes, then already-aligned plugin, linting, workflow, and
  agent surfaces remain intentionally preserved.
- **AC-47-6**: Given the realignment touches Markdown and plugin metadata
  guidance, when verification runs, then Markdown linting, plugin-version checks,
  and commitlint sample checks pass.

## Out of Scope

- Replacing every repository file with bootstrap templates.
- Changing issue or pull-request labels on existing items.
- Changing merge settings that already match the baseline.
- Proving release immutability through an API that does not expose it.
