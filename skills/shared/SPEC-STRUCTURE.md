# Spec Directory Structure

This document defines the canonical spec directory layout, naming conventions, reference format, status annotations, and validation rules. It is the shared source of truth for all skills that interact with the `spec/` directory.

## Directory Layout

```
spec/
  GLOSSARY.md                          # all domain terms — read this first
  CRITERION.md                         # optional project-level topic overrides
  latest/                              # authoritative current state
    [subsystem]/
      INDEX.md                         # purpose + scope of this subsystem
      REQUIREMENTS.md                  # business and technical requirements
      ADR.md                           # architecture decision records (conditional)
      DESIGN.md                        # implementation: architecture, components, failure handling
      SCENARIOS.md                     # Gherkin-style test scenarios by user flow
  features/
    NNNN-[feature-name]/               # active or historical feature work
      INDEX.md                         # feature-level purpose + scope
      REQUIREMENTS.md                  # feature-level new requirements
      ADR.md                           # feature-level decisions (conditional)
      DESIGN.md                        # feature-level design
      SCENARIOS.md                     # feature-level test scenarios
      TASKS.md                         # crosscutting implementation tasks (created by /to-plan)
      SPEC-PROPOSAL.md                 # implementation-discovered spec changes (temporary)
      [subsystem]/                     # optional: changes to existing latest/ subsystems
        REQUIREMENTS.md                # only sections that change
        ADR.md
        DESIGN.md
        SCENARIOS.md
```

## Document Set

| Document | Responsibility | Required |
|----------|---------------|----------|
| INDEX.md | What, why, who, boundaries | Always |
| REQUIREMENTS.md | What must be true. Failure expectations embedded in requirements. | Always |
| DESIGN.md | How to build it. Error handling, consistency, recovery design. References SCENARIOS.md. | Always |
| SCENARIOS.md | How to verify it. Gherkin-style markdown grouped by user flow. `Verifies:` links to REQUIREMENTS.md. | Always |
| ADR.md | Non-obvious decisions with context, alternatives, consequences. | Conditional — only when non-obvious decisions exist |

**Minimum viable spec:** INDEX.md, REQUIREMENTS.md, DESIGN.md, SCENARIOS.md. ADR.md only when triggered.

## Naming Conventions

### Feature Folders

Feature folders use a numeric prefix and descriptive slug:
- Format: `NNNN-slug` (e.g., `0023-payment-retry-logic`)
- Numeric prefix is zero-padded to 4 digits
- Slug is lowercase, hyphen-separated

### Feature Branches

```
feat/NNNN-slug                                      # feature branch
feat/NNNN-slug/{issue-id}-short-description          # task branch
```

### Bootstrap Branches (suggested, not enforced by skill)

```
spec/bootstrap/[domain-direction]                    # e.g., spec/bootstrap/user-management
```

### Subsystem Folders

Subsystem names under `latest/` are lowercase, hyphen-separated, matching the domain name (e.g., `authentication`, `order-management`, `user-profiles`).

## Cross-Reference Format (`spec://`)

### URI Structure

```
spec://latest/[subsystem]/[DOCUMENT.md]#[anchor]
spec://features/NNNN-slug/[DOCUMENT.md]#[anchor]
spec://features/NNNN-slug/[subsystem]/[DOCUMENT.md]#[anchor]
```

The path after `spec://` is relative to the `spec/` directory. The `#anchor` is a markdown section heading converted to lowercase-hyphenated form.

### Resolution Rules

1. Parse the reference: extract file path and section anchor.
2. Check feature layering: if a feature is active, check the feature folder first.
3. Read the file and navigate to the anchor.
4. If the file or anchor does not exist, report a broken reference. Do not silently fall back.

### Cross-Reference Direction Rules

| From | To | Link |
|------|----|------|
| REQUIREMENTS.md | INDEX.md | Stakeholder references |
| DESIGN.md | SCENARIOS.md | Verification references |
| SCENARIOS.md | REQUIREMENTS.md | `Verifies:` links (never to DESIGN.md) |
| Any topic | Any related topic | `spec://` cross-links |
| Code | Spec | `spec://` comments at function/method/type level |

### Placement in Code

Place `spec://` references in comments at the function, method, or type level — not at the file level. Use the most specific anchor available.

```go
// spec://latest/orders/REQUIREMENTS.md#payment-flow
func (o *Order) ProcessPayment(amount Money) error {
```

### Rules

- Prefer `spec://latest/` in production code
- Use `spec://features/` only in code not yet promoted to `latest/`
- Always reference the most specific section anchor, not just the file
- Verify references resolve correctly after writing them
- Cross-links within spec documents follow one level deep only

## Feature Layering

When a feature branch is active, feature documents are a **layer on top of** `latest/`:

1. Determine active feature from branch name: strip `feat/` prefix, extract first path segment.
2. Check both locations:
   - `spec/features/NNNN-slug/[document]` — feature-level docs
   - `spec/features/NNNN-slug/[subsystem]/[document]` — subsystem overrides
3. Feature docs take precedence over `latest/` for the sections they define.
4. If no feature is active, use `latest/` only.

## Spec Authoring Conventions

### Every Item Must Be a Subsection

Do not use bullet lists for enumerable items. Every item that may be referenced from code must be a markdown heading with an addressable anchor.

### Section Anchors

Markdown anchors are derived from headings: lowercase, spaces replaced with hyphens, punctuation removed. Verify anchors match what is referenced in code.

### Cross-Links in Spec Documents

Use full `spec://` format:

```markdown
See [Token Expiry](spec://latest/authn/REQUIREMENTS.md#token-expiry) for constraints.
```

## Status Annotations

Status annotations mark the review state of individual spec sections. Each section heading gets an explicit annotation.

```markdown
### Payment Capture [status: verified]

### Retry Logic [status: tech-debt]
```

### Vocabulary

| Annotation | Meaning | Resolution Path |
|------------|---------|-----------------|
| `[status: verified]` | Behavior is correct and intentional | None — this is the spec |
| `[status: tech-debt]` | Works but implementation is suboptimal | Refactor via feature spec |
| `[status: known-bug]` | Behavior is incorrect, fix needed | Bug fix via feature spec |
| `[status: deprecated]` | Still in code but planned for removal | Don't build on top of it |
| `[status: undetermined]` | Couldn't confirm with user, needs review | Review in next `/spec-review` session |
| `[status: conflict]` | Code analysis disagrees with existing spec | Resolve via `/spec-review` |

### Usage Rules

- Every section gets an explicit status annotation (absence is not allowed)
- `[status: verified]` is the target state — sections confirmed as correct
- `[status: conflict]` is used during merge operations when code analysis disagrees with existing spec content
- Annotations are removed when the status is resolved (bug fixed, debt paid, deprecation completed)
- Status annotations are primarily produced by `/bootstrap-spec` and consumed by `/spec-review`
- Forward-flow specs (`/to-spec`) do not use status annotations — all content is new and assumed correct

## Validation Rules

A valid `spec/` structure must satisfy:

1. **Root:** `spec/` directory exists at repository root
2. **Glossary:** `spec/GLOSSARY.md` exists
3. **Subsystem completeness:** each `spec/latest/[subsystem]/` contains at minimum INDEX.md, REQUIREMENTS.md, DESIGN.md, SCENARIOS.md
4. **Feature completeness:** each `spec/features/NNNN-slug/` contains at minimum INDEX.md, REQUIREMENTS.md, DESIGN.md, SCENARIOS.md
5. **Feature naming:** feature folders match pattern `NNNN-[a-z0-9-]+`
6. **Reference integrity:** all `spec://` references in spec documents and code resolve to existing files and anchors
7. **Scenario links:** every `Verifies:` link in SCENARIOS.md points to a valid REQUIREMENTS.md anchor
8. **Stakeholder links:** REQUIREMENTS.md references to INDEX.md stakeholder sections resolve correctly

### Pre-Flight Check

Before any skill writes to `spec/`, validate:

1. If `spec/` exists, check it matches the expected structure (contains `latest/` or `features/` or `GLOSSARY.md`)
2. If `spec/` contains content that looks foreign (e.g., RSpec test files, OpenAPI specs), warn the user and ask to confirm before proceeding
3. If `spec/` does not exist, it will be created

## Temporary Files

### TASKS.md

Lives inside a feature folder. Contains crosscutting implementation tasks with descriptions, spec references, test scenarios, and acceptance criteria. After `/to-linear`, each task includes a Linear issue ID and URL. See `/to-plan` for the full template.

### SPEC-PROPOSAL.md

Temporary file created by `/implement` when implementation reveals spec changes needed. Each entry includes the originating task, proposed change, and rationale. Proposals are approved or rejected via PR review. File is deleted when all proposals are resolved. See `/implement` and `/address-review` for the lifecycle.
