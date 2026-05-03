# DClaw Flow — Product Specification

> **Status:** Spec v1.0 — Ready for implementation  
> **Priority:** P1  
> **Owner:** Vault Coordinator  
> **Target Implementation:** Shell Agent  
> **ETA:** 2026-06-15 (MVP)  
> **Decision Ref:** DECISIONS.md §2026-05-04: DClaw Flow Orchestration Engine

---

## 1. Overview

**DClaw Flow** is a visual workflow automation platform — the DClaw Stack equivalent of Zapier/Make/n8n. Users build automations by connecting nodes on a canvas. Flows run reliably on Temporal.io with full observability.

| Attribute | Value |
|-----------|-------|
| **App ID** | `flow` |
| **Display Name** | DClaw Flow |
| **Category** | Automation |
| **Tagline** | "Connect anything, automate everything" |
| **Color** | `#10B981` (Green) |
| **Icon** | 🌊 |
| **Domain** | flow.dclawstack.io |
| **Competitors** | Zapier, Make, n8n |

---

## 2. Architecture

### 2.1 High-Level Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         DClaw Flow                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Frontend (Next.js 16 + React Flow)                         │   │
│  │  • Visual node editor canvas                                │   │
│  │  • Node palette (triggers, actions, conditionals)           │   │
│  │  • Workflow list / run history                              │   │
│  │  • Real-time execution status (SSE)                         │   │
│  └────────────────────────┬────────────────────────────────────┘   │
│                           │ REST API + WebSocket/SSE                │
│  ┌────────────────────────▼────────────────────────────────────┐   │
│  │  Backend (FastAPI)                                          │   │
│  │  • Workflow CRUD (validate DAG, save state)                 │   │
│  │  • Execution API (start, pause, cancel)                     │   │
│  │  • Integration registry (metadata, credentials)             │   │
│  │  • Webhook receiver (trigger ingestion)                     │   │
│  │  • Temporal client (submit workflows, query status)         │   │
│  └────────────────────────┬────────────────────────────────────┘   │
│                           │ gRPC                                    │
│  ┌────────────────────────▼────────────────────────────────────┐   │
│  │  Temporal Cluster (workers + server)                        │   │
│  │  • Workflow definitions (Go + Temporal SDK)                 │   │
│  │  • Activity implementations (HTTP, DB, integrations)        │   │
│  │  • Durable execution, retries, sagas                        │   │
│  └────────────────────────┬────────────────────────────────────┘   │
│                           │ SQL / gRPC                              │
│  ┌────────────────────────▼────────────────────────────────────┐   │
│  │  Data Layer                                                 │   │
│  │  • PostgreSQL — workflow defs, executions, audit logs       │   │
│  │  • Redis — webhook queue, rate limiting, cache              │   │
│  │  • MinIO — execution artifacts (large payloads, files)      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Per-App Stack

```
dclaw-flow/
├── frontend/               → Next.js 16, React Flow, Tailwind, shadcn/ui
├── backend/                → FastAPI, Pydantic v2, SQLAlchemy 2.0, Temporal SDK (Python)
├── worker/                 → Temporal worker (Go or Python — see §5.1)
├── integrations/           → Integration node definitions (YAML + Python/Go handlers)
├── helm/                   → K8s manifests, Temporal server dependency
└── tauri/                  → Desktop shell (optional for P1)
```

---

## 3. Data Model

### 3.1 Core Entities

#### Workflow

```python
class Workflow(BaseModel):
    id: UUID
    name: str                          # e.g., "Slack → Chat Summary"
    description: str | None
    owner_id: UUID                     # DClaw user ID
    team_id: UUID | None               # For shared workspaces
    status: Literal["draft", "active", "paused", "archived"]
    
    # Canvas state
    nodes: list[Node]
    edges: list[Edge]
    
    # Execution config
    trigger: TriggerConfig
    error_handling: ErrorHandlingConfig = ErrorHandlingConfig()
    
    # Metadata
    version: int = 1                   # Increment on every save
    last_executed_at: datetime | None
    execution_count: int = 0
    
    created_at: datetime
    updated_at: datetime
```

#### Node

```python
class Node(BaseModel):
    id: str                            # React Flow node ID (e.g., "node-1")
    type: Literal[
        "trigger", "action", "conditional", 
        "loop", "delay", "merge", "transform"
    ]
    integration_id: str | None         # e.g., "slack", "http", "chat"
    action_id: str | None              # e.g., "slack.post_message"
    
    # Positioning (React Flow)
    position: dict[str, float]         # { x: 100, y: 200 }
    width: float | None
    height: float | None
    
    # Configuration
    config: dict[str, Any]             # Node-specific parameters
    label: str | None                  # Display label on canvas
    
    # Execution
    timeout_seconds: int = 30
    retry_policy: RetryPolicy = RetryPolicy()
```

#### Edge

```python
class Edge(BaseModel):
    id: str
    source: str                        # Source node ID
    target: str                        # Target node ID
    source_handle: str | None          # Output port (for conditional branches)
    target_handle: str | None          # Input port
    
    # Conditional routing
    condition: Condition | None        # e.g., "status == 200"
    label: str | None                  # "true", "false", "success"
```

#### Execution (Run)

```python
class Execution(BaseModel):
    id: UUID
    workflow_id: UUID
    status: Literal[
        "pending", "running", "completed", 
        "failed", "cancelled", "timed_out"
    ]
    
    # Temporal mapping
    temporal_workflow_id: str
    temporal_run_id: str | None
    
    # Trigger context
    trigger_source: str                # "webhook", "schedule", "manual", "api"
    trigger_payload: dict[str, Any] | None
    
    # Results
    started_at: datetime
    completed_at: datetime | None
    duration_ms: int | None
    
    # Node-level results
    node_results: list[NodeExecution]
    
    # Error
    error: ExecutionError | None
```

#### NodeExecution

```python
class NodeExecution(BaseModel):
    node_id: str
    status: Literal["pending", "running", "completed", "failed", "skipped"]
    started_at: datetime | None
    completed_at: datetime | None
    duration_ms: int | None
    
    input: dict[str, Any] | None       # Resolved inputs (after variable substitution)
    output: dict[str, Any] | None      # Node output (available to downstream nodes)
    error: ExecutionError | None
    
    retry_count: int = 0
    logs: list[str] = []
```

### 3.2 Database Schema (PostgreSQL)

```sql
-- Workflows
create table workflows (
    id uuid primary key default gen_random_uuid(),
    name text not null,
    description text,
    owner_id uuid not null,
    team_id uuid,
    status text not null default 'draft',
    nodes jsonb not null default '[]',
    edges jsonb not null default '[]',
    trigger jsonb not null default '{}',
    error_handling jsonb not null default '{}',
    version int not null default 1,
    last_executed_at timestamptz,
    execution_count int not null default 0,
    created_at timestamptz not null default now(),
    updated_at timestamptz not null default now()
);

-- Executions
create table executions (
    id uuid primary key default gen_random_uuid(),
    workflow_id uuid not null references workflows(id),
    status text not null default 'pending',
    temporal_workflow_id text not null,
    temporal_run_id text,
    trigger_source text not null,
    trigger_payload jsonb,
    started_at timestamptz not null default now(),
    completed_at timestamptz,
    duration_ms int,
    error jsonb,
    created_at timestamptz not null default now()
);

-- Node executions (child of execution)
create table node_executions (
    id uuid primary key default gen_random_uuid(),
    execution_id uuid not null references executions(id),
    node_id text not null,
    status text not null default 'pending',
    started_at timestamptz,
    completed_at timestamptz,
    duration_ms int,
    input jsonb,
    output jsonb,
    error jsonb,
    retry_count int not null default 0,
    logs jsonb not null default '[]'
);

-- Integration credentials (encrypted at rest)
create table integration_credentials (
    id uuid primary key default gen_random_uuid(),
    integration_id text not null,
    owner_id uuid not null,
    display_name text not null,
    encrypted_config bytea not null,    -- AES-256-GCM encrypted
    created_at timestamptz not null default now(),
    updated_at timestamptz not null default now()
);

-- Indexes
create index idx_executions_workflow on executions(workflow_id, created_at desc);
create index idx_executions_status on executions(status) where status in ('running', 'pending');
create index idx_node_executions_execution on node_executions(execution_id);
```

---

## 4. API Contracts

### 4.1 Workflow CRUD

```yaml
# POST /api/v1/flows/workflows
# Create or update a workflow
Request:
  name: string
  description?: string
  nodes: Node[]
  edges: Edge[]
  trigger: TriggerConfig
  error_handling?: ErrorHandlingConfig

Response:
  id: UUID
  version: int
  status: "draft" | "active" | "paused"
  created_at: datetime
  updated_at: datetime

# GET /api/v1/flows/workflows/{id}
# GET /api/v1/flows/workflows
# DELETE /api/v1/flows/workflows/{id}

# POST /api/v1/flows/workflows/{id}/validate
# Validate DAG (no cycles, all nodes reachable, required configs present)
Response:
  valid: bool
  errors: ValidationError[]
```

### 4.2 Execution

```yaml
# POST /api/v1/flows/workflows/{id}/execute
# Trigger a manual execution
Request:
  payload?: dict              # Optional override for trigger payload
  wait_for_completion?: bool  # Default false (async)

Response:
  execution_id: UUID
  temporal_workflow_id: string
  status: "pending" | "running"

# GET /api/v1/flows/executions/{id}
Response:
  id: UUID
  status: ExecutionStatus
  node_results: NodeExecution[]
  logs: ExecutionLog[]

# POST /api/v1/flows/executions/{id}/cancel
# Signal Temporal to cancel

# GET /api/v1/flows/executions?workflow_id={id}&limit=50
```

### 4.3 Real-Time Status (SSE)

```yaml
# GET /api/v1/flows/executions/{id}/stream
# Server-Sent Events stream of execution updates
Event: node_started
Data: { node_id, timestamp }

Event: node_completed
Data: { node_id, output, duration_ms }

Event: node_failed
Data: { node_id, error, retry_count }

Event: execution_completed
Data: { status, duration_ms }
```

### 4.4 Webhook Triggers

```yaml
# POST /api/v1/flows/webhooks/{webhook_id}
# Public webhook endpoint (HMAC-verified)
Headers:
  X-Flow-Signature: sha256=<hmac>

# Returns 202 Accepted immediately, executes asynchronously
```

---

## 5. Temporal Workflows

### 5.1 Worker Language Decision

**Decision:** Use **Python** for the Temporal worker in P1. Rationale:
- FastAPI backend is Python — shared Pydantic models, no serialization boundary
- Integration handlers are I/O-bound (HTTP requests) — Python asyncio is sufficient
- Go worker can be introduced later for high-throughput paths

**Revisit condition:** >100 workflows/sec sustained throughput.

### 5.2 Workflow Definition

```python
# temporal/workflows/flow_workflow.py
from temporalio import workflow
from temporalio.common import RetryPolicy

with workflow.unsafe.imports_passed_through():
    from activities import execute_node_activity

@workflow.defn
class FlowWorkflow:
    @workflow.run
    async def run(self, workflow_def: WorkflowDef, trigger_payload: dict) -> ExecutionResult:
        """Execute a DClaw Flow workflow definition."""
        # Build adjacency list from edges
        graph = build_graph(workflow_def.edges)
        
        # Start with trigger node(s)
        queue = get_trigger_nodes(workflow_def.nodes)
        results = {}
        
        while queue:
            node_id = queue.pop(0)
            node = get_node(workflow_def.nodes, node_id)
            
            # Resolve inputs from upstream node outputs
            inputs = resolve_inputs(node, results, trigger_payload)
            
            # Execute node activity
            result = await workflow.execute_activity(
                execute_node_activity,
                args=(node, inputs),
                start_to_close_timeout=timedelta(seconds=node.timeout_seconds),
                retry_policy=RetryPolicy(
                    maximum_attempts=node.retry_policy.max_attempts,
                    initial_interval=timedelta(seconds=node.retry_policy.initial_interval),
                ),
            )
            
            results[node_id] = result
            
            # Queue downstream nodes
            for next_node_id in get_downstream(graph, node_id, result):
                if all_deps_satisfied(next_node_id, graph, results):
                    queue.append(next_node_id)
        
        return ExecutionResult(node_results=results)
```

### 5.3 Activity Definition

```python
# temporal/activities/node_activity.py
from temporalio import activity

@activity.defn
async def execute_node_activity(node: Node, inputs: dict) -> NodeResult:
    """Execute a single flow node."""
    integration = get_integration(node.integration_id)
    
    try:
        output = await integration.execute(node.action_id, node.config, inputs)
        return NodeResult(status="completed", output=output)
    except Exception as e:
        return NodeResult(status="failed", error=str(e))
```

---

## 6. Integration Model

### 6.1 Tier 1 (Launch — built in-house)

| Integration | Triggers | Actions |
|-------------|----------|---------|
| **HTTP Webhook** | Receive POST/GET | Send HTTP request |
| **DClaw Chat** | New message, mention | Send message, create conversation |
| **DClaw RAG** | — | Query knowledge base, ingest document |
| **Slack** | New message, reaction | Post message, create channel |
| **Email (SMTP)** | Receive email (IMAP) | Send email |
| **Telegram** | New message in channel | Send message |
| **Schedule** | Cron trigger | — |
| **Filter** | — | Conditional logic, data transform |

### 6.2 Integration Definition Format

```yaml
# integrations/slack.yaml
integration:
  id: slack
  name: Slack
  icon: 🔷
  color: "#4A154B"
  
  auth:
    type: oauth2
    scopes: ["chat:write", "channels:read"]
    
  triggers:
    - id: new_message
      name: New Message
      description: Triggers when a message is posted
      config:
        - name: channel_id
          type: string
          required: true
      output_schema:
        text: string
        user: string
        timestamp: string
        
  actions:
    - id: post_message
      name: Post Message
      description: Sends a message to a channel
      config:
        - name: channel_id
          type: string
          required: true
        - name: text
          type: string
          required: true
      input_schema:
        text: string
      output_schema:
        ok: bool
        ts: string
```

### 6.3 Credential Encryption

- All integration credentials encrypted at rest with **AES-256-GCM**
- Encryption key stored in K8s Secret, mounted to backend pods
- Credentials never logged or returned to frontend
- In-memory decryption only during activity execution

---

## 7. Security & Compliance

| Concern | Mitigation |
|---------|------------|
| Webhook spoofing | HMAC-SHA256 signature verification on all webhooks |
| Credential exposure | AES-256-GCM at rest, no frontend access, audit logging |
| Workflow injection | Strict input validation on all node configs; no raw code execution |
| Resource exhaustion | Per-user workflow limits (Free: 5, Pro: 50, Team: unlimited); per-node timeout (max 5 min) |
| SSRF via HTTP node | URL allowlist/blocklist; private IP ranges denied by default |
| PII in execution logs | ClawShield scan on all payloads before persistence |

---

## 8. UI/UX Specification

### 8.1 Canvas Requirements

- **Drag-and-drop** node creation from left sidebar palette
- **Pan/zoom** with mouse wheel and trackpad gestures
- **Auto-layout** button (hierarchical layout algorithm)
- **Mini-map** in bottom-right corner
- **Node search** (Cmd+K) to find and add nodes
- **Undo/redo** stack (Cmd+Z / Cmd+Shift+Z)
- **Validation indicators** — red borders on invalid nodes, green on valid

### 8.2 Key Screens

1. **Workflow List** — grid of cards with status, last run, execution count
2. **Canvas Editor** — React Flow canvas with node palette, property panel
3. **Execution History** — table of runs with status filters, drill-down to node-level logs
4. **Integration Settings** — connect OAuth apps, manage credentials
5. **Webhook Management** — copy webhook URLs, regenerate secrets, view delivery logs

### 8.3 Variable Templating

Users reference upstream node outputs via `{{ node_id.output_key }}` syntax.

Example:
```
{{ trigger.text }}               → Slack message text
{{ trigger.user }}               → Slack user ID
{{ summarize.output.summary }}   → Output from a "Summarize" node
```

Auto-complete dropdown appears when user types `{{`.

---

## 9. Acceptance Criteria

### MVP (P1 — June 2026)

- [ ] User can create a workflow with 2+ nodes on a canvas
- [ ] User can connect nodes with edges (visual + functional)
- [ ] User can configure node parameters in a side panel
- [ ] User can save and activate a workflow
- [ ] Workflow can be triggered manually via "Run" button
- [ ] HTTP Webhook trigger works (receive POST, execute flow)
- [ ] HTTP Request action works (send GET/POST)
- [ ] Execution history shows run status and node-level results
- [ ] Temporal worker executes workflows durably (survives pod restart)
- [ ] Helm chart deploys Flow to K8s with Temporal dependency

### v1.1 (P1+ — July 2026)

- [ ] Schedule trigger (cron expressions)
- [ ] Slack integration (OAuth, post message)
- [ ] Conditional node (if/else branching)
- [ ] Variable templating with auto-complete
- [ ] Real-time execution status via SSE
- [ ] Workflow versioning (rollback to previous version)

### v1.2 (P2 — August 2026)

- [ ] DClaw Chat integration
- [ ] DClaw RAG integration
- [ ] Loop node (iterate over array)
- [ ] Error handling (fallback branches, retry config)
- [ ] Team sharing (workspaces, permissions)

---

## 10. Open Questions

| Question | Impact | Recommendation |
|----------|--------|----------------|
| Temporal server self-hosted vs Temporal Cloud? | Cost, ops overhead | Self-hosted in K8s for local-first principle; evaluate Cloud for enterprise |
| Python vs Go worker for P1? | Development velocity vs performance | Python for P1 (see §5.1); benchmark and revisit |
| Should Flow nodes execute inside the DClaw Operator? | Architecture consistency | No — Flow is a product, not platform infra. Keep separate. |

---

## 11. Handoff

## Handoff: Vault → Shell

- **Feature:** DClaw Flow MVP
- **Spec:** `dclaw-prd/FLOW.md` (this document)
- **Acceptance Criteria:** See §9 (MVP checklist)
- **Repos affected:**
  - `dclawstack/dclaw-flow` — new repo to scaffold
  - `dclawstack/dclaw-platform` — may need Operator changes for Flow app provisioning
- **Estimated complexity:** High
- **Priority:** P1
- **Decision ref:** DECISIONS.md §2026-05-04 (Flow Engine, Node Model, Integration Strategy)
- **Blockers:** None
- **Suggested order of implementation:**
  1. Scaffold `dclaw-flow` repo (Next.js + FastAPI + Temporal SDK)
  2. Database schema + SQLAlchemy models
  3. Workflow CRUD API + DAG validation
  4. Temporal workflow + activity skeleton
  5. React Flow canvas + node palette
  6. HTTP Webhook trigger + HTTP Request action
  7. Execution history UI
  8. Helm chart
  9. Integration with DPanel app store
