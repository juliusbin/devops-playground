# ADR-0008: Manual evidence capture; integration ports deferred

**Status:** Accepted · **Date:** 2026-10-05

## Context

During discovery the user excluded "pull evidence from external tools" from the agent's autonomy. Evidence is still central to the product (promotion case). Integrations (GitHub, Jira or Linear, calendar, documents) add OAuth flows, token storage, sync jobs, and privacy questions about employer data.

## Decision

In v1 evidence is **captured manually** through a fast form with link suggestions, and the agent may **propose** pre-filled evidence items from check-in conversations. The `evidence` module exposes an `EvidenceSuggestionPort` interface (input: period and user context; output: suggested evidence drafts) with no adapters implemented. A later feature (016) may add read-only adapters running in the worker that create proposals, never evidence directly.

## Alternatives considered

- **GitHub integration in v1.** The most tempting source for a lead, but it would import employer data into a personal system and requires per-organisation approval; deferred deliberately.
- **Browser extension clipper.** Lower integration burden; possible later as a client of the normal evidence API.

## Consequences

- Evidence quality depends on the weekly review habit; the agent's evidence prompts during reviews are therefore P1, not optional.
- The port keeps the future integration a contained addition with the proposal flow already in place.

## Revisit triggers

- The user finds manual capture is the main reason evidence is missing (check the evidence gap report after one quarter).
