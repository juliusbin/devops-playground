# Agent Architecture

The agent is the part of Career OS that drafts roadmaps, proposes re-plans and weekly plans, and runs check-ins and coaching conversations. This document specifies its runtime, tool surface, autonomy policy, prompt and caching design, scheduling, cost controls, failure handling, and evaluation. The harness decision is ADR-0004; the model and inference policy is ADR-0006.

## Design principles

1. **Domain tools, not shell tools.** The agent works on the user's roadmap, plans, objectives, reflections, and evidence through typed tools that call the same domain use cases the UI uses. It has no file system, shell, or network tools.
2. **Propose, never apply.** All plan-state changes go through the `propose_changes` tool. Direct writes are limited to append-only records.
3. **One model, frozen prompts, stable tool sets.** Each job type has a fixed system prompt and a fixed, alphabetically ordered tool list so prompt caching works. Dynamic context travels in messages, never in the system prompt.
4. **Everything recorded.** Every request's usage, every tool call, and every message is persisted before the next step proceeds.
5. **Append-only conversation history.** Stored messages are never edited or deleted; this is required for cache efficiency and for the current models' preserved-thinking checks (see "Model behaviours that constrain the harness").

## Runtime

### Harness

The agent runtime is built on the Anthropic TypeScript SDK (`@anthropic-ai/sdk`) using the beta **tool runner** (`client.beta.messages.toolRunner`) with tools defined through `betaZodTool`. The runner drives the request → tool execution → result loop; the runtime wraps it with persistence, streaming, budget checks, and iteration caps.

Why not the Claude Agent SDK, which the discovery answer named: it packages the Claude Code harness, spawns a bundled Claude Code binary per session, and ships file, shell, and web tools built for coding agents. Career OS needs none of those and would have to disable them, while taking on a native binary inside the web container and losing direct control of caching and structured outputs. Anthropic's own comparison table routes "call the Claude API directly from your own code" to the client SDK and its tool runner. Full reasoning and revisit triggers are in ADR-0004.

### Two execution modes

| Mode | Where it runs | Jobs | Characteristics |
|---|---|---|---|
| Interactive | `web` process | `checkin.run`, `coach.chat` | Streams tokens to the browser over SSE; user is present; multi-turn; stops when the user ends or the template completes |
| Background | `worker` process | `roadmap.draft`, `roadmap.replan`, `plan.weekly`, `links.suggest`, later `report.compose` | No user present; typically one to three model requests; produces proposals and a notification; idempotent per scope |

Both modes use the same `AgentRuntime` class from `packages/agent`; only the transport (SSE vs. none) and the trigger differ.

### Job catalogue

| Job type | Trigger | Input context | Output | Effort (default) |
|---|---|---|---|---|
| `roadmap.draft` | User action | profile, self-assessment, competency model, existing objectives | `Proposal(kind=roadmap_draft)` with tracks, milestones, activities | `xhigh` |
| `roadmap.replan` | Nightly slip detection or user action | roadmap, slipped and at-risk milestones, recent reflections, capacity | `Proposal(kind=replan)` | `high` |
| `plan.weekly` | Schedule or user action | open activities, milestone priorities, key results, last week's close-out, capacity | `Proposal(kind=weekly_plan)` | `high` |
| `links.suggest` | User action | milestones, objectives | `Proposal(kind=updates)` with link operations | `medium` |
| `checkin.run` | User opens a check-in | template, today's or this week's plan, recent reflections, agent notes | reflections, `Proposal(kind=updates)`, summary | `medium` |
| `coach.chat` | User opens coach | thread history, on-demand reads via tools | answer text, optional proposal | `medium` |
| `report.compose` (later) | User action | evidence, closed milestones, objectives for a period | document draft | `high` |

Effort is a per-job-type setting with these defaults. It is the first cost lever (ADR-0006); the model is the same for all jobs.

### Request shape (interactive example)

```ts
// packages/agent/src/runtime/turn.ts (illustrative, not final code)
const runner = client.beta.messages.toolRunner({
  model: settings.model,                       // default "claude-opus-5-5"
  max_tokens: 64_000,                          // streaming, so give the model room
  stream: true,
  betas: ["server-side-fallback-2026-07-01"],
  fallbacks: "default",                        // safety-classifier refusals re-run on a fallback model
  output_config: { effort: settings.effort[jobType] },
  system: [
    { type: "text", text: PROMPTS[jobType].system, cache_control: { type: "ephemeral" } },
  ],
  tools: TOOLSETS[jobType],                    // fixed, sorted by name
  messages: [...storedHistory, latestUserTurn],
  max_iterations: LIMITS[jobType].maxToolIterations,
});

for await (const stream of runner) {
  stream.on("text", (delta) => sse.send("message.delta", { delta }));
  const message = await stream.finalMessage();
  await persist.assistantMessage(sessionId, message);   // full content blocks, append-only
  await persist.usage(sessionId, message.usage, settings.model);
  if (message.stop_reason === "refusal") { await markFailed(sessionId, "refused", message.stop_details); break; }
  if (message.stop_reason === "max_tokens") { await markFailed(sessionId, "truncated"); break; }
}
const final = await runner.done();
```

Notes for implementers:

- Thinking is on by default on the chosen model and cannot be disabled; do not send a `thinking` parameter with a budget. Control depth with `output_config.effort`.
- Forced tool choice is not supported on this model; steer with the prompt and keep `strict: true` on tool schemas so arguments are schema-valid.
- Background jobs that need one structured result (roadmap draft, weekly plan) use `client.messages.parse` with `output_config.format = zodOutputFormat(ProposalSchema)` after an optional read-tools phase, rather than asking the model to emit JSON in text.
- Always append the model's full `content` array to history, including thinking blocks, when continuing the same conversation.

## Tool surface

Tools are grouped by effect. All are `betaZodTool` definitions in `packages/agent/src/tools`, with input and output schemas in `packages/contracts`. Each job type gets a fixed subset; the table shows the v1 assignment.

### Read tools (no side effects, parallel-safe)

| Tool | Purpose | Jobs |
|---|---|---|
| `get_profile` | Profile and latest self-assessment summary | all |
| `get_competency_model` | Domains, competencies, level descriptors; optional domain filter | draft, replan, checkin, coach |
| `get_roadmap` | Tracks and milestones with status, dates, links; compact form | all except draft |
| `get_milestone` | One milestone with activities, history, linked evidence | replan, checkin, coach |
| `list_objectives` | Objectives and key results for a quarter | all |
| `get_week` | Weekly plan and items for a given week (default current) | weekly, checkin, coach |
| `list_recent_checkins` | Summaries of the last N check-ins | replan, weekly, checkin, coach |
| `search_reflections` | Keyword search over reflections with date range | replan, checkin, coach |
| `list_evidence` | Evidence filtered by period, milestone, objective, competency | checkin, coach, report |
| `get_agent_notes` | Facts the agent previously chose to remember | all |
| `get_capacity` | Weekly hours available, upcoming time off | draft, replan, weekly |

### Append-only write tools (allowed without approval)

| Tool | Purpose | Guard |
|---|---|---|
| `record_reflection` | Store a reflection from the conversation with tags and links | Content attributed to the user's words; max length; linked IDs must exist |
| `remember_note` | Store an agent note about preferences or constraints | Category enum; user can delete; max 50 active notes |
| `summarize_checkin` | Store the end-of-check-in summary (wins, blockers, next focus, decisions) | Only in `checkin.run`; one per check-in |

### Proposal tool (the only mutation path)

`propose_changes({ kind, title, summary, operations: Operation[] })` creates a proposal and returns `{ proposalId, status: "pending_user_review" }`. The model continues the conversation; the UI renders an approval card. The `Operation` union is defined in `04-api-contracts.md`. Each operation requires `rationale` and `references` (IDs of check-ins, reflections, evidence, or milestones that justify it). The tool validates that referenced IDs exist and that operations are internally consistent (for example a weekly plan whose hours exceed capacity is rejected with an error the model can correct).

### Tools deliberately absent in v1

- No external integrations (GitHub, Jira, calendar): excluded by the user's decision; the port exists in the `evidence` module for a later release.
- No web search or fetch: the coach answers from the user's data and general knowledge; adding server-side web tools would change caching and `pause_turn` handling and is a separate decision.
- No file, shell, or code execution tools.

## Autonomy policy

| Level | Behaviour | Default |
|---|---|---|
| `proposals_only` | Every plan-state change requires explicit approval in the inbox or inline card | Yes |
| `auto_apply_low_risk` | Operations tagged low-risk are applied immediately and shown as "applied by agent, undo available": `plan_item.set_status` when the user stated the status in the same check-in; `key_result.update_progress` when the user stated the value | Off |

Everything else (creating, moving, or closing milestones; weekly plans; evidence; objectives) is never auto-applied. Undo for auto-applied operations is a reverse operation recorded in the audit log.

## Prompt design

### Structure (per job type)

```
system (frozen, cached):
  1. Role and purpose for this job type
  2. Hard rules: propose-never-apply; cite references; never invent milestones or evidence; question limits for check-ins
  3. Domain glossary excerpt and status vocabularies (from 00-discovery/02-glossary.md)
  4. Output expectations (what a good proposal or check-in looks like)
  5. Competency model summary (domains and level names only; details via tool)
messages:
  1. user: "Context pack" text block with the dynamic state snapshot the job needs (profile summary, current week, template steps, agent notes), rendered deterministically
  2. ... conversation turns, tool calls and results ...
```

The context pack is sent as the first user message rather than in the system prompt so the system prefix stays byte-identical across sessions and is served from cache. Within a session the context pack is stable, so a second cache breakpoint on it pays off for multi-turn check-ins. Per-turn volatile data (current time) is appended as a short text block at the end of the latest user turn. When an operator instruction must change mid-conversation (for example switching a check-in from daily to weekly template on the user's request), it is appended as a `{"role": "system"}` message in `messages`, which the chosen model supports, instead of editing the top-level system prompt.

### Caching plan

| Prefix segment | Changes | Breakpoint | TTL |
|---|---|---|---|
| Tools (sorted) + system prompt per job type | Only on deploy | Explicit on last system block | Default 5-minute TTL. Within a session every turn refreshes it. Switch an interactive job type to the 1-hour TTL only if measured gaps between turns are commonly over five minutes (for example weekly reviews interrupted on a phone); the 1-hour write costs twice as much and only pays off in that window |
| Context pack (first user message) | Per session | Explicit on the context pack block | default |
| Conversation tail | Per turn | Top-level automatic `cache_control` | default |

Verification: `usage.cache_read_input_tokens` is stored per request; the usage page shows cache hit ratio per job type; a ratio under 50% for interactive jobs is a bug.

Silent invalidators to grep for in review: `Date.now()` or timestamps inside system or context pack, unsorted JSON serialisation, conditional system sections, per-session IDs early in the prefix, tool list built dynamically.

### Check-in templates

Templates are data in `packages/agent/src/templates/*.ts`: ordered steps with intent, maximum questions, which tools to use, and completion criteria. The agent follows them; the template name and steps are part of the context pack. v1 templates: `daily` (3 questions max), `weekly_review` (walk items, lessons, evidence prompts), `monthly_retro` (track health, neglected domains, sustainability), `quarterly_review` (objectives vs baselines, re-assessment prompts), `adhoc`.

### Guardrails in prompts

- Never claim an item exists without having read it through a tool in this session.
- Quote the user's words when recording reflections; do not paraphrase into stronger claims.
- Prefer fewer, higher-value operations; one proposal per check-in unless the user asks for more.
- When uncertain, ask one clarifying question rather than guessing; in background jobs, list open questions in the proposal summary instead.

## Memory and context management

| Need | Mechanism |
|---|---|
| Long-term facts about the user | `agent_notes` table via `remember_note`; injected in the context pack; user-editable |
| Plan state | The database, read through tools on demand; never duplicated into notes |
| Within a session | Full message history stored and resent; sessions are bounded (a check-in is typically under 20 turns) |
| Long coach threads | Thread history resent; if a thread approaches the configured token ceiling, the runtime opens a new thread seeded with a stored summary (server-side compaction is an option to evaluate later, kept off in v1 for simplicity) |
| Cross-session continuity | Check-in summaries and recent reflections in the context pack |

## Scheduling and background execution

- pg-boss schedules in the worker: `plan.weekly` (default Sunday 18:00 local), `roadmap.replan.detect` (nightly 02:00 local; runs the agent only when slips exist and no re-plan proposal is pending), reminders (`checkin.remind.daily`, `.weekly`, `.monthly`, `.quarterly`), `usage.rollup` (daily), `budget.check` (daily), `proposals.expire` (daily).
- Each agent job is a pg-boss singleton per idempotency key with retry limit 2 and exponential backoff; a failed job creates a notification with the reason and a link to the session record.
- The budget guard runs before every background agent job and before starting an interactive session; see cost controls.

## Cost controls

| Control | Mechanism |
|---|---|
| Record everything | `llm_requests` row per API response: model, effort, input, cache creation, cache read, output tokens, cost computed from a price table in settings, latency, stop reason |
| Monthly budget | Setting in USD; `budget.check` compares month-to-date spend; at 80% notify; at 100% pause background jobs and require confirmation for interactive sessions |
| Per-session cap | Max tool iterations and max cumulative tokens per session by job type; exceeding ends the session gracefully with a stored summary and a notification |
| Effort tuning | Per-job effort setting; lower effort before changing model (ADR-0006) |
| Caching | As above; cache hit ratio visible; TTL per job type is a setting with the 5-minute default |

Indicative monthly cost at default settings, assuming the default model's list prices and the caching plan above. These are planning numbers, not commitments.

| Activity per month | Volume | Approx. USD |
|---|---|---|
| Daily check-ins | 22 × about 6 turns | 6 to 9 |
| Weekly reviews | 4 × about 12 turns | 3 to 5 |
| Weekly plan proposals | 4 | 1 to 2 |
| Monthly retro and re-plans | 1 to 4 agent runs | 1 to 3 |
| Coach chats | about 10 short threads | 2 to 4 |
| Roadmap draft (first month only) | 1 | 1 to 2 |
| **Total** | | **roughly 15 to 25** |

## Failure handling

| Failure | Handling |
|---|---|
| API error (rate limit, 5xx) | SDK retries (default 2) then job retry via pg-boss; interactive turn shows a retry button; the stored user message is not lost |
| `stop_reason: "refusal"` | Server-side fallback is enabled; if the whole chain refuses, the session is marked failed with the category, the user sees a plain explanation, no tools from that turn run |
| `stop_reason: "max_tokens"` | Treated as failure for tool-bearing turns (inputs may be truncated); the runtime retries once with a higher cap for background jobs |
| Tool input fails schema validation | Return `is_error: true` tool result with the validation message so the model can correct |
| Proposal references unknown IDs | Tool returns an error listing the unknown IDs; the model re-reads and retries |
| Worker crash mid-job | pg-boss re-delivers; the job checks for an existing proposal for its idempotency key before creating another |
| Web restart mid-turn | Browser reconnects to the SSE stream; if the turn did not complete, the UI offers to resend; stored messages are intact |

## Model behaviours that constrain the harness

Recorded here so the plan and tasks do not fight the API.

- **Thinking always on; effort is the dial.** No `thinking.budget_tokens`; `output_config.effort` per job.
- **No forced tool choice.** Use prompt steering and `strict: true`.
- **Preserved thinking.** Thinking blocks are bound to the model and conversation; editing earlier turns can invalidate them and, for newer accounts, return an error. The message store is append-only and the runtime never rewrites history. Model switches start a new session.
- **Refusal fallbacks.** `fallbacks: "default"` with the corresponding beta header, so category-based fallback happens server-side.
- **Streaming for long outputs.** Interactive turns and large structured outputs use streaming to avoid HTTP timeouts.
- **Cache minimum prefix.** The chosen model caches prefixes above a small minimum; the system prompt plus tools is well above it.

## Observability for the agent

- Per session: timeline of messages, tool calls with durations, usage per request, cost, cache ratio, stop reasons; visible in the UI under the session record linked from each proposal and check-in.
- Logs: one structured log line per request and per tool call with `sessionId`, `jobType`, `requestSeq`.
- Metrics (optional OpenTelemetry): tokens by job type, cost by day, cache ratio, tool error rate, session failure rate.

## Evaluation

See `08-quality-and-testing.md` for the harness. In short: scenario fixtures (profiles, roadmaps, transcripts) with programmatic checks (capacity respected, references valid, question limits, no invented IDs) and rubric grading by a judge model for qualities such as specificity and tone. Runs nightly against the real model; recorded fixtures guard the runtime logic in unit tests without network.
