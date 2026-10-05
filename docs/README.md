# Career OS – Architecture and System Design

**Status:** Architecture recommendation, pre-implementation. Nothing in this folder is code; it is the input for a Spec Driven Development (SDD) workflow using [GitHub Spec Kit](https://github.com/github/spec-kit). Spec Kit 1.1.1.dev0 is initialised in this repository for Claude Code: skills under `.claude/skills/speckit-*/`, scaffold under `.specify/`. The installed state and the exact commands are in [the handoff guide](05-spec-kit-handoff/01-handoff-guide.md).

**Working title:** Career OS. Rename freely; the name appears only in these docs.

## What the application is

A private, self-hosted web application for one person (a Software Engineering Lead) that:

1. holds a curated roadmap toward the Software Architect role, built on an explicit competency model;
2. links that roadmap to quarterly objectives and key results in the current role, so growth work and delivery work reinforce each other;
3. uses a Claude-powered agent that **proposes** roadmap drafts, re-plans and weekly plans, and **runs** scheduled check-ins and reflections, while the user **decides** what is applied;
4. accumulates an evidence trail (decisions, designs, results, feedback) that later supports a promotion case.

The agent does **not** reach into external tools (GitHub, Jira, calendar) and does **not** author reports on its own in v1. Those were explicit scoping decisions; see the decision log in `00-discovery/01-vision-and-scope.md`.

## How to read this folder

| Folder | Purpose | Feeds which Spec Kit step |
|---|---|---|
| `00-discovery/` | Vision, scope, decisions taken, glossary, user journeys | Context for every step |
| `01-constitution/` | Project principles; the readable source mirrored in `.specify/memory/constitution.md`, which is installed and filled (v1.0.0) | `/speckit-plan` (Constitution Check), `/speckit-analyze`; amendments via `/speckit-constitution` |
| `02-requirements/` | Product requirements (what and why) and the feature breakdown with one `specify` prompt per feature | `/speckit-specify`, `/speckit-clarify` |
| `03-architecture/` | System architecture, agent architecture, data model, API contracts, competency model, security, deployment, quality | `/speckit-plan` (technical context) |
| `04-decisions/` | Architecture Decision Records (ADRs) | Constraints for `/speckit-plan` and `/speckit-analyze` |
| `05-spec-kit-handoff/` | Step-by-step handoff guide and open questions | The operator's runbook |

Suggested reading order for a first pass: `00-discovery/01-vision-and-scope.md` → `02-requirements/01-product-requirements.md` → `03-architecture/01-system-architecture.md` → `03-architecture/02-agent-architecture.md` → `04-decisions/README.md` → `05-spec-kit-handoff/01-handoff-guide.md`.

## Document index

### 00 Discovery
- [01 Vision and scope](00-discovery/01-vision-and-scope.md)
- [02 Glossary](00-discovery/02-glossary.md)
- [03 User journeys](00-discovery/03-user-journeys.md)

### 01 Constitution
- [Constitution](01-constitution/constitution.md)

### 02 Requirements
- [01 Product requirements](02-requirements/01-product-requirements.md)
- [02 Feature breakdown and specify prompts](02-requirements/02-feature-breakdown.md)

### 03 Architecture
- [01 System architecture](03-architecture/01-system-architecture.md)
- [02 Agent architecture](03-architecture/02-agent-architecture.md)
- [03 Data model](03-architecture/03-data-model.md)
- [04 API contracts](03-architecture/04-api-contracts.md)
- [05 Competency model and results framework](03-architecture/05-competency-model.md)
- [06 Security and privacy](03-architecture/06-security-and-privacy.md)
- [07 Deployment and operations](03-architecture/07-deployment-and-operations.md)
- [08 Quality and testing](03-architecture/08-quality-and-testing.md)

### 04 Decisions
- [ADR index](04-decisions/README.md)

### 05 Spec Kit handoff
- [01 Handoff guide](05-spec-kit-handoff/01-handoff-guide.md)
- [02 Open questions](05-spec-kit-handoff/02-open-questions.md)

## Conventions used in these documents

- Requirement IDs: `FR-xxx` functional, `NFR-xxx` non-functional, `US-xxx` user story. Features are numbered `001-…` to match Spec Kit's `specs/NNN-feature-name/` folders. Spec Kit assigns `NNN` sequentially after the highest existing folder, so the numbers line up only when features are specified in the listed order or the directory is pinned (handoff guide, section 3.2).
- `[NEEDS CLARIFICATION]` marks an open question, in the same style Spec Kit uses. All of them are collected in `05-spec-kit-handoff/02-open-questions.md`.
- Diagrams are Mermaid and render on GitHub.
- "v1" means the first usable release (features marked P0 and P1 in the feature breakdown). "Later" means explicitly out of v1 scope.
