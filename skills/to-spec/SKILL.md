---
name: to-spec
description: Turn the current conversation context into structured specification documents under spec/features/. Use after /grill-me to create a feature branch and populate INDEX.md, REQUIREMENTS.md, ADR.md, DESIGN.md, and SCENARIOS.md. Do NOT interview the user — synthesize what you already know.
---

# To Spec

This skill takes the current conversation context (typically after a `/grill-me` session) and produces structured specification documents for a new feature. Do NOT interview the user — just synthesize what you already know from the conversation.

## Process

### Step 1: Load Criterion and Understand the Current State

1. Read `skills/shared/TOPICS-CRITERION.md` — this defines which topics to generate and their quality criteria.
2. If the project has `spec/CRITERION.md`, read it and apply overrides (add, elevate, skip).
3. Read `spec/GLOSSARY.md` to ground domain terminology.
4. Read `spec/latest/*/INDEX.md` files to understand existing subsystems and their scope.
5. Read relevant `spec/latest/` documents that the new feature may touch or depend on.
6. Analyze existing code where relevant to understand current implementation patterns.

Use the spec-navigator skill (Mode 3: Research) for navigation guidance.

### Step 2: Evaluate Topic Activation

Using the merged criterion (plugin defaults + project overrides):

1. **Required topics** — always include.
2. **Conditional topics** — scan conversation context for keyword signals. If signals are present, include the topic. If signals are present but context is insufficient, flag the topic as a gap in the report.
3. **Optional topics** — include only if rich context exists in the conversation.
4. **Skipped topics** — exclude (project override `skip: true`).

List the activated topics internally before generating documents.

### Step 3: Determine the Feature Number and Slug

1. Glob for existing feature folders: `spec/features/*/`
2. Parse the numeric prefix from each folder name.
3. Take the maximum number and add 1. If no features exist, start at `0001`.
4. Derive a short descriptive slug from the conversation context (e.g. `payment-retry-logic`).
5. The full feature folder name is `NNNN-slug` (e.g. `0023-payment-retry-logic`).
6. **Guard check**: If the feature folder already exists, fail with a clear message. Suggest `/spec-review` for modifying an existing spec.

Present the proposed slug to the user and confirm before proceeding.

### Step 4: Create the Feature Branch

1. Ensure you are on the main branch (main or master) and it is up to date.
2. Create a new branch: `feat/NNNN-slug` (e.g. `feat/0023-payment-retry-logic`).
3. Switch to the new branch.

### Step 5: Create the Feature Folder

Create the directory: `spec/features/NNNN-slug/`

### Step 6: Populate INDEX.md

Write `INDEX.md` with purpose, scope, user roles, and optionally success criteria:

```markdown
# [Feature Name]

## Purpose

One paragraph describing what this feature does and why it exists.

## Scope

### In Scope

### Out of Scope

## User Roles

### [Role Name]

Description of what this actor does in relation to this feature.
```

If the optional topic `success-criteria-and-metrics` is activated, add:

```markdown
## Success Criteria

### [Metric Name]

Target value, measurement method, and business impact.
```

### Step 7: Populate REQUIREMENTS.md

Write `REQUIREMENTS.md` with business and technical requirements, constraints, and any activated conditional requirement topics. Structure requirements as addressable subsections, not bullet lists.

```markdown
# [Feature Name] Requirements

## Business Requirements

### [Requirement Name]

Description. References [spec://features/NNNN-slug/INDEX.md#role-name].
Requirements with failure risk must state the failure expectation.

## Technical Requirements

### [Requirement Name]

Description. Verifiable through testing or inspection.
Requirements with failure risk must state the failure expectation.

## Constraints

### [Constraint Name]

Source: [regulatory | infrastructure | dependency | organizational].
Description of the constraint.
```

If conditional requirement topics are activated, add their sections:

```markdown
## Non-Functional Requirements

### [NFR Name]

Measurable target with units and conditions.

## Security Considerations

### [Consideration Name]

Threat and mitigation pair.

```

Every requirement that may be referenced from code or scenarios must be a markdown heading.

### Step 8: Populate ADR.md (Conditional)

Only create `ADR.md` if the conversation surfaced non-obvious decisions with meaningful alternatives (keyword signals: alternative, trade-off, debated, versus, chose).

```markdown
# [Feature Name] Architecture Decision Records

## ADR-001: [Decision Title]

### Context

Forces and constraints that drove this decision.

### Alternatives

1. **[Option A]** — Description. Trade-offs.
2. **[Option B]** — Description. Trade-offs.

### Decision

What was chosen and why.

### Consequences

What becomes easier and harder.
```

### Step 9: Populate DESIGN.md

Write `DESIGN.md` with architecture overview and any activated conditional design topics. References SCENARIOS.md for verification.

```markdown
# [Feature Name] Design

## Architecture Overview

How this feature integrates with the existing system.
Data flow for the primary use case.
References to relevant spec://latest/ subsystems.
```

Add activated conditional design topics as sections. Each section must meet the adequacy criteria from the criterion document. Possible sections:

```markdown
## Error Handling & Failure Modes

### [Failure Mode Name]

Trigger, system behavior, recovery/escalation path.
Traces back to failure expectation in REQUIREMENTS.md.

## Data Consistency & Integrity

How consistency is maintained during concurrent/partial operations.

## Disaster Recovery

Stateful components at risk, RPO/RTO, recovery procedures.

## Migration & Rollout Strategy

What changes, backwards compatibility approach, rollback procedure.

## Observability

Metrics, log events, alerting conditions.

## API Contracts & Interfaces

### [Interface Name]

Request/response schemas, error responses, versioning.

## Dependencies & Integration Points

### [Dependency Name]

Role, availability, testing impact, version constraints.

## User-Facing Behavior

User-visible touchpoints, flows, and feedback mechanisms.
```

### Step 10: Populate SCENARIOS.md

Write `SCENARIOS.md` with Gherkin-style test scenarios grouped by user flow. Each scenario links to requirements it verifies.

```markdown
# [Feature Name] Test Scenarios

## Suite: [User Flow Name]

Verifies:
- [spec://features/NNNN-slug/REQUIREMENTS.md#requirement-section]
- [spec://features/NNNN-slug/REQUIREMENTS.md#another-requirement]

Prose context explaining this user flow.

  Scenario: [Happy path scenario name]
    Given [precondition in domain language]
    When [action]
    Then [expected outcome]

  Scenario: [Failure path scenario name]
    Given [precondition]
    When [action that triggers failure]
    Then [expected failure behavior]

  Scenario Outline: [Parameterized scenario name]
    Given <parameter>
    When <action>
    Then <expected_result>

    Examples:
      | parameter | action | expected_result |
      | value1    | act1   | res1            |
```

**Scenario rules:**
- Use `Given/When/Then` keywords as mandatory structure
- Use domain language, not implementation details
- Group by user flow / behavior, not by individual requirement
- Use `Scenario Outline` with `Examples` tables for parameterized cases
- Cover both happy path and failure paths
- Prose context between Gherkin blocks is permitted for clarity

### Step 11: Validate Against Adequacy Criteria

Before committing, validate each generated section against its adequacy criteria from the criterion document. Flag any sections that don't meet the criteria — include these flags in the report.

### Step 12: Detect Changes Needed in Latest

This is critical. Compare the new feature spec against `spec/latest/`:

1. For each subsystem the feature touches, read the corresponding `spec/latest/[subsystem]/` documents.
2. Identify if the feature requires changes to existing requirements, decisions, or designs in `latest/`.
3. If changes are needed:
   - Create subsystem subfolders within the feature folder mirroring the `latest/` structure (e.g. `spec/features/NNNN-slug/orders/REQUIREMENTS.md`).
   - Include only the sections that change — not the entire document.
   - Add cross-links from the feature spec to the relevant `latest/` sections using `spec://latest/...` references.
4. Document in ADR.md any decisions that override or extend existing `latest/` decisions.

### Step 13: Update GLOSSARY.md

If the feature introduces new domain terms, add them to `spec/GLOSSARY.md`. Each term is a subsection with its definition.

### Step 14: Commit and Push

1. Stage all new and modified files.
2. Commit with message: `feat(spec): add specification for NNNN-slug`
3. Push the branch: `git push -u origin feat/NNNN-slug`

### Step 15: Report

Present a summary to the user:

- Feature slug and branch name
- List of created files
- Activated topics (required + conditional that triggered)
- Skipped conditional topics (with reason: no signals detected)
- Adequacy gaps flagged (if any sections don't fully meet criteria)
- Cross-links to `latest/` subsystems identified
- Any new glossary terms added
- Suggest running `/spec-review` to refine the specification
