# Headless BPM 0.16.0 Changelog

Date: 2026-09-24

## Summary

0.16.0 adds a second converging Parallel Gateway synchronization mode — a "first branch wins, cancel the rest"
discriminator — plus an Open Issue documenting FEEL as a candidate future Sequence Flow condition language. No
previously defined MUST behavior was weakened or removed, and no requirement ID was reused.

## Added

- **WF-007 / WF-016 — `FIRST_ACTIVE_BRANCH` Parallel Gateway convergence.** A converging Parallel Gateway can now declare
  `FIRST_ACTIVE_BRANCH` (alongside the existing `ALL_ACTIVE_BRANCHES`) via `<hbpm:gateway synchronization="..."/>`. Under
  this mode the join completes as soon as the first branch created for its synchronization scope arrives, and every
  other still-non-terminal branch is cancelled through the same EXEC-015 cascade used for whole-instance cancellation
  (active Flow Tokens, non-terminal Flow Node Instances, Task Instances, Task Attempts, and recursively invoked child
  Process Instances), scoped to just those branches. Cancelling a losing branch does not raise an Incident and is
  recorded in execution history as an ordinary outcome. Cancellation is best-effort: a durable side effect a losing
  branch already committed before cancellation takes effect remains committed, and cancellation only occurs at the next
  safe point (LOOP-012) rather than corrupting in-flight state. New **ACC-140**.
- Publish-time validation rejects `FIRST_ACTIVE_BRANCH` declared on a Parallel Gateway that is not a converging join
  (fewer than two incoming Sequence Flows), or on any node type other than a Parallel Gateway.

## Open Issues

- **New #15**: whether FEEL (as used by DMN/Zeebe) is adopted as an additional, explicitly-tagged Sequence Flow
  condition language alongside Open Issue #1's canonical expression language. Documents that a general-purpose Complex
  Gateway remains out of scope regardless; an AI-driven or otherwise complex routing decision is expected to be computed
  by a Service Task and exposed as a plain output variable, with the Gateway only ever evaluating a deterministic
  condition over that variable.

## Compatibility

All existing requirement and acceptance IDs are unchanged; **WF-016** is a new ID appended after WF-015, and **ACC-140**
extends acceptance scenario numbering (previously 139). A converging Parallel Gateway with no declared synchronization
mode continues to behave exactly as `ALL_ACTIVE_BRANCHES` did before this release. `TRACE.md` was updated accordingly.

## Reference implementation (informative, 2026-09-24)

- Implemented in `src/bpmn/{model,xml,authoring,compile}.ts` (parsing, BPMN XML round-trip, and publish-time
  validation of the new `synchronization` attribute) and `src/engine/{runtime,tasks}.ts` (join-satisfaction and the
  scoped branch-cancellation cascade).
- Known scope limit (documented in `IMPLEMENTATION.md`): branch cancellation is single-level — a losing branch that
  itself contains a further nested Parallel Gateway fork is not walked past its own immediate node to cancel that
  nested fork's own siblings, which remain dangling but harmless since the outer join has already advanced the process.
- `IMPLEMENTATION_VERSION`/`SPEC_VERSION`, `package.json`, and the REST/MCP/CLI `system.info`/`version` surfaces now
  report **0.16.0**.
- `TRACE.md` regenerated: 329 requirements, 34 invariants, 140 acceptance scenarios, all with automated evidence.
- Full test suite verified green (489/489) on both PGlite and a real PostgreSQL server via `TEST_DATABASE_URL`, with
  clean `tsc --noEmit` and `eslint`.
