# Agent fleet

Domain agents live in `~/.copilot/agents/*.agent.md` machine-wide and in `.github/agents/*.agent.md` per repo.

## Cast

- `mail-agent` — email, calendar, inbox triage (any mail provider)
- `project-agent` — issues, PRs, tasks, releases (GitHub, GitLab, Jira, Azure DevOps)
- `data-agent` — SQL queries, file analysis, data summaries (any database)
- `devops-agent` — CI/CD, pipelines, Dockerfiles (any CI/CD tool, shell fallback)
- `workspace-agent` — team collaboration, notes, documents (M365, Slack, Notion, Confluence)
- `research-agent` — web + code deep research, cited reports

## Model assignment

| Agent | Model | Why |
|---|---|---|
| mail-agent | `claude-sonnet-4.6` | Fast, good at tone-matching and summarisation |
| project-agent | `gpt-5.3-codex` | Strong on code-adjacent project tasks |
| data-agent | `claude-opus-4.7` | Complex reasoning for query planning and anomaly detection |
| devops-agent | `gpt-5.3-codex` | Sharp on YAML, shell, and pipeline patterns |
| workspace-agent | `claude-sonnet-4.6` | Balanced speed and reasoning for collaboration workflows |
| research-agent | `claude-opus-4.7` | Deepest reasoning for synthesis and recommendation |

## How to invoke

```
/agent                    ← browse and select interactively
Use the data-agent to...  ← call by name in prompt
copilot --agent=devops-agent --prompt "..."  ← CLI flag
```

## Agent boundaries (hard rules)

- Each agent only uses the MCPs and tools listed in its own profile.
- Agents do not merge code, delete data, or send messages without human confirmation.
- If a task spans two agents, do them sequentially — one agent hands off to the next.
- Lessons learned go into the agent's own `## Lessons` section before the session ends.

## Relationship to copilot-fleet-starter

This repo is the **domain execution layer**. The persona fleet in `copilot-fleet-starter` is the **code review layer**.

- Use **agents** to *do* something (query data, manage tickets, summarise meetings).
- Use **personas** via `/fleet` to *review* something (security, performance, UX, accessibility).
