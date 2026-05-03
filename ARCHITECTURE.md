# ARCHITECTURE

## System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                          DClaw Platform                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│  │   Users     │    │   Users     │    │  Enterprise │            │
│  │  (Web)      │    │  (Desktop)  │    │  (On-Prem)  │            │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘            │
│         │                  │                  │                    │
│         └──────────────────┼──────────────────┘                    │
│                            ▼                                        │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  DPanel (Next.js + Tailwind + shadcn/ui)                    │   │
│  │  • 9-dot app launcher                                       │   │
│  │  • App store (install/uninstall)                            │   │
│  │  • Billing dashboard                                        │   │
│  │  • Team management                                          │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                            │                                        │
│         ┌──────────────────┼──────────────────┐                    │
│         ▼                  ▼                  ▼                    │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│  │ DClaw Chat  │    │ DClaw Flow  │    │ DClaw Med   │            │
│  │ (Product 1) │    │ (Product 2) │    │ (Product 3) │            │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘            │
│         │                  │                  │                    │
│         └──────────────────┼──────────────────┘                    │
│                            ▼                                        │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  DClaw Operator (Go + controller-runtime)                   │   │
│  │  • Watches DClawApp CRDs                                    │   │
│  │  • Auto-provisions: namespace, deployments, DB, ingress     │   │
│  │  • Network policies for isolation                           │   │
│  │  • Resource quotas per app                                  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                            │                                        │
│         ┌──────────────────┼──────────────────┐                    │
│         ▼                  ▼                  ▼                    │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│  │  Identity   │    │   Billing   │    │   Shield    │            │
│  │  (Logto)    │    │  (Stripe)   │    │(PII protect)│            │
│  └─────────────┘    └─────────────┘    └─────────────┘            │
│                                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│  │   Voice     │    │   Updater   │    │  Message Bus │           │
│  │(Wake word)  │    │(Auto-update)│    │   (Redis)   │            │
│  └─────────────┘    └─────────────┘    └─────────────┘            │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Infrastructure Layer                                       │   │
│  │  • Kubernetes (k3s for local, managed for cloud)            │   │
│  │  • PostgreSQL (CloudNativePG)                               │   │
│  │  • Redis (message bus + cache)                              │   │
│  │  • MinIO (object storage)                                   │   │
│  │  • Prometheus + Grafana (monitoring)                        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Per-App Architecture

Every app follows the same pattern:

```
App (e.g., dclaw-chat)
├── Next.js Frontend        → Serves UI, talks to backend API
├── FastAPI Backend         → Business logic, DB access, AI proxies
├── PostgreSQL              → Per-app database (CloudNativePG cluster)
├── Tauri Desktop           → WebView wrapper with native features
└── Helm Chart              → K8s manifests for deployment
```

## Data Flow

```
User Input
    │
    ▼
┌─────────────────┐
│  ClawShield     │ ← PII detection/redaction
│  (local, always)│
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌────────┐  ┌──────────┐
│ Ollama │  │ OpenRouter│ ← Local LLM vs Cloud fallback
│ (local)│  │ (cloud)   │
└───┬────┘  └─────┬────┘
    │             │
    └──────┬──────┘
           ▼
    ┌─────────────┐
    │  Response   │
    │  Stream     │
    └─────────────┘
```

## CRD: DClawApp

```yaml
apiVersion: platform.dclaw.io/v1
kind: DClawApp
metadata:
  name: chat
spec:
  appId: chat
  appName: DClaw Chat
  version: 0.1.0
  category: communication
  enabled: true
  frontend:
    image: ghcr.io/dclawstack/dclaw-chat:latest
    replicas: 2
  backend:
    image: ghcr.io/dclawstack/dclaw-chat-backend:latest
    replicas: 2
  database:
    enabled: true
    storage: 10Gi
  ingress:
    enabled: true
    host: chat.dclawstack.io
    tls: true
  resources:
    limits: { cpu: 1000m, memory: 2Gi }
    requests: { cpu: 250m, memory: 512Mi }
  branding:
    primaryColor: "#3B82F6"
  billing:
    tier: pro
```

## Tech Stack Detail

| Layer | Technology | Version |
|-------|------------|---------|
| K8s Operator | Go + controller-runtime | 0.18 |
| DPanel + Frontends | Next.js 14 + Tailwind + shadcn/ui | latest |
| App Backends | FastAPI + uvicorn + SQLAlchemy 2.0 | latest |
| Databases | PostgreSQL 16 + CloudNativePG | latest |
| Desktop | Tauri v2 | 2.0.0-beta |
| Local LLMs | Ollama | latest |
| Cloud LLMs | OpenRouter + Kimi K2.5 | API |
| Message Bus | Redis | 7.x |
| Object Storage | MinIO | latest |
| Monitoring | Prometheus + Grafana | latest |
| Auth | Logto | latest |
| Billing | Stripe | API |

## Network Architecture

```
Internet
    │
    ▼
┌─────────────┐
│  Ingress    │ ← nginx-ingress + cert-manager
│  Controller │
└──────┬──────┘
       │
   ┌───┴───┐
   ▼       ▼
┌─────┐ ┌─────┐
│ /   │ │ /api│
│ Web │ │ API │
└──┬──┘ └──┬──┘
   │       │
   ▼       ▼
┌────────┐ ┌────────┐
│Frontend│ │Backend │
│ Pods   │ │ Pods   │
└───┬────┘ └───┬────┘
    │          │
    │    ┌─────┴─────┐
    │    ▼           ▼
    │ ┌──────┐  ┌────────┐
    │ │Postgre│  │ Ollama │
    │ │SQL    │  │ (sidecar│
    │ └──────┘  │ or svc) │
    │           └────────┘
    │
    └──────► Redis (cache + pub/sub)
```
