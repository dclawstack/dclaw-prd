---
tags: [meta, prd, master, swarm, blueprint]
version: 2.0
date: 2026-05-04
---

# 📘 DClaw Master PRD

> **The single document every agent must read before writing code.**  
> Share this with any kimi.com web agent, ClaudeCoder, or CodeXCoder before assigning work.

---

## 1. What is DClaw Stack?

DClaw Stack is an AI-native application platform — 65+ vertical SaaS products unified under one launcher (DPanel), one auth layer (Logto), one billing layer (Stripe), and one Kubernetes operator (DClaw Operator).

**Vision:** Every team runs their own AI app store. Install, use, and monetize AI agents like mobile apps.

**Current Phase:** P0 Foundation (May 2026) — building the platform backbone + first 3 products.

---

## 2. Sacred Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         DClaw Platform                              │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ │
│  │   DPanel    │    │   Users     │    │      Enterprise         │ │
│  │ (Launcher)  │◄───┤  (Web/Dsk)  │◄───┤      (On-Prem)          │ │
│  │  Next.js 16 │    │             │    │                         │ │
│  └──────┬──────┘    └─────────────┘    └─────────────────────────┘ │
│         │                                                           │
│  ┌──────┴────────────────────────────────────────────────────────┐ │
│  │  DClaw Products (dclaw-chat, dclaw-flow, dclaw-rag, ...)      │ │
│  │  ├── Next.js Frontend (Vercel / K8s)                         │ │
│  │  ├── FastAPI Backend (K8s)                                   │ │
│  │  ├── PostgreSQL (CloudNativePG)                              │ │
│  │  └── Optional: Qdrant, Redis, Temporal, MinIO               │ │
│  └──────┬────────────────────────────────────────────────────────┘ │
│         │                                                           │
│  ┌──────┴────────────────────────────────────────────────────────┐ │
│  │  DClaw Operator (Go + controller-runtime)                     │ │
│  │  • Watches DClawApp CRDs                                     │ │
│  │  • Auto-provisions: namespace, deployments, DB, ingress      │ │
│  │  • Network policies + resource quotas                        │ │
│  └────────────────────────────────────────────────────────────────┘ │
│         │                                                           │
│  ┌──────┴──────┐  ┌─────────────┐  ┌─────────────┐               │
│  │  Identity   │  │   Billing   │  │   Shield    │               │
│  │  (Logto)    │  │  (Stripe)   │  │(PII protect)│               │
│  └─────────────┘  └─────────────┘  └─────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Tech Stack Bible (NON-NEGOTIABLE)

Every DClaw product MUST use this exact stack. No exceptions.

| Layer | Technology | Version | Notes |
|-------|------------|---------|-------|
| **Frontend** | Next.js 14+ | App Router | Tailwind CSS + shadcn/ui |
| **Backend** | FastAPI | latest | Pydantic v2, SQLAlchemy 2.0, asyncpg |
| **Desktop** | Tauri v2 | 2.0.0-beta+ | Optional; defer to v1.1 |
| **Database** | PostgreSQL 16 | latest | CloudNativePG operator in K8s |
| **Vector DB** | Qdrant / pgvector | latest | Only if RAG / semantic search |
| **Cache / Bus** | Redis | 7.x | Celery broker, cache, pub/sub |
| **Object Storage** | MinIO | latest | File/media storage |
| **Workflow** | Temporal.io | latest | Only if automation/orchestration |
| **Auth** | Logto | latest | JWT validation on all protected routes |
| **Billing** | Stripe | API | Metered or per-seat |
| **K8s Operator** | Go + controller-runtime | 0.18 | Watches DClawApp CRDs |
| **LLM Local** | Ollama | latest | Local inference on Apple Silicon |
| **LLM Cloud** | OpenRouter + Kimi K2.5 | API | Fallback when local is unavailable |
| **Monitoring** | Prometheus + Grafana | latest | Metrics and dashboards |

### 3.1 Python Rules
- `ruff` formatting enforced
- Type hints on ALL public APIs
- `pydantic` v2 for schemas
- `sqlalchemy` 2.0 style (`Mapped`, `mapped_column`)
- `pytest` + `pytest-asyncio` for tests
- Docstrings on all routers and services
- Functions < 50 lines
- No `print()` — use `structlog` or standard logging

### 3.2 TypeScript / Next.js Rules
- Strict TypeScript (`strict: true`)
- Functional components with hooks
- Co-locate hooks with components
- Tailwind for ALL styling
- `cn()` utility for conditional classes
- No `any` types without `// @ts-ignore` comment

### 3.3 Go Rules (Operator only)
- `gofmt` enforced
- Idiomatic error handling
- Structured logging with `slog` or `zap`
- Table-driven tests
- No `panic` in production code

---

## 4. Repo Structure Convention

Every product repo MUST follow this layout:

```
dclaw-{app_id}/
├── frontend/               → Next.js 14+, Tailwind, shadcn/ui
│   ├── app/                → App router pages
│   ├── components/         → Product-specific UI
│   ├── lib/                → Client utils, API clients
│   └── public/
│       └── dclaw-manifest.json   ← DPanel registration
├── backend/                → FastAPI
│   ├── app/
│   │   ├── api/v1/         → Routers
│   │   ├── models/         → SQLAlchemy 2.0 models
│   │   ├── schemas/        → Pydantic v2 schemas
│   │   ├── services/       → Business logic
│   │   └── core/           → Config, deps, logging
│   ├── tests/              → pytest + pytest-asyncio
│   ├── alembic/            → DB migrations
│   ├── Dockerfile
│   └── requirements.txt or pyproject.toml
├── helm/                   → K8s manifests
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── ingress.yaml
│       └── cloudnativepg.yaml
└── .github/workflows/
    ├── build-frontend.yml
    ├── build-backend.yml
    └── deploy.yml
```

**Exceptions:**
- Backend-only services (e.g., RAG v0.1.0) may omit `frontend/` but MUST add one before v1.0.
- Root-level `Dockerfile` is acceptable for single-service repos (builds from root context).

---

## 5. Port Registry

**Rule:** Check this table before assigning ANY port. Update it when you change a port.

### System Ports (DO NOT USE)
| Port | Process | Notes |
|------|---------|-------|
| 80 | Docker/Colima | HTTP ingress |
| 443 | Docker/Colima | HTTPS ingress |
| 3001 | PM2 | Unknown Node service |
| 4040 | ngrok | Tunnel dashboard |
| 5000 | macOS | AirPlay / Control Center |
| 5002 | PM2 | OCR service |
| 5200 | PM2 | SeaClip lite |
| 5300 | PM2 | Mortgage lite |
| 5310 | PM2 | Law lite |
| 5432 | Docker | PostgreSQL |
| 6443 | K8s tunnel | Kubernetes API |
| 7000 | macOS | Control Center alt |
| 8000 | Docker | Some Docker service |
| 8080 | kubectl / PM2 | K8s port-forward |
| 9090 | PM2 | Hub dashboard |
| 9093 | PM2 | App store |
| 10010 | K8s tunnel | K8s port-forward |
| 10248-10259 | K8s tunnel | Kubelet ports |
| 11434 | ollama | Local LLM inference |
| 30080 | K8s tunnel | K8s port-forward |
| 4321 | PM2 | Unknown Node service |
| 4322 | PM2 | Hub docs |
| 5180 | PM2 | Unknown Node service |
| 8174 | PM2 | Aina service |
| 8175 | PM2 | Unknown Node service |

### DClaw Stack Ports (ASSIGNED)
| Port | Service | Environment | Status |
|------|---------|-------------|--------|
| 3000 | DPanel dev server | Local dev | ✅ Free |
| 3002 | DClaw Chat frontend dev | Local dev | ✅ Free |
| 8008 | DClaw Chat backend dev | Local dev | ✅ Free |
| **8088** | dpanel-api | Local + K8s | ✅ Assigned |
| **8089** | dclaw-operator metrics | K8s cluster | ✅ Assigned |
| **8090** | dclaw-rag backend | Local + K8s | ✅ Assigned |
| **8091** | dclaw-agent backend | Local + K8s | ✅ Assigned |
| 18080 | dclaw-operator metrics (local fallback) | Local dev | ✅ Free |
| 3003 | *Reserved: DClaw Flow dev* | Future | ✅ Free |
| 3004 | *Reserved: DClaw Med dev* | Future | ✅ Free |
| 3005 | *Reserved: DClaw Learn dev* | Future | ✅ Free |
| 8443 | *Reserved: DClaw HTTPS dev* | Future | ✅ Free |

### Port Ranges
- **3000–3009:** DClaw frontend dev servers (Next.js apps)
- **8008–8010:** DClaw backend dev servers (FastAPI / Go)
- **8088–8095:** DClaw platform services (dpanel-api, operator, products)
- **18080–18090:** DClaw platform local fallbacks

---

## 6. Current Build Status

### P0: Foundation (May 2026) — ACTIVE

| Component | Repo | Version | Status | Notes |
|-----------|------|---------|--------|-------|
| **DClaw Chat** | [dclaw-chat](https://github.com/dclawstack/dclaw-chat) | v0.1.0 | ✅ Shipped | Next.js + FastAPI + Tauri + Helm |
| **DClaw Flow** | [dclaw-flow](https://github.com/dclawstack/dclaw-flow) | v0.1.0 | ✅ Scaffolded | Visual workflow builder |
| **DClaw RAG** | [dclaw-rag](https://github.com/dclawstack/dclaw-rag) | v0.1.0 | ✅ Scaffolded | Qdrant-based RAG engine |
| **DClaw Agent** | [dclaw-agent](https://github.com/dclawstack/dclaw-agent) | v0.1.0 | ✅ Scaffolded | Agent marketplace + builder |
| **DPanel** | [dclaw-platform/dpanel](https://github.com/dclawstack/dclaw-platform) | — | 🏗️ Scaffolded | Next.js 16 app launcher |
| **dpanel-api** | [dclaw-platform/dpanel-api](https://github.com/dclawstack/dclaw-platform) | — | 🏗️ Scaffolded | Go ConfigMap reader |
| **DClaw Operator** | [dclaw-platform](https://github.com/dclawstack/dclaw-platform) | v0.1.0 | ✅ Reconciler | 9-step pipeline, AppId, nested resources |
| **Agent Swarm** | [dclaw-platform](https://github.com/dclawstack/dclaw-platform) | v0.1.0 | ✅ Runtime | 7 agent prompts, registry, orchestrator |
| **Port Registry** | [dclaw-platform](https://github.com/dclawstack/dclaw-platform) | — | ✅ Fixed | 8088, 8090, 8091 assigned |

### P1: Platform (Jun 2026) — QUEUED

| Product | Repo | Spec | Status |
|---------|------|------|--------|
| DClaw Flow | dclaw-flow | ✅ Done | Implementing |
| DClaw RAG | dclaw-rag | ✅ Done | Implementing |
| DClaw Agent | dclaw-agent | ✅ Done | Implementing |
| DClaw Agent Marketplace | — | ⏳ Pending | P1.5 |

### Blockers

| Blocker | Impact | Resolution |
|---------|--------|----------|
| Apple Developer Cert ($99/yr) | Signed macOS desktop apps | Defer to v1.1, unsigned for dev |
| OPENROUTER_API_KEY | Cloud fallback for Chat | User to provide |
| Stripe account | Billing integration | Apply after first user |
| Logto instance | SSO/auth | Self-host or use cloud |

---

## 7. The 65-Product Grid

```
P0: Foundation (May 2026)
├── 01 chat      → DClaw Chat      → Communication  → "AI conversations that remember"
├── 02 flow      → DClaw Flow      → Automation     → "Connect anything, automate everything"
├── 03 med       → DClaw Med       → Healthcare     → "Clinical intelligence"
├── 04 learn     → DClaw Learn     → Education      → "Adaptive learning"
├── 05 seo       → DClaw SEO       → Marketing      → "Rank higher with AI"
├── 06 create    → DClaw Create    → Media          → "Generate anything"
├── 07 code      → DClaw Code      → Development    → "AI-native IDE"
├── 08 agent     → DClaw Agent     → Platform       → "Build, share, sell AI agents"
└── 09 rag       → DClaw RAG       → Platform       → "Universal knowledge retrieval"

P1: Platform (Jun 2026)
├── Marketplace → Agent store, ratings, revenue sharing
├── Flow Engine → Temporal.io workflows, 100+ integrations
└── RAG Infra   → Universal document ingestion, hybrid search

P2: Verticals (Jul 2026)
├── Med, Learn, Code deep features
└── Enterprise RBAC, HIPAA, SOC2

P3: Scale (Aug 2026)
├── SEO, Create, Enterprise white-label
└── Billing, teams, API access

P4: YC (Sep 2026)
└── Demo day: Chat + Flow + Agent marketplace
```

### Full Product Catalog (65+ Apps)

| # | App ID | Name | Category | Tagline | Color | Status |
|---|--------|------|----------|---------|-------|--------|
| 1 | chat | DClaw Chat | Communication | AI conversations that remember | #3B82F6 | ✅ P0 |
| 2 | flow | DClaw Flow | Automation | Connect anything, automate everything | #10B981 | ✅ P0 |
| 3 | med | DClaw Med | Healthcare | Clinical intelligence | #EF4444 | ⏳ P2 |
| 4 | learn | DClaw Learn | Education | Adaptive learning | #3B82F6 | ⏳ P2 |
| 5 | seo | DClaw SEO | Marketing | Rank higher with AI | #10B981 | ⏳ P3 |
| 6 | create | DClaw Create | Media | Generate anything | #EC4899 | ⏳ P3 |
| 7 | code | DClaw Code | Development | AI-native IDE | #1F2937 | ⏳ P2 |
| 8 | agent | DClaw Agent | Platform | Build, share, sell AI agents | #8B5CF6 | ✅ P0 |
| 9 | rag | DClaw RAG | Platform | Universal knowledge retrieval | #F59E0B | ✅ P0 |
| 10 | legal | DClaw Legal | Legal | Contract review, case law | #6366F1 | 🔮 Future |
| 11 | finance | DClaw Finance | Finance | Financial modeling | #10B981 | 🔮 Future |
| 12 | sales | DClaw Sales | Sales | CRM AI, email sequences | #F59E0B | 🔮 Future |
| 13 | support | DClaw Support | Support | Ticket resolution | #3B82F6 | 🔮 Future |
| 14 | hr | DClaw HR | HR | Resume screening, interviews | #EC4899 | 🔮 Future |
| 15 | design | DClaw Design | Design | UI/UX generation | #8B5CF6 | 🔮 Future |
| 16 | translate | DClaw Translate | Language | Real-time translation | #10B981 | 🔮 Future |
| 17 | write | DClaw Write | Content | Long-form writing | #3B82F6 | 🔮 Future |
| 18 | meet | DClaw Meet | Meetings | Transcription, summaries | #1F2937 | 🔮 Future |
| 19 | doc | DClaw Doc | Documents | Smart documents | #6366F1 | 🔮 Future |
| 20 | sheet | DClaw Sheet | Spreadsheets | AI-powered Excel | #10B981 | 🔮 Future |
| 21 | slide | DClaw Slide | Presentations | AI-generated decks | #EC4899 | 🔮 Future |
| 22 | email | DClaw Mail | Email | Smart inbox | #3B82F6 | 🔮 Future |
| 23 | calendar | DClaw Calendar | Scheduling | AI scheduling | #10B981 | 🔮 Future |
| 24 | task | DClaw Task | Productivity | Smart to-do | #F59E0B | 🔮 Future |
| 25 | wiki | DClaw Wiki | Knowledge | Internal Wikipedia | #6366F1 | 🔮 Future |
| 26 | data | DClaw Data | Analytics | Natural language analysis | #3B82F6 | 🔮 Future |
| 27 | api | DClaw API | Developer | API design, testing | #1F2937 | 🔮 Future |
| 28 | test | DClaw Test | QA | Automated testing | #10B981 | 🔮 Future |
| 29 | deploy | DClaw Deploy | DevOps | CI/CD pipeline builder | #EC4899 | 🔮 Future |
| 30 | monitor | DClaw Monitor | Observability | AI-powered alerting | #3B82F6 | 🔮 Future |
| 31 | secure | DClaw Secure | Security | Threat detection | #6366F1 | 🔮 Future |
| 32 | backup | DClaw Backup | Infrastructure | Intelligent backup | #10B981 | 🔮 Future |
| 33 | migrate | DClaw Migrate | Infrastructure | Cloud migration | #F59E0B | 🔮 Future |
| 34 | cost | DClaw Cost | FinOps | Cloud cost optimization | #3B82F6 | 🔮 Future |
| 35 | carbon | DClaw Carbon | Sustainability | Carbon tracking | #10B981 | 🔮 Future |
| 36 | compliance | DClaw Compliance | Governance | GDPR, SOC2, HIPAA | #6366F1 | 🔮 Future |
| 37 | audit | DClaw Audit | Governance | Automated audit trails | #1F2937 | 🔮 Future |
| 38 | policy | DClaw Policy | Governance | Policy management | #3B82F6 | 🔮 Future |
| 39 | train | DClaw Train | L&D | Employee training | #10B981 | 🔮 Future |
| 40 | recruit | DClaw Recruit | Talent | Job posting, candidates | #EC4899 | 🔮 Future |
| 41 | onboard | DClaw Onboard | HR | Automated onboarding | #3B82F6 | 🔮 Future |
| 42 | offboard | DClaw Offboard | HR | Secure offboarding | #6366F1 | 🔮 Future |
| 43 | assets | DClaw Assets | IT | Hardware/software management | #10B981 | 🔮 Future |
| 44 | network | DClaw Network | IT | Network monitoring | #F59E0B | 🔮 Future |
| 45 | inventory | DClaw Inventory | Operations | Supply chain | #3B82F6 | 🔮 Future |
| 46 | forecast | DClaw Forecast | Operations | Demand forecasting | #10B981 | 🔮 Future |
| 47 | quality | DClaw Quality | Manufacturing | Quality control AI | #EC4899 | 🔮 Future |
| 48 | maintenance | DClaw Maintenance | Operations | Predictive maintenance | #3B82F6 | 🔮 Future |
| 49 | route | DClaw Route | Logistics | Route optimization | #10B981 | 🔮 Future |
| 50 | warehouse | DClaw Warehouse | Logistics | Warehouse automation | #6366F1 | 🔮 Future |
| 51 | fleet | DClaw Fleet | Logistics | Fleet management | #3B82F6 | 🔮 Future |
| 52 | energy | DClaw Energy | Utilities | Energy optimization | #10B981 | 🔮 Future |
| 53 | water | DClaw Water | Utilities | Water management | #3B82F6 | 🔮 Future |
| 54 | waste | DClaw Waste | Sustainability | Waste reduction | #10B981 | 🔮 Future |
| 55 | building | DClaw Building | Real Estate | Smart building | #EC4899 | 🔮 Future |
| 56 | space | DClaw Space | Real Estate | Office optimization | #3B82F6 | 🔮 Future |
| 57 | lease | DClaw Lease | Real Estate | Lease management | #10B981 | 🔮 Future |
| 58 | vendor | DClaw Vendor | Procurement | Vendor evaluation | #6366F1 | 🔮 Future |
| 59 | contract | DClaw Contract | Legal | Contract lifecycle | #3B82F6 | 🔮 Future |
| 60 | risk | DClaw Risk | Governance | Risk management | #10B981 | 🔮 Future |
| 61 | crisis | DClaw Crisis | Operations | Crisis response | #EC4899 | 🔮 Future |
| 62 | continuity | DClaw Continuity | Operations | Business continuity | #3B82F6 | 🔮 Future |
| 63 | knowledge | DClaw Knowledge | Platform | Enterprise knowledge graph | #10B981 | 🔮 Future |
| 64 | research | DClaw Research | R&D | Paper analysis | #6366F1 | 🔮 Future |
| 65 | patent | DClaw Patent | Legal | Patent search | #3B82F6 | 🔮 Future |

---

## 8. How to Scaffold a New DClaw App

### Step 1: Read This PRD
Understand the stack, conventions, and port registry.

### Step 2: Pick from the Product Grid
Choose an unbuilt app from §7. Check its status. If it's 🔮 Future, ask the Vault Coordinator before building.

### Step 3: Check Port Registry
Pick an available port from the 8088–8095 range (or 3000–3009 for frontend dev). Update this PRD.

### Step 4: Create Repo
```bash
# Use GitHub CLI (already authenticated)
gh repo create dclawstack/dclaw-{app_id} --public \
  --description "DClaw {AppName} — {Tagline}"
```

### Step 5: Scaffold
Follow the standard repo structure from §4. Use these exact technologies:
- Frontend: Next.js 14+, Tailwind, shadcn/ui
- Backend: FastAPI, Pydantic v2, SQLAlchemy 2.0, asyncpg
- Database: PostgreSQL (CloudNativePG in Helm)

### Step 6: Add Helm Chart
Copy from `dclaw-flow/helm/dclaw-flow/` as a template. Replace:
- `{app_id}` → your app ID
- `{port}` → your assigned port
- `{domain}` → `{app_id}.dclawstack.io`

**Rules:**
- Service type MUST be `ClusterIP` (no NodePort)
- Backend containerPort MUST match port registry
- Ingress MUST use TLS with cert-manager

### Step 7: Add CI/CD
Copy `.github/workflows/` from `dclaw-flow/.github/workflows/`.

### Step 8: Add DPanel Manifest
Create `frontend/public/dclaw-manifest.json`:
```json
{
  "app_id": "{app_id}",
  "name": "DClaw {AppName}",
  "version": "0.1.0",
  "category": "{category}",
  "icon": "/icons/{app_id}.svg",
  "color": "#HEXCODE",
  "entrypoint": "/",
  "api_base": "https://{app_id}.dclawstack.io",
  "permissions": [],
  "requires_auth": true,
  "billing_plan": "free"
}
```

### Step 9: Commit & Push
```bash
git add .
git commit -m "feat({app_id}): initial scaffold"
git push origin main
```

### Step 10: Notify Vault Coordinator
Update this PRD with the new app's status.

---

## 9. Common Backend Patterns

### 9.1 FastAPI App Factory
```python
# backend/app/main.py
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup: create tables
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    # Shutdown: cleanup
    await engine.dispose()

app = FastAPI(title="DClaw {AppName}", lifespan=lifespan)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### 9.2 Pydantic Settings
```python
# backend/app/core/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    app_env: str = "dev"
    log_level: str = "INFO"
    api_host: str = "0.0.0.0"
    api_port: int = 8090  # MUST match port registry
    database_url: str = "postgresql+asyncpg://postgres:postgres@localhost:5432/dclaw_{app_id}"
    
    class Config:
        env_file = ".env"

settings = Settings()
```

### 9.3 SQLAlchemy 2.0 Models
```python
# backend/app/models.py
from sqlalchemy import String
from sqlalchemy.orm import Mapped, mapped_column, DeclarativeBase

class Base(DeclarativeBase):
    pass

class Widget(Base):
    __tablename__ = "widgets"
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(255))
    created_at: Mapped[datetime] = mapped_column(DateTime, default=func.now())
```

### 9.4 Health Endpoint
```python
# backend/app/api/routes/health.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/health")
async def health():
    return {"status": "ok", "version": "0.1.0"}
```

### 9.5 Async Database Session Dependency
```python
# backend/app/core/deps.py
from sqlalchemy.ext.asyncio import AsyncSession

async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        yield session
```

---

## 10. Common Frontend Patterns

### 10.1 API Client
```typescript
// frontend/lib/api.ts
const API_BASE = process.env.NEXT_PUBLIC_API_URL || "http://localhost:8088";

export async function apiFetch(path: string, options?: RequestInit) {
  const res = await fetch(`${API_BASE}${path}`, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...options?.headers,
    },
  });
  if (!res.ok) throw new Error(`API error: ${res.status}`);
  return res.json();
}
```

### 10.2 Design Tokens
```typescript
// frontend/lib/tokens.ts
export const APP_COLOR = "#3B82F6"; // Replace with app's color
export const APP_NAME = "DClaw Chat";
export const APP_ICON = "💬";
```

---

## 11. Git Conventions

### Branch Naming
```
feat/{app-id}/description       # New feature
fix/{app-id}/description        # Bug fix
docs/{app-id}/description       # Documentation
chore/{app-id}/description      # Maintenance
```

### Commit Format
```
feat(chat): add voice input button
fix(operator): resolve namespace race condition
docs(chat): update API contract
chore(platform): bump controller-runtime to v0.18.0
```

---

## 12. GitHub Org & Repos

**Org:** https://github.com/dclawstack

### Active Repos

| Repo | Purpose | Link |
|------|---------|------|
| dclaw-chat | Chat product (P0) | https://github.com/dclawstack/dclaw-chat |
| dclaw-flow | Flow product (P0) | https://github.com/dclawstack/dclaw-flow |
| dclaw-rag | RAG product (P0) | https://github.com/dclawstack/dclaw-rag |
| dclaw-agent | Agent product (P0) | https://github.com/dclawstack/dclaw-agent |
| dclaw-platform | Operator + DPanel + Infra | https://github.com/dclawstack/dclaw-platform |
| dclaw-obsidian | Obsidian vault (private) | https://github.com/dclawstack/dclaw-obsidian |
| .github | Workflows, org profile | https://github.com/dclawstack/.github |
| dclaw-enterprise | White-label builds | https://github.com/dclawstack/dclaw-enterprise |
| dclaw-prd | Sacred context docs | https://github.com/dclawstack/dclaw-prd |

---

## 13. Security Rules (Shield Agent Enforced)

1. **NO hardcoded secrets** — use `.env.example` + K8s Secrets
2. **NO system ports** in manifests — check port registry
3. **Service type MUST be ClusterIP** — no NodePort without approval
4. **TLS required** on all ingress — cert-manager + letsencrypt
5. **PII scan** — ClawShield on all user-generated content
6. **Non-root containers** — Dockerfile must use `USER` directive
7. **CORS strict** — no wildcard origins in production

---

## 14. Agent Instructions

### If You Are a Web Agent (kimi.com)
1. Read this PRD fully before starting
2. Pick an app from the product grid
3. Scaffold using the exact stack from §3
4. Push to `github.com/dclawstack/dclaw-{app_id}`
5. Use port from registry (§5)
6. Include `frontend/public/dclaw-manifest.json`

### If You Are a Local Agent (Shell / ClaudeCoder / CodeXCoder)
1. Clone the repo
2. Run smoke tests (`uvicorn` + health check)
3. Add Helm chart + CI + manifest if missing
4. Fix port violations immediately
5. Commit with `feat({app_id}): description`
6. Update this PRD when architecture changes

---

## 15. Links & Resources

| Resource | URL |
|----------|-----|
| GitHub Org | https://github.com/dclawstack |
| DPanel (Vercel) | https://dpanel.dclawstack.io |
| Port Registry | `dclaw-platform/PORT_REGISTRY.md` |
| Agent Swarm Guide | `dclaw-platform/AGENT_SWARM.md` |
| Master Blueprint | Obsidian Vault → `00-META/🏛️ Master Blueprint.md` |
| PRD Template | Obsidian Vault → `00-META/📐 App PRD Template.md` |
| Web-to-DPanel Playbook | Obsidian Vault → `00-META/🌐 Web-to-DPanel Playbook.md` |

---

*Master PRD version: 2.0*  
*Last updated: 2026-05-04 by Vault Coordinator*  
*Next review: When adding P2 products or changing stack*
