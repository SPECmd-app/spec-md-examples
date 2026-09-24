# Headless BPM 0.18.0 Changelog

Date: 2026-09-24

## Summary

0.18.0 closes a real operability gap found while diagnosing a live deployment: a failed readiness/liveness check had
nowhere to record *why* it failed, since the public response is intentionally minimal and non-leaking by design
(SYS-003). No previously defined MUST behavior was weakened or removed, and no requirement ID was reused.

## Added

- **SYS-003 extended — operator-visible failure detail.** When a readiness or liveness check fails, the implementation
  MUST record the underlying failure detail (for example the specific connectivity or dependency error) to a
  server-side, operator-only channel such as application logs, distinct from the check's own minimal public response.
  The public interface's non-leaking guarantee is unchanged; this only requires that the detail exist *somewhere*
  server-side instead of nowhere at all. **ACC-134**'s Given/When/Then extended to cover this.

## Compatibility

This is an additive obligation on an existing requirement (SYS-003) and its existing acceptance scenario (ACC-134); no
new requirement or acceptance ID was introduced, and the public response shape/behavior is unchanged. `TRACE.md` was
updated accordingly (no new rows — the same SYS-003/ACC-134 rows now carry updated text).

## Reference implementation (informative, 2026-09-24)

- Found and fixed during a real deployment: a Vercel `DATABASE_URL` missing its `postgresql://` scheme prefix produced
  an opaque `503 not_ready` with no way to see why, since `readiness()`'s catch block previously discarded the error
  entirely rather than logging it anywhere.
- `src/app/ops/system.ts`'s `readiness()` now writes `[hbpm] readiness check failed: <message>` to `process.stderr`
  (picked up as function logs by any host, including `vercel logs`) before returning the same minimal `not_ready`
  response as before. The exact log line format is an implementation choice, not mandated by SPEC.md.
- `test/ops.test.ts`'s ACC-134 test extended to simulate a genuine thrown persistence error (not just the
  `dbHealthy() -> false` short-circuit it already covered) and assert the failure is both recorded to `stderr` and
  still absent from the public response body.
- `IMPLEMENTATION_VERSION`/`SPEC_VERSION`, `package.json`, and the REST/MCP/CLI `system.info`/`version` surfaces now
  report **0.18.0**.
- Full test suite verified green (490/490) on both PGlite and a real PostgreSQL server via `TEST_DATABASE_URL`, with
  clean `tsc --noEmit` and `eslint`.
