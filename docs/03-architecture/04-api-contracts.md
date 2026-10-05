# API Contracts

The contracts between browser and server, between server and agent tools, and between agent and proposals. Spec Kit's `contracts/` folder per feature should be derived from this document. All shapes are defined once as Zod schemas in `packages/contracts`; the OpenAPI document is generated from them and committed (constitution V).

## Conventions

- Base path `/api/v1`. JSON request and response bodies. Dates as ISO 8601 strings in UTC; calendar dates (`week_start`, `target_date`) as `YYYY-MM-DD`.
- Authentication: session cookie (`cos_session`, HttpOnly, Secure, SameSite=Lax). All routes except `/auth/*` and `/health` require it. State-changing requests must carry the `Origin` header matching the app origin.
- Errors: `application/problem+json` with `type`, `title`, `status`, `detail`, `instance`, and `errors[]` for validation (`path`, `message`).
- Idempotency: mutating routes accept `Idempotency-Key`; replays return the original response for 24 hours.
- Pagination: `?cursor=&limit=` with `next_cursor` in responses; lists default to 50.
- Versioning: breaking changes go to `/api/v2`; additive changes are unversioned.

## Routes

### Health and auth

| Method | Path | Purpose | Body / response |
|---|---|---|---|
| GET | `/health` | Liveness and readiness (db, worker heartbeat, last backup) | `{ status, checks: { db, worker, backup } }` |
| POST | `/auth/setup` | Create the single account on first run (fails if one exists) | `{ email, display_name, password, timezone, week_start_day }` |
| POST | `/auth/login` | Sign in | `{ email, password, totp? }` → sets cookie |
| POST | `/auth/logout` | Sign out | – |
| GET | `/auth/session` | Current session | `{ user, expires_at }` |
| POST | `/auth/totp/enroll`, `/auth/totp/confirm` | Optional second factor | |

### Settings and usage

| Method | Path | Purpose |
|---|---|---|
| GET | `/settings` | All settings as a typed object |
| PATCH | `/settings` | Partial update; validated per key (`model`, `effort`, `monthly_budget_usd`, `autonomy_level`, `checkin_times`, `features`, `proposal_expiry_days`) |
| GET | `/usage?month=YYYY-MM` | Spend and tokens by day and job type, cache hit ratio, budget status |
| GET | `/schedules` / PUT `/schedules/{job_type}` | View and edit cron schedules |

### Profile and competency

| Method | Path | Purpose |
|---|---|---|
| GET / PUT | `/profile` | Profile |
| GET | `/competency-model` | Current model version: domains → competencies → levels |
| GET | `/self-assessments` | Latest assessment per competency plus gap summary |
| GET | `/self-assessments/{competency_key}/history` | History |
| POST | `/self-assessments` | `{ competency_key, level, confidence, note }` (append) |

### Roadmap

| Method | Path | Purpose |
|---|---|---|
| GET | `/roadmap` | Active roadmap with tracks, milestones (compact), progress per track |
| GET | `/roadmap/versions` and `/roadmap/versions/{n}` | History snapshots |
| POST | `/roadmap/draft` | Start `roadmap.draft` job → `202 { job_id, session_id }` |
| POST | `/roadmap/replan` | Start `roadmap.replan` on demand → `202` |
| POST | `/roadmap/suggest-links` | Start `links.suggest` → `202` |
| GET | `/milestones?track=&status=&competency=&objective=` | List |
| POST | `/milestones` | Create (user) |
| GET / PATCH | `/milestones/{id}` | Read full / update fields |
| POST | `/milestones/{id}/status` | `{ status, reason? }` |
| POST | `/milestones/reorder` | `{ track_id, ordered_ids[] }` |
| POST / PATCH / DELETE | `/milestones/{id}/activities`, `/activities/{id}` | Activities |
| POST | `/activities/{id}/status` | `{ status }` |

### Objectives

| Method | Path | Purpose |
|---|---|---|
| GET | `/objectives?quarter=` | Objectives with key results and linked milestones |
| POST / PATCH | `/objectives`, `/objectives/{id}` | CRUD |
| POST | `/objectives/{id}/close` | `{ status, retro_note }` |
| POST / PATCH / DELETE | `/objectives/{id}/key-results`, `/key-results/{id}` | Key results |
| POST | `/key-results/{id}/updates` | `{ value, note }` |
| GET | `/objectives/unlinked-report` | Growth-only milestones and objectives without growth leverage |

### Weekly planning

| Method | Path | Purpose |
|---|---|---|
| GET | `/weeks/current` and `/weeks/{week_start}` | Plan, items, capacity, totals |
| POST | `/weeks/{week_start}/propose` | Start `plan.weekly` on demand → `202` |
| POST | `/weeks/{week_start}` | Create manually `{ capacity_hours, focus_theme, items[] }` |
| PATCH | `/weeks/{week_start}` | Capacity, theme |
| POST / PATCH / DELETE | `/weeks/{week_start}/items`, `/plan-items/{id}` | Items |
| POST | `/plan-items/{id}/status` | `{ status, actual_hours?, outcome_note? }` |
| POST | `/weeks/{week_start}/close` | `{ decisions: [{ item_id, action: carry_over | reschedule | drop, to_week_start? }], closeout_note }` |

### Check-ins and reflections

| Method | Path | Purpose |
|---|---|---|
| GET | `/check-ins?from=&to=&type=` | List with summaries |
| POST | `/check-ins` | `{ type }` → creates check-in and an interactive agent session → `201 { check_in, session_id }` |
| GET | `/check-ins/{id}` | Detail incl. summary, reflections, proposals |
| POST | `/check-ins/{id}/complete` | Ends early; agent writes a summary |
| POST | `/check-ins/{id}/abandon` | |
| GET / POST | `/reflections` | List with search `?q=&from=&to=&tag=`; create `{ body, tags[], links[] }` |

### Agent sessions (interactive transport)

| Method | Path | Purpose |
|---|---|---|
| POST | `/agent/sessions` | `{ job_type: "coach.chat", thread_id? }` → `201 { session_id }` |
| GET | `/agent/sessions/{id}` | Session record: messages (rendered), tool calls, usage |
| POST | `/agent/sessions/{id}/messages` | `{ text }` → `202 { turn_id }`; stores the message, starts the turn |
| GET | `/agent/sessions/{id}/stream` | SSE; supports `Last-Event-ID` |
| POST | `/agent/sessions/{id}/cancel` | Stops the current turn |
| GET | `/agent/threads` | Coach threads |
| GET / PATCH / DELETE | `/agent/notes`, `/agent/notes/{id}` | Agent memory management |

SSE event types and payloads (all carry `id`, `session_id`, `turn_id`):

| Event | Payload |
|---|---|
| `turn.started` | `{ request_seq }` |
| `message.delta` | `{ delta }` |
| `message.completed` | `{ message_seq, text }` |
| `tool.started` | `{ tool_use_id, tool_name, input_summary }` |
| `tool.completed` | `{ tool_use_id, is_error, result_summary, duration_ms }` |
| `proposal.created` | `{ proposal_id, kind, title, operation_count }` |
| `usage` | `{ input_tokens, cache_read_input_tokens, cache_creation_input_tokens, output_tokens, cost_usd }` |
| `turn.completed` | `{ stop_reason }` |
| `turn.failed` | `{ reason, category?, retryable }` |
| `session.completed` | `{ summary? }` |

### Jobs (background transport)

| Method | Path | Purpose |
|---|---|---|
| GET | `/jobs/{id}` | `{ status: queued | running | completed | failed, session_id?, proposal_id?, error? }` |
| GET | `/jobs/{id}/events` | SSE with `job.progress`, `job.completed`, `job.failed` |

### Proposals

| Method | Path | Purpose |
|---|---|---|
| GET | `/proposals?status=pending` | Inbox; count in header `X-Pending-Count` |
| GET | `/proposals/{id}` | Full proposal with operations, rationale, references resolved to titles |
| POST | `/proposals/{id}/decide` | `{ decisions: [{ seq, decision: approve | edit | reject, edited_payload?, reject_reason? }] }` → `{ status, applied: [seq], rejected: [seq], errors: [{ seq, message }] }` |
| POST | `/proposals/{id}/undo-operation` | `{ seq }` for auto-applied low-risk operations within 7 days |

### Evidence

| Method | Path | Purpose |
|---|---|---|
| GET | `/evidence?quarter=&kind=&competency=&objective=&milestone=&status=` | Portfolio list |
| POST / GET / PATCH / DELETE | `/evidence`, `/evidence/{id}` | CRUD; `PATCH` with `status: final` completes a draft |
| POST / DELETE | `/evidence/{id}/links`, `/evidence/{id}/links/{link_id}` | Links |
| GET | `/evidence/gaps` | Closed milestones requiring evidence with none |
| POST | `/attachments` | multipart; enforces size limit; returns `{ attachment_id, sha256 }` |
| GET | `/attachments/{id}` | Authenticated download |

### Dashboard, notifications, data

| Method | Path | Purpose |
|---|---|---|
| GET | `/dashboard` | Aggregated view model for the home screen |
| GET | `/insights/coverage?months=` | Competency coverage over time |
| GET | `/notifications?unread=true` / POST `/notifications/{id}/read` / POST `/notifications/read-all` | |
| GET | `/export` | Streams a zip: JSON per table plus attachments |
| POST | `/import` | multipart zip; dry-run flag; only on an empty installation |
| GET | `/backups` | Backup runs |

## Proposal operation schema

The `Operation` discriminated union is shared by the `propose_changes` tool, the proposal API, and the application service. Every operation has these common fields:

```ts
const OperationBase = z.object({
  rationale: z.string().min(10).max(600),
  references: z.array(z.object({
    type: z.enum(["check_in", "reflection", "evidence", "milestone", "objective", "key_result", "self_assessment"]),
    id: z.string().uuid(),
  })).max(10),
  risk: z.enum(["low", "normal"]).default("normal"),
});
```

| `op` | Payload | Applies to | Notes |
|---|---|---|---|
| `roadmap.create` | `{ title, tracks: [{ kind, name, milestones: [MilestoneDraft] }] }` | roadmap drafting only | `MilestoneDraft` includes `activities: ActivityDraft[]`; rejected if an active roadmap exists |
| `milestone.create` | `{ track_id, ...MilestoneDraft }` | re-plan, coach | |
| `milestone.update` | `{ milestone_id, changes: Partial<MilestoneFields> }` | re-plan | Changing `target_date` requires `rationale` to mention the slip or capacity |
| `milestone.set_status` | `{ milestone_id, status, reason? }` | check-ins, re-plan | `done` with missing evidence yields a warning in the result |
| `milestone.split` | `{ milestone_id, into: [MilestoneDraft, MilestoneDraft] }` | re-plan | Original becomes `dropped` with a link note |
| `activity.create` / `activity.update` / `activity.set_status` | as milestone equivalents | all | |
| `weekly_plan.set` | `{ week_start, capacity_hours, focus_theme, excluded_note, items: [{ source_type, source_id?, title, planned_hours }] }` | weekly planning | Hours sum ≤ capacity validated at proposal creation |
| `plan_item.set_status` | `{ plan_item_id, status, actual_hours?, outcome_note? }` | check-ins | `risk: low` when the user stated it |
| `plan_item.carry_over` | `{ plan_item_id, to_week_start }` | weekly review | |
| `key_result.update_progress` | `{ key_result_id, value, note }` | check-ins | `risk: low` when the user stated it |
| `objective.link_milestone` | `{ objective_id, milestone_id }` | links.suggest | |
| `evidence.capture` | `{ title, kind, occurred_on, url?, impact: { situation, task, action, result }, links: [{ target_type, target_id | target_key }] }` | weekly review, coach | Creates `evidence` with `status: draft`; user completes |
| `self_assessment.suggest` | `{ competency_key, level, confidence, note }` | quarterly review | Appends an assessment on approval |
| `note.remember` | `{ category, body }` | any (if the agent prefers approval for a note) | Usually written directly via `remember_note` |

Validation at proposal creation (tool and API): every referenced ID exists and belongs to the user; operations within one proposal are consistent (no two operations on the same item with conflicting statuses); risk tags are only `low` for the two allowed operation types.

## Agent tool contracts

Tool input and output schemas live beside the operation schema in `packages/contracts/agent-tools.ts`. Summary (outputs are compact JSON designed to be small in the context window):

| Tool | Input | Output |
|---|---|---|
| `get_profile` | `{}` | `{ profile, gap_summary: [{ domain, current_avg, target, gap }] }` |
| `get_competency_model` | `{ domain_key? }` | `{ version, domains: [{ key, name, competencies: [{ key, name, levels: [{ level, name, behaviours }] }] }] }` |
| `get_roadmap` | `{ include_done?: boolean }` | `{ roadmap_id, tracks: [{ id, kind, name, milestones: [{ id, title, status, target_date, priority, competency_key?, objective_id?, evidence_required, open_activities }] }] }` |
| `get_milestone` | `{ milestone_id }` | full milestone incl. activities, history, evidence links |
| `list_objectives` | `{ quarter? }` | objectives with key results and linked milestone IDs |
| `get_week` | `{ week_start? }` | plan with items and totals |
| `list_recent_checkins` | `{ limit?: number ≤ 10, type? }` | summaries |
| `search_reflections` | `{ query?, from?, to?, tag?, limit? ≤ 20 }` | `[{ id, recorded_at, excerpt, tags }]` |
| `list_evidence` | `{ quarter?, milestone_id?, objective_id?, competency_key?, limit? ≤ 50 }` | compact items |
| `get_agent_notes` | `{}` | active notes |
| `get_capacity` | `{ week_start? }` | `{ weekly_growth_hours, overrides: [...] }` |
| `record_reflection` | `{ body, tags[], links[] }` | `{ reflection_id }` |
| `remember_note` | `{ category, body }` | `{ note_id }` |
| `summarize_checkin` | `{ wins[], blockers[], next_focus, decisions[], open_questions[] }` | `{ ok }` |
| `propose_changes` | `{ kind, title, summary, operations: Operation[] }` | `{ proposal_id, status: "pending_user_review", warnings[] }` or `{ error, invalid_operations: [{ seq, message }] }` |

All tool definitions set `strict: true` and `additionalProperties: false` in their generated schemas.

## Structured outputs for background jobs

Background jobs that produce one artefact use `output_config.format` with a Zod schema instead of free text:

- `RoadmapDraftOutput = { title, summary, excluded_note, open_questions[], operations: [roadmap.create] }`
- `WeeklyPlanOutput = { summary, operations: [weekly_plan.set] }`
- `ReplanOutput = { summary, options_considered[], operations: Operation[] }`

The runtime wraps the parsed output into a proposal through the same validation as the tool path.
