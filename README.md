# copilot-agent-starter

A reference setup for **domain-scoped agents** on GitHub Copilot CLI. Each agent owns a domain, pins its own model, and wires its own MCP servers — no shared context, no tool bleed.

Companion to [copilot-fleet-starter](https://github.com/sethiramicrosoft/copilot-fleet-starter) (code review personas). This repo is the execution layer; that repo is the review layer.

## What this repo gives you

```
~/.copilot/
├── copilot-instructions.md      ← global house rules
├── AGENTS.md                    ← agent catalog + model assignments
└── agents/
    ├── mail-agent.agent.md      ← Outlook/Gmail MCP, claude-sonnet-4.6
    ├── github-agent.agent.md    ← GitHub MCP scoped, gpt-5.3-codex
    ├── data-agent.agent.md      ← SQLite/DB MCP, claude-opus-4.7
    ├── devops-agent.agent.md    ← GitHub Actions + shell, gpt-5.3-codex
    ├── m365-agent.agent.md      ← Microsoft Graph MCP, claude-sonnet-4.6
    └── research-agent.agent.md  ← web + GitHub search, claude-opus-4.7
```

## What makes this different from personas

| copilot-fleet-starter (personas) | copilot-agent-starter (agents) |
|---|---|
| `~/.copilot/personas/*.md` | `~/.copilot/agents/*.agent.md` |
| Plain markdown | YAML frontmatter + markdown |
| Model pinned via instruction rule | Native `model:` property |
| Tool use scoped by instructions | Native `tools:` array (platform-enforced) |
| Shared global MCPs | `mcp-servers:` scoped per agent |
| Invoked via `/fleet` for parallel review | Invoked via `/agent` or `--agent=` |
| **Purpose: review code** | **Purpose: do domain work** |

## Install (2 minutes)

```powershell
# Windows
git clone https://github.com/sethiramicrosoft/copilot-agent-starter.git
Copy-Item .\copilot-agent-starter\copilot-instructions.md $env:USERPROFILE\.copilot\
Copy-Item .\copilot-agent-starter\AGENTS.md $env:USERPROFILE\.copilot\
Copy-Item .\copilot-agent-starter\agents $env:USERPROFILE\.copilot\ -Recurse
```

```bash
# macOS/Linux
git clone https://github.com/sethiramicrosoft/copilot-agent-starter.git
cp copilot-agent-starter/copilot-instructions.md ~/.copilot/
cp copilot-agent-starter/AGENTS.md ~/.copilot/
cp -r copilot-agent-starter/agents ~/.copilot/
```

Then add your MCP secrets — see [Configuring secrets](#configuring-secrets).

## Usage

```
# Browse and select an agent interactively
/agent

# Call by name in a prompt
Use the m365-agent to collect all action items from #project-alpha this week

# CLI flag for one-shot tasks
copilot --agent=research-agent --prompt "Compare Zustand vs Jotai for a large React app"
```

## Configuring secrets

Each agent that uses an external MCP needs credentials. Set them as Copilot agent secrets:

| Agent | Secret name(s) |
|---|---|
| mail-agent | `COPILOT_OUTLOOK_CLIENT_ID`, `COPILOT_OUTLOOK_CLIENT_SECRET`, `COPILOT_OUTLOOK_TENANT_ID` |
| m365-agent | `COPILOT_GRAPH_CLIENT_ID`, `COPILOT_GRAPH_CLIENT_SECRET`, `COPILOT_GRAPH_TENANT_ID` |
| data-agent | `COPILOT_DB_PATH` |

`github-agent`, `devops-agent`, and `research-agent` use the built-in GitHub MCP — no extra secrets needed.

## Agent boundaries

- Each agent only accesses the MCPs listed in its own profile.
- No agent merges code, deletes data, or sends a message without human confirmation.
- Lessons learned go into the agent's own `## Lessons` section at the end of the session.

## Use with copilot-fleet-starter

Install both repos. They complement each other:

```
# Do the work
copilot --agent=m365-agent --prompt "Draft a summary of this week's standup notes"

# Then review the output
/fleet review the draft using ux-critic and new-engineer
```

- **Agents** → do domain tasks (mail, data, GitHub, M365, research)
- **Personas via /fleet** → review code and content before it ships

## References

- [Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [Creating custom agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/create-custom-agents)
- [copilot-fleet-starter](https://github.com/sethiramicrosoft/copilot-fleet-starter)

## License

MIT.
