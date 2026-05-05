# Project deltas (Team Lead supporting reference)

This file is referenced from `SKILL.md` `## Project deltas (Team Lead lookup)` and
carries the literal bodies that section names. `SKILL.md` remains the orchestration
spec; this file is normative for the literal token list, halt strings, audit-log
format strings, and the `resolve_role_config` pseudocode body.

---

## resolve_role_config algorithm

Team Lead runs this once per delegation:

```python
def resolve_role_config(role, host, host_tool_capabilities):
    shipped = load_shipped_agent_file(role, host)      # D1, host-aware (D3)
    rules_sha = sha256_8(shipped.non_negotiable_rules_block)
    delta_path = repo_root / "docs/superpowers" / f"{role}.md"
    if not delta_path.exists():
        return shipped                                 # missing -> shipped unchanged
    raw = read_text(delta_path)
    if raw.strip() == "":                              # zero-byte / whitespace
        log_audit(f"superteam delta empty: {role}")
        return shipped
    parsed = parse_delta_file(raw)
    if not parsed.has_frontmatter and parsed.has_body_sections:
        halt(f"superteam halted at Team Lead: project delta for "
             f"{role} is missing required frontmatter agent field")
    if parsed.has_frontmatter and not parsed.has_body_sections:
        log_audit(f"superteam delta empty: {role}")
        return shipped
    if parsed.frontmatter_agent != role:
        halt("superteam halted at Team Lead: project delta for "
             f"{role} declares agent {parsed.frontmatter_agent}")
    # Forbidden-append lint (LC5) — runs before any merge.
    matched = match_invariant_denylist(parsed.append)
    if matched:
        halt(f"superteam halted at Team Lead: project delta for "
             f"{role} attempts to weaken non-negotiable rules (matched: {matched})")
    config = shipped.copy()
    if parsed.model is not None:
        if parsed.model not in {"opus", "sonnet", "haiku", "inherit"}:
            halt("superteam halted at Team Lead: project delta for "
                 f"{role} has invalid model value {parsed.model}")
        if parsed.model == "inherit" and shipped.model != "inherit":
            log_audit(f"superteam delta inherit-redundant: {role}")
            # resolves to shipped.model; no override applied
        else:
            config.model = parsed.model
    # Tool merge with host-capability filter (N4).
    proposed_allow = parsed.tools_allow
    unavailable = proposed_allow - host_tool_capabilities
    for tool in unavailable:
        log_audit(f"superteam delta tool unavailable: {role} {tool}@{host}")
    proposed_allow = proposed_allow & host_tool_capabilities
    config.tools = (config.tools | proposed_allow) - parsed.tools_deny
    if parsed.append:
        config.system_prompt = config.system_prompt + "\n\n" + parsed.append
    log_audit(
        f"superteam delta applied: {role} ({applied_fields(parsed)}); "
        f"non-negotiable-rules-sha={rules_sha}"
    )
    # Operator R26 model override (model layer only) applied AFTER this,
    # before binding the dispatch parameter.
    return config
```

---

## Forbidden-append denylist (LC5)

The closed token list Team Lead matches against every `## System prompt append` block
before merging. A match halts dispatch:

```text
["AC IDs are advisory", "AC-<issue>- is advisory", "may push", "may open PR",
 "may merge", "skip writing-skills", "redefine done-report", "override halt"]
```

---

## Halt strings (Team Lead emits these verbatim)

| Condition | Verbatim halt string |
|---|---|
| Invalid model value in delta | `superteam halted at Team Lead: project delta for <role> has invalid model value <value>` |
| Agent mismatch (frontmatter ≠ filename) | `superteam halted at Team Lead: project delta for <role> declares agent <other>` |
| Missing frontmatter `agent:` field | `superteam halted at Team Lead: project delta for <role> is missing required frontmatter agent field` |
| Denylist match in append | `superteam halted at Team Lead: project delta for <role> attempts to weaken non-negotiable rules (matched: <pattern>)` |
| Unsupported host at pre-flight | `superteam halted at pre-flight: host <host> has no shipped per-role agent files; supported hosts: claude-code, codex` |

---

## Audit-log strings (Team Lead emits these verbatim)

| Event | Verbatim audit string |
|---|---|
| Delta applied (model, append, or tools) | `superteam delta applied: <role> (<applied-fields>); non-negotiable-rules-sha=<8-char-prefix>` |
| Delta file empty or frontmatter-only | `superteam delta empty: <role>` |
| Orphan file found during pre-flight scan | `superteam delta orphan: docs/superpowers/<file> does not match any shipped role` |
| Delta `model: inherit` on non-inherit shipped role | `superteam delta inherit-redundant: <role>` |
| Delta requests tool not available on host | `superteam delta tool unavailable: <role> <tool>@<host>` |
| Active host resolved at pre-flight | `superteam active host: <name> (probe=<source>)` |

Default destination: operator-facing chat surface. Stderr fallback when no chat
surface is available (e.g. headless CI); re-emit at the start of Team Lead's next
chat-bearing message so the operator never permanently loses the audit trail (N9, LC3).

Every `superteam delta applied` line carries `non-negotiable-rules-sha=<8-char-prefix>`
(LC4).

---

## Active-host probe order

Probe order — first match wins; result logged once at pre-flight as
`superteam active host: <name> (probe=<source>)`:

1. `CLAUDECODE` / `CLAUDE_CODE_*` env-var family present → `claude-code`
2. `CODEX_*` env-var family present → `codex`
3. Runtime self-id via capability probe (same surface used by R26 model-override detection)

Supported host set: `{ claude-code, codex }`. Out-of-supported-set hosts halt at
pre-flight (see halt strings above).
