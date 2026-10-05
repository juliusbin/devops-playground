# Product Requirements

This document states **what** Career OS must do and **why**, in Spec Kit style: user stories with acceptance criteria, functional requirements, and non-functional requirements. It avoids implementation detail; technology decisions live in `03-architecture/` and `04-decisions/`. Each Spec Kit feature in `02-feature-breakdown.md` cites the requirement IDs it covers, so `/speckit-specify` can pull the relevant subset.

Priorities: **P0** must exist for the product to be useful at all; **P1** completes v1; **P2** is planned but after v1; **P3** is a recorded idea.

## Personas

One persona: **the Lead**, a Software Engineering Lead with a full calendar, around five to eight hours a week for deliberate growth work, who wants to become a Software Architect within roughly two years and needs demonstrable results in the current role to get there. Uses a laptop at work and a phone in between.

## User stories

Acceptance criteria use Given/When/Then.

### Access and settings

**US-001 (P0)** As the Lead, I want to sign in with a password so that nobody else on my network can read my career data.
- Given no account exists, when I open the app, then I am guided to create the single account with a strong password.
- Given an account exists, when I enter wrong credentials five times in ten minutes, then further attempts are delayed and the event is logged.
- Given I am signed in, when I close the browser and return within the session lifetime, then I remain signed in.

**US-002 (P0)** As the Lead, I want to set my timezone, week start day, and check-in times so that schedules match my life.
- Given I change the timezone, when the next schedule fires, then it fires at the configured local time.

**US-003 (P0)** As the Lead, I want to set a monthly LLM budget and see spend so that cost never surprises me.
- Given spend reaches 80% of budget, when the next day starts, then I receive a warning notification.
- Given spend reaches 100%, when a background agent job is due, then it is skipped with a notification, and interactive sessions ask for confirmation before starting.

### Profile and self-assessment

**US-010 (P0)** As the Lead, I want to describe my current situation and target so that the roadmap fits me.
- Given I complete the profile, when I save, then every required field is validated and the dashboard shows the target date countdown.

**US-011 (P0)** As the Lead, I want to browse the competency model and rate myself per competency so that gaps are explicit.
- Given a competency, when I select a level, then I can add a confidence rating and a note, and the assessment is dated.
- Given I reassess later, when I view a competency, then I see my assessment history.

### Roadmap

**US-020 (P0)** As the Lead, I want the agent to draft a roadmap from my profile, self-assessment, and the competency model so that I start from a strong proposal rather than a blank page.
- Given a completed profile and at least a partial self-assessment, when I request a draft, then within a bounded time I receive a proposal containing tracks, milestones with target dates, definitions of done, evidence requirements, and starter activities, each with a rationale.
- Given the proposal, when I approve some operations and reject others, then only approved ones become the roadmap and the rejections are recorded with my optional reason.

**US-021 (P0)** As the Lead, I want to create, edit, reorder, and close milestones and activities myself so that I stay in control.
- Given a milestone, when I change its status, then the change is recorded with actor and time.
- Given I mark a milestone done that requires evidence and has none linked, then I am warned and can proceed, and the gap shows on the dashboard.

**US-022 (P1)** As the Lead, I want the agent to propose a re-plan when milestones slip or when I ask, so that the roadmap stays realistic.
- Given a milestone whose target date has passed without completion, when the nightly check runs, then a re-plan proposal is created within one day with options and rationale, unless one is already pending.
- Given I approve a re-plan, when it applies, then a new roadmap version is recorded and the previous version remains viewable.

### Objectives and key results

**US-030 (P0)** As the Lead, I want to record quarterly objectives and key results for my current role so that delivery results are tracked alongside growth.
- Given an objective, when I add a key result, then I enter metric, baseline, target, unit, and can update current value over time with a dated note.

**US-031 (P0)** As the Lead, I want to link milestones and objectives so that dual-purpose work is visible.
- Given a growth milestone with no linked objective, when I view the roadmap, then it is marked as "growth only" and the objectives page lists unlinked objectives.

**US-032 (P1)** As the Lead, I want the agent to suggest links and dual-purpose milestones so that I notice opportunities I would miss.
- Given objectives and milestones exist, when I ask for suggestions, then a proposal lists suggested links with rationale.

### Weekly planning

**US-040 (P0)** As the Lead, I want the agent to propose a weekly plan within my capacity so that each week starts with a decision already prepared.
- Given the schedule fires, when the proposal is ready, then it contains items with planned hours whose total does not exceed capacity, a focus theme, and a note on what was deliberately excluded.
- Given I edit hours or remove items, when I approve, then the week becomes active with my edits.

**US-041 (P0)** As the Lead, I want to update plan item status during the week so that check-ins start from current state.
- Given an active week, when I mark an item done, then the linked activity or key result reflects it.

**US-042 (P0)** As the Lead, I want to close the week with carry-over decisions so that nothing is silently lost.
- Given unfinished items, when I close the week, then each is explicitly carried over, dropped, or rescheduled, and the decision is stored.

### Check-ins and reflections

**US-050 (P0)** As the Lead, I want short daily check-ins led by the agent so that reflection fits into five minutes.
- Given today's plan items, when the daily check-in starts, then the agent asks no more than three questions, referencing items by name.
- Given my answers, when the check-in ends, then a reflection is stored, status updates are proposed, and a one-line summary is shown.

**US-051 (P0)** As the Lead, I want weekly, monthly, and quarterly check-ins with templates so that reviews are consistent.
- Given a weekly review, when it runs, then each plan item is covered and evidence captures are proposed for items that required evidence.
- Given a monthly retrospective, when it runs, then competency domains with no activity in the month are named.

**US-052 (P0)** As the Lead, I want to write a reflection at any time so that insights are not lost between check-ins.
- Given a reflection, when I tag it with a milestone or objective, then it appears in that item's history.

**US-053 (P1)** As the Lead, I want to resume an interrupted check-in so that a phone call does not cost me the session.
- Given a check-in in progress, when I return within 24 hours, then the conversation resumes where it stopped.

### Coach chat

**US-060 (P1)** As the Lead, I want to ask the agent questions about my roadmap, plan, and evidence so that I get advice grounded in my data.
- Given a question, when the agent answers, then any plan change it suggests arrives as a proposal, never as a direct change.

### Evidence

**US-070 (P0)** As the Lead, I want to capture evidence with an impact statement and links so that results are reusable later.
- Given a new evidence item, when I save, then kind, date, title, and impact statement are required, and a link or attachment is optional.
- Given an attachment, when I upload a file within the size limit, then it is stored locally and downloadable only when signed in.

**US-071 (P1)** As the Lead, I want the agent to propose evidence captures during check-ins so that proof is recorded while fresh.
- Given I mention completing an item that required evidence, when the check-in ends, then a pre-filled evidence proposal exists for me to complete.

**US-072 (P0)** As the Lead, I want to filter and read evidence by quarter, competency, objective, or kind so that preparing a review is fast.

### Dashboard and insights

**US-080 (P0)** As the Lead, I want a dashboard that shows this week, pending proposals, upcoming check-ins, roadmap progress by track, and evidence gaps so that I know where I stand in one glance.

**US-081 (P1)** As the Lead, I want to see competency coverage over time so that neglected domains are obvious.

### Notifications and schedules

**US-090 (P0)** As the Lead, I want in-app notifications for proposals, reminders, and budget events so that I act on time.

**US-091 (P2)** As the Lead, I want optional email or push delivery of notifications so that I notice them away from the app.

### Reports (later)

**US-100 (P2)** As the Lead, I want to generate an on-demand quarterly review or promotion-case draft from approved evidence and closed milestones so that writing the document is editing, not authoring.

### Data ownership

**US-110 (P0)** As the Lead, I want automated backups and a tested restore so that a disk failure does not erase two years of work.

**US-111 (P1)** As the Lead, I want to export everything in open formats and import it into a fresh installation.

**US-112 (P0)** As the Lead, I want to see and delete what the agent remembers about me so that its memory stays accurate and acceptable.

## Functional requirements

| ID | Requirement | Stories |
|---|---|---|
| FR-001 | The system supports exactly one user account with password authentication and session persistence. | US-001 |
| FR-002 | The system stores user settings: timezone, week start, check-in times, schedules, model, effort per job type, monthly budget, autonomy level. | US-002, US-003 |
| FR-003 | The system records every LLM request's usage and cost and aggregates spend per day and month. | US-003 |
| FR-004 | The system pauses background agent jobs when the monthly budget is reached and notifies the user. | US-003 |
| FR-010 | The system stores a profile with current role, target role, target date, weekly growth hours, context, strengths, growth areas. | US-010 |
| FR-011 | The system ships a versioned competency model and results framework and allows per-competency self-assessment with history. | US-011 |
| FR-020 | The agent can produce a roadmap draft proposal from profile, self-assessment, and competency model. | US-020 |
| FR-021 | The system provides manual CRUD for tracks, milestones, and activities, with status history and actor attribution. | US-021 |
| FR-022 | The system detects slipped milestones nightly and the agent proposes a re-plan when no re-plan proposal is pending. | US-022 |
| FR-023 | The system versions the roadmap on each applied re-plan and keeps prior versions readable. | US-022 |
| FR-030 | The system provides CRUD for objectives and key results, with dated progress updates. | US-030 |
| FR-031 | The system links milestones to objectives and competencies and surfaces unlinked items. | US-031 |
| FR-032 | The agent can propose links and dual-purpose milestones on request. | US-032 |
| FR-040 | The agent produces a weekly plan proposal on schedule or on demand, respecting capacity. | US-040 |
| FR-041 | The system provides a week board for plan item status updates that propagate to linked activities and key results. | US-041 |
| FR-042 | The system requires explicit carry-over, drop, or reschedule decisions at week close. | US-042 |
| FR-050 | The agent runs check-ins following templates (daily, weekly, monthly, quarterly, ad hoc) with bounded question counts. | US-050, US-051 |
| FR-051 | Check-ins produce stored reflections, proposals, and a summary; transcripts are kept. | US-050, US-051 |
| FR-052 | The user can create reflections at any time and tag them. | US-052 |
| FR-053 | Interrupted check-ins can be resumed within 24 hours. | US-053 |
| FR-060 | The user can hold an ad hoc coach chat; plan changes from it arrive as proposals. | US-060 |
| FR-070 | The system provides evidence CRUD with impact statements, links, and local attachments with a size limit. | US-070, US-072 |
| FR-071 | The agent can propose pre-filled evidence captures from check-in content. | US-071 |
| FR-080 | The dashboard shows current week, pending proposals, upcoming check-ins, progress by track, and evidence gaps. | US-080 |
| FR-081 | The system shows competency coverage over time. | US-081 |
| FR-090 | The system delivers in-app notifications for proposals, reminders, budget events, and job failures. | US-090 |
| FR-091 | Optional email or push delivery through the user's own server. | US-091 |
| FR-100 | On-demand report generation from approved evidence and closed milestones (later). | US-100 |
| FR-110 | Automated database and attachment backups with documented, tested restore. | US-110 |
| FR-111 | Full export and import in open formats. | US-111 |
| FR-112 | The user can view, edit, and delete agent notes. | US-112 |
| FR-120 | All agent-originated changes to plan state pass through proposals with per-operation approval (constitution II). | All agent stories |
| FR-121 | Every proposal operation carries a rationale and references. | All agent stories |

## Non-functional requirements

| ID | Requirement | Target |
|---|---|---|
| NFR-001 | Privacy | No user content leaves the host except to the configured LLM endpoint and the user's own notification server. |
| NFR-002 | Availability | Single host; restart on failure; the app is usable within one minute of host boot. No high-availability requirement. |
| NFR-003 | Interactive latency | Page loads under one second on a LAN; agent replies begin streaming within three seconds of sending a message. |
| NFR-004 | Background job latency | Weekly plan proposals complete within ten minutes of the schedule; roadmap drafts within fifteen minutes. |
| NFR-005 | Cost | Default configuration keeps monthly LLM spend within the user's budget; spend is visible per session. |
| NFR-006 | Data durability | Daily backups retained for 30 days; restore tested per release. |
| NFR-007 | Security | Passwords hashed with a modern memory-hard algorithm; sessions in HttpOnly, Secure cookies; brute-force throttling; TLS at the edge. |
| NFR-008 | Usability | Core flows (daily check-in, status update, evidence capture) usable on a phone browser with one hand. |
| NFR-009 | Accessibility | Keyboard navigable; WCAG 2.1 AA colour contrast. |
| NFR-010 | Operability | One `docker compose up`; health endpoints; structured logs; documented runbook. |
| NFR-011 | Maintainability | Typed end to end; contracts generated, not hand-written; ADRs current. |
| NFR-012 | Agent quality | Scenario evaluation suite with a pass threshold agreed in `03-architecture/08-quality-and-testing.md`; regressions block release. |
| NFR-013 | Agent safety | The agent cannot mutate plan state outside proposals; tool inputs are schema-validated; per-session iteration and token caps. |
| NFR-014 | Resumability | Agent sessions and jobs survive a worker restart without duplicate side effects. |

## Assumptions

- The user's organisation does not forbid sending non-confidential personal planning text to an LLM API. Company-confidential detail should be paraphrased by the user; the UI reminds them once.
- English is the only UI language in v1.
- The host has outbound HTTPS access to the LLM API.

## Open questions

See `05-spec-kit-handoff/02-open-questions.md` for the consolidated `[NEEDS CLARIFICATION]` list.
