# Loopback trailers

Heavy reference for the conventional-commit trailer convention that makes loopback class recoverable from `git log` across sessions. See `SKILL.md` `## Loopback trailers` for the concise summary. See `pre-flight.md` `## Loopback-class recovery` for how the recovery integrates into pre-flight.

## Trailer grammar

- `Loopback: spec-level`
- `Loopback: plan-level`
- `Loopback: implementation-level`
- `Loopback: resolved`

`Loopback:` MUST appear in the commit trailer / footer block, one trailer per
line after the message body. A prose mention such as "this resolves
Loopback: plan-level" in the commit body is not a durable loopback signal.

## When to emit

When work originating from a loopback is committed, intermediate commits MUST include the matching `Loopback:` class trailer. When the loopback is resolved (the work is complete and the workflow returns to its prior phase), the terminating commit MUST include `Loopback: resolved`.

The class trailer is mandatory on every non-resolving loopback-originated commit. The resolving commit may also include the matching class trailer as evidence of what was resolved, but `Loopback: resolved` is the required durable resolution signal.

If one commit contains both a loopback class trailer and `Loopback: resolved`,
`resolved` wins for that commit. The class trailer is evidence of what was
resolved and MUST NOT reopen the loopback.

## Worked examples

Spec-level loopback intermediate commit:

```text
docs: #39 update design after spec-level feedback

Loopback: spec-level
```

Plan-level loopback intermediate commit:

```text
docs: #39 split SKILL.md edits per plan-level loopback

Loopback: plan-level
```

Resolution commit:

```text
feat: #39 land routing-table.md after plan-level loopback

Loopback: resolved
```

Resolution commit with optional class evidence:

```text
docs: #39 close plan-level loopback for routing table

Loopback: plan-level
Loopback: resolved
```

## Recovery algorithm

```text
1. Determine the branch-only range for the active issue, normally
   `<merge-base>..<branch>` where `<merge-base>` is the merge base between
   the active branch and its default branch. Do not use a triple-dot range
   with `git log`; that is a symmetric difference and can include default-
   branch-only commits. Inspect the scoped range from oldest to newest.
2. Ignore commits whose conventional-commit subject does not reference the
   active issue tag (for example `#39`). This prevents stale trailers from
   other issues or inherited history from becoming the active loopback class.
3. Run `git log --pretty=format:'%H%x09%s%x09%(trailers:key=Loopback,valueonly)' <range>`
   over the scoped range.
4. For any commit with multiple `Loopback:` trailers, treat `resolved` as
   winning for that commit.
5. Find the most recent commit whose Loopback trailer is `resolved`.
   Call its index R (0 if none).
6. Among commits with index > R, find the most recent commit whose Loopback
   trailer is one of {spec-level, plan-level, implementation-level}. That is
   the active loopback class.
7. If no such commit exists, no active loopback is in flight.
8. If multiple unresolved Loopback: trailers are present, the most recent one
   wins.
```

The algorithm is intentionally pure `git log` — no new tooling, no sidecar files, no parser beyond `git log`'s built-in trailer extraction.

## Compatibility

This convention extends `AGENTS.md`'s existing conventional-commits rule (no scope, required GitHub issue tag) with a defined trailer; it is NOT a new persistence file or sidecar artifact.
