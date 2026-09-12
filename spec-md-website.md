---
specmd: "0.4.2"
spec_version: "0.4.0"
status: draft
name: "SPEC.md Website"
last_updated: "2026-09-12"
---

# SPEC.md Website Specification

## Specification Contract

This document defines the required behavior of the public SPEC.md website (the GitHub Pages site published from `SPECmd-app/SPEC.md`).

Uppercase **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** use BCP 14 semantics.

Normative requirements define conformance. Examples, notes, and rationale are informative unless explicitly marked normative.

A conforming implementation MAY use different static-site tooling, hosting, or visual design unless explicitly constrained here.

If an unspecified choice cannot materially affect conformance, the implementer MAY choose an appropriate solution. If an omission could materially affect correctness of the published version information, it MUST be surfaced as `TBD` rather than guessed.

## 1. Overview and Scope

The website presents the SPEC.md format — its Core standard, Optional companion, worked examples, coding-agent integrations, and companion tooling — so both humans and LLMs can find and adopt it.

In scope: the homepage, the standard's own versioned documents, worked examples, agent-integration files, companion-tooling coverage, version links, machine-readable discovery files, and the changelog.

Out of scope: user accounts, comments, search, analytics, and any interactive tooling beyond in-page navigation. The site does not host the standard's own governance process, only its content and presentation.

## 2. Context and Definitions

**Current version** — the SPEC.md Core version presented as authoritative today.

**Previous version** — the Core version immediately superseded by the Current version.

**Worked example** — the Judo Club Website SPEC.md at a given Core version.

**Site source** — the `docs/` directory of `SPECmd-app/SPEC.md`, configured as this repository's GitHub Pages publishing source.

## 3. System Model

The published site is a static build with no backend and no database. The Site Source (`docs/`) holds the homepage (`index.html`), its logo asset, per-version Core/Optional documents (`docs/standard/`), and the machine-readable discovery files (`docs/robots.txt`, `docs/llms.txt`, `docs/_config.yml`, the homepage's JSON-LD block).

Worked examples (`examples/`) and agent-integration files (`integrations/`) live at the repository root, outside the Site Source. They are not built by Jekyll; the homepage and `llms.txt` reference them by absolute GitHub (blob) or raw-content URL rather than by a built relative page.

At any time, exactly one version is Current and at most one is Previous; both MUST correspond to files that exist in the repository. Every link on the homepage, in `llms.txt`, and in the agent-integration files MUST resolve within `SPECmd-app/SPEC.md` — none MUST point at a different repository for required content.

## 4. Requirements

- FUN-001: The homepage MUST link to the Current Core version and its Optional companion.
- FUN-002: The homepage's version section MUST show only the Current and Previous main versions, plus a link to the full changelog for earlier history.
- FUN-003: The site MUST expose `llms.txt` at the Site Source root, linking machine-readable (raw) sources for Core, Optional, the worked example, both agent integrations, and the changelog.
- FUN-004: The site MUST expose `robots.txt` at the Site Source root, allowing crawling and referencing this site's own sitemap.
- FUN-005: The homepage MUST include machine-readable structured data (JSON-LD) describing this site and the Current Core version.
- FUN-006: The homepage MUST include a section covering companion tooling (e.g. a CLI/MCP-server implementation of the format), linking to that tooling's own repository, and MUST NOT claim capabilities that tooling's own documentation does not claim for itself.
- UX-001: Where multiple version cards are displayed together (Current/Previous), their primary actions (buttons) MUST align to the same position regardless of differing description length.
- DATA-001: The site MUST NOT link to a version file, example, or integration file that does not exist in the repository.
- DATA-002: When a new Core version is released, the homepage, `llms.txt`, and the agent-integration files MUST be updated in the same change to reference it.
- INT-001: No page, file, or integration prompt in this repository MUST depend on content hosted in a different repository.

## 5. Interfaces and External Contracts

The site is built and served by GitHub Pages from the `main` branch, with `docs/` configured as the publishing source. Files with YAML front matter under `docs/` are converted to HTML by Jekyll; files without front matter (e.g. `robots.txt`, `llms.txt`, `index.html`) are served as-is. Content outside `docs/` (`examples/`, `integrations/`, `CHANGELOG.md`, `README.md`, `LICENSE`) is not processed by Jekyll and is reached only via GitHub's own blob/raw URLs.

This repository has no required external interfaces to other repositories or services.

## 6. Constraints and Non-Goals

No JavaScript-driven interactivity is required beyond in-page anchor navigation. No CMS, server-side backend, or visitor tracking is required.

Companion tooling coverage (FUN-006) is informational: the website MUST NOT reimplement or fork the tooling's behavior, and MUST NOT get out of sync with that tooling's own disclosed capabilities and known gaps.

## 7. Verification and Acceptance

Given a new Core version is released, when the release is complete, then the homepage's version section MUST show that version as Current, and the previously Current version MUST become Previous (DATA-001, DATA-002, FUN-002).

`llms.txt` and `robots.txt` MUST be reachable at the Site Source root and MUST reference only files that exist in the repository (FUN-003, FUN-004, INT-001).

Given the Current and Previous version cards are rendered together, when their description text differs in length, then their primary-action buttons MUST still align to the same vertical position (UX-001).

## 8. Notes and Rationale

An earlier draft of this specification (`spec_version` 0.3.0) described a standalone, private companion repository (`xonix999/spec-md-website`) that duplicated Core, Optional, worked examples, and integrations so it had no outside dependency. That repository was created but subsequent site development (the `docs/` publishing-source split, the logo change, the Tooling section, and this version's other changes) happened only on the public `SPECmd-app/SPEC.md` repository. `xonix999/spec-md-website` was not kept in sync and no longer reflects the live site; this revision re-anchors the specification on the site that is actually published, and treats the private repository as an unmaintained fork rather than the system of record.

Splitting worked examples and agent-integration files out of the Jekyll-built `docs/` directory keeps the Site Source small while letting GitHub itself serve those files' rendered/raw views, at the cost of losing a Pages-rendered HTML view for them (they render via GitHub's own blob/raw viewer instead).

Version links are updated manually alongside each Core release; this could be automated later, but that is not required for conformance.
