---
specmd: "0.4.3"
spec_version: "0.6.1"
status: draft
name: "SPEC.md Website"
last_updated: "2026-09-20"
---

<!-- SPDX-License-Identifier: Apache-2.0 -->

# SPEC.md Website Specification

## Specification Contract

This document defines the required behavior of the public SPEC.md website (the GitHub Pages site published from `SPECmd-app/SPEC.md`).

Uppercase **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** use BCP 14 semantics.

Normative requirements define conformance. Examples, notes, and rationale are informative unless explicitly marked normative.

A conforming implementation MAY use different static-site tooling, hosting, or visual design unless explicitly constrained here.

If an unspecified choice cannot materially affect conformance, the implementer MAY choose an appropriate solution. If an omission could materially affect correctness of the published version information, it MUST be surfaced as `TBD` rather than guessed.

## 1. Overview and Scope

The website presents the SPEC.md format — its Core standard, Optional companion, worked examples, coding-agent integrations, and companion tooling — so both humans and LLMs can find and adopt it.

In scope: the homepage, the standard's own versioned documents, worked examples, agent-integration files, companion-tooling coverage, a separate Portables page for real-project specs, version links, machine-readable discovery files, and the changelog, and consent-gated visitor analytics.

Out of scope: user accounts, comments, search, and any interactive tooling beyond in-page navigation and the analytics-consent control. The site does not host the standard's own governance process, only its content and presentation.

## 2. Context and Definitions

**Current version** — the SPEC.md Core version presented as authoritative today.

**Previous version** — the Core version immediately superseded by the Current version.

**Worked example** — the Judo Club Website SPEC.md at a given Core version. It is the canonical reference example of the format.

**Portables** — the collection of complete, implementable SPEC.md files presented on the Portables page: the Judo Club first, then the independent real-project files (with a paired TRACE.md where one exists) maintained in the separate `SPECmd-app/spec-md-examples` repository. They are not tied to the standard's release cycle.

**Site source** — the `docs/` directory of `SPECmd-app/SPEC.md`, configured as this repository's GitHub Pages publishing source.

## 3. System Model

The published site is a static build with no backend and no database. The Site Source (`docs/`) holds the homepage (`index.html`), its logo asset, per-version Core/Optional documents (`docs/standard/`), and the machine-readable discovery files (`docs/robots.txt`, `docs/llms.txt`, `docs/_config.yml`, the homepage's JSON-LD block), plus the analytics include (`docs/_includes/head-custom.html`) that Jekyll injects into every built page.

Worked examples (`examples/`) and agent-integration files (`integrations/`) live at the repository root, outside the Site Source. They are not built by Jekyll; the homepage and `llms.txt` reference them by absolute GitHub (blob) or raw-content URL rather than by a built relative page.

At any time, exactly one version is Current and at most one is Previous; both MUST correspond to files that exist in the repository. Every link on the homepage, in `llms.txt`, and in the agent-integration files MUST resolve within `SPECmd-app/SPEC.md` — none MUST point at a different repository for required content. Links to the Portables and to companion tooling are optional navigation to sibling repositories, not required content.

## 4. Requirements

- FUN-001: The homepage MUST link to the Current Core version and its Optional companion.
- FUN-002: The homepage's version section MUST show only the Current and Previous main versions, plus a link to the full changelog for earlier history.
- FUN-003: The site MUST expose `llms.txt` at the Site Source root, linking machine-readable (raw) sources for Core, Optional, the worked example, both agent integrations, and the changelog.
- FUN-004: The site MUST expose `robots.txt` at the Site Source root, allowing crawling and referencing this site's own sitemap.
- FUN-005: The homepage MUST include machine-readable structured data (JSON-LD) describing this site and the Current Core version.
- FUN-007: The site MUST present the Portables on their own page (`portables.html`), not on the homepage, linked from the primary navigation as the last item after Versions. That page MUST link to the `SPECmd-app/spec-md-examples` repository and to each portable's SPEC.md (and TRACE.md where one exists), and MUST list the Judo Club first, identified as the canonical example that is also a fully implementable portable. Portables MUST be listed one below the other. Each portable entry MUST show the SPEC.md version it targets (Core, and Optional where the portable uses it). The primary-navigation link, and any homepage button that leads to the Portables page, MUST open it in a new browser tab. The homepage MUST NOT list the individual portables; the Judo Club keeps its own reference-example section there, which MUST point readers to the Portables page for more implementable examples.
- FUN-008: `llms.txt` MUST list each portable that the Portables page lists, with a machine-readable (raw) link.
- FUN-006: The homepage MUST include a section covering companion tooling (e.g. a CLI/MCP-server implementation of the format), linking to that tooling's own repository, and MUST NOT claim capabilities that tooling's own documentation does not claim for itself.
- ANL-001: The site MUST load Google Analytics 4 (measurement ID `G-6ELH0H1RPK`) on the homepage and on every Jekyll-built page, but only after the visitor has granted consent (PRIV-001).
- PRIV-001: Before consent is granted, the site MUST NOT request the analytics script, set analytics cookies, or send visitor data to any analytics provider.
- PRIV-002: When no consent choice is stored, the homepage and every Jekyll-built page MUST present a consent banner offering Accept and Decline, with equal prominence of access.
- PRIV-003: Accept MUST store the choice client-side and load the analytics tag, including on later visits without asking again. Decline MUST store the choice and MUST NOT load the analytics tag, including on later visits.
- PRIV-004: A persistent control (labelled "Cookie settings") MUST let the visitor reopen the choice at any time. Declining after having accepted MUST stop analytics for the rest of the visit (a page reload is acceptable).
- PRIV-005: The consent choice MUST be stored only in the visitor's browser (`localStorage`); the site MUST NOT persist it server-side. If browser storage is unavailable, the site MUST still render correctly and MUST treat consent as not granted on the next visit.
- UX-001: Where multiple version cards are displayed together (Current/Previous), their primary actions (buttons) MUST align to the same position regardless of differing description length.
- DATA-001: The site MUST NOT link to a version file, example, or integration file that does not exist in the repository, nor to a portable that does not exist in `SPECmd-app/spec-md-examples`.
- DATA-002: When a new Core version is released, the homepage, `llms.txt`, and the agent-integration files MUST be updated in the same change to reference it.
- INT-001: No page, file, or integration prompt in this repository MUST depend on content hosted in a different repository. The consent-gated analytics service (ANL-001) is not repository content, and the site MUST remain fully functional when it is declined or blocked.

## 5. Interfaces and External Contracts

The site is built and served by GitHub Pages from the `main` branch, with `docs/` configured as the publishing source. Files with YAML front matter under `docs/` are converted to HTML by Jekyll; files without front matter (e.g. `robots.txt`, `llms.txt`, `index.html`) are served as-is. Content outside `docs/` (`examples/`, `integrations/`, `CHANGELOG.md`, `README.md`, `LICENSE`) is not processed by Jekyll and is reached only via GitHub's own blob/raw URLs.

This repository has no required external interfaces to other repositories.

**Google Analytics 4 (optional, consent-gated).**
- **Purpose/role**: Aggregate visit measurement for the homepage and built pages.
- **Data authority**: Google Analytics is authoritative only for its own reports; nothing on the site depends on its data.
- **Mechanism**: `gtag.js` loaded from `googletagmanager.com` by an inline script, only after consent (PRIV-001, PRIV-003).
- **Failure behavior**: If the script is blocked, fails, or consent is declined, the site MUST behave identically for content and navigation (INT-001).

## 6. Constraints and Non-Goals

No JavaScript-driven interactivity is required beyond in-page anchor navigation and the analytics-consent control (PRIV-002, PRIV-004). No CMS or server-side backend is required. Visitor tracking is permitted only as specified in ANL-001 and PRIV-001..005.

Companion tooling coverage (FUN-006) is informational: the website MUST NOT reimplement or fork the tooling's behavior, and MUST NOT get out of sync with that tooling's own disclosed capabilities and known gaps.

## 7. Verification and Acceptance

Given a new Core version is released, when the release is complete, then the homepage's version section MUST show that version as Current, and the previously Current version MUST become Previous (DATA-001, DATA-002, FUN-002).

`llms.txt` and `robots.txt` MUST be reachable at the Site Source root and MUST reference only files that exist in the repository (FUN-003, FUN-004, INT-001).

Given a first visit with no stored choice (PRIV-001, PRIV-002), when the page loads, then the banner is shown and no request to `googletagmanager.com` has been made.

Given the visitor selects Accept (ANL-001, PRIV-003), when the choice is made, then the analytics script is requested, the choice is stored, and on a later visit it loads without the banner.

Given the visitor selects Decline (PRIV-003), when the choice is made and on later visits, then no analytics script is requested and the banner is not shown.

Given the visitor previously accepted (PRIV-004), when they use "Cookie settings" and select Decline, then analytics stops for the rest of the visit.

Given a Jekyll-built page such as a Core version document (ANL-001, PRIV-002), when it loads, then it shows the same banner and follows the same rules.

Given the homepage is loaded (FUN-007), when the primary navigation is inspected, then "Portables" is the last link after Versions, the homepage shows no portable cards, and the link opens a page that links to the examples repository and to each listed portable and lists Judo Club first as the canonical example and a fully implementable portable, followed by the others, one below the other.

Given the Portables page is loaded (FUN-007), when a portable entry is inspected, then it shows the Core version (and Optional version where used) it targets. Given the homepage's Judo Club section, when it is inspected, then it points to the Portables page, and following that link opens a new tab.

Given a portable is added to `spec-md-examples` (FUN-007, FUN-008, DATA-001), when the site is updated, then the Portables page and `llms.txt` list it, and every link resolves.

Given the Current and Previous version cards are rendered together, when their description text differs in length, then their primary-action buttons MUST still align to the same vertical position (UX-001).

## 8. Notes and Rationale

An earlier draft of this specification (`spec_version` 0.3.0) described a standalone, private companion repository (`xonix999/spec-md-website`) that duplicated Core, Optional, worked examples, and integrations so it had no outside dependency. At `spec_version` 0.4.0, site development (the `docs/` publishing-source split, the logo change, the Tooling section) had continued only on the public `SPECmd-app/SPEC.md` repository, and the private repository had fallen out of sync.

It has since been brought back up to date — logo, Tooling section (including the Blackbox test card), the button-alignment fix, and the em-dash cleanup are all mirrored across — so it once again matches the live site's content. Its own GitHub Pages instance still cannot be published under the account's current plan (see §6), which is unrelated to whether its content is current. `SPECmd-app/SPEC.md` remains this specification's system of record regardless of the private repository's sync state.

Splitting worked examples and agent-integration files out of the Jekyll-built `docs/` directory keeps the Site Source small while letting GitHub itself serve those files' rendered/raw views, at the cost of losing a Pages-rendered HTML view for them (they render via GitHub's own blob/raw viewer instead).

### Version 0.5.0 Clarification

Added consent-gated Google Analytics 4 (ANL-001, PRIV-001..005). This reverses the earlier non-goal of "no analytics / no visitor tracking" (0.4.1 and before), so it is a MINOR change. It also updates `specmd` to Core 0.4.3; Core 0.4.2 and 0.4.3 were clarification-only and do not change this document's required behavior. Consent is opt-in rather than opt-out because the site is public and may be visited from jurisdictions requiring prior consent (informative, not legal advice).

### Version 0.6.0 Clarification

Added the Portables page (FUN-007, FUN-008) and named the Judo Club the canonical example, which is also listed as the first portable. This adds requirements, so it is a MINOR change. ### Version 0.6.1 Clarification

Clarified FUN-007: per-portable version display, opening the Portables page in a new tab, and the homepage's pointer from the Judo Club section. These describe behavior already implemented and add no new capability, so this is a PATCH change.

Portable cards on that page are maintained by hand alongside the examples repository, like the version links.

Version links are updated manually alongside each Core release; this could be automated later, but that is not required for conformance.
