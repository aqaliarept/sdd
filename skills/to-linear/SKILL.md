---
name: to-linear
description: Create Linear issues from TASKS.md in the current feature's spec folder. Uses Linear MCP integration. Updates TASKS.md with Linear issue IDs and URLs.
---

# To Linear

This skill reads `TASKS.md` from the current feature's spec folder and creates Linear issues for each task using the Linear MCP integration.

## Prerequisites

- Linear MCP server must be connected
- User must provide the Linear project name/ID as an argument: `/to-linear PROJECT-NAME`

## Process

### Step 1: Load Feature Context

1. Run `git branch --show-current` to determine the active branch.
2. Derive the feature folder:
   - Strip the `feat/` prefix from the branch name.
   - Extract the first path segment (everything before the second `/`) — this is the feature slug.
   - Feature folder = `spec/features/[slug]/`
   - Example: `feat/0023-payment-retry` → `spec/features/0023-payment-retry/`
3. If not on a feature branch (no `feat/` prefix), fail with a clear message.
4. Read `TASKS.md` from the feature folder. If it does not exist, fail and suggest running `/to-plan` first.

### Step 2: Parse Tasks

For each task in `TASKS.md`, extract:

- Task ID (e.g. `TASK-001`)
- Title
- Description
- References (spec links)
- Test scenarios
- Acceptance criteria

### Step 3: Create Linear Issues

For each task, create a Linear issue using the MCP integration:

1. **Title**: `[Task Title]` (use the task title from TASKS.md)
2. **Description**: Include the full task content — description, spec references, test scenarios, and acceptance criteria formatted in markdown.
3. **Project**: The project specified by the user argument.
4. **Labels**: Add appropriate labels if the project uses them.

Create issues in order so dependencies can be set up if needed.

### Step 4: Update TASKS.md

After each issue is created, perform two updates:

1. **Rename the task heading** from `TASK-NNN` to the Linear issue ID. This makes the Linear ID the canonical task identifier throughout the entire workflow.

   Before: `## TASK-001: Retry Endpoint`
   After: `## ENG-123: Retry Endpoint`

2. **Add a Linear link** under the task:

```markdown
### Linear

- Issue: [ENG-123](https://linear.app/workspace/issue/ENG-123)
```

### Step 5: Update Branch Naming Reference

After all issues are created, add a branch naming reference section at the top of `TASKS.md`:

```markdown
## Branch Naming Convention

Task branches use Linear issue IDs for automatic linking:
`feat/NNNN-slug/{issue-id}-short-description`
```

### Step 6: Commit and Push

1. Stage the updated `TASKS.md`.
2. Commit with message: `feat(spec): link tasks to Linear issues for NNNN-slug`
3. Push to the feature branch.

### Step 7: Report

Present a summary:

- Number of Linear issues created
- List of issue IDs with URLs
- Remind the user that task branches should use the format: `feat/NNNN-slug/{issue-id}-short-description` for automatic Linear linking
- Suggest running `/implement {ISSUE-ID}` to start implementation
