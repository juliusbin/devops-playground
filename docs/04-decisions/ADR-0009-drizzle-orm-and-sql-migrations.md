# ADR-0009: Drizzle ORM with SQL migrations

**Status:** Accepted · **Date:** 2026-10-05

## Context

Typed access to PostgreSQL from TypeScript, migrations that are reviewable and forward-only, and a thin layer that does not hide SQL from a developer practising data architecture.

## Decision

Use **Drizzle ORM** for schema definition and queries and **drizzle-kit** to generate SQL migration files that are committed and reviewed. Repositories in `packages/db` implement interfaces from `packages/domain`. Complex reads (dashboard, coverage) use SQL views or raw SQL through Drizzle's `sql` tag.

## Alternatives considered

- **Prisma.** Excellent developer experience, but its own schema language, a generated client, and a query engine binary add indirection; migrations less transparent.
- **Kysely or raw `pg`.** Maximum control, but more boilerplate for a solo developer; Drizzle is close to SQL while providing inferred types.
- **TypeORM / MikroORM.** Heavier, decorator-based; not needed.

## Consequences

- Schema and migrations live beside the code; `drizzle-kit check` in CI detects drift (constitution V).
- Generated migration SQL is reviewed like code; hand edits are allowed when generation is insufficient (for example partial unique indexes), with a comment.
- Integration tests run migrations against Testcontainers on every push.

## Revisit triggers

- Drizzle lacks a needed PostgreSQL feature with no `sql`-tag workaround (unlikely).
