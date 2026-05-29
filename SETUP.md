# Setup Guide

This guide helps you get agents running and then wire them to your own tools via MCP.

---

## Step 1 — Install

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

---

## Step 2 — Verify it works before touching anything else

Two agents work with zero configuration. Test these first:

```
# No credentials needed — uses built-in web + GitHub search
gh copilot -- --agent research-agent -p "What is the latest stable version of Node.js?" --allow-all-urls

# No credentials needed — uses shell + built-in GitHub MCP
gh copilot -- --agent devops-agent -p "List the open GitHub Actions workflows in this repo" --allow-all
```

If both respond, your install is good. Move to Step 3 only when you're ready to connect a specific tool.

---

## Step 3 — Wire an MCP (optional, per agent)

Each agent ships without MCP configured so it loads immediately. When you want to connect a real backend, open the agent file and add the `mcp-servers:` block from the matching example in `mcp-examples/`.

### How to add MCP to an agent

1. Open the agent file, e.g. `~/.copilot/agents/data-agent.agent.md`
2. Find the `tools:` line in the frontmatter
3. Add the MCP tool names and `mcp-servers:` block **inside the `---` frontmatter fence**

**Before (ships like this — works immediately):**
```yaml
---
name: data-agent
model: claude-opus-4.7
tools: ["read", "search", "execute"]
---
```

**After (with Postgres MCP wired in):**
```yaml
---
name: data-agent
model: claude-opus-4.7
tools: ["read", "search", "execute", "db-mcp/query", "db-mcp/list_tables", "db-mcp/describe_table"]
mcp-servers:
  db-mcp:
    type: stdio
    command: npx
    args: ["-y", "@modelcontextprotocol/server-postgres"]
    env:
      DB_CONNECTION_STRING: ${{ secrets.COPILOT_DB_CONNECTION_STRING }}
---
```

The full args and env for each provider are in `mcp-examples/`. Just copy the relevant JSON into YAML format.

### Which example to use per agent

| Agent | Pick from |
|---|---|
| mail-agent | `mcp-examples/mail/` — outlook.json (`@microsoft/workiq`) |
| project-agent | `mcp-examples/vcs/` — github.json, gitlab.json, jira.json (Atlassian remote MCP — no install needed) |
| devops-agent | `mcp-examples/cicd/` — github-actions.json, gitlab-ci.json, azure-devops.json (`@azure-devops/mcp`) |
| data-agent | `mcp-examples/database/` — postgres.json, sqlite.json (`uvx`, no secret), mssql.json (`@azure/mcp`), snowflake.json (`uvx`) |
| workspace-agent | `mcp-examples/workspace/` — m365.json (`@microsoft/workiq`), slack.json, notion.json (`@notionhq/notion-mcp-server`) |
| research-agent | No MCP needed — built-in web + GitHub search |

All packages are official — published by the tool vendor or the modelcontextprotocol org. No community packages.

---

## Step 4 — Set secrets (only for agents with MCP wired)

Set secrets as environment variables or Copilot agent secrets. Only set what you actually use.

| Secret name | Used by |
|---|---|
| `COPILOT_MAIL_CLIENT_ID` | mail-agent (Outlook via `@microsoft/workiq`) |
| `COPILOT_MAIL_CLIENT_SECRET` | mail-agent |
| `COPILOT_MAIL_TENANT_ID` | mail-agent |
| `COPILOT_VCS_TOKEN` | project-agent — GitHub/GitLab PAT, or Atlassian API token for Jira |
| `COPILOT_VCS_ORG` | project-agent (GitHub/GitLab only) |
| `COPILOT_CICD_TOKEN` | devops-agent |
| `COPILOT_CICD_ORG` | devops-agent |
| `COPILOT_DB_CONNECTION_STRING` | data-agent (Postgres) |
| `AZURE_SUBSCRIPTION_ID` | data-agent (Azure SQL via `@azure/mcp`) |
| `AZURE_RESOURCE_GROUP` | data-agent (Azure SQL via `@azure/mcp`) |
| `SNOWFLAKE_ACCOUNT` | data-agent (Snowflake) |
| `SNOWFLAKE_USER` | data-agent (Snowflake) |
| `SNOWFLAKE_PASSWORD` | data-agent (Snowflake) |
| `SNOWFLAKE_DATABASE` | data-agent (Snowflake) |
| `SNOWFLAKE_WAREHOUSE` | data-agent (Snowflake) |
| `COPILOT_WORKSPACE_CLIENT_ID` | workspace-agent (M365/Slack/Notion) |
| `COPILOT_WORKSPACE_CLIENT_SECRET` | workspace-agent (M365 only) |
| `COPILOT_WORKSPACE_TENANT_ID` | workspace-agent (M365 only) |

---

## Who uses which agent — by role

### Software engineer / developer

| Agent | Use case |
|---|---|
| `project-agent` | Create issues from code review findings, open PRs, link commits to tickets |
| `devops-agent` | Diagnose failing CI builds, update pipeline configs, manage Dockerfiles |
| `data-agent` | Query production data for debugging, analyse logs, check data migrations |
| `research-agent` | Compare libraries, look up API docs, find reference implementations |

**Example:**
> "Our CI pipeline has been failing for the last 3 runs. Use devops-agent to diagnose it, then open a GitHub issue with the findings using project-agent."

---

### DevOps / platform engineer

| Agent | Use case |
|---|---|
| `devops-agent` | Monitor deployments, manage pipeline templates, enforce security gates |
| `project-agent` | Track infra work items, link pipeline failures to issues |
| `data-agent` | Query deployment logs, analyse incident timelines |

**Example:**
> "Use devops-agent to check all pipelines that ran in the last 24 hours and flag any that took more than 20 minutes."

---

### Data analyst / data engineer

| Agent | Use case |
|---|---|
| `data-agent` | Write and run SQL, analyse CSVs, identify anomalies, produce summaries |
| `project-agent` | Create data quality issues, track schema change requests |
| `research-agent` | Research data modelling approaches, compare tools |

**Example:**
> "Use data-agent to find customers who haven't ordered in 90 days, grouped by region, and show me the top 10 by lifetime value."

---

### Product manager / project manager

| Agent | Use case |
|---|---|
| `project-agent` | Create sprint issues from retro notes, update milestones, generate changelogs |
| `workspace-agent` | Collect action items from standup channels, summarise weekly activity |
| `research-agent` | Competitive research, feature comparison, user research synthesis |
| `mail-agent` | Draft stakeholder updates, triage meeting requests |

**Example:**
> "Use workspace-agent to collect all action items from #product-standup this week, then use project-agent to create issues for each one in the current sprint."

---

### Engineering manager / team lead

| Agent | Use case |
|---|---|
| `workspace-agent` | Summarise what each team member shipped this week from channel activity |
| `project-agent` | Sprint health — open issues, blocked tickets, PRs waiting for review |
| `devops-agent` | Pipeline health — flaky tests, slow builds, deployment frequency |
| `mail-agent` | Draft performance feedback, summarise email threads |

**Example:**
> "Use workspace-agent to summarise what the team shipped this sprint from Teams, and use project-agent to show me all PRs still open against this milestone."

---

### Executive assistant / operations

| Agent | Use case |
|---|---|
| `mail-agent` | Triage inbox, draft replies, manage calendar, find free meeting slots |
| `workspace-agent` | Collect notes from meetings, summarise decisions, create follow-up tasks |

**Example:**
> "Use mail-agent to summarise my inbox from the last 48 hours, flag anything that needs a reply today, and draft responses for the top 3."

---

### Executive / senior leadership

| Agent | Use case |
|---|---|
| `workspace-agent` | Weekly summary of key channels and decisions made |
| `data-agent` | Business metrics on demand — revenue, churn, usage trends |
| `research-agent` | Market research, competitor analysis, technology briefings |

**Example:**
> "Use data-agent to show me monthly active users and revenue for the last 6 months, then use research-agent to find what our top competitor shipped last quarter."

---

## Adding your own agents

Each `.agent.md` is a template. To add a new domain agent:

1. Copy any existing agent file
2. Change `name:`, `description:`, `model:`, `tools:`, and `mcp-servers:`
3. Update the instructions section for the new domain
4. Drop it in `~/.copilot/agents/` or `.github/agents/` for repo-scoped agents
5. Add it to `AGENTS.md` so the model assignment is documented

Common additions organisations make:
- `salesforce-agent` — CRM queries and record updates
- `confluence-agent` — wiki search and page creation
- `servicenow-agent` — ITSM ticket management
- `analytics-agent` — Google Analytics / Mixpanel / Amplitude queries
- `finance-agent` — expense reports, budget queries
