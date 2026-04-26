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

When work originating from a loopback is committed, the commit message MUST include the matching `Loopback:` class trailer. When the loopback is resolved (the work is complete and the workflow returns to its prior phase), the terminating commit MUST include `Loopback: resolved`.

The trailer is mandatory on every loopback-originated commit and on the resolving commit.

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

## Recovery algorithm

```text
1. git log --pretty=format:'%H%x09%(trailers:key=Loopback,valueonly)' <branch>
   from oldest to newest.
2. For any commit with multiple `Loopback:` trailers, treat `resolved` as
   winning for that commit.
3. Find the most recent commit whose Loopback trailer is `resolved`.
   Call its index R (0 if none).
4. Among commits with index > R, find the most recent commit whose Loopback
   trailer is one of {spec-level, plan-level, implementation-level}. That is
   the active loopback class.
5. If no such commit exists, no active loopback is in flight.
6. If multiple unresolved Loopback: trailers are present, the most recent one
   wins.
```

The algorithm is intentionally pure `git log` — no new tooling, no sidecar files, no parser beyond `git log`'s built-in trailer extraction.

## Compatibility

This convention extends `AGENTS.md`'s existing conventional-commits rule (no scope, required GitHub issue tag) with a defined trailer; it is NOT a new persistence file or sidecar artifact.
