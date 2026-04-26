# RED-phase baseline: superteam should auto-switch from the default branch to the issue branch during pre-flight [#44](https://github.com/patinaproject/superteam/issues/44)

This file captures verbatim today's `Team Lead` pre-flight behavior under the trigger conditions in the design doc (`docs/superpowers/specs/2026-04-26-44-superteam-should-auto-switch-from-the-default-branch-to-the-issue-branch-during-pre-flight-design.md`), before the auto-switch rule lands in `skills/superteam/pre-flight.md`. It is the GREEN-phase entry gate per AC-44-8 and the design's `## RED-phase baseline obligation`.

The harness is a dry-run pre-flight walkthrough against the committed `skills/superteam/pre-flight.md` at HEAD prior to Workstream 2. Inputs and observed outputs are recorded verbatim.

## Scenario A: clean tree

Inputs:

- Operator prompt: `/superteam #44`
- Current branch: `main` (repository default branch)
- `git status --porcelain`: empty
- No design or plan doc on either branch at the canonical paths.

Observed pre-flight execution against today's `pre-flight.md` `## Detection sequence`:

1. Resolve the active issue. Source 1 (explicit `#44` in operator prompt) fires. `active_issue=44`.
2. Inspect committed artifacts on the active branch. Active branch is still `main`. No design doc at `docs/superpowers/specs/...` for issue 44. No plan doc.
3. Inspect branch state. Current branch `main`, tracking `origin/main`, no divergence, clean tree.
4. Inspect PR state. No PR exists for `main` for issue 44.
5. Derive detected phase. No design, no plan, no PR -> `phase=brainstorm`.
6-9. Continue to prompt classification, execution-mode probe, loopback recovery, routing.

Outcome: `Team Lead` proceeds to committed-artifact inspection on the default branch and derives `phase=brainstorm` without switching branches. There is no rule in the current `## Detection sequence` that compels a branch switch when source 1 fires on the default branch. Halt conditions 1-4 do not match. The run advances into `brainstorm` routing and would dispatch `Brainstormer` to author `docs/superpowers/specs/2026-04-26-44-...-design.md` on the default branch.

## Scenario B: dirty tree

Inputs:

- Operator prompt: `/superteam #44`
- Current branch: `main`
- `git status --porcelain`: non-empty (e.g. an unstaged edit to `README.md`)
- No design or plan doc on either branch.

Observed pre-flight execution against today's `pre-flight.md`:

1. Resolve the active issue. Source 1 fires. `active_issue=44`.
2. Inspect committed artifacts on the active branch. Same as Scenario A.
3. Inspect branch state. Records uncommitted state for completeness, but per the existing rule "do NOT use uncommitted state as the phase signal". Dirty tree is not a phase signal.
4-5. Same as Scenario A. `phase=brainstorm`.

Outcome: today's pre-flight neither halts nor switches. Halt conditions 1-4 do not include "dirty working tree". The run advances into `brainstorm` routing on the default branch with a dirty tree, which downstream will either be picked up by `Brainstormer`'s edits or stomped by a later checkout.

## Captured rationalizations

The following verbatim rationalizations are produced by today's pre-flight when asked why it did not switch from `main` to `44-<kebab-title>`:

1. "The operator is on `main` on purpose; they clearly meant to start work here."
2. "No rule in `## Detection sequence` says to switch branches; pre-flight only inspects state."
3. "Skipping the switch is faster; the operator (or `Finisher`) will branch later."
4. "There's no plan doc on this branch yet, so we're already in `phase=brainstorm`; nothing breaks if we keep going."
5. "If the working tree is dirty I can just stash and continue; the rule about dirty trees is in `github-flows`, not in `superteam`."
6. "Rebase conflicts are easy; auto-aborting and retrying is fine."
7. "I can write the auto-switch logic inline so we don't depend on `github-flows`."

## Targets the new rule must close

Each captured rationalization above maps to an acceptance criterion in the design doc and to a planned `## Rationalization table` row in `skills/superteam/SKILL.md` (Workstream 3):

| Rationalization | AC | Planned rationalization-table row |
|---|---|---|
| 1. "Operator is on `main` on purpose." | AC-44-1 | "The operator is on the default branch on purpose; they clearly meant to start work here." |
| 2. "No rule says to switch." | AC-44-1 | (closed by the new `## Auto-switch to issue branch` section in `pre-flight.md`) |
| 3. "Skipping is faster; branch later." | AC-44-1 | "Skipping the auto-switch saves a step; we can branch later." |
| 4. "Already in `brainstorm`; keep going." | AC-44-1 | "The operator is on the default branch on purpose..." (phase-derivation closure) |
| 5. "Stash and continue on dirty tree." | AC-44-2 | "Dirty working tree? I can stash and continue." |
| 6. "Rebase conflicts -- abort and retry." | AC-44-5 | "Rebase conflict on the existing issue branch is fine; I'll abort and try again." |
| 7. "Inline the kebab/checkout/rebase logic." | AC-44-6 | "We can re-implement the kebab + checkout + rebase logic inside `superteam` so we don't depend on `github-flows`." |

The GREEN-phase verification (Workstream 4) re-runs Scenarios A and B and the AC-44-3 / AC-44-4 / AC-44-5 scenarios against the updated `pre-flight.md`, and appends results below.
