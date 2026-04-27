---
name: spec-navigator
description: Use when navigating between code and specifications, resolving spec:// references, finding code related to a spec section, or researching requirements, decisions, and constraints before writing code.
---

# Spec Navigator

This skill teaches you how to navigate the `spec/` directory structure in this repository — resolving references from code to spec, finding code from spec, and doing design-time research.

For the canonical directory layout, naming conventions, reference format, status annotations, and validation rules, see `skills/shared/SPEC-STRUCTURE.md`.

---

## Mode 1: Code → Spec (Resolve a Reference)

Use when you encounter a `spec://` reference in code and need to understand what it points to.

### Steps

1. Parse the reference: extract the file path and section anchor.
   - Example: `spec://latest/orders/REQUIREMENTS.md#payment-flow` → file `spec/latest/orders/REQUIREMENTS.md`, section `#payment-flow`

2. Check feature layering (see SPEC-STRUCTURE.md). If a feature is active and the same file exists under the feature folder, read the feature version instead.

3. Read the file. Navigate to the specified section anchor.

4. If the file does not exist or the section anchor is not present:
   - **Report the broken reference explicitly.** Do not silently fall back or guess.
   - State: the file path, whether the file was found, and whether the section was found.
   - Treat this as a signal: either the spec is missing (needs to be written) or the reference in code is stale (needs to be updated).

5. While reading, note any cross-links (`spec://...` references within the document). Follow each cross-link **one level deep** — read the linked section but do not recursively follow further links within it.

6. If the section has a status annotation (e.g., `[status: tech-debt]`, `[status: known-bug]`), report the status alongside the content. This affects how the caller should interpret the section.

---

## Mode 2: Spec → Code (Find Referencing Code)

Use when you have a spec section and need to find all code and tests that reference it.

### Steps

1. Identify the full reference string, e.g. `spec://latest/orders/REQUIREMENTS.md#payment-flow`.

2. Search for **exact reference matches** across all non-spec files:
   - Grep for the full string: `spec://latest/orders/REQUIREMENTS.md#payment-flow`

3. Search for **anchor-only matches** (informal references):
   - Grep for the section anchor alone: `#payment-flow`
   - Review results to determine relevance.

4. Search for **bare file references** (no section anchor):
   - Grep for the file path: `spec://latest/orders/REQUIREMENTS.md`

5. Exclude files under `spec/` from all searches — you are looking for code, not spec cross-links.

6. **Group results into two categories:**

### Production Code

Files implementing the behavior described in the spec section.

### Tests

Files verifying the behavior described in the spec section.

Report both groups separately. If a group is empty, state that explicitly.

---

## Mode 3: Research (Design-Time Exploration)

Use when you are about to write or modify code and need to understand requirements, decisions, and constraints for a subsystem or feature.

### Steps

#### Step 1: Load the Glossary

Read `spec/GLOSSARY.md` in full. This grounds all domain terminology used in subsequent documents.

#### Step 2: Determine Active Feature

Run `git branch --show-current`. Identify the active feature folder if any (see Feature Layering in SPEC-STRUCTURE.md).

#### Step 3: Identify Relevant Subsystems

Use `Glob` to list all subsystem folders:

- `spec/latest/*/INDEX.md`
- If a feature is active: `spec/features/[feature-name]/*/INDEX.md`

Read each `INDEX.md` for the subsystems likely to be relevant. The INDEX contains purpose and scope — use it to decide whether to go deeper without loading the full documents.

#### Step 4: Read Documents in Order

For each relevant subsystem, read in this order:

1. `REQUIREMENTS.md` — what must be true (business and technical constraints)
2. `ADR.md` — what was decided and why (architecture decisions)
3. `DESIGN.md` — how it was implemented (frameworks, patterns, components)
4. `SCENARIOS.md` — how it is verified (Gherkin-style test scenarios)

Do not skip ahead to DESIGN.md before reading REQUIREMENTS.md. Requirements constrain decisions; decisions constrain implementation.

Note status annotations on sections — `[status: known-bug]` or `[status: tech-debt]` sections may need special handling during implementation.

#### Step 4b: Read Implementation Plan (If Available)

If the feature has `TASKS.md`, read it to understand the implementation breakdown — which tasks exist, their test scenarios, acceptance criteria, and which spec sections they implement. This is useful when you need to understand what has been planned or implemented for a feature.

#### Step 4c: Read Pending Spec Proposals (If Available)

If `SPEC-PROPOSAL.md` exists in the feature folder, read it to understand pending spec changes that were discovered during implementation but not yet approved. These proposals may affect your understanding of the current spec state — the spec may be evolving.

#### Step 5: Follow Cross-Links

While reading, collect all `spec://` cross-links encountered. After finishing each document, follow each cross-link **one level deep** — read the linked section in the referenced document. Do not follow links within those linked sections.

---

## Mode 4: Writing Spec References in Code

Use when adding or updating `spec://` references in code. See `SPEC-STRUCTURE.md` for the full reference format and placement rules.

### Rules

#### Find the Spec Section First

Before writing a reference, use Mode 3 (Research) to locate the exact spec section that governs the behavior you are implementing. Do not guess at paths or anchors.

#### Reference the Most Specific Anchor

Always reference the most specific section anchor available, not just the file.

Prefer:

```
spec://latest/orders/REQUIREMENTS.md#payment-flow
```

Over:

```
spec://latest/orders/REQUIREMENTS.md
```

#### Prefer `latest/` in Production Code

Reference `spec://latest/...` in production code. Only use `spec://features/...` in code that is explicitly part of that feature and has not yet been promoted to `latest/`.

#### Placement

Place the reference in a comment as close as possible to the code it governs — at the function, method, or type level, not at the file level.

Example (Go):

```go
// spec://latest/orders/REQUIREMENTS.md#payment-flow
func (o *Order) ProcessPayment(amount Money) error {
```

Example (TypeScript):

```typescript
// spec://latest/authn/REQUIREMENTS.md#token-expiry
export function validateToken(token: string): boolean {
```

#### Verify the Reference

After writing a reference, verify it resolves correctly using Mode 1 (Code → Spec). If the section does not exist, create the spec section before committing the code.
