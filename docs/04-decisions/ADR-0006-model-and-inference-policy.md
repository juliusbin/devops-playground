# ADR-0006: Model and inference policy

**Status:** Accepted · **Date:** 2026-10-05

## Context

Agent quality drives the product's value; cost must stay within a personal budget; the Claude API has specific behaviours on current models that constrain the harness. The user did not name a specific model.

## Decision

1. **Default model:** `claude-opus-5-5` for every job type. The model is a setting; changing it starts new sessions (caches and thinking blocks are model-scoped).
2. **Thinking:** adaptive thinking as provided by the model; no thinking budget parameter. Depth is controlled through `output_config.effort`.
3. **Effort per job type** is the first cost lever: `xhigh` for roadmap drafting, `high` for re-planning and weekly planning, `medium` for check-ins, coaching, and link suggestions. Tune by measurement using the evaluation suite before changing model.
4. **Prompt caching:** frozen system prompt and sorted tool list per job type with an explicit cache breakpoint; per-session context pack as a second breakpoint; automatic caching for the conversation tail; default 5-minute TTL everywhere, with the 1-hour TTL available per job type only where measured gaps between turns are commonly over five minutes. Cache hit ratio is recorded and displayed.
5. **Structured outputs** (`output_config.format` with Zod) for single-artefact background jobs; `strict: true` on all tools; no forced tool choice.
6. **Refusal handling:** server-side fallbacks enabled (`fallbacks: "default"` with the corresponding beta header); refused turns run no tools and are reported to the user.
7. **Streaming** for interactive turns and large outputs; non-streaming only for small structured calls.
8. **Append-only history:** stored messages are never edited; a model switch starts a new session. This keeps caches valid and respects the preserved-thinking rules on current models.
9. **Budget:** monthly USD cap in settings; 80% warning; 100% pauses background jobs and gates interactive sessions behind a confirmation.
10. **Price table** for cost computation lives in settings and is updated with releases.

## Alternatives considered

- **A cheaper model for routine jobs by default.** Rejected as a default: lower effort on the most capable model is the simpler first lever, keeps one cache namespace, and avoids quality regressions in exactly the jobs (check-ins) where tone and specificity matter most. The model remains user-selectable.
- **Disabling thinking for chat.** Not available on the chosen model and not desirable; `medium` or `low` effort achieves the cost goal.
- **Client-side JSON parsing of free text for proposals.** Rejected in favour of structured outputs and strict tools.

## Consequences

- Predictable monthly cost in the planning range given in `03-architecture/02-agent-architecture.md`.
- The runtime must store full content blocks (including thinking) and never rewrite history.
- Evaluation suite results are required evidence for any change to model, effort defaults, or prompts.

## Revisit triggers

- A new model generation changes pricing or API behaviour (follow the provider migration guide; update price table and prompts; re-run evals).
- Measured eval quality at lower effort equals current defaults (lower the defaults).
