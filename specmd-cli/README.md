<!-- SPDX-License-Identifier: Apache-2.0 -->

# specmd CLI — SPEC.md Example

> A worked example of a **SPEC.md-driven CLI design** for authoring, validating, inspecting, rendering, tracing, testing, and adapting portable specifications.

This directory is part of [`SPECmd-app/spec-md-examples`](https://github.com/SPECmd-app/spec-md-examples) and demonstrates how a non-trivial command-line tool can be described using the SPEC.md standard.

The example is based on the specification for the [`specmd`](https://github.com/SPECmd-app/specmd-cli) tool itself.

---

## What this example demonstrates

The `specmd-cli` example shows how SPEC.md can be used to describe a real developer tool with:

- a public CLI contract;
- versioned specification metadata;
- stable requirement identifiers;
- invariants;
- black-box behavior;
- deterministic and cognitive analysis;
- traceability through `TRACE.md`;
- acceptance criteria;
- safety and privacy requirements;
- compatibility/version-alignment rules;
- optional normative modules;
- machine-readable output contracts; and
- coding-agent integration.

The goal is not to make the tool dependent on SPEC.md tooling. The specification remains readable as ordinary Markdown and remains the authoritative design artifact independently of any particular implementation.

---

## Files in this example

```text
specmd-cli/
├── SPEC.md
└── TRACE.md
```

### `SPEC.md`

The Root Specification.

It defines the required behavior of the `specmd` tool, including:

- scope and actors;
- terminology;
- command behavior;
- validation;
- trace handling;
- inspection;
- rendering;
- verification coverage;
- Black-Box analysis;
- coding-agent adapters;
- safety and privacy;
- compatibility;
- interfaces;
- acceptance criteria; and
- conformance expectations.

### `TRACE.md`

An informative traceability companion.

It maps specification requirements to:

```text
Requirement
    ↓
Logical design area
    ↓
Implementation reference
    ↓
Verification evidence
```

`TRACE.md` does **not** define product behavior and must not override `SPEC.md`.

---

## Important: this is an example snapshot

This directory is intentionally an illustrative snapshot of the `specmd` Specification Set.

The Root Specification declares four additional **Normative Modules**:

```text
SPECMD_AGENT_INTEGRATIONS.md
SPECMD_CLI_ICD.md
SPECMD_CUCUMBER_CONNECTOR.md
SPECMD_HELP_AND_SOURCES.md
```

Those modules are **not copied into this examples directory**.

For the complete, resolvable Specification Set, use the main repository:

https://github.com/SPECmd-app/specmd-cli

The main repository is the appropriate source when:

- validating the complete Specification Set;
- inspecting exact CLI grammar;
- reviewing cognitive/agent integration rules;
- reviewing Cucumber integration;
- reviewing help and standards-source behavior; or
- implementing the tool.

---

# Specification metadata

The example Root Specification declares:

```yaml
specmd: "0.4.3"
specmd_optional: "0.4.3"
spec_version: "0.13.0"
status: draft
name: "specmd Tool"
```

It uses several SPEC.md Optional capabilities, including:

```yaml
optional_features:
  requirement_metadata: true
  advanced_modularization: true
  diagrams: mermaid
  composed_verification: true
  human_only_processing: true
  trace: true
```

The distinction between these versions matters:

| Field | Meaning |
|---|---|
| `specmd` | Exact SPEC.md Core standard used by the document |
| `specmd_optional` | Exact SPEC.md Optional companion used by the document |
| `spec_version` | Version of this particular product/design specification |
| `specmd_trace` | Version of the TRACE format used by `TRACE.md` |

A tool must not silently validate a document against another SPEC.md version simply because that version is newer or easier to obtain.

---

# What is `specmd`?

`specmd` is specified as a portable command-line tool for working with SPEC.md documents.

Its purpose is to help humans and AI-assisted development tools:

1. create a useful starting specification;
2. validate structural and normative conformance;
3. inspect quality and ambiguity;
4. render specifications for review;
5. evaluate verification coverage;
6. create and maintain traceability;
7. generate thin coding-agent adapters;
8. perform Black-Box analysis;
9. optionally coordinate cognitive or reviewer-agent analysis; and
10. optionally integrate with external acceptance-test runtimes.

The tool is deliberately **not** specified as:

- a project-management system;
- a sprint or approval workflow engine;
- an autonomous implementation agent;
- a replacement for SPEC.md itself;
- a mandatory hosted service;
- a mandatory LLM service; or
- a proprietary specification format.

---

# Core design principles

## SPEC.md remains authoritative

The Root Specification and its declared Normative Modules define required behavior.

Derived artifacts such as:

- `TRACE.md`;
- generated reports;
- adapters;
- rendered HTML/PDF;
- test plans;
- cognitive findings; and
- implementation notes

must not become competing sources of normative behavior.

---

## Implementation freedom is preserved

A conforming implementation may choose its own:

- programming language;
- frameworks;
- package layout;
- libraries;
- storage model;
- renderer;
- parser architecture; and
- internal design

unless the specification explicitly constrains an observable behavior.

This is an important SPEC.md principle: specify **what must be true**, not unnecessary implementation detail.

---

## Deterministic and heuristic evidence stay separate

`specmd` distinguishes deterministic processing from cognitive/heuristic reasoning.

Examples of deterministic work include:

- parsing;
- version resolution;
- structural checks;
- reference checks;
- trace alignment;
- requirement-ID checks; and
- policy checks.

Examples of cognitive work include:

- ambiguity analysis;
- semantic completeness review;
- interface facilitation;
- relationship inference; and
- proposed wording improvements.

Cognitive findings remain **heuristic**.

They must never silently become:

- normative requirements;
- implementation evidence;
- verification evidence; or
- proof of conformance.

---

# Command overview

The public command families specified by this example are:

```text
specmd init
specmd validate
specmd inspect
specmd render
specmd test
specmd blackbox
specmd trace create
specmd trace update
specmd adapt
specmd standards
specmd help
specmd capabilities
specmd cucumber
```

The Cucumber family is optional and separately discoverable.

---

# Quick start

The examples below illustrate the intended user model.

Exact availability depends on the implementation version in use.

## Create a Core specification

```bash
specmd init
```

Equivalent explicit form:

```bash
specmd init SPEC.md --profile core
```

A Core-only specification declares `specmd` but does not need `specmd_optional`.

---

## Create Core + Optional

```bash
specmd init SPEC.md \
  --profile optional \
  --feature trace
```

Optional is always used **with Core**, never instead of Core.

---

## Validate a specification

```bash
specmd validate SPEC.md
```

When trace is enabled:

```bash
specmd validate SPEC.md --trace auto
```

For CI or automation:

```bash
specmd --output-format json validate SPEC.md --trace auto
```

---

## Inspect quality

```bash
specmd inspect SPEC.md
```

Inspection is read-only and is intended to surface issues such as:

- ambiguity;
- undefined terms;
- duplication;
- hidden implementation assumptions;
- weak verification language;
- portability concerns; and
- requirements misplaced in informative material.

---

## Render for human review

```bash
specmd render SPEC.md
```

HTML is the required baseline render format.

Implementations may also advertise PDF support through capabilities.

---

## Evaluate verification coverage

```bash
specmd test SPEC.md
```

This evaluates specification coverage.

It does **not**, by default, execute implementation code or external test suites.

A specification-coverage result and an implementation-test result are different things.

---

# Black-Box analysis

`specmd blackbox` is one of the most important capabilities in this design.

It analyzes the specification **from the outside in**.

The objective is to answer questions such as:

- What operations can an external actor invoke?
- Who invokes them?
- What triggers them?
- What inputs are required?
- What outputs are produced?
- What externally observable errors exist?
- What state changes can be observed?
- Are important interface elements missing?
- Can the specified behavior actually be verified from the outside?
- Are requirements connected to the interfaces that realize them?

Example:

```bash
specmd blackbox SPEC.md --trace auto
```

Machine-readable output:

```bash
specmd --output-format json blackbox SPEC.md --trace auto
```

Export a report:

```bash
specmd blackbox SPEC.md \
  --trace auto \
  --export blackbox-report.json
```

Black-Box analysis is intentionally read-only.

It must not:

- call the implementation;
- probe endpoints;
- execute tests;
- run code from the specification; or
- claim runtime behavior that was never actually tested.

---

# Cognitive analysis

The CLI specification defines three cognitive modes:

| Mode | Meaning |
|---|---|
| `off` | Deterministic processing only |
| `auto` | Use available cognitive analysis; otherwise continue and disclose the limitation |
| `required` | Cognitive analysis must complete or the semantic result is incomplete |

Example:

```bash
specmd blackbox SPEC.md --cognitive off
```

```bash
specmd blackbox SPEC.md --cognitive auto
```

```bash
specmd blackbox SPEC.md --cognitive required
```

Cognitive analysis must never silently resolve a material ambiguity or invent product behavior.

---

# Host-Agent cognitive workflow

The current reference implementation supports a particularly useful pattern for Black-Box analysis: **Host-Agent Mode**.

Instead of giving `specmd` credentials to an LLM provider, the coding agent already running the command supplies the semantic reasoning.

Conceptually:

```text
┌─────────────────────┐
│  Coding Agent       │
│ Claude / Codex/etc. │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ specmd blackbox     │
│ deterministic pass  │
└──────────┬──────────┘
           │
           ▼
   cognitive_package
           │
           ▼
┌─────────────────────┐
│ Host Agent reasons  │
│ over bounded input  │
└──────────┬──────────┘
           │
           ▼
   cognitive_input
           │
           ▼
┌─────────────────────┐
│ specmd validates +  │
│ merges the result   │
└─────────────────────┘
```

A typical two-pass flow is:

### 1. Ask `specmd` for the deterministic analysis

```bash
specmd --output-format json \
  blackbox SPEC.md \
  --trace auto \
  --cognitive auto
```

When semantic input is still required, the result may provide a bounded `cognitive_package`.

### 2. The host agent performs the semantic reasoning

The coding agent analyzes only the prepared context and creates the requested structured cognitive response.

### 3. Supply the response back

```bash
specmd --output-format json \
  blackbox SPEC.md \
  --trace auto \
  --cognitive required \
  --cognitive-input cognitive-result.json
```

Standard input may also be used when supported:

```bash
cat cognitive-result.json | \
  specmd --output-format json \
  blackbox SPEC.md \
  --cognitive required \
  --cognitive-input -
```

The tool checks cited requirement IDs against IDs it extracted deterministically.

An invented or unknown requirement ID is not silently trusted.

This model keeps responsibilities separated:

```text
specmd
  → deterministic extraction, policy and validation

Host Agent
  → semantic reasoning

specmd
  → validates provenance and merges structured findings
```

---

# Additional reviewer agents

The complete specification also defines optional additional reviewer agents.

A command may request one or more configured reviewers:

```bash
specmd inspect SPEC.md \
  --reviewer reviewer-a \
  --reviewer reviewer-b
```

Review policies include:

```text
best-effort
required
```

Reviewer conclusions remain heuristic even when several reviewers agree.

Consensus does not convert an LLM judgment into deterministic evidence.

Material reviewer disagreement must remain visible.

---

# Traceability

`TRACE.md` connects specification requirements to implementation and verification.

The example trace uses the model:

```text
Requirement
    ↓
Logical design area
    ↓
Implementation reference
    ↓
Verification evidence
```

## Validate the pair

```bash
specmd validate SPEC.md --trace auto
```

A Trace Pair has separate states for:

- specification conformance;
- trace conformance;
- pair alignment;
- implementation-reference completeness; and
- verification-evidence completeness.

These dimensions must not be collapsed into one generic “pass”.

---

## Create a trace

For a specification that has the trace feature enabled:

```bash
specmd trace create SPEC.md
```

The conventional output is:

```text
TRACE.md
```

The trace binds itself to the exact Root Specification:

```yaml
specmd_trace: "<trace format>"
traces_file: "SPEC.md"
traces_spec: "<spec_version>"
```

---

## Update a trace

```bash
specmd trace update SPEC.md
```

Trace update should:

- add newly introduced requirement IDs;
- preserve existing manual implementation references;
- preserve existing evidence for unchanged IDs;
- surface removed or changed IDs; and
- validate the resulting pair.

It must not silently delete human-maintained trace information.

---

# Exact version alignment

Version alignment is a central behavior of this design.

A target specification must be evaluated against the **exact standards version it declares**.

Example:

```yaml
specmd: "0.4.3"
specmd_optional: "0.4.3"
```

A tool must not silently substitute:

```text
0.4.2
```

or:

```text
latest
```

because those versions happen to be installed.

If the exact declared version cannot be resolved, the appropriate outcome is:

```text
indeterminate
```

not a false conformance claim.

This makes SPEC.md documents portable and prevents tool upgrades from silently changing the meaning of an older specification.

---

# Standards management

The complete CLI contract includes:

```text
specmd standards list
specmd standards show
specmd standards fetch
specmd standards verify
```

Examples:

```bash
specmd standards list
```

```bash
specmd standards show core 0.4.3
```

```bash
specmd standards show optional 0.4.3
```

```bash
specmd standards verify
```

Offline operation is a first-class requirement when the exact required standards are already available locally.

---

# Coding-agent adapters

The `adapt` command generates **thin instructions**, not a second specification.

Example:

```bash
specmd adapt claude-code SPEC.md
```

Other initially specified targets include:

```text
codex
claude-code
cursor
github-copilot
base44
lovable
```

An adapter must tell the coding agent to:

- read the Root Specification and Normative Modules;
- treat SPEC.md as authoritative;
- preserve declared standards versions;
- keep the specification synchronized with behavior changes;
- keep TRACE synchronized when trace is enabled;
- distinguish implementation freedom from ambiguity and conflict;
- exclude human-only comments from implementation input; and
- avoid duplicating normative product rules into hidden agent instructions.

Generating an adapter and installing an adapter are separate actions.

---

# MCP integration

The complete `specmd` design recommends a structured tool interface such as MCP.

The current reference implementation exposes an MCP stdio server with operations corresponding to core CLI behavior.

Conceptually:

```text
create
validate
inspect
blackbox
trace_create
trace_update
validate_pair
propose_patch
capabilities
```

Structured tools must preserve the same semantics as the CLI.

Using MCP must not create another source of product behavior.

---

# Human-only comments

SPEC.md can contain explicitly delimited human-only material.

Such content may be useful for:

- editorial notes;
- negotiation context;
- private drafting reminders; or
- information intentionally excluded from implementation agents.

Human-only content must not influence:

- implementation;
- conformance;
- acceptance output;
- coding-agent adapters; or
- cognitive-provider context.

Malformed or unclosed human-only blocks are validation errors.

---

# Safety model

The specification deliberately takes a conservative approach.

Analytical commands are read-only.

A command must not transmit specification content externally unless the user explicitly selects functionality that requires transmission.

Generated or write operations must not silently overwrite existing files.

Implementation or test execution must require explicit authorization.

In particular, ordinary analysis must never execute code merely because executable-looking text appears inside:

- `SPEC.md`;
- a Normative Module;
- `TRACE.md`; or
- an acceptance example.

---

# JSON and automation

Commands such as:

```text
validate
inspect
blackbox
test
trace
```

support machine-readable results.

Typical usage:

```bash
specmd --output-format json validate SPEC.md --trace auto
```

In JSON mode:

- stdout should contain the machine-readable result;
- diagnostics should not corrupt that output;
- findings identify severity;
- findings distinguish deterministic vs heuristic evidence;
- trace information remains structurally separate from spec conformance; and
- reviewer provenance remains available when applicable.

This makes the CLI suitable for:

- CI pipelines;
- coding agents;
- IDE integrations;
- MCP wrappers;
- automated quality gates; and
- report generation.

---

# Exit statuses

The full CLI contract defines stable exit statuses:

| Code | Meaning |
|---:|---|
| `0` | Requested operation succeeded |
| `1` | Operation completed but policy/conformance/alignment findings make it non-successful |
| `2` | Invalid command usage or option combination |
| `3` | Input, parsing, path, or required local-artifact failure |
| `4` | Required capability or exact standards material unavailable/incompatible |
| `5` | Write not authorized, conflicted, or could not be completed safely |
| `6` | Unexpected internal failure |

This separation matters for automation.

For example:

- a malformed command is not the same as a non-conforming specification;
- a missing exact standards version is not the same as a parser crash; and
- a denied write is not the same as a failed quality check.

---

# Help and capability discovery

The specification defines:

```bash
specmd help
specmd help <topic>
specmd capabilities
```

Capability discovery should identify:

- tool version;
- supported SPEC.md Core versions;
- supported Optional versions;
- available command families;
- cognitive modes;
- adapter targets;
- interface versions;
- optional integrations; and
- unavailable capabilities with a reason.

Help must be usable without reading a project specification or requiring network access.

---

# Cucumber integration

Cucumber is deliberately modeled as an **optional external integration**.

The command family is:

```text
specmd cucumber export
specmd cucumber validate
specmd cucumber run
specmd cucumber import
specmd cucumber capabilities
```

The design separates:

```text
SPEC.md acceptance intent
```

from:

```text
implementation-specific step definitions and runtime execution
```

Only an explicitly invoked `cucumber run` operation may launch project-controlled test code.

Ordinary specification analysis must never implicitly execute it.

---

# Recommended workflows

## Author workflow

```text
1. Create or edit SPEC.md
2. Validate structure
3. Inspect quality
4. Resolve material ambiguity
5. Maintain TRACE.md
6. Run Black-Box review
7. Review acceptance coverage
8. Render for human review
```

Example:

```bash
specmd validate SPEC.md --trace auto
specmd inspect SPEC.md
specmd blackbox SPEC.md --trace auto
specmd test SPEC.md
specmd render SPEC.md
```

---

## Coding-agent workflow

```text
1. Agent reads SPEC.md
2. Agent resolves Normative Modules
3. Agent reads TRACE.md if enabled
4. Agent validates the Specification Set
5. Agent implements behavior
6. Agent updates relevant specification artifacts
7. Agent updates TRACE implementation/evidence references
8. Agent validates again
9. Agent runs Black-Box / acceptance checks
```

A coding agent must not treat its own prompt as a hidden source of requirements.

---

## CI workflow

A conservative deterministic CI check might use:

```bash
specmd --output-format json \
  validate SPEC.md \
  --trace auto \
  --cognitive off
```

A stricter policy can separately require cognitive review.

This keeps deterministic conformance and semantic review distinct.

---

# Verification and acceptance

The example specification contains explicit acceptance identifiers such as:

```text
ACC-001
ACC-002
...
```

These scenarios demonstrate expected behavior for important requirements.

Examples cover:

- initialization;
- version handling;
- trace alignment;
- large specifications;
- human-only content;
- module resolution;
- Black-Box analysis;
- machine-readable output;
- adapter generation;
- safe writes;
- offline operation; and
- cognitive/version-alignment behavior.

The verification model intentionally distinguishes:

```text
planned evidence
```

from:

```text
executed evidence
```

and:

```text
trace coverage
```

from:

```text
implementation conformance
```

---

# Current reference implementation

The main implementation repository is:

https://github.com/SPECmd-app/specmd-cli

At the time this README was prepared, its Python package identifies itself as:

```text
specmd 0.4.0
```

with:

```text
Python >= 3.9
PyYAML >= 6.0
```

and installed entry points:

```text
specmd
specmd-mcp
```

The implementation is intentionally partial and reports unavailable features rather than silently claiming them.

Consult the implementation repository's own README and `specmd capabilities` for the current availability of each command.

---

# Development of the reference implementation

Typical local setup in the implementation repository:

```bash
python3 -m venv .venv
.venv/bin/pip install -e .
.venv/bin/pip install pytest
.venv/bin/pytest
```

Example validation:

```bash
.venv/bin/specmd \
  --output-format json \
  validate SPEC.md \
  --trace auto
```

Run the MCP server:

```bash
.venv/bin/specmd-mcp
```

or:

```bash
python -m specmd.mcp_server
```

---

# SPEC.md standard

The authoritative SPEC.md standard is maintained separately:

https://github.com/SPECmd-app/SPEC.md

A specification should always be evaluated against the exact Core and Optional versions declared in its own frontmatter.

Do not assume that the latest standard is automatically the correct standard for an existing document.

---

# Repository relationship

```text
SPECmd-app/SPEC.md
    │
    │ defines the specification standard
    ▼
SPEC.md Core + Optional
    │
    │ used to specify
    ▼
SPECmd-app/specmd-cli
    │
    │ implementation + complete Specification Set
    ▼
specmd
    │
    │ worked snapshot copied into
    ▼
SPECmd-app/spec-md-examples/specmd-cli
```

The examples repository is therefore best used for:

- learning;
- reviewing specification structure;
- demonstrating traceability;
- comparing design patterns;
- testing SPEC.md-aware tools; and
- onboarding humans or coding agents to the SPEC.md model.

For implementation or full conformance evaluation, use the complete `specmd-cli` repository.

---

# Key takeaways

This example demonstrates several important SPEC.md practices:

1. **The specification is the source of truth.**
2. **Requirement IDs are stable and traceable.**
3. **TRACE.md is evidence mapping, not a second specification.**
4. **Deterministic checks and cognitive judgments remain separate.**
5. **Black-Box analysis focuses on externally observable behavior.**
6. **Coding-agent integration is thin and does not replace the specification.**
7. **Exact standards-version alignment prevents silent reinterpretation.**
8. **Human-only context is excluded from implementation-facing processing.**
9. **Writes, remote transmission, and code execution require explicit authority.**
10. **Implementation gaps are surfaced rather than fabricated away.**

---

# Related repositories

- SPEC.md standard  
  https://github.com/SPECmd-app/SPEC.md

- `specmd` CLI implementation  
  https://github.com/SPECmd-app/specmd-cli

- SPEC.md examples  
  https://github.com/SPECmd-app/spec-md-examples

---

# License

The example specification files identify the license using:

```text
SPDX-License-Identifier: Apache-2.0
```

See the relevant repository license for the complete license terms.
