# Specification Topics Criterion

This document defines the complete inventory of topics that a feature specification may contain, their classification, quality criteria, and activation signals. It is the shared source of truth consumed by `/grill-me`, `/to-spec`, and `/spec-review`.

## How This Document Is Used

| Skill | Usage |
|-------|-------|
| `/grill-me` | Topic inventory + keyword signals to guide interview naturally from broad to narrow. Adequacy criteria as completion gate. |
| `/to-spec` | Topic inventory + keyword signals as safety net. Adequacy criteria to validate generated content. |
| `/spec-review` | Adequacy criteria as review checklist. |

## Tier Definitions

| Tier | Meaning | `/grill-me` behavior | `/to-spec` behavior |
|------|---------|----------------------|---------------------|
| **required** | Always included | Always probe this topic | Always generate; flag as incomplete if insufficient context |
| **conditional** | Included when keyword signals detected | Probe when signals appear organically in conversation | Generate if context exists; flag gap if signals present but context insufficient |
| **optional** | Only if explicitly relevant | Don't probe unless user raises it | Generate only if rich context exists; skip silently otherwise |

## Document Set and Structure

See `skills/shared/SPEC-STRUCTURE.md` for the canonical directory layout, naming conventions, `spec://` reference format, status annotations, and validation rules.

| Document | Responsibility |
|----------|---------------|
| INDEX.md | What, why, who, boundaries |
| REQUIREMENTS.md | What must be true. References INDEX.md for stakeholders. Failure expectations embedded in requirements. |
| ADR.md | Non-obvious decisions with context, alternatives, consequences. Conditional — only when triggered. |
| DESIGN.md | How to build it. References SCENARIOS.md for verification. Failure handling, consistency, recovery design. |
| SCENARIOS.md | How to verify it. Gherkin-style markdown grouped by user flow. `Verifies:` links to REQUIREMENTS.md. |

**Minimum viable spec:** INDEX.md, REQUIREMENTS.md, DESIGN.md, SCENARIOS.md. ADR.md only when triggered.

## Cross-Reference Rules

- REQUIREMENTS.md references INDEX.md stakeholder sections
- DESIGN.md references SCENARIOS.md for verification
- SCENARIOS.md `Verifies:` links point to REQUIREMENTS.md (never to DESIGN.md)
- ADR.md activates as a byproduct of non-obvious decisions in any topic
- All references use `spec://` format (see SPEC-STRUCTURE.md for full format and resolution rules)

## Project Overrides

Projects may define `spec/CRITERION.md` to layer on top of this document.

**Allowed overrides:**
- Add new project-specific topics
- Elevate tier (conditional -> required)
- Skip conditional or optional topics via `skip: true`

**Not allowed:**
- Demote required topics (required -> conditional)
- Skip required topics

**Format:**
```markdown
# Project Criterion

## Overrides
- topic-slug: required        # elevate from conditional
- topic-slug: skip            # exclude for this project

## Additional Topics
### topic-slug: Topic Name
(same structure as plugin topics)
```

---

# Topics

---

## purpose-and-scope

- **Tier:** required
- **Document:** INDEX.md
- **Purpose:** Establish what this feature does and what it explicitly does not do.
- **Adequacy criteria:**
  - [ ] Purpose is a single paragraph that explains both what the feature does and why it exists
  - [ ] In Scope lists concrete capabilities being delivered
  - [ ] Out of Scope lists related capabilities explicitly excluded
  - [ ] A reader can determine whether a given requirement belongs to this feature or not
- **Inadequate:** "This feature improves the payment system."
- **Adequate:** "This feature adds automatic retry logic for failed payment captures. It exists because 3.2% of captures fail due to transient gateway timeouts, resulting in lost revenue. Retries are limited to idempotent capture operations; refunds and void operations are out of scope."

---

## user-roles

- **Tier:** required
- **Document:** INDEX.md
- **Purpose:** Define who interacts with this feature and in what capacity. Grounds business requirements and scenario actors.
- **Adequacy criteria:**
  - [ ] Each role is a named subsection with a markdown anchor
  - [ ] Each role describes what the actor does in relation to this feature
  - [ ] Roles are distinct — no two roles describe the same actor
  - [ ] Every role referenced in REQUIREMENTS.md or SCENARIOS.md is defined here
- **Inadequate:** "Users and admins."
- **Adequate:** "### Customer\nEnd user who initiates a purchase. Interacts with checkout flow and receives order confirmations.\n### Operations Agent\nInternal user who monitors failed payments and can manually trigger retries from the admin dashboard."

---

## business-requirements

- **Tier:** required
- **Document:** REQUIREMENTS.md
- **Purpose:** Define what the feature must achieve from a business perspective. Each requirement is testable and tied to a user role.
- **Adequacy criteria:**
  - [ ] Each requirement is a separate subsection with a markdown anchor
  - [ ] Each requirement specifies which user role it serves (references INDEX.md)
  - [ ] Each requirement is verifiable — can be mapped to at least one test scenario
  - [ ] No ambiguous qualifiers ("fast", "easy", "intuitive") without measurable definition
  - [ ] Requirements describe WHAT, not HOW
  - [ ] Requirements with failure risk state the failure expectation
- **Inadequate:** "The system should handle payments reliably."
- **Adequate:** "### Payment Capture Retry\nWhen a payment capture fails due to a transient gateway error (see [spec://INDEX.md#customer]), the system must attempt to collect the payment again. No payment data may be lost between attempts."

---

## technical-requirements

- **Tier:** required
- **Document:** REQUIREMENTS.md
- **Purpose:** Define technical constraints and capabilities the implementation must satisfy.
- **Adequacy criteria:**
  - [ ] Each requirement is a separate subsection with a markdown anchor
  - [ ] Each requirement is verifiable through testing or inspection
  - [ ] Requirements specify concrete thresholds where applicable (not "should be fast")
  - [ ] Requirements describe WHAT, not HOW
  - [ ] Requirements with failure risk state the failure expectation
- **Inadequate:** "The retry mechanism should be efficient."
- **Adequate:** "### Retry Idempotency\nRetried capture requests must be idempotent. Sending the same capture request multiple times must not result in duplicate charges. The system must use the gateway's idempotency key mechanism."

---

## constraints

- **Tier:** required
- **Document:** REQUIREMENTS.md
- **Purpose:** Define external limitations the feature must comply with. Constraints are not chosen by us — they are imposed by environment, regulation, or dependencies.
- **Adequacy criteria:**
  - [ ] Each constraint is a separate subsection with a markdown anchor
  - [ ] Each constraint identifies its source (regulatory, infrastructure, dependency, organizational)
  - [ ] Constraints are distinguishable from requirements — they describe limitations, not desired behavior
- **Inadequate:** "Must use PostgreSQL."
- **Adequate:** "### Payment Gateway Rate Limit\nSource: Stripe API terms of service.\nThe gateway enforces a maximum of 100 retry requests per second per merchant account. The retry mechanism must respect this limit."

---

## architecture-overview

- **Tier:** required
- **Document:** DESIGN.md
- **Purpose:** Describe how the feature integrates with the existing system. Provides the structural context for all other design topics.
- **Adequacy criteria:**
  - [ ] Identifies which existing subsystems are touched or extended
  - [ ] Describes the data flow through the system for the primary use case
  - [ ] References relevant `spec://latest/` subsystem documents
  - [ ] A developer unfamiliar with the codebase can understand where this feature lives
- **Inadequate:** "This feature adds retry logic to the payment service."
- **Adequate:** "The retry mechanism is added as a new component within the `orders` subsystem. When `PaymentGateway.Capture()` returns a transient error, the `RetryScheduler` enqueues the operation into the existing `async_jobs` table. The `JobProcessor` (shared infrastructure) picks up retry jobs and invokes `PaymentGateway.Capture()` with the original idempotency key. On success, the order transitions to `captured` state via the existing `OrderStateMachine`."

---

## test-scenarios

- **Tier:** required
- **Document:** SCENARIOS.md
- **Purpose:** Define verifiable behaviors using Gherkin-style markdown grouped by user flow. Each scenario links to requirements it verifies.
- **Adequacy criteria:**
  - [ ] Scenarios use `Given/When/Then` keywords as mandatory structure
  - [ ] Scenarios use domain language, not implementation details (no "Given the database has row...")
  - [ ] Each scenario has a `Verifies:` link to one or more REQUIREMENTS.md sections
  - [ ] Scenarios are grouped into suites by user flow / behavior, not by individual requirement
  - [ ] `Scenario Outline` with `Examples` tables used for parameterized cases
  - [ ] Happy path and failure paths are both covered
  - [ ] Prose context between Gherkin blocks is permitted for clarity
- **Inadequate:**
```
Scenario: Test retry
  Given a payment
  When it fails
  Then retry it
```
- **Adequate:**
```
## Suite: Payment Capture Recovery

Verifies:
- [spec://features/NNNN-slug/REQUIREMENTS.md#payment-capture-retry]
- [spec://features/NNNN-slug/REQUIREMENTS.md#retry-idempotency]

Captures that fail due to transient gateway errors are retried
automatically while preserving idempotency.

  Scenario: Successful retry after transient failure
    Given a customer has completed checkout for order "ORD-100"
    And the payment gateway returns a transient timeout error on first capture attempt
    When the retry scheduler processes the failed capture
    Then the payment is captured successfully using the original idempotency key
    And the order transitions to "captured" state

  Scenario Outline: Retry respects maximum attempt limit
    Given a customer has completed checkout
    And the payment gateway returns transient errors on <attempts> consecutive attempts
    When all retry attempts are exhausted
    Then the order transitions to "capture_failed" state
    And an operations agent is notified

    Examples:
      | attempts |
      | 3        |
```

---

## non-functional-requirements

- **Tier:** conditional
- **Document:** REQUIREMENTS.md
- **Keyword signals:** performance, latency, throughput, scalability, SLA, response time, load, capacity, availability, uptime, p99, p95
- **Purpose:** Define measurable performance, scalability, and reliability targets.
- **Adequacy criteria:**
  - [ ] Each NFR has a measurable target with units (e.g., "p99 latency < 200ms", not "should be fast")
  - [ ] Each NFR specifies the conditions under which the target applies (e.g., "at 1000 concurrent users")
  - [ ] NFRs are verifiable through load testing or monitoring
- **Inadequate:** "The retry system should be fast and scalable."
- **Adequate:** "### Retry Scheduling Latency\nFailed captures must be enqueued for retry within 500ms of the failure response (p99). At peak load (200 failed captures/minute), retry scheduling must not degrade order processing latency."

---

## security-considerations

- **Tier:** conditional
- **Document:** REQUIREMENTS.md
- **Keyword signals:** authentication, authorization, user data, PII, passwords, tokens, API keys, payments, encryption, trust boundary, external API, OAuth, RBAC, permissions, credentials, secrets, GDPR, compliance
- **Purpose:** Identify security threats, data sensitivity, and protection requirements.
- **Adequacy criteria:**
  - [ ] Identifies what sensitive data the feature handles or accesses
  - [ ] Identifies trust boundaries the feature crosses
  - [ ] Each security requirement is a separate subsection with a markdown anchor
  - [ ] Threat and mitigation are stated as pairs, not just threats
- **Inadequate:** "The feature should be secure."
- **Adequate:** "### Payment Token Exposure\nRetry jobs store a reference to the original payment intent ID, not the raw card token. The idempotency key is generated server-side and never exposed to the client. Retry job payloads are encrypted at rest using the existing `async_jobs` encryption configuration."

---

## data-consistency-and-integrity

- **Tier:** conditional
- **Document:** DESIGN.md
- **Keyword signals:** transaction, eventual consistency, distributed, concurrent, race condition, duplicate, idempotent, atomic, rollback, two-phase, saga, lock, conflict
- **Purpose:** Describe how the design maintains data correctness during concurrent, partial, or distributed operations.
- **Adequacy criteria:**
  - [ ] Identifies operations that modify shared state
  - [ ] Describes consistency model chosen (strong, eventual, etc.) with rationale
  - [ ] Addresses what happens during partial failure mid-operation
  - [ ] Identifies race conditions or concurrent access patterns and their resolution
- **Inadequate:** "We use transactions to keep data consistent."
- **Adequate:** "The retry scheduler and the manual retry trigger (operations agent) can race on the same failed capture. The design uses an optimistic lock on the `payment_attempts` row: both paths read the current `attempt_version`, and the capture call includes it. The gateway's idempotency key prevents duplicate charges even if both paths fire simultaneously. The first successful response wins; the loser detects the version mismatch and no-ops."

---

## disaster-recovery

- **Tier:** conditional
- **Document:** DESIGN.md
- **Keyword signals:** backup, restore, failover, data loss, outage, recovery, RTO, RPO, replication, resilience, catastrophic, corruption
- **Purpose:** Describe how the system recovers from catastrophic failures without data loss.
- **Adequacy criteria:**
  - [ ] Identifies what stateful components are at risk
  - [ ] States recovery point objective (RPO) and recovery time objective (RTO) where applicable
  - [ ] Describes recovery procedure for each identified failure scenario
  - [ ] Addresses whether existing infrastructure recovery (database backups, etc.) is sufficient or feature-specific recovery is needed
- **Inadequate:** "We rely on database backups."
- **Adequate:** "Retry jobs are stored in the `async_jobs` table which is covered by the existing PostgreSQL WAL replication (RPO: < 1 second). If the job processor crashes mid-retry, the job remains in `processing` state. The existing stale-job reaper reclaims jobs in `processing` for longer than 5 minutes and re-enqueues them. No feature-specific disaster recovery is needed beyond the existing infrastructure."

---

## error-handling-and-failure-modes

- **Tier:** conditional
- **Document:** DESIGN.md
- **Keyword signals:** error, failure, exception, timeout, retry, fallback, circuit breaker, dead letter, graceful degradation, crash, unavailable, partial failure
- **Purpose:** Define concrete failure scenarios the feature may encounter and how the design responds to each.
- **Adequacy criteria:**
  - [ ] Each failure mode is a separate subsection
  - [ ] Each failure mode describes: trigger condition, system behavior, recovery or escalation path
  - [ ] Failure modes trace back to failure expectations stated in REQUIREMENTS.md
  - [ ] Distinguishes between transient and permanent failures where applicable
- **Inadequate:** "Errors are logged and retried."
- **Adequate:** "### Gateway Timeout\nTrigger: `PaymentGateway.Capture()` returns HTTP 504 or connection timeout.\nBehavior: The capture is marked as `retry_pending` and enqueued with exponential backoff (1s, 4s, 16s).\nEscalation: After 3 failed attempts, the order transitions to `capture_failed` and an alert is sent to the operations channel.\n\n### Gateway Permanent Rejection\nTrigger: Gateway returns HTTP 402 (card declined) or 422 (invalid request).\nBehavior: No retry. Order transitions immediately to `capture_failed`. Customer is notified via email to update payment method."

---

## migration-and-rollout-strategy

- **Tier:** conditional
- **Document:** DESIGN.md
- **Keyword signals:** migration, rollout, backwards compatible, feature flag, schema change, data migration, blue-green, canary, rollback, breaking change, deprecation, versioning
- **Purpose:** Describe how to deploy the feature without disrupting existing functionality.
- **Adequacy criteria:**
  - [ ] Identifies what existing data, schemas, or APIs change
  - [ ] Describes backwards compatibility approach or breaking change mitigation
  - [ ] Specifies rollback procedure if deployment fails
  - [ ] Addresses data migration for existing records if applicable
- **Inadequate:** "We'll add a feature flag."
- **Adequate:** "The `async_jobs` table gains a new `job_type = 'payment_retry'` value. No schema migration needed — `job_type` is a text column. Existing job types are unaffected. The retry scheduler is deployed behind the `ENABLE_PAYMENT_RETRY` feature flag. Rollback: disable the flag. Pending retry jobs will be reaped by the stale-job reaper after 5 minutes and discarded (order stays in `capture_failed`)."

---

## observability

- **Tier:** conditional
- **Document:** DESIGN.md
- **Keyword signals:** logging, metrics, monitoring, alerting, tracing, dashboard, SLI, SLO, health check, audit trail, telemetry
- **Purpose:** Define what instrumentation the feature requires for production visibility.
- **Adequacy criteria:**
  - [ ] Identifies key metrics to track (counters, gauges, histograms)
  - [ ] Identifies log events with severity levels
  - [ ] Identifies alerting conditions and thresholds
  - [ ] Leverages existing observability infrastructure where possible
- **Inadequate:** "We'll add logging."
- **Adequate:** "Metrics: `payment_retry_total` (counter, labels: outcome=[success|exhausted|permanent_failure]), `payment_retry_delay_seconds` (histogram). Logs: Each retry attempt logged at INFO with order_id, attempt_number, gateway_response_code. Alert: `payment_retry_exhausted_rate > 5% over 15 minutes` triggers PagerDuty via existing alerting pipeline."

---

## api-contracts-and-interfaces

- **Tier:** conditional
- **Document:** DESIGN.md
- **Keyword signals:** API, endpoint, request, response, schema, payload, event, webhook, gRPC, REST, GraphQL, contract, interface, message, queue, topic, publish, subscribe
- **Purpose:** Define the shape of APIs, events, or shared interfaces the feature exposes or consumes.
- **Adequacy criteria:**
  - [ ] Each interface is a separate subsection
  - [ ] Request/response or event schemas are defined with field names and types
  - [ ] Error responses are documented
  - [ ] Versioning strategy is stated if the interface is public
- **Inadequate:** "The retry endpoint accepts a payment ID."
- **Adequate:** "### POST /admin/payments/{payment_id}/retry\nRequest: empty body. `payment_id` is the UUID from the orders subsystem.\nResponse 202: `{ \"retry_job_id\": \"uuid\", \"scheduled_at\": \"ISO8601\" }`\nResponse 404: Payment not found.\nResponse 409: Payment is not in `capture_failed` state.\nResponse 429: Retry already scheduled for this payment."

---

## dependencies-and-integration-points

- **Tier:** conditional
- **Document:** DESIGN.md
- **Keyword signals:** external service, third-party, SDK, library, shared, upstream, downstream, integration, dependency, vendor, package
- **Purpose:** Identify external systems and shared components the feature relies on, and their impact on testing and reliability.
- **Adequacy criteria:**
  - [ ] Each dependency is listed with its role in the feature
  - [ ] Availability and failure characteristics of each dependency are noted
  - [ ] Impact on testing strategy is described (what needs mocking at system boundaries)
  - [ ] Version constraints or compatibility requirements are stated where applicable
- **Inadequate:** "Uses Stripe SDK."
- **Adequate:** "### Stripe Payment Intents API\nRole: Executes payment capture and provides idempotency key support.\nAvailability: 99.99% SLA. Transient failures (timeouts, 5xx) expected at ~0.1% of requests.\nTesting: Mock at the `PaymentGateway` interface boundary. Use recorded gateway responses for retry scenario tests.\nVersion: Stripe API version `2024-04-10` pinned in SDK configuration."

---

## user-facing-behavior

- **Tier:** conditional
- **Document:** DESIGN.md
- **Keyword signals:** UI, UX, screen, page, form, button, notification, email, SMS, dashboard, CLI, display, user interaction, workflow, flow
- **Purpose:** Describe user-visible interactions, flows, and feedback mechanisms.
- **Adequacy criteria:**
  - [ ] Identifies user-facing touchpoints (screens, notifications, emails, CLI output)
  - [ ] Describes the user flow for the primary use case
  - [ ] Specifies feedback the user receives at each step (success, error, loading states)
  - [ ] Edge cases in user interaction are addressed
- **Inadequate:** "The admin can retry payments from the dashboard."
- **Adequate:** "The operations dashboard's payment detail page gains a 'Retry Capture' button, visible only when payment status is `capture_failed`. Clicking it shows a confirmation dialog: 'Retry payment capture for order ORD-XXX? The customer will be charged $XX.XX.' On confirm, the button is replaced with 'Retry Scheduled' (disabled) showing the `scheduled_at` timestamp. If the retry succeeds, the page auto-refreshes to show `captured` status. If the retry is exhausted, a banner shows 'All retry attempts failed. Contact the customer to update their payment method.'"

---

## architecture-decision-records

- **Tier:** conditional
- **Document:** ADR.md
- **Keyword signals:** alternative, trade-off, option, considered, debated, chose, decision, approach, versus, vs, pros, cons, why not
- **Purpose:** Capture non-obvious technical decisions with context, alternatives considered, and consequences. Activates as a byproduct of any topic surfacing a meaningful choice.
- **Adequacy criteria:**
  - [ ] Each ADR has a numbered title (ADR-001, ADR-002, ...)
  - [ ] Context describes the forces and constraints that drove the decision
  - [ ] At least two alternatives were considered with trade-offs stated
  - [ ] Decision states what was chosen and why
  - [ ] Consequences state what becomes easier and harder
- **Inadequate:** "We decided to use a queue for retries."
- **Adequate:** "### ADR-001: Retry Mechanism — Job Queue vs. In-Process Timer\n\n#### Context\nFailed payment captures need retry logic. The system already has an `async_jobs` infrastructure for background processing.\n\n#### Alternatives\n1. **In-process timer** — Goroutine with `time.After`. Simpler, no infrastructure dependency. But retries are lost on process restart and cannot be monitored externally.\n2. **Job queue (async_jobs)** — Leverages existing infrastructure. Retries survive restarts, are visible in the admin dashboard, and reuse the stale-job reaper.\n\n#### Decision\nJob queue. The existing infrastructure eliminates custom code and provides durability and visibility for free.\n\n#### Consequences\nEasier: monitoring, debugging, manual intervention. Harder: local development requires the job processor running."

---

## success-criteria-and-metrics

- **Tier:** optional
- **Document:** INDEX.md
- **Keyword signals:** success, KPI, metric, measure, goal, target, OKR, impact, outcome, adoption
- **Purpose:** Define how we know this feature achieved its intended impact after deployment.
- **Adequacy criteria:**
  - [ ] At least one measurable outcome is defined
  - [ ] Each metric has a target value and measurement method
  - [ ] Metrics measure business impact, not just technical output
- **Inadequate:** "We'll track if it works."
- **Adequate:** "Success: Reduce lost revenue from failed captures by 80% within 30 days of launch. Measured by comparing `capture_failed` terminal rate (currently 3.2%) against post-launch rate. Target: < 0.7%."
