---
name: spec-review
description: Review and refine feature specifications based on user concerns. Use when user wants to challenge, refine, or improve a feature spec. Can be run multiple times until all concerns are resolved.
---

# Spec Review

This skill reviews and refines feature specification documents based on user concerns. It combines grill-me style questioning with autonomous spec updates, using the topics criterion as a review checklist.

## Process

### Step 1: Load Feature Context and Criterion

1. Run `git branch --show-current` to determine the active branch.
2. Derive the feature folder:
   - Strip the `feat/` prefix from the branch name.
   - Extract the first path segment (everything before the second `/`) — this is the feature slug.
   - Feature folder = `spec/features/[slug]/`
   - Example: `feat/0023-payment-retry/eng-123-retry-endpoint` → `spec/features/0023-payment-retry/`
   - Example: `feat/0023-payment-retry` → `spec/features/0023-payment-retry/`
3. If not on a feature branch (no `feat/` prefix), ask the user which feature to review and switch to it.
4. Read all spec documents in the feature folder: INDEX.md, REQUIREMENTS.md, DESIGN.md, SCENARIOS.md, and ADR.md (if it exists).
5. Read `spec/GLOSSARY.md` for domain terminology.
6. Read any `spec/latest/` documents cross-linked from the feature spec.
7. Read `skills/shared/TOPICS-CRITERION.md` — the adequacy criteria checklist.
8. If the project has `spec/CRITERION.md`, read it and apply overrides.

### Step 2: Understand User Concerns

The user provides their concerns as arguments when invoking the skill (e.g. `/spec-review "I'm not sure the retry logic handles timeouts correctly"`).

Parse the concerns and categorize them:

- **Requirements concerns** — missing, incorrect, or unclear requirements; failure expectations not stated
- **Architecture concerns** — questionable decisions, missing alternatives considered
- **Design concerns** — implementation approach, error handling gaps, consistency issues
- **Scope concerns** — feature boundary issues, missing constraints
- **Scenario concerns** — missing test coverage, inadequate Gherkin scenarios, broken `Verifies:` links
- **Topic coverage concerns** — conditional topics that should be activated but aren't, or sections that don't meet adequacy criteria

### Step 3: Adequacy Audit

Before addressing user concerns, run a quick audit of the existing spec against the criterion:

1. For each **required** topic, check if the section exists and meets adequacy criteria.
2. For each **conditional** topic, check if keyword signals appear in the spec content suggesting it should be activated but wasn't.
3. Verify cross-references:
   - REQUIREMENTS.md references INDEX.md stakeholder sections
   - DESIGN.md references SCENARIOS.md for verification
   - SCENARIOS.md `Verifies:` links point to valid REQUIREMENTS.md anchors
4. Report any gaps found alongside the user's concerns.

### Step 4: Clarify Details (Grill-Me Style)

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time:

1. State which spec document and section the concern relates to.
2. Quote the current spec content that is in question.
3. Explain why this is ambiguous, potentially problematic, or fails an adequacy criterion.
4. Provide your recommended resolution.
5. Ask the user to confirm or provide their preferred resolution.

If a question can be answered by exploring the codebase, explore the codebase instead.

Continue until all concerns are fully resolved. Do not batch questions — ask one, wait for the answer, then ask the next.

### Step 5: Update Spec Documents

Once all questions for a concern are resolved, immediately update the relevant spec files:

1. Modify the specific sections that need changes.
2. Ensure all new items are addressable subsections (not bullet lists).
3. Update cross-links if the changes affect references to `spec/latest/` or other feature docs.
4. Update `spec/GLOSSARY.md` if new terms were introduced.
5. Ensure SCENARIOS.md `Verifies:` links align with updated REQUIREMENTS.md anchors.
6. If a new conditional topic was activated during review, add the full section meeting adequacy criteria.
7. If a non-obvious decision emerged, add or update ADR.md (create the file if it didn't exist).

### Step 6: Commit Changes

1. Stage all modified files.
2. Commit with message: `feat(spec): refine specification based on review`
3. Push to the feature branch.

### Step 7: Report

Present a summary:

- List of concerns addressed
- Adequacy gaps found and resolved (from the audit)
- Files modified and sections changed
- Any new conditional topics activated
- Any new ADRs added
- Any new cross-links or glossary terms added
- Suggest running `/spec-review` again if the user has additional concerns, or `/to-plan` if the spec is finalized
