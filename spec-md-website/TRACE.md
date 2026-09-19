---
specmd_trace: "0.4.0"
traces_file: "SPEC.md"
traces_spec: "0.5.0"
status: draft
name: "SPEC.md Website Traceability"
last_updated: "2026-09-19"
---

# SPEC.md Website — TRACE.md

## Purpose

This informative companion maps the requirements of `SPEC.md` version `0.5.0` to the parts of the site that implement them and to verification evidence.

The normative specification remains authoritative. This document does not add, remove, or reinterpret required behavior.

Implementation paths refer to the `SPECmd-app/SPEC.md` repository, the site's system of record. `TBD` identifies missing trace evidence, not an unresolved product requirement.

## Trace Model

```text
Requirement → Design area → Implementation reference → Verification evidence
```

Verification methods use:

- **T** — Test
- **A** — Analysis
- **I** — Inspection
- **D** — Demonstration

## Requirements Traceability Matrix

| Requirements | Design area | Method | Implementation | Evidence |
|---|---|---:|---|---|
| FUN-001, FUN-002 | Homepage version section | I | `docs/index.html` (#versions) | TBD |
| FUN-003 | Machine-readable index | I | `docs/llms.txt` | TBD |
| FUN-004 | Crawler policy | I | `docs/robots.txt` | TBD |
| FUN-005 | Structured data | I | `docs/index.html` (JSON-LD block) | TBD |
| FUN-006, DATA-001 | Tooling section, link integrity | I | `docs/index.html` (#tooling) | TBD |
| UX-001 | Version card layout | D | `docs/index.html` (card CSS) | TBD |
| DATA-002 | Release procedure | I | `CHANGELOG.md`, `docs/index.html`, `docs/llms.txt`, `integrations/` | TBD |
| INT-001 | Repository self-containment | I | whole repository | TBD |
| ANL-001 | Consent-gated GA4 loader | D | `docs/index.html` (head script), `docs/_includes/head-custom.html` | Live-site check 2026-09-19: tag requested only after Accept (homepage). Standard-page check: TBD |
| PRIV-001 | Consent gate | D | same as ANL-001 | Live-site check 2026-09-19: 0 `googletagmanager.com` requests before consent; 0 after Decline |
| PRIV-002 | Consent banner | D, I | same as ANL-001 | Live-site check 2026-09-19 (homepage): banner shown on first visit. Banner on Jekyll-built pages: source inspected (`curl` of `standard/0.4.3.html` contains the banner), not yet exercised in a browser |
| PRIV-003 | Choice persistence | D | same as ANL-001 | Live-site check 2026-09-19: `granted` and `denied` stored. Loading without banner on a later visit after Accept: TBD |
| PRIV-004 | "Cookie settings" control | D | same as ANL-001 | Live-site check 2026-09-19: control reopens the banner; Decline after Accept reloads with no GA |
| PRIV-005 | Client-side storage only | I | same as ANL-001 (`localStorage` in try/catch) | Source inspection only. Storage-unavailable case: TBD |

## Acceptance Coverage (SPEC.md §7)

| Acceptance scenario | Covered by | Status |
|---|---|---|
| First visit: banner shown, no analytics request | PRIV-001, PRIV-002 | Verified on homepage, 2026-09-19 |
| Accept: script requested, choice stored | ANL-001, PRIV-003 | Verified on homepage, 2026-09-19 |
| Decline: no script, choice stored | PRIV-003 | Verified on homepage, 2026-09-19 |
| Accept then Decline via "Cookie settings" | PRIV-004 | Verified on homepage, 2026-09-19 |
| Jekyll-built page shows same banner | ANL-001, PRIV-002 | Source-inspected only; browser check TBD |
| New Core release updates Current/Previous | DATA-001, DATA-002, FUN-002 | TBD |
| `llms.txt` / `robots.txt` reachable and reference existing files | FUN-003, FUN-004, INT-001 | TBD |
| Version cards' buttons align | UX-001 | TBD |

## Known Gaps

- No automated tests exist; the consent behavior was checked by hand in a browser against the live site.
- Whether GA4 actually receives hits (Realtime report) has not been checked. That is a property of the Google account, not of the site.
- The private mirror `xonix999/spec-md-website` carries the same code but cannot be served under the current plan, so it has no live evidence.
