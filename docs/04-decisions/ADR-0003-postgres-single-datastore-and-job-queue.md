# ADR-0003: PostgreSQL as the single datastore, including the job queue

**Status:** Accepted · **Date:** 2026-10-05

## Context

The system needs durable storage, a job queue with cron scheduling and retries, keyword search over reflections, and eventually semantic search. Data volume is small (one user). Constitution IV limits runtime components.

## Decision

Use **PostgreSQL 17** for everything: domain data, auth sessions, settings, audit log, usage, and the **job queue via pg-boss** (cron, retries, singleton jobs, dead-letter handling). Keyword search uses PostgreSQL full-text search. Semantic search, when added, uses the pgvector extension in the same instance.

## Alternatives considered

- **Redis + BullMQ for jobs.** Mature, but adds a second stateful service to back up and monitor; pg-boss covers the needs at this scale with transactional job enqueueing alongside domain writes.
- **SQLite.** Simplest to run, but weaker concurrency between web and worker processes, no pg-boss, and a migration later would be painful.
- **Separate search engine.** Unnecessary for the data volume; PostgreSQL FTS and pgvector suffice.

## Consequences

- One backup, one restore procedure, one connection string.
- Jobs can be enqueued in the same transaction as the domain change that triggers them.
- pg-boss adds a `pgboss` schema; its tables are included in backups.
- Any future need for a cache or broker must be argued in an ADR (constitution IV).

## Revisit triggers

- Job throughput or latency needs beyond what polling a table provides (not expected for one user).
- Multi-user growth that makes a managed Postgres with read replicas attractive; the decision still holds, only the hosting changes.
