---
specmd: "0.4.2"
spec_version: "0.3.0"
status: draft
name: "SPEC.md Website"
last_updated: "2026-09-12"
---

# SPEC.md Website Specification

## Specification Contract

This document defines the required behavior of the public SPEC.md website (the GitHub Pages site published from this repository).

Uppercase **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** use BCP 14 semantics.

Normative requirements define conformance. Examples, notes, and rationale are informative unless explicitly marked normative.

A conforming implementation MAY use different static-site tooling, hosting, or visual design unless explicitly constrained here.

If an unspecified choice cannot materially affect conformance, the implementer MAY choose an appropriate solution. If an omission could materially affect correctness of the published version information, it MUST be surfaced as `TBD` rather than guessed.

## 1. Overview and Scope

The website presents the SPEC.md format — its Core standard, Optional companion, worked examples, and coding-agent integrations — so both humans and LLMs can find and adopt it.

This repository is standalone: it holds the Core standard, Optional companion, worked examples, agent integrations, and changelog itself, and MUST NOT depend on any other repository for its content to render or resolve correctly.

In scope: the homepage, the standard's own versioned documents, worked examples, agent-integration files, version links, machine-readable discovery files, and the changelog.

Out of scope: user accounts, comments, search, analytics, and any interactive tooling. The site does not host the standard's own governance process, only its content and presentation.

## 2. Context and Definitions

**Current version** — the SPEC.md Core version presented as authoritative today.

**Previous version** — the Core version immediately superseded by the Current version.

**Worked example** — the Judo Club Website SPEC.md at a given Core version.

## 3. System Model

This repository is a static build with no backend and no database. Its content consists of: one homepage (`index.html`); per-version documents (Core, Optional, worked example) under `standard/` and `examples/`; agent-integration files under `integrations/`; `CHANGELOG.md`; and root-level machine-readable files (`robots.txt`, `llms.txt`, the homepage's JSON-LD block).

At any time, exactly one version is Current and at most one is Previous; both MUST correspond to files that exist in this repository. Every link on the homepage, in `llms.txt`, and in the agent-integration files MUST resolve within this repository — none MUST point at another repository for required content.

## 4. Requirements

- FUN-001: The homepage MUST link to the Current Core version and its Optional companion.
- FUN-002: The homepage's version section MUST show only the Current and Previous main versions, plus a link to the full changelog for earlier history.
- FUN-003: The site MUST expose `llms.txt` at the root, linking machine-readable (raw) sources for Core, Optional, the worked example, both agent integrations, and the changelog, all within this repository.
- FUN-004: The site MUST expose `robots.txt` at the root, allowing crawling and referencing this site's own sitemap.
- FUN-005: The homepage MUST include machine-readable structured data (JSON-LD) describing this site and the Current Core version.
- DATA-001: The site MUST NOT link to a version file that does not exist in this repository.
- DATA-002: When a new Core version is released, the homepage, `llms.txt`, and the agent-integration files MUST be updated in the same change to reference it.
- INT-001: No page, file, or integration prompt in this repository MUST depend on content hosted in a different repository.

## 5. Interfaces and External Contracts

The site is built and served by GitHub Pages from the `main` branch root. Files with YAML front matter are converted to HTML; files without front matter (e.g. `robots.txt`, `llms.txt`) are served as-is.

This repository has no required external interfaces. It is self-contained.

## 6. Constraints and Non-Goals

No JavaScript-driven interactivity is required beyond in-page anchor navigation. No CMS, server-side backend, or visitor tracking is required.

This repository is private. If GitHub Pages cannot be served from a private repository under the current plan, the site is not publicly reachable until either the repository is made public or the account's plan supports private Pages; that constraint does not change any requirement above.

## 7. Verification and Acceptance

Given a new Core version is released, when the release is complete, then the homepage's version section MUST show that version as Current, and the previously Current version MUST become Previous (DATA-001, DATA-002, FUN-002).

`llms.txt` and `robots.txt` MUST be reachable at the site root and MUST reference only files that exist in this repository (FUN-003, FUN-004, INT-001).

## 8. Notes and Rationale

The site was split from the original `SPECmd-app/SPEC.md` repository into its own repository so it could have independent visibility (private) without depending on that repository staying public or unchanged. All Core, Optional, worked-example, integration, and changelog content was copied in rather than linked, so this repository has no outside dependency.

Version links are updated manually alongside each Core release; this could be automated later, but that is not required for conformance.
