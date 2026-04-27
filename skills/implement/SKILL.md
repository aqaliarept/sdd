---
name: implement
description: Implement a task from the feature's TASKS.md. Creates a task branch, uses TDD to implement, adds spec:// references, creates SPEC-PROPOSAL.md if spec changes are needed, then commits, pushes, and creates a PR to the feature branch. Usage - /implement {ISSUE-ID}
---

# Implement

This skill implements a single task from the feature's `TASKS.md`. It creates a task branch, follows TDD methodology, adds `spec://` references to code, and creates a PR back to the feature branch.

## Arguments

The user provides the Linear issue ID: `/implement {ISSUE-ID}` (e.g. `/implement ENG-123`)

## Process

### Step 1: Load Context

1. Run `git branch --show-current` to determine the active branch.
2. Derive the feature folder:
   - Strip the `feat/` prefix from the branch name.
   - Extract the first path segment (everything before the second `/`) — this is the feature slug.
   - Feature folder = `spec/features/[slug]/`
   - Example: `feat/0023-payment-retry` → `spec/features/0023-payment-retry/`
3. If not on a feature branch (no `feat/` prefix), fail with a clear message.
4. Read `TASKS.md` and find the task heading matching the provided Linear issue ID (e.g. `## ENG-123: Retry Endpoint`).
5. If the task is not found, fail with a clear message listing available tasks.

### Step 2: Load Spec Context

Read the feature spec documents using the spec-navigator Research mode:

1. `spec/GLOSSARY.md`
2. Feature's `INDEX.md`
3. Feature's `REQUIREMENTS.md` — focus on sections referenced by the task
4. Feature's `ADR.md` — understand decisions and constraints
5. Feature's `DESIGN.md` — understand the intended implementation approach
6. Follow cross-links to `spec/latest/` one level deep

### Step 3: Create Task Branch

1. Ensure you are on the feature branch and it is up to date.
2. Derive a short description from the task title (lowercase, hyphenated).
3. Create the task branch: `feat/NNNN-slug/{issue-id}-short-description` (e.g. `feat/0023-payment-retry/eng-123-retry-endpoint`)
4. Switch to the task branch.

### Step 4: Implement Using TDD

Follow the `/tdd` skill methodology:

1. **Plan**: Review the task's test scenarios and acceptance criteria. Identify the behaviors to test and the order to implement them.

2. **Tracer Bullet**: Write ONE test for the first behavior → verify it fails (RED) → write minimal code to pass (GREEN).

3. **Incremental Loop**: For each remaining test scenario:
   - Write one test → fails (RED)
   - Write minimal code to pass (GREEN)
   - Refactor if needed (REFACTOR)
   - Run all tests to ensure nothing broke

4. **Test Scenarios**: The test scenarios from `TASKS.md` guide what to test. Each test should reference the spec requirement it verifies.

### Step 5: Add Spec References

During implementation, add `spec://` references to code following the spec-navigator Writing mode:

1. Place references in comments at the function/method/type level.
2. Reference `spec://features/NNNN-slug/...` for feature-specific behavior.
3. Reference `spec://latest/...` for behavior governed by existing subsystem specs.
4. Use the most specific section anchor available.
5. Update existing `spec://` references if the feature changes the behavior they describe.

### Step 6: Spec Proposals (If Needed)

If during implementation you discover that the spec needs changes (missing requirements, incorrect assumptions, design gaps):

1. Create or update `spec/features/NNNN-slug/SPEC-PROPOSAL.md`:

```markdown
# Spec Proposals

## SP-001: [Title]

### Task

{ISSUE-ID}

### Proposed Change

Which file and section to change, and what the change is.

### Rationale

Why this change is needed — what was discovered during implementation.
```

2. Proceed with implementation using the proposed spec changes as your guide. Do not block on approval.
3. Highlight the proposals in the PR description so reviewers can approve or reject them.

### Step 7: Commit and Push

1. Stage all new and modified files (code, tests, SPEC-PROPOSAL.md if created).
2. Commit with a descriptive message: `feat(NNNN-slug): implement {ISSUE-ID} - [brief description]`
3. Push the branch: `git push -u origin feat/NNNN-slug/{issue-id}-short-description`

### Step 8: Create PR

Create a pull request from the task branch into the feature branch using `gh pr create`:

```
## Summary

- What was implemented, referencing the Linear issue ID
- Key implementation decisions

## Linear

{ISSUE-ID}

## Spec References

- spec://features/NNNN-slug/REQUIREMENTS.md#section-1
- spec://features/NNNN-slug/DESIGN.md#section-2

## Test Scenarios Covered

- TS-001: [Scenario name] — PASS
- TS-002: [Scenario name] — PASS

## Spec Proposals

> Remove this section if no proposals were made.

The following spec changes are proposed in `SPEC-PROPOSAL.md`:

- SP-001: [Title] — [brief description of proposed change]

Please review and approve/reject each proposal in your review comments.
```

### Step 9: Report

Present a summary:

- Task branch name
- Files created/modified
- Tests written and their status
- Spec references added
- Any spec proposals created
- PR URL
- Suggest running `/address-review` after the PR review is done
