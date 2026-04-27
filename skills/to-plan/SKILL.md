---
name: to-plan
description: Turn a feature specification into crosscutting implementation tasks with test scenarios and acceptance criteria, saved as TASKS.md in the feature folder. Use when the spec is finalized and ready for implementation planning.
---

# To Plan

Break a feature specification into crosscutting implementation tasks using vertical slices. Output is `TASKS.md` in the feature's spec folder.

## Process

### Step 1: Load Feature Context

1. Run `git branch --show-current` to determine the active branch.
2. Derive the feature folder:
   - Strip the `feat/` prefix from the branch name.
   - Extract the first path segment (everything before the second `/`) — this is the feature slug.
   - Feature folder = `spec/features/[slug]/`
   - Example: `feat/0023-payment-retry` → `spec/features/0023-payment-retry/`
3. If not on a feature branch (no `feat/` prefix), fail with a clear message.
4. Read all spec documents in order:
   - `spec/GLOSSARY.md`
   - Feature's `INDEX.md`
   - Feature's `REQUIREMENTS.md`
   - Feature's `ADR.md`
   - Feature's `DESIGN.md`
5. Follow cross-links to `spec/latest/` one level deep to understand the broader context.

### Step 2: Explore the Codebase

If you have not already explored the codebase, do so to understand:

- Current architecture and existing patterns
- Integration layers (API, database, UI)
- Existing test patterns and frameworks
- Code that will be modified by this feature

### Step 3: Draft Vertical Slices

Break the feature into **crosscutting tasks**. Each task is a thin vertical slice that cuts through ALL relevant layers end-to-end — NOT a horizontal slice of one layer.

<vertical-slice-rules>
- Each task delivers a narrow but COMPLETE path through every layer it touches (schema, API, UI, tests)
- A completed task is verifiable on its own
- Prefer many thin tasks over few thick ones
- Tasks typically include both UI and Backend work
- Each task must reference the spec sections it implements
- Order tasks so dependencies are built first
</vertical-slice-rules>

### Step 4: Quiz the User

Present the proposed task breakdown. For each task show:

- **Title**: short descriptive name
- **Spec sections covered**: which requirements and design sections this addresses
- **Layers touched**: which parts of the stack (API, DB, UI, etc.)

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Should any tasks be merged or split further?
- Is the ordering correct?

Iterate until the user approves the breakdown.

### Step 5: Write TASKS.md

Write `TASKS.md` in the feature folder using this template:

<tasks-template>
# Tasks: [Feature Name]

## TASK-001: [Task Title]

### Description

Comprehensive description of what needs to be done. Include which layers are affected and how they connect.

### References

- spec://features/NNNN-slug/DESIGN.md#relevant-section
- spec://features/NNNN-slug/REQUIREMENTS.md#relevant-requirement

### Test Scenarios

#### TS-001: [Scenario Name]

Verifies: spec://features/NNNN-slug/REQUIREMENTS.md#requirement-section

Description of the test scenario, inputs, and expected outcomes.

#### TS-002: [Scenario Name]

Verifies: spec://features/NNNN-slug/REQUIREMENTS.md#another-requirement

Description of the test scenario, inputs, and expected outcomes.

### Acceptance Criteria

#### AC-001: [Criterion]

Clear, verifiable statement of what must be true when this task is complete.

#### AC-002: [Criterion]

Clear, verifiable statement of what must be true when this task is complete.

---

## TASK-002: [Task Title]

### Description

...

### References

...

### Test Scenarios

...

### Acceptance Criteria

...

<!-- Repeat for each task -->
</tasks-template>

### Step 6: Commit and Push

1. Stage `TASKS.md`.
2. Commit with message: `feat(spec): add implementation tasks for NNNN-slug`
3. Push to the feature branch.

### Step 7: Report

Present a summary:

- Total number of tasks
- Brief description of each task
- Suggest running `/to-linear` to create Linear issues
