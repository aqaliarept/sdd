---
name: address-review
description: Pull unresolved PR review comments and address them. Handles code fixes, spec proposal approvals/rejections, and pushes updates. Can be run multiple times until all comments are resolved.
---

# Address Review

This skill pulls unresolved review comments from the current branch's PR and addresses them. It handles code changes, spec proposal approvals, and pushes fixes.

## Process

### Step 1: Identify the PR and Feature Folder

1. Run `git branch --show-current` to get the current branch.
2. Derive the feature folder:
   - Strip the `feat/` prefix from the branch name.
   - Extract the first path segment (everything before the second `/`) — this is the feature slug.
   - Feature folder = `spec/features/[slug]/`
   - Example: `feat/0023-payment-retry/eng-123-retry-endpoint` → `spec/features/0023-payment-retry/`
   - Example: `feat/0023-payment-retry` → `spec/features/0023-payment-retry/`
3. Use `gh pr view --json number,url,headRefName` to find the open PR for this branch.
4. If no PR is found, fail with a clear message.

### Step 2: Pull Unresolved Comments

1. Use `gh api repos/{owner}/{repo}/pulls/{number}/comments` to get all review comments.
2. Filter for unresolved/pending comments (not yet resolved by the author).
3. Group comments by category:

#### Code Comments
General code review feedback — bugs, style, logic issues, suggestions.

#### Spec Proposal Comments
Comments that reference SPEC-PROPOSAL.md entries. Look for patterns like:
- "SPEC-PROPOSAL: approved" or "SP-001: approved" — reviewer approves the proposal
- "SPEC-PROPOSAL: rejected" or "SP-001: rejected" — reviewer rejects the proposal
- Other comments on SPEC-PROPOSAL.md lines

### Step 3: Address Code Comments

For each unresolved code comment:

1. Read the referenced file and line range.
2. Understand the reviewer's concern.
3. Fix the code to address the concern.
4. If the fix requires test changes, update tests and ensure they pass.
5. If the fix changes behavior covered by `spec://` references, verify the references are still correct.

### Step 4: Handle Spec Proposals

Read `SPEC-PROPOSAL.md` from the feature folder (if it exists).

#### For Approved Proposals

1. Read the proposal entry (e.g. SP-001).
2. Apply the proposed change to the actual spec file (e.g. update `REQUIREMENTS.md`, `ADR.md`, or `DESIGN.md` in the feature folder).
3. Remove the approved entry from `SPEC-PROPOSAL.md`.

#### For Rejected Proposals

1. Read the rejection reason from the review comment.
2. Revert any code that was written assuming the proposed spec change.
3. Adjust the implementation to work within the existing spec.
4. Remove the rejected entry from `SPEC-PROPOSAL.md`.
5. Update tests if the behavior changed.

#### Cleanup

If all entries in `SPEC-PROPOSAL.md` are resolved (approved or rejected), delete the file entirely.

### Step 5: Run Tests

Run the full test suite (or at minimum, tests related to the changes) to ensure everything passes after the fixes.

### Step 6: Commit and Push

1. Stage all modified files.
2. Commit with message: `fix(NNNN-slug): address review comments for {ISSUE-ID}`
3. Push to the current branch.

### Step 7: Report

Present a summary:

- Number of comments addressed
- Code changes made
- Spec proposals approved/rejected and files updated
- Whether SPEC-PROPOSAL.md was cleaned up
- Test results
- Suggest running `/address-review` again after the next review cycle
- If all comments are resolved and PR is approved, remind the user to merge the PR manually
