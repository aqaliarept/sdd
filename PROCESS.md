# Spec-Driven Feature Lifecycle

This document describes the end-to-end process for developing features using the spec-driven framework. Each step corresponds to a skill that can be invoked in Claude Code.

## Scenarios

This framework supports two entry points:

1. **New feature development** — greenfield features on a codebase that already has `spec/latest/` established.
2. **Bootstrap** — retroactively creating `spec/latest/` from an existing codebase with no specs.

Bootstrap is typically a one-time activity per domain area. Once `spec/latest/` is established, all subsequent work follows the new feature flow.

---

## Scenario 1: Bootstrap — Establishing `spec/latest/`

Use when onboarding an existing codebase to spec-driven development. This creates the initial `spec/latest/` from existing code, tests, git history, and additional knowledge sources.

### Flow Overview

```
/bootstrap-spec → /spec-review → (spec/latest/ established) → use Scenario 2 for new features
                    ↻ repeat
```

### Phase B1: Bootstrap — `/bootstrap-spec`

**Input:** A domain direction (e.g., "user management") and optional knowledge sources (docs, URLs).
**Output:** `spec/latest/` subsystem documents and `spec://` references in code (all local, no git operations).

The agent explores the codebase following the domain direction, discovers related subsystems, and proposes a decomposition. The user confirms subsystem boundaries, then the agent processes each subsystem:

1. Deep-reads code, tests, and key git commits
2. Loads topic criterion (`skills/shared/TOPICS-CRITERION.md` + project overrides)
3. Writes draft spec documents (INDEX.md, REQUIREMENTS.md, DESIGN.md, SCENARIOS.md, conditional ADR.md)
4. Interviews the user to fill gaps and validate inferences — applies status annotations (`[status: verified]`, `[status: tech-debt]`, `[status: known-bug]`, `[status: deprecated]`, `[status: undetermined]`)
5. Updates `spec/GLOSSARY.md` with confirmed domain terms
6. Adds `spec://latest/` references to key code entry points
7. Validates all cross-references resolve correctly

No git operations are performed. The user handles branching, committing, and reviewing.

```
/bootstrap-spec user management
/bootstrap-spec payment processing
```

Multiple runs build up `spec/latest/` incrementally. Each run checks for existing subsystems and offers skip, overwrite, or merge (additive with `[status: conflict]` annotations).

### Phase B2: Review — `/spec-review` (repeatable)

After bootstrapping, use `/spec-review` to refine sections marked `[status: undetermined]` or `[status: conflict]`, and to address any adequacy gaps.

```
/spec-review "The auth subsystem is missing security considerations"
/spec-review "The retry logic status annotations need review"
```

Once `spec/latest/` is established and reviewed, all new work follows Scenario 2.

---

## Scenario 2: New Feature Development

Use for all new features once `spec/latest/` is established.

### Flow Overview

```
/grill-me → /to-spec → /spec-review → /to-plan → /to-linear → /implement → /address-review → /complete
                          ↻ repeat                               ↻ per task    ↻ per review
```

### Phase 1: Discovery — `/grill-me`

**Input:** A rough feature idea or problem statement.
**Output:** Shared understanding of requirements, constraints, and decisions.

The user invokes `/grill-me` with their feature idea. The agent interviews relentlessly, following a natural broad-to-narrow flow guided by the topics criterion (`skills/shared/TOPICS-CRITERION.md`). It tracks topic coverage internally, uses keyword signals to detect conditional topics organically, and validates all required topics against adequacy criteria before declaring done.

At the end of this phase, the conversation contains all the context needed to write the specification.

```
/grill-me I want to add payment retry logic with exponential backoff
```

---

### Phase 2: Specification — `/to-spec`

**Input:** Conversation context from `/grill-me`.
**Output:** Feature branch with populated spec documents.

The agent synthesizes the conversation into structured specification documents. It does NOT ask further questions.

What happens:

1. Loads topic criterion and evaluates topic activation (required + conditional signals)
2. Determines the next feature number (e.g. `0023`)
3. Creates branch `feat/0023-payment-retry`
4. Creates `spec/features/0023-payment-retry/` with:
   - `INDEX.md` — purpose, scope, user roles
   - `REQUIREMENTS.md` — business and technical requirements, constraints, conditional topics
   - `DESIGN.md` — architecture overview, conditional design topics
   - `SCENARIOS.md` — Gherkin-style test scenarios grouped by user flow
   - `ADR.md` — architecture decisions (conditional, only if non-obvious choices exist)
5. Validates generated content against adequacy criteria
6. Analyzes `spec/latest/` to detect required changes and adds cross-links
7. Updates `spec/GLOSSARY.md` with new terms
8. Commits and pushes

```
/to-spec
```

---

### Phase 3: Review — `/spec-review` (repeatable)

**Input:** User concerns about the spec.
**Output:** Updated spec documents.

The user reviews the generated spec and raises concerns. The agent runs an adequacy audit against the topics criterion, clarifies details (grill-me style), then autonomously updates the spec files. Run as many times as needed.

```
/spec-review "I'm not sure the retry logic handles timeouts correctly"
/spec-review "The backoff multiplier should be configurable"
```

---

### Phase 4: Planning — `/to-plan`

**Input:** Finalized feature spec.
**Output:** `TASKS.md` in the feature folder.

The agent reads the spec and breaks it into crosscutting vertical-slice tasks. Each task includes UI and Backend work, test scenarios, acceptance criteria, and references to spec sections.

The agent proposes the task breakdown, the user approves or adjusts, and then `TASKS.md` is written.

```
/to-plan
```

---

### Phase 5: Linear Integration — `/to-linear`

**Input:** `TASKS.md` from the feature folder.
**Output:** Linear issues created, `TASKS.md` updated with issue IDs.

The agent creates Linear issues for each task via MCP, renames task headings from `TASK-NNN` to the Linear issue ID (e.g. `ENG-123`), and adds Linear URLs. The Linear ID becomes the canonical task identifier used in branch naming for automatic linking.

```
/to-linear PROJECT-NAME
```

---

### Phase 6: Implementation — `/implement` (per task)

**Input:** A Linear issue ID.
**Output:** Task branch with implementation, tests, spec references, and PR.

For each task, the agent:

1. Creates a task branch: `feat/0023-payment-retry/eng-123-retry-endpoint`
2. Reads the task's spec references for full context
3. Implements using TDD (red-green-refactor)
4. Adds `spec://` references to code
5. Creates `SPEC-PROPOSAL.md` if spec changes are discovered
6. Commits, pushes, and creates a PR to the feature branch

```
/implement ENG-123
/implement ENG-124
/implement ENG-125
```

---

### Phase 7: Review Fixes — `/address-review` (repeatable)

**Input:** PR with review comments.
**Output:** Fixed code, resolved spec proposals, pushed updates.

After PR review, the agent pulls unresolved comments and addresses them:

- Fixes code issues raised by reviewers
- Applies approved spec proposals to the feature spec
- Reverts code for rejected proposals
- Cleans up `SPEC-PROPOSAL.md` when all entries are resolved

```
/address-review
```

This phase repeats until the PR is approved. The user merges the PR manually. Repeat for each task PR.

---

### Phase 8: Completion — `/complete`

**Input:** Feature branch with all tasks merged.
**Output:** PR from feature branch to trunk.

When all task PRs are merged into the feature branch, the agent:

1. Verifies all tasks are merged and no spec proposals are pending
2. Merges feature spec into `spec/latest/` (content merge, not file copy)
3. Updates `spec://features/...` references in code to `spec://latest/...`
4. Creates a PR from the feature branch to trunk (main/master)

The feature folder in `spec/features/` is preserved as a historical record. The user merges the PR manually. Feature and task branches are cleaned up by GitHub's auto-delete policy after merge.

```
/complete
```

Use `/address-review` to handle any review comments or merge conflicts on the final PR.

---

## Shared Resources

| File | Purpose |
|------|---------|
| `skills/shared/TOPICS-CRITERION.md` | Topic inventory with tiers, keyword signals, and adequacy criteria. Consumed by `/grill-me`, `/to-spec`, `/spec-review`, `/bootstrap-spec`. |
| `skills/shared/SPEC-STRUCTURE.md` | Canonical directory layout, naming conventions, `spec://` format, status annotations, validation rules. Consumed by all spec skills. |
| `spec/CRITERION.md` | Optional project-level topic overrides (add, elevate, skip). |
| `spec/GLOSSARY.md` | Domain terminology shared across all subsystems and features. |

## Spec Structure

See `skills/shared/SPEC-STRUCTURE.md` for the full canonical layout. Summary:

```
spec/
  GLOSSARY.md                          # all domain terms
  CRITERION.md                         # optional project-level topic overrides
  latest/                              # authoritative current state
    [subsystem]/
      INDEX.md                         # purpose + scope + user roles
      REQUIREMENTS.md                  # business and technical requirements
      ADR.md                           # architecture decision records (conditional)
      DESIGN.md                        # implementation design
      SCENARIOS.md                     # Gherkin-style test scenarios
  features/
    NNNN-[feature-name]/               # feature work (historical record)
      INDEX.md                         # feature-level purpose + scope
      REQUIREMENTS.md                  # feature-level new requirements
      ADR.md                           # feature-level decisions (conditional)
      DESIGN.md                        # feature-level design
      SCENARIOS.md                     # feature-level test scenarios
      TASKS.md                         # implementation tasks
      SPEC-PROPOSAL.md                 # temporary: implementation-discovered changes
      [subsystem]/                     # optional: changes to existing latest/ subsystems
        REQUIREMENTS.md                # only sections that change
        ADR.md
        DESIGN.md
        SCENARIOS.md
```

## Branch Convention

```
main (or master)                       # trunk
└── feat/0023-payment-retry            # feature branch
    ├── feat/0023-payment-retry/eng-123-retry-endpoint    # task branch
    ├── feat/0023-payment-retry/eng-124-retry-ui          # task branch
    └── feat/0023-payment-retry/eng-125-retry-tests       # task branch
```

## Key Principles

- **Spec is the source of truth.** Code references spec sections via `spec://` links. Requirements constrain decisions; decisions constrain implementation.
- **Feature layering.** An active feature's spec is a layer on top of `latest/`. When complete, it merges into `latest/`.
- **No bullet lists in specs.** Every addressable item is a markdown subsection with a heading anchor.
- **Cross-links.** Spec documents link to each other using `spec://` format. Code links to spec. This creates a navigable web of decisions and implementations.
- **Topics criterion.** All specs are governed by `skills/shared/TOPICS-CRITERION.md` which defines required, conditional, and optional topics with adequacy criteria.
- **Status annotations.** Retroactive specs use explicit status annotations (`[status: verified]`, `[status: tech-debt]`, etc.) on every section. Forward-flow specs do not use annotations.
- **TDD implementation.** All code is written test-first using the red-green-refactor cycle.
- **Spec proposals, not blockers.** If implementation reveals spec gaps, propose changes and proceed. Don't block on approval.
- **User merges PRs.** No skill ever merges a PR — merging is always a manual user action. Branches are cleaned up by GitHub's auto-delete policy.
