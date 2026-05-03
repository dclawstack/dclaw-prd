# AGENTS

## Agent Swarm Protocol

### Roles

| Agent | Domain | Repos | Priority |
|-------|--------|-------|----------|
| **Build Agent** (you) | Code, build, test, deploy | All | P0 |
| **Vault Coordinator** | Docs, specs, tracking | dclaw-obsidian | P0 |
| **Frontend Agent** | Next.js, React, Tailwind | `*/components/`, `*/app/` | P1 |
| **Backend Agent** | FastAPI, DB, APIs | `*/backend/`, `*/routers/` | P1 |
| **Desktop Agent** | Tauri, native features | `*/src-tauri/` | P2 |
| **DevOps Agent** | K8s, Helm, CI/CD | `*/helm/`, `.github/` | P2 |
| **Shield Agent** | PII, security, compliance | `*/lib/swarm/agents/shield*` | P1 |

### Ownership Rules

1. **No cross-agent edits** — If Agent A needs a change in Agent B's domain, write a TODO comment, don't modify the file.
2. **API contract is sacred** — Backend owns the API schema. Frontend consumes it. Never break without both agents agreeing.
3. **Branch per agent** — `feat/{agent-id}/description`
4. **Main is protected** — All changes via PR, even for single-agent work.

### Handoff Format

```python
# TODO[BackendAgent]: Implement POST /api/v1/export/pdf
# Contract: Accepts { conversation_id, format: "pdf" }
# Returns: { download_url, expires_at }
# Blocker: Need PDF generation library (weasyprint or playwright)
# Target: 2026-05-06
```

### Daily Standup Format

Each agent updates the vault daily:

```markdown
## Agent: {name}
### Yesterday
- [x] Completed item
### Today
- [ ] Planned item
### Blockers
- Waiting on X from Agent Y
```

### Communication Channels

| Channel | Use For |
|---------|---------|
| Obsidian Vault | Specs, decisions, progress tracking |
| GitHub PRs | Code review, merge decisions |
| Telegram Bot | CI/CD notifications, alerts |
| Kimi Web Chat | High-level planning, architecture review |
| Kimi CLI | Implementation, build, deploy |

## Seeding a New Agent

To bring a new agent up to speed, paste this into their chat:

```
You are a {role} agent for the DClaw Platform.

Read these docs before writing any code:
1. https://github.com/dclawstack/dclaw-prd/blob/main/ARCHITECTURE.md
2. https://github.com/dclawstack/dclaw-prd/blob/main/CONVENTIONS.md
3. https://github.com/dclawstack/dclaw-prd/blob/main/STATUS.md

Your domain: {domain}
Your repos: {repos}
Current focus: {focus}

Ask clarifying questions before starting.
```
