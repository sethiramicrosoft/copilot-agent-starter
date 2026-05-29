# copilot-agent-starter

A vendor-agnostic reference setup for **domain-scoped agents** on GitHub Copilot CLI. Each agent owns a domain, pins its own model, and wires its own MCP servers. Swap the MCP config for your stack — the agent behaviour stays the same.

Companion to [copilot-fleet-starter](https://github.com/sethiramicrosoft/copilot-fleet-starter) — the code review persona fleet. This repo does the work; that repo reviews it.

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

Pick your configs from `mcp-examples/`, paste into the agent's `mcp-servers:` block, set secrets, done. See [SETUP.md](SETUP.md) for the full walkthrough.

## Install (2 minutes)

```powershell
# Windows
git clone https://github.com/sethiramicrosoft/copilot-agent-starter.git
Copy-Item .\copilot-agent-starter\copilot-instructions.md $env:USERPROFILE\.copilot\
Copy-Item .\copilot-agent-starter\AGENTS.md $env:USERPROFILE\.copilot\
Copy-Item .\copilot-agent-starter\agents $env:USERPROFILE\.copilot\ -Recurse
```

```bash
# macOS / Linux
git clone https://github.com/sethiramicrosoft/copilot-agent-starter.git
cp copilot-agent-starter/copilot-instructions.md ~/.copilot/
cp copilot-agent-starter/AGENTS.md ~/.copilot/
cp -r copilot-agent-starter/agents ~/.copilot/
```

## Usage

```
/agent                                        ← browse and select interactively
Use the workspace-agent to...                 ← call by name in a prompt
copilot --agent=research-agent --prompt "..."  ← CLI flag for one-shot tasks
```

## Examples by agent

### mail-agent
```
"Summarise my inbox from the last 48 hours, flag anything urgent, and draft replies for the top 3"
"Find a free 1-hour slot for 5 people next week and send a calendar invite"
"Triage everything in my inbox older than 7 days and suggest what to archive"
```

### project-agent
```
"Create issues for all action items from today's retro and assign them to the current sprint"
"Open a PR for the feature branch, link it to issue #42, and fill the PR template"
"Generate a changelog from commits since the last release tag"
```

### data-agent
```
"Which customers haven't ordered in 90 days? Show me the top 20 by lifetime value"
"Describe the users table — schema, row count, null rates per column, sample rows"
"Find any orders where the total doesn't match the sum of line items"
```

### devops-agent
```
"Our CI pipeline has failed 3 times in a row — diagnose it and propose a fix"
"Add a caching step to the build workflow to speed it up"
"Show me all deployments to production in the last 7 days and who triggered them"
```

### workspace-agent
```
"Collect all action items from the #project-alpha channel this week and create tasks for each"
"Summarise what was decided in the last 3 sprint planning meetings"
"Find the architecture decision record for the payments service in SharePoint/Notion"
```

### research-agent
```
"Compare Zustand vs Jotai vs Redux Toolkit for a large React app — trade-off table and recommendation"
"What are the breaking changes in Postgres 17 that would affect our schema?"
"Find reference implementations of event sourcing in Go on GitHub"
```

## Configuring secrets

| Agent | Secret names |
|---|---|
| mail-agent | `COPILOT_MAIL_CLIENT_ID`, `COPILOT_MAIL_CLIENT_SECRET`, `COPILOT_MAIL_TENANT_ID` |
| project-agent | `COPILOT_VCS_TOKEN`, `COPILOT_VCS_ORG` |
| data-agent | `COPILOT_DB_CONNECTION_STRING` |
| devops-agent | `COPILOT_CICD_TOKEN`, `COPILOT_CICD_ORG` |
| workspace-agent | `COPILOT_WORKSPACE_CLIENT_ID`, `COPILOT_WORKSPACE_CLIENT_SECRET`, `COPILOT_WORKSPACE_TENANT_ID` |
| research-agent | none — uses built-in web + GitHub search |

## Who uses which agent — by role

**Technical roles:**

| Role | Primary agents | What they use it for |
|---|---|---|
| Software engineer | project-agent, devops-agent, research-agent | Open PRs, debug pipelines, research libraries |
| DevOps / platform | devops-agent, project-agent | Monitor CI, manage pipeline templates |
| Data analyst | data-agent, research-agent | SQL on demand, methodology research |
| Data engineer | data-agent, project-agent | Schema analysis, data quality issues |
| QA engineer | project-agent, devops-agent | Track test failures, trigger pipeline reruns |
| Security engineer | research-agent, project-agent | CVE research, create security issues |

**Non-technical roles:**

| Role | Primary agents | What they use it for |
|---|---|---|
| Product manager | project-agent, workspace-agent, research-agent | Sprint issues from retros, competitor research |
| Project manager | project-agent, workspace-agent | Task tracking, status summaries across channels |
| Engineering manager | workspace-agent, project-agent, devops-agent | Team shipping summary, sprint health, pipeline health |
| Executive assistant | mail-agent, workspace-agent | Inbox triage, meeting notes, follow-up tasks |
| Executive | workspace-agent, data-agent, research-agent | Business metrics, market intelligence, weekly summaries |

See [SETUP.md](SETUP.md) for role-by-role examples with full prompts.

## How this complements M365 Copilot and Claude for Work

These are fundamentally different tools solving different problems:

| | M365 Copilot | Claude for Work | This fleet |
|---|---|---|---|
| **Lives in** | Office apps (Teams, Outlook, Word) | claude.ai browser tab | Your terminal |
| **What it does** | Helps you work *inside* Office | Answers questions, drafts content | Takes *actions* across your tools |
| **Can run code?** | ❌ | ❌ | ✅ |
| **Can query a database?** | ❌ | ❌ | ✅ |
| **Can trigger CI/CD?** | ❌ | ❌ | ✅ |
| **Can open a GitHub issue?** | ❌ | ❌ | ✅ |
| **Can switch models per task?** | ❌ | ❌ | ✅ |
| **Locked to one vendor?** | Microsoft only | Anthropic only | Any stack |

M365 Copilot and Claude for Work are **conversation tools** — they help you think and write faster inside their platform. This fleet is an **action system** — it reads from those platforms and does something with the output.

**Concrete example:**
> 1. M365 Copilot summarises your Teams meeting (inside Teams)
> 2. `workspace-agent` pulls those notes and extracts action items
> 3. `project-agent` creates the tickets in Jira or GitHub
> 4. `devops-agent` checks if the related CI pipeline is green
>
> M365 Copilot stayed in Teams. This fleet crossed every boundary it can't.

**M365 Copilot and Claude for Work are data sources. This fleet is what acts on the data.**

## Use with copilot-fleet-starter

Install both repos. Agents do the work; personas review it.

```
1. research-agent   → research the approach
2. Main session     → build it
3. /fleet personas  → security, performance, UX, accessibility reviewed in parallel
4. project-agent    → open the PR
5. devops-agent     → diagnose CI if it fails
6. workspace-agent  → post the release summary to the team channel
```

The persona fleet from [copilot-fleet-starter](https://github.com/sethiramicrosoft/copilot-fleet-starter) dispatches 10 specialist reviewers in parallel via `/fleet` — each on the model best suited to its role (Opus for architecture, Codex for security, Haiku for readability). These two repos run on the same CLI and complement each other.

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

## Adding your own agents

Copy any `.agent.md`, update the frontmatter and instructions, drop it in `~/.copilot/agents/`. Common additions:
- `salesforce-agent` — CRM queries and record updates
- `confluence-agent` — wiki search and page authoring
- `servicenow-agent` — ITSM ticket management
- `analytics-agent` — Mixpanel / Amplitude / GA queries
- `finance-agent` — expense and budget queries

## References

- [Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [Creating custom agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/create-custom-agents)
- [copilot-fleet-starter](https://github.com/sethiramicrosoft/copilot-fleet-starter)

## License

MIT. Fork, swap the MCPs, add your own agents.


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
