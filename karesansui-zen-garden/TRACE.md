---
traces_file: "SPEC.md"
traces_spec: "1.16.0"
last_updated: "2026-09-26"
---

<!-- SPDX-License-Identifier: Apache-2.0 -->

# TRACE — Karesansui Zen Garden

Retrospective traceability from SPEC.md 1.16.0 to the implementation in `Karesansui.html` (one self-contained file). Informative; SPEC.md is authoritative. The implementation declares the same version in `<meta name="spec-version">` and `SPEC_VERSION`, and each publish of the game is labelled "Spec <version>"; all four (SPEC.md, this file, the game, the publish label) are updated together.

There is no backend, database, or row-level security: the host sandbox forbids storage (INV-001), so every "entity" is an in-memory JavaScript structure. Design decisions below name those structures. Locations name the function or block in `Karesansui.html`. Verification IDs (V-xx) refer to SPEC.md §7.

## Invariants

| ID | Design decision | Implementation | Verification |
|---|---|---|---|
| INV-001 | No storage, frame, popup, or modal APIs used. Confirmations are the in-page `#modal`. Files go through the shim or an in-page dialog. | whole file; `openModal`, `trySave` | V-18 (grep for forbidden APIs returns nothing) |
| INV-002 | Circular footprints; a single validity check is used for place, move, and import. | `footprintValid`; `importGarden` overlap loop; `endDrag` | V-05, V-06, V-17 |
| INV-003 | Every write clamps to range; import rejects out-of-range values. | `dailyTick` (`clamp`), Water/Prune handlers, `importGarden` | V-10, V-17 |
| INV-004 | Score computed from current state each refresh; never stored. | `tranquility`, `bandOf` | V-12 |
| INV-005 | `WEED_CAP` 30 and `LEAF_CAP` 200 checked before each spawn (leaves: `leafCap()`, 120 in portrait); import rejects larger arrays. | `dailyTick`, `importGarden` | V-11, V-17 |
| INV-006 | Validate everything and decode the sand image first; only then replace `G`, `cov`, and the sand raster. | `importGarden` (commit block at end) | V-17 |
| INV-007 | Sand edits are clipped with an even-odd path that excludes every footprint; placing or moving calls `smoothUnder`. | `clipOpen`, `smoothUnder`, `rakeTo`, `ripple`, `smoothAt` | V-03 |

## Functional

| ID | Design decision | Implementation | Verification |
|---|---|---|---|
| FUN-001 | 1200 × 800 logical surface drawn with a single uniform scale × devicePixelRatio; mouse and touch mapped via the canvas bounding box; `touch-action: none` and no text selection or callout on the canvas. | `resize`, `toLogical`, `render`, `canvas#garden` CSS | V-22, V-47 |
| FUN-002 | `TOOLS` table drives the toolbar; `aria-pressed` marks the active tool; default `rake`. | `TOOLS`, `setTool` | V-20 |
| FUN-003 | Rake stroke keeps its last vertex and smoothed normal so band segments join; the band is cleared to base sand, then tines are drawn. A cell counts as covered when its center is within half the band width of the stroke segment (`distToSeg`). | `rakeStart`, `rakeTo`, `groove`, `markCells` | V-02, V-45 |
| FUN-004 | Nearest object within 1.6 × radius becomes the ring center; covered cells are those whose centers lie in the ring annulus. | `ripple`, `markCells` | V-03, V-45 |
| FUN-005 | Brush stamped every 10 u along the drag; covered cells are those whose centers lie within the brush radius. | `smoothAt`, pointermove handler, `markCells` | V-04, V-45 |
| FUN-006 | Press starts a place drag; the ghost follows the pointer while held (and on mouse hover); placement is attempted on `pointerup` only (cancel places nothing). Invalid: dashed red ring and a slash; the reason is shown in a toast. | `render` (place preview), `ringCue`, pointerdown `place`, `placeAt`, `endDrag` | V-05, V-23 |
| FUN-007 | Drag state keeps the original position; revert when invalid or not moved, with the reason as a message; while dragging, the object is drawn at the pointer with `ringCue`. | pointerdown `move`, `endDrag` | V-06, V-43, V-48 |
| FUN-008 | Removes the object from `G.objects`. | pointerdown `remove` | V-10 |
| FUN-009 | `maxlength=40` input; blank on blur becomes "Untitled garden". | `#garden-name` listeners | V-33, V-16 |
| FUN-010 | Sets hydration to 100 for non-withered plants only. | pointerdown `water` | V-07 |
| FUN-011 | Stage-2 check; message otherwise. | pointerdown `prune` | V-08 |
| FUN-012 | Removes weeds and leaves within `HAND_R` on click and along a drag; the count is gathered in `ui.drag.picked` and reported once on release (FUN-038). | pointerdown/pointermove `hand`, `endDrag` | V-09, V-43 |
| FUN-013 | Speeds 0, 0.5, 1, 4 as `data-speed` buttons in the header `.seg` and the phone `.menu-speed`; `ui.speed` starts at 1 and `newGarden`/`importGarden` leave it alone; `setSpeed` marks every matching button and remembers the last running speed for Space. rAF loop adds `dt × speed / 45 s`; ticks each crossed day; skips while `document.hidden`. | `frame`, `dailyTick`, `setSpeed` | V-10, V-13, V-49 |
| FUN-014 | Chips for day, season, phase, and weather, shown in every layout (the phone layout had hidden phase and weather; fixed in 1.13.3). | `refreshPanel`, `phaseOf`, `.clock` | V-01, V-34 |
| FUN-015 | Panel refreshed every 250 ms with score, band, ensō arc, and R/P/C/G bars. Below 1060 px, `#tranq-card` is hidden and a compact `#tranq-badge` (score + band only) sits above the canvas instead; tapping it rebuilds the same score/ensō/bars markup on demand inside the existing modal, so the four components stay reachable in one tap rather than always on screen. | `refreshPanel`, `tranquility`, `#tranq-badge` click handler | V-02, V-12 |
| FUN-016 | Notes derived from state; an empty-garden note comes first when there are no objects; ordered withered → thirsty → overgrown → weeds → leaves → raking. | `refreshPanel` (notes block) | V-01, V-40 |
| FUN-017 | Details line under the garden, built with text nodes and refreshed with the panel so values stay live. Mouse: follows hover, clears on leave. Touch: set on press over an object and kept after lift and over empty sand. | `updateHover`, `showInspect`, `refreshPanel`, `pointerleave` handler | V-07, V-23, V-24 |
| FUN-018 | Dry tint below 25 % water, brown and dashed ring when withered, shag strokes for overgrowth, seasonal maple palette and bare winter branches, night overlay with additive lantern glow. | `plantTint`, `drawShag`, `drawObject`, `render` (night block), `darkness` | V-10, V-19 |
| FUN-019 | Named by `randomName()` (FUN-024). Sample built procedurally: stones, moss, lantern, basin, four plants, raked rows over the upper garden, two ripples, maple leaves, four weeds; day 12.3. Azalea starts thirsty, maple overgrown. | `sampleGarden` | V-01 |
| FUN-020 | Confirmation sheet with Keep gardening / Start new garden; new name from `randomName(current)`, shown in the toast. | `#btn-new` handler, `newGarden` | V-21 |
| FUN-021 | Global keydown; ignored in inputs and textareas and while a dialog is open. Every shortcut duplicates an on-screen control. | `keydown` listener | V-13, V-20, V-49 |
| FUN-022 | One Pointer Events path for mouse, pen, and touch; no hover-only, right-click, or multi-touch actions. Pointer capture is attempted but optional. `(hover: none)` CSS hides key badges and hints and gives controls 40 px+ targets, including the file menu's items (`.menu-item{min-height:44px}`); tool tooltips omit keys when there is no hover. After a touch lifts, previews are hidden. | pointer handlers, `endDrag`, `canHover`, `@media (hover:none)` CSS Every control has a 40 × 40 CSS px touch target: 52 px tools, 40 px option and menu buttons, a 40 px badge, a 40 × 40 area around each view-switch segment (`.mode-dot::before`), and `a.btn` links shown as flex so they keep their height. | V-23, V-24, V-47 |
| FUN-023 | New garden, Open garden, Save garden, and Save picture (FUN-020, INT-003, INT-004) are grouped behind a single "Garden menu" hamburger button rather than shown as separate top-level buttons; opening it reveals a `role="menu"` list, closed by choosing an item, clicking outside, or Escape. Save garden itself opens a `<form>` sheet with a prefilled, selected name field and a live "Saves as …" preview. Submit (button or Enter) sets `G.name` and the header field, closes the sheet, then calls `saveGarden`. Cancel closes without saving. | `#btn-menu`/`#menu-list` (`closeFileMenu`, `openFileMenu`), `#btn-save` handler, `saveGarden` | V-14, V-25 |
| FUN-024 | `GARDEN_NAMES` holds Table N as `"<kanji> <English>"` strings; `randomName(avoid)` picks uniformly from the entries other than `avoid`. Used on load and on New garden. | `GARDEN_NAMES`, `randomName`, `sampleGarden`, `newGarden` | V-01, V-21, V-26 |
| FUN-027 | Phone media query `(max-width:640px)`: `#tools` becomes a 4-column grid with `[data-tool="move"]` hidden; `.bar` becomes a 3-column grid (title / name / `.top-right`) with the chips on row 2 and `.files` hidden; `.mode-row` (a second `.mode-switch`, `#mode-switch-garden`) is shown centered above `.garden-col` and stays shown in `notes-mode`. `#tranq-badge-band` is hidden at ≤ 400 px. `portraitPhoneMQ` switches Move to Rake. Both switches share `.mode-dot` state and the swipe handler. | portrait CSS blocks, `.top-right`, `.mode-row`, `setMobileMode`, `.mode-switch` swipe handler, `portraitPhoneMQ`, `leaveMoveInPortrait` | V-28, V-47 |
| FUN-028 | `attachLongPress` fires `openToolSheet` after 500 ms unless moved > 10 px; the click that follows is swallowed and `contextmenu` is prevented. The sheet reuses `buildToolOptions`, and `renderToolOptions` refreshes both the panel and an open sheet. Touch placeholder mentions long press. | `attachLongPress`, `openToolSheet`, `buildToolOptions`, `renderToolOptions`, `showInspect` | V-29, V-48 |
| FUN-029 | The badge and menu live in `.top-right`, the last header child (`margin-left:auto` on desktop). The portrait header is a grid ending in `.top-right`. `fitName` steps the name's font size down by 0.5 px to 12 px until it fits; it runs on input, resize, name changes, and font load. The brand shows only `.kanji` 枯山水; the Latin span was removed (`aria-label="Karesansui"` keeps the reading for screen readers). | `.top-right`, portrait header CSS, `fitName` | V-30 |
| FUN-030 | `portraitPhoneMQ` is defined with the constants so it is ready before the starting garden is built. `render` passes `LITTER_SCALE_PORTRAIT` (1.5) to `drawLeaf`/`drawWeed`, which scale about each item's own position. `scaleLeafCount` multiplies counts by `LEAF_FACTOR_PORTRAIT` (0.6) with the fraction as a random extra leaf; `leafCap()` returns 120 in portrait. Used by `dailyTick` step 7 and `sampleGarden`. | `portraitPhoneMQ`, `isPortraitPhone`, `scaleLeafCount`, `leafCap`, `drawLeaf`, `drawWeed`, `render`, `dailyTick`, `sampleGarden` | V-31 |
| FUN-031 | No layout rules target phones held sideways; `phoneMQ` and `portraitPhoneMQ` are `(max-width:640px)` and `sideStretchMQ` is `(min-width:1061px)`. `#rotate-note` is a fixed full-screen notice. `updateRotateNote` shows it when `isPhoneDevice()` (coarse pointer, shorter screen side ≤ 500) and `isLandscapeDevice()` (`screen.orientation.type`, then `window.orientation`, then viewport shape) are both true and it has not been dismissed. It runs on `screen.orientation` change, `orientationchange`, `resize`, and load. Continue anyway sets `rotateDismissed`, which is cleared when the phone is upright. | `#rotate-note`, `isPhoneDevice`, `isLandscapeDevice`, `updateRotateNote`, `#rotate-continue` | V-32 |
| FUN-032 | Table L breakpoints: `sideStretchMQ` `(min-width:1061px)` for Web (side columns pinned to the garden frame's height; the canvas is capped at the viewport height minus 170 px, so the page never scrolls); `@media (max-width:1060px)` for Tablet; `phoneMQ` `(max-width:640px)` for Phone. | layout CSS, `resize`, `syncSideHeights`, `syncStageHeight` | V-35 |
| FUN-033 | `.stage.notes-mode` hides tools and garden and shows only `.notes-card`; two `.mode-switch` controls share `.mode-dot` state; a swipe of 30 px or more on either switch calls `setMobileMode`. | `setMobileMode`, `.mode-switch` swipe handler, phone CSS | V-36, V-48 |
| FUN-034 | `#menu-list` holds `.menu-speed` (shown only at ≤ 640 px) and four `.menu-item`s in order; it closes on item click, outside click, and Escape. | `openFileMenu`, `closeFileMenu`, menu listeners, `keydown` | V-37 |
| FUN-035 | `#tranq-badge` is shown at ≤ 1060 px and `#tranq-card` is hidden; clicking the badge opens a sheet built from `tranquility()`. | badge CSS, `#tranq-badge` handler | V-38 |
| FUN-036 | `#tool-card` in `.panel` renders `buildToolOptions(box, true)` for the active tool (name, description, options); `renderToolOptions` keeps the panel and any open sheet in step. | `renderToolOptions`, `buildToolOptions` | V-39 |
| FUN-037 | `TOOLS` holds each description (Table T, normative wording); `showInspect(null)` shows the hover or touch hint; empty-garden note in `refreshPanel`. | `TOOLS`, `showInspect`, `refreshPanel` | V-40 |
| FUN-038 | `toast()` shows a non-blocking message for 1.9 s; called by place, move, water, prune, Hand (on release), remove, new, open, save, picture, and cancel paths; not by rake, ripple, or smooth. | `toast` and its callers | V-41, V-48 |
| FUN-039 | New garden uses a confirmation sheet; the Open garden sheet says "The current garden will be replaced." | `#btn-new`, `#btn-open` handlers | V-42 |
| FUN-040 | The `frame` loop advances time whenever the speed is above 0 and the page is visible; dialogs do not touch it. | `frame` | V-42 |
| FUN-041 | `render` outlines `ui.hover` for water, prune, remove, and move (red for remove); `keydown` closes the modal or menu on Escape. | `render`, `updateHover`, `keydown` | V-43 |
| FUN-042 | `reduceMotion` freezes rain and snow particles and lantern flicker; `@media (prefers-reduced-motion)` removes toast and bar transitions. | `reduceMotion`, `render`, reduced-motion CSS | V-44 |
| FUN-043 | `@media (max-width:359px)` moves `.name` to its own grid row. On phones `resize` caps the garden width so header, tools, switch, garden, and details line fit the height, never below 240 px. The garden is re-fitted when the header changes height (`ResizeObserver` on `.bar`), when fonts load, and by `checkFit` on the 250 ms refresh when the window size changed or the page overflows. Dialog sheets are capped at the viewport and scroll inside. | small-screen CSS, `resize`, `checkFit`, `.bar` observer | V-47 |
| FUN-044 | All texts in Table M and Table T come from string literals in the page and `TOOLS`; nothing is generated. | string literals, `TOOLS`, `importGarden` messages | V-46 |

### FUN-013 daily tick, step by step (informative, expands SPEC §3.4)

`dailyTick(d)` runs once for every simulated day crossed, applying SPEC §3.4's seven steps in order:

1. **Weather.** Roll the day's weather against season odds (Spring rain 35 %, Summer rain 20 %, Autumn rain 25 %, Winter snow 35 %, else clear), stored in `G.weather`.
2. **Hydration.** For each non-withered plant, subtract its Table K `decay` × season multiplier (`mult`: Summer 1.5, Winter 0.5, else 1); a rainy day sets hydration to 100 instead.
3. **Health.** Hydration below 25 costs health −15 that day, otherwise +5, clamped 0–100; reaching 0 sets `withered = true`.
4. **Growth.** Outside Winter, a healthy (health ≥ 50) plant below stage 2 advances `stageDays`; reaching its Table K `grow` value advances `stage` and resets `stageDays` to 0.
5. **Overgrowth.** Outside Winter, a stage-2 plant's `overgrowth` grows by its Table K `over` rate, clamped to 100.
6. **Weeds.** Spawns `[3, 2, 1, 0][season]` weeds, plus 1 more if it's raining; each lands at a random point at least 10 u inside the edge and outside every non-moss footprint, capped at `WEED_CAP` (30).
7. **Leaves.** Autumn drops 6 (maple) or 1 (azalea, pine, bamboo) per eligible plant; Spring and Summer drop 1 for maple only; Winter drops none; each leaf lands within 1.6 × the plant's footprint radius, capped at `LEAF_CAP` (200).

Winter (`season === 3`) is the one season that skips steps 4 and 5 entirely — plants can still dry out and wither, but nothing matures or gets shaggy until Spring returns.

## Data

| ID | Design decision | Implementation | Verification |
|---|---|---|---|
| DATA-001 | `G` = `{ name, day, weather, objects, weeds, leaves, nextId }`, plus `cov` (Uint8Array 2400) and `sandCanvas` raster. | `blankGarden`, globals | V-16 |
| DATA-002 | Object `{ id, kind, x, y, seed, size?, hydration?, health?, stage?, stageDays?, overgrowth?, withered? }`; seeds drive cached shapes. | `makeObject`, `shapeOf` | V-16 |
| DATA-003 | Weed `{ x, y, seed }`. | `dailyTick`, `drawWeed` | V-16 |
| DATA-004 | Leaf `{ x, y, rot, c }`, colors from `LEAF_COLORS`. | `dailyTick`, `drawLeaf` | V-16 |
| DATA-005 | No persistence code; load always runs `sampleGarden`. | bootstrap at end of script | V-18 |

## Identity and access

| ID | Design decision | Implementation | Verification |
|---|---|---|---|
| AUTH-001 | No sign-in UI or identity data exists. | whole file | V-18 |
| AUTH-002 | No network or sharing code; the only output is Gardener-initiated files. | `trySave`, save/picture handlers | V-18 |

## Security

| ID | Design decision | Implementation | Verification |
|---|---|---|---|
| SEC-001 | One HTML document; only external loads are Google Fonts, with system fallbacks. | `<link>` in head; font stacks in `:root` | V-18 |
| SEC-002 | Imported strings are written only through `textContent`, input `value`, and text nodes. | `importGarden`, `showInspect`, `refreshPanel` | V-17 |
| SEC-003 | Size check (file and text), schema and range checks, overlap and bounds, caps, PNG prefix, and decoded 1200 × 800 check. | `importGarden`, file input handler | V-17 |
| SEC-004 | No `fetch`, XHR, WebSocket, or beacon. | whole file | V-18 |

## Interfaces

| ID | Design decision | Implementation | Verification |
|---|---|---|---|
| INT-001 | `getDownloads` resolves `claude.use('downloads')` once, shortly after load, and caches it; it accepts a direct or promised value, and `null` means unavailable. `trySave` returns `saved`, `cancelled` (`declined`/`rate_limited`: toast only), or `failed`. On claude.ai the published artifact declares the `downloads` capability. | `getDownloads`, `trySave`, save/picture handlers; Artifact publish `capabilities: {downloads: true}` | V-14, V-27 |
| INT-002 | Shown only on `failed` (including claude.ai's `rejected_extension` for `.garden`). Garden: read-only textarea, Copy (Clipboard API, falls back to select), Blob-URL "Try download". Picture: image preview and data-URL "Try download". | `#btn-save` and `#btn-picture` handlers | V-15 |
| INT-003 | `serialize` writes the v1 schema to `<file-name>.garden`; `fileNameOf` replaces reserved and control characters and trims spaces and dots. `importGarden` reads a chosen file (picker offers `.garden` and `.json`) or pasted text, validating content, not extension. | `serialize`, `fileNameOf`, `saveGarden`, `#btn-open` handler, `importGarden` | V-14, V-16, V-17 |
| INT-004 | `canvas.toBlob` of the rendered view, named `fileNameOf(G.name) + '.png'`. | `#btn-picture` handler, `fileNameOf` | V-15 |

## Open issues

| ID | Design | Implementation | Verification |
|---|---|---|---|
| OI-001 | TBD in SPEC.md (silent download failure). Current behavior follows INT-001: a save the shim reports as successful is trusted, and no fallback dialog is shown. | `trySave`, `saveGarden`, `#btn-picture` handler | None until decided |

## Verification status (2026-09-26, spec 1.16.0)

- V-49 checked at 1280 × 800 and 375 × 812. 1× is active on load; the header reads Pause, 0.5×, 1×, 4×. Choosing 0.5× marks it in both the header and the phone menu. Space paused, then resumed at 0.5×. New garden kept 0.5×. The Web layout still fits one screen at 1280 × 800 and 1100 × 700 with the menu at the top-right. The phone menu's speed buttons are 55, 49, 44, and 44 px wide by 40 px tall. The half pace (90 s per day) was checked in the code only (`G.day += dt × speed / 45`), not timed.

## Verification status (2026-09-26, spec 1.15.0)

- V-47 checked. With "寂庭 Garden of Silence" at 320 × 568 the name is on its own row at 16 px; the menu is top-right; there is no horizontal scroll; the four chips share one row. The garden and details line fit without scrolling at 320 × 520 (garden 254 px), 320 × 568, 360 × 480 (251 px), 375 × 667, and 390 × 500. At 320 × 480 the garden stops at 240 px and the page scrolls 26 px, which the rule allows. The badge is 40 px tall; a tap 12 px above a view-switch segment still hits it; the "Try download" link is 40 px tall.
- Found and fixed while checking: the long names were cut at 320 px (now on their own row); the view-switch segments (22 × 6 px), the badge (31 px), and the "Try download" link were below 40 px; short screens scrolled; the garden sometimes kept a stale size after the header reflowed.
- V-46: the Table M and Table T texts were taken from the page's string literals, so they match by construction. They have not been checked on screen one by one.
- V-48: the timing values were checked against the code (500 ms and 10 px long press, 1.9 s plus 0.2 s messages, 30 px swipe, 3 u move threshold, 7 u rake sampling, 10 u smooth stamps, 0.1 s frame cap). Not timed in the browser.

## Verification status (2026-09-26, spec 1.14.0)

- V-35 checked: at 1280 × 800, 1440 × 900, and 1100 × 700 the page height equals the viewport (no scrolling), with header, tools, garden, options, Tranquility card, notes, speed, and menu all visible. At 1100 × 700 the notes list scrolls inside its card.
- V-36 checked at 375 × 812: the garden view shows tools, switch, and garden; the notes view shows only the notes and the switch, with notes marked active; a rightward swipe returned to the garden view; the day was unchanged.
- V-37 checked: the menu lists New, Open, Save garden, Save picture in that order. The speed control is in the header on the Web layout and at the top of the menu on the Phone layout. The menu closes on an outside click and on Escape.
- V-38 checked at 900 × 800: the badge is shown and the card hidden; the badge's sheet shows the score and Raked sand, Plant health, Clean ground, Groomed.
- V-39 checked at 1280 × 800: Ripple's name, description, and 3/5/7 are shown in the panel; choosing 7 took effect.
- V-40 checked: after New garden the first note reads "The garden is empty. Choose Place to set stones and plants, then Rake the sand."; the details line shows the hover hint.
- V-41 checked: "Azalea watered" appeared and was gone after 2.3 s; "A fresh bed of sand: …" after New garden; raking produced no message. A Hand drag first produced no message (a bug); fixed so the drag reports "Picked 2 items" on release.
- V-42 checked: the Open dialog says "The current garden will be replaced." Time running during dialogs was checked by reading the code (`frame` ignores dialogs), not measured.
- V-43 partly checked: Escape closes dialogs and the menu; the Hand drag removes weeds along its path. The outlines and Move's cue were checked by reading the code only.
- V-44 and V-45: checked by reading the code only (`reduceMotion` branches; `markCells` uses cell centers). A browser test needs forced reduced motion and access to the raked-cell grid.

## Verification status (2026-09-26, spec 1.13.3, specmd-cli 0.4.0)

- `specmd validate`: succeeded (0 errors, 0 warnings).
- `specmd inspect`: succeeded; one information note (no Cognitive Provider configured, so the semantic analysis was not run).
- `specmd trace update`: succeeded; nothing added or flagged, and content unchanged.
- `specmd blackbox --cognitive required` with Host-Agent input: succeeded. There are 5 interface elements, all covered, and no rejected citations. FUN-019, FUN-024, and AUTH-002 are internal behavior with no external interface.
- `specmd test`: 51 of 52 IDs covered; OI-001 is uncovered because it is an open issue (TBD).
- V-33 checked: a blank name becomes "Untitled garden" when the field loses focus, and the field keeps 40 characters at most. V-34 checked: all four clock chips are shown at 375 px on one row after the 1.13.3 fix.

## Verification status (2026-09-26, spec 1.13.0–1.13.1)

- 1.13.1 changed only informative text (the §5 host table), so the 1.13.0 results below still apply.

- Phone layout rechecked at 375 × 812 after the media query was simplified to `(max-width:640px)`: 8 tools in 4 columns without Move, swipe row shown, menu top-right, no horizontal scroll, no console errors.

## Verification status (2026-09-26, spec 1.12.0)

- V-32 checked with a stubbed phone (coarse pointer, 390 × 844 screen, switchable `screen.orientation`): shown on sideways load, hidden after Continue anyway, stayed hidden while still sideways, hidden upright, shown again when turned sideways. No console errors.
- After Continue anyway at 844 × 390: the regular layout under 1060 px with all 9 tools and no horizontal scroll; the menu wraps to the second header row (allowed by the FUN-029 exception).
- Not yet tried on a physical phone or inside an embedding host.
- **Known conformance gap (FUN-029):** at viewport widths of about 641–1000 px that are not a sideways phone (tablets, narrow desktop windows), the header wraps and the menu sits on the second row. Seen at 696 × 574 in the 1.10.0 checks. Not fixed yet.

## Verification status (2026-09-26, spec 1.11.0)

- V-31: with the portrait check forced true from load, the starting garden had 13 leaves; at desktop-layout size it had 20–23. Weeds and leaves render through a 1.5× transform in portrait (checked by inspection). Daily drop averages were not measured.

## Verification status (2026-09-26, spec 1.10.0)

- V-30 checked with the name "寂庭 Garden of Silence" at 375 × 812 (14 px, badge without band word), 412 × 915 (14 px), and 1280 × 800 (18 px). The menu was at the right end of the first header row every time, with no horizontal scroll.
- V-28 checked at 375 × 812: the view switch is centered above the garden; the notes view keeps it visible.

## Verification status (2026-09-26, spec 1.9.0)

- V-28 checked at 375 × 812: rake/ripple/smooth/place on row 1, water/prune/hand/remove on row 2, no Move, no horizontal scroll, view switch and menu right-aligned.
- V-29 checked with synthetic touch long-press: the Ripple sheet opened and 7 rings was selected; in the Place sheet Stone and Large were selected (8 kinds shown); the active tool was set without a stray click.
- Not yet tried on a physical phone.

## Verification status (2026-09-25 to 2026-09-26, spec 1.0.0–1.8.0)

Checked in the Claude desktop browser pane:

- Checked: V-27 and claude.ai behavior with stub shims (declined: no dialog, "Save cancelled"; allowlist stub: `.png` saved, `.garden` falls back to the dialog), V-26 (20 New gardens: 10 distinct names, no repeat of the previous), V-21 (naming), V-14 (stub shim got `Stones- Autumn-.garden` and a JSON string), V-25, V-01, V-02, V-07, V-15 (garden dialog), V-16 (round-trip import), V-17 (overlap rejected; markup name shown literally), V-18 (source scan), V-22 (375 px, no horizontal scroll).
- V-23, partly: synthetic touch events at 375 px. Place on release, invalid-place message, tap to water, touch raking, details kept after lift, shortcut hints hidden, 40 px segment buttons. Not yet tried on a physical phone.
- V-24, partly: mouse raking, watering, and buttons; details clear when the mouse leaves.
- Not yet run: V-03–V-06, V-08–V-13, V-19–V-21. V-10 and V-11 need a scripted harness with a stub shim and forced state.

### 2026-09-26 update

- Checked FUN-015's narrow-width behavior: at widths up to 1060 px `#tranq-card` is hidden and `#tranq-badge` is shown; tapping the badge opens a sheet with the same score, band, and R/P/C/G bars. At 1400 × 900 the badge stays hidden and the card shows.
- The rotate-to-landscape prompt (FUN-025, 1.6.0–1.7.0) was removed in 1.8.0 after field reports from a real phone in an embedding host: the frame did not resize on rotation, and rotation lock kept phones in portrait. Its checks no longer apply.

Known gap: a saved garden file is about 2.5 MB, mostly the sand PNG, because the per-pixel sand grain compresses poorly. It is within SEC-003's 8 MB limit.
