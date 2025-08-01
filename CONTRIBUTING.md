# Contributing to GigFlux

Thanks for your interest in improving GigFlux! This document outlines how we work.

## Values

- **Safety, then correctness, then clarity, then simplicity, then performance.**
- Community plugins thrive when APIs are stable and docs are crisp.

## Project setup

- Monorepo with `pnpm` and Turborepo.
- Code is TypeScript-strict across backend, frontend, and SDKs.
- Local services run via Docker Compose (Postgres 16 + pgvector, Redis, Mailpit, OTEL Collector).

> See repository README for current setup commands (scaffold in progress).

## Workflow

1. **Find/raise an issue** and discuss the approach.
2. **TDD**: write tests first (Vitest, Supertest, Playwright).
3. Implement minimal code to make tests pass.
4. Update docs inline (docblocks) and in `/docs` if needed.
5. Open a PR following our template.

### Branch naming

`feat/<scope>-<short-title>` · `fix/<scope>-<short-title>` · `chore/<scope>-<short-title>` · `docs/…` · `refactor/…`

### Commits

Use **Conventional Commits**:
- `feat(api): add JSON-RPC plugin host`
- `fix(web): restore keyboard focus after dialog close`
- `chore(ci): enable CodeQL`
- `docs: add plugin manifest spec`

### Pull requests

Every PR must include:
- **Problem** and **Solution** summary.
- **Tests**: unit and, if applicable, API/UI e2e.
- **Screenshots/GIFs** for UI.
- **Security/Privacy** notes (PII, secrets, ToS).
- **Follow-ups** (if you’ve cut scope).

We target PRs ≤ 300 LOC; if larger, justify and split where possible.

## Testing strategy

- **Unit**: Vitest — mock **ports**, not frameworks.
- **API**: Nest testing module + Supertest; validate request/response with Zod.
- **E2E/UI**: Playwright — keyboard navigation, roles, labels, and critical flows.
- **Contract tests**: for connectors; golden fixtures detect source or schema drift.
- **Coverage**: ≥ 90% for domain/plugin host; ≥ 80% elsewhere. Keep CI green.

## Style

- Small, pure functions; dependency injection over singletons.
- Zod schemas at IO boundaries; `env.ts` validates environment.
- Avoid logging PII/secrets; use structured logs with correlation IDs.

## Docs

- Update `/docs` and component READMEs.
- Add an **ADR** for any notable technology or architecture change (template in `/docs/adr/0000-template.md`).

## Code of Conduct

By participating you agree to the [Code of Conduct](./CODE_OF_CONDUCT.md).

## License

Unless stated otherwise, contributions to the **core** are under **AGPL-3.0**; contributions to SDKs/example plugins are under **MIT**.
