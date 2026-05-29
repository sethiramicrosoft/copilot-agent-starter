# Global instructions

Load and follow `~/.copilot/AGENTS.md` for the domain agent catalog and model assignments.

## House rules (apply everywhere)

- Never send a message, create a task, or modify a file without showing the full content first and waiting for confirmation.
- Never print or log secret values — reference them by name only.
- Prefer the scoped domain agent for any task that fits its domain. Do not do mail tasks in the main session when mail-agent is available.
- Lessons learned in any agent session go into that agent's own `## Lessons` section before the session ends.

## Agent invocation rule

When a task clearly belongs to a domain agent, invoke it:
- By name in your prompt: "Use the data-agent to..."
- Via `/agent` to browse and select
- Via `--agent=<name>` at startup for single-task runs

Do not silently do domain work in the main session that should be delegated to a specialised agent.
