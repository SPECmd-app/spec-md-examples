<!-- SPDX-License-Identifier: Apache-2.0 -->

# 枯山水 Karesansui Zen Garden — SPEC.md Example

> A SPEC.md for a **small browser game** that pins down not only the rules but the experience: layouts for web and phone, exact wording, timing, and small-screen behavior, so that a different agent building from the spec produces a game that plays and reads the same.

This directory is part of [`SPECmd-app/spec-md-examples`](https://github.com/SPECmd-app/spec-md-examples). It holds the specification (spec version 1.16.0, SPEC.md Core 0.4.3) and its paired TRACE.md for Karesansui, a single-player zen garden: rake the sand, set stones and plants, and tend the garden through the seasons to keep it tranquil.

---

## What this example demonstrates

- **Exact simulation rules** as normative tables and formulas: object sizes and growth rates (Table K), an ordered daily tick, a clock, and a Tranquility score defined as a formula over the current state.
- **A portable file format:** the version 1 `.garden` JSON schema with its validation rules and atomic import.
- **A sandboxed host contract:** the game runs in an iframe with only `allow-scripts allow-forms allow-downloads`, saves through the host's downloads API, and falls back to an in-page dialog. Section 5 describes each external interface (purpose, authority, operations, failure behavior).
- **UX as requirements:** the intent for web (the whole game on one screen) versus phone (one view at a time) with a layout table (Table L), phone views and gestures, the menu, guidance, action feedback, pointer affordances, and reduced motion.
- **Wording and timing as normative tables:** every text the player sees (Table M, Table T) and every duration and distance that shapes the feel (Table Z).
- **Small-screen rules** down to 320 CSS px wide, including touch targets and how the garden shrinks on short screens.
- **An informative first-minute walkthrough** for web and phone, next to the normative requirements.
- **An open issue left as `TBD`** (OI-001, silent download failure) instead of a guessed behavior.
- **Honest verification:** 49 acceptance checks in Given/When/Then form; the TRACE.md records which were checked in a browser, which only by reading code, and conformance gaps found along the way.
- **A documented versioning policy** for a draft: breaking changes raise MINOR while `status` is `draft`, an explicit, noted departure from Core §12.

---

## Files in this example

```text
karesansui-zen-garden/
├── README.md
├── SPEC.md
└── TRACE.md
```

| File | Purpose |
|---|---|
| [`SPEC.md`](SPEC.md) | The authoritative specification: 57 requirements (FUN, DATA, AUTH, SEC, INT), 7 invariants, 1 open issue, 49 acceptance checks. |
| [`TRACE.md`](TRACE.md) | Maps every requirement and invariant to its design decision, its location in the implementation, and its verification, with a dated verification history. |

This is a snapshot copy. The implementation (`Karesansui.html`, a single self-contained HTML file) is not included here, so the function names in TRACE.md's Implementation column refer to code outside this repository.

---

## Validation

Both files validate with [`specmd-cli`](https://github.com/SPECmd-app/specmd-cli) 0.4.0:

```bash
specmd validate SPEC.md --trace TRACE.md   # succeeded
specmd test SPEC.md --trace TRACE.md       # 64 of 65 IDs covered; OI-001 is open (TBD)
```

`specmd blackbox` finds five external interfaces (Host Downloads API, Garden File, Garden Picture, Embedding Frame, Gardener Input).
