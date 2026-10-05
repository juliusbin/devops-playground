# Career OS Constitution

## Core Principles

### I. Private by default, one user, own infrastructure

- All user data MUST live in the self-hosted PostgreSQL database and the local attachment volume.
- Third-party analytics, telemetry, and error-reporting services MUST NOT receive user content.
- Requests to the LLM provider MUST carry only the content the task needs, and every such request
  MUST be recorded.
- There is exactly one user in v1. Code MUST NOT assume more than one user, and MUST NOT build
  multi-tenancy. See ADR-0007.

**Compliance check**: no outbound network call except to the configured LLM API endpoint and, if
enabled, the user's own SMTP or push server. Verified by an integration test that runs the app
against a deny-all egress policy with only the LLM host allowed.

### II. The agent proposes, the user decides

- Any change to roadmap, milestones, activities, objectives, key results, weekly plans, or evidence
  that originates from the agent MUST be expressed as a proposal with a per-operation rationale,
  and MUST be applied only after the user's explicit approval.
- Append-only records (reflections, check-in transcripts, agent notes, usage) MAY be written
  directly.
- The optional "auto-apply low-risk operations" setting is the only exception. It MUST be off by
  default, it covers only the two operation types ADR-0005 names (plan item status and key
  result progress, when the user stated the fact in the same conversation), and it auto-approves
  a proposal rather than bypassing the proposal path.

Rationale: the agent's output is probabilistic; the roadmap and the plans are the user's
commitments. See ADR-0005.

**Compliance check**: domain services that mutate plan state accept an `actor` argument; when
`actor` is `agent`, the only permitted path is `applyApprovedProposal`. Enforced by a unit test
over the service layer.

### III. Explainable and auditable agent

- Every proposal MUST carry the references (check-in, reflection, evidence, and milestone IDs) it
  was derived from.
- Every agent session MUST record the model, the effort, each request's token usage, cache hits,
  cost, and stop reason, and every tool call with its input and a result summary.
- The user MUST be able to read all of it in the UI.

**Compliance check**: schema constraints (`NOT NULL`) on `agent_sessions` usage columns and
`proposals.rationale`; a test that a session with missing usage rows cannot be marked completed.

### IV. Simplicity: one web app, one worker, one database

- The system is a modular monolith: a Next.js application, a Node worker, and PostgreSQL.
- v1 MUST NOT add a message broker, a cache server, a search engine, or microservices.
- Any new runtime component MUST have an ADR that names the problem the existing three cannot
  solve. See ADR-0001, ADR-0003, and ADR-0010.

**Compliance check**: `docker-compose.yml` services are limited to `web`, `worker`, `postgres`,
`caddy`, and an optional `backup` sidecar. A new service fails review without a linked ADR.

### V. Contracts first, typed end to end

- Zod schemas in `packages/contracts` are the single source of truth for API request and response
  shapes, agent tool inputs and outputs, proposal operations, and SSE events.
- OpenAPI MUST be generated from those schemas, never hand-written.
- UI, API, worker, and agent MUST import the same types.
- Database schema changes MUST ship with a migration in the same change. See ADR-0002 and
  ADR-0009.

**Compliance check**: CI fails if the generated OpenAPI document differs from the committed one, or
if `drizzle-kit check` reports schema drift.

### VI. Tests prove behaviour; the database is real

- Domain logic MUST have unit tests.
- Anything touching PostgreSQL MUST be tested against a real PostgreSQL started by Testcontainers,
  never a mock or SQLite.
- Agent behaviour MUST be tested two ways: deterministic unit tests on recorded fixtures (no
  network), and a scenario-based evaluation suite with rubric grading that runs on a schedule, not
  on every pull request.
- Domain and API work MUST follow red-green ordering: write the failing test first, then
  implement.

**Compliance check**: CI runs unit and integration suites on every pull request; coverage
thresholds are set per package in the repo and MUST NOT decrease.

### VII. Observable and cost-bounded

- Logs MUST be structured JSON with a correlation ID per request and per agent session.
- LLM spend MUST be aggregated daily and compared with the monthly budget. When the budget is
  reached, background agent jobs MUST pause and the user MUST be notified.
- Interactive sessions MUST warn before exceeding a per-session cap. See ADR-0006.

**Compliance check**: a test that a budget breach pauses the weekly plan job and creates a
notification.

### VIII. Decisions are written down before they are built

- Architecture-significant choices MUST be recorded as ADRs in `docs/04-decisions/` before
  implementation.
- Architecture-significant means at least: a new runtime component, datastore, language, or
  external service; a change to the agent harness, the authentication model, or the data model
  of a core entity.
- Every feature MUST follow the Spec Kit flow: specification, clarification, plan, tasks,
  implementation.
- A plan that contradicts an ADR MUST either be changed or supersede the ADR explicitly.

**Compliance check**: `/speckit-analyze` is run before `/speckit-implement` for every feature; the
plan's "Constitution check" section lists the ADRs it relies on.

### IX. Portable and recoverable

- The whole system MUST start from `docker compose up` with a single `.env`.
- Backups MUST run automatically at least daily, MUST be retained for at least 30 days, and
  MUST be encrypted at rest when stored off-host. Restore MUST be tested at least once per
  release.
- Data MUST be exportable in open formats (JSON and the original attachments) without the
  application running.

**Compliance check**: a release checklist item "restore last backup into a clean stack and log in"
MUST be ticked.

## Additional Constraints

- **Language and runtime**: TypeScript (strict) on the current Node.js LTS everywhere: UI, API,
  worker, agent runtime, scripts. No second backend language without an ADR.
- **Accessibility and devices**: the UI MUST be usable on a phone browser and with keyboard only.
  Check-ins in particular are often done on a phone. Compliance check: the Playwright journeys
  for the daily check-in, status update, and evidence capture run at a phone viewport and with
  keyboard only, and colour contrast meets WCAG 2.1 AA (NFR-008, NFR-009).
- **LLM policy**: model, effort, and budget are configuration, not code. Prompts and tool
  definitions MUST be versioned in the repository. See ADR-0006.
- **Secrets**: never in the repository, never in images. Secrets MUST be injected at runtime
  through environment variables or Docker secrets.

## Development Workflow

Every feature follows these steps in order.

1. Pick the next feature from `docs/02-requirements/02-feature-breakdown.md`.
2. Run `/speckit-specify` with that feature's prompt, then `/speckit-clarify` until no
   open clarification marker remains.
3. Run `/speckit-plan` with the technical context from `docs/03-architecture/`, listing the
   relevant ADRs.
4. Run `/speckit-tasks`, `/speckit-analyze`, then `/speckit-implement`.
5. Open a pull request with passing CI; merge; update ADR status if a decision changed.

## Governance

This constitution supersedes ad hoc practice.

Amendments require all of the following:

- a pull request that edits this file;
- a version bump according to the policy below;
- a note in the constitution amendment log in `docs/04-decisions/README.md`.

| Bump | Applies when |
|---|---|
| MAJOR | A principle or governance rule is removed or redefined |
| MINOR | A principle or section is added, or guidance is materially expanded |
| PATCH | Clarifications, wording, typo fixes, and other non-semantic refinements |

Reviews check pull requests against the compliance checks above. Unexplained violations block
merge.

**Version**: 1.0.0 | **Ratified**: 2026-10-05 | **Last Amended**: 2026-10-05
