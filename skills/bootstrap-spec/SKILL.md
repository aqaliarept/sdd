---
name: bootstrap-spec
description: Retroactively generate spec/latest/ from existing codebases without specs. Analyzes code, tests, git history, and additional sources to produce structured specification documents. Use when onboarding a project to spec-driven development.
---

# Bootstrap Spec

This skill retroactively generates `spec/latest/` documentation from an existing codebase that has no specs. It analyzes code, tests, git history, and user-provided sources to produce structured specification documents with status annotations indicating review confidence.

**No git operations are performed.** All files are written locally. The user handles branching, committing, and reviewing.

## Invocation

```
/bootstrap-spec [domain direction]
/bootstrap-spec user management
/bootstrap-spec payment processing
```

The domain direction guides exploration — the skill discovers related subsystems by domain relevance.

## Process

### Step 1: Validate Spec Structure

Read `skills/shared/SPEC-STRUCTURE.md` for the canonical directory layout and validation rules.

Run the pre-flight check:

1. If `spec/` exists, verify it matches the expected structure (`latest/`, `features/`, `GLOSSARY.md`).
2. If `spec/` contains foreign content (RSpec tests, OpenAPI files, etc.), warn the user and ask to confirm before proceeding.
3. If `spec/` does not exist, inform the user it will be created.

### Step 2: Gather Additional Sources

Ask the user if they have additional knowledge sources beyond the codebase:

- Local documentation files (markdown, text, PDF)
- URLs (wikis, API docs, design documents, external references)
- README files or other in-repo documentation

These sources supplement code analysis. Accept sources both now and during the subsystem interviews (Step 6e) when specific topics arise.

### Step 3: Explore Codebase

Starting from the user's domain direction, explore the codebase:

1. **Scan code structure** — packages, modules, directory layout. Identify domain boundaries.
2. **Follow imports and dependencies** — trace how code in the domain direction connects to other parts of the codebase.
3. **Analyze tests** — test files reveal intended behavior and domain terminology.
4. **Analyze key git commits** — filter for high-value commits:
   - `git log --diff-filter=A` — file creation (foundational decisions)
   - Large diffs — refactors and major changes
   - PR merge commits — often contain "why" context in descriptions
5. **Read additional sources** — fetch URLs and read local files provided by the user.
6. **Discover related subsystems** — based on domain relevance, not just code dependencies.

### Step 4: Propose Subsystem Decomposition

Present the proposed decomposition to the user:

```
Proposed subsystem decomposition for "[domain direction]":

1. [Subsystem Name]           [NEW - no existing spec]
   - path/to/package/
   - path/to/related/code.go
   - Related tests: path/to/*_test.go

2. [Subsystem Name]           [EXISTS - spec/latest/subsystem/]
   - path/to/package/

3. [Subsystem Name]           [NEW - no existing spec]
   - path/to/package/

Excluded (referenced but out of scope):
   - path/to/other/ (reason for exclusion)
```

For each subsystem, indicate whether it's `[NEW]` or `[EXISTS]`.

**User actions:**
- Confirm, rename, merge, or split subsystems
- Adjust code path mappings
- For `[EXISTS]` subsystems, choose: **skip**, **overwrite**, or **merge**

**Merge strategy for `[EXISTS]` subsystems:**
- Additive: new sections are added to existing spec documents
- Conflicts: sections where code analysis disagrees with existing spec content are annotated `[status: conflict]`
- Existing verified content is preserved

### Step 5: Create Spec Directory (if needed)

If `spec/` or `spec/latest/` doesn't exist, create the directory structure. If `spec/GLOSSARY.md` doesn't exist, create it with an empty template.

### Step 6: Process Each Subsystem (Serial)

Process subsystems one at a time. Knowledge from earlier subsystems informs later ones.

#### Step 6a: Deep-Read Code

For the current subsystem's code paths:

1. Read all source files — types, functions, methods, interfaces
2. Read all test files — test names, assertions, setup patterns
3. Read key git commits touching these paths (file creation, large changes, PR merges)
4. Read any additional sources the user provided that are relevant to this subsystem

#### Step 6b: Load Topic Criterion

1. Read `skills/shared/TOPICS-CRITERION.md`
2. If `spec/CRITERION.md` exists, apply project overrides (add, elevate, skip)

#### Step 6c: Evaluate Topic Activation

Scan the code analysis results for keyword signals defined in the criterion:

- **Required topics** — always include
- **Conditional topics** — include if keyword signals detected in code, tests, comments, or additional sources
- **Optional topics** — include only if strong evidence exists

#### Step 6d: Write Draft Spec Documents

Create `spec/latest/[subsystem]/` with draft documents:

- **INDEX.md** — purpose and scope inferred from package structure, comments, README files
- **REQUIREMENTS.md** — requirements reverse-engineered from behavior, tests, and additional sources
- **DESIGN.md** — architecture and design described from the actual implementation, with activated conditional design topics
- **SCENARIOS.md** — derived from existing tests first, converted to Gherkin-style markdown grouped by user flow. Gaps in test coverage marked `[status: undetermined]`
- **ADR.md** (conditional) — decisions reconstructed from git history, code comments, and additional sources

Apply initial status annotations:
- Sections derived from tests with clear assertions: `[status: verified]` (pending user confirmation)
- Sections inferred from code without test coverage: `[status: undetermined]`
- All sections get explicit annotations

#### Step 6e: Interview User

Conduct a targeted interview — not a full `/grill-me` session. The code analysis provides the context; the interview fills gaps and validates inferences.

**Present findings per document**, then ask targeted questions:

1. **Validate inferences** — "I found that admin users can delete any account based on the handler code. Is this intentional, or is it a missing authorization check?"
2. **Fill gaps** — "I couldn't determine the expected behavior when the payment gateway times out. What should happen?"
3. **Classify status** — based on user answers, apply annotations:
   - "Is this intentional?" → `[status: verified]` or `[status: known-bug]`
   - "Is this implementation acceptable?" → `[status: verified]` or `[status: tech-debt]`
   - "Is this still in use?" → `[status: verified]` or `[status: deprecated]`
   - User doesn't know → `[status: undetermined]`

**Flag suspicious patterns** for user review:
- Empty catch/error blocks (swallowed errors)
- Commented-out code
- TODO/FIXME/HACK comments
- Functions with no test coverage
- Inconsistent naming patterns within the subsystem
- Dead code (unreachable branches)
- Hardcoded values that look like they should be configurable

**Accept additional sources** — the user may provide URLs or documents relevant to specific questions ("oh, there's a wiki page about that decision").

#### Step 6f: Update GLOSSARY.md

Extract domain-specific terms from the subsystem's code:
- Type names, constants, and enums that represent domain concepts
- Domain-specific language in comments and documentation

Present candidates to the user: "I found these domain terms — which are worth defining?"

Add confirmed terms to `spec/GLOSSARY.md`. Skip generic programming terms (`Handler`, `Service`, `Repository`).

#### Step 6g: Validate Against Adequacy Criteria

Check each generated section against its adequacy criteria from `TOPICS-CRITERION.md`. Flag sections that don't meet criteria — include in the final report.

### Step 7: Add spec:// References to Code

After all subsystems are processed, annotate the codebase with `spec://latest/` references.

**Scope: key entry points only:**
- Public API functions
- HTTP/gRPC handlers
- Service methods (public interface)
- Type definitions (domain types, interfaces)

**Skip:**
- Internal/private helpers
- Utility functions
- Test files

Follow the placement rules from `SPEC-STRUCTURE.md`:
- Place in comments at function/method/type level
- Use the most specific anchor available
- Use `spec://latest/` format

### Step 8: Cross-Subsystem Validation

After all subsystems are processed and code is annotated:

1. Scan all `spec://latest/` references across all generated spec documents
2. Scan all `spec://` comments in annotated code
3. Verify every reference resolves to an existing file and anchor
4. Report dangling references

### Step 9: Report

Present a summary:

- **Subsystems bootstrapped** — list with `[NEW]`, `[OVERWRITE]`, or `[MERGE]` indicator
- **Files created/modified** — full list for the user to `git add`
- **Status annotation summary** — count per status per subsystem:
  ```
  authentication: 12 verified, 2 tech-debt, 1 known-bug, 0 deprecated, 3 undetermined
  authorization:  8 verified, 0 tech-debt, 0 known-bug, 1 deprecated, 1 undetermined
  ```
- **Adequacy gaps** — sections that don't meet criterion quality bar
- **Dangling references** — any `spec://` references that don't resolve
- **Sources consumed** — list of additional sources used (URLs, files, git commits) for traceability
- **New glossary terms** — terms added to GLOSSARY.md
- **Suggested commit messages:**
  ```
  spec(latest): bootstrap [subsystem] specification
  spec(code): add spec references for [subsystem]
  ```
- **Next steps** — suggest `/spec-review` to refine sections with `[status: undetermined]` or `[status: conflict]`
