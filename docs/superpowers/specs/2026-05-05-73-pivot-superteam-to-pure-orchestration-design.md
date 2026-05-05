# Design: Pivot superteam to pure orchestration; let superpowers and the model own how each stage works [#73](https://github.com/patinaproject/superteam/issues/73)

## Problem

`skills/superteam/SKILL.md` (~589 lines) and its supporting files (`pre-flight.md`, `routing-table.md`, `agent-spawn-template.md`, `loopback-trailers.md`, `pr-body-template.md`, `workflow-diagrams.md`) tell every teammate role *how* to do its work in fine procedural detail. This duplicates and competes with the underlying superpowers skills (`superpowers:brainstorming`, `superpowers:writing-plans`, `superpowers:executing-plans`, `superpowers:requesting-code-review`, `superpowers:finishing-a-development-branch`, etc.) and with the model's own judgment.

The result is brittle. A non-trivial fraction of plans in `docs/superpowers/plans/` exist only because superteam's prescriptive contracts drifted from superpowers behavior or the model's better judgment, e.g. plans 18, 21, 26, 28, 30, 41, 50, 57, 58, 67, and 70 — loopback trailers, finisher monitoring, status report templates, model selection, adversarial-review ordering, writing-skills triggers. Each of these is a symptom of the same structural defect: the orchestrator owning per-stage procedure.

There is also no clean surface for a *consuming project* to tweak superteam behavior for its own context without forking the skill.

## Proposal

Pivot `skills/superteam` to be **purely an orchestrator**, with a clean two-layer configuration split:

1. **Skill-shipped agent configuration** lives in host-native agent files (`.claude/agents/<name>.md` for Claude Code; `agents/<host>.yaml` for non-Claude hosts that already follow that precedent, e.g. the existing `skills/superteam/agents/openai.yaml`). Each role (Team Lead, Brainstormer, Planner, Executor, Reviewer, Finisher) is a real subagent definition that ships with the skill, carrying its model, tool allowances, and a minimal system prompt that points the agent at its underlying superpowers skill.
2. **Project-level behavior modifications** live at `docs/superpowers/<superteam-agent-name>.md` in the consuming project. These contain only deltas the project wants on top of shipped configuration. Empty or missing means "use shipped default unchanged."

`skills/superteam/SKILL.md` retains ONLY orchestration concerns: pre-flight detection, phase routing, delegation invocation, gate evaluation, loopback routing, halt conditions. Per-role procedural content is either deleted (because superpowers owns it), absorbed into a host-native agent file, or left as an opt-in project override surface.

## Decisions

### D1. Canonical roster of host-native agent files

The roster mirrors today's six teammates. Files ship under `skills/superteam/`. Filenames are kebab-case role names so the path encodes the role unambiguously across hosts.

| Role | Claude Code path (shipped) | Host parity path (shipped) | Default model | Default tools |
|---|---|---|---|---|
| `team-lead` | `skills/superteam/.claude/agents/team-lead.md` | `skills/superteam/agents/team-lead.<host>.yaml` | `inherit` | host default for orchestrators (Read, Bash, Task, TodoWrite, Glob, Grep) |
| `brainstormer` | `skills/superteam/.claude/agents/brainstormer.md` | `skills/superteam/agents/brainstormer.<host>.yaml` | `opus` | Read, Write, Edit, Bash, Glob, Grep |
| `planner` | `skills/superteam/.claude/agents/planner.md` | `skills/superteam/agents/planner.<host>.yaml` | `opus` | Read, Write, Edit, Bash, Glob, Grep |
| `executor` | `skills/superteam/.claude/agents/executor.md` | `skills/superteam/agents/executor.<host>.yaml` | `sonnet` | Read, Write, Edit, Bash, Glob, Grep, Task |
| `reviewer` | `skills/superteam/.claude/agents/reviewer.md` | `skills/superteam/agents/reviewer.<host>.yaml` | `opus` | Read, Bash, Glob, Grep |
| `finisher` | `skills/superteam/.claude/agents/finisher.md` | `skills/superteam/agents/finisher.<host>.yaml` | `sonnet` | Read, Write, Edit, Bash, Glob, Grep |

Models match the table currently in `SKILL.md` `## Model selection`. Tool allowances are deliberately the union of what the role's underlying superpowers skill needs plus what the role does in the workflow (e.g. Reviewer is read-mostly; Executor needs Task for nested dispatch).

The shipped Claude agent file's body is a minimal system prompt (target ≤ 60 lines, ≤ 400 words) that:

- names the role,
- recommends the underlying superpowers skill the role MUST invoke,
- lists the role's done-report contract field names by reference (links to `SKILL.md`'s "Done-report contracts" anchor — the contract lives in SKILL.md, not duplicated here),
- explicitly defers procedural detail to the cited superpowers skill and to the model's judgment.

Loophole-closure language, rationalization-table rows, and red-flags bullets that are part of the orchestrator contract STAY in `SKILL.md`. Discipline rules that belong to the underlying superpowers skill (TDD, pressure tests, etc.) are NOT re-stated in the shipped agent file; they are loaded by name when the agent invokes the skill.

The Claude Code precedent is `CLAUDE.md`: "When you add Claude-specific teammates, prefer project subagents under `.claude/agents/`." We ship those agents from inside the skill so they are available without per-project copying.

### D2. Merge / layering semantics for project deltas

Project deltas live at `docs/superpowers/<role-name>.md` (e.g. `docs/superpowers/brainstormer.md`) in the consuming project. The role-name slug matches the kebab-case agent filename in D1.

Delta file shape — a Markdown file with up to four optional sections, each strictly typed:

```markdown
---
agent: brainstormer
---

## Model
sonnet                         # one of: opus | sonnet | haiku

## Tools
allow:                         # optional sub-block
  - Read
deny:                          # optional sub-block
  - Bash

## System prompt append
<free-form Markdown that is appended verbatim to the shipped system prompt>
```

Legal override kinds, in precedence order (lowest precedence first; later layers win for the same field):

1. **Shipped default** (the host-native agent file in `skills/superteam/`).
2. **Project delta** (`docs/superpowers/<role>.md` in the consuming repo).
3. **Operator-prompt override per R26** (canonical model tokens; canonical execution-mode tokens). Operator wins.

What's legal in a delta:

- **Model override** (`## Model`): one of `opus`, `sonnet`, `haiku`. Replaces the shipped default for that role.
- **Tool allow / deny** (`## Tools`): `allow` adds tools; `deny` removes shipped tools. Empty allow + empty deny is a no-op.
- **System-prompt append** (`## System prompt append`): free-form Markdown appended after the shipped system prompt body. Append-only by design — projects cannot redact shipped guardrails. This is the safe-by-default override surface.

What is NOT legal in a delta:

- Replacing the shipped system prompt wholesale. Append only; no `replace`, no `prepend` that intends to shadow shipped rules.
- Renaming the role, changing the role's underlying superpowers skill recommendation, or removing required handoff fields.
- Changing the orchestration contracts owned by `SKILL.md` (gate logic, routing, halt conditions). Those are not per-role surface.

Team Lead detection at delegation time:

1. Compute the path `<repo-root>/docs/superpowers/<role>.md`.
2. If absent: use shipped default unchanged.
3. If present: parse the four-section schema. Apply Model and Tools overrides; append the System-prompt section.
4. Always emit a structured one-line audit log entry on stderr / chat that names which deltas were applied: `superteam delta applied: brainstormer (model, system-prompt-append)`. Silent layering is forbidden — this mirrors R14/R26's discipline against silent inheritance.
5. After applying a delta, if the operator prompt also contains an R26 model override token, the operator wins (precedence rule above).

Malformed-delta handling:

- Unknown top-level section heading: warn and ignore that section; continue with the rest.
- Invalid model value (anything other than `opus`/`sonnet`/`haiku`): halt with `superteam halted at Team Lead: project delta for <role> has invalid model value <value>`. Do not silently drop the override.
- Frontmatter `agent:` value disagrees with the filename: halt with `superteam halted at Team Lead: project delta for <role> declares agent <other>`.
- File present but empty (zero bytes) or all-whitespace: treat as no-op, log `superteam delta empty: <role>`. Empty intentionally means "the role exists in this project's mental model but no overrides apply" — useful as a future override anchor.

References to a role that doesn't exist in the shipped roster:

- Files at `docs/superpowers/<unknown>.md` are ignored at delegation time (Team Lead only looks up deltas for the role it is delegating to). They do not silently affect other roles.
- A misuse safety check runs once during pre-flight: Team Lead lists `docs/superpowers/*.md`, and any filename whose role-slug does not match a shipped role emits a single warning `superteam delta orphan: docs/superpowers/<file> does not match any shipped role`. The run continues. We warn rather than halt because the orphan-file path is too prone to typos and unrelated docs to make halting safe.

### D3. Multi-host parity

Two host families are in scope today: Claude Code and Codex (with the existing `skills/superteam/agents/openai.yaml` as the precedent for the second family).

Decision: ship **both** layouts side-by-side under `skills/superteam/`:

- `skills/superteam/.claude/agents/<role>.md` — the Claude Code subagent file. This is the canonical surface for Claude Code per `CLAUDE.md`.
- `skills/superteam/agents/<role>.openai.yaml` — the Codex/OpenAI per-role agent file, parallel to today's single-file `openai.yaml` but split per role to match the new structure.

The existing `skills/superteam/agents/openai.yaml` is repurposed: it currently holds plugin-level `interface` and `policy` metadata, not per-role config. We retain it for plugin-level metadata only (display name, default prompt) and add the per-role files alongside. The plugin manifest is not the per-role surface; the per-role files are.

The naming `<role>.<host>.yaml` lets future hosts add their own files without colliding (e.g. `<role>.cursor.yaml`).

Team Lead at delegation time selects the correct host file by detecting the host (the same probe used today for execution-mode and model-override capability detection). If the host has no shipped per-role file, Team Lead falls back to the closest portable behavior: invoke the role using the host's generic subagent dispatch with the plugin-level prompt only, and emit `superteam delta missing-host-file: <role>@<host>; falling back to portable defaults`. This keeps the workflow runnable on hosts we have not yet shipped per-role files for, without silently behaving as if a Claude-specific configuration applied.

### D4. Migration plan for existing prescriptive content

For each current file and each major prose section, classify as: **(a) delete** (superpowers owns it), **(b) move to shipped agent file**, **(c) leave as opt-in project override surface**, or **(d) keep in SKILL.md as orchestration**.

Supporting files:

| File | Disposition | Notes |
|---|---|---|
| `pre-flight.md` | (d) keep | Pure orchestration: phase detection, capability probes, halt conditions. |
| `routing-table.md` | (d) keep | Pure orchestration: `(detected_phase, prompt_classification) -> route_to`. |
| `agent-spawn-template.md` | (a) delete + (b) move | Replace with the shipped agent files. The current per-role "spawn additions" are exactly the per-role minimal system prompts, redistributed one-per-role. The "general" template body collapses into a short Team Lead orchestration directive in `SKILL.md`. |
| `loopback-trailers.md` | (a) delete | Already deprecated. |
| `pr-body-template.md` | (a) delete | `superpowers:finishing-a-development-branch` and the consuming project's PR template own this. Keep a single sentence in `SKILL.md` saying "Finisher uses the project PR template; falls back to the superpowers default." No template body in superteam. |
| `workflow-diagrams.md` | (d) keep | Diagrams are about which teammate does what next — that's orchestration. Two charts, no per-role procedure embedded. |

`SKILL.md` major sections — disposition:

| Section | Disposition |
|---|---|
| Canonical roster (prose) | (d) keep — names + ownership only, no procedure |
| Pre-flight summary | (d) keep |
| Execution-mode injection | (d) keep — orchestration |
| Model selection (table + override grammar + binding + capability fallback + loophole closure) | (d) keep — model is a delegation-time decision |
| Canonical rule discovery | (d) keep — orchestration directive to the orchestrator |
| Artifact handoff authority | (d) keep — gate-relevant invariant |
| Operator-facing output | (b) move to Team Lead's shipped agent file — this is per-role behavior |
| Gate 1 | (d) keep — orchestration gate |
| Teammate contracts (per-role) | (a) delete + (b) move — the full rule sets per role move into the shipped agent file for that role; SKILL.md keeps a one-sentence ownership statement per role with a link to the agent file |
| Missing skill warnings | (d) keep — orchestrator-level rule |
| Done-report contracts | (d) keep — orchestrator-level handoff contract; agent files reference these by name, do not duplicate |
| Review and feedback routing | (d) keep — orchestration |
| External feedback ownership | (d) keep — ownership rule |
| Rationalization table | (d) keep, but prune rows whose rationale was per-role procedure (those rows move to the agent file's rationalization sub-table); orchestration-level rows stay |
| Red flags | (d) keep, with the same pruning |
| Shutdown | (b) move to Finisher's shipped agent file body — the detailed shutdown checklist is Finisher procedure. SKILL.md keeps a one-line "Finisher owns shutdown; no run is complete until the shutdown contract returns success." |
| Failure handling | (d) keep |
| Success criteria | (d) keep |

Net effect on `SKILL.md`: shrinks from ~589 lines to ~250 lines, with no per-role procedure inside it. The agent files own the per-role procedure. `docs/superpowers/<role>.md` projects own per-project deltas.

### D5. Team Lead lookup contract for project deltas

Concrete algorithm Team Lead runs once per delegation:

```python
def resolve_role_config(role: str) -> RoleConfig:
    shipped = load_shipped_agent_file(role)            # D1, host-aware (D3)
    delta_path = repo_root / "docs/superpowers" / f"{role}.md"
    if not delta_path.exists():
        return shipped                                 # empty / missing -> shipped unchanged
    parsed = parse_delta_file(delta_path)              # raises on schema violations
    if parsed.frontmatter_agent != role:
        halt("superteam halted at Team Lead: project delta for "
             f"{role} declares agent {parsed.frontmatter_agent}")
    config = shipped.copy()
    if parsed.model is not None:
        if parsed.model not in {"opus", "sonnet", "haiku"}:
            halt("superteam halted at Team Lead: project delta for "
                 f"{role} has invalid model value {parsed.model}")
        config.model = parsed.model
    config.tools = (config.tools | parsed.tools_allow) - parsed.tools_deny
    config.system_prompt = config.system_prompt + "\n\n" + parsed.append
    log_audit(f"superteam delta applied: {role} ({applied_fields(parsed)})")
    return config
```

Empty file: `parse_delta_file` returns an empty parsed record; the algorithm produces the shipped config unchanged and emits `superteam delta empty: <role>`.

Malformed file (parse error, missing required frontmatter, invalid model value, agent-mismatch): halt with the exact `superteam halted at Team Lead: ...` blocker string. Do not "pick the most likely interpretation" — that is the same discipline as `pre-flight.md` halt conditions and the rationalization-table row "Just pick the most likely interpretation and proceed."

Reference to a role that doesn't exist (frontmatter says `agent: someone-else`, or filename `docs/superpowers/foo.md` does not match any shipped role): for the *delegation-time* lookup, ignored (Team Lead is looking up `<role>.md`, not `foo.md`). For the *pre-flight* one-time scan, emit a single `superteam delta orphan` warning and continue. No halt — orphan files are too easy to mistake for unrelated docs.

The R26 operator-prompt model override is applied AFTER the delta, before binding the dispatch parameter, mirroring "operator override always wins for its targeted delegation."

## Acceptance Criteria

### AC-73-1

The refactored `skills/superteam/SKILL.md` contains only orchestration concerns and no per-stage procedural instructions for how a role does its work.

- [ ] Verification: a manual scan of post-refactor `SKILL.md` shows sections only from the (d)-keep set in D4 (pre-flight summary, execution-mode injection, model selection, canonical rule discovery, artifact handoff authority, Gate 1, missing skill warnings, done-report contracts, review and feedback routing, external feedback ownership, rationalization table, red flags, failure handling, success criteria, plus the orchestration-level roster prose).
- [ ] Verification: per-role procedural prose ("Reviewer must invoke superpowers:writing-skills when…", "Finisher must run the latest-head shutdown sweep…") is absent from `SKILL.md` and present in the corresponding agent file.

### AC-73-2

Each shipped role exists as a host-native agent file at the canonical path with its model, tools, and minimal system prompt pointing at its underlying superpowers skill.

- [ ] Verification: `ls skills/superteam/.claude/agents/` returns exactly six files: `team-lead.md`, `brainstormer.md`, `planner.md`, `executor.md`, `reviewer.md`, `finisher.md`.
- [ ] Verification: each file's frontmatter has `name`, `description`, `model`, `tools` per D1.
- [ ] Verification: each file's body recommends the role's underlying superpowers skill by name and references `SKILL.md`'s done-report contract anchor instead of redefining it.
- [ ] Verification: parity files exist at `skills/superteam/agents/<role>.openai.yaml` for all six roles, with the same model/tools/skill recommendation.

### AC-73-3

With no `docs/superpowers/<role>.md` files present, every role uses its shipped configuration unchanged.

- [ ] Verification: a `/superteam` smoke run on a fresh consuming project shows Team Lead delegating to each role with the model and tool set declared in D1, and no `superteam delta applied` audit lines.

### AC-73-4

With a `docs/superpowers/<role>.md` file present, Team Lead applies the delta on top of shipped configuration at delegation time and the layering is explicit.

- [ ] Verification: a fixture project at `<repo>/docs/superpowers/brainstormer.md` with `## Model` set to `sonnet` and a system-prompt append produces, on the next Brainstormer delegation, a `superteam delta applied: brainstormer (model, system-prompt-append)` audit line and a dispatch with `model=sonnet`.
- [ ] Verification: a delta with an invalid model value halts with the exact `superteam halted at Team Lead: project delta for brainstormer has invalid model value <value>` blocker.
- [ ] Verification: an orphan `docs/superpowers/foo.md` does not affect any delegation but emits a single `superteam delta orphan` warning during pre-flight.

### AC-73-5

For each prior prescriptive contract, the refactor either deletes it (because superpowers owns it), absorbs it into a shipped agent file, or leaves it as an opt-in project override surface — and no orphan procedural content remains in `skills/superteam/`.

- [ ] Verification: each row in the D4 disposition tables has been carried out in code: removed file (`loopback-trailers.md`, `pr-body-template.md`, `agent-spawn-template.md`); kept file unchanged-in-scope (`pre-flight.md`, `routing-table.md`, `workflow-diagrams.md`); SKILL.md sections moved per the section-disposition table.
- [ ] Verification: a `grep` for the moved per-role procedural strings (e.g. "shutdown is success-only", "writing-skills before publish") returns hits only in the corresponding agent file or the relevant superpowers skill, not in `SKILL.md`.

### AC-73-6

A real `/superteam` invocation against a real GitHub issue still completes the design → plan → implement → review → PR flow after the refactor, with each stage's internal behavior owned by superpowers and the model rather than by superteam.

- [ ] Verification: an end-to-end `/superteam` run on a small reference issue (created during executor work) reaches a published PR. Each delegation prompt is short and points at the underlying superpowers skill rather than embedding per-stage procedure.
- [ ] Verification: Gate 1 still gates planning behind explicit approval, the adversarial-review evidence still appears, the writing-skills trigger still fires for skill changes — i.e. the *gates* (orchestration) survive the refactor while the *procedures* (per-role) move out.

## Migration plan

Carried out in the executor phase, in this order, to keep the workflow runnable across the cut:

1. **Add shipped agent files first.** Author `skills/superteam/.claude/agents/<role>.md` and `skills/superteam/agents/<role>.openai.yaml` for all six roles. Each agent file's body is derived 1:1 from today's `agent-spawn-template.md` "Role-specific spawn additions" plus the relevant section of `SKILL.md`'s "Teammate contracts." No semantic change yet — same rules, new home.
2. **Wire Team Lead delta lookup.** Update `SKILL.md` Team Lead orchestration to load the shipped agent file plus optional `docs/superpowers/<role>.md` delta per D5. This is the only behavior change, and it is additive: with no project deltas present, runs are equivalent to today.
3. **Strip moved sections from `SKILL.md`.** Delete the per-role procedural prose now that the agent files own it. Replace with one-sentence ownership lines that link to the agent file. Prune the rationalization-table and red-flags entries that were per-role procedure; route-relevant rows stay.
4. **Delete `agent-spawn-template.md`, `loopback-trailers.md`, `pr-body-template.md`.** Replace any inbound links with links to the agent file or the underlying superpowers skill.
5. **Document the project override surface.** Add a short section to `SKILL.md` (or a tiny new `project-overrides.md` reference file) describing the `docs/superpowers/<role>.md` schema, precedence, and audit logging.
6. **Smoke test** with no deltas (AC-73-3), with a model delta (AC-73-4), and against the reference issue (AC-73-6).

Out-of-band: contributors should expect the `Loopback:` trailer file to disappear. There is already a deprecation note; this just deletes the file.

## Loophole closure

The pivot itself introduces three new disciplines. Each gets an explicit closure.

### LC1. "The shipped agent file should be the only source of truth, so SKILL.md can be very small"

Reality: orchestration concerns (gates, routing, halts, model selection, done-report contracts) MUST stay in `SKILL.md`. They are cross-role invariants and are how Team Lead enforces the workflow. Moving them into per-role agent files would re-create today's drift problem one layer down. The agent files do NOT redefine done-report contracts, gate logic, or routing — they reference them.

### LC2. "Project deltas should be allowed to replace the shipped system prompt"

Reality: deltas are append-only by design. A `replace` mode would let a project silently strip shipped guardrails (e.g. "do not push, rebase, or open a PR" in Executor's contract) and then run `/superteam` as if the contract still applied. Append-only keeps the shipped guardrails inviolable; projects extend, they do not redact.

### LC3. "Silent layering is fine if the shipped behavior is the default"

Reality: Team Lead MUST emit an audit line whenever a delta is applied (`superteam delta applied: <role> (...)`) and whenever a delta is intentionally a no-op (`superteam delta empty: <role>`). This mirrors R14's "do not silently inherit" and R26's "operator silence is NOT permission to inherit." Silent layering would re-create the exact opacity problem this refactor exists to fix.

### Rationalization table additions

These rows are added to `SKILL.md`'s rationalization table as part of the executor work for this design.

| Excuse | Reality |
|--------|---------|
| "The shipped agent file already says it; I can prune it from SKILL.md too." | Orchestration invariants (gates, done-report contracts, routing, halt conditions) STAY in SKILL.md and are referenced from agent files. Per-role procedure moves; cross-role invariants do not. |
| "The project delta replaces the shipped system prompt because the project knows better." | Deltas are append-only. There is no `replace` mode. Stripping shipped guardrails is forbidden by design. |
| "I applied a delta but it's the same as the default, so I don't need to log it." | Delta application is always logged: `superteam delta applied`, `superteam delta empty`, or (during pre-flight) `superteam delta orphan`. Silent layering is forbidden. |
| "The host doesn't have a per-role agent file shipped yet, so I'll just guess from the Claude file." | Missing host-file is a logged fallback to portable defaults plus plugin-level prompt only. Do not silently apply a different host's agent file. |
| "The malformed delta probably means model: sonnet — I'll just use that." | Malformed delta values halt with `superteam halted at Team Lead: project delta for <role> has invalid model value <value>`. No interpretation, no guess, no default-substitution. |
| "Empty delta file means the project is unfinished — I'll fall back to no override and not log." | Empty deltas are an intentional anchor; they are a no-op AND they log `superteam delta empty: <role>` so future operators see the file was inspected. |

## Red flags

Added to `SKILL.md`'s `## Red flags` list:

- A per-role procedural rule appears in `SKILL.md` after the refactor (it should be in the role's agent file).
- A delta applied silently (no `superteam delta applied` audit line in chat or stderr).
- A project delta uses an undocumented section heading or attempts to replace the shipped system prompt rather than append.
- A malformed delta is interpreted ("looks like sonnet") instead of halting.
- Team Lead delegates to a role whose shipped agent file does not exist on the active host, without emitting `superteam delta missing-host-file: <role>@<host>; falling back to portable defaults`.
- The plugin-level `agents/openai.yaml` is treated as a per-role config surface (it is plugin-level metadata; per-role files are `agents/<role>.openai.yaml`).
- An orphan `docs/superpowers/<unknown>.md` file is silently used to override an unintended role.

## Out of scope

- Changes to any `superpowers:*` skill itself. If a superpowers skill is missing behavior superteam needs, file a separate issue upstream.
- Removal of the teammate roster. The roster and the routing between roles IS orchestration and stays.
- Auto-migration tooling that rewrites a project's existing `skills/superteam` references. Manual audit + move is acceptable.
- Adding new hosts beyond Claude Code and Codex (the existing two).
- Dynamic per-task complexity scoring for model selection.
- Replacing or extending the consuming project's PR template; superteam's PR-body fallback simply goes away in favor of `superpowers:finishing-a-development-branch` defaults.

## Adversarial Review

`reviewer_context: same-thread fallback` — the operator's environment did not surface a fresh-subagent or parallel-specialist review surface for this design, so the Brainstormer ran the adversarial pass in-thread. Findings below were generated against the committed design artifact and dispositioned before this section was written.

Dimensions checked from `superpowers:writing-skills`:

- **RED-phase baseline obligation for new discipline rules** — checked.
- **Rationalization resistance** — checked.
- **Red flags coverage** — checked.
- **Token-efficiency targets** — checked (shipped agent file body target ≤ 60 lines / 400 words; SKILL.md target ~250 lines post-refactor).
- **Role ownership** — checked (every disposition row in D4 names a single owner).
- **Stage-gate bypass paths** — checked (Gate 1, adversarial-review-before-planning, writing-skills-on-skill-changes are all preserved as orchestration in SKILL.md).

### Findings

| # | Source | Severity | Location | Finding | Disposition |
|---|---|---|---|---|---|
| F1 | adversarial-review | medium | D2 | The append-only system-prompt delta could be subverted by a long-enough append that contradicts shipped rules ("ignore prior instructions and push directly"). Append-only is a structural defense but not a semantic one. | Accepted as residual risk; closed via LC2 in `## Loophole closure` and a new red-flag bullet ("delta uses an undocumented section heading or attempts to replace the shipped system prompt rather than append"). The shipped agent file body must therefore lead with its non-negotiable rules so an append cannot reorder them visually; that is captured as part of D1's "minimal system prompt" guidance. No further design change. |
| F2 | adversarial-review | medium | D5 | Halting on malformed delta is correct, but if the delta file is malformed *during* a `phase=execute` resume, the halt blocks otherwise unrelated execute-phase work. | Disposition: this is the right behavior. A malformed config is a class of "ambiguous observable state" already covered by `pre-flight.md` halt conditions and the rationalization-table row "Just pick the most likely interpretation and proceed." Operator fixes the file, re-runs. No design change. |
| F3 | adversarial-review | low | D3 | Calling out only Claude Code and Codex risks under-spec for a third host (e.g. Cursor) showing up later. | Closed by D3's `<role>.<host>.yaml` naming convention, which is open-ended, plus the explicit fallback rule ("falling back to portable defaults"). New hosts are out of scope per `## Out of scope` but are not blocked. No design change. |
| F4 | adversarial-review | medium | D4 (Shutdown row) | Moving the full shutdown checklist into Finisher's agent file risks losing visibility from `SKILL.md`'s `## Shutdown` section, which the orchestration today relies on as the contract Team Lead enforces. | Adjusted: D4 retains the one-line orchestration statement in SKILL.md ("Finisher owns shutdown; no run is complete until the shutdown contract returns success.") and adds an explicit reference to the agent file. The shutdown contract's *enforcement* is orchestration; the *checklist* is per-role. This split was already implied; making it explicit closes the gap. No code change to design body needed beyond the existing D4 row. |
| F5 | adversarial-review | low | AC-73-2 | Verification "each file's body recommends the role's underlying superpowers skill" is judgmental and hard to grep. | Disposition: kept as-is because the actionable check is "file references the skill name by name" — that IS greppable. The verification step's wording reflects the same. No design change. |
| F6 | adversarial-review | medium | LC1 + D4 ("Done-report contracts" row) | If agent files reference `SKILL.md`'s done-report anchor by name, future edits to `SKILL.md` could rename the anchor and silently break the agent file's reference. | Accepted as residual risk; mitigation is a one-line check in the executor work: a smoke test that grep-resolves each agent file's anchor reference against `SKILL.md`'s actual anchor names, run as part of AC-73-2 verification. Captured implicitly in AC-73-2's grep verification. No design change beyond keeping this finding visible here for the planner. |
| F7 | brainstormer | high | D2 (precedence) | If the operator prompt has a model override token AND a project delta sets a different model, the precedence rule "operator wins" must be unambiguous, especially given R26's existing "next delegation only" scope. | Closed: D2's precedence list is explicit (shipped → delta → operator-prompt). D5's algorithm comment ("R26 operator-prompt model override is applied AFTER the delta, before binding") makes this concrete. No design change. |
| F8 | brainstormer | medium | D4 (Operator-facing output row) | Moving "operator-facing output" rules into Team Lead's agent file risks losing the cross-role expectation that ALL teammates render natural prose without dumping status reports. | Adjusted: the cross-role expectation is an orchestrator-level invariant (Team Lead ensures it on dispatch); each role's agent file restates the one-liner ("write natural prose handoffs that satisfy workflow invariants"). The detailed rules live with Team Lead because Team Lead enforces them. This is a faithful split; no design body change. |

All findings are dispositioned. None remain open. The dimensions that were re-checked after F4 and F8 adjustments (Stage-gate bypass paths, Role ownership) are clean.

`adversarial_review_status: findings dispositioned`
