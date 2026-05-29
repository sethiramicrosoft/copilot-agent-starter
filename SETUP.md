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

## Step 2 — Authenticate (one-time, only for agents you'll use)

### GitHub (project-agent, devops-agent)

**Nothing to do** — both agents use Copilot CLI's built-in GitHub MCP, which authenticates with your active Copilot session. Just run them.

### Microsoft 365 (mail-agent, workspace-agent) — default

```powershell
# 1. Accept the workiq EULA
npx -y @microsoft/workiq accept-eula

# 2. Trigger device-code sign-in (caches your account in Windows Credential Manager)
npx -y @microsoft/workiq ask -q "hello"
```

The second command prints a device code, opens your browser, you sign in with your work/school Microsoft account → done.

### Google Workspace (Gmail / Calendar / Drive / Docs / Sheets — alternative to M365)

If you're on Google Workspace instead of M365, swap the workiq config for the community `workspace-mcp` package (PyPI, by taylorwilsdon — actively maintained, covers Gmail, Calendar, Drive, Docs, Sheets, Slides, Forms, Tasks, Contacts, Chat).

**One-time setup:**

1. Create a Google Cloud project at [console.cloud.google.com](https://console.cloud.google.com) (or reuse an existing one).
2. Enable the APIs you need (Gmail API, Calendar API, Drive API, etc.) — APIs & Services → Library.
3. Create OAuth 2.0 credentials — APIs & Services → Credentials → Create Credentials → OAuth client ID → Desktop app. Note the Client ID and Client Secret.
4. Set as env vars (or Copilot secrets):

```powershell
$env:GOOGLE_OAUTH_CLIENT_ID = "<your-client-id>"
$env:GOOGLE_OAUTH_CLIENT_SECRET = "<your-client-secret>"
```

5. Replace the `mcp-servers:` block in `~/.copilot/agents/mail-agent.agent.md` (or `workspace-agent.agent.md`) with the contents of `mcp-examples/mail/gmail.json` (or `mcp-examples/workspace/google-workspace.json`).
6. First run triggers an OAuth consent flow in your browser. Approve, done.

**Important caveats:**
- `workspace-mcp` is **community-maintained**, not a first-party Google package. (No equivalent first-party MCP from Google as of May 2026.)
- You own the OAuth app — you choose which scopes to grant. More setup than workiq's cached device code, but more control over what the agent can do.
- See [github.com/taylorwilsdon/google_workspace_mcp](https://github.com/taylorwilsdon/google_workspace_mcp) for the latest docs.

### Verify

```
copilot --agent research-agent -p "What is the latest stable version of Node.js?" --allow-all-urls
copilot --agent project-agent -p "List the 5 most recent issues in sethiramicrosoft/copilot-agent-starter" --allow-all
copilot --agent mail-agent -p "What are the subjects of my 3 most recent emails?" --allow-all
```

If all three respond, your install is good. Move to Step 3 only if you want to swap a default (e.g. use GitLab instead of GitHub) or wire data-agent to your database.

---

## Step 3 — Swap or add an MCP (optional)

The shipped agents are pre-wired with sensible defaults (M365 for mail/workspace, GitHub for project/devops). To swap, replace the `mcp-servers:` block with a different one from `mcp-examples/`. To wire `data-agent`, follow the example below.

### How to add MCP to an agent

1. Open the agent file, e.g. `~/.copilot/agents/data-agent.agent.md`
2. The `tools:` line already includes the MCP namespace (e.g. `"db-mcp/*"`) — no edit needed
3. Add the `mcp-servers:` block from the matching example **inside the `---` frontmatter fence**

**Before (ships like this — works immediately, no MCP):**
```yaml
---
name: data-agent
model: claude-opus-4.7
tools: ["read", "search", "execute", "db-mcp/*"]
---
```

**After (with SQLite MCP wired in):**
```yaml
---
name: data-agent
model: claude-opus-4.7
tools: ["read", "search", "execute", "db-mcp/*"]
mcp-servers:
  db-mcp:
    type: 'local'
    command: uvx
    args: ['mcp-server-sqlite', '--db-path', '/absolute/path/to/your.db']
    tools: ["*"]
---
```

> **Why `db-mcp/*` in the agent's `tools:` array?** The Copilot CLI filters MCP server tools through the agent's `tools:` list. Without the namespace, the MCP server loads but its tools are blocked silently. The MCP server key inside `mcp-servers:` (e.g. `db-mcp`) must match the namespace prefix in `tools:`. The shipped configs already match: `mail-mcp`, `vcs-mcp`, `db-mcp`, `cicd-mcp`, `workspace-mcp`.

The full args and env for each provider are in `mcp-examples/`. The JSON keys (e.g. `db-mcp`, `vcs-mcp`) match the namespaces already in each agent's `tools:` list — just copy the JSON contents into YAML format under `mcp-servers:`.

### Which example to use per agent

| Agent | Pick from |
|---|---|
| mail-agent | `mcp-examples/mail/` — outlook.json (`@microsoft/workiq` — default), gmail.json (Google Workspace via `workspace-mcp`) |
| project-agent | `mcp-examples/vcs/` — github.json, gitlab.json, jira.json (Atlassian remote MCP — no install needed) |
| devops-agent | `mcp-examples/cicd/` — github-actions.json, gitlab-ci.json, azure-devops.json (`@azure-devops/mcp`) |
| data-agent | `mcp-examples/database/` — postgres.json, sqlite.json (`uvx`, no secret), mssql.json (`@azure/mcp`), snowflake.json (`uvx`) |
| workspace-agent | `mcp-examples/workspace/` — m365.json (`@microsoft/workiq` — default), google-workspace.json (`workspace-mcp`), slack.json, notion.json |
| research-agent | No MCP needed — built-in web + GitHub search |

All these match what their `mcp-examples/` configs reference. Notes:

- **GitHub / GitHub Actions**: pre-wired agents use Copilot CLI's built-in `github/*` MCP — no PAT needed for project-agent and devops-agent out of the box. Use `vcs/github.json` or `cicd/github-actions.json` only if you want to override (e.g. scope to a specific PAT).
- **Google Workspace**: `workspace-mcp` is a community PyPI package by taylorwilsdon — actively maintained, covers Gmail/Calendar/Drive/Docs/Sheets/Slides/Forms/Tasks/Contacts/Chat. Needs your own Google Cloud OAuth app (see Step 2).
- **Snowflake**: `mcp-snowflake-server` is a community PyPI package (no Snowflake-official one exists as of May 2026).
- **Postgres / GitLab / Slack**: the `@modelcontextprotocol/server-*` packages are functional but marked deprecated on npm. They still work; replace when better-maintained alternatives ship.

---

## Step 4 — Set secrets (only for agents wired to a backend that needs them)

Set as environment variables in your shell (or your CI environment for headless runs). The `${{ secrets.<NAME> }}` syntax in the MCP example files is the GitHub Actions / Copilot cloud-agent secret resolution form; for local CLI use, set the same names as env vars in your shell:

```powershell
# PowerShell
$env:COPILOT_VCS_TOKEN = "<your token>"
```
```bash
# bash
export COPILOT_VCS_TOKEN="<your token>"
```

### Secrets matrix (only set what you wire)

| Secret | Required by | Notes |
|---|---|---|
| (none) | mail-agent, workspace-agent | workiq uses cached device-code auth — see Step 2 |
| (none) | research-agent | built-in web + GitHub search |
| `COPILOT_VCS_TOKEN` | `vcs/github.json` (override), `vcs/gitlab.json`, `vcs/jira.json` | GitHub PAT, GitLab PAT, or Atlassian API token |
| `COPILOT_VCS_ORG` | `vcs/gitlab.json` | GitLab base URL (e.g. `https://gitlab.com` or your self-hosted instance) |
| `COPILOT_CICD_TOKEN` | `cicd/github-actions.json` (override), `cicd/gitlab-ci.json`, `cicd/azure-devops.json` | Same idea per provider |
| `COPILOT_CICD_ORG` | `cicd/azure-devops.json`, `cicd/gitlab-ci.json` | Azure DevOps org name, or GitLab URL |
| `COPILOT_DB_CONNECTION_STRING` | `database/postgres.json` | Full Postgres URL e.g. `postgres://user:pass@host/db` |
| `AZURE_SUBSCRIPTION_ID`, `AZURE_RESOURCE_GROUP` | `database/mssql.json` | Used by `@azure/mcp` |
| `SNOWFLAKE_ACCOUNT`, `SNOWFLAKE_USER`, `SNOWFLAKE_PASSWORD`, `SNOWFLAKE_DATABASE`, `SNOWFLAKE_WAREHOUSE` | `database/snowflake.json` | Passed as `--account`/`--user`/... CLI args by the example config |
| `COPILOT_WORKSPACE_CLIENT_ID` | `workspace/slack.json` (bot token), `workspace/notion.json` (API key) | Reused across both; rename if you want them separate |
| `COPILOT_WORKSPACE_TENANT_ID` | `workspace/slack.json` (Slack team ID) | |
| `GOOGLE_OAUTH_CLIENT_ID`, `GOOGLE_OAUTH_CLIENT_SECRET` | `mail/gmail.json`, `workspace/google-workspace.json` | OAuth 2.0 Desktop credentials from your Google Cloud project — see Step 2 |

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
