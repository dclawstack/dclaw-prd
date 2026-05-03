# SECURITY

## Secrets

| Secret | Location | Set By | Status |
|--------|----------|--------|--------|
| `TELEGRAM_BOT_TOKEN` | GitHub org + dclaw-platform repo | User via @BotFather | ✅ Set |
| `TELEGRAM_META_CHANNEL` | GitHub org + dclaw-platform repo | Build Agent | ✅ Set (890034905) |
| `VERCEL_TOKEN` | GitHub org | User | ✅ Set |
| `VERCEL_ORG_ID` | GitHub org | User | ✅ Set |
| `OPENROUTER_API_KEY` | dclaw-chat backend env | User | ⏳ Pending |
| `APPLE_CERT` | GitHub org | User | ⏳ Deferred |
| `STRIPE_SECRET_KEY` | Backend env | User | ⏳ Pending |
| `DATABASE_URL` | Backend env | DevOps | ✅ Default set |

## PII Handling

**Rule:** All user data is considered PII until proven otherwise.

1. **Local-first:** Default to Ollama (local). Cloud fallback only with explicit user toggle.
2. **ClawShield:** Every outbound cloud call runs through PII detection/redaction.
3. **No storage:** Cloud LLM providers never receive identifiable information.
4. **Audit:** All PII operations logged with before/after hashes.

## Compliance Targets

| Standard | Target | Notes |
|----------|--------|-------|
| GDPR | P2 | Data residency, right to deletion |
| HIPAA | P2 | DClaw Med only, BAA required |
| SOC2 Type II | P3 | Enterprise customers |
| ISO 27001 | P4 | Global expansion |

## Reporting

Found a vulnerability? Do not open a public issue. Contact: security@dclawstack.io (placeholder)
