# SETUP

## Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| macOS | 14+ | Development OS |
| Go | 1.23+ | Operator |
| Node.js | 20+ | Frontend |
| Python | 3.12+ | Backend |
| Rust | 1.75+ | Tauri desktop |
| Docker | Latest | Containers |
| kubectl | Latest | K8s CLI |
| Helm | 3.14+ | K8s packages |
| gh CLI | Latest | GitHub |

## Mac M4 96GB Setup

```bash
# 1. Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. Install core tools
brew install go node python@3.12 rust docker kubectl helm gh

# 3. Install Ollama
brew install ollama
ollama serve &

# 4. Pull local models
ollama pull gemma:4b
ollama pull gemma:27b
ollama pull qwen:32b

# 5. Clone repos
mkdir -p ~/DClaw-Stack && cd ~/DClaw-Stack
git clone https://github.com/dclawstack/dclaw-platform.git
git clone https://github.com/dclawstack/dclaw-chat.git
git clone https://github.com/dclawstack/dclaw-obsidian.git

# 6. Install frontend deps
cd dclaw-chat && npm install

# 7. Install backend deps
cd backend && pip install -r requirements.txt

# 8. Run frontend
npm run dev          # http://localhost:3000

# 9. Run backend
uvicorn main:app --reload --port 8000

# 10. Run desktop
npm run tauri:dev
```

## Local K8s (k3s)

```bash
# One-command install
curl -sfL https://get.k3s.io | sh -

# Set kubeconfig
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

# Verify
kubectl get nodes
```

## Environment Variables

```bash
# Required for cloud fallback
export OPENROUTER_API_KEY="your-key-here"

# Required for Stripe billing
export STRIPE_SECRET_KEY="sk_..."

# Required for Logto auth
export LOGTO_ENDPOINT="https://auth.dclawstack.io"
export LOGTO_APP_SECRET="..."

# Database (local dev)
export DATABASE_URL="postgresql+asyncpg://postgres:postgres@localhost:5432/dclaw_chat"
```

## GitHub CLI Auth

```bash
gh auth login
gh auth refresh -h github.com -s admin:org  # For org secrets
```

## Common Commands

```bash
# dclaw-operator
cd dclaw-platform/dclaw-operator
go build ./...
go test ./...

# dclaw-chat frontend
cd dclaw-chat
npm run dev
npm run build
npm run tauri:build

# dclaw-chat backend
cd dclaw-chat/backend
uvicorn main:app --reload
pytest

# Helm deploy
cd dclaw-chat/helm/dclaw-chat
helm install dclaw-chat . --namespace dclaw-chat --create-namespace
```
