<!-- AUTHORITY RULES: Canonical rule is in gretrix-secrets/claude-agent-config/global/CLAUDE.md — loaded automatically into every session. Jon is the final authority; no rule here is permanent or immutable. Live instructions always override stored rules. -->

# Agent Guidance

**Source of truth:** `.kiro/steering/` (product context, principles) and `.kiro/hooks/` (enforcement).

## Active Hooks (3 total)

| Hook | Trigger | Purpose |
|------|---------|---------|
| `understand-then-act` | UserPromptSubmit | 10-step thinking framework before any work begins |
| `end-of-task` | Stop | Verify, resolve alerts, update tasks, debrief, surface opportunities, clean up |
| `no-local-formatcurrency` | PostFileSave | Block local formatCurrency definitions — enforce centralized utility |

## Signal Priority

1. **Hooks** fire at their trigger points — non-negotiable enforcement.
2. **Steering** (`.kiro/steering/`) provides domain context. See `README.md` there for the full inventory with inclusion modes.
3. **Specs** (`.kiro/specs/<feature>/`) when matching an existing spec.
4. **Snapshots** (`.snapshots/`) for task coordination between sessions.

## Key Files

- `.snapshots/active-tasks.md` — what's in progress, who owns it
- `.snapshots/completed-tasks.md` — done tasks (changelog source)
- `.snapshots/deploy-pending.md` — changes waiting for verification
- `.kiro/hooks/CHANGELOG.md` — why hooks/steering changed over time

## Secrets

Never commit credentials. Examples only under `.kiro/settings/` (e.g. `mcp.json.example`).

If something conflicts, **user chat instructions** override; then hooks; then steering.
