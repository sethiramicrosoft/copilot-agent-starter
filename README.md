# copilot-agent-starter

Six domain agents for GitHub Copilot CLI. Each one owns a specific job, scopes its own tools, and wires to whatever backend you already use via MCP. You swap the MCP config — the agent behaviour doesn't change.

No extra subscriptions. Works with whatever GitHub Copilot plan you already have.

Companion to [copilot-fleet-starter](https://github.com/sethiramicrosoft/copilot-fleet-starter) — the code review persona fleet. This repo does the work; that repo reviews it.

## What this repo gives you

```
~/.copilot/
├── copilot-instructions.md
├── AGENTS.md                         ← agent catalog + model assignments
└── agents/
    ├── mail-agent.agent.md           ← Outlook / M365
    ├── project-agent.agent.md        ← GitHub, GitLab
    ├── data-agent.agent.md           ← Postgres, SQLite, Azure SQL
    ├── devops-agent.agent.md         ← GitHub Actions, GitLab CI
    ├── workspace-agent.agent.md      ← M365 / Teams, Slack, Notion
    └── research-agent.agent.md       ← web + code search, cited reports

mcp-examples/
├── mail/       → outlook.json
├── database/   → postgres.json, sqlite.json, mssql.json, snowflake.json
├── cicd/       → github-actions.json, gitlab-ci.json, azure-devops.json
├── workspace/  → m365.json, slack.json, notion.json
└── vcs/        → github.json, gitlab.json, jira.json
```

## Why this exists

If you only have GitHub Copilot CLI — no M365 Copilot CoWork, no Claude Cowork — you can still get domain-specific agents for your day-to-day work. This repo gives you a ready-made starting point so you're not reading docs and figuring out YAML formats from scratch.

Each agent separates two things:

1. **Behaviour** — what the agent does, how it thinks, what it won't touch. Never changes.
2. **MCP config** — which external tool it connects to. The only thing you swap.

Pick a config from `mcp-examples/`, paste it into the agent's `mcp-servers:` block, set your secrets, done. See [SETUP.md](SETUP.md) for the full walkthrough.

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

That's it. All six agents load immediately — no MCP config needed to get started.

## Try it now (no credentials needed)

Run these two commands to confirm everything is working before you touch anything else:

```bash
# research-agent — built-in web + GitHub search, zero setup
gh copilot -- --agent research-agent -p "What is the latest stable version of Node.js?" --allow-all-urls

# devops-agent — shell + built-in GitHub MCP, zero setup
gh copilot -- --agent devops-agent -p "Are there any GitHub Actions workflows in sethiramicrosoft/copilot-agent-starter?" --allow-all
```

If both respond, you're good. The other four agents work the same way once you wire their MCP — see [SETUP.md](SETUP.md).

## Usage

**Interactive session** — start a session and select or call an agent by name:
```bash
gh copilot
# inside the session:
# /agent                            ← browse and pick an agent
# Use the research-agent to...      ← call by name in your prompt
```

**One-shot from the terminal** — useful for scripting or quick tasks:
```bash
gh copilot -- --agent research-agent -p "your prompt here" --allow-all-urls
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

All MCP configs use official vendor packages only — no community packages. Each config ships in `mcp-examples/` ready to copy into the agent's `mcp-servers:` block. See [SETUP.md](SETUP.md) for how.

| Agent | Config | Package / endpoint | Secrets needed |
|---|---|---|---|
| mail-agent | `mail/outlook.json` | `@microsoft/workiq` (Microsoft) | `COPILOT_MAIL_CLIENT_ID`, `COPILOT_MAIL_CLIENT_SECRET`, `COPILOT_MAIL_TENANT_ID` |
| project-agent | `vcs/github.json` | `@modelcontextprotocol/server-github` | `COPILOT_VCS_TOKEN` |
| project-agent | `vcs/gitlab.json` | `@modelcontextprotocol/server-gitlab` | `COPILOT_VCS_TOKEN` |
| project-agent | `vcs/jira.json` | Atlassian remote MCP (HTTPS, no install) | `COPILOT_VCS_TOKEN` (Atlassian API token) |
| data-agent | `database/postgres.json` | `@modelcontextprotocol/server-postgres` | `COPILOT_DB_CONNECTION_STRING` |
| data-agent | `database/sqlite.json` | `uvx mcp-server-sqlite` (Anthropic) | path to `.db` file — no secret |
| data-agent | `database/mssql.json` | `@azure/mcp` (Microsoft) | `AZURE_SUBSCRIPTION_ID`, `AZURE_RESOURCE_GROUP` |
| data-agent | `database/snowflake.json` | `uvx mcp-server-snowflake` (Snowflake) | `SNOWFLAKE_ACCOUNT`, `SNOWFLAKE_USER`, `SNOWFLAKE_PASSWORD`, `SNOWFLAKE_DATABASE`, `SNOWFLAKE_WAREHOUSE` |
| devops-agent | `cicd/github-actions.json` | `@modelcontextprotocol/server-github` | `COPILOT_CICD_TOKEN` |
| devops-agent | `cicd/gitlab-ci.json` | `@modelcontextprotocol/server-gitlab` | `COPILOT_CICD_TOKEN` |
| devops-agent | `cicd/azure-devops.json` | `@azure-devops/mcp` (Microsoft) | `COPILOT_CICD_ORG`, `COPILOT_CICD_TOKEN` |
| workspace-agent | `workspace/m365.json` | `@microsoft/workiq` (Microsoft) | `COPILOT_WORKSPACE_CLIENT_ID`, `COPILOT_WORKSPACE_CLIENT_SECRET`, `COPILOT_WORKSPACE_TENANT_ID` |
| workspace-agent | `workspace/slack.json` | `@modelcontextprotocol/server-slack` | `COPILOT_WORKSPACE_CLIENT_ID` (bot token), `COPILOT_WORKSPACE_TENANT_ID` (team ID) |
| workspace-agent | `workspace/notion.json` | `@notionhq/notion-mcp-server` (Notion) | `COPILOT_WORKSPACE_CLIENT_ID` (API key) |
| research-agent | none needed | built-in web + GitHub search | none |

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

## How this fits with M365 Copilot CoWork and Claude Cowork

Quick answer: they're different products solving different problems. You don't have to choose.

### What each product does

**M365 Copilot CoWork** (Microsoft, Frontier preview May 2026, $30–99/user/month on top of M365):
- Runs in the cloud — no desktop app needed
- Has Work IQ underneath: a semantic index of your org's M365 data, org graph, Dynamics, and external data via federated MCP connectors
- Work IQ CLI lets developers pull M365 org context into their own tools
- Microsoft controls model routing — you don't pick
- Native to M365 ecosystem; 3rd party tools available via MCP connectors (Frontier preview, requires admin setup)

**Claude Cowork** (Anthropic, 2026, requires a paid Claude plan):
- Runs on your desktop — your laptop has to be on and the app open
- Computer use: can open apps, navigate browsers, fill spreadsheets, organise local files
- Plugin marketplace with Slack, Notion, GitHub, Jira, Zapier and more
- Scheduled tasks — but only while your computer is awake
- Claude model only, no model switching
- Built for knowledge workers, not developers specifically

**copilot-agent-starter** (runs on your existing GitHub Copilot subscription):
- **Interactive use: your machine needs to be on**, same as any CLI tool
- Each agent has platform-enforced tool scoping (`tools:` array, not just instructions)
- Each agent scopes its own MCP servers — devops-agent can't reach mail MCP
- Works with GitHub, GitLab, Jira, Postgres, Snowflake, Slack, M365, Notion — whatever you already use
- No second vendor contract

**What runs in GitHub Actions (headless, no laptop):**

| Agent | Works in GitHub Actions? | Notes |
|---|---|---|
| devops-agent | ✅ Yes | Shell + GitHub MCP built-in. Natural fit for CI. |
| project-agent | ⚠️ Partially | GitHub operations work; other VCS needs MCP on the runner |
| data-agent | ⚠️ Partially | Works if your DB is reachable from the runner |
| research-agent | ❌ No | Web search is not available in cloud/headless mode |
| workspace-agent | ❌ Not really | Technically possible but no practical CI use case |
| mail-agent | ❌ Not really | Technically possible but no practical CI use case |

### Feature comparison

| | M365 CoWork | Claude Cowork | agent-starter |
|---|---|---|---|
| **Runs without a laptop** | ✅ cloud — confirmed, tasks keep running across devices | ❌ laptop must stay on and app open | ❌ interactive use needs your machine on; devops-agent runs headless in GitHub Actions |
| **Works outside Microsoft stack** | ⚠️ 3rd party via MCP (Frontier preview, admin setup) | ✅ plugins | ✅ any MCP server |
| **You control model per agent** | ❌ Microsoft routes | ❌ Claude only | ✅ per-agent in YAML |
| **Platform-enforced tool scoping** | ✅ | ❌ plugin OAuth | ✅ `tools:` array |
| **Runs in GitHub Actions** | ❌ | ❌ | ⚠️ devops-agent yes; project-agent partially; research/mail/workspace no |
| **Can query a database** | limited | ❌ | ✅ via MCP |
| **No extra subscription** | ❌ $30–99/mo add-on | ❌ separate paid plan | ✅ uses your GHCP seat |
| **Org semantic index** | ✅ Work IQ | ❌ | ❌ |
| **Computer use (apps, browser)** | ❌ | ✅ | ❌ |
| **Non-technical user UX** | ✅ | ✅ | ❌ CLI only |

### When to use which

**M365 CoWork** — your org is deep in M365 and you want cloud-native org intelligence (who owns what, collaboration patterns, Dynamics data). Great for knowledge workers who don't want to touch a terminal.

**Claude Cowork** — you need computer use (browser automation, local file work, spreadsheets) or recurring tasks that run from a desktop. Strong for non-technical users not on M365.

**agent-starter** — you have GHCP and want domain agents for your actual dev workflow. Especially useful if:
- You're on GitLab, Jira, Snowflake, or anything outside Microsoft's ecosystem
- You want devops-agent running headless in GitHub Actions as a CI diagnostic tool
- You don't want to pay for another platform subscription
- You want tool boundaries enforced at the platform level, not just by prompt

### What it doesn't do

- No org-wide semantic index like Work IQ — it doesn't know your org's collaboration graph
- CLI only — non-technical teammates won't find it friendly
- Mail and workspace agents need OAuth app registration — not hard, but not instant either
- Model inference goes through GitHub Copilot's endpoints — it's not air-gapped
- Heavy usage may hit premium request limits on your Copilot plan

### Complementary use example

> 1. **M365 CoWork** surfaces a risk in the Work IQ org graph
> 2. **workspace-agent** pulls the meeting notes and extracts action items
> 3. **project-agent** creates issues in Jira or GitHub
> 4. **devops-agent** checks if the related CI pipeline is green
>
> M365 CoWork stays in its lane. agent-starter crosses the boundaries it can't.

## Use with copilot-fleet-starter

Install both repos. Agents do the work; personas review it.

```
1. research-agent   → research the approach
2. Main session     → build it
3. project-agent    → open the PR
4. devops-agent     → diagnose CI if it fails
5. workspace-agent  → post the release summary to the team channel
6. /fleet personas  → optional: run security, perf, UX review in parallel before merge
```

The persona fleet from [copilot-fleet-starter](https://github.com/sethiramicrosoft/copilot-fleet-starter) dispatches 10 specialist reviewers via `/fleet` — each pinned to the model best suited to its role. Both repos run on the same CLI.

## What makes this different from personas

| copilot-fleet-starter (personas) | copilot-agent-starter (agents) |
|---|---|
| `~/.copilot/personas/*.md` | `~/.copilot/agents/*.agent.md` |
| Plain markdown | YAML frontmatter + markdown |
| Model pinned via instruction rule | `model:` property in frontmatter |
| Tool use scoped by instructions | `tools:` array (platform-enforced) |
| Shared global MCPs | `mcp-servers:` scoped per agent |
| Invoked via `/fleet` for parallel review | Invoked via `/agent` or `--agent=` |
| **Purpose: review code** | **Purpose: do domain work** |

> **Note on `model:`** — honoured in VS Code and interactive CLI. If you're using cloud agent (GitHub Actions), verify it's respected in your GHCP plan.

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
- [Automate Copilot CLI with GitHub Actions](https://docs.github.com/en/copilot/how-tos/copilot-cli/automate-copilot-cli/automate-with-actions)
- [copilot-fleet-starter](https://github.com/sethiramicrosoft/copilot-fleet-starter)

## License

MIT. Fork, swap the MCPs, add your own agents.
