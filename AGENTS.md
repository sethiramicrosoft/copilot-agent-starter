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
| mail-agent | `claude-haiku-4.5` | Fast, cheap, excellent at tone-matching and summarisation — email doesn't need heavyweight reasoning |
| project-agent | `gpt-4.1` | Strong code understanding for reading PRs/issues; output at $8/M vs $14/M for Codex — better value for read-heavy tasks |
| data-agent | `claude-opus-4.7` | Complex SQL planning, anomaly detection, multi-table reasoning — justified use of the most capable model |
| devops-agent | `gpt-4.1` | Handles YAML, shell, and pipeline debugging well; Codex specialisation is for generation not diagnosis |
| workspace-agent | `claude-haiku-4.5` | Summarisation and action-item extraction are squarely in Haiku's wheelhouse at a fraction of Sonnet cost |
| research-agent | `claude-opus-4.7` | Deep multi-source synthesis and recommendation — the clearest justification for Opus |

## How to invoke

```
/agent                                                              ← browse and select interactively
Use the data-agent to...                                            ← call by name in prompt
gh copilot -- --agent devops-agent -p "..." --allow-all-urls        ← one-shot CLI
gh copilot                                                          ← interactive session
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
