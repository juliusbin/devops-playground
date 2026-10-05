# Architecture Decision Records

ADRs record architecture-significant decisions with their context, alternatives, and consequences. Spec Kit plans must list the ADRs they rely on in their "Constitution check" section (constitution VIII). A plan that needs to deviate supersedes the ADR explicitly with a new one.

Format: Status · Date · Context · Decision · Alternatives considered · Consequences · Revisit triggers.

| ADR | Title | Status |
|---|---|---|
| [0001](ADR-0001-modular-monolith-typescript-monorepo.md) | Modular monolith in a TypeScript monorepo | Accepted |
| [0002](ADR-0002-nextjs-web-and-separate-worker.md) | Next.js for UI and API, separate Node worker, JSON API with generated OpenAPI | Accepted |
| [0003](ADR-0003-postgres-single-datastore-and-job-queue.md) | PostgreSQL as the single datastore, including the job queue | Accepted |
| [0004](ADR-0004-agent-harness-anthropic-sdk-tool-runner.md) | Agent harness: Anthropic TypeScript SDK tool runner | Accepted |
| [0005](ADR-0005-propose-then-apply-for-agent-writes.md) | Propose-then-apply for all agent writes | Accepted |
| [0006](ADR-0006-model-and-inference-policy.md) | Model and inference policy | Accepted |
| [0007](ADR-0007-single-user-auth-and-private-exposure.md) | Single-user authentication and private network exposure | Accepted |
| [0008](ADR-0008-manual-evidence-capture-integration-ports-deferred.md) | Manual evidence capture; integration ports deferred | Accepted |
| [0009](ADR-0009-drizzle-orm-and-sql-migrations.md) | Drizzle ORM with SQL migrations | Accepted |
| [0010](ADR-0010-docker-compose-first-kubernetes-later.md) | Docker Compose first, Kubernetes later | Accepted |

## Constitution amendment log

| Date | Version | Change |
|---|---|---|
| 2026-10-05 | 1.0.0 | Initial constitution |
