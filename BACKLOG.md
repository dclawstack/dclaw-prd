# Vault Coordinator Backlog

> Active tasks and priority queue for the Vault Coordinator agent.  
> Updated after every vault session.

---

## Completed (Archive)

- [x] **2026-05-04:** Review and approve agent swarm backbone
  - Updated `AGENTS.md` to align with new 7-role topology
  - Updated `STATUS.md` with new agent names and current state
  - Created `DECISIONS.md` with historical + new decisions

- [x] **2026-05-04:** Plan DClaw Flow spec
  - Created `FLOW.md` v1.0 with full architecture, data model, API contracts, acceptance criteria
  - Logged Flow decisions in `DECISIONS.md` (orchestration engine, node model, integration strategy)

---

## Current Priority Queue

1. **Define dpanel-api → Operator → DPanel full data flow spec**
   - Status: Pending
   - Context: DPanel needs live app data; `dpanel-api` is scaffolded but not wired
   - Deliverable: Sequence diagram + API contract doc

2. **Complete CloudNativePG integration spec for operator**
   - Status: Pending
   - Context: Operator `reconcileDatabase` is placeholder (logs only)
   - Deliverable: CRD extension for database specs, reconciliation flow

3. **Define Tauri signing strategy (Apple cert vs ad-hoc vs notarize)**
   - Status: Pending
   - Context: Deferred to v1.1, but strategy should be documented
   - Deliverable: Decision in DECISIONS.md with cost analysis

4. **Plan next 3 products after Chat**
   - DClaw Flow — **SPEC DONE**, handoff to Shell Agent
   - DClaw RAG — queued for spec (P1)
   - DClaw Med — queued for spec (P2)

5. **Review DPanel API spec once Shell Agent drafts it**
   - Status: Blocked (waiting on Shell Agent)

---

## Notes for Shell Agent

- Update `dclaw-platform/agents/VAULT_COORDINATOR.md` priority queue to match this backlog
- Update `dclaw-platform/AGENT_SWARM.md` §6 (Current Project State) after Flow work begins
- Create `dclawstack/dclaw-flow` repo when ready to implement

---

*Last updated: 2026-05-04 by Vault Coordinator*
