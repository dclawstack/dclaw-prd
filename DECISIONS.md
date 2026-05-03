# Decision Log

> Architecture and product decisions for the DClaw Stack.  
> Format: Context → Decision → Alternatives → Owner → Status  
> Maintained by: Vault Coordinator

---

## 2026-05-03: Agent Swarm Topology

- **Context:** Multiple Kimi web chat sessions need to coordinate on a single codebase without real-time communication between agents.
- **Decision:** Seven-role topology with GitHub as the sole shared memory layer. Agents communicate exclusively through commits, PRs, and issues.
- **Alternatives considered:**
  - Real-time chat between agents (rejected: no API available, too complex)
  - Single-agent monolith (rejected: context window limits, no specialization)
  - Shared database state (rejected: too fragile, git history is authoritative)
- **Owner:** @vault-coordinator
- **Status:** Adopted — documented in `dclaw-platform/AGENT_SWARM.md`

---

## 2026-05-03: DPanel API Read Strategy

- **Context:** DPanel needs live app data from the K8s cluster to render the app store and launcher.
- **Decision:** Use a dedicated `dpanel-api` Go service reading ConfigMap `dclaw-core/dclaw-apps-registry`. The Operator writes app state to this ConfigMap; `dpanel-api` reads it and exposes a REST API.
- **Alternatives considered:**
  - Direct CRD reads from DPanel frontend (rejected: CORS/auth complexity, exposes K8s API to browser)
  - Operator exposes HTTP endpoint directly (rejected: violates controller pattern, security risk)
  - WebSocket push from Operator (rejected: overkill for current scale, adds complexity)
- **Owner:** @shell-agent
- **Status:** Implemented — `dpanel-api` compiles, Dockerfile ready, needs K8s deployment

---

## 2026-05-04: DClaw Flow Orchestration Engine

- **Context:** DClaw Flow (Product #2) needs reliable execution of user-defined workflows with retries, timeouts, and observability. Competitors (Zapier, Make, n8n) all use custom execution engines.
- **Decision:** Adopt **Temporal.io** as the workflow execution engine. Temporal provides durable execution, exactly-once semantics, built-in retries, and visibility into running workflows.
- **Alternatives considered:**
  - Custom execution engine in Go (rejected: 6+ months to reach Temporal's reliability)
  - Apache Airflow (rejected: Python-centric, poor fit for our Go/TS stack)
  - n8n core (rejected: AGPL license contamination risk)
  - AWS Step Functions (rejected: cloud-only, violates local-first principle)
- **Owner:** @vault-coordinator
- **Status:** Decided — spec written, pending Shell Agent implementation

---

## 2026-05-04: DClaw Flow Node Model

- **Context:** Flow needs a visual node editor where users connect triggers, actions, and conditionals.
- **Decision:** Use **React Flow** (`@xyflow/react`) for the canvas engine. It is MIT-licensed, widely adopted, and integrates cleanly with Next.js. Custom node types per integration.
- **Alternatives considered:**
  - Rete.js (rejected: smaller community, v2 breaking changes)
  - Custom Canvas API (rejected: 3+ months for basic interactions)
  - LogicFlow (rejected: Chinese docs, smaller ecosystem)
- **Owner:** @vault-coordinator
- **Status:** Decided — spec written, pending Shell Agent implementation

---

## 2026-05-04: DClaw Flow Integration Strategy

- **Context:** Flow must ship with meaningful integrations at launch. Building 100+ integrations from scratch is infeasible by June.
- **Decision:** Tiered integration strategy:
  - **Tier 1 (Launch):** HTTP Webhook, DClaw Chat, DClaw RAG, Slack, Email (SMTP), Telegram — built in-house
  - **Tier 2 (P1):** GitHub, Notion, Airtable, Google Sheets, Stripe — community PRs or Shell Agent sprints
  - **Tier 3 (P2):** 100+ integrations via **n8n-compatible community nodes** (MIT-licensed only, no AGPL)
- **Alternatives considered:**
  - All integrations in-house (rejected: blocks launch by 3+ months)
  - Partner with integration platform (rejected: vendor lock-in, cost)
  - AGPL n8n nodes (rejected: license contamination risk for enterprise customers)
- **Owner:** @vault-coordinator
- **Status:** Decided — spec written

---

*Last updated: 2026-05-04 by Vault Coordinator*
