# Spec Kit Handoff Guide

How to turn this `docs/` folder into working software with GitHub Spec Kit. Spec Kit 1.1.1.dev0 is initialised in this repository for Claude Code. Every statement below about Spec Kit was checked against the installed files under `.claude/skills/` and `.specify/` and against the `specify` CLI help on 2026-10-05; re-check after upgrading Spec Kit. Other coding agents are supported; only the integration key changes.

Command names are hyphenated in this version: `/speckit-specify`, `/speckit-plan`, and so on. `.specify/integration.json` records the separator (`"invoke_separator": "-"`). Older Spec Kit versions used dots (`/speckit.specify`). The dotted form still appears inside the skill files as extension hook ids (for example `speckit.git.commit`), which the skills translate to `/speckit-git-commit` before running.

## 1. Installed state

### 1.1 Commands that were run

```bash
# install the CLI; uv puts it in ~/.local/bin, which must be on PATH
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
export PATH="$HOME/.local/bin:$PATH"

specify --version   # specify 1.1.1.dev0
specify check       # lists supported agents; Claude Code reported as available

# in the repository root: scripted init, no prompts, bash scripts, Claude Code skills
specify init --here --force --non-interactive --integration claude --script sh
```

`specify init` scaffolds from assets bundled in the CLI package, so templates match the CLI version and no network access is needed. For Claude Code it installs skills (`.specify/init-options.json` records `"ai_skills": true`), not slash-command files.

### 1.2 Resulting layout

| Path | Content | Notes |
|---|---|---|
| `.claude/skills/speckit-*/SKILL.md` | Ten skills, one per Spec Kit command (section 3.1) | Commit. Managed by the CLI; do not edit by hand |
| `.specify/memory/constitution.md` | Project constitution, read by `/speckit-plan`, `/speckit-analyze`, `/speckit-converge` | Commit. Filled: Career OS Constitution v1.0.0, mirrors `docs/01-constitution/constitution.md` (section 2) |
| `.specify/memory/.constitution-template.json` | Hash of the template that seeded the constitution file | Commit |
| `.specify/templates/` | `spec-template.md`, `plan-template.md`, `tasks-template.md`, `checklist-template.md`, `constitution-template.md` | Commit. Managed by the CLI |
| `.specify/scripts/bash/` | `create-new-feature.sh`, `setup-plan.sh`, `setup-tasks.sh`, `check-prerequisites.sh`, `common.sh`, `resolve-template.sh` | Commit. Managed by the CLI |
| `.specify/workflows/` | `speckit/workflow.yml` ("Full SDD Cycle") and `workflow-registry.json` | Commit. Section 4 |
| `.specify/integrations/` | `claude.manifest.json`, `speckit.manifest.json`: file hashes the CLI uses when refreshing | Commit |
| `.specify/init-options.json` | Choices made at init: `integration: claude`, `script: sh`, `feature_numbering: sequential`, `ai_skills: true`, `here: true` | Commit |
| `.specify/integration.json` | Installed integration (`claude`) and `invoke_separator` (`-`) | Commit |
| `.specify/.gitignore` | Spec Kit's own ignore file: `feature.json` and `extensions/*/local-config.yml` | Commit |
| `.specify/extensions/.cache/` | Extension catalog cache (four `catalog*.json` files) written by catalog lookups such as `specify extension search`, `specify extension add`, and `specify extension info` | Ignored by the root `.gitignore`; machine-local |
| `.specify/feature.json` | Pointer to the current feature directory, written by `/speckit-specify` | Machine-local, ignored by the file above. Does not exist until the first feature |
| `specs/` | One directory per feature, created by `/speckit-specify` | Commit. Does not exist yet |
| `.gitignore` (repository root) | Keeps `.claude/skills` committed; ignores `.claude/settings.local.json` and `.specify/extensions/.cache/` | Commit |

`specify init` does not commit anything. The scaffold (`.claude/`, `.specify/`, the root `.gitignore`) and the docs updates from this handoff were committed together on 2026-10-05 on branch `claude/happy-feynman-u77oqz`. Nothing in `docs/` needs to move; Spec Kit writes its own artefacts to `.specify/` and `specs/`.

## 2. Constitution

`.specify/memory/constitution.md` is installed and filled. It is the Career OS Constitution, version 1.0.0, ratified 2026-10-05, last amended 2026-10-05, with the nine principles and their compliance checks, Additional Constraints, Development Workflow, and Governance. It contains no `[ALL_CAPS]` placeholders and no Sync Impact Report. Its content mirrors `docs/01-constitution/constitution.md`, which remains the readable source; the Spec Kit copy states the same rules in MUST/SHOULD form with ADR references. `/speckit-plan` reads it to fill the Constitution Check gates, `/speckit-analyze` reports conflicts with it as CRITICAL, and `/speckit-converge` checks the implemented code against it.

Do not run `/speckit-constitution` to seed it. The skill loads an existing file as the current constitution and, per its own step 6, overwrites it with a regenerated version: a Sync Impact Report is prepended, Last Amended moves to the run date if anything changed, and the version may be bumped. That would replace a reviewed file with a generated one.

Before the first feature:

1. Confirm the state. `grep -n '^\*\*Version\*\*' .specify/memory/constitution.md` prints `**Version**: 1.0.0 | **Ratified**: 2026-10-05 | **Last Amended**: 2026-10-05`, and `grep -n '\[[A-Z_]\+\]' .specify/memory/constitution.md` prints nothing. Read the two constitution files side by side if anything looks off.
2. It is committed with the rest of `.specify/` (section 1.2); keep it committed after any amendment.

Fallback, only if the file is ever reset to the template (`cmp .specify/memory/constitution.md .specify/templates/constitution-template.md` reports no difference): run `/speckit-constitution` and paste the content of `docs/01-constitution/constitution.md` as the input, prefaced by:

> Use the following as the project constitution. Keep the nine principles, the additional constraints, the development workflow, and the governance section. Fill the compliance checks into the template's gates. Version 1.0.0, ratified 2026-10-05.

Then verify the result. If the skill kept template examples (library-first, CLI interface), remove them: Career OS is a web application and they do not apply. Remove the Sync Impact Report HTML comment at the top of the file; the skill describes it as temporary review material to be removed before the file is committed. Commit.

From then on `/speckit-constitution` is the amendment path: run it with the change, let it bump the version (MAJOR for removed or redefined principles, MINOR for additions, PATCH for wording), remove the Sync Impact Report, mirror the change into `docs/01-constitution/constitution.md`, which remains the readable source, and add a row to the constitution amendment log in `docs/04-decisions/README.md`, as the Governance section requires. Both constitution files must say the same thing.

## 3. Per feature

### 3.1 Installed skills

| Skill | What it does (from the skill's own description) | Writes |
|---|---|---|
| `/speckit-constitution` | Create or update the project constitution from provided principle inputs | `.specify/memory/constitution.md` |
| `/speckit-specify` | Create or update the feature specification from a natural language description | `specs/NNN-name/spec.md`, `specs/NNN-name/checklists/requirements.md`, `.specify/feature.json` |
| `/speckit-clarify` | Ask up to 5 targeted clarification questions and encode the answers into the spec | `spec.md`; toggles checkbox markers in `checklists/requirements.md` when it exists |
| `/speckit-plan` | Run the planning workflow using the plan template to generate design artefacts | `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md` |
| `/speckit-checklist` | Generate a custom requirements-quality checklist for a domain (UX, API, security) | `checklists/<domain>.md` |
| `/speckit-tasks` | Generate a dependency-ordered `tasks.md` from the design artefacts | `tasks.md` |
| `/speckit-analyze` | Non-destructive consistency and quality analysis across spec, plan, and tasks | Nothing; read-only report |
| `/speckit-implement` | Execute the tasks defined in `tasks.md` | Source code; marks tasks `[X]` |
| `/speckit-converge` | Assess the codebase against spec, plan, and tasks after implementation and append the remaining work as new tasks | Appends a Convergence phase to `tasks.md` |
| `/speckit-taskstoissues` | Convert tasks into dependency-ordered GitHub issues | GitHub issues, through the GitHub MCP server; the git remote must be a GitHub URL |

### 3.2 How Spec Kit tracks the current feature

- `/speckit-specify` creates `specs/NNN-<short-name>/`. With `feature_numbering: sequential` (our init choice) `NNN` is the next number after the highest existing directory in `specs/`, zero-padded to three digits. The numbers in `docs/02-requirements/02-feature-breakdown.md` therefore match only if features are specified in the listed order. To pin a directory, state `SPECIFY_FEATURE_DIRECTORY=specs/005-agent-roadmap-drafting` in the prompt; the skill uses that value as given.
- The resolved directory is written to `.specify/feature.json`. `/speckit-plan`, `/speckit-tasks`, `/speckit-analyze`, `/speckit-implement`, and `/speckit-converge` read that file to find the feature. The `SPECIFY_FEATURE_DIRECTORY` environment variable takes precedence and rewrites the file, which is how you switch back to an earlier feature. To do so, edit `.specify/feature.json` to `{"feature_directory": "specs/001-secure-access-settings"}`, or export `SPECIFY_FEATURE_DIRECTORY` in the shell before starting Claude Code (then unset it before the next `/speckit-specify`, because while it is set it overrides `feature.json` for every command); `setup-plan.sh`, `setup-tasks.sh`, and `check-prerequisites.sh` read the environment and `.specify/feature.json`, not the prompt. Mentioning the variable in the prompt is honoured only by `/speckit-specify`. Git branch names play no part in this.
- The core `/speckit-specify` does not create or switch git branches. The skill delegates branch creation to a `before_specify` hook provided by Spec Kit's git extension, and `.specify/scripts/bash/create-new-feature.sh` contains no git commands. No extension is installed here (`specify extension list` reports none). Two options:
  - Create the branch yourself before `/speckit-implement`, for example `git switch -c 001-secure-access-settings`.
  - Install the git extension once, from the repository root: `specify extension add git` (catalog entry "Git Branching Workflow", id `git`). `/speckit-specify` then runs the hook and creates a branch per feature. The branch name and the spec directory name stay independent.

### 3.3 `/speckit-specify`

Paste the feature's prompt from the feature breakdown. Add one line at the end:

> Requirement IDs covered: <FR and NFR ids from the feature table>. User stories and acceptance criteria are in docs/02-requirements/01-product-requirements.md; reuse their wording. Domain terms must follow docs/00-discovery/02-glossary.md.

The spec template has three mandatory sections (User Scenarios & Testing, Requirements, Success Criteria) plus Edge Cases, Key Entities, and Assumptions. The skill allows at most three `[NEEDS CLARIFICATION]` markers and may present them as questions in the same run; answer from `docs/05-spec-kit-handoff/02-open-questions.md`. It also writes `checklists/requirements.md`, a spec-quality checklist that `/speckit-implement` later reads as a gate. Check that the spec contains no technology choices; if it does, delete them (they belong in the plan).

### 3.4 `/speckit-clarify`

Asks at most five questions per run, highest impact first, and writes the answers into the spec. Run it until it reports no critical ambiguities. Use `docs/05-spec-kit-handoff/02-open-questions.md` as the answer sheet; record answers back into that file so later features inherit them.

### 3.5 `/speckit-plan`

Input: `spec.md` and `.specify/memory/constitution.md`. Output: `plan.md` (Technical Context, Constitution Check, Project Structure, Complexity Tracking), `research.md` (Phase 0), then `data-model.md`, `contracts/`, and `quickstart.md` (Phase 1). The command stops after Phase 1 and does not create `tasks.md`. The Constitution Check is a gate that must pass before Phase 0 research and is re-checked after Phase 1 design; any violation must be justified in Complexity Tracking.

Use this prompt, adjusting the feature-specific pointers:

> Technical context: follow docs/03-architecture/01-system-architecture.md (monorepo layout, module boundaries, technology stack), docs/03-architecture/03-data-model.md (tables for this feature), docs/03-architecture/04-api-contracts.md (routes, SSE events, operation schema), and for agent features docs/03-architecture/02-agent-architecture.md. Honour ADRs 0001 to 0010 in docs/04-decisions/; list the ones this feature relies on in the constitution check. Testing per docs/03-architecture/08-quality-and-testing.md: unit tests for domain, Testcontainers for anything touching PostgreSQL, Playwright for the user journey, replay fixtures for agent runtime. Deployment per docs/03-architecture/07-deployment-and-operations.md; no new runtime components. Generate research.md only for genuinely open technical questions; data-model.md and contracts/ must be derived from the referenced docs, not invented.

Review `plan.md`, `research.md`, `data-model.md`, `contracts/`, and `quickstart.md`. Reject anything that contradicts an ADR unless you intend to supersede it; if so, write the new ADR first.

### 3.6 `/speckit-checklist` (optional) and `/speckit-tasks`

`/speckit-checklist` writes `checklists/<domain>.md` with `CHK###` items. These are requirements-quality reviews, not implementation trackers: a ticked item means a reviewer judged the requirement well written. `/speckit-implement` counts unticked items across all checklists and asks before proceeding if any remain.

`/speckit-tasks` requires `plan.md` and `spec.md` and uses `data-model.md`, `contracts/`, `research.md`, and `quickstart.md` when present. The tasks template treats test tasks as optional unless the spec or the prompt asks for them, so ask explicitly, for example: "Include test tasks before implementation tasks for every domain and API change (constitution VI)." Check that parallelisable tasks are marked `[P]` and that tasks are grouped by user story. Agent features must include tasks for fixtures, replay tests, and at least one evaluation scenario.

### 3.7 `/speckit-analyze`

Read-only. Run before implementing. Constitution conflicts are reported as CRITICAL; fix the spec, plan, or tasks rather than the principle. This is where ADR conflicts surface.

### 3.8 `/speckit-implement` and `/speckit-converge`

Make sure you are on a feature branch first (section 3.2). `/speckit-implement` checks the checklists, executes the tasks in `tasks.md`, and marks completed ones `[X]`. Afterwards `/speckit-converge` compares the code with spec, plan, and tasks and appends any unmet work as a Convergence phase in `tasks.md`; run `/speckit-implement` again to finish it. Definition of done is in `docs/03-architecture/08-quality-and-testing.md`. Open a pull request; CI must be green. `/speckit-taskstoissues` is available if you want the tasks mirrored as GitHub issues.

## 4. Bundled workflow

`specify workflow list` shows one installed workflow, "Full SDD Cycle" (id `speckit`, version 1.0.1, defined in `.specify/workflows/speckit/workflow.yml`). Its steps are `specify`, a review gate, `plan`, a review gate, `tasks`, `implement`. Inputs: `spec` (required) and `integration` (optional; defaults to the initialised integration). Run it with:

```bash
specify workflow run speckit --input spec="<feature prompt>"
```

`specify workflow info speckit` prints the step graph; `specify workflow status` and `specify workflow resume` handle runs in progress. The workflow skips `/speckit-clarify` and `/speckit-analyze`, which constitution VIII requires before implementation, so for this project use the step-by-step flow in section 3 and treat the workflow as a reference.

## 5. Recommended first milestone: the walking skeleton

Deliver features 001, 003, and 004 first, with the Compose stack running end to end. The stubbed agent that proves the proposal flow before any model call is made is part of 004's plan and tasks, not a separate spec: a worker job that emits a fixed proposal into the inbox, replaced by the real agent in 005. This proves authentication, the data model core, the proposal flow, the worker, and the deployment. Then 002 (profile and self-assessment), because 005 depends on it (feature table in `docs/02-requirements/02-feature-breakdown.md`); then 005 (roadmap drafting), the first real agent feature; then 015 (evaluation harness) immediately after. Feature 001 also carries the repository bootstrap: the workspace layout in `03-architecture/01-system-architecture.md`, the Compose stack in `03-architecture/07-deployment-and-operations.md`, and the CI gates in `03-architecture/08-quality-and-testing.md`; say so in 001's plan and tasks prompts. For 004, add to its plan and tasks prompts: "Include a stub worker job that enqueues one fixed proposal into the inbox so the propose-then-apply flow runs end to end before any model call; 005 replaces it."

This order skips numbers, and sequential numbering (section 3.2) would otherwise create 003 as `specs/002-...`. Pin the directory in every `/speckit-specify` prompt so directory names stay predictable: `SPECIFY_FEATURE_DIRECTORY=specs/001-secure-access-settings`, then `specs/003-roadmap-milestone-management`, `specs/004-agent-proposals-review-inbox`, `specs/002-profile-self-assessment`, `specs/005-agent-roadmap-drafting`, `specs/015-agent-evaluation-harness`, and so on for every later feature. Once 015 exists the next unpinned number would be 016, so the pin is needed for 006 onward as well.

## 6. Mapping of documents to Spec Kit artefacts

| Spec Kit artefact | Primary source in `docs/` |
|---|---|
| `.specify/memory/constitution.md` | `01-constitution/constitution.md` |
| `specs/NNN/spec.md` | `02-requirements/02-feature-breakdown.md` (prompt), `02-requirements/01-product-requirements.md` (stories, FRs, NFRs), `00-discovery/03-user-journeys.md` |
| `specs/NNN/plan.md` | `03-architecture/01-system-architecture.md`, `04-decisions/*` |
| `specs/NNN/research.md` | `03-architecture/02-agent-architecture.md` (API behaviours), `04-decisions/ADR-0004`, `ADR-0006` |
| `specs/NNN/data-model.md` | `03-architecture/03-data-model.md` |
| `specs/NNN/contracts/` | `03-architecture/04-api-contracts.md` |
| `specs/NNN/quickstart.md` | `00-discovery/03-user-journeys.md`, `03-architecture/07-deployment-and-operations.md` |
| `specs/NNN/tasks.md` | `02-requirements/02-feature-breakdown.md` (dependencies), `03-architecture/08-quality-and-testing.md` (test layers, definition of done) |
| `specs/NNN/checklists/` | `03-architecture/08-quality-and-testing.md` (definition of done), `03-architecture/06-security-and-privacy.md` (release checklist); `requirements.md` in that folder is generated by Spec Kit itself |

## 7. Keeping the docs alive

- When a plan changes a decision, add or supersede an ADR in the same pull request.
- When the data model or contracts change during implementation, update `03-architecture/03-data-model.md` and `03-architecture/04-api-contracts.md`; the per-feature copies in `specs/` are derived.
- When the constitution is amended through `/speckit-constitution`, mirror the change into `01-constitution/constitution.md` and add a row to the constitution amendment log in `04-decisions/README.md` in the same pull request.
- After the first quarter of real use, revisit `00-discovery/01-vision-and-scope.md` success criteria and the agent evaluation thresholds.

## 8. Things Spec Kit will ask that are already decided

The plan template (`.specify/templates/plan-template.md`) has nine Technical Context fields. All nine are already decided for this project.

| Technical Context field | Answer |
|---|---|
| Language/Version | TypeScript (strict) on the current Node.js LTS; Next.js web app and Node worker (constitution additional constraints, ADR-0001, ADR-0002) |
| Primary Dependencies | Next.js, Drizzle ORM, pg-boss, Zod, `@anthropic-ai/sdk` (ADR-0002, ADR-0003, ADR-0004, ADR-0009; stack table in `03-architecture/01-system-architecture.md`) |
| Storage | PostgreSQL only, including the job queue; Drizzle, pg-boss (ADR-0003, ADR-0009) |
| Testing | Vitest, Testcontainers, Playwright, agent evals (`03-architecture/08-quality-and-testing.md`) |
| Target Platform | Docker Compose on a self-hosted host (ADR-0010) |
| Project Type | Web application, single user |
| Performance Goals | NFR-003, NFR-004 |
| Constraints | Constitution additional constraints; NFR-001 privacy |
| Scale/Scope | One user; small data; no horizontal scaling |
