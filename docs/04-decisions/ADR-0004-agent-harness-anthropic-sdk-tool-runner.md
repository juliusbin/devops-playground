# ADR-0004: Agent harness: Anthropic TypeScript SDK tool runner

**Status:** Accepted · **Date:** 2026-10-05

## Context

During discovery the user chose "Claude API via Agent SDK" as the LLM option. Anthropic offers several ways to build an agent on Claude in TypeScript:

1. **Client SDK (`@anthropic-ai/sdk`) with a manual loop** – you write the request → tool → result loop.
2. **Client SDK tool runner** (`client.beta.messages.toolRunner` with `betaZodTool`) – the SDK drives the loop over tools you define; you host and persist everything.
3. **Managed Agents** – Anthropic hosts the agent loop and a per-session sandbox; configuration through the API.
4. **Claude Agent SDK (`@anthropic-ai/claude-agent-sdk`)** – the Claude Code harness as a library: spawns a bundled Claude Code binary per session; ships file, shell, grep, and web tools; custom tools attach through in-process MCP servers; permissions, hooks, sessions, subagents.

Career OS's agent needs typed **domain tools** backed by the application database (read roadmap, record reflection, create proposal), streaming to a browser, strict control of prompts for caching, structured outputs for proposals, full persistence of every message and usage row, and a budget guard. It needs no file system, shell, or web access. It runs inside the `web` and `worker` containers on the user's host.

Anthropic's own comparison table in the Agent SDK documentation routes "call the Claude API directly from your own code" to the client SDK and its tool runner, and reserves the Agent SDK for embedding Claude Code's agent (built-in coding tools) in an application.

## Decision

Use the **Anthropic TypeScript client SDK's tool runner** as the harness, wrapped by a small `AgentRuntime` in `packages/agent` that adds persistence, SSE streaming, iteration and token caps, budget checks, and job idempotency. Tools are `betaZodTool` definitions calling domain use cases. Single-artefact background jobs use `messages.parse` with Zod structured outputs.

This honours the intent behind the discovery answer (Claude models, tool use, structured output, prompt caching, TypeScript orchestration) while choosing the Anthropic library that fits a web service.

## Alternatives considered

- **Claude Agent SDK.** Rejected for v1. It would run a native Claude Code binary inside the web container, bring built-in tools that must be disabled for a non-coding agent, route custom tools through MCP, and reduce direct control over request shape (cache breakpoints, structured outputs, fallbacks). Its strengths (file editing, shell, coding workflows) are not needed. It remains the right choice if the agent ever has to operate on files or repositories, for example analysing the user's architecture documents in a repo; that would be a separate subagent and a new ADR.
- **Managed Agents.** Rejected for v1 because the user chose self-hosting and the hosted loop would move conversation state and tool execution orchestration to Anthropic's side; also, the agent's tools are database calls that must run next to the database. Revisit if scheduled agent work should run without the user's host being on, or if the operational burden of the worker becomes a problem.
- **Manual loop.** Possible, but the tool runner already provides the loop, streaming, and typed tools; the manual loop adds code without adding control that the runtime needs.

## Consequences

- One dependency for the agent (`@anthropic-ai/sdk` plus `zod`); no extra binary in images.
- The tool runner is a beta surface; pin the SDK version and cover the runtime with replay tests so SDK upgrades are safe.
- The runtime must handle `stop_reason` values itself (`refusal`, `max_tokens`), persist full content blocks, and implement its own session resumption; these are specified in `03-architecture/02-agent-architecture.md`.
- No `pause_turn` handling is needed while there are no server-side tools; adding web search later requires it.

## Revisit triggers

- A need for file or repository tools (consider the Claude Agent SDK as a subagent).
- A need to run agent jobs while the user's host is off (consider Managed Agents scheduled deployments).
- The tool runner leaves beta with breaking changes (upgrade path, not a redesign).
