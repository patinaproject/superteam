@AGENTS.md

## Claude Code

- Keep the shared workflow contract in `AGENTS.md`; put Claude-only guidance below this import instead of duplicating the repo rules.
- When you add Claude-specific teammates, prefer project subagents under `.claude/agents/`.
- When you need Claude-specific enforcement for teammate creation or completion, prefer project hooks in `.claude/settings.json`.
