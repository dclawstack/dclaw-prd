# AGENTS

## Agent Swarm Protocol

> For the complete topology, protocols, and spawn procedures, see `dclaw-platform/AGENT_SWARM.md`.  
> This document summarizes the sacred-context-relevant details.

---

### Roles

| Agent | Domain | Primary Repo | Priority |
|-------|--------|-------------|----------|
| **Vault Coordinator** | Architecture, specs, PRDs, decision logs | `dclaw-prd` | P0 |
| **Shell Agent** | Implementation, builds, CI/CD, deploy | `dclaw-platform`, `dclaw-chat` | P0 |
| **Shield Agent** | Security, compliance, audits, reviews | All repos (review-only) | P1 |
| **Code Agent** | Deep engineering, refactoring, algorithms | Any repo (assigned) | P1 |
| **Research Agent** | R&D, prototyping, benchmarking, PoCs | `dclaw-prd` (research docs) | P2 |
| **Memory Agent** | Docs, knowledge graphs, weekly synthesis | `dclaw-prd`, `dclaw-obsidian` | P2 |
| **General Agent** | User intake, routing, orchestration | None (orchestrates others) | P0 |

### Ownership Rules

1. **No cross-agent edits** — If Agent A needs a change in Agent B's domain, write a TODO comment or open an issue, don't modify the file.
2. **API contract is sacred** — Backend owns the API schema. Frontend consumes it. Never break without both agents agreeing.
3. **Branch per agent** — `feat/{agent-id}/description`
4. **Main is protected** — All changes via PR, even for single-agent work.
5. **GitHub as shared memory** — Agents do not talk to each other in real-time. Commits, PRs, and issues are the only coordination channel.

### Handoff Format

```markdown
## Handoff: Vault → Shell

- **Feature:** [name]
- **Spec:** [link to dclaw-prd doc or section]
- **Acceptance Criteria:**
  1. [criterion]
  2. [criterion]
- **Repos affected:** [list]
- **Estimated complexity:** [low/medium/high]
- **Priority:** [P0/P1/P2]
- **Decision ref:** [link to DECISIONS.md entry]
```

### Commit Prefix Convention

Every commit must be prefixed with the agent role:

```
[agent:vault] docs(arch): add dpanel-api sequence diagram
[agent:shell] feat(dpanel): add install/uninstall UI
[agent:shield] audit(chat): review PII handling in chat.py
[agent:code] refactor(operator): extract reconciler helpers
[agent:research] spike(rag): evaluate pgvector vs milvus
[agent:memory] docs(kg): update component dependency graph
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
4. https://github.com/dclawstack/dclaw-platform/blob/main/AGENT_SWARM.md

Your domain: {domain}
Your repos: {repos}
Current focus: {focus}

Ask clarifying questions before starting.
```
