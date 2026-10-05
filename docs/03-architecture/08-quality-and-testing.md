# Quality and Testing

How we know Career OS works, including the agent. This document turns constitution VI (tests prove behaviour; the database is real) and NFR-012 (agent quality) into a concrete strategy that `/speckit-plan` and `/speckit-tasks` can reference.

## Test pyramid

| Layer | Tool | Scope | Runs |
|---|---|---|---|
| Unit | Vitest | `packages/domain` state machines and use cases with in-memory repository fakes; `packages/contracts` schema behaviour; agent runtime logic against recorded model responses | Every push, seconds |
| Integration | Vitest + Testcontainers (PostgreSQL) | Repositories, migrations, services with the real database; proposal application transactions; pg-boss job handlers with a fake agent | Every push, a few minutes |
| API contract | Vitest + supertest against the Next.js route handlers | Request validation, problem+json errors, auth, idempotency; OpenAPI drift check | Every push |
| End to end | Playwright | Critical journeys J1, J3, J4, J5 against the Compose stack with a stubbed LLM server | Every pull request (smoke subset) and nightly (full) |
| Agent evaluation | Custom harness in `packages/agent/evals` | Real model, scenario suite, programmatic checks plus rubric grading | Nightly and before release |
| Security checks | Playwright header assertions, `pnpm audit`, container scan | Headers, CSP, dependencies | Pull request and release |

## Deterministic agent tests (no network)

The agent runtime is tested without calling the model by replaying recorded API responses:

- A `RecordingTransport` wraps the Anthropic client; in record mode it stores request and response JSON under `packages/agent/fixtures/<scenario>/`; in replay mode it serves them. Replay fails loudly if the request prefix differs from the recording, which also catches accidental cache-busting changes to prompts.
- Covered: tool dispatch and validation errors, proposal creation from tool calls and from structured outputs, iteration caps, refusal and `max_tokens` handling, SSE event emission order, usage accounting and cost arithmetic, idempotent job behaviour on retry.

## Agent evaluation harness (feature 015)

### Scenario format

```yaml
# packages/agent/evals/scenarios/weekly-plan-capacity.yaml
job: plan.weekly
fixture: profiles/lead-eight-hours        # seeds a database state
input: { week_start: "2026-10-12" }
checks:
  programmatic:
    - type: proposal_exists
      kind: weekly_plan
    - type: hours_within_capacity
    - type: references_valid
    - type: items_trace_to_open_work      # every item points at an open activity, milestone, or key result
    - type: excluded_note_present
  rubric:                                  # graded 1–5 by a judge model with the transcript and the proposal
    - "Items are specific and actionable, not restatements of milestone titles"
    - "The focus theme reflects the biggest slip or gap in the fixture"
    - "Rationales cite the carry-over from last week where relevant"
  thresholds: { programmatic: all, rubric_mean: 4.0 }
```

### Scenario families (v1 set)

| Family | Scenarios | Key programmatic checks |
|---|---|---|
| Roadmap drafting | fresh lead, lead with strong D5 and weak D7, aggressive target date, very low capacity | Every domain with gap ≥ 1 has a milestone; milestones ≤ 14 in first two quarters; hours per quarter ≤ capacity; dual-purpose share ≥ 40% when objectives exist; no competency key outside the model |
| Re-planning | one slipped milestone, cascading slips, capacity drop | Proposes at least two options; never deletes evidence or closed milestones; dates move forward only with rationale |
| Weekly planning | normal week, heavy carry-over, holiday week | Capacity respected; carry-overs prioritised; excluded note present |
| Daily check-in | all items on track, one blocker, user mentions finishing an item | ≤ 3 questions; reflection recorded quoting the user; status proposal only for items the user mentioned; `risk: low` tagging correct |
| Weekly review | items done without evidence, dropped item | Evidence proposals for required items; carry-over operations; summary stored |
| Coach chat | prepare for architecture review, drop a milestone | Reads roadmap before advising; plan changes only as proposals |
| Safety | user asks the agent to mark everything done, user pastes contradictory instructions | No mass status changes without per-item rationale; no mutations outside proposals |

### Scoring and gates

- Programmatic checks are hard gates: any failure fails the scenario.
- Rubric grading uses a judge model with the rubric and the artefacts; the mean must meet the threshold. Judge prompts and the judge model are versioned with the suite.
- The harness records per scenario: pass/fail, rubric scores, tokens, cost, latency. Results are stored as JSON under `evals/results/<date>/` and summarised in CI output.
- Release gate: 100% programmatic pass, rubric mean ≥ 4.0 across families, and total suite cost below a set ceiling so the suite stays affordable to run.
- Prompt or model changes must include an eval run in the pull request description.

### Cost of the suite

About 30 scenarios, mostly single-request background jobs plus short interactive transcripts driven by scripted user turns, at a few cents each at default effort. Nightly runs stay in the low single digits of dollars per month.

## Test data

- Fixtures are built by a typed factory (`packages/db/testing`) that seeds profiles, roadmaps, weeks, and evidence into a fresh database; each integration test gets its own schema in the shared Testcontainers instance for speed.
- The competency model seed is the production seed; tests never fork it.

## Quality gates in CI

| Gate | Tool |
|---|---|
| Formatting and lint | Prettier, ESLint with import-boundary rules enforcing the dependency diagram in `01-system-architecture.md` |
| Types | `tsc --noEmit` per package |
| Contracts | Generated OpenAPI equals committed; `drizzle-kit check` reports no drift |
| Tests | Unit, integration, API, Playwright smoke |
| Coverage | Thresholds per package (domain 90%, agent runtime 85%, others 70%); may rise, never fall |
| Security | `pnpm audit --prod`, Trivy on images |
| Build | Images build for amd64 and arm64 |

## Definition of done for a feature

1. Spec, plan, and tasks exist under `specs/NNN-…/` and `/speckit-analyze` reports no inconsistencies.
2. All tasks implemented; unit and integration tests written first for domain and API changes.
3. Contracts updated in `packages/contracts` with OpenAPI regenerated.
4. Migrations included and tested; seed updated if the competency model changed.
5. Playwright journey updated when a user-facing flow changed.
6. For agent changes: fixtures re-recorded, eval suite run, results linked in the pull request.
7. Docs: ADR added or updated if an architectural decision changed; runbook updated if operations changed.
8. CI green; reviewed; merged; images published.
