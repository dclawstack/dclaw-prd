# CONVENTIONS

## Git

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

## Go (Operator)

- `gofmt` enforced
- Idiomatic error handling
- Structured logging with `slog` or `zap`
- Table-driven tests
- No `panic` in production code

## TypeScript / Next.js

- Strict TypeScript (`strict: true`)
- Functional components with hooks
- Co-locate hooks with components
- Tailwind for all styling
- `cn()` utility for conditional classes
- No `any` types without explicit `@ts-ignore` comment

## Python (FastAPI)

- PEP 8 formatting (`ruff`)
- Type hints on all public APIs
- `pydantic` v2 for models
- `sqlalchemy` 2.0 style (Mapped, mapped_column)
- `pytest` with `pytest-asyncio`
- Docstrings on all routers and services

## General

- Keep functions focused and small (<50 lines)
- Write tests for business logic
- Run linter before committing
- Update `AGENTS.md` or `STATUS.md` when architecture changes
