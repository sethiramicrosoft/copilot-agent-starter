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

## How this compares to M365 Copilot CoWork and Claude CoWork

These products solve genuinely different problems. Here is an honest, grounded comparison.

### What each product actually is

**M365 Copilot CoWork** (Microsoft, preview as of May 2026, $30–99/user/month add-on):
- Cloud execution — no laptop required, no desktop app
- Work IQ layer: semantic org index across M365 + Dynamics + external data via federated MCP connectors
- Work IQ CLI brings M365 org context into developer environments via MCP
- Multi-model internally (Microsoft controls routing — you do not choose)
- Scoped to Microsoft ecosystem: SharePoint, Teams, Outlook, Dynamics, M365 Graph
- 13 built-in skills + 20 Frontier preview skills

**Claude CoWork** (Anthropic, May 2026):
- Desktop execution — laptop must be on and running
- Computer use: opens apps, navigates browser, fills spreadsheets, organises local files
- Plugin marketplace: Slack, Notion, GitHub, Jira, Zapier, CRM connectors
- Scheduled recurring tasks (laptop-bound)
- Claude model only — no model switching
- Positioned at knowledge workers and non-technical users
- Requires Claude for Work subscription (separate cost from any GHCP license)

**This fleet** (GHCP CLI agents + persona fleet, included in your existing GitHub Copilot subscription):
- Runs in terminal, CI/CD runners, containers, headless — anywhere the CLI runs
- Platform-enforced tool and MCP scoping per agent (YAML frontmatter, not just instructions)
- Multi-model: each agent/persona pins the model best suited to its job
- Vendor-neutral: GitHub, GitLab, Jira, ADO, Postgres, Snowflake, Slack, M365, Notion, and more
- Review artifacts written as git-tracked markdown files (`reviews/<ticket>-<persona>.md`)
- Persistent lessons baked back into persona files between sessions

### Feature comparison

| | M365 CoWork | Claude CoWork | This fleet |
|---|---|---|---|
| **Runs without a laptop** | ✅ cloud | ❌ desktop-bound | ✅ CI / server / headless |
| **Works outside Microsoft stack** | ❌ M365 only | ✅ plugins | ✅ any MCP server |
| **Multi-model per task** | ❌ Microsoft routes | ❌ Claude only | ✅ per-agent pinned |
| **Platform-enforced tool scoping** | ✅ | ❌ plugin OAuth | ✅ YAML `tools:` array |
| **Auditable git-tracked artifacts** | ❌ | ❌ | ✅ markdown in repo |
| **Can gate CI/CD pipelines** | ❌ | ❌ | ✅ |
| **Can query a database** | limited | ❌ | ✅ via MCP |
| **Adversarial parallel review** | ❌ | ❌ | ✅ `/fleet` |
| **No extra subscription cost** | ❌ add-on | ❌ separate license | ✅ in GHCP |
| **Org semantic index / graph** | ✅ Work IQ | ❌ | ❌ |
| **Computer use (open apps, browser)** | ❌ | ✅ | ❌ |
| **Non-technical user UX** | ✅ | ✅ | ❌ CLI only |

### When to use which

**Use M365 CoWork when:** your team lives in M365, you want cloud-native org-wide memory (semantic index, collaboration graph, Dynamics data), and you need skills that work without developer involvement.

**Use Claude CoWork when:** you need computer use (browser automation, spreadsheet filling, local file organisation) or recurring scheduled tasks driven by a desktop agent. Strong for non-technical knowledge workers on non-M365 stacks.

**Use this fleet when:**
- You need multi-model adversarial code review (10 specialist reviewers in parallel, each on the best-fit model)
- CI/CD integration — running as a pre-merge gate, on ephemeral runners, without a laptop
- Platform-enforced least-privilege scoping per agent (required for regulated environments)
- Your team is on GitLab, Jira, Snowflake, Slack, Notion, or any non-M365/non-Anthropic stack
- You already have GHCP and cannot justify a second vendor contract
- You want review artifacts as version-controlled files attached to PRs, not chat transcripts

### Honest limitations of this fleet

- CLI only — non-technical stakeholders (PMs, execs) face a steeper learning curve than CoWork products
- No org-wide semantic index — Work IQ's relationship graph and Dynamics integration are genuinely better for org-level intelligence
- MCP server ecosystem is uneven in practice — some connectors are half-maintained or lack full auth flows
- 10 parallel persona reviews can generate review fatigue if teams don't triage the output
- Dependent on GHCP CLI's continued development roadmap

### Concrete example of complementary use

> 1. **M365 CoWork** surfaces a project risk using the Work IQ org graph (inside M365)
> 2. **workspace-agent** pulls the meeting notes and extracts action items
> 3. **project-agent** creates issues in Jira or GitHub
> 4. **devops-agent** checks if the related CI pipeline is green
> 5. **`/fleet`** runs 10 parallel reviews on the fix before merge
>
> M365 CoWork found the signal. This fleet acted on it across every boundary it cannot cross.

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
