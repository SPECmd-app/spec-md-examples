<!-- SPDX-License-Identifier: Apache-2.0 -->

# 枯山水 Karesansui Zen Garden — SPEC.md Example

> A SPEC.md for a **small browser game** that specifies not only the rules, but the experience: how the game is laid out on the web and on a phone, what every message says, how long things take, and how it behaves on very small screens.

This directory is part of [`SPECmd-app/spec-md-examples`](https://github.com/SPECmd-app/spec-md-examples) and demonstrates a **Core SPEC.md with a paired TRACE.md** for a single-player dry-garden (karesansui) game.

The goal of the specification is simple to state and hard to achieve:

A different person or coding agent, given only `SPEC.md`, should build a game that **plays, reads, and feels the same**.

---

## Repository location

```text
SPECmd-app/spec-md-examples/
└── karesansui-zen-garden/
    ├── README.md
    ├── SPEC.md
    └── TRACE.md
```

Current example:

- [`SPEC.md`](SPEC.md) — the authoritative specification
- [`TRACE.md`](TRACE.md) — the paired traceability document

SPEC.md standard:

- https://github.com/SPECmd-app/SPEC.md

Examples repository:

- https://github.com/SPECmd-app/spec-md-examples

---

# Why this example exists

Most specifications for small apps and games stop at the rules.

The rules are the easy part to write down: how fast a plant dries out, what a tool does, what a file contains.

The experience is usually left to taste:

- where things sit on the screen;
- what the phone version looks like;
- what the game says when you do something;
- how long a long press is;
- what happens on a very small phone.

When the experience is left to taste, two correct implementations of the same specification can feel like two different products.

This example shows the opposite approach. It keeps the **visual style free** (colors, fonts, how a stone is drawn), but makes the **experience normative**:

1. layouts for web, tablet, and phone, with a stated intent;
2. every text the player sees, word for word;
3. the timing of presses, swipes, messages, and days;
4. small-screen and touch-target rules;
5. an informative "first minute" walkthrough that conveys the intended feel.

It is also a worked example of a **Living Specification** that grew through many small releases, each recorded with a reason, and of a TRACE.md that honestly separates what was checked in a browser from what was only read in code.

---

# Specification metadata

The specification declares:

```yaml
specmd: "0.4.3"
spec_version: "1.16.0"
status: draft
name: "Karesansui Zen Garden"
optional_features:
  requirement_metadata: false
  trace: true
```

The trace declares:

```yaml
traces_file: "SPEC.md"
traces_spec: "1.16.0"
```

| Field | Meaning |
|---|---|
| `specmd` | The SPEC.md Core standard version the document follows |
| `spec_version` | The version of this game's specification |
| `status` | `draft`: the design is still evolving |
| `traces_file` / `traces_spec` | Bind TRACE.md to this exact SPEC.md and version |

The two versions are independent. `0.4.3` is the standard; `1.16.0` is the game's design.

---

# Specification authority

The specification opens with a short contract:

- `MUST`, `MUST NOT`, `SHOULD`, `SHOULD NOT`, and `MAY` follow BCP 14 (RFC 2119 + RFC 8174).
- Normative requirements, invariants, tables marked **Normative**, and the garden file format define conformance.
- Examples, notes, and rationale are informative unless marked normative.
- SPEC.md wins over implementation code or documentation; a conflict must be surfaced.
- Any omission or ambiguity that could change observable behavior must be surfaced, not silently resolved.
- Undecided product questions are marked `TBD` in Open Issues and must not be guessed.

## Implementation freedom

The design is independent of:

- programming language;
- framework;
- rendering technology (Canvas, SVG, WebGL, DOM);
- hosting;
- source organization; and
- UI styling.

The visual appearance of stones, plants, sand grooves, and weather is free, except where a requirement names a visible cue that must exist.

This is the balance the example strikes: **appearance is free, experience is specified.**

---

# Product purpose

Karesansui is a single-player, browser-based dry-garden simulation.

The Gardener:

1. lays out a rectangular sand garden with stones, lanterns, a water basin, moss, and plants;
2. rakes patterns into the sand;
3. keeps the garden healthy over simulated days and seasons: watering, pruning, pulling weeds, and clearing fallen leaves.

A **Tranquility** score from 0 to 100 summarizes the garden's condition.

The garden lives only in page memory. The Gardener keeps it by exporting a garden file and opening it again later.

The system is **one self-contained HTML document** that runs inside a sandboxed iframe. It is played with a mouse on desktop browsers and with touch on phones and tablets.

---

# Actors

## Gardener

The single human player.

The Gardener has full control over the one in-memory garden in their page.

## Host frame

The page that embeds the game in a sandboxed iframe.

It may inject a downloads shim for saving files. It is not trusted with, and not given, garden data.

There are no administrators, accounts, or other players.

---

# Scope

## In scope

- garden creation and layout;
- sand raking;
- plant care simulation;
- day, season, and weather cycle;
- Tranquility scoring;
- garden file export and import;
- garden picture export.

## Explicitly out of scope

The specification lists its non-goals:

- persistence across reloads other than exported files;
- undo or redo;
- sound;
- multiplayer, a sharing service, leaderboards, or accounts;
- migration of garden files from other formats or versions;
- browser dialogs (all confirmations are in-page);
- keyboard-only and screen-reader operation of the garden surface;
- tracking unsaved changes or warning on reload (the sandbox blocks the browser's leave-page prompt);
- a tutorial beyond the required guidance;
- grooves that deform around objects (they only must not alter sand under objects).

Each non-goal is a deliberate decision, not an oversight.

---

# Core concepts

| Term | Meaning |
|---|---|
| Garden | The whole playable state: surface, objects, sand, weeds, leaves, clock, name. Exactly one exists at a time. |
| Logical unit (u) | The garden's coordinate unit. The surface is **1200 u × 800 u**. |
| Cell | One of **60 × 40** cells of 20 u × 20 u tiling the surface. |
| Object | A placed stone, lantern, water basin, or plant. |
| Plant | Moss, azalea, maple, pine, or bamboo. |
| Footprint | The circle of an object's radius around its center. |
| Open cell | A cell whose center is outside every footprint. |
| Covered cell | A cell covered by a rake band, ripple rings, or smooth brush: **only the cell's center counts**. |
| Day / Season | Day index `d ≥ 0`, shown as "Day d+1"; season is `floor((d mod 24) / 6)`. |

The "covered cell" definition is a good example of the Core rule on material ambiguity. Whether a cell counts as raked when the rake only grazes it changes the Tranquility score, so the specification settles it explicitly.

---

# System model

## Object kinds — Table K

| Kind | Display name | Living | Footprint radius (u) | Water loss / day | Days per growth stage | Overgrowth / day |
|---|---|---|---|---|---|---|
| rock | Stone | no | S 22 · M 36 · L 54 | – | – | – |
| lantern | Lantern | no | 26 | – | – | – |
| basin | Water basin | no | 30 | – | – | – |
| moss | Moss | yes | 40 | 25 | 2 | 0 |
| shrub | Azalea | yes | 34 | 18 | 3 | 12 |
| maple | Maple | yes | 48 | 14 | 4 | 10 |
| pine | Pine | yes | 42 | 8 | 5 | 8 |
| bamboo | Bamboo | yes | 30 | 22 | 2 | 15 |

A plant's footprint is fixed for its whole life, so growth can never create an overlap.

## Plant states

Each plant has hydration, health, and overgrowth (each 0–100), a growth stage, and a withered flag.

```text
Sprout ──► Young ──► Mature
   │          │         │
   └──────────┴─────────┴──► Withered   (when health reaches 0; terminal)
```

A withered plant can only be removed.

## Daily tick

When simulated time crosses into a new day, seven steps run **in order**:

```text
1. Weather      roll rain (Spring 35%, Summer 20%, Autumn 25%) or snow (Winter 35%)
2. Hydration    lose Table K amount × season multiplier; rain refills to 100
3. Health       hydration < 25 → −15, otherwise +5; health 0 → withered
4. Growth       outside Winter, healthy plants advance toward the next stage
5. Overgrowth   outside Winter, mature plants grow shaggier
6. Weeds        spawn by season (+1 on rain), up to the cap
7. Leaves       plants drop leaves (maples most in Autumn), up to the cap
```

## Clock

Time is a real number of days.

Its fractional part is the time of day: **Dawn, Day, Dusk, Night**.

Four speeds are available:

| Speed | One day lasts |
|---|---|
| Paused | — |
| 0.5× | 90 s |
| 1× (default) | 45 s |
| 4× | 11.25 s |

## Tranquility

The score is a pure function of the current garden:

```text
Tranquility = round(35·R + 25·P + 25·C + 15·G)

R  raked      raked open cells ÷ open cells
P  plants     mean health ÷ 100 (withered counts as 0)
C  clean      max(0, 1 − 0.04 × weeds − 0.01 × leaves)
G  groomed    1 − mean overgrowth ÷ 100 of mature plants
```

Bands: **Restless** (0–39), **Settling** (40–69), **Calm** (70–89), **Still** (90–100).

P and G count as 1 when there is nothing to measure, so a classic garden of stones and sand alone can reach full Tranquility.

## Garden names — Table N

Each new game is named at random from 15 names, written kanji first, then English:

```text
苔石 Moss and Stone      短夏 Short Summer        残夏 Fading Summer
寂庭 Garden of Silence   静月 Quiet Moon          孤石 Lone Stone
石影 Stone Shadow        松影 Pine Shadow         空山 Empty Mountain
残雪 Remaining Snow      静水 Still Water         白砂 White Sand
月影 Moon Shadow         遠山 Distant Mountain    静寂 Stillness
```

---

# Critical invariants

| ID | Invariant |
|---|---|
| INV-001 | **Sandbox containment.** No host storage, no parent or top frame access, no popups, browser dialogs, fullscreen, or pointer lock. |
| INV-002 | **No overlap.** Footprints never overlap and always lie inside the surface, after every place, move, and import. |
| INV-003 | **Bounded attributes.** Plant attributes always stay within their ranges. |
| INV-004 | **Score is a pure function** of the current garden state. |
| INV-005 | **Caps.** At most 30 weeds and 200 leaves (120 leaves in the phone layout). |
| INV-006 | **Atomic import.** Opening a file replaces the whole garden or changes nothing. |
| INV-007 | **Sand under objects is smooth.** Placing or moving resets the sand under the footprint; sand tools never alter it. |

---

# Requirements overview

The specification contains **57 requirements** and **7 invariants**:

| Family | Count | Covers |
|---|---:|---|
| `FUN` | 42 | tools, care, time, displays, layouts, UX, wording, small screens |
| `DATA` | 5 | the garden, objects, weeds, leaves, memory-only state |
| `AUTH` | 2 | no identity, no sharing |
| `SEC` | 4 | sandbox profile, untrusted import, import validation, no data egress |
| `INT` | 4 | downloads shim, save fallback, garden file, picture |
| `INV` | 7 | the invariants above |

IDs are stable. The retired FUN-025 is not reused.

---

# The tools

Nine tools, one active at a time, Rake by default:

| Tool | What it does |
|---|---|
| Rake | Drag to draw 3, 5, or 7 parallel grooves; the band replaces earlier patterns and clears leaves. |
| Ripple | Click to draw 3, 5, or 7 rings; beside a stone, they ring the stone. |
| Smooth | Drag to restore flat sand with a small, medium, or large brush. |
| Place | Choose a kind (and a stone size); a preview follows the pointer; placement happens **on release**. |
| Move | Drag an object; an invalid drop returns it and says why. |
| Water | Refill a plant's hydration. |
| Prune | Trim a mature plant's overgrowth. |
| Hand | Click or drag to pick up weeds and leaves. |
| Remove | Lift an object out; the only way to clear a withered plant. |

Invalid positions are shown by a cue that is **not color alone**: a dashed ring with a slash.

---

# Screens and layouts

This is the heart of the example.

## The intent

The specification states the intent in plain words before any rule:

> On the web the whole game is one screen: everything the Gardener uses is visible at once, and nothing needs scrolling or opening to be found. A phone is too small for that, so the phone shows one view at a time and moves secondary things behind a single tap. The garden itself is never shrunk to make room for panels.

## Layout by width — Table L

| Layout | Viewport width | On screen |
|---|---|---|
| **Web** | 1061 px and wider | Everything at once, no page scrolling from 700 px tall: header, tool column, garden, and a side panel with tool options, the Tranquility card, and garden notes. |
| **Tablet** | 641–1060 px | Tools and garden first; the panel below (the page may scroll); a Tranquility badge replaces the card. |
| **Phone** | 640 px and narrower | One view at a time: the garden view or the notes view. |

## The phone, held upright

```text
┌────────────────────────────────────┐
│ 枯山水  寂庭 Garden of Silence  ◯ 75  ≡ │   title · name · badge · menu
│ Day 13  Autumn  Day  Clear          │   clock chips
├────────────────────────────────────┤
│ Rake   Ripple  Smooth  Place        │   8 tools in two rows of four
│ Water  Prune   Hand    Remove       │   (no Move on the phone)
│               ━━  ──                │   view switch
│ ┌────────────────────────────────┐ │
│ │                                │ │
│ │            garden              │ │
│ │                                │ │
│ └────────────────────────────────┘ │
│ Tap a plant to see how it is doing.│   details line
└────────────────────────────────────┘
```

- **Views.** The garden view is for playing; the notes view shows only the garden notes. Tap a segment of the view switch, or swipe on it (at least 30 px).
- **Menu.** New garden, Open garden, Save garden, Save picture, and, on the phone, the speed control.
- **Score details.** Tap the badge for a sheet with the score, band, and its four parts.
- **Tool options.** Long-press a tool (500 ms) for a sheet with its description and every option.
- **Details.** Tapping a plant shows its state; the details stay after the finger lifts.
- **Litter.** Weeds and leaves are drawn 1.5× larger, and 40% fewer leaves fall.

## The phone, held sideways

There is **no landscape layout**.

A phone held sideways sees a notice asking it to be held upright, with **Continue anyway** available at once.

Orientation is read from the device, not from the frame, because an embedding frame may not resize when the phone turns. A phone with rotation lock stays upright, so the notice never traps anyone.

TRACE.md records why: an earlier release did the opposite (asked portrait phones to rotate to landscape) and was removed after real-device reports.

## Small screens

- Down to **320 px** wide with no horizontal scrolling.
- Below 360 px the garden name moves to its own row so every name fits.
- The garden view fits without vertical scrolling from 480 px tall (520 px below 360 px wide); on shorter screens the garden shrinks, but not below 240 px wide.
- Every control has a **40 × 40 px** touch target, even when its visible mark is smaller.

---

# Wording and timing

## Wording — Table M and Table T

Every text the player sees is written down and required word for word:

- tool names and one-sentence tool descriptions;
- clock chips, speed control, menu items, panel headings;
- the details line for stones, plants, and withered plants;
- every action message, refusal, and placement reason;
- every garden note;
- every dialog's title, body, and buttons;
- all 26 file-opening error messages;
- the turn-upright notice.

A few examples:

```text
"Maple watered"
"Can't place here. It would overlap the stone."
"The azalea is thirsty. Water it."
"The garden is empty. Choose Place to set stones and plants, then Rake the sand."
"The garden is in balance. Sit with it a while."
```

Texts not listed are free, but should keep the same plain, calm tone.

## Timing — Table Z

| What | Value |
|---|---|
| One simulated day | 90 s at 0.5×, 45 s at 1×, 11.25 s at 4× |
| Score, notes, clock refresh | at least every 500 ms |
| Long press on a tool | 500 ms, cancelled if moved more than 10 px |
| Action message | about 2 s, then a 0.2 s fade |
| View-switch swipe | at least 30 px |
| Move starts after | 3 u |
| Rake sampling / smooth stamping | 7 u / 10 u |
| Frame time step | at most 0.1 s |
| Dawn and dusk | linear fades of the night shade |
| Double tap | has no meaning |

## The first minute

An informative walkthrough describes what a new player sees in their first minute, on the web and on an upright phone: an autumn garden already running, a thirsty azalea and an overgrown maple listed in the notes, the score rising as they tend it, the lantern glowing at dusk, and nothing asking them to sign in or read instructions.

The walkthrough adds no requirements. It exists so that an implementer understands the intended feel before reading the rules.

---

# Data and the garden file

## Memory only

All garden state lives in page memory. Reloading returns to the sample garden.

## The `.garden` file

"Save garden" asks for a name, then writes `<name>.garden`: UTF-8 JSON with this version 1 schema:

| Field | Content |
|---|---|
| `format` | `"karesansui-garden"` |
| `version` | `1` |
| `name` | 0–40 characters |
| `day` | simulated time |
| `weather` | `clear`, `rain`, or `snow` |
| `objects` | ≤ 200 objects with id, kind, position, size, shape seed, and plant state |
| `weeds` | ≤ 30 |
| `leaves` | ≤ 200 |
| `coverage` | 2400 characters of `0`/`1`, the raked-cell grid |
| `sand` | the 1200 × 800 sand pattern as a PNG data URI |

Opening validates everything first: size (8 MB maximum), every value's range, kinds, unique ids, overlaps, caps, and the sand image. Anything invalid is rejected with a reason and the current garden is unchanged.

---

# External interfaces

Section 5 of the specification describes each external interface by purpose, authority, operations, and failure behavior.

## Host Downloads API

```text
window.claude.use('downloads').save({ filename, data })
```

| Outcome | What the game does |
|---|---|
| saved | shows "Saved {filename}" |
| `declined` / `rate_limited` | shows "Save cancelled"; no fallback |
| any other failure, or no API | shows an in-page dialog: the file's text with Copy, or the picture |

Two known hosts are described (informative). One injects the shim and saves any filename directly. The claude.ai artifact viewer confirms each save and allows only certain extensions, which excludes `.garden`, so garden files fall back to the copy dialog there. The specification deliberately keeps `.garden` everywhere and accepts that fallback.

## Garden File, Garden Picture

The interchange format between sessions and between players, and a PNG of the garden as shown.

## Embedding Frame

The sandbox: `allow-scripts allow-forms allow-downloads`, opaque origin, one self-contained document.

## Gardener Input

Mouse, single-finger touch, and optional keyboard shortcuts (1–9 for tools, Space to pause). No action may require hover, a right click, a multi-finger gesture, or the keyboard.

---

# Security and privacy

- **SEC-001 Sandbox profile.** Works with only `allow-scripts allow-forms allow-downloads`; may load public CDN resources; depends on no other file.
- **SEC-002 Untrusted import.** Text from a file, including the garden name, is shown as plain text, never markup.
- **SEC-003 Import validation.** Strict limits on size, ranges, kinds, ids, overlaps, caps, and the sand image.
- **SEC-004 No data egress.** No garden data or input is sent over the network.

Identity is out of the picture entirely: no accounts, no sign-in, and no sharing except through files the player exports.

---

# Verification and acceptance

The specification contains **49 acceptance checks**, `V-01` to `V-49`, each citing the requirements it covers.

They use Given / When / Then:

```text
V-10 — FUN-008, FUN-018, INV-003
Given  a plant at hydration 20, health 15, on a clear Autumn day.
When   one day passes.
Then   health is 0, the plant is withered and shown withered; Remove clears it.
```

```text
V-47 — FUN-043, FUN-027, FUN-022
Given  viewports from 320 × 520 to 390 × 500, with the name "寂庭 Garden of Silence".
Then   no page scrolling, the menu at the top-right, the name fits at 12 px or larger,
       the garden and details line fully visible, and every control has a 40 × 40 touch target.
```

Each check names its method:

| Method | Meaning |
|---|---|
| T | Test |
| D | Demonstration |
| I | Inspection |

Every requirement and invariant is cited by at least one check. The only uncovered ID is OI-001, which is an open issue.

---

# Traceability

`TRACE.md` maps every requirement and invariant to:

```text
Requirement
    ↓
Design decision
    ↓
Implementation location
    ↓
Verification (V-xx)
```

It is informative. SPEC.md stays authoritative.

## Verification history

TRACE.md keeps a dated verification section for each release.

It separates, explicitly:

- checks run in a browser, with the viewport sizes and measured values;
- checks confirmed only by reading the code;
- checks not yet run.

It also records **bugs found by verification** and how they were fixed, for example:

- the phone layout hid the time-of-day and weather, which broke the clock requirement;
- dragging the Hand tool picked things up without the required message;
- long garden names were cut off at 320 px;
- several controls were smaller than the 40 × 40 touch target.

## Known gaps

TRACE.md also states what is not yet right, instead of hiding it:

- at viewport widths of about 641–1000 px (tablets, narrow desktop windows), the header wraps and the menu sits on the second row, which does not meet the menu-position requirement;
- a saved garden file is about 2.5 MB, mostly the sand image, which is within the 8 MB limit.

---

# Open Issues

## OI-001 — Detecting silent download failure

Status: `TBD`

Some hosts accept a save without error but never deliver the file. Whether the game should always also show the fallback dialog, or trust a host that reports success, is undecided.

The current requirement trusts the host. The question is left open rather than guessed.

---

# Versioning

The specification is a **draft**, and it says how it versions itself:

- while `status` is `draft`, a change that could make an older implementation non-conforming raises **MINOR** (`1.x.0`);
- additive or clarifying changes raise **PATCH**;
- Core §12's MAJOR rule applies from the first release with `status: production`.

This is a deliberate, documented departure from Core §12, recorded in the specification itself.

The Notes and Rationale section keeps a one-line reason for every release. A few milestones:

| Version | Change |
|---|---|
| 1.2.0 | Mouse and touch required; placing happens on release |
| 1.3.0 | Garden files end in `.garden`; saving asks for a name |
| 1.4.0 | Random kanji-and-English garden names |
| 1.8.0 | A rotate-to-landscape prompt removed after real-device reports |
| 1.9.0 | Phone toolbar of two rows of four; long press for tool options |
| 1.12.0 | Phones held sideways get a turn-upright notice instead of a layout |
| 1.14.0 | The UX specified: layouts, phone views, menu, feedback, affordances |
| 1.15.0 | Wording, timing, and small-screen rules made normative |
| 1.16.0 | A 0.5× speed |

---

# Validating this example

Both files validate with [`specmd-cli`](https://github.com/SPECmd-app/specmd-cli):

```bash
specmd validate SPEC.md --trace TRACE.md
specmd inspect SPEC.md --trace TRACE.md
specmd trace update SPEC.md --trace TRACE.md
specmd blackbox SPEC.md --trace TRACE.md
specmd test SPEC.md --trace TRACE.md
```

TRACE.md records the results with specmd-cli 0.4.0:

- `validate` succeeded with no errors or warnings;
- `trace update` found nothing to add or flag;
- `blackbox` found five interface elements (Host Downloads API, Garden File, Garden Picture, Embedding Frame, Gardener Input), all covered by the Host-Agent input;
- `test` covered every ID except the open issue OI-001.

---

# Using this example

## To learn from it

Read it in this order:

1. **§3.7 Screens and layouts**, for the web-versus-phone intent;
2. **§3.8 First minute**, for the intended feel;
3. **§3.6 Tranquility** and **§3.4 Daily tick**, for the rules;
4. **§4 Requirements**, for the details;
5. **§7 Verification**, for how each rule is checked;
6. **TRACE.md**, for how an implementation was held to it.

## To implement from it

```text
1. Read SPEC.md completely, including Tables K, L, M, N, T, and Z
2. Treat SPEC.md as authoritative; appearance is yours to choose
3. Build the rules, then the layouts, then the wording and timing
4. Use the acceptance checks V-01 to V-49 as your test plan
5. Keep a TRACE.md of your own, and record what you actually checked
6. Surface anything ambiguous instead of guessing
```

---

# Key takeaways

This example demonstrates several SPEC.md practices:

1. **Appearance can be free while experience is specified.**
2. **Intent comes before rules**: the web-versus-phone intent is stated in plain words, then made precise.
3. **Wording and timing can be normative tables**, which makes independent implementations read and feel alike.
4. **Material ambiguities are settled in the spec**, such as when a cell counts as raked.
5. **Non-goals are decisions**, written down so they are not mistaken for gaps.
6. **Open questions stay `TBD`** rather than being invented.
7. **A Living Specification records why it changed**, release by release.
8. **TRACE.md separates evidence from belief**: browser checks, code reading, and not-yet-run are kept apart.
9. **Known gaps are published**, not hidden.

---

# Related repositories

- SPEC.md standard  
  https://github.com/SPECmd-app/SPEC.md

- `specmd` CLI  
  https://github.com/SPECmd-app/specmd-cli

- SPEC.md examples  
  https://github.com/SPECmd-app/spec-md-examples

---

# License

The example files identify their license with:

```text
SPDX-License-Identifier: Apache-2.0
```

See the repository [LICENSE](../LICENSE) for the complete terms.
