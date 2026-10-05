# Career OS Constitution

**Purpose of this file:** paste-ready input for `/speckit-constitution`. Spec Kit will write the result to `.specify/memory/constitution.md` and use it as a gate in every plan. Keep principles few, testable, and phrased so an automated review can check compliance.

**Version:** 1.0.0 · **Ratified:** 2026-10-05 · **Last amended:** 2026-10-05

## Core principles

### I. Private by default, one user, own infrastructure
All user data lives in the self-hosted PostgreSQL database and local attachment volume. No third-party analytics, telemetry, or error-reporting services receive user content. Only the content a task needs is sent to the LLM provider, and every such request is recorded. There is exactly one user in v1; code must not assume more, but must not build multi-tenancy either.

*Compliance check:* no outbound network call except to the configured LLM API endpoint and, if enabled, the user's own SMTP or push server. Verified by an integration test that runs the app against a deny-all egress policy with only the LLM host allowed.

### II. The agent proposes, the user decides
Any change to roadmap, milestones, activities, objectives, key results, weekly plans, or evidence that originates from the agent MUST be expressed as a proposal with per-operation rationale and MUST be applied only after the user's explicit approval. Append-only records (reflections, check-in transcripts, agent notes, usage) may be written directly. The optional "auto-apply low-risk operations" setting is the only exception and is off by default.

*Compliance check:* domain services that mutate plan state accept an `actor` argument; when `actor` is `agent`, the only permitted path is `applyApprovedProposal`. Enforced by a unit test over the service layer.

### III. Explainable and auditable agent
Every proposal carries the references (check-in, reflection, evidence, milestone IDs) it was derived from. Every agent session records the model, effort, each request's token usage, cache hits, cost, stop reason, and every tool call with its input and a result summary. The user can read all of it in the UI.

*Compliance check:* schema constraints (`NOT NULL`) on `agent_sessions` usage columns and `proposals.rationale`; a test that a session with missing usage rows cannot be marked completed.

### IV. Simplicity: one web app, one worker, one database
The system is a modular monolith: a Next.js application, a Node worker, and PostgreSQL. No message broker, no cache server, no search engine, no microservices in v1. Adding any new runtime component requires an ADR that names the problem the existing three cannot solve.

*Compliance check:* `docker-compose.yml` services are limited to `web`, `worker`, `postgres`, `caddy`, and an optional `backup` sidecar. A new service fails review without a linked ADR.

### V. Contracts first, typed end to end
Zod schemas in `packages/contracts` are the single source of truth for API request and response shapes, agent tool inputs and outputs, proposal operations, and SSE events. OpenAPI is generated from them, never hand-written. UI, API, worker, and agent import the same types. Database schema changes ship with a migration in the same change.

*Compliance check:* CI fails if the generated OpenAPI document differs from the committed one, or if `drizzle-kit check` reports schema drift.

### VI. Tests prove behaviour; the database is real
Domain logic has unit tests. Anything touching PostgreSQL is tested against a real PostgreSQL started by Testcontainers, never a mock or SQLite. Agent behaviour is tested two ways: deterministic unit tests on recorded fixtures (no network), and a scenario-based evaluation suite with rubric grading that runs on a schedule, not on every pull request. Red-green ordering is expected: write the failing test first for domain and API work.

*Compliance check:* CI runs unit and integration suites on every pull request; coverage thresholds are set per package in the repo and must not decrease.

### VII. Observable and cost-bounded
Logs are structured JSON with a correlation ID per request and per agent session. LLM spend is aggregated daily and compared with the monthly budget; when the budget is reached, background agent jobs pause and the user is notified. Interactive sessions warn before exceeding a per-session cap.

*Compliance check:* a test that a budget breach pauses the weekly plan job and creates a notification.

### VIII. Decisions are written down before they are built
Architecture-significant choices are recorded as ADRs in `docs/04-decisions/` before implementation. Every feature follows the Spec Kit flow: specification, clarification, plan, tasks, implementation. A plan that contradicts an ADR must either be changed or supersede the ADR explicitly.

*Compliance check:* `/speckit-analyze` is run before `/speckit-implement` for every feature; the plan's "Constitution check" section lists the ADRs it relies on.

### IX. Portable and recoverable
The whole system starts from `docker compose up` with a single `.env`. Backups are automated, encrypted at rest when stored off-host, and restore is tested at least once per release. Data can be exported in open formats (JSON and the original attachments) without the application running.

*Compliance check:* a release checklist item "restore last backup into a clean stack and log in" must be ticked.

## Additional constraints

- **Language and runtime:** TypeScript (strict) on the current Node.js LTS everywhere: UI, API, worker, agent runtime, scripts. No second backend language without an ADR.
- **Accessibility and devices:** The UI must be usable on a phone browser and with keyboard only. Check-ins in particular are often done on a phone.
- **LLM policy:** Model, effort, and budget are configuration, not code. Prompts and tool definitions are versioned in the repository. See ADR-0006.
- **Secrets:** Never in the repository, never in images; injected at runtime through environment or Docker secrets.

## Development workflow

1. Pick the next feature from `docs/02-requirements/02-feature-breakdown.md`.
2. `/speckit-specify` with that feature's prompt, then `/speckit-clarify` until no `[NEEDS CLARIFICATION]` remains.
3. `/speckit-plan` with the technical context from `docs/03-architecture/`, listing relevant ADRs.
4. `/speckit-tasks`, `/speckit-analyze`, then `/speckit-implement`.
5. Pull request with passing CI; merge; update ADR status if a decision changed.

## Governance

This constitution supersedes ad hoc practice. Amendments require: a pull request that edits this file, a bumped version (MAJOR for removed or redefined principles, MINOR for added principles or sections, PATCH for wording), and a note in `docs/04-decisions/README.md`. Reviews check pull requests against the compliance checks above; unexplained violations block merge.
