# Agent fleet

Domain agents live in `~/.copilot/agents/*.agent.md` machine-wide and in `.github/agents/*.agent.md` per repo.

## Cast

- `mail-agent` — email, calendar, inbox triage (Outlook/Gmail MCP)
- `github-agent` — issues, PRs, releases, project management
- `data-agent` — SQL queries, file analysis, data summaries
- `devops-agent` — CI/CD, GitHub Actions, Dockerfiles, deployments
- `m365-agent` — Teams, SharePoint, OneDrive, Planner (Graph MCP)
- `research-agent` — web + GitHub deep research, cited reports

## Model assignment

| Agent | Model | Why |
|---|---|---|
| mail-agent | `claude-sonnet-4.6` | Fast, good at tone-matching and summarisation |
| github-agent | `gpt-5.3-codex` | Strong on code-adjacent GitHub tasks |
| data-agent | `claude-opus-4.7` | Complex reasoning for query planning and anomaly detection |
| devops-agent | `gpt-5.3-codex` | Sharp on YAML, shell, and pipeline patterns |
| m365-agent | `claude-sonnet-4.6` | Balanced speed and reasoning for M365 workflows |
| research-agent | `claude-opus-4.7` | Deepest reasoning for synthesis and recommendation |

## How to invoke

**Interactive:**
```
/agent
```
Select from the list, or call by name:
```
Use the m365-agent to collect action items from the #project-alpha channel this week
```

**CLI flag:**
```bash
copilot --agent=m365-agent --prompt "Summarise Teams channel #general from this week"
```

## Agent boundaries (hard rules)

- Each agent only uses the MCPs and tools listed in its own profile — no cross-agent tool use.
- Agents do not merge code, delete data, or send messages without human confirmation.
- If a task spans two agents (e.g. research + write a GitHub issue), do them sequentially: research first, then github-agent.
- Lessons learned go into the agent's own `## Lessons` section before the session ends.

## Relationship to copilot-fleet-starter

This repo is the **domain execution layer**. The persona fleet in `copilot-fleet-starter` is the **code review layer**. They run on the same CLI and complement each other:

- Use **agents** when you want to *do* something (send mail, query data, manage GitHub, research a topic).
- Use **personas** via `/fleet` when you want to *review* something (security, performance, UX, accessibility).
