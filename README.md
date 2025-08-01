# GigFlux

**GigFlux** is a self-hostable, open-core job-matching platform with a **plugin ecosystem** (Stremio-style).  
It ingests jobs from official sources (Greenhouse, Lever, Ashby, Workable, etc.), parses your resume, and delivers explainable matches with privacy at the core.

> Status: **Pre-alpha** — architecture and contributor docs are ready; first code is landing soon.

---

## Why GigFlux?

- **Privacy-first**: run it yourself; redact PII before storage.
- **Explainable matching**: transparent scores and “why this job” reasons.
- **Plugins as first-class citizens**: connectors and notifiers are swappable.
- **Open-core**: community features in the open; optional commercial add-ons later.

---

## Architecture (high level)

- **Hexagonal (Ports & Adapters)** — domain isolated from frameworks.
- **Backend**: NestJS · Postgres 16 + pgvector · Redis/BullMQ · Drizzle ORM · Zod validation.
- **Frontend**: Next.js (App Router) · shadcn/ui + Radix · Tailwind · TanStack Query · RHF + Zod.
- **Plugins**: out-of-process **JSON-RPC over stdio**; manifest + allow-listed networking via Fetch Proxy.
- **Observability**: OpenTelemetry → OTEL Collector.

See `/docs/architecture.md` (coming soon) and `/docs/adr` for decision records.

---

## Roadmap (milestones)

1. Monorepo scaffold (web, api, sdk, plugins) with tests-first.
2. JSON-RPC plugin host + **Greenhouse** connector.
3. Resume parsing pipeline (spaCy + embeddings + Presidio redaction).
4. Matching v1 (semantic + rules) with explainability.
5. Alerts (email + webhooks) and connector health dashboard.

Track progress in GitHub projects / milestones.

---

## Contributing

We welcome issues, discussions, and PRs. Please read:

- [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)
- [CONTRIBUTING.md](./CONTRIBUTING.md)
- [SECURITY.md](./SECURITY.md)

**Highlights**
- TDD required for new code.
- Conventional Commits.
- Zod validation at all external boundaries.
- A11y and keyboard support for UI work.

---

## License

- **Core**: AGPL-3.0 (see `LICENSE`).
- **SDKs & example plugins**: MIT (each package will include its own `LICENSE`).

© GigFlux Contributors.
