# Setup Guide

This guide helps you configure the agent fleet for your organisation's specific toolstack, and shows which roles in your organisation benefit most from each agent.

---

## Step 1 — Pick your stack

Choose one MCP config per category and copy it into the relevant agent's `mcp-servers:` block.

### Mail

| Provider | Config file |
|---|---|
| Microsoft Outlook / Exchange | `mcp-examples/mail/outlook.json` |
| Gmail / Google Workspace | `mcp-examples/mail/gmail.json` |

### Version control & project management

| Tool | Config file |
|---|---|
| GitHub | `mcp-examples/vcs/github.json` |
| GitLab | `mcp-examples/vcs/gitlab.json` |
| Jira | `mcp-examples/vcs/jira.json` |
| Azure DevOps Boards | `mcp-examples/vcs/azure-devops.json` |

### CI/CD

| Tool | Config file |
|---|---|
| GitHub Actions | `mcp-examples/cicd/github-actions.json` |
| Azure DevOps Pipelines | `mcp-examples/cicd/azure-devops.json` |
| GitLab CI | `mcp-examples/cicd/gitlab-ci.json` |
| No MCP available (Jenkins, CircleCI, etc.) | Use `execute` shell fallback — agent uses your CI CLI |

### Database

| Database | Config file |
|---|---|
| PostgreSQL | `mcp-examples/database/postgres.json` |
| SQLite | `mcp-examples/database/sqlite.json` |
| SQL Server | `mcp-examples/database/mssql.json` |
| Snowflake | `mcp-examples/database/snowflake.json` |

### Workspace / collaboration

| Platform | Config file |
|---|---|
| Microsoft 365 (Teams, SharePoint, Planner) | `mcp-examples/workspace/m365.json` |
| Slack | `mcp-examples/workspace/slack.json` |
| Notion | `mcp-examples/workspace/notion.json` |

---

## Step 2 — Set secrets

After choosing your configs, set these as Copilot agent secrets at the org or repo level:

| Secret name | Used by |
|---|---|
| `COPILOT_MAIL_CLIENT_ID` | mail-agent |
| `COPILOT_MAIL_CLIENT_SECRET` | mail-agent |
| `COPILOT_MAIL_TENANT_ID` | mail-agent (Outlook only) |
| `COPILOT_VCS_TOKEN` | project-agent |
| `COPILOT_VCS_ORG` | project-agent |
| `COPILOT_CICD_TOKEN` | devops-agent |
| `COPILOT_CICD_ORG` | devops-agent |
| `COPILOT_DB_CONNECTION_STRING` | data-agent |
| `COPILOT_WORKSPACE_CLIENT_ID` | workspace-agent |
| `COPILOT_WORKSPACE_CLIENT_SECRET` | workspace-agent |
| `COPILOT_WORKSPACE_TENANT_ID` | workspace-agent (M365 only) |

---

## Step 3 — Install

```powershell
# Windows
Copy-Item .\copilot-instructions.md $env:USERPROFILE\.copilot\
Copy-Item .\AGENTS.md $env:USERPROFILE\.copilot\
Copy-Item .\agents $env:USERPROFILE\.copilot\ -Recurse
```

```bash
# macOS / Linux
cp copilot-instructions.md ~/.copilot/
cp AGENTS.md ~/.copilot/
cp -r agents ~/.copilot/
```

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
