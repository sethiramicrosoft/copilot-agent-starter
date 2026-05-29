# copilot-agent-starter

A vendor-agnostic reference setup for **domain-scoped agents** on GitHub Copilot CLI. Each agent owns a domain, pins its own model, and wires its own MCP servers. Swap the MCP config for your stack — the agent behaviour stays the same.

Companion to [copilot-fleet-starter](https://github.com/sethiramicrosoft/copilot-fleet-starter) (code review personas). This repo is the execution layer; that repo is the review layer.

## What this repo gives you

```
~/.copilot/
├── copilot-instructions.md
├── AGENTS.md                         ← agent catalog + model assignments
└── agents/
    ├── mail-agent.agent.md           ← any mail provider (Outlook, Gmail, IMAP)
    ├── project-agent.agent.md        ← any VCS/PM tool (GitHub, GitLab, Jira, ADO)
    ├── data-agent.agent.md           ← any database (Postgres, Snowflake, SQLite...)
    ├── devops-agent.agent.md         ← any CI/CD (GitHub Actions, ADO, GitLab, Jenkins)
    ├── workspace-agent.agent.md      ← any workspace (M365, Slack, Notion, Confluence)
    └── research-agent.agent.md       ← web + code search, cited reports

mcp-examples/
├── mail/       → outlook.json, gmail.json
├── database/   → postgres.json, sqlite.json, snowflake.json, mssql.json
├── cicd/       → github-actions.json, azure-devops.json, gitlab-ci.json
├── workspace/  → m365.json, slack.json, notion.json
└── vcs/        → github.json, gitlab.json, jira.json
```

## Why vendor-agnostic?

Each `.agent.md` separates two things:

1. **Behaviour** — what the agent does, how it behaves, what it ignores. Never changes regardless of your stack.
2. **MCP config** — which external tool it connects to. The only thing you swap.

Pick your configs from `mcp-examples/`, paste into the agent's `mcp-servers:` block, set secrets, done.

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
