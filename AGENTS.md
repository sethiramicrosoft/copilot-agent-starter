# Agent fleet

Domain agents live in `~/.copilot/agents/*.agent.md` machine-wide and in `.github/agents/*.agent.md` per repo.

## Cast (with default backend)

- `mail-agent` — email, calendar, inbox triage. **Pre-wired**: Microsoft 365 (Outlook) via `@microsoft/workiq`. Alternative: Google Workspace (Gmail) via community `workspace-mcp` — see `mcp-examples/mail/`.
- `project-agent` — issues, PRs, tasks, releases. **Pre-wired**: GitHub via Copilot CLI's built-in `github/*` MCP (no setup). Alternatives for GitLab, Jira in `mcp-examples/vcs/`.
- `data-agent` — SQL queries, file analysis, data summaries. **Not pre-wired**: DB path/creds vary per user — see `mcp-examples/database/` for SQLite, Postgres, MS SQL, Snowflake.
- `devops-agent` — CI/CD, pipelines, Dockerfiles. **Pre-wired**: GitHub Actions via built-in `github/*` MCP + shell. Alternatives for GitLab CI, Azure DevOps in `mcp-examples/cicd/`.
- `workspace-agent` — team collaboration, notes, documents. **Pre-wired**: Microsoft 365 (Teams, SharePoint, OneDrive) via `@microsoft/workiq`. Alternative: Google Workspace (Drive, Docs, Sheets, Calendar, Chat) via community `workspace-mcp`; Slack and Notion configs also in `mcp-examples/workspace/`.
- `research-agent` — web + code deep research, cited reports. **Pre-wired**: built-in web search + GitHub search (no MCP needed).

## Model assignment

| Agent | Model | Why |
|---|---|---|
| mail-agent | `claude-haiku-4.5` | Fast, cheap, excellent at tone-matching and summarisation — email doesn't need heavyweight reasoning |
| project-agent | `gpt-4.1` | Strong code understanding for reading PRs/issues; output at $8/M vs $14/M for Codex — better value for read-heavy tasks |
| data-agent | `claude-opus-4.7` | Complex SQL planning, anomaly detection, multi-table reasoning — justified use of the most capable model |
| devops-agent | `gpt-4.1` | Handles YAML, shell, and pipeline debugging well; Codex specialisation is for generation not diagnosis |
| workspace-agent | `claude-haiku-4.5` | Summarisation and action-item extraction are squarely in Haiku's wheelhouse at a fraction of Sonnet cost |
| research-agent | `claude-opus-4.7` | Deep multi-source synthesis and recommendation — the clearest justification for Opus |

## How to invoke

```
/agent                                                          ← interactive: browse and select
Use the data-agent to...                                        ← interactive: call by name in prompt
copilot --agent devops-agent -p "..." --allow-all               ← one-shot CLI
copilot                                                         ← start interactive session
copilot --resume                                                ← pick a past session to continue
```

These same agent files also work in **VS Code Copilot Chat** — VS Code reads from the same `~/.copilot/agents/` directory ([docs](https://code.visualstudio.com/docs/copilot/customization/custom-agents#_custom-agent-file-locations)).

## Agent boundaries (hard rules)

- Each agent only uses the MCPs and tools listed in its own profile.
- Agents do not merge code, delete data, or send messages without human confirmation.
- If a task spans two agents, do them sequentially — one agent hands off to the next.
- Lessons learned go into the agent's own `## Lessons` section before the session ends.

## Relationship to copilot-fleet-starter

This repo is the **domain execution layer**. The persona fleet in `copilot-fleet-starter` is the **code review layer**.

- Use **agents** to *do* something (query data, manage tickets, summarise meetings).
- Use **personas** via `/fleet` to *review* something (security, performance, UX, accessibility).
