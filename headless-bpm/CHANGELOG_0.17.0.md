# Headless BPM 0.17.0 Changelog

Date: 2026-09-24

## Summary

0.17.0 makes MCP's existing stateless HTTP transport a normative requirement, so a conforming deployment is guaranteed
to be usable from a horizontally-scaled or serverless/hosted environment (such as Vercel), not only from a local
stdio-connected MCP client. No previously defined MUST behavior was weakened or removed, and no requirement ID was
reused.

## Added

- **API-034 — MCP hosted mode.** The MCP interface MUST be exposable over a network-reachable, stateless
  request/response transport (for example HTTP), in addition to any local/stdio transport a deployment also offers.
  A hosted MCP request MUST authenticate using the same credential mechanisms as REST, and MUST be authorized
  identically to the equivalent REST/CLI operation (API-004). All state required to service one MCP tool call MUST be
  durable (database-backed) rather than held in server-process memory, so consecutive calls from the same caller MAY be
  served by different server instances or serverless invocations without loss of correctness. New **ACC-141**.

## Compatibility

This is a promotion of already-existing, unchanged behavior to normative status, not a behavior change: the reference
implementation already served MCP over a stateless `POST /mcp` HTTP JSON-RPC endpoint, authenticated identically to
REST, and already ran as a stateless Vercel Function. **API-034** is a new requirement ID appended after API-033;
**ACC-141** extends acceptance scenario numbering (previously 140). `TRACE.md` was updated accordingly.

## Reference implementation (informative, 2026-09-24)

- No runtime code changed. `src/http/server.ts`'s `POST /mcp` route and `api/index.ts`'s stateless Vercel Function
  wrapper already satisfied API-034 exactly as written.
- New test: **ACC-141** ([test/ops.test.ts](test/ops.test.ts)) drives the real `POST /mcp` HTTP route (not the in-process
  operation dispatcher used by other MCP tests) with a JSON-RPC `tools/call` request, over two independent calls with no
  shared session, and confirms the unauthenticated case is rejected the same way REST is (`401`, JSON-RPC error
  `-32001`). This closes a coverage gap: no prior test exercised the actual hosted HTTP transport end to end.
- `IMPLEMENTATION_VERSION`/`SPEC_VERSION`, `package.json`, and the REST/MCP/CLI `system.info`/`version` surfaces now
  report **0.17.0**.
- `TRACE.md` regenerated: 330 requirements, 34 invariants, 141 acceptance scenarios, all with automated evidence.
- Full test suite verified green (490/490) on both PGlite and a real PostgreSQL server via `TEST_DATABASE_URL`, with
  clean `tsc --noEmit` and `eslint`.
