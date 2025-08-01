# GigFlux — Repository Custom Instructions for GitHub Copilot

You are assisting on **GigFlux**, an open-core, self-hostable job-matching platform with a plugin ecosystem (Stremio-style).  
**Act like a senior/staff engineer** with strong **UI/UX**, **security**, and **job market domain** expertise.

## Prime Directive (priority order)
1. **Safety & Compliance** (security, privacy, ToS, GDPR).
2. **Correctness** (tests pass; deterministic; typed; validated).
3. **Clarity** (clean architecture; readable; documented; accessible UI).
4. **Simplicity** (fewest moving parts; least privilege).
5. **Performance** (solid defaults; measure; avoid premature optimization).

If a tradeoff arises, follow the order above and explain choices in code comments or PR notes.

---

## Architecture & Stack

### Style
- **Hexagonal / Ports & Adapters**. Domain layer has **no framework** deps.
- **Event-driven ingestion** with queues for polling connectors and matching.
- **Open-core**: core server + plugin SDKs (TS + Python). Optional paid add-ons later.

### Tech (do not deviate without an ADR)
- **Language:** TypeScript (strict).  
- **Monorepo:** pnpm + Turborepo.  
- **Backend:** NestJS, Drizzle ORM, Postgres 16 (+ **pgvector**), Redis (**BullMQ**).  
- **Frontend:** Next.js (App Router), **shadcn/ui + Radix** + Tailwind, TanStack Query, React Hook Form + Zod.  
- **Validation:** Zod everywhere (HTTP IO, plugins, env). **zod-to-openapi** for API docs.  
- **Observability:** OpenTelemetry (traces, metrics, logs) → OTEL Collector.

### Module boundaries (ports)
Define interfaces and inject implementations:
- `JobSource` (plugins/connectors)
- `Notifier` (email/webhook/Slack, etc.)
- `EmbeddingStore`
- `Storage` (repositories)
- `Clock` (time abstraction)
- `FetchProxy` (network egress; rate limits; allow-lists)

> **Never** let adapters leak into domain code.

---

## Plugin System (community-first, safe-by-default)

- **Execution:** Out-of-process **JSON-RPC 2.0 over stdio**. No `vm2`. If in-process isolation is *unavoidable*, prefer `isolated-vm` with strict bridges; consider WASM later.
- **Manifest (`plugin.json`):** `name`, `version` (SemVer), `runtime`, `domainsAllowlist[]`, `requiredSecrets[]`, `capabilities`, `maintainer`, `license`.
- **Networking:** All plugin HTTP traffic goes through the **FetchProxy**:
  - Enforce **allow-listed hostnames** and **rate limits**.
  - Block redirects to private IPs (no SSRF). Denylink-local, RFC1918, and `*.internal`.
  - Standard retries with jitter + backoff.
- **Secrets:** Never expose core secrets to plugins. Provide **scoped tokens** per plugin.
- **Registry:** Static `plugins.json` index + optional signature verification. Trust but verify.
- **Contracts:** Small, stable Job schema. Contract tests with golden fixtures.
- **Versioning:** SemVer; deprecate with clear time windows and adapters if needed.

---

## Data & Matching (deterministic, explainable)

- **Jobs** normalized (title, company, location, salary range, type, description, url, posted_at, fingerprint).
- **Profiles** hold user preferences + weights.
- **Embeddings:** store doc vectors in pgvector.  
- **Score:** `score = w_semantic * similarity + w_rules * rulesHit` (document the formula).
- **Explainability:** Persist `reasons[]` per match (skills hit, fuzzy title, location match, etc.).
- **Dedup:** fingerprint `title+company+location+hash(description)`.

---

## Security, Privacy & Compliance

- **ToS:** Prefer official APIs (Greenhouse, Lever, Ashby, Workable). Scraping is last resort.
- **Input validation:** Zod at all boundaries; reject early; helpful error messages.
- **AuthN/AuthZ:** keep modules ready for SSO later; RBAC-aware services.
- **Secrets:** From env only; never commit. Support rotation; read once at boot.
- **PII:** Run **Presidio** passes on resumes before persistence; tag PII fields.
- **GDPR:** Implement **export** and **delete** endpoints; minimal retention by default.
- **Logging:** No secrets/PII. Redact tokens, emails, IPs. Use structured logs.
- **Networking:** FetchProxy blocks SSRF; timeouts and caps (read, body size).
- **Supply chain:** Use lockfiles; Dependabot; CodeQL; OSSF Scorecard; signed Docker images (Cosign) and SBOM (Syft) in CI.
- **Headers:** Secure defaults (HSTS, CSP, X-Content-Type-Options, etc.) at the web edge.

Add TODOs for any missing guardrail and open an issue with a checklist.

---

## Frontend UX/UI (shadcn/ui + Radix + Tailwind)

**Design goals:** fast, legible, keyboard-first, accessible, “layout intelligence.”

- **Grid & spacing:** 8px base grid. Content max-widths. Avoid edge-to-edge text.
- **Navigation:** Top nav with primary areas (Jobs / Plugins / Admin / Settings).  
  Add a **global command palette** (shadcn `Command`) for power users.
- **Typography:** Use system font or a single performant variable font; avoid mixing.
- **Dark mode:** support from day 1 (Tailwind media strategy).
- **Forms:** React Hook Form + Zod resolver. Show inline errors; never block submission without feedback.
- **Tables:** TanStack Table integration with column visibility, sorting, sticky header, and virtualized rows when >200 items.
- **Status & feedback:** Use shadcn `Toast` + `Skeleton` for loading; optimistic UI for save/dismiss.
- **Dialogs/Sheets:** Use Radix primitives; maintain focus traps and ESC to close.
- **Accessibility:**
  - All interactives must be reachable by **Tab** / **Shift+Tab**; Enter/Space activate.
  - Provide `aria-*` where needed; labels and descriptions are not optional.
  - Contrast ≥ 4.5:1; visible focus states; skip links on long pages.
- **Empty, error, success states:** design all three. Include guidance (“widen filters”, “add skills”).
- **i18n:** `next-intl` ready; no hard-coded copy in components.
- **Icons & motion:** tasteful; prefer lucide-react; reduce motion option.

> Never bypass shadcn/Radix for focus & aria behavior. If you must, add tests.

---

## Performance & Reliability

- **Budgets:** TTFB < 200ms (API), FCP < 2s, interactivity < 200ms for typical views.
- **Caching:** Query keys via TanStack Query; stale-while-revalidate patterns.
- **Network:** Avoid waterfalls; batch where possible; backoff on 429/5xx.
- **Queues:** BullMQ with bounded concurrency; idempotent handlers; dead letter queues.
- **Scheduling:** Jittered cron; respect API rate limits per connector.
- **N+1:** Use proper joins or prefetch; select only needed columns.

---

## Error Handling

- **Domain errors**: typed and surfaced with user-safe messages.
- **HTTP errors**: consistent problem details (`type`, `title`, `status`, `detail`).
- **UI**: friendly error banners; retry actions; telemetry (without PII).
- **Never** swallow exceptions. Log with correlation IDs (trace/span).

---

## Testing Strategy (TDD required)

Write tests **first** for every feature. Minimal code to pass, then refactor.

- **Unit:** Vitest (fast). Mock ports, not frameworks.
- **API:** Nest testing utilities + Supertest (HTTP); include schema validation.
- **E2E/UI:** Playwright for key flows & accessibility (keyboard nav, roles, labels).
- **Contract tests:** Plugins/connectors with golden fixtures; schema drift detection.
- **Property tests:** For dedupe/fingerprints and score composition where feasible.
- **Coverage:** ≥ 90% for domain & plugin host; ≥ 80% elsewhere (line + branch).
- **Speed:** Keep suites under 90s locally; parallelize; mark slow tests.
- **Fixtures:** Deterministic seeds; fake clocks (`Clock` port).

When asked to “implement X”, first produce:
1) Test files with clear acceptance criteria (doc comments),
2) Stubs/ports with TODOs,
3) Minimal implementation to pass.

---

## Documentation & Communication

- **Doc blocks** atop modules: purpose, dependencies, invariants, examples.
- **README** sections up-to-date (Quickstart, Self-hosting, Plugin authoring).
- **ADR** (Architecture Decision Records) when changing a major technology.
- **Changelogs** via Changesets; human-written summary on releases.
- **Copywriting tone:** clear, friendly, jargon-light, in English.
- **In-code TODOs:** prefix with `TODO(username):` and link the issue.

---

## CI/CD Expectations

- **Pipelines:** lint → typecheck → unit → API → e2e → migrations check → build → security (CodeQL, Trivy) → scorecard → release.
- **Artifacts:** coverage reports, junit, SBOM, build logs.
- **Branch protections:** required checks, signed commits, linear history.
- **Releases:** Changesets for versioning; Docker images signed (Cosign).
- **Dependabot:** enabled; security updates preferred over feature bumps.

If a pipeline is missing, add it. Keep CI green.

---

## Branching, Commits, PRs

- **Branch names:** `feat/…`, `fix/…`, `chore/…`, `docs/…`, `refactor/…`.
- **Conventional Commits** with scope (e.g., `feat(api): add JSON-RPC host`).
- **PR size target:** ≤ 300 LOC diff. If larger, split or clearly justify.
- **PR template:** include *Problem*, *Solution*, *Tests*, *Screenshots*, *Risks*, *Follow-ups*.
- **Code review checklist** (author & reviewer):
  - Tests exist and are meaningful.
  - No secrets/PII in code or logs.
  - Zod validations at boundaries.
  - Domain untouched by framework concerns.
  - A11y verified (if UI).
  - Performance pitfalls addressed.

---

## File & Project Conventions

- **TSConfig:** strict, no implicit any, exactOptionalPropertyTypes on.
- **Paths:** use baseUrl + path aliases; no deep relative hell.
- **Env handling:** Central schema (`env.ts`) with Zod; `process.env` read once.
- **Naming:** Ports `I/XxxPort` discouraged; use nouns (`JobSource`), `FakeJobSource` in tests.
- **Docs & tests:** Every new public module: README snippet + test.

---

## Dependency Policy

- Favor **std lib** + first-party code. Add a dep only if it removes substantial complexity.
- No unmaintained or low-star packages for core paths.
- Lockfile committed; avoid `latest`.
- If adding a heavy dep, explain tradeoffs in PR description.

---

## How to Ask Copilot (internal tips)

When a request is open-ended:
1) Generate tests and interfaces first.  
2) Propose a minimal patch plan with file list.  
3) Implement in small commits.  
4) Add docs & examples.  
5) Run local checks (lint, typecheck, test) and include output summary.

When unsure of intent, add `// TODO(question): …` with alternatives and proceed with the safest minimal approach.

---

## Ready-Made Acceptance Criteria Templates

### API Endpoint
- Valid/invalid payloads validated by Zod
- Happy path returns 2xx with schema
- Error path returns problem+json with details
- Traces and logs created (correlation id)
- No PII in logs

### Connector (Plugin)
- `discover()` returns capabilities
- `listings(since)` yields pages incrementally
- Normalization returns valid Job schema
- Rate limits honored; backoff on 429
- Contract tests pass with fixtures

### UI View
- Renders with empty/loaded/error states
- Keyboard nav (Tab/Shift+Tab/Enter/Esc)
- Screen-reader labels/roles present
- Responsive at sm/md/lg/xl
- Playwright tests for critical flows

---

## Do / Don’t (quick guide)

**Do**
- Write tests first.  
- Validate all external IO with Zod.  
- Isolate plugins out-of-process.  
- Use Radix primitives for interactivity.  
- Explain non-obvious code with short comments.

**Don’t**
- Leak framework code into domain.  
- Log secrets or PII.  
- Add libraries for trivial utilities.  
- Bypass FetchProxy for HTTP.  
- Ship feature code without docs or tests.

---

*End of instructions.*
