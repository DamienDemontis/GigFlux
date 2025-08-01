# GigFlux API (NestJS)

## Goals
Clean hexagonal architecture; deterministic tests; explainable matching.

## Tech
- NestJS, Drizzle ORM, Postgres 16 + pgvector, Redis/BullMQ, Zod
- Vitest (unit), Supertest (HTTP), OpenTelemetry

## Key Modules (planned)
- `jobs/` — ingestion, normalization, storage
- `profiles/` — resumes, preferences
- `matching/` — embeddings, scoring, reasons
- `plugins/` — JSON-RPC host and registry
- `alerts/` — digests and webhooks
- `health/` — status endpoints

## Conventions
- Ports in `core/ports/*`, adapters in `infra/*`.
- Validate IO with Zod; no framework types in domain code.

## Scripts (placeholder)
See root README once scaffolded.
