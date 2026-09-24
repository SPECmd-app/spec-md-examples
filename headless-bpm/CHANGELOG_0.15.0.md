# Headless BPM 0.15.0 Changelog

Date: 2026-09-23

## Summary

0.15.0 is a specification-only refinement of 0.14.0, reviewed with the specification owner: additive normative
clarifications, one new optional capability, two SHOULD→MUST promotions, and a terminology alignment with BPMN 2.0. No
previously defined MUST behavior was weakened or removed, no requirement ID was reused, and Process Version/BPMN
execution semantics are untouched.

## Normative clarifications

(previously ambiguous or unstated; behavior now made explicit)

- **WF-005** defines what a "deterministic evaluation rule" is for an Exclusive Gateway with multiple conditional
  Sequence Flows: an explicit declared evaluation order, or conditions that are provably mutually exclusive by
  construction.
- **HUM-001** clarifies that a User Task whose assignment resolves only at runtime (for example, by Organization Unit
  membership) does not block publication merely because it currently resolves to nobody.
- **WF-001** now states explicitly that a Process Version MAY contain only Service Task Flow Nodes and no User Task at
  all — fully autonomous, agent-only orchestration is an explicitly conforming shape. New **ACC-137**.
- **ORG-008** defines how a Task Instance's Organization Unit scope is derived, for the purpose of organization-scoped
  authorization narrowing (from the task's own assignment-rule selectors, or its assignee's membership).
- **PAC-005** clarifies that an associated ACTIVE Process Access Profile is itself a source of allowed operations (not
  only a filter requiring a separate Actor Grant for the same action), that it narrows access to its own list even when
  a broader grant would otherwise allow more, and that an empty or interface-less profile denies that interface rather
  than falling back to unrestricted access.
- **LOOP-010** defines how the causal-chain identifier and hop count are conveyed through the message-publication
  contract: explicit fields, or derivation from a declared source Process Instance.

## New optional capability

- **ORG-011** (new requirement ID): a deployment MAY optionally associate a Process with one or more Organization Units
  for classification and discovery, independent of BPMN Lane/Pool metadata and granting no assignment eligibility,
  authorization, or platform permission by itself (same non-authority guarantee as ORG-010/INV-013). Not implemented by
  the reference implementation in this repository; see `IMPLEMENTATION.md`. New **ACC-139** proves a deployment remains
  conforming without it.

## Promoted from SHOULD to MUST

- **TLIST-011 / OPS-020**: when Organization Units are enabled, administrative task queries and the fleet Operations
  Summary MUST support filtering by Organization Unit (matched against each task's ORG-008-derived scope), so an Admin
  can inspect every task and Process Instance touching one organization across all Processes. New **ACC-138**.

## Terminology alignment with BPMN 2.0 (no behavior change)

`Inbound Event` and `Event Subscription` were renamed to **Inbound Message** and **Message Subscription** throughout
`SPEC.md`, `ERD.md`, and the reference implementation, and the `event.publish` operation (REST `POST /v1/events` and
`/v1/actor/events`; MCP `event_publish`/`actor_event_publish`; CLI `event publish`) was renamed to
**`message.publish`** (`POST /v1/messages`/`/v1/actor/messages`; `message_publish`/`actor_message_publish`; CLI
`message publish`). This removes a naming collision with the BPMN Event Flow Node category (Start Event, End Event,
Intermediate Catch Event) and with the unrelated Execution Event/Audit Event history-log terms; EVT-001 through EVT-008
correlation semantics are unchanged, and INV-008 was retitled "Single message consumption" to match. ACC-029 was
reworded ("Message ping-pong guard") to match, with no change to its Given/When/Then. This is a pre-launch rename — no
deployment or external integrator existed yet, so it carries no migration burden.

## Compatibility

Requirement IDs referenced above (WF-001, WF-005, HUM-001, ORG-008, PAC-005, LOOP-010, TLIST-011, OPS-020, INV-008,
EVT-002 through EVT-008) are unchanged; **ORG-011** is a new ID appended after ORG-010 per the stable-ID convention.
Acceptance scenario numbering extends through **ACC-139** (previously 136). `TRACE.md`, `ERD.md`, and this
implementation were updated accordingly.

## Reference implementation (informative, 2026-09-23)

- `IMPLEMENTATION_VERSION` and `SPEC_VERSION` (`src/core/config.ts`), `package.json`, and the REST/MCP/CLI
  `system.info`/`version` surfaces now report **0.15.0**.
- `TRACE.md` regenerated: 328 requirements, 34 invariants, and 139 acceptance scenarios, all with automated evidence.
- Full test suite verified green (487/487) on both PGlite and a real PostgreSQL server via `TEST_DATABASE_URL`, with
  clean `tsc --noEmit` and `eslint`.
