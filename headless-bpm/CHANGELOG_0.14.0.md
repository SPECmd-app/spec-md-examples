<!-- SPDX-License-Identifier: Apache-2.0 -->

# Headless BPM 0.14.0 Changelog

Date: 2026-09-20

## Summary

0.14.0 adds three fleet-scale administrative capabilities plus standardized CLI discoverability and system health/version discovery while preserving the 0.13.x runtime, BPMN, authorization, and recovery semantics.

## Added

- Runtime Process Instance diagram rendering: SVG derived from the exact bound Process Version/BPMN DI plus authorized runtime state/history.
- Fleet Operations Summary: authorized aggregate counts for Process Instance states, actionable/claimed/overdue tasks, open Incidents, and Operational Findings, with observation time and no-leak filtering.
- Preview-first Bulk Operations for incident retry, Process Instance cancel/pause/resume, task reassignment, and stale-claim release.
- Frozen preview target sets, per-target revalidation, partial-result reporting, bulk idempotency, and preservation of normal per-target audit/history semantics.
- Equivalent CLI, REST, and MCP surfaces for all three capabilities.
- CLI usability baseline: `-h/--help` at every level, `help`, documented public flags, human-first plus JSON output, stdout/stderr separation, conventional exit behavior, typo suggestions, and safe preview/confirmation patterns for high-impact operations.
- Runtime system discovery through CLI `version`/`status`, REST system info/status, and MCP `system_info`/`system_status`.
- Separate minimal REST liveness/readiness health checks suitable for infrastructure probing.

## Deferred / unchanged

The 0.13.1 Open Issues remain deferred, including Process Instance version migration, a dedicated global timers/jobs/subscriptions/claim-lease resource, standard metrics/export, process analytics/SLA reporting, and first-class alerting.

## Compatibility

This release is additive. Existing Process definitions, Process Instances, Entry Points, task operations, administrative interventions, Activity Streams, and BPMN interchange semantics remain unchanged.
