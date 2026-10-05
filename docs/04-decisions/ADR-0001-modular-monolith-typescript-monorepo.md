# ADR-0001: Modular monolith in a TypeScript monorepo

**Status:** Accepted · **Date:** 2026-10-05

## Context

Career OS is built and operated by one person in spare time, serves one user, and has a domain with several cohesive modules (roadmap, objectives, planning, reflection, evidence, proposals, agent). The discovery decision fixed TypeScript end to end. The risk is not scale; it is complexity that one person cannot carry.

## Decision

Build a **modular monolith**: one repository (pnpm workspaces, Turborepo), two runnable applications (`web`, `worker`) and shared packages (`contracts`, `domain`, `db`, `agent`, `competency-model`, `ui`, `config`). Module boundaries live inside `packages/domain` and are enforced by lint rules on import paths, not by network boundaries.

## Alternatives considered

- **Microservices per module.** Rejected: operational cost and distributed-systems complexity with no benefit for one user.
- **Single Next.js app with everything inside.** Rejected: background jobs and long-running agent work do not fit the request lifecycle of a web server; a worker process is needed (ADR-0002). The monorepo keeps shared code without duplicating it.
- **Polyglot (Python agent, TypeScript web).** Rejected by the discovery decision; two languages double tooling, contracts, and context switching.

## Consequences

- One language, one test runner, one lint configuration, shared Zod types from UI to database.
- Module boundaries depend on discipline and lint rules; a dependency diagram in `03-architecture/01-system-architecture.md` is the reference.
- Extracting a module into a service later is possible because modules communicate through use-case interfaces, but it is not planned.

## Revisit triggers

- More than one developer regularly working in the same modules.
- A module needs a runtime the others cannot share (for example GPU or Python-only libraries).
