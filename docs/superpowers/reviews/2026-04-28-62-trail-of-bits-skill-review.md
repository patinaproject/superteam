# Review: Trail of Bits skill loop hardening [#62](https://github.com/patinaproject/superteam/issues/62)

## Purpose

Record the adapted Trail of Bits workflow-skill review and skill-improver evidence for Superteam issue #62.

## RED Baseline

| Check | Command | Expected RED result | Observed |
| --- | --- | --- | --- |
| Missing local review guide | `test ! -f skills/superteam/skill-quality-review.md` | Exit code 0 | RED confirmed before implementation: exit code 0, no stdout |
| Missing Trail of Bits loop references | `rg -n "Trail of Bits\|workflow-skill-reviewer\|skill-improver\|skill-quality-review" skills/superteam docs/superpowers/pressure-tests/superteam-orchestration-contract.md` | Exit code 1, no output | RED confirmed before implementation: exit code 1, no stdout |
| Missing pressure tests | `rg -n "minor findings\|severity.*disposition\|skill-quality review\|Trail of Bits" docs/superpowers/pressure-tests/superteam-orchestration-contract.md` | Exit code 1, no output | RED confirmed before implementation: exit code 1, no stdout |

## Adapted Review Sources

- Trail of Bits `workflow-skill-reviewer`: structural analysis, workflow pattern analysis, content quality, tool assignment, and anti-pattern scan.
- Trail of Bits `skill-improver`: critical and major findings must be fixed or explicitly dispositioned; minor findings are evaluated before applying.
- Superteam local contract: review evidence feeds Brainstormer, Planner, Executor, Reviewer, and Finisher; it does not replace them.

## Findings

| ID | Severity | Surface | Finding | Disposition | Verification |
| --- | --- | --- | --- | --- | --- |
| F-62-1 | major | `skills/superteam` | No repo-owned adaptation guide currently tells Superteam reviewers how to run or record the Trail of Bits workflow-skill review loop. | fixed: added `skills/superteam/skill-quality-review.md`. | `rg -n "skill-quality-review\|Trail of Bits" skills/superteam` |
| F-62-2 | major | `skills/superteam/SKILL.md` and `skills/superteam/agent-spawn-template.md` | Skill/workflow-contract changes do not yet require severity, disposition, and minor-finding evaluation from the adapted loop. | fixed: linked the review guide and updated Executor and Reviewer role guidance. | `rg -n "severity\|disposition\|minor findings" skills/superteam/SKILL.md skills/superteam/agent-spawn-template.md` |
| F-62-3 | major | `docs/superpowers/pressure-tests/superteam-orchestration-contract.md` | Existing pressure tests do not fail the shortcuts introduced by this issue. | fixed: added pressure tests for skipped loop, vague findings, blind minor fixes, and local-contract conflicts. | `rg -n "Trail of Bits\|minor findings\|local contract" docs/superpowers/pressure-tests/superteam-orchestration-contract.md` |

## Minor Finding Evaluation

No minor findings are accepted by default. During execution, add one row per minor finding discovered by the adapted review loop.

| ID | Surface | Finding | Evaluation | Decision |
| --- | --- | --- | --- | --- |

## Conflict Disposition

Trail of Bits guidance is advisory when it conflicts with Superteam's local contracts. Superteam preserves committed design and plan artifacts, visible-state resume, Reviewer before Finisher, latest-head Finisher shutdown, and the prohibition on `Loopback:` trailers or hidden workflow-state markers.

## Completion Marker

The adapted skill-improver loop is complete only when no critical or major finding remains open, and all minor findings have been evaluated for usefulness.
