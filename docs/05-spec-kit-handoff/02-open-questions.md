# Open Questions

Consolidated `[NEEDS CLARIFICATION]` items from the docs, for `/speckit-clarify`. Record answers here so every later feature inherits them. None of these block the architecture; each has a proposed default.

| # | Question | Where it matters | Proposed default | Answer |
|---|---|---|---|---|
| Q1 | Target date for the architect role, and whether it is a hard deadline or an aspiration | Roadmap drafting pacing (005) | 24 months from first use; aspiration | |
| Q2 | Weekly hours realistically available for growth work, and whether weekends count | Capacity, weekly planning (007) | 6 hours; weekdays only | |
| Q3 | Does the employer publish an architect ladder or competency framework to import or merge with the built-in model? | Competency model (002, 005) | Use the built-in model; add company competencies as a new model version later | |
| Q4 | Default check-in times, and whether daily check-ins run on weekends | Check-ins and schedules (008, 011) | Daily 17:30 local on weekdays; weekly review Friday 16:00; weekly plan proposal Sunday 18:00; monthly retro first Monday; quarterly review first week of quarter | |
| Q5 | Default expiry for untouched proposals | Proposals (004) | 14 days; weekly plan proposals expire when the week starts and fall back to a manual week | |
| Q6 | Attachment size limit and total storage expectation | Evidence (009), deployment | 25 MB per file, 5 GB total quota | |
| Q7 | Must the app be reachable from a phone outside the home network, and is installing a VPN client (Tailscale) on the phone acceptable? | Exposure and auth (001), ADR-0007 | Yes via Tailscale; no public exposure | |
| Q8 | Preferred notification channel beyond in-app (email via own SMTP, self-hosted push such as ntfy, none) | Notifications (011) | In-app only in v1; ntfy later | |
| Q9 | Should the agent address the user by name and in which tone (direct and brief vs. warm and reflective)? | Prompts (008, 012) | Direct and brief, warm when discussing setbacks; configurable later via agent notes | |
| Q10 | Monthly LLM budget to start with | Settings (001), ADR-0006 | 40 USD | |
| Q11 | Quarter boundaries: calendar quarters or the employer's fiscal quarters | Objectives (006) | Calendar quarters | |
| Q12 | Is any employer-confidential information expected in evidence or reflections, requiring stronger guidance or a local-only mode for some items? | Privacy (006), prompts | Paraphrase guidance only; attachments never sent to the model | |
| Q13 | Which domains should target L4 (two or three) in the roadmap draft | Competency model, roadmap drafting (005) | Agent proposes based on profile context; user confirms during draft review | |
| Q14 | Should roadmap versions be diffable in the UI, or is a readable snapshot enough for v1 | Roadmap management (003) | Snapshot list with a simple field-level diff of milestones | |
| Q15 | Language and locale for dates and numbers | UI | English, locale from browser | |
