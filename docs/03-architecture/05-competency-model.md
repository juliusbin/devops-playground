# Competency Model and Results Framework

The built-in model that seeds roadmap drafting and structures self-assessment. It ships as versioned data in `packages/competency-model` and is loaded into the `competency_*` tables by the seed. The user may edit descriptors later; edits create a new model version so history stays comparable.

This is a starting model for a Software Architect role in a product engineering organisation. `[NEEDS CLARIFICATION: does the user's company publish an architect ladder or competency framework that should replace or extend this model?]`

## Levels

| Level | Name | Meaning |
|---|---|---|
| L1 | Aware | Knows the concepts and vocabulary; can follow others' work |
| L2 | Practitioner | Applies the competency within own team with guidance; produces usable artefacts |
| L3 | Leads | Owns the competency for a team or system; others rely on their judgement; teaches it |
| L4 | Shapes | Sets direction across teams or the organisation; defines standards; influences strategy |

A Software Engineering Lead typically sits at L2–L3 in team-facing competencies and L1–L2 in organisation-facing ones. An architect role typically expects L3 across most domains and L4 in two or three.

## Domains and competencies

### D1. Architecture design and trade-offs

| Key | Competency | L2 behaviours (examples) | L3 behaviours (examples) | Example evidence |
|---|---|---|---|---|
| `d1.styles` | Selects architectural styles and patterns | Explains when to use modular monolith vs. services; applies patterns in own system | Chooses style for a new system with explicit trade-offs; reviews others' designs | Design document with alternatives considered |
| `d1.tradeoffs` | Analyses trade-offs explicitly | Lists pros and cons for a decision | Uses quality-attribute scenarios and fitness functions to compare options; makes the trade-off visible to stakeholders | Trade-off matrix, ADR with consequences |
| `d1.adr` | Records decisions | Writes ADRs for team decisions | Establishes ADR practice; curates the decision log; revisits superseded decisions | ADR log, review of decision outcomes |
| `d1.review` | Runs design reviews | Participates and gives structured feedback | Facilitates reviews across teams; sets review criteria | Review notes, checklist authored |
| `d1.evolution` | Plans evolutionary architecture | Refactors toward a target within a team | Defines target architecture and incremental migration path with checkpoints | Migration plan, strangler-fig rollout |

### D2. Quality attributes and systems thinking

| Key | Competency | L2 | L3 | Example evidence |
|---|---|---|---|---|
| `d2.nfr` | Elicits and specifies quality attributes | Writes measurable NFRs for own service | Negotiates NFRs with product and ops; prioritises conflicting attributes | NFR specification, SLO definitions |
| `d2.reliability` | Designs for reliability and resilience | Adds retries, timeouts, health checks | Designs failure modes, degradation paths, capacity plans; runs failure injection | Resilience design, incident postmortem with architectural actions |
| `d2.performance` | Designs for performance and scalability | Profiles and fixes hot paths | Sets performance budgets; models load; chooses scaling strategy | Load test report, capacity model |
| `d2.observability` | Designs for observability | Adds structured logs and metrics | Defines SLIs, dashboards, and alert strategy as part of design | Observability design, SLO review |
| `d2.systems` | Thinks in systems | Maps dependencies of own service | Models cross-system flows, feedback loops, and second-order effects | System context diagram, dependency risk analysis |

### D3. Technical strategy and roadmapping

| Key | Competency | L2 | L3 | Example evidence |
|---|---|---|---|---|
| `d3.vision` | Articulates technical vision | Contributes to team tech goals | Writes a vision for a system area linked to business outcomes | Vision document |
| `d3.roadmap` | Builds technical roadmaps | Plans a quarter of tech work | Sequences multi-quarter initiatives with dependencies and outcomes | Roadmap with milestones and measures |
| `d3.buildbuy` | Makes build/buy/reuse decisions | Compares libraries | Runs evaluations with criteria, cost, and exit strategy | Evaluation report |
| `d3.debt` | Manages technical debt deliberately | Tracks debt in own system | Quantifies debt impact; negotiates paydown with product | Debt register, paydown outcomes |
| `d3.cost` | Reasons about total cost | Knows own service's cost drivers | Models TCO and cost-to-serve; drives cost decisions | Cost analysis, savings realised |

### D4. Domain modelling and data architecture

| Key | Competency | L2 | L3 | Example evidence |
|---|---|---|---|---|
| `d4.ddd` | Models domains and boundaries | Uses ubiquitous language; models aggregates | Defines bounded contexts and context maps across teams | Context map, event storming output |
| `d4.data` | Designs data architecture | Designs schemas for own service | Chooses storage per workload; designs data flows, ownership, and lifecycle | Data architecture document |
| `d4.consistency` | Handles consistency and integration | Uses transactions correctly | Designs eventual consistency, idempotency, and integration contracts | Integration design, contract tests |
| `d4.api` | Designs APIs and contracts | Designs REST or event APIs for own service | Sets API standards; manages versioning and compatibility across consumers | API guidelines, versioning policy |

### D5. Platform, cloud, and delivery

| Key | Competency | L2 | L3 | Example evidence |
|---|---|---|---|---|
| `d5.infra` | Designs infrastructure and runtime topology | Containers and CI for own service | Designs environments, networking, and runtime platform choices for several systems | Infrastructure design, IaC |
| `d5.delivery` | Designs delivery pipelines and release strategy | Owns team pipeline | Defines deployment strategies (blue/green, canary), release governance | Pipeline design, rollout policy |
| `d5.platform` | Shapes platform and developer experience | Uses platform capabilities well | Identifies platform gaps; drives paved-road capabilities | Platform proposal, adoption metrics |
| `d5.ops` | Designs for operability | Runbooks for own service | On-call model, operational readiness reviews, capacity management | Operational readiness checklist |

### D6. Security, compliance, and risk

| Key | Competency | L2 | L3 | Example evidence |
|---|---|---|---|---|
| `d6.threat` | Threat models systems | Participates in threat modelling | Leads threat modelling; integrates it into design reviews | Threat model document |
| `d6.secure` | Applies secure-by-design principles | Follows secure coding and auth patterns | Designs identity, authorisation, secrets, and data protection across systems | Security architecture section |
| `d6.compliance` | Designs for compliance and privacy | Knows applicable requirements | Translates regulations into architectural controls and evidence | Control mapping |
| `d6.risk` | Manages architectural risk | Lists risks for own project | Maintains risk register with mitigations; communicates risk to leadership | Risk register, mitigation outcomes |

### D7. Communication, influence, and stakeholders

| Key | Competency | L2 | L3 | Example evidence |
|---|---|---|---|---|
| `d7.diagrams` | Communicates architecture visually | Produces C4-style diagrams for own system | Standardises diagrams; tailors views to audience | Diagram set, presentation |
| `d7.writing` | Writes clear technical documents | Design docs for own team | Documents read and used across teams; sets templates | Document with cross-team readership |
| `d7.presenting` | Presents to varied audiences | Presents to team | Presents to leadership and non-technical stakeholders; handles challenge | Talk, exec briefing |
| `d7.influence` | Influences without authority | Persuades own team | Builds alignment across teams; resolves conflicting interests | Decision adopted across teams |
| `d7.facilitation` | Facilitates decisions | Runs team design discussions | Facilitates cross-team decision forums; drives to closure | Decision forum outcomes |

### D8. Leadership, mentoring, and organisational design

| Key | Competency | L2 | L3 | Example evidence |
|---|---|---|---|---|
| `d8.mentoring` | Grows engineers in architecture skills | Mentors individuals | Runs guilds or learning programmes; raises team design capability | Mentoring outcomes, programme |
| `d8.topologies` | Aligns teams and architecture | Understands Conway's law | Proposes team boundaries aligned with system boundaries | Team topology proposal |
| `d8.governance` | Establishes lightweight governance | Follows standards | Defines standards and review processes that enable rather than block | Governance model, adoption data |
| `d8.business` | Connects architecture to business outcomes | Understands product goals | Frames architectural choices in business terms; quantifies impact | Business case, outcome measures |

## Results framework for the current role

Used to shape quarterly objectives so that "proven results" are explicit. Each objective picks one result area.

| Key | Result area | Typical key results | Architect-visible behaviours at lead level |
|---|---|---|---|
| `r.delivery` | Delivery predictability and flow | Lead time, planned vs. delivered, release frequency | Sequence work with architectural dependencies made explicit |
| `r.quality` | Quality and reliability | Change failure rate, incidents, SLO attainment, defect escape rate | Introduce fitness functions, resilience patterns, observability |
| `r.team` | Team growth and health | Skills coverage, retention, onboarding time, engagement signals | Run design reviews, mentor in architecture, build a decision log culture |
| `r.excellence` | Technical excellence and debt | Debt paydown, platform adoption, test coverage trends, cost efficiency | Own a modernisation initiative with a target architecture |
| `r.stakeholders` | Stakeholder outcomes | Satisfaction, adoption, business metrics moved | Present trade-offs to product and leadership; align roadmaps |

## How the agent uses this model

- **Gap analysis:** for each domain, the average of latest self-assessment levels vs. the target level (default L3, L4 for two domains the user picks or the agent proposes based on the role context).
- **Roadmap drafting:** proposes milestones per domain with a gap, favouring those that can be achieved through current-role work (dual-purpose), sequenced by dependency (for example `d1.adr` before `d8.governance`) and spread across quarters to respect weekly capacity.
- **Monthly retro:** reports domains with no activity or evidence in the month.
- **Quarterly review:** suggests self-assessment updates for competencies that gained evidence.

## Versioning

`packages/competency-model/model.v1.json` is the seed. Changing descriptors creates `model.v2.json`; self-assessments reference the version they were made against; competency `key` values are stable, so gap history survives version changes.
