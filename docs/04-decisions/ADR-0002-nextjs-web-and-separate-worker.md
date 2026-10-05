# ADR-0002: Next.js for UI and API, separate Node worker, JSON API with generated OpenAPI

**Status:** Accepted · **Date:** 2026-10-05

## Context

The product needs a phone-friendly web UI, a JSON API, real-time streaming of agent turns, and background jobs that run when the user is absent. Spec Kit expects explicit API contracts per feature.

## Decision

- **`apps/web`:** Next.js (App Router) serves the UI and the versioned JSON API under `/api/v1` as route handlers, including Server-Sent Events for interactive agent sessions. Interactive agent turns run inside `web` because they stream to the browser.
- **`apps/worker`:** a plain Node process that owns schedules and background jobs through pg-boss, including background agent jobs, reminders, usage roll-ups, and budget checks.
- **API style:** JSON over HTTP with request and response shapes defined as Zod schemas in `packages/contracts`; OpenAPI is generated from Zod and committed; `application/problem+json` for errors; SSE for streams.

## Alternatives considered

- **tRPC.** Attractive for a solo TypeScript developer, but contracts become implicit in procedure types, which fits Spec Kit's `contracts/` artefact poorly and makes non-browser clients (scripts, a future mobile shell) harder. Rejected for v1.
- **GraphQL.** Overkill for one client and one user.
- **Separate API server (Fastify or Hono) plus a static SPA.** Clean separation but one more deployable and duplicated auth plumbing. Next.js route handlers already give an API in the same process. If API concerns outgrow route handlers, a Hono app mounted under `/api` is a contained change.
- **Run everything in `web` with in-process timers.** Rejected: timers die with restarts, no retries, no single-execution guarantee, and long agent jobs compete with request handling.

## Consequences

- Two processes to deploy, both from the same codebase and sharing the `domain` and `agent` packages.
- Streaming works without WebSockets infrastructure; SSE reconnect uses `Last-Event-ID`.
- A web restart interrupts an interactive turn; the design stores messages before calling the model so the session resumes cleanly.
- OpenAPI drift is caught in CI (constitution V).

## Revisit triggers

- Need for bidirectional real-time features beyond streaming (would justify WebSockets).
- API consumers outside the browser (would favour moving the API into a standalone service).
