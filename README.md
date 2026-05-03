# DClaw Platform — Sacred Context Document

> **Purpose:** This repository is the single source of truth for the DClaw Platform. Paste this entire repo (or key files) into any AI agent chat to instantly bootstrap context. Clone it into Obsidian for local knowledge graph integration.

---

## Quick Facts

| | |
|---|---|
| **Company** | DClawstack |
| **Vision** | Kubernetes-native AI app store and operating system for companies |
| **Model** | Adobe Creative Cloud — one platform, 65 independently branded AI apps |
| **YC Target** | Summer 2026 |
| **GitHub Org** | [github.com/dclawstack](https://github.com/dclawstack) |
| **Primary Stack** | Go (operator), Next.js 14 (frontend), FastAPI (backend), Tauri v2 (desktop) |
| **Local AI** | Ollama on Mac M4 96GB (Gemma 4B/27B, Qwen 32B) |
| **Cloud AI** | OpenRouter + Kimi K2.5 |
| **Infra** | Kubernetes, CloudNativePG, Redis, MinIO |

---

## How to Use This PRD

### Option A: Seed a Kimi Web Chat
1. Open [kimi.com](https://kimi.com)
2. Paste `VISION.md` + `ARCHITECTURE.md` + `PRODUCTS.md` into the chat
3. Add: "You are building the DClaw Platform. Read these docs and ask clarifying questions."

### Option B: Clone into Obsidian
```bash
cd ~/DClaw-Stack/Obsidian-Vault/09-RESEARCH/
git clone https://github.com/dclawstack/dclaw-prd.git
cd dclaw-prd
```

### Option C: Reference in Code Agent Prompts
```
Before writing any code, read:
1. ARCHITECTURE.md for system design
2. CONVENTIONS.md for coding standards
3. STATUS.md for what's already built
```

---

## Document Index

| Document | Purpose | Paste Priority |
|----------|---------|----------------|
| [VISION.md](VISION.md) | Why we exist, business model, YC alignment | **High** |
| [ARCHITECTURE.md](ARCHITECTURE.md) | System design, data flow, K8s architecture | **High** |
| [PRODUCTS.md](PRODUCTS.md) | All 65 products with IDs, categories, specs | **High** |
| [CONVENTIONS.md](CONVENTIONS.md) | Coding standards, commit format, review process | **Medium** |
| [SETUP.md](SETUP.md) | Local dev setup, M4 Mac config, dependencies | **Medium** |
| [AGENTS.md](AGENTS.md) | How agents coordinate, swarm protocol, ownership | **Medium** |
| [STATUS.md](STATUS.md) | What's built, what's queued, blockers | **High** |
| [SECURITY.md](SECURITY.md) | Secrets, PII handling, compliance notes | **Low** |

---

## One-Line Descriptions

| Layer | Description |
|-------|-------------|
| **DClaw Operator** | K8s controller that watches `DClawApp` CRDs and auto-provisions namespaces, deployments, databases, ingress |
| **DPanel** | 9-dot launcher + app store (Next.js + shadcn/ui) |
| **ClawShield** | PII anonymization before cloud inference |
| **DClaw Voice** | "Hey DClaw" wake word + STT/TTS |
| **Per-App Stack** | Next.js frontend, FastAPI backend, PostgreSQL, Tauri desktop, Helm chart |
| **BYOK** | Customers bring their own API keys — we never hold cloud LLM credentials |

---

## Current Status (2026-05-04)

| Component | Status |
|-----------|--------|
| DClaw Operator reconciler | ✅ v0.1.0 (9-step pipeline) |
| DClaw Chat frontend | ✅ v0.1.0 (Next.js + Tauri) |
| DClaw Chat backend | ✅ v0.1.0 (FastAPI + Ollama proxy) |
| Agent Swarm runtime | ✅ v0.1.0 (5 agents + orchestrator) |
| Helm chart | ✅ v0.1.0 |
| Telegram CI pipeline | ✅ LIVE |
| DPanel | ⏳ Not started |
| Apple Developer Cert | ⏳ Deferred to v1.1 |
| CloudNativePG types in operator | ⏳ Needs cnpg import |

---

*Last updated: 2026-05-04 by Build Agent (Kimi CLI)*
