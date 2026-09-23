<!-- SPDX-License-Identifier: Apache-2.0 -->

# Internal IT Ticketing System — SPEC.md Example

> A deliberately small **Living Specification** showing how SPEC.md captures only the product decisions that have actually been made — and leaves everything else explicitly unresolved instead of inventing behavior.

This directory is part of [`SPECmd-app/spec-md-examples`](https://github.com/SPECmd-app/spec-md-examples) and demonstrates a **Core-only SPEC.md** for an internal IT service desk.

The example is intentionally incomplete as a product design.

That is the point.

It shows how to start a specification when only a handful of business decisions are known, while keeping the document precise enough to guide implementation without silently filling in undecided behavior.

---

## Repository location

```text
SPECmd-app/spec-md-examples/
└── internal-it-ticketing-system/
    └── SPEC.md
```

Current example:

- [`SPEC.md`](https://github.com/SPECmd-app/spec-md-examples/blob/main/internal-it-ticketing-system/SPEC.md)

SPEC.md standard:

- https://github.com/SPECmd-app/SPEC.md

Examples repository:

- https://github.com/SPECmd-app/spec-md-examples

---

# Why this example exists

Many specifications fail in one of two ways:

1. they are too vague to implement; or
2. they invent large amounts of product behavior that nobody actually decided.

This example demonstrates a third approach.

It records the decisions that are known:

1. the product is an **internal IT service desk**;
2. the system serves **one organization**;
3. **Agents authenticate with Google or Microsoft SSO**;
4. tickets can enter through **Web Portal, API, or Email**.

Everything else that could materially affect the product is either:

- left explicitly `TBD`;
- identified as an **Open Issue**; or
- placed out of scope for this revision.

This is the SPEC.md **Living Specification** principle in practice.

The specification can grow as decisions are made without pretending that future design choices are already settled.

---

# Current specification status

The current Root Specification declares:

```yaml
specmd: "0.4.3"
spec_version: "0.1.1"
status: draft
name: "Internal IT Ticketing System"
last_updated: "2026-09-13"
```

This example uses **SPEC.md Core 0.4.3** only.

It does not currently declare the Optional companion and does not use `TRACE.md`.

That is intentional: a small early-stage specification does not need additional artifacts merely for the sake of having them.

---

# Specification authority

`SPEC.md` is the authoritative source for required behavior.

The specification explicitly distinguishes:

```text
Normative behavior
    vs.
Examples / notes / rationale / implementation guidance
```

Normative terms such as:

```text
MUST
MUST NOT
SHOULD
SHOULD NOT
MAY
```

use BCP 14 meanings.

If implementation code, implementation documentation, examples, or this README conflict with `SPEC.md`, the specification wins.

This README is explanatory.

It does not create additional product requirements.

---

# Product purpose

The system is an **internal IT service desk ticketing system**.

Employees report technical issues or service requests to IT.

IT staff receive those requests as Tickets and work toward resolving them.

At this stage, the specification deliberately focuses on:

- who the main actors are;
- how tickets enter the system;
- the single-organization boundary; and
- how IT Agents authenticate.

It does **not** yet define a complete ticket-management application.

---

# Actors

The specification currently defines three external actor categories.

## Requester

A **Requester** is an employee of the organization who raises an IT issue or service request.

A Requester may create a Ticket through one of the supported intake channels:

```text
Web Portal
API
Email
```

How Requesters authenticate is intentionally unresolved.

---

## Agent

An **Agent** is a member of IT staff who works Tickets.

Agents may eventually perform activities such as triage, assignment, response, and resolution, but the detailed workflow for those activities is not yet specified.

One requirement is already fixed:

> Agents must authenticate using Google or Microsoft SSO before using Agent-facing functionality.

---

## System integrations

External systems may create Tickets through supported machine-facing channels.

The current specification explicitly recognizes:

- API clients; and
- inbound email infrastructure.

The authentication model for Ticket-creation API clients remains an Open Issue.

---

# Organization model

The system serves exactly **one Organization**.

This is a deliberate product constraint, not simply an implementation shortcut.

The system therefore does **not** need to provide:

- tenant isolation;
- tenant switching;
- per-tenant configuration;
- per-tenant branding;
- per-tenant billing; or
- other multi-tenant SaaS behavior.

The key requirement is:

```text
TENANT-001
```

Every Ticket belongs to the same single Organization context.

---

# Current scope

## In scope

The current revision includes:

- one internal organization;
- Requesters;
- IT Agents;
- Ticket intake;
- Web Portal ticket creation;
- API ticket creation;
- inbound email-to-ticket;
- Google SSO for Agents;
- Microsoft SSO for Agents; and
- recording the originating intake channel.

---

## Explicitly out of scope

The current revision does not require:

- multi-tenancy;
- chat-widget intake;
- phone or IVR intake;
- SMS intake;
- Slack intake;
- Microsoft Teams intake;
- a defined Ticket workflow;
- SLA behavior;
- notifications;
- analytics/reporting;
- defined Requester authentication; or
- other capabilities not yet added to the specification.

An implementation is not defective merely because one of these capabilities is absent.

---

# Core domain concepts

## Organization

The single company or organizational entity served by the system.

There is exactly one Organization in the current product model.

---

## Requester

An employee who originates a Ticket.

Each Ticket has exactly one originating Requester identity.

The exact identity-resolution mechanism depends on the intake channel and remains partially unresolved.

---

## Agent

IT staff who work Tickets.

Agents authenticate through the organization's Google or Microsoft identity provider.

---

## Ticket

A Ticket represents one reported issue or service request.

At this stage only a minimal Ticket model is normative.

A Ticket must have:

```text
one Organization context
one originating Requester
one originating Intake channel
```

The complete Ticket field set is intentionally not defined yet.

---

## Intake channel

Every Ticket is created through exactly one of:

```text
WEB_PORTAL
API
EMAIL
```

The specific internal representation is implementation-defined, but the originating channel must be recorded.

---

# System model

The current conceptual model can be summarized as:

```text
                 ┌───────────────────────┐
                 │   Single Organization │
                 └───────────┬───────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
        ┌───────────┐                 ┌─────────┐
        │ Requester │                 │  Agent  │
        └─────┬─────┘                 └────┬────┘
              │                            │
              │ creates                    │ works
              ▼                            ▼
        ┌──────────────────────────────────────┐
        │                Ticket                │
        └──────────────────────────────────────┘
                    ▲        ▲        ▲
                    │        │        │
                  Web       API     Email
```

This diagram is explanatory only.

`SPEC.md` remains authoritative.

---

# Invariants

The current version defines two important system invariants.

## INV-001 — Single Organization context

Every Ticket must belong to exactly one Organization context.

A Ticket must never be attributable to multiple organizations.

Because the current product is single-organization, all Ticket, Requester, and Agent data remains inside that one organizational context.

---

## INV-002 — Exactly one originating intake channel

Every Ticket must record exactly one originating intake channel.

The permitted channels in this revision are:

```text
Web Portal
API
Email
```

This enables the system to retain provenance without changing how Agents handle Tickets.

---

# Requirements overview

The current specification is intentionally compact.

Its primary requirements fall into three groups:

```text
Tenancy
Authentication
Ticket Intake
```

---

# Tenancy requirements

## TENANT-001

The product serves exactly one Organization.

It must not implement cross-organization product behavior such as:

- tenant isolation;
- per-tenant configuration;
- multi-tenant branding; or
- equivalent multi-tenant features.

This requirement keeps the initial product boundary explicit.

---

# Authentication requirements

## AUTH-001 — Agent SSO

Agents must authenticate using either:

```text
Google SSO
Microsoft SSO
```

using OAuth 2.0 / OpenID Connect.

Local Agent passwords are therefore not part of the currently specified authentication contract.

---

## AUTH-002 — No unauthenticated Agent access

A caller who has not successfully completed the required SSO authentication must not gain access to Agent-facing capabilities.

This remains true regardless of the channel from which the caller arrived.

---

## AUTH-003 — Requester authentication

Requester authentication is deliberately unresolved.

Possible future choices might include:

- the same organization SSO;
- a different identity mechanism;
- no login at all;
- email-only identification; or
- another explicitly designed mechanism.

None of those possibilities should be treated as current behavior until the specification resolves the issue.

---

# Ticket intake requirements

Three intake channels are required.

```text
Web Portal
API
Inbound Email
```

All three create the same conceptual Ticket type.

---

## INTAKE-001 — Web Portal

A Requester must be able to create a Ticket using a Web Portal.

The complete form schema is not yet defined.

Therefore an implementation must not treat a guessed set of fields as normative product behavior.

---

## INTAKE-002 — API

An external caller must be able to create a Ticket programmatically through an API.

Creation must be equivalent in effect to Web Portal Ticket creation.

The API authentication mechanism remains unresolved.

---

## INTAKE-003 — Email

The system must support email-to-ticket.

An email sent to the designated support address creates a Ticket.

The sender's email address is recorded as the originating Requester identity.

---

## INTAKE-004 — Channel equivalence

Once a Ticket exists, Agents must be able to handle it independently of how it entered the system.

Conceptually:

```text
Web Portal ─┐
API ────────┼──> Ticket ───> Agent handling
Email ──────┘
```

The originating channel remains visible through the explicitly recorded intake-channel field.

Other Agent-facing handling behavior must not arbitrarily differ only because a Ticket came from another supported channel.

---

## INTAKE-005 — Email thread behavior

This is unresolved.

A reply to an existing email conversation might:

```text
append to the existing Ticket
```

or:

```text
create a new Ticket
```

The specification deliberately refuses to choose until the product decision is made.

---

## INTAKE-006 — No additional intake requirement

The current revision does not require any intake mechanism beyond:

```text
Web Portal
API
Email
```

For example, absence of these features is conforming:

```text
Slack
Teams
SMS
Phone
IVR
Chat widget
```

until a future revision explicitly adds them.

---

# Intake flows

The following diagrams explain the currently specified observable behavior.

They are not additional requirements.

---

## Web Portal intake

```text
Requester
   │
   │ opens portal
   ▼
Web Ticket Form
   │
   │ submit
   ▼
Ticket Creation
   │
   ▼
Ticket
   │
   ▼
Visible to Agents
```

The exact required fields remain TBD.

---

## API intake

```text
External API Client
        │
        │ Ticket creation request
        ▼
Ticket Creation API
        │
        ▼
      Ticket
        │
        ▼
Visible to Agents
```

The authentication mechanism remains TBD.

---

## Email intake

```text
Requester
   │
   │ sends email
   ▼
Designated Support Address
   │
   ▼
Email Ingestion
   │
   │ From address becomes
   │ Requester identity
   ▼
Ticket
   │
   ▼
Visible to Agents
```

The internal email-ingestion mechanism is not currently constrained.

---

# Agent authentication flow

At a high level:

```text
Agent
  │
  ▼
Sign in
  │
  ├─────────────┐
  ▼             ▼
Google        Microsoft
OIDC          OIDC
  │             │
  └──────┬──────┘
         ▼
Successful identity verification?
         │
    ┌────┴────┐
   No         Yes
    │          │
    ▼          ▼
 Deny      Agent access
 access      permitted
```

Exact:

- scopes;
- organization/tenant restriction;
- account provisioning;
- group mapping; and
- directory synchronization

are not yet fixed.

---

# External interfaces

The specification identifies three major external contracts.

---

## Google / Microsoft SSO

### Role

Authenticates Agents.

### Authority

The external identity provider is authoritative for the Agent's:

- verified email;
- name; and
- active-account status.

### Protocol

```text
OAuth 2.0 / OIDC
```

### Not yet defined

- requested scopes;
- Google Workspace domain restriction;
- Microsoft Entra tenant restriction;
- just-in-time Agent creation;
- pre-provisioned Agent records;
- group/role mapping;
- SCIM or directory synchronization.

### Required failure behavior

Failed or denied SSO must not result in Agent access.

---

# Inbound Email interface

## Role

Allows a Requester to create a Ticket by sending email.

## Identity rule

The email:

```text
From:
```

address is the authoritative originating Requester identity for the created Ticket.

## Implementation freedom

The internal mechanism may be chosen by the implementer where it does not change observable behavior.

Possible implementations include:

```text
IMAP polling
Inbound provider webhook
SMTP relay
Inbound parsing service
```

These are examples, not requirements.

## Unresolved failure behavior

The current specification does not yet decide what happens to malformed or unparseable email.

Possible future policies could include:

- reject;
- bounce;
- quarantine;
- create a review Ticket;
- log and alert.

No option should be treated as normative until chosen.

---

# Ticket Creation API

## Role

Allows external systems to create Tickets programmatically.

## Current operation

```text
Create Ticket
```

is the only API behavior explicitly required in this revision.

## Not yet defined

The specification does not yet choose:

- API keys;
- OAuth client credentials;
- Agent SSO;
- another machine authentication method;
- read operations;
- update operations;
- deletion operations; or
- complete API resource design.

These should not be invented merely because a conventional REST API often includes them.

---

# Acceptance and verification

Even though the design is small, the specification already defines observable acceptance behavior.

This is important: a Living Specification can still be testable before the entire product is designed.

---

## Agent authentication acceptance

Expected behavior:

```text
Unauthenticated caller
        │
        ▼
Agent-facing capability
        │
        ▼
      DENY
```

and:

```text
Successful Google/Microsoft SSO
        │
        ▼
Identity resolved from IdP
        │
        ▼
Agent-facing access permitted
```

---

## Single-organization acceptance

Inspection of the system model must demonstrate that no:

```text
Ticket
Requester
Agent
```

is associated with more than one Organization.

---

## Web intake acceptance

A valid Web Portal submission creates a Ticket that becomes visible to Agents.

The precise form fields remain TBD.

---

## API intake acceptance

A valid Ticket-creation API request creates a Ticket equivalent to one created through the Web Portal except for the recorded intake-channel value.

---

## Email intake acceptance

A received and successfully parsed support email creates a Ticket whose originating Requester identity is the sender's email address.

---

## Unsupported-channel acceptance

If Slack, SMS, phone, chat, or another unsupported channel is not implemented, that absence is conforming for this revision.

This is an example of an explicit **non-goal** protecting the implementation from accidental scope expansion.

---

# Open Issues

The specification intentionally keeps important future decisions visible.

These are not bugs in the specification.

They are unresolved product decisions.

---

## 1. Requester authentication

Questions include:

- Must Requesters authenticate?
- If yes, how?
- Is organization SSO required?
- Can email-originated Requesters exist without interactive login?
- How does an API-created Ticket identify the Requester?

---

## 2. Ticket lifecycle

Still undefined:

- complete Ticket fields;
- status vocabulary;
- allowed transitions;
- assignment;
- ownership;
- priority;
- severity;
- SLA policy;
- reopening;
- cancellation;
- resolution semantics.

Until these decisions are made, no lifecycle should be treated as normative.

---

## 3. Email threading

Still unresolved:

```text
email reply → existing Ticket
```

versus:

```text
email reply → new Ticket
```

This decision may affect both data modeling and the inbound-email integration contract.

---

## 4. Email ingestion and failure handling

Still unresolved:

- IMAP vs. webhook vs. SMTP ingestion;
- malformed MIME;
- unsupported encoding;
- oversized email;
- unparseable sender;
- missing subject;
- spam;
- duplicate delivery;
- rejection/bounce behavior.

Only the observable behavior that has already been specified should be assumed.

---

## 5. Ticket API authentication and scope

Still unresolved:

- API key;
- OAuth client credentials;
- SSO;
- service identity;
- authorization scope;
- operations beyond create.

---

## 6. Agent provisioning and tenant restriction

Still unresolved:

- organization-only Google Workspace accounts;
- organization-only Microsoft Entra tenant;
- JIT Agent creation;
- pre-provisioning;
- account disablement;
- group mapping.

---

## 7. Additional roles and permissions

Only the Requester/Agent distinction is currently established.

Potential future concepts such as:

```text
Administrator
Team Lead
Service Desk Manager
Auditor
```

are not yet required.

---

## 8. Notifications

No notification contract currently exists.

Future decisions may address:

- Ticket-created confirmation;
- assignment;
- Agent reply;
- resolution;
- SLA warning;
- reopening.

These are not current requirements.

---

## 9. Attachments

Attachment behavior is not yet specified for:

- Web Portal;
- API;
- Email.

Future design may need to define:

- supported types;
- maximum size;
- malware scanning;
- retention;
- download authorization;
- inline images.

---

## 10. Data retention and deletion

Still undefined:

- Ticket retention;
- audit retention;
- Requester data retention;
- deleted employee behavior;
- legal holds;
- deletion/anonymization;
- email-body retention.

---

# Why the Open Issues matter

A common anti-pattern would be to see an IT ticketing system and immediately assume:

```text
NEW
OPEN
IN_PROGRESS
WAITING
RESOLVED
CLOSED
```

or to assume:

```text
Priority = Low / Medium / High / Critical
```

or to assume:

```text
Requester must sign in with SSO
```

Those are plausible designs.

They are not currently specified designs.

SPEC.md encourages the author to preserve that distinction.

The implementer should have freedom over non-material implementation choices but must not silently decide externally observable product behavior that the product owner has not decided.

---

# Living Specification workflow

This example is designed to evolve incrementally.

A healthy workflow is:

```text
Business/product decision
        │
        ▼
Update SPEC.md
        │
        ▼
Add/modify requirement IDs
        │
        ▼
Add acceptance behavior
        │
        ▼
Implement
        │
        ▼
Verify
```

For example, when the Ticket lifecycle is decided, one change might introduce:

```text
Ticket fields
Status definitions
Transition rules
Assignment rules
Acceptance scenarios
```

The specification should be updated in the same change that introduces the corresponding behavior.

---

# What should not happen

Avoid this workflow:

```text
Developer guesses behavior
        │
        ▼
Implementation ships
        │
        ▼
Documentation later describes the guess
```

The project should instead make the decision explicit before treating it as required behavior.

---

# Using this example with coding agents

This example is especially useful for demonstrating how a coding agent should react to an intentionally incomplete specification.

A good implementation instruction is:

> Read `SPEC.md` first. Implement the behavior that is normatively defined. Do not invent behavior for Open Issues or TBD areas. Where the specification intentionally leaves an implementation detail open and the choice cannot materially affect observable behavior, choose a simple appropriate solution. If an unresolved choice would materially affect external behavior, security, privacy, or data semantics, surface it instead of guessing.

That distinction is central to SPEC.md.

---

# Suggested implementation approach

The following is **informative guidance only**.

It does not add product requirements.

A prototype could reasonably separate:

```text
Identity
Ticket Intake
Ticket Storage
Agent Access
External Integrations
```

For example:

```text
┌──────────────────┐
│   Web Portal     │
└────────┬─────────┘
         │
┌────────▼─────────┐
│ Ticket Intake    │◄──────── API
│ Application      │◄──────── Email
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Ticket Storage   │
└──────────────────┘

Agent
  │
  ▼
Google / Microsoft SSO
  │
  ▼
Agent-facing application
```

The specification intentionally does not require:

- a particular programming language;
- a particular database;
- microservices;
- serverless;
- containers;
- a particular cloud;
- a particular frontend framework.

---

# Minimal data model implied by the current spec

Without inventing future fields, an implementation needs enough durable information to satisfy the existing invariants.

Conceptually:

```text
Organization
-----------
id

Requester Identity
------------------
identity/reference

Agent Identity
--------------
identity/reference from IdP

Ticket
------
id
organization_context
originating_requester_identity
originating_intake_channel
```

The exact schema is implementation-defined.

This is not a complete future Ticket model.

---

# What an implementer may choose

The specification intentionally allows implementation freedom where a choice does not materially change conformance.

Examples may include:

- language;
- framework;
- database;
- deployment platform;
- source layout;
- internal architecture;
- ORM;
- email provider;
- OIDC library;
- HTTP framework.

Implementation freedom ends when a choice affects required observable behavior.

---

# What an implementer must not guess

Examples include:

- Ticket status model;
- required form fields;
- Ticket priority system;
- SLA behavior;
- Requester authentication;
- email-thread semantics;
- API authentication;
- notification rules;
- attachment limits;
- retention/deletion rules;
- additional Agent roles.

These are product decisions, not implementation details.

---

# Example implementation phases

The following sequence is only a development suggestion.

## Phase 1 — Foundation

Implement the already-decided foundation:

- one Organization context;
- Agent identity;
- Google/Microsoft SSO;
- authenticated Agent boundary.

---

## Phase 2 — Intake core

Implement a shared Ticket-creation application service so:

```text
Web
API
Email
```

produce equivalent Tickets.

---

## Phase 3 — Web intake

Provide Requester-facing Ticket creation without inventing unnecessary workflow behavior.

Any required field decision that materially affects the public contract should first be added to the specification.

---

## Phase 4 — API intake

Expose Ticket creation through an API.

Do not silently choose a permanent production authentication contract while that issue remains open.

A prototype-specific mechanism may be documented clearly as implementation scaffolding rather than normative product behavior.

---

## Phase 5 — Email intake

Integrate inbound email.

Record:

```text
From address
```

as the originating Requester identity.

Preserve the intake channel.

Do not invent email-thread behavior while it remains unresolved.

---

## Phase 6 — Verification

Verify:

- Agent SSO enforcement;
- single-organization invariant;
- Web Ticket creation;
- API Ticket creation;
- Email Ticket creation;
- channel equivalence;
- originating-channel persistence.

---

# When to expand this specification

The document should grow when a real product decision is made.

Good candidates for the next specification revisions include:

```text
Requester identity/authentication
Ticket schema
Ticket lifecycle
Assignment
Priority
SLA
Comments/replies
Email threading
Notifications
Attachments
API authorization
Agent roles
Retention
Audit
Search
Reporting
```

The order should be driven by actual product decisions, not by a desire to fill every common ticketing-system feature.

---

# When TRACE.md may become useful

The current example is deliberately **Core-only** and does not include TRACE.md.

That is appropriate for its present size.

As the system grows, the project may choose to adopt SPEC.md Optional traceability.

A future trace could map:

```text
Requirement ID
    ↓
Design area
    ↓
Implementation
    ↓
Verification evidence
```

For example:

```text
INTAKE-003
    ↓
Email ingestion
    ↓
src/...
    ↓
integration test / acceptance evidence
```

TRACE should be introduced because it provides value, not because every project is required to have one.

---

# Using `specmd`

If you have the `specmd` CLI available, the Root Specification can be used with operations such as:

```bash
specmd validate SPEC.md
```

For quality inspection:

```bash
specmd inspect SPEC.md
```

For Black-Box analysis:

```bash
specmd blackbox SPEC.md
```

For human review:

```bash
specmd render SPEC.md
```

Exact command availability depends on the installed `specmd` version.

The specification itself remains usable without the CLI.

---

# Black-Box review ideas

This example is a good candidate for Black-Box reasoning because the public behavior is intentionally small.

A reviewer can ask:

### Agent identity

- Can an unauthenticated user access any Agent capability?
- Is successful Google/Microsoft identity verification enough to establish Agent identity?
- Are unspecified provisioning decisions being accidentally invented?

### Ticket intake

- Can each required channel create a Ticket?
- Does every created Ticket retain the correct originating Requester?
- Does every Ticket retain exactly one originating channel?
- Are Web/API/Email Tickets equivalent to Agents after creation?

### System boundary

- Is any multi-tenant behavior accidentally becoming required?
- Is an unsupported channel being incorrectly treated as a missing feature?
- Has a Ticket workflow been invented even though the specification leaves it TBD?

This demonstrates how Black-Box analysis can reveal accidental scope expansion as well as missing behavior.

---

# Example tests implied by the current specification

The following examples are explanatory translations of the acceptance criteria.

They are not a replacement for the normative text.

---

## Agent SSO denial

```text
GIVEN a caller has not completed Google/Microsoft SSO
WHEN the caller accesses an Agent-facing capability
THEN access is denied
```

---

## Agent SSO success

```text
GIVEN an Agent successfully authenticates through Google/Microsoft SSO
WHEN the Agent accesses an Agent-facing capability
THEN access is permitted using identity resolved from the IdP
```

---

## Web Ticket creation

```text
GIVEN a Requester uses the Web Portal
WHEN a valid new-ticket submission is made
THEN a Ticket is created
AND it becomes visible to Agents
AND its originating intake channel is Web Portal
```

---

## API Ticket creation

```text
GIVEN a valid Ticket creation API call
WHEN the request succeeds
THEN a Ticket is created
AND it is equivalent to a Web-created Ticket
EXCEPT for the recorded intake channel
```

---

## Email Ticket creation

```text
GIVEN an email is sent to the designated support address
WHEN the email is successfully received and parsed
THEN a Ticket is created
AND the sender address is recorded as the originating Requester identity
AND the intake channel is Email
```

---

# Example of a future specification change

Suppose the product owner later decides:

```text
Statuses:
NEW
IN_PROGRESS
WAITING_FOR_REQUESTER
RESOLVED
CLOSED
```

That should not simply appear in code.

The project should update `SPEC.md` with:

- definitions;
- requirement IDs;
- transition rules;
- authorization rules;
- interface implications;
- acceptance scenarios; and
- Open Issue cleanup.

Then the implementation should follow that specification.

That is the Living Specification model.

---

# Security considerations already implied

Although the complete security model is not yet specified, several current requirements already establish important boundaries.

## Agent authentication

Agent-facing functionality must be protected by successful external SSO.

---

## Identity provider authority

Google/Microsoft are authoritative for the Agent identity attributes identified by the specification.

An implementation should not silently contradict that authority with unrelated local identity values.

---

## API authentication remains unresolved

Because Ticket API authentication is undecided, production security behavior should not be invented as a normative contract.

This is a design decision that needs explicit resolution.

---

## Requester authentication remains unresolved

The same applies to Requester Web Portal behavior.

The system may eventually require authentication or may choose another identity model.

The current spec does not decide.

---

# Data semantics already implied

Even this small specification establishes important data semantics:

```text
One Ticket
  → exactly one Organization context

One Ticket
  → exactly one originating Requester identity

One Ticket
  → exactly one originating Intake channel
```

These rules should survive implementation changes.

---

# Portability

The specification intentionally avoids constraining implementation technology.

A conforming implementation could be written with:

```text
Node.js
Python
Java
Go
.NET
Ruby
PHP
Rust
```

and could use:

```text
PostgreSQL
MySQL
SQLite
MongoDB
another appropriate persistence layer
```

provided the chosen design satisfies the normative behavior.

Likewise, deployment might use:

```text
Vercel
AWS
Azure
Google Cloud
Kubernetes
traditional VM hosting
on-premises infrastructure
```

Nothing in the current specification requires one of these.

---

# Example project layouts

These layouts are only illustrative.

## Monolithic application

```text
src/
├── auth/
├── tickets/
├── intake/
│   ├── web/
│   ├── api/
│   └── email/
└── agents/
```

---

## Layered application

```text
src/
├── domain/
├── application/
├── infrastructure/
├── web/
└── integrations/
```

---

## Serverless application

```text
api/
├── tickets/
├── auth/
└── inbound-email/

src/
├── domain/
└── services/
```

All can be valid if externally observable behavior remains conforming.

---

# Contribution guidance

When changing this example:

1. read `SPEC.md`;
2. decide whether the change represents a real product decision;
3. update the relevant definitions;
4. add or modify stable requirement IDs;
5. update interfaces if the decision affects external contracts;
6. update verification/acceptance behavior;
7. resolve or revise related Open Issues;
8. preserve the distinction between normative and informative text;
9. do not add implementation detail unless it is required for observable behavior;
10. keep the example concise enough to demonstrate the Living Specification principle.

---

# Guidance for AI-assisted changes

When an AI coding or documentation agent works on this project, it should be instructed:

```text
Treat SPEC.md as authoritative.

Do not infer conventional ticketing-system behavior merely because it is common.

Implement only defined normative behavior.

Where an implementation choice does not materially change observable behavior,
choose an appropriate implementation.

Where an unresolved choice would materially change observable behavior,
data semantics, security, privacy, accessibility, interoperability, or another
normative property, surface it rather than guessing.

Keep SPEC.md synchronized when a real product decision changes.
```

This example is particularly useful for teaching agents the difference between:

```text
implementation freedom
```

and:

```text
product-design invention
```

---

# Frequently asked questions

## Why is the Ticket model incomplete?

Because the product has not yet decided the complete Ticket structure.

The specification intentionally does not invent it.

---

## Why are there no Ticket statuses?

The lifecycle is an Open Issue.

Adding conventional statuses without a product decision would undermine the purpose of the example.

---

## Why is there no SLA model?

SLA behavior has not been decided yet.

It is explicitly outside the current defined behavior.

---

## Why can Agents authenticate but Requester authentication is undefined?

Because that reflects the decisions made so far.

SPEC.md does not require every area of a product to reach the same design maturity at the same time.

---

## Why are there three intake channels?

The product decision currently requires:

```text
Web
API
Email
```

Other channels are explicit non-goals for this revision.

---

## Why not add Slack or Teams since this is an internal IT tool?

Because they have not been selected as product requirements.

SPEC.md prevents plausible ideas from becoming accidental requirements.

---

## Why is there no TRACE.md?

This is a small **Core-only** example.

Traceability is not required merely because SPEC.md is being used.

TRACE may be added later if the project adopts the relevant Optional capability.

---

## Can I implement this now?

Yes, but only the behavior that is sufficiently specified.

You can implement:

- the single-organization boundary;
- Agent SSO;
- the three Ticket intake channels;
- minimal Ticket persistence required by the invariants.

For unresolved externally observable product behavior, either keep the implementation appropriately minimal or obtain a product decision and update the specification.

---

## Does this README add requirements?

No.

It is informative.

`SPEC.md` remains authoritative.

---

# What this example teaches

The most important lesson is not how to design an IT ticketing system.

It is how **not to over-specify one**.

The example demonstrates that a useful specification can truthfully say:

```text
We know this.
We do not know that yet.
This behavior is required.
This behavior is intentionally not required.
This implementation choice is free.
This product decision still needs to be made.
```

That clarity is more valuable than a larger document filled with assumptions.

---

# Relationship to other SPEC.md examples

The examples repository contains specifications of different maturity and complexity.

This example represents the lightweight end of the spectrum:

```text
Internal IT Ticketing System
    → Core-only
    → early Living Specification
    → few decided requirements
    → many explicit Open Issues
```

Other examples demonstrate larger Specification Sets, Optional capabilities, traceability, diagrams, and implementation-oriented companions.

The variation is intentional.

SPEC.md is meant to support both small and large designs without forcing every project into the same document structure.

---

# Related resources

## SPEC.md standard

https://github.com/SPECmd-app/SPEC.md

## SPEC.md examples

https://github.com/SPECmd-app/spec-md-examples

## This example

https://github.com/SPECmd-app/spec-md-examples/tree/main/internal-it-ticketing-system

## `specmd` CLI

https://github.com/SPECmd-app/specmd-cli

---

# License

The examples repository states that its example artifacts are licensed under **Apache-2.0**.

This README carries:

```text
SPDX-License-Identifier: Apache-2.0
```

See the repository `LICENSE` file for the complete license terms.

---

# Summary

The Internal IT Ticketing System is a deliberately early-stage SPEC.md example.

Its current design can be summarized in four decisions:

```text
1. Internal IT service desk
2. One Organization
3. Agents authenticate through Google or Microsoft SSO
4. Tickets enter through Web, API, or Email
```

Everything else remains explicit.

That makes this example a useful demonstration of one of SPEC.md's core disciplines:

> **Document decisions. Do not invent them.**
