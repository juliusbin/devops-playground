# User Journeys

These journeys describe the product from the user's point of view. They are the narrative source for the user stories in `02-requirements/01-product-requirements.md`. Each journey names where the agent acts and where the user decides.

## J1. First run: from empty app to approved roadmap

1. The user signs in for the first time, sets a password, timezone, and week start day.
2. The user fills in the profile: current role, target role, target date, weekly hours for growth work, context, strengths, growth areas.
3. The user browses the competency model and self-assesses each competency (level, confidence, note). Skipping is allowed; unknowns are marked.
4. The user starts **roadmap drafting**. The agent reads the profile, self-assessment, and competency model, and produces a proposal: two tracks, milestones across the next two to four quarters with target dates, definitions of done, evidence requirements, and starter activities. Each milestone carries a rationale.
5. The user reviews the proposal in the inbox: approve, edit, or reject per milestone. Approved operations become the active roadmap.
6. The dashboard now shows the roadmap, the first upcoming milestones, and the next scheduled check-in.

Agent acts: step 4. User decides: step 5.

## J2. Setting the quarter's objectives

1. At the start of a quarter the user creates objectives for the current role, with key results and baselines. The results framework offers prompts and examples.
2. The user links objectives to roadmap milestones where the work overlaps. The UI highlights unlinked objectives and growth milestones with no delivery anchor.
3. Optionally, the user asks the agent to suggest links and dual-purpose milestones. The suggestions arrive as a proposal.

Agent acts: step 3 (optional). User decides: steps 1 to 3.

## J3. The weekly cycle

1. On the scheduled day (default Sunday evening), the worker runs **weekly plan drafting**. The agent reads active milestones, open activities, key results, last week's carry-over and close-out notes, and capacity. It proposes a plan: items with planned hours, a focus theme, and notes on what it deliberately left out.
2. The user receives a notification, opens the proposal, adjusts hours and items, approves. The week becomes active.
3. During the week the user updates plan item status from the week board, or through the daily check-in.
4. At week end, the **weekly review check-in** walks through each item: done, carried over, dropped; what was learned; what evidence should be captured. The agent proposes status changes, carry-overs, and evidence captures. The user approves. The week closes with a summary.

Agent acts: steps 1 and 4. User decides: steps 2, 3, 4.

## J4. The daily check-in (five minutes)

1. A reminder appears at the configured time. The user opens the daily check-in.
2. The agent asks three things at most: what moved, what is blocked, what is the one thing for tomorrow. It references today's plan items by name.
3. The user answers in free text. The agent records a reflection, proposes plan item status updates, and ends with a one-line summary.
4. The user approves the status updates with one tap, or lets them sit in the inbox.

Agent acts: steps 2 and 3. User decides: step 4.

## J5. Capturing evidence while it is fresh

1. After finishing a design review, writing a decision record, or receiving feedback, the user opens **Evidence → New**.
2. The user enters title, kind, link or attachment, date, and an impact statement (situation, task, action, result). The form suggests milestones and objectives to link, based on the current week's items.
3. Alternatively, during a check-in the agent notices a finished item that required evidence and proposes an evidence capture pre-filled from the conversation. The user completes and approves it.

Agent acts: step 3. User decides: steps 2 and 3.

## J6. Monthly retrospective and re-plan

1. The monthly check-in runs through: milestones on track or slipping, competencies touched or neglected, objectives trending, energy and sustainability.
2. The agent proposes a re-plan: move dates, split milestones, deprioritise, add activities, or flag a competency domain that has had no work. Each operation has a rationale.
3. The user approves or rejects per operation. A new roadmap version is recorded; history stays visible.

Agent acts: steps 1 and 2. User decides: step 3.

## J7. Quarterly review and the evidence read-through

1. The quarterly check-in reviews objectives and key results against baselines, closes or rolls them, and revisits the self-assessment for competencies with new evidence.
2. The user filters evidence by quarter and reads the impact statements end to end, editing where needed.
3. (Later release) The user asks for an on-demand report: a quarterly review narrative or a promotion-case draft composed from approved evidence and closed milestones.

Agent acts: step 1 (and 3, later). User decides: steps 1 to 3.

## J8. Asking the coach

1. At any time the user opens **Coach** and asks a question: "I have an architecture review on Thursday, how do I prepare given my roadmap?" or "Which milestone should I drop if I lose five hours a week this month?"
2. The agent answers using the user's actual data, and may attach a proposal if the conversation leads to a plan change.

Agent acts: step 2. User decides: whether to approve any proposal.

## J9. Keeping the system healthy

1. The user checks **Settings → Usage** to see LLM spend this month against the budget.
2. The user exports all data (JSON plus attachments) before a host migration and restores it on the new host.
3. The user reviews agent notes, deleting anything the agent should not remember.
