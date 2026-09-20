---
specmd: "0.4.3"
spec_version: "0.1.1"
status: draft
name: "Internal IT Ticketing System"
last_updated: "2026-09-13"
---

<!-- SPDX-License-Identifier: Apache-2.0 -->

# SPEC.md — Internal IT Ticketing System

## Specification Contract

- Uppercase MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY use BCP 14 (RFC 2119 + RFC 8174).
- Normative requirements define conformance.
- Examples, notes, rationale, history, and implementation guidance are informative unless explicitly marked normative.
- This SPEC.md is authoritative for required behavior when implementation code or implementation documentation conflicts with it.
- A conforming implementation MAY use different languages, frameworks, databases, infrastructure, architecture, source organization, or visual design unless explicitly constrained.
- If an unspecified implementation choice cannot materially alter conformance, the implementer MAY choose an appropriate solution.
- If an omission or ambiguity could materially affect observable behavior, data semantics, security, privacy, accessibility, interoperability, safety, or another normative property, it MUST be surfaced rather than silently resolved.
- Unresolved product/design decisions are marked `TBD` / Open Issue below, not guessed.

## 1. Overview and Scope

**Purpose.** An internal IT service desk ticketing system: employees of the organization report issues or requests to IT, and IT staff (agents) track and resolve them as tickets.

**Actors.**
- **Requester** — an employee of the organization who submits a ticket to IT.
- **Agent** — IT staff who work tickets (view, triage, respond, resolve).
- **System integrations** — email infrastructure and API clients that create tickets on behalf of a Requester.

**Goals.**
- Give employees a reliable way to report issues/requests to IT through the channels in scope.
- Give IT agents a single place to see, respond to, and resolve those requests, authenticated via the organization's existing identity provider.

**Design/system boundary.** This SPEC.md covers ticket intake, the identity of the two actor types, and authentication for Agents. It does not yet cover ticket lifecycle/workflow, SLAs, notifications, or reporting — these are undecided (see Open Issues) and will be added as the design grows.

**In scope (this revision).**
- Single organization, internal IT service desk.
- Ticket intake via web portal, API, and inbound email.
- Agent authentication via Google or Microsoft SSO.

**Out of scope (this revision).**
- Multi-tenancy (serving more than one separate organization).
- Any intake channel other than web portal, API, and email (e.g., chat widget, phone/IVR, SMS, Slack/Teams).
- Ticket status workflow, SLAs, notifications, reporting/analytics, and Requester authentication — not yet decided (Open Issues).

## 2. Context and Definitions

- **Organization** — the single company/entity this system serves. There is exactly one Organization (see TENANT-001).
- **Requester** — see Overview.
- **Agent** — see Overview. "Staff" and "Agent" are used interchangeably; **Agent** is the canonical term in normative text.
- **Ticket** — a record of a single issue or request raised by a Requester, tracked by Agents through to resolution. (Ticket structure/fields/lifecycle: TBD.)
- **Intake channel** — a means by which a new Ticket is created: Web Portal, API, or Email.
- **SSO** — authentication delegated to an external identity provider (Google or Microsoft) using OAuth 2.0 / OIDC, rather than a locally-managed password.
- Dates use ISO 8601. Text uses UTF-8. `null`, absent, empty string, and empty collection are distinct unless stated otherwise.

## 3. System Model

**Actors and relationship.**
- A Requester belongs to the Organization and creates Tickets.
- An Agent belongs to the Organization and works Tickets created by any Requester.
- One Ticket has exactly one originating Requester and zero or more Agents involved over its life (assignment model: TBD).

**Entities (minimal, to be extended).**
- **Ticket**: created through exactly one Intake channel; has an originating Requester identity. Full field set, states, and transitions: TBD (Open Issue) — not invented here.
- **User identity**: an Agent's identity is resolved from Google/Microsoft SSO at authentication time (see AUTH-001). A Requester's identity-resolution method is TBD (Open Issue) — e.g., whether Requesters authenticate at all, or are identified only by email address on submission.

**Invariants.**
- INV-001: Every Ticket MUST have exactly one Organization context (the single Organization — see TENANT-001), and MUST NOT be attributable to more than one Organization.
- INV-002: Every Ticket MUST have exactly one originating Intake channel recorded (Web Portal, API, or Email).

## 4. Requirements

### Tenancy

- **TENANT-001**: The system MUST serve exactly one Organization. It MUST NOT implement cross-organization data isolation, per-organization branding/config, or any other multi-tenant product feature.

### Authentication

- **AUTH-001**: Agents MUST authenticate using Google or Microsoft SSO (OAuth 2.0 / OIDC) before accessing any Agent-facing capability.
- **AUTH-002**: The system MUST NOT grant Agent-facing access to a caller who has not completed AUTH-001, regardless of Intake channel.
- **AUTH-003** (Open Issue / TBD): Whether Requesters authenticate to use the Web Portal or API, and if so by what method (e.g., the same SSO, a separate mechanism, or none), is undecided. Do not assume Requesters use the same SSO as Agents.

### Ticket Intake

- **INTAKE-001**: The system MUST allow a Requester to create a Ticket via a Web Portal.
- **INTAKE-002**: The system MUST allow a Ticket to be created via an API call equivalent in effect to INTAKE-001.
- **INTAKE-003**: The system MUST allow a Ticket to be created from an inbound email sent to a designated support address (email-to-ticket). The sender's email address MUST be recorded as the originating Requester identity.
- **INTAKE-004**: A Ticket created via any of INTAKE-001, INTAKE-002, or INTAKE-003 MUST be equivalent to Agents once created — Agents MUST NOT be able to tell, from ticket handling alone, which channel was used, beyond an explicitly recorded intake-channel field (INV-002).
- **INTAKE-005** (Open Issue / TBD): Whether a reply to an existing email thread appends to the original Ticket (vs. always creating a new Ticket) is undecided.
- **INTAKE-006** (non-goal / explicit prohibition): The system MUST NOT be required to support intake channels other than Web Portal, API, and Email in this revision (e.g., chat widget, phone/IVR, SMS, Slack/Teams are explicitly out of scope until added in a future revision).

## 5. Interfaces and External Contracts

### Google / Microsoft SSO (Agent authentication)

- **Purpose/role**: External identity provider for Agent login.
- **Data authority**: The IdP (Google Workspace or Microsoft Entra ID / Microsoft 365) is authoritative for the Agent's verified email, name, and account-active status.
- **Authentication**: OAuth 2.0 / OIDC. Exact scopes, tenant-restriction rules (e.g., restricting to the Organization's own Google/Microsoft tenant), and provisioning behavior (just-in-time account creation vs. pre-provisioned Agent list) are Open Issues (TBD).
- **Supported operations**: Authentication (sign-in) only, in this revision. Directory sync / SCIM provisioning is not specified (TBD).
- **Failure behavior**: If SSO authentication fails or is denied by the IdP, the system MUST NOT grant Agent-facing access (see AUTH-002).

### Inbound Email (Email-to-ticket)

- **Purpose/role**: Lets a Requester create a Ticket by sending email to a designated address.
- **Data authority**: The email's `From` address is authoritative for the Requester's identity on that Ticket (INTAKE-003).
- **Mechanism** (Open Issue / TBD): Whether ingestion is via IMAP polling, an inbound-email provider webhook (e.g., SendGrid/Mailgun/Postmark inbound parse), or a direct SMTP relay is undecided — an implementer MAY choose, since it does not materially alter observable ticket-creation behavior, unless threading behavior (INTAKE-005) depends on it.
- **Failure behavior**: Undecided — e.g., whether a malformed/unparseable email is dropped, bounced, or creates a Ticket flagged for review (Open Issue).

### Ticket Creation API

- **Purpose/role**: Lets external systems create Tickets programmatically (INTAKE-002).
- **Authentication/identity expectations** (Open Issue / TBD): Whether the API uses API keys, OAuth client credentials, or the same Agent SSO is undecided.
- **Supported operations**: Ticket creation, in this revision. Read/update/delete via API: TBD.

## 6. Constraints and Non-Goals

- The system is single-tenant (TENANT-001). Multi-tenant SaaS concerns (tenant isolation, per-tenant billing, per-tenant branding) are explicit non-goals for this revision.
- Agent authentication is delegated to Google/Microsoft SSO (AUTH-001); this SPEC.md does not constrain the underlying implementation technology, database, or hosting.
- Intake is limited to Web Portal, API, and Email in this revision (INTAKE-006).

## 7. Verification and Acceptance

- **AUTH-001 / AUTH-002** — Given a person without a completed Google/Microsoft SSO sign-in, when they attempt to access any Agent-facing capability, then access MUST be denied. Given a person completes Google/Microsoft SSO sign-in successfully, when they access an Agent-facing capability, then access MUST be granted with an identity resolved from the IdP.
- **TENANT-001 / INV-001** — Given the system's data model, when inspected, then no Ticket, Requester, or Agent record is associated with more than one Organization.
- **INTAKE-001** — Given a Requester on the Web Portal, when they submit a new-ticket form with the required fields (fields: TBD), then a new Ticket is created and becomes visible to Agents.
- **INTAKE-002** — Given a valid API call to the ticket-creation endpoint, when the request is well-formed, then a new Ticket is created equivalent to one created via INTAKE-001 (verified by comparing resulting Ticket data, excluding the intake-channel field).
- **INTAKE-003** — Given an email sent to the designated support address, when it is received and parsed, then a new Ticket is created with the sender's email recorded as the originating Requester identity.
- **INTAKE-006** — Given an attempt to create a Ticket through a channel other than Web Portal, API, or Email, when that channel is not implemented, then this is conforming (not a defect) for this revision.

## 8. Notes and Rationale

This is a starting SPEC.md, deliberately minimal per SPEC.md Core §13 (Living Specification). It captures the four decisions made so far — service-desk type, SSO scope, single-organization tenancy, and intake channels — and leaves everything else open rather than inventing product behavior. Expect this document to grow section-by-section as ticket lifecycle, roles/permissions, SLAs, notifications, attachment handling, data retention, and Requester authentication are decided; each such decision should update this SPEC.md in the same change that implements it.

### Version 0.1.1 Clarification

This revision updates the document to SPEC.md Core 0.4.3 (from 0.4.1). Core 0.4.2 and 0.4.3 were both clarification-only PATCH releases — none of their changes (the design-vs-invention boundary, non-goal wording, and stable-ID/TRACE-coverage clarifications) affect this document's required behavior, since it uses no `TRACE.md` or behavioral-flow identifiers. This is therefore a PATCH-level change.

## Open Issues (must be resolved or explicitly deferred before related implementation)

1. Requester authentication: does a Requester need to sign in at all, and if so how (AUTH-003)?
2. Ticket lifecycle: fields, statuses, transitions, assignment model, priority/SLA — none defined yet.
3. Email threading: does a reply to a ticket's email thread append to the existing Ticket, or always create a new one (INTAKE-005)?
4. Email ingestion mechanism and failure handling (malformed/unparseable email).
5. Ticket API authentication mechanism (API key vs. OAuth client credentials vs. Agent SSO) and which operations beyond creation it exposes.
6. SSO provisioning: tenant restriction to the Organization's own Google/Microsoft tenant, and just-in-time vs. pre-provisioned Agent accounts.
7. Roles/permissions beyond the binary Requester/Agent distinction (e.g., Admin, Team Lead).
8. Notifications (email/other) to Requesters and Agents on ticket events.
9. Attachment support and limits on any Intake channel.
10. Data retention and deletion policy.
