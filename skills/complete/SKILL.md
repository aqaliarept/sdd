---
name: complete
description: Complete a feature by merging its spec into latest/ and creating a PR to trunk. Use when all tasks are implemented and merged into the feature branch.
---

# Complete

This skill finalizes a feature by merging the feature's spec documents into `spec/latest/` and creating a PR from the feature branch to trunk (main/master).

## Process

### Step 1: Verify Feature State

1. Run `git branch --show-current` to determine the active branch.
2. Derive the feature folder:
   - Strip the `feat/` prefix from the branch name.
   - Extract the first path segment (everything before the second `/`) — this is the feature slug.
   - Feature folder = `spec/features/[slug]/`
   - Example: `feat/0023-payment-retry` → `spec/features/0023-payment-retry/`
3. If not on a feature branch (no `feat/` prefix), fail with a clear message.

### Step 2: Verify All Tasks Complete

1. Read `TASKS.md` from the feature folder.
2. For each task with a Linear issue ID, check via `gh` that the task's PR has been merged into the feature branch.
3. If any tasks have unmerged PRs, report them and fail. All tasks must be merged before completing.

### Step 3: Verify No Pending Spec Proposals

1. Check if `SPEC-PROPOSAL.md` exists in the feature folder.
2. If it exists and contains unresolved proposals, fail with a clear message. All proposals must be resolved before completing.
3. If the file exists but is empty or all proposals are resolved, delete it.

### Step 4: Merge Feature Spec into Latest

This is a **content merge**, not a file copy. For each subsystem the feature touches:

1. Identify which `spec/latest/` subsystems are affected:
   - Grep the feature's spec documents for all `spec://latest/` references — collect the list of referenced subsystems.
   - Check if the feature folder contains subsystem subfolders (e.g. `spec/features/NNNN-slug/orders/`) — these map directly to `spec/latest/` subsystems and contain only the changed sections.
   - Check if the feature introduces an entirely new subsystem (a subsystem subfolder with no corresponding folder in `spec/latest/`).
   - Present the list of affected subsystems to the user and confirm before proceeding.

2. For each affected subsystem in `spec/latest/`:

#### If the Subsystem Exists in Latest

- Read both the `latest/` version and the feature's subsystem subfolder version of each document.
- **Add new sections** that the feature introduces.
- **Update existing sections** that the feature modifies — replace the content with the feature's version.
- **Preserve sections** that the feature does not touch.
- Update `INDEX.md` if the subsystem's purpose or scope changed.

#### If the Subsystem is New

- Create the subsystem folder under `spec/latest/`.
- Copy the feature's subsystem subfolder documents (INDEX.md, REQUIREMENTS.md, ADR.md, DESIGN.md) into it.

#### Feature-Level Documents

- Feature-level documents (the root INDEX.md, REQUIREMENTS.md, ADR.md, DESIGN.md) contain new requirements, decisions, and designs that may need to go into an existing or new subsystem in `latest/`. Merge these based on the cross-links and subsystem references they contain.

3. Update `spec/GLOSSARY.md` with any new terms from the feature.

### Step 5: Update Code References

1. Grep for all `spec://features/NNNN-slug/` references in code files (not under `spec/`).
2. Replace them with the corresponding `spec://latest/` references, since the feature content is now in `latest/`.
3. Verify each replacement resolves correctly.

### Step 6: Commit

1. Stage all changes (updated `spec/latest/` files, updated code references).
2. Commit with message: `feat(spec): promote NNNN-slug specification to latest`

### Step 7: Create PR to Trunk

1. Determine the trunk branch (main or master).
2. Push the feature branch if not already pushed.
3. Create a PR from the feature branch to trunk using `gh pr create`:

```
## Summary

- Feature NNNN-slug specification promoted to latest/
- All task PRs merged into feature branch
- Code spec:// references updated from features/ to latest/

## Spec Changes

### New Subsystems
- [list any new subsystem folders created in latest/]

### Updated Subsystems
- [list subsystems modified in latest/ with brief description of changes]

### Glossary Updates
- [list new terms added]

## Tasks Completed

- ENG-123: [Title]
- ENG-124: [Title]
- ...
```

### Step 8: Report

Present a summary:

- Subsystems created or updated in `spec/latest/`
- Number of code references updated from `features/` to `latest/`
- Glossary terms added
- PR URL
- Remind the user to use `/address-review` if PR review comments need to be addressed or if there are merge conflicts with trunk
- Note that the feature folder in `spec/features/` is preserved as a historical record
- Feature and task branches are cleaned up by GitHub's auto-delete policy after PR merge
