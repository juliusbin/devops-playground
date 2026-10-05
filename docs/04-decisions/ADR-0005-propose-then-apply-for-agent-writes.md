# ADR-0005: Propose-then-apply for all agent writes

**Status:** Accepted · **Date:** 2026-10-05

## Context

The user wants the agent to draft and adjust the roadmap and weekly plans and to run check-ins, but chose a hybrid model: the agent proposes, the user curates. The agent's output is probabilistic; the roadmap and plan are the user's commitments. Trust requires that nothing changes without consent and that every change is explainable.

## Decision

All agent-originated changes to roadmap, milestones, activities, objectives, key results, weekly plans, plan items, evidence, and self-assessments are expressed as a **Proposal** containing **Operations**, each with a rationale and references. Proposals are reviewed in an inbox or inline card; the user approves, edits, or rejects per operation; approved operations are applied transactionally by the `proposals` service calling the same domain use cases the UI uses, with `actor = agent` and a `via_proposal_id`.

Append-only records (reflections quoting the user, check-in summaries, agent notes) may be written directly by the agent because they do not change commitments and are visible and deletable.

An optional **auto-apply low-risk** setting (off by default) lets two operation types apply immediately with undo: plan item status and key result progress, only when the user stated the fact in the same conversation.

## Alternatives considered

- **Direct writes with undo.** Faster, but shifts the burden to noticing and reverting; contradicts the user's stated preference.
- **Approval through chat only.** Fine inside a check-in, but background jobs have no conversation; a persistent inbox is needed anyway.
- **Confirm each tool call synchronously (human-in-the-loop blocking).** Stalls background jobs and makes interactive sessions choppy; proposals decouple review from generation.

## Consequences

- A `proposals` module with its own schema, API, and UI is a prerequisite for every agent feature (feature 004 before 005).
- Domain services need an actor-aware mutation path and an audit log; this is constitution II's compliance check.
- The agent's tool surface is small and uniform: one `propose_changes` tool, many read tools.
- Operations must be validated at creation so the model can self-correct before the user sees them.

## Revisit triggers

- The user reports approval fatigue for a class of operations: extend the low-risk set deliberately, with undo and audit.
