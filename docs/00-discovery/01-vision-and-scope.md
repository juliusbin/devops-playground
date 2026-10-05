# Vision and Scope

## Purpose

The user is a Software Engineering Lead who wants to become a Software Architect **with proven results in the current role**. The two goals compete for the same hours. Career OS exists to make them reinforce each other: every week should contain work that both delivers for the team now and builds, demonstrates, or documents an architect-level competency.

The product is a personal "operating system" for that journey:

- a **roadmap** toward the architect role, grounded in an explicit competency model and curated by the user;
- **current-role objectives** (quarterly objectives and key results) linked to roadmap milestones, so delivery work is planned with its growth value in view;
- an **agent** that drafts and re-plans the roadmap, proposes weekly plans, and runs scheduled check-ins and reflections;
- an **evidence trail** that turns finished work into reusable proof for reviews and a future promotion case.

## Outcomes the product must produce

| # | Outcome | How we will know |
|---|---|---|
| O1 | The user always knows the next best action that advances both goals | Every active week has an approved weekly plan whose items trace to a milestone or key result |
| O2 | Growth work does not get silently dropped under delivery pressure | Slipped milestones are detected and re-planned within one week, with the user's approval |
| O3 | Results in the current role are captured while they are fresh | Each closed milestone or key result has at least one linked evidence item |
| O4 | Reflection becomes a habit, not a chore | Check-ins complete in under ten minutes for daily and under thirty for weekly, and the user keeps doing them |
| O5 | Preparing a review or promotion case is assembly, not archaeology | A quarter's evidence can be filtered and read in one sitting, with impact statements already written |

## Decision log from discovery

These were the user's answers during discovery on 2026-10-05. They are binding for v1 and are referenced throughout the docs.

| Topic | Decision | Consequence |
|---|---|---|
| Core purpose | **Career OS**: roadmap, link to current-role work, and evidence trail in one system | Broad domain model; three tracks of data (growth, results, evidence) must interlink |
| Users | **Only the author.** Single user. | No multi-tenancy, no sharing, simple auth. The data model keeps a `user` entity so sharing can be added later without a rewrite |
| Agent autonomy | The agent may **draft and adjust the roadmap and weekly plans** and **run check-ins and reflections** | Agent writes go through a propose-then-apply flow; check-in transcripts and reflections are recorded directly |
| Agent autonomy (excluded) | The agent does **not** pull evidence from external tools and does **not** generate reports on its own | Evidence is captured manually (the agent may prompt for it). Reports exist only as a user-triggered feature, scheduled for a later release |
| Roadmap source | **Hybrid**: the agent proposes from a built-in competency model and the user's profile; the user curates | A built-in, versioned competency model ships with the app; proposals are editable before approval |
| SDD framework | **GitHub Spec Kit** | Docs are shaped to feed `/speckit-constitution`, `/speckit-specify`, `/speckit-plan` |
| Technology | **TypeScript end to end** | One language for UI, API, worker, and agent runtime |
| Hosting | **Self-hosted containers** (Docker Compose first, Kubernetes path later) | Personal data stays on the user's infrastructure; operations must stay light for one person |
| LLM | **Claude models from Anthropic**, orchestrated in TypeScript | See ADR-0004 for the harness choice and ADR-0006 for model and inference policy |

## In scope for v1

- Profile and self-assessment against the competency model
- Roadmap with tracks, milestones, and activities; manual editing
- Agent-drafted roadmap and agent-proposed re-plans, reviewed through a proposal inbox
- Quarterly objectives and key results for the current role, linkable to milestones
- Agent-proposed weekly plans; manual adjustment; week close-out
- Scheduled and ad hoc check-ins run by the agent (daily, weekly, monthly, quarterly templates), with reflections and summaries
- Ad hoc coaching chat with full context of the user's data
- Manual evidence capture with impact statements and links to milestones, objectives, and competencies
- Dashboard with progress, pending proposals, and upcoming check-ins
- In-app notifications and configurable schedules
- Settings for model, effort, monthly budget, autonomy level, timezone
- Backup, export, and restore of all data

## Explicitly out of scope for v1

- Reading from GitHub, Jira, Linear, calendars, or documents to detect progress
- Autonomous report generation (on-demand reports are a later feature)
- Multi-user access, mentor or manager sharing, public product concerns (billing, compliance)
- Native mobile apps (the web UI must work on a phone browser)
- Semantic search over journal and evidence (a later enhancement)

## Non-goals

- Replacing the user's task manager, calendar, or note-taking app
- Being a general-purpose chatbot
- Scoring or ranking the user against other people

## Constraints

- One developer, building in evenings and weekends: favour fewer moving parts over theoretical scalability.
- Personal and career-sensitive data: keep it on infrastructure the user controls and send the LLM provider only what a task needs.
- Must run on a single small host (for example a home server or a modest VPS) with Docker.
- Monthly LLM spend must be bounded and visible.

## Success criteria for the first three months of use

1. The user completes onboarding, approves a roadmap, and runs at least three weekly cycles end to end.
2. At least one quarterly objective closes with linked evidence.
3. The user keeps using check-ins without being nagged; if not, the check-in design is wrong and gets revisited.
