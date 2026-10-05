# Glossary

Domain language used across requirements, architecture, data model, and agent prompts. The data model uses these names as table and type names; the agent's tools use them in tool names and descriptions. Keep them stable.

| Term | Definition |
|---|---|
| **Profile** | The user's description of their situation: current role, target role, target date, weekly hours available for growth work, context (company, team, tech domain), strengths and growth areas. |
| **Competency model** | The built-in, versioned description of what a Software Architect does, organised as domains → competencies → levels, each level with observable behaviours and example evidence. Seeds roadmap drafting. See `03-architecture/05-competency-model.md`. |
| **Competency domain** | A top-level grouping in the competency model, for example "Architecture design and trade-offs". |
| **Competency** | A single capability within a domain, for example "Documents decisions as ADRs". |
| **Level** | One of four proficiency stages per competency: L1 Aware, L2 Practitioner, L3 Leads, L4 Shapes. |
| **Self-assessment** | The user's own rating of their current level per competency, with confidence and notes. Repeated over time. |
| **Results framework** | The built-in list of result areas for the current role (delivery, quality and reliability, team growth, technical excellence, stakeholder outcomes) used to shape objectives. |
| **Roadmap** | The user's curated plan toward the target role. One active roadmap at a time; versioned when re-planned. |
| **Track** | A lane within the roadmap. v1 has two kinds: `architect_growth` (competency-driven) and `role_results` (objective-driven). Milestones belong to exactly one track. |
| **Milestone** | A meaningful, dated outcome in a track with a definition of done and an evidence requirement. Can link to one competency and/or one objective. The unit of progress. |
| **Activity** | A concrete piece of work under a milestone (learn, practice, deliver, share, connect). The unit of weekly planning. |
| **Objective** | A quarterly goal in the current role, expressed as outcome language. Has one or more key results. |
| **Key result** | A measurable indicator for an objective: metric, baseline, target, current value. |
| **Dual-purpose work** | A milestone or activity that advances both a competency and an objective. The product surfaces and favours these. |
| **Weekly plan** | The set of plan items the user commits to for one week, with capacity and a focus theme. Proposed by the agent, approved by the user, closed at week end. |
| **Plan item** | An entry in a weekly plan pointing at an activity, milestone, key result, or an ad hoc task, with planned hours and a status. |
| **Check-in** | A time-boxed, agent-led conversation following a template (daily, weekly, monthly, quarterly, ad hoc). Produces reflections, proposals, and a summary. |
| **Reflection** | A journal entry written during or outside a check-in, tagged and linkable to milestones and objectives. Append-only. |
| **Coach chat** | An ad hoc conversation with the agent outside any check-in template. |
| **Evidence** | A manually captured proof of a result or capability: a design document, a decision record, a review, a metric, feedback, a presentation. Carries an impact statement and links. |
| **Impact statement** | A short structured narrative on an evidence item: situation, task, action, result. Written to be reusable in reviews. |
| **Proposal** | A set of changes the agent wants to make to plan state, with rationale and references. Pending until the user approves, edits, or rejects each operation. The only way agent output changes roadmap, plan, objective, or evidence state. |
| **Operation** | One atomic change inside a proposal, for example "set milestone status" or "create weekly plan". |
| **Agent session** | One run of the agent: an interactive check-in or coach chat, or a background job such as weekly plan drafting. Holds messages, tool calls, usage, and cost. |
| **Agent note** | A fact the agent chose to remember about the user (preferences, constraints) that is injected into later sessions. Editable by the user. |
| **Job** | A unit of background work executed by the worker: an agent session that runs without the user present, a reminder, a maintenance task. |
| **Schedule** | A cron-like rule that creates jobs: when weekly plans are proposed, when check-in reminders fire. |
| **Autonomy level** | A setting controlling what the agent may apply without approval. v1 default: proposals only. Optional: auto-apply low-risk operations. |
| **Budget** | The monthly LLM spend cap in USD. When reached, background agent jobs pause and the user is notified. |
