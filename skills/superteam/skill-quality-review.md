# Skill Quality Review

Use this guide when a Superteam run changes `skills/superteam/SKILL.md`, `skills/superteam/*.md`, `docs/superpowers/pressure-tests/*.md`, or another repository-owned workflow-contract surface.

The guide adapts Trail of Bits workflow-skill review and skill-improver discipline into Superteam's local workflow. It is a review method, not a runtime dependency.

## Review Sequence

1. Confirm the changed surfaces and read the governing repository rules.
2. Apply the workflow-skill review categories:
   - frontmatter and trigger description quality
   - workflow pattern and phase/gate clarity
   - progressive disclosure and reference size
   - file-reference integrity
   - tool or agent portability
   - anti-patterns, especially reference dumps, hidden state, and unbounded delegation
3. Categorize findings:
   - `critical`: blocks loading, references missing files, or makes the workflow impossible to execute
   - `major`: materially weakens triggering, gates, role ownership, verification, or handoff safety
   - `minor`: polish, style, or optional clarity improvement
4. Fix or explicitly disposition every critical and major finding.
5. Evaluate every minor finding before applying it. Apply only when it improves execution reliability, evidence quality, or operator clarity.
6. Record evidence in an issue-specific review artifact or PR acceptance-criteria section.

## Required Evidence

Each finding needs:

- severity: `critical`, `major`, or `minor`
- affected surface
- finding summary
- disposition: `fixed`, `not applicable`, `deferred with rationale`, or `accepted minor`
- verification command, pressure-test scenario, or review evidence

## Local Contract Priority

When Trail of Bits guidance conflicts with Superteam's local contracts, keep the local contract and record the decision. Local contracts include committed design and plan artifacts, visible-state resume, no hidden routing markers, Reviewer before Finisher, and latest-head Finisher shutdown.

## Completion Criteria

The adapted loop is complete when:

- no critical or major finding remains open
- every minor finding has been evaluated for usefulness
- pressure tests cover any new workflow discipline rule
- the PR body maps acceptance criteria to review evidence that actually ran
