---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

# Grill Me

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

If a question can be answered by exploring the codebase, explore the codebase instead.

## Topic Coverage

Read `skills/shared/TOPICS-CRITERION.md` at the start of the session. This is your topic inventory. Use it to guide the interview naturally from broad context to narrow details.

### Interview Flow

1. **Start broad** — understand the problem, motivation, and who the feature is for. Cover required topics first: purpose, scope, user roles.
2. **Narrow into requirements** — probe business requirements, technical requirements, and constraints. Ensure failure expectations are embedded in requirements where relevant.
3. **Detect conditional topics organically** — as the conversation develops, watch for keyword signals defined in the criterion. When signals appear, dig deeper into the triggered conditional topic naturally, without announcing that a topic has been "triggered."
4. **Explore design implications** — as requirements solidify, probe architecture, component design, and any activated conditional design topics (error handling, consistency, migration, etc.).
5. **Cover verification** — ensure enough context exists to define test scenarios as Gherkin-style behaviors grouped by user flow.

### What NOT to do

- Do NOT ask screening questions as a block or checklist.
- Do NOT mention the criterion document, topic names, or tiers to the user.
- Do NOT follow a rigid topic order — let the conversation flow naturally.
- Do NOT skip a required topic because the user hasn't raised it — steer the conversation toward it.

### Internal Tracking

Track internally which topics have been covered and to what depth. Before declaring the interview complete:

1. Check all **required** topics against their adequacy criteria.
2. Check all **conditional** topics whose keyword signals appeared during the conversation.
3. If any topic has insufficient coverage, circle back with targeted questions.

### Detecting ADR-Worthy Decisions

When any topic surfaces a non-obvious technical choice with meaningful alternatives (signals: "we could also", "trade-off", "debated between", "alternative", "versus"), note it internally as an ADR candidate. Probe the alternatives, trade-offs, and consequences.

## Completion

After collecting all of the information, relentlessly reevaluate the resulting model for flaws, inconsistencies, or ambiguity. If you find any, reiterate the Q&A session, reevaluate again. Repeat the process until no problems remain.

When complete, confirm that sufficient context has been gathered for `/to-spec` to produce:
- INDEX.md (purpose, scope, user roles)
- REQUIREMENTS.md (business, technical, constraints, plus any conditional requirement topics)
- DESIGN.md (architecture overview, plus any conditional design topics)
- SCENARIOS.md (Gherkin-style test scenarios by user flow)
- ADR.md (if non-obvious decisions were identified)
