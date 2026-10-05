# Spec Kit Handoff Guide

How to turn this `docs/` folder into working software with GitHub Spec Kit. This guide assumes Claude Code as the coding agent, but Spec Kit supports others; only the integration key changes.

Spec Kit command names were hyphenated (`/speckit-specify`) at the time of writing; older versions used dots (`/speckit.specify`). Check `specify --help` for the version you install.

## 1. Install and initialise

```bash
# prerequisites: Python 3.11+, uv, and a supported coding agent
# (check the Spec Kit README for the current install command and integration keys)
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# in the repository root (this repo), initialise Spec Kit for Claude Code
specify init . --integration claude
```

This creates `.specify/` (memory, templates, scripts, `feature.json`) and the agent's command files. Commit them. Nothing in `docs/` needs to move; Spec Kit writes its own artefacts to `.specify/` and `specs/`.

## 2. Constitution

Run `/speckit-constitution` and paste the content of `docs/01-constitution/constitution.md` as the input, prefaced by:

> Use the following as the project constitution. Keep the nine principles, the additional constraints, the development workflow, and the governance section. Fill the compliance checks into the template's gates. Version 1.0.0, ratified 2026-10-05.

Verify the result in `.specify/memory/constitution.md`. If the template added default principles (for example library-first or CLI-interface articles), remove them: Career OS is a web application and those do not apply.

## 3. Per feature: specify → clarify → plan → tasks → analyze → implement

Work through `docs/02-requirements/02-feature-breakdown.md` in the listed order. For each feature:

### 3.1 `/speckit-specify`

Paste the feature's prompt from the feature breakdown. Add one line at the end:

> Requirement IDs covered: <FR and NFR ids from the feature table>. User stories and acceptance criteria are in docs/02-requirements/01-product-requirements.md; reuse their wording. Domain terms must follow docs/00-discovery/02-glossary.md.

Spec Kit creates `specs/NNN-<name>/spec.md`. Check that it contains no technology choices; if it does, delete them (they belong in the plan).

### 3.2 `/speckit-clarify`

Run until no `[NEEDS CLARIFICATION]` remains. Use `docs/05-spec-kit-handoff/02-open-questions.md` as the answer sheet; record answers back into that file so later features inherit them.

### 3.3 `/speckit-plan`

Use this prompt, adjusting the feature-specific pointers:

> Technical context: follow docs/03-architecture/01-system-architecture.md (monorepo layout, module boundaries, technology stack), docs/03-architecture/03-data-model.md (tables for this feature), docs/03-architecture/04-api-contracts.md (routes, SSE events, operation schema), and for agent features docs/03-architecture/02-agent-architecture.md. Honour ADRs 0001 to 0010 in docs/04-decisions/; list the ones this feature relies on in the constitution check. Testing per docs/03-architecture/08-quality-and-testing.md: unit tests for domain, Testcontainers for anything touching PostgreSQL, Playwright for the user journey, replay fixtures for agent runtime. Deployment per docs/03-architecture/07-deployment-and-operations.md; no new runtime components. Generate research.md only for genuinely open technical questions; data-model.md and contracts/ must be derived from the referenced docs, not invented.

Review `plan.md`, `research.md`, `data-model.md`, `contracts/`, and `quickstart.md`. Reject anything that contradicts an ADR unless you intend to supersede it; if so, write the new ADR first.

### 3.4 `/speckit-checklist` (optional) and `/speckit-tasks`

Generate tasks. Check that tasks follow the test-first order from the constitution and that parallelisable tasks are marked. Agent features must include tasks for fixtures, replay tests, and at least one evaluation scenario.

### 3.5 `/speckit-analyze`

Run before implementing. Fix inconsistencies between spec, plan, and tasks; this is where ADR conflicts surface.

### 3.6 `/speckit-implement`

Implement on a feature branch. Definition of done is in `docs/03-architecture/08-quality-and-testing.md`. Open a pull request; CI must be green.

## 4. Recommended first milestone: the walking skeleton

Deliver features 001, 003, and 004 first, with the Compose stack running end to end and a stubbed agent that emits a fixed proposal. This proves authentication, the data model core, the proposal flow, the worker, and the deployment before any model call is made. Then 005 (roadmap drafting) is the first real agent feature, and 015 (evaluation harness) should follow immediately after.

## 5. Mapping of documents to Spec Kit artefacts

| Spec Kit artefact | Primary source in `docs/` |
|---|---|
| `.specify/memory/constitution.md` | `01-constitution/constitution.md` |
| `specs/NNN/spec.md` | `02-requirements/02-feature-breakdown.md` (prompt), `02-requirements/01-product-requirements.md` (stories, FRs, NFRs), `00-discovery/03-user-journeys.md` |
| `specs/NNN/plan.md` | `03-architecture/01-system-architecture.md`, `04-decisions/*` |
| `specs/NNN/research.md` | `03-architecture/02-agent-architecture.md` (API behaviours), `04-decisions/ADR-0004`, `ADR-0006` |
| `specs/NNN/data-model.md` | `03-architecture/03-data-model.md` |
| `specs/NNN/contracts/` | `03-architecture/04-api-contracts.md` |
| `specs/NNN/quickstart.md` | `00-discovery/03-user-journeys.md`, `03-architecture/07-deployment-and-operations.md` |
| `specs/NNN/checklists/` | `03-architecture/08-quality-and-testing.md` (definition of done), `03-architecture/06-security-and-privacy.md` (release checklist) |

## 6. Keeping the docs alive

- When a plan changes a decision, add or supersede an ADR in the same pull request.
- When the data model or contracts change during implementation, update `03-architecture/03-data-model.md` and `04-api-contracts.md`; the per-feature copies in `specs/` are derived.
- After the first quarter of real use, revisit `00-discovery/01-vision-and-scope.md` success criteria and the agent evaluation thresholds.

## 7. Things Spec Kit will ask that are already decided

| Question Spec Kit tends to raise | Answer |
|---|---|
| Language and framework | TypeScript, Next.js, Node worker (ADR-0001, ADR-0002) |
| Storage | PostgreSQL only, Drizzle, pg-boss (ADR-0003, ADR-0009) |
| Testing approach | Vitest, Testcontainers, Playwright, agent evals (`08-quality-and-testing.md`) |
| Target platform | Docker Compose on a self-hosted host (ADR-0010) |
| Project type | Web application, single user |
| Performance goals | NFR-003, NFR-004 |
| Constraints | Constitution additional constraints; NFR-001 privacy |
| Scale | One user; small data; no horizontal scaling |
