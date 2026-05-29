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

## Tool scoping — enforced, not instructed

Most agent setups rely on a prompt saying "only use relevant tools." That's a suggestion. This repo uses the `tools:` array in frontmatter — the platform enforces it. `devops-agent` cannot call mail MCP tools because they're not in its list. Each agent only loads the MCP servers declared in its own file.

```yaml
# devops-agent — cannot reach mail or database MCP, platform-enforced
tools: ["read", "edit", "search", "execute"]
mcp-servers:
  cicd-mcp:
    type: 'local'
    command: npx
    args: ['-y', '@azure-devops/mcp']
    tools: ["*"]
```

The boundary is verifiable and diffable in version control.

## Install (2 minutes)

> **Note on the CLI binary** — The official GitHub Copilot CLI is the standalone `copilot` binary, installed via `npm install -g @github/copilot` (see [docs](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli)). The examples below use `gh copilot --` (which works if you have the GitHub CLI Copilot extension installed); substitute `copilot` if you're on the standalone install. The flags after `--` are identical.

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
gh copilot -- --agent mail-agent -p "Find every unread email from the last 48 hours where someone is waiting on a response from me. List them by urgency with a one-line summary and draft a reply for the top 3." --allow-all

gh copilot -- --agent mail-agent -p "Find a free 1-hour slot that works for alice@company.com, bob@company.com and me next Tuesday or Wednesday between 10am-4pm London time. Propose 2 options." --allow-all

gh copilot -- --agent mail-agent -p "List every email thread older than 14 days where I sent the last message and got no reply. Group by sender domain." --allow-all
```

### project-agent
```
gh copilot -- --agent project-agent -p "Find all GitHub issues labelled 'bug' opened in the last 14 days in this repo. Group by assignee, sort by comment count descending, and output a markdown triage table." --allow-all

gh copilot -- --agent project-agent -p "Open a PR from branch 'feature/payments-v2' into main. Link it to issue #84, use the repo's PR template, and fill the 'What changed' section from the commit messages since the branch point." --allow-all

gh copilot -- --agent project-agent -p "Generate a changelog from all commits merged to main since the last git tag. Group by type: Features, Fixes, Chores. Exclude merge commits." --allow-all
```

### data-agent
```
gh copilot -- --agent data-agent -p "Query the orders table for rows where total != SUM(line_items.amount) for the same order_id. Return the top 20 discrepancies sorted by absolute delta, with order_id, customer_id, recorded total, and calculated total." --allow-all

gh copilot -- --agent data-agent -p "Describe the users table: row count, column names and types, null rate per column, min/max/avg for numeric columns, and 5 sample rows. Flag any column with null rate > 10%." --allow-all

gh copilot -- --agent data-agent -p "Find customers who placed at least 3 orders in the past but have had no activity in the last 90 days. Show the top 20 by lifetime value with their last order date and total spend." --allow-all
```

### devops-agent
```
gh copilot -- --agent devops-agent -p "Fetch the logs from the last 3 failed runs of the 'build' workflow in this repo. Identify if they share a common root cause and propose a fix." --allow-all

gh copilot -- --agent devops-agent -p "Review .github/workflows/build.yml and add a cache step for npm dependencies. Show the diff only — do not write the file." --allow-all

gh copilot -- --agent devops-agent -p "List all workflow runs that deployed to production in the last 7 days. For each: run ID, triggered by, duration, and outcome." --allow-all
```

### workspace-agent
```
gh copilot -- --agent workspace-agent -p "Collect every message from the #project-alpha Slack channel this week that contains an action item or a decision. Format as: Decision/Action | Owner | Due date (if mentioned)." --allow-all

gh copilot -- --agent workspace-agent -p "Summarise the last 3 sprint planning meeting notes from SharePoint/Notion. Extract: goals committed, risks flagged, and any items that were pushed to next sprint." --allow-all

gh copilot -- --agent workspace-agent -p "Search SharePoint and Notion for any document titled or tagged 'ADR' related to the payments service. Summarise the decisions made and the date of each." --allow-all
```

### research-agent
```
gh copilot -- --agent research-agent -p "Compare Zustand, Jotai, and Redux Toolkit for a large React app (50+ components, team of 8). Produce a trade-off table covering: bundle size, devtools, async handling, learning curve, and community activity. End with a recommendation." --allow-all-urls

gh copilot -- --agent research-agent -p "What are the breaking changes in Postgres 16 and 17 that would affect a schema using JSONB columns, partial indexes, and row-level security? Cite the release notes." --allow-all-urls

gh copilot -- --agent research-agent -p "Find production-grade reference implementations of the outbox pattern in Go on GitHub. Show repo name, stars, last commit date, and a one-line summary of the approach each uses." --allow-all-urls
```

## Configuring secrets

All MCP configs ship in `mcp-examples/` ready to copy into the agent's `mcp-servers:` block. See [SETUP.md](SETUP.md) for how.

> ⚠️ **Heads-up on MCP package status (May 2026)**: The official `@modelcontextprotocol/server-{github,gitlab,postgres,slack}` packages are marked **deprecated** (`"Package no longer supported"`) on npm — they still work but are no longer maintained by Anthropic. Recommended replacements:
> - **GitHub** → [`github/github-mcp-server`](https://github.com/github/github-mcp-server) (GitHub's own MCP server)
> - **GitLab** → [`gitlab-org/editor-extensions/gitlab-mcp-server`](https://gitlab.com/gitlab-org/editor-extensions/gitlab-mcp-server) (GitLab official)
> - **Postgres / Slack** → no official first-party replacement yet; community packages exist
>
> The `mcp-examples/` configs still reference the deprecated packages so the repo works out of the box — swap when ready.

| Agent | Config | Package / endpoint | Secrets needed |
|---|---|---|---|
| mail-agent | `mail/outlook.json` | `@microsoft/workiq` (Microsoft) | none in MCP config — uses cached device-code auth (run `npx -y @microsoft/workiq accept-eula` then `ask -q "hello"` once before first use) |
| project-agent | `vcs/github.json` | `@modelcontextprotocol/server-github` | `COPILOT_VCS_TOKEN` |
| project-agent | `vcs/gitlab.json` | `@modelcontextprotocol/server-gitlab` | `COPILOT_VCS_TOKEN` |
| project-agent | `vcs/jira.json` | Atlassian remote MCP (HTTPS at `mcp.atlassian.com/v1/mcp/authv2`, no install for cloud clients; IDEs need `mcp-remote` Node proxy) | `COPILOT_VCS_TOKEN` (Atlassian API token) |
| data-agent | `database/postgres.json` | `@modelcontextprotocol/server-postgres` | `COPILOT_DB_CONNECTION_STRING` |
| data-agent | `database/sqlite.json` | `uvx mcp-server-sqlite` (Anthropic) | path to `.db` file — no secret |
| data-agent | `database/mssql.json` | `@azure/mcp` (Microsoft) | `AZURE_SUBSCRIPTION_ID`, `AZURE_RESOURCE_GROUP` |
| data-agent | `database/snowflake.json` | `uvx mcp_snowflake_server` (community package by `allends`; no Snowflake-official PyPI package as of May 2026) | `SNOWFLAKE_ACCOUNT`, `SNOWFLAKE_USER`, `SNOWFLAKE_PASSWORD`, `SNOWFLAKE_DATABASE`, `SNOWFLAKE_WAREHOUSE` |
| devops-agent | `cicd/github-actions.json` | `@modelcontextprotocol/server-github` | `COPILOT_CICD_TOKEN` |
| devops-agent | `cicd/gitlab-ci.json` | `@modelcontextprotocol/server-gitlab` | `COPILOT_CICD_TOKEN` |
| devops-agent | `cicd/azure-devops.json` | `@azure-devops/mcp` (Microsoft) | `COPILOT_CICD_ORG`, `COPILOT_CICD_TOKEN` |
| workspace-agent | `workspace/m365.json` | `@microsoft/workiq` (Microsoft) | none in MCP config — uses cached device-code auth (see mail-agent row) |
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

## How this fits with Microsoft Copilot Cowork, Agent 365, and Claude Cowork

Quick answer: they're different products solving different problems. You don't have to choose.

### What each product does

**Microsoft Copilot Cowork** (Microsoft, Research Preview March 2026; included with M365 Copilot via the Frontier program — no separate add-on price published):
- Runs in the cloud — no desktop app needed
- Powered by Anthropic's Claude (Opus 4.8 rolling out late May 2026; earlier previews used Opus 4.6/Sonnet 4.6)
- Has Work IQ underneath: a semantic index of your org's M365 data, org graph, Dynamics, and external data via federated MCP connectors (GA May 5, 2026)
- Microsoft controls model routing — you don't pick
- Native to M365 ecosystem; 3rd party tools available via MCP connectors (requires admin setup)

**Microsoft Agent 365** (Microsoft, available in M365 E7 bundle at $99/user/month; standalone pricing not publicly listed):
- Not an AI agent itself — it's the **governance and control plane** for AI agents inside your Microsoft tenant
- Lets IT admins observe, govern, and secure both Microsoft and third-party agents
- Relevant for enterprises that need audit trails and policy controls over agent activity
- E7 bundle ($99/user/mo) combines E5 + Copilot + Agent 365 + Entra Suite

**Claude Cowork** (Anthropic, 2026, requires a paid Claude plan):
- Runs on your desktop — your laptop has to be on and the app open
- Computer use: includes Claude-in-Chrome connector for browser navigation, clicking, form-filling; plus desktop app/spreadsheet/file work
- Connectors include Slack and Google Drive (built-in web connectors); custom MCP servers for the rest
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

| | Copilot Cowork (M365) | Agent 365 | Claude Cowork | agent-starter |
|---|---|---|---|---|
| **Runs without a laptop** | ✅ cloud | ✅ cloud (governance layer) | ❌ laptop must stay on | ❌ interactive needs machine on; devops-agent runs headless in GitHub Actions |
| **Works outside Microsoft stack** | ❌ M365 apps only (preview) | ⚠️ tenant-bound; partner agents can integrate external services | ✅ plugins (Slack, Notion, GitHub, Google Drive and more) | ✅ any MCP server |
| **You control model per agent** | ❌ Claude only (currently Opus 4.8; previews used Opus 4.6 / Sonnet 4.6) | ⚠️ model choice available via Copilot Studio | ❌ Claude only | ✅ per-agent in YAML |
| **Platform-enforced tool scoping** | ✅ tenant permissions + governance | ✅ governance layer | ❌ plugin OAuth consent | ✅ `tools:` array |
| **Runs in GitHub Actions** | ❌ | ❌ | ❌ | ⚠️ devops-agent yes; project-agent partially |
| **Can query a database** | ⚠️ M365 data only (Excel, SharePoint) | ⚠️ via connectors/agents | ✅ via Data plugin (Snowflake, BigQuery, Databricks, SQL) | ✅ via MCP |
| **No extra subscription** | ⚠️ included with M365 Copilot via Frontier opt-in | ❌ $99/user/mo E7 bundle (standalone price not published) | ❌ separate paid plan | ✅ uses your GHCP seat |
| **Org semantic index** | ✅ Work IQ | ✅ via Work IQ | ❌ | ❌ |
| **Computer use (apps, browser)** | ❌ | ❌ | ✅ Claude-in-Chrome connector + desktop apps | ❌ |
| **Non-technical user UX** | ✅ knowledge worker UX + IT admin controls | ✅ IT admin UX | ✅ | ❌ CLI only |

### When to use which

**Copilot Cowork** — your org is deep in M365 and you want cloud-native org intelligence (who owns what, collaboration patterns, Dynamics data). Great for knowledge workers who don't want to touch a terminal.

**Agent 365** — your IT/security team needs governance and audit trails over AI agent activity across the tenant. This is an enterprise IT product, not an end-user productivity tool.

**Claude Cowork** — you need Chrome browser automation, desktop-app control, local file work, or recurring tasks that run from a desktop. Strong for non-technical users not on M365.

**agent-starter** — you have GHCP and want domain agents for your actual dev workflow. Especially useful if:
- You're on GitLab, Jira, Snowflake, or anything outside Microsoft's ecosystem
- You want devops-agent running headless in GitHub Actions as a CI diagnostic tool
- You don't want to pay for another platform subscription
- You want tool boundaries enforced at the platform level, not just by prompt

### What it doesn't do

- No org-wide semantic index like Work IQ — it doesn't know your org's collaboration graph
- CLI only — non-technical teammates won't find it friendly
- Mail and workspace agents (M365/Outlook) require a one-time interactive login via `@microsoft/workiq` — see SETUP.md
- Model inference goes through GitHub Copilot's endpoints — it's not air-gapped
- Heavy usage may hit premium request limits on your Copilot plan

### Complementary use example

> 1. **Copilot Cowork** surfaces a risk in the Work IQ org graph
> 2. **workspace-agent** pulls the meeting notes and extracts action items
> 3. **project-agent** creates issues in Jira or GitHub
> 4. **devops-agent** checks if the related CI pipeline is green
>
> Copilot Cowork stays in its lane. agent-starter crosses the boundaries it can't.

## Running devops-agent headless in GitHub Actions

`devops-agent` works without a laptop — no interactive session needed. The included workflow auto-diagnoses failing CI runs and posts the result as a PR comment.

**`.github/workflows/diagnose-failure.yml`** — included in this repo. Triggers on any `workflow_run: failure` event, fetches the failed logs, runs devops-agent to identify root cause and suggest a fix, then posts a comment on the open PR (or writes to the workflow summary if no PR exists).

To use it in your own repo:

```bash
cp .github/workflows/diagnose-failure.yml your-repo/.github/workflows/
cp agents/devops-agent.agent.md your-repo/agents/  # or ~/.copilot/agents/
```

No extra secrets — uses the standard `GITHUB_TOKEN`. Requires a GitHub Copilot seat on the account running the workflow.

The workflow:
1. Installs `gh` CLI and the Copilot extension on the runner
2. Copies `devops-agent` into `~/.copilot/agents/`
3. Fetches the last 200 lines of failed logs via `gh run view --log-failed`
4. Runs `gh copilot -- --agent devops-agent` with the logs and a structured prompt
5. Posts the diagnosis (root cause, why it failed, suggested fix) as a PR comment

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
| Invoked via `/fleet` for parallel review | Invoked via `/agent` or `--agent` flag |
| **Purpose: review code** | **Purpose: do domain work** |

> **Note on `model:`** — honoured in VS Code and interactive CLI. If you're using cloud agent (GitHub Actions), verify it's respected in your GHCP plan.

## Adding your own agents

Copy any `.agent.md`, update the frontmatter and instructions, drop it in `~/.copilot/agents/`. Common additions:
- `salesforce-agent` — CRM queries and record updates
- `confluence-agent` — wiki search and page authoring
- `servicenow-agent` — ITSM ticket management
- `analytics-agent` — Mixpanel / Amplitude / GA queries
- `finance-agent` — expense and budget queries

## Cost

No extra subscription needed — runs on your existing GitHub Copilot plan.

### How billing works (from June 1, 2026)

GitHub Copilot switched to **usage-based billing** on June 1, 2026. Every agent interaction draws from your plan's monthly **AI Credits** (1 credit = $0.01). Code completions stay free; chat, CLI, and agent use draw credits.

| Plan | Monthly cost | AI Credits / month (base + flex) |
|---|---|---|
| Copilot Free | $0 | Very limited — not practical for regular agent use |
| Copilot Pro | $10/mo | 1,500 (1,000 base + 500 flex) |
| Copilot Pro+ | $39/mo | 7,000 (3,900 base + 3,100 flex) |
| Copilot Business | $19/user/mo | 1,900/user |
| Copilot Enterprise | $39/user/mo | 3,900/user |

Credits beyond the included amount are billed at the token rates below. Admins can cap spending to prevent overruns.

### Per-model rates used by these agents

| Model | Used by | Input (per 1M tokens) | Output (per 1M tokens) | Why this model |
|---|---|---|---|---|
| `claude-haiku-4.5` | mail-agent, workspace-agent | $1.00 | $5.00 | Fast, cheap, excellent at tone-matching and summarisation |
| `gpt-4.1` | project-agent, devops-agent | $2.00 | $8.00 | Strong code/YAML understanding; cheaper output than Codex |
| `claude-opus-4.7` | data-agent, research-agent | $5.00 | $25.00 | Complex SQL reasoning and deep research synthesis justify the cost |

### Rough cost per agent task

A typical agent task uses ~5,000 input tokens + ~1,000 output tokens:

| Agent | Model | Estimated cost per task |
|---|---|---|
| mail-agent, workspace-agent | claude-haiku-4.5 | ~$0.01 (~1 credit) |
| project-agent, devops-agent | gpt-4.1 | ~$0.018 (~2 credits) |
| data-agent, research-agent | claude-opus-4.7 | ~$0.05 (~5 credits) |

On **Copilot Pro** (1,000 credits/mo) that's roughly **200–1,000 agent tasks per month** depending on which agents you use. On **Pro+** (3,900 credits), roughly 4× that.

### Reducing cost further

If you're still hitting limits, swap `claude-opus-4.7` to `claude-sonnet-4.6` in `data-agent.agent.md` and `research-agent.agent.md`. Change the `model:` line in the frontmatter. You get ~40% of the cost for comparable quality on most tasks.

Full model rates: [docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)

## References

- [Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [Creating custom agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/create-custom-agents)
- [Automate Copilot CLI with GitHub Actions](https://docs.github.com/en/copilot/how-tos/copilot-cli/automate-copilot-cli/automate-with-actions)
- [copilot-fleet-starter](https://github.com/sethiramicrosoft/copilot-fleet-starter)

## License

MIT. Fork, swap the MCPs, add your own agents.
