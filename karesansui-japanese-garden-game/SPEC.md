---
specmd: "0.4.3"
spec_version: "1.16.0"
status: draft
name: "Karesansui Zen Garden"
last_updated: "2026-09-26"
optional_features:
  requirement_metadata: false
  trace: true
changes:
  modified:
    - FUN-013
    - FUN-034
    - "§3.5 Clock, Table Z, Table M"
  added:
    - V-49
---

<!-- SPDX-License-Identifier: Apache-2.0 -->

# Karesansui Zen Garden

## Specification Contract

- Uppercase MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY use BCP 14 semantics (RFC 2119 + RFC 8174).
- Normative requirements, invariants, tables marked **Normative**, and the garden file format define conformance. Examples, notes, and rationale are informative unless explicitly marked normative.
- This SPEC.md is authoritative for required behavior. Where implementation code or implementation documentation conflicts with it, the spec wins and the conflict MUST be surfaced.
- If an omission or ambiguity could materially affect observable behavior, data semantics, security, privacy, accessibility, or interoperability, it MUST be surfaced rather than silently resolved. If two normative statements conflict, the conflict MUST be surfaced.
- Unresolved product or design decisions are marked `TBD` in [Open Issues](#open-issues) and MUST NOT be guessed.

### Implementation Freedom

The design is independent of programming language, framework, rendering technology (Canvas, SVG, WebGL, DOM), hosting, source organization, and UI styling. A conforming implementation MAY choose any of these, provided every normative requirement below holds. The visual appearance of stones, plants, sand grooves, and weather is free, except where a requirement names a visible cue that MUST exist.

## 1. Overview and Scope

### System at a Glance

A single-player, browser-based dry-garden (karesansui) simulation. The Gardener lays out a rectangular sand garden with stones, lanterns, a water basin, moss, and plants, rakes patterns into the sand, and then keeps the garden healthy over simulated days and seasons: watering, pruning, pulling weeds, and clearing fallen leaves. A **Tranquility** score (0–100) summarizes the garden's current condition. The garden lives only in page memory; the Gardener keeps it by exporting a garden file and re-opening it later.

The system is delivered as one self-contained HTML document that runs inside a sandboxed iframe (see INT-001, SEC-001). It is played with a mouse on desktop web browsers and with touch on phones and tablets (FUN-022).

### Actors

| Actor | Description |
|---|---|
| Gardener | The single human player. Has full control over the one in-memory garden in their page. |
| Host frame | The page that embeds the garden in a sandboxed iframe and MAY inject a downloads shim (INT-001). It is not trusted with, and not given, garden data. |

There are no administrators, accounts, or other players.

### In scope

Garden creation and layout; sand raking; plant care simulation; day, season, and weather cycle; Tranquility scoring; garden file export/import; garden picture export.

### Out of scope

See [Constraints and Non-Goals](#6-constraints-and-non-goals).

## 2. Context and Definitions

| Term | Meaning |
|---|---|
| Garden | The whole playable state: surface, objects, sand, weeds, leaves, clock, name. Exactly one exists at a time. |
| Logical unit (u) | The garden's coordinate unit. The surface is 1200 u wide and 800 u tall; origin at top-left, x to the right, y downward. |
| Cell | One of 60 × 40 square cells of 20 u × 20 u tiling the surface, indexed row-major from the top-left. |
| Object | A placed item: a stone, lantern, water basin, or plant. |
| Plant | An object of a living kind: moss, azalea, maple, pine, bamboo. |
| Footprint | The circle of an object's footprint radius (Table K) around its center. |
| Open cell | A cell whose center lies outside every object's footprint. |
| Raked cell | A cell whose sand currently carries a groove pattern. |
| Covered cell | A cell is covered by a rake band, ripple rings, or a smooth brush when the cell's center lies inside that shape. Only the center counts; a cell whose center is outside is not covered, however much of it the shape overlaps. |
| Day | One unit of simulated time. Day index `d` is an integer ≥ 0. The Gardener sees "Day d+1". |
| Season | Derived from the day index: `floor((d mod 24) / 6)` → 0 Spring, 1 Summer, 2 Autumn, 3 Winter. |
| Withered | Terminal plant state reached when health hits 0. |

Numbers in this spec are decimal with `.` separator. Percentages shown to the Gardener are rounded to the nearest integer.

## 3. System Model

### 3.1 Object kinds

**Normative — Table K**

| Kind | Display name | Living | Footprint radius (u) | Hydration loss per day | Days per growth stage | Overgrowth per day |
|---|---|---|---|---|---|---|
| rock | Stone | no | S 22 · M 36 · L 54 | – | – | – |
| lantern | Lantern | no | 26 | – | – | – |
| basin | Water basin | no | 30 | – | – | – |
| moss | Moss | yes | 40 | 25 | 2 | 0 |
| shrub | Azalea | yes | 34 | 18 | 3 | 12 |
| maple | Maple | yes | 48 | 14 | 4 | 10 |
| pine | Pine | yes | 42 | 8 | 5 | 8 |
| bamboo | Bamboo | yes | 30 | 22 | 2 | 15 |

A plant's footprint radius is fixed at the value above for its whole life, regardless of growth stage.

### 3.2 Suggested garden names

**Normative — Table N.** A suggested name is written as the kanji, one space, then the English name, for example `苔石 Moss and Stone`.

| Kanji | English |
|---|---|
| 苔石 | Moss and Stone |
| 短夏 | Short Summer |
| 残夏 | Fading Summer |
| 寂庭 | Garden of Silence |
| 静月 | Quiet Moon |
| 孤石 | Lone Stone |
| 石影 | Stone Shadow |
| 松影 | Pine Shadow |
| 空山 | Empty Mountain |
| 残雪 | Remaining Snow |
| 静水 | Still Water |
| 白砂 | White Sand |
| 月影 | Moon Shadow |
| 遠山 | Distant Mountain |
| 静寂 | Stillness |

### 3.3 Plant attributes and states

Each plant has: `hydration` 0–100, `health` 0–100, `stage` ∈ {0 Sprout, 1 Young, 2 Mature}, `stageDays` ≥ 0, `overgrowth` 0–100, `withered` boolean.

A newly placed plant starts with hydration 100, health 100, stage 0, stageDays 0, overgrowth 0, withered false.

State flow: `Sprout → Young → Mature` (growth), and from any stage `→ Withered` when health reaches 0. Withered is terminal; a withered plant can only be removed.

### 3.4 Daily tick

**Normative.** When simulated time crosses into a new day index `d`, the system MUST apply the following steps in order, once per day crossed:

1. **Weather.** Roll the day's weather: Spring rain 35 %; Summer rain 20 %; Autumn rain 25 %; Winter snow 35 %; otherwise clear.
2. **Hydration.** For each non-withered plant: subtract its Table K hydration loss × season multiplier (Summer 1.5, Winter 0.5, else 1.0). If the day is rainy, set hydration to 100 instead.
3. **Health.** For each non-withered plant: if hydration < 25, health −15; otherwise health +5. Clamp to 0–100. If health is 0, the plant becomes withered.
4. **Growth.** Outside Winter, for each non-withered plant with health ≥ 50 and stage < 2: stageDays +1; when stageDays reaches its Table K "days per growth stage", stage +1 and stageDays = 0.
5. **Overgrowth.** Outside Winter, for each non-withered plant at stage 2: overgrowth + its Table K rate, clamped to 100.
6. **Weeds.** Spawn weeds: Spring 3, Summer 2, Autumn 1, Winter 0, plus 1 on a rainy day. Each weed is placed at a random point at least 10 u inside the surface edge and outside the footprint of every non-moss object. Weeds MAY grow on moss. Spawning stops at the weed cap (INV-005).
7. **Leaves.** Non-withered plants at stage ≥ 1 drop leaves: in Autumn a maple drops 6 and each azalea, pine, or bamboo drops 1; in Spring and Summer a maple drops 1; nothing drops in Winter or from moss. Each leaf lands at a random point within 1.6 × the plant's footprint radius of its center, clamped to the surface. Dropping stops at the leaf cap (INV-005).

### 3.5 Clock

Simulated time is a non-negative real number of days. Its integer part is the day index; its fractional part is the time of day: Dawn [0, 0.10), Day [0.10, 0.60), Dusk [0.60, 0.72), Night [0.72, 1). Speeds: **Paused**, **0.5×** (1 day = 90 real seconds), **1×** (1 day = 45 real seconds, the default on load and for New garden and Open garden), **4×** (1 day = 11.25 real seconds).

### 3.6 Tranquility

**Normative.** Tranquility = round(35·R + 25·P + 25·C + 15·G), where:

- **R (raked)** = raked open cells ÷ open cells; 1 if there are no open cells.
- **P (plant health)** = mean health ÷ 100 over all plants, withered plants counting as 0; 1 if there are no plants.
- **C (clean)** = max(0, 1 − 0.04 × weeds − 0.01 × leaves).
- **G (groomed)** = 1 − mean overgrowth ÷ 100 over non-withered stage-2 plants; 1 if there are none.

Band labels: 0–39 Restless, 40–69 Settling, 70–89 Calm, 90–100 Still.

### 3.7 Screens and layouts

**Intent (informative).** On the web the whole game is one screen: everything the Gardener uses (tools, garden, tool options, Tranquility, garden notes, speed, and the garden menu) is visible at once, and nothing needs scrolling or opening to be found. A phone is too small for that, so the phone shows one view at a time and moves secondary things behind a single tap: the garden view is for playing, the notes view is for reading what the garden needs, tool options open on a long press, the score details open from the badge, and speed and files live in the menu. The garden itself is never shrunk to make room for panels.

**Normative — Table L (layouts).** The layout is chosen by viewport width:

| Layout | Viewport width | What is on screen |
|---|---|---|
| Web | 1061 CSS px and wider | One screen, no page scrolling at heights of 700 CSS px and more: header (title, garden name, day, season, phase, weather, speed control, garden menu); tool column; garden with details line; side panel with the active tool's options (FUN-036), the Tranquility card (FUN-015), and garden notes (FUN-016). A panel whose content does not fit scrolls inside itself. |
| Tablet | 641–1060 CSS px | Tool column and garden first; the options and notes panel below the garden (the page may scroll); the Tranquility card is replaced by the badge (FUN-035). |
| Phone | 640 CSS px and narrower | One view at a time (FUN-033): the header, then either the garden view or the notes view. Details in FUN-027, FUN-030, FUN-033 to FUN-035. |

**Normative — Table T.** Tool descriptions (FUN-037, FUN-044).

| Tool | Description |
|---|---|
| Rake | Drag to comb parallel grooves into the sand. Raking clears fallen leaves. |
| Ripple | Click beside a stone to ring it with concentric grooves, or anywhere for a ripple. |
| Smooth | Drag to smooth sand flat again and sweep up leaves. |
| Place | Choose a stone, ornament, or plant, then click to set it in the garden. |
| Move | Drag an object to a new spot. If it would overlap, it returns. |
| Water | Click a plant to water it fully. Plants below 25% water lose health each day. |
| Prune | Click a mature plant to trim its overgrowth. |
| Hand | Click or drag to pull weeds and pick up leaves by hand. |
| Remove | Click an object to lift it out of the garden. Withered plants can only be removed. |

### 3.8 First minute (informative)

This walkthrough shows the intended feel. It adds no requirements; where it differs from a requirement, the requirement wins.

**Web.** The page opens on an autumn garden with a random name such as 寂庭 Garden of Silence. Everything is on one screen: tools on the left, the garden in the middle, and on the right the Rake's description and tine options, the Tranquility score (for example 76, Calm) with its four bars, and the garden notes ("The azalea is thirsty. Water it.", "The maple is overgrown. Prune it.", "4 weeds are growing…"). Time is already running at 1×. The Gardener moves the mouse over the maple and the details line reads "Maple · Mature · Water 40% · Health 82% · Overgrowth 64%". They pick Water, click the azalea ("Azalea watered"), pick Prune, click the maple ("Maple pruned"), and the score rises. They rake a few strokes across the bare lower sand; the grooves appear under the pointer and Raked sand climbs. Dusk comes after about half a minute and the lantern begins to glow. Nothing has asked them to sign in, save, or read instructions.

**Phone (upright).** The same garden opens under a compact header: 枯山水, the name, the score badge, and the menu. Below are the eight tools in two rows, the view switch, then the garden filling the width. The hint under the garden says "Tap a plant to see how it is doing. Long-press a tool for its options." The Gardener taps the maple with Rake active to read its details, which stay after lifting the finger. They long-press Ripple, choose 7 rings in the sheet, tap Done, and tap beside a stone to ring it. To see what the garden needs they tap the second segment of the view switch; the notes view replaces the tools and garden. They swipe right on the switch to return. Speed, New, Open, and Save are in the menu. If they turn the phone sideways, a notice asks them to hold it upright, with Continue anyway.

### 3.9 Timing

**Normative — Table Z.** Durations and distances that shape the feel. "About" allows ±20 %.

| What | Value |
|---|---|
| One simulated day | 90 s at 0.5×, 45 s at 1×, 11.25 s at 4× |
| Score, notes, clock, and details refresh | at least every 500 ms (the reference build refreshes every 250 ms) |
| Long press on a tool | 500 ms, cancelled if the pointer moves more than 10 CSS px |
| Action message | visible about 2 s, fading out over about 0.2 s (instant with reduced motion, FUN-042) |
| View-switch swipe | at least 30 CSS px horizontally |
| Move starts counting as a move | after the pointer travels 3 u |
| Rake stroke sampling | a new groove segment once the pointer has moved at least 7 u |
| Smooth brush stamping | one stamp every 10 u or less along the drag |
| Frame time step | at most 0.1 s of real time per frame, so a stalled page never jumps days ahead |
| Dawn and dusk | the night shade rises linearly over Dusk [0.60, 0.72) and falls linearly over Dawn [0, 0.10) |
| Double tap, double click | have no meaning; every tap or click is handled on its own |

### 3.10 Wording

**Normative — Table M.** The system MUST use this wording for the texts it shows (FUN-044). `{Kind}` is a Table K display name and `{kind}` the same in lower case; `{name}` is the garden name, `{filename}` a file name from INT-003 or INT-004, `{n}` a count, and `{reason}` a placement reason from this table.

| Where | Text |
|---|---|
| Header title | 枯山水 |
| Clock chips | "Day {n}"; Spring, Summer, Autumn, Winter; Dawn, Day, Dusk, Night; Clear, Rain, Snow |
| Speed control | Pause, 0.5×, 1×, 4× |
| Garden menu | New garden, Open garden, Save garden, Save picture |
| Tool names | Rake, Ripple, Smooth, Place, Move, Water, Prune, Hand, Remove |
| Tool option labels | Tines (3, 5, 7); Rings (3, 5, 7); Brush (Small, Medium, Large); Object (Table K display names); Stone size (Small, Medium, Large) |
| Panels | "Garden notes"; "Tranquility"; components Raked sand, Plant health, Clean ground, Groomed; bands Restless, Settling, Calm, Still |
| Keyboard hint (mouse devices only) | "1–9 tools · Space pause" |
| Details line, nothing selected | "Hover a plant to see how it is doing." (mouse); "Tap a plant to see how it is doing. Long-press a tool for its options." (touch) |
| Details line, stone, lantern, basin | "{Kind} · Stone and still." (stones add their size: "Stone (L) · Stone and still.") |
| Details line, plant | "{Kind} · {Sprout, Young, or Mature} · Water {n}% · Health {n}% · Overgrowth {n}%" |
| Details line, withered plant | "{Kind} · Withered. Remove it to make room." |
| Messages (FUN-038) | "{Kind} placed"; "{Kind} moved"; "{Kind} watered"; "{Kind} pruned"; "{Kind} removed"; "Picked 1 item" / "Picked {n} items"; "A fresh bed of sand: {name}"; "Opened {name}"; "Saved {filename}"; "Save cancelled" |
| Refusals | "Can't place here. {reason}"; "Moved back. {reason}"; "Only mature plants need pruning."; "This plant has withered. Remove it." |
| Placement reasons | "It must sit fully inside the garden."; "It would overlap the {kind}." |
| Garden notes (FUN-016) | "The garden is empty. Choose Place to set stones and plants, then Rake the sand."; "The {kind} has withered. Remove it."; "The {kind} is thirsty. Water it."; "The {kind} is overgrown. Prune it."; "A weed is growing. Pull it by hand." / "{n} weeds are growing. Pull them by hand."; "A leaf has fallen on the sand." / "{n} leaves have fallen. Rake, smooth, or pick them up."; "Only {n}% of the open sand is raked."; "{n} more needs."; "The garden is in balance. Sit with it a while." |
| New garden dialog | Title "Start a new garden?"; body "The current garden will be cleared. Save it first if you want to keep it."; buttons "Keep gardening", "Start new garden" |
| Save dialog | Title "Save garden"; body "Give this garden a name. You can open the file later with Open garden."; field label "Name"; hint "Saves as {filename}"; buttons "Cancel", "Save" |
| Save fallback, garden | Title "Copy your garden file"; body "This frame could not save {filename} directly. Copy the text below and keep it somewhere safe. Paste it into Open garden to come back."; buttons "Try download", "Close", "Copy" (then "Copied", or "Selected. Press Ctrl+C or ⌘C" when copying is refused) |
| Save fallback, picture | Title "Your garden picture"; body "This frame could not save the picture directly. Right-click or long-press the image to save it."; buttons "Try download", "Close" |
| Open dialog | Title "Open a garden"; body "Choose a .garden file or paste its text. The current garden will be replaced."; paste placeholder "Or paste garden file text here"; buttons "Cancel", "Open garden" |
| Open errors (SEC-003) | "Choose a file or paste garden text first."; "That file is larger than 8 MB, so it is not a garden file."; "That text is larger than 8 MB, so it is not a garden file."; "The file could not be read."; "That is not valid garden file text."; "This is not a Karesansui garden file."; "This garden file uses an unsupported version."; "The garden name must be text of at most 40 characters."; "The garden day is out of range."; "The weather value is not recognized."; "The objects list is missing or too long."; "A garden can hold at most 30 weeds."; "A garden can hold at most 200 leaves."; "The raked-sand grid is damaged."; "The sand image is missing or not a PNG."; "The file contains an unknown kind of object."; "Object ids must be unique positive whole numbers."; "An object has an invalid position or seed."; "A stone has an invalid size."; "A plant has invalid care values."; "An object sits outside the garden."; "Two objects in the file overlap."; "A weed has invalid values."; "A leaf has invalid values."; "The sand image could not be read."; "The sand image must be 1200 × 800." |
| Tool options sheet | Tool name as title; the tool's description (Table T); option labels as above; button "Done" |
| Tranquility sheet | Title "Tranquility"; score, band, and components as in Panels; button "Close" |
| Turn-upright notice (FUN-031) | 枯山水; title "Turn your phone upright"; body "The garden is laid out for portrait. Hold your phone upright to tend it."; button "Continue anyway" |

### Critical Invariants

- **INV-001:** **Sandbox containment.** The system MUST NOT use host storage (localStorage, sessionStorage, IndexedDB, cookies), MUST NOT access or message the parent or top frame, and MUST NOT call popups, alert/confirm/prompt, fullscreen, or pointer lock.
- **INV-002:** **No overlap.** No two object footprints overlap, and every footprint lies fully inside the surface. This holds after every place, move, and import.
- **INV-003:** **Bounded attributes.** Every plant attribute stays within its declared range at all times.
- **INV-004:** **Score is a pure function.** Tranquility is always an integer in 0–100 computed only from the current garden state by §3.6.
- **INV-005:** **Caps.** A garden never holds more than 30 weeds or more than 200 leaves. While the portrait phone layout (FUN-027) applies, no new leaf is added once the garden holds 120 leaves; leaves already present are kept.
- **INV-006:** **Atomic import.** An import either replaces the whole garden with the file's contents or changes nothing.
- **INV-007:** **Sand under objects is smooth.** Placing or moving an object resets the sand under its new footprint to smooth (unraked), and no rake, ripple, or smooth action alters sand inside any footprint.

## 4. Requirements

### 4.1 Functional — layout and sand (FUN)

- **FUN-001:** **Surface.** The system MUST present the garden surface at a 3:2 aspect ratio, scaled uniformly to fit the available width without horizontal page scrolling at any viewport width from 320 px up (FUN-043), and MUST map mouse and touch input to logical units at that scale. Touching the surface MUST NOT scroll or zoom the page.
- **FUN-002:** **Tool palette.** The system MUST offer exactly one active tool at a time from: Rake, Ripple, Smooth, Place, Move, Water, Prune, Hand, Remove. The active tool MUST be visibly indicated. The default tool is Rake. Exception: the portrait phone layout (FUN-027) does not offer Move.
- **FUN-003:** **Rake.** Dragging with Rake MUST draw parallel grooves following the pointer path, with 3, 5, or 7 tines (default 5) spaced 9 u apart. The band swept by the rake MUST replace any earlier pattern beneath it, mark the open cells it covers as raked, and remove leaves within it.
- **FUN-004:** **Ripple.** Clicking with Ripple MUST draw 3, 5, or 7 (default 5) concentric groove rings spaced 10 u apart. If the click is within 1.6 × the footprint radius of an object's center, the rings center on that object and start 10 u outside its footprint; otherwise they center on the click point starting at radius 12 u. Covered open cells become raked and leaves within the rings are removed.
- **FUN-005:** **Smooth.** Dragging with Smooth MUST restore flat sand within a brush of radius 24, 40, or 64 u (default 40), mark covered cells unraked, and remove leaves within it.
- **FUN-006:** **Place.** With Place the Gardener chooses a kind (Table K) and, for stones, a size S/M/L (default M). The system MUST show a preview of the footprint while a mouse hovers over the surface and while a press (mouse or touch) is held and dragged, and MUST distinguish an invalid position by a cue that is not color alone. Placement happens on release: releasing at a valid position creates the object; releasing at an invalid one creates nothing and tells the Gardener why. Placement removes weeds and leaves within the new footprint.
- **FUN-007:** **Move.** Dragging an object with Move MUST relocate it on release if the new position is valid; otherwise it MUST return to its original position and tell the Gardener why. While it is dragged, the object MUST follow the pointer and show the same invalid-position cue as Place (FUN-006). Weeds and leaves under the new footprint are removed.
- **FUN-008:** **Remove.** Clicking an object with Remove MUST delete it. Removing is the only way to clear a withered plant.
- **FUN-009:** **Garden name.** The Gardener MUST be able to edit the garden name, 1–40 characters; an empty name is treated as "Untitled garden".

### 4.2 Functional — care and time (FUN)

- **FUN-010:** **Water.** Clicking a non-withered plant with Water MUST set its hydration to 100. Clicking anything else changes nothing.
- **FUN-011:** **Prune.** Clicking a non-withered stage-2 plant with Prune MUST set its overgrowth to 0. Clicking a younger plant MUST tell the Gardener that only mature plants need pruning.
- **FUN-012:** **Hand.** Clicking or tapping with Hand MUST remove every weed and leaf within 16 u of the point, and dragging MUST keep removing them along the path.
- **FUN-013:** **Simulation.** The system MUST advance simulated time at the selected speed and apply the daily tick (§3.4) for each day crossed. The Gardener MUST be able to switch between Paused, 0.5×, 1×, and 4× at any time, in that order on the speed control; 1× is the default on page load. Choosing a speed MUST mark it as active and keep it until the Gardener changes it (New garden and Open garden do not change it). Simulated time SHOULD NOT advance while the page is hidden.
- **FUN-014:** **Clock display.** The system MUST show the day number, season, time-of-day phase, and weather.
- **FUN-015:** **Tranquility display.** The system MUST show the current Tranquility score, its band label, and the four components R, P, C, G, updated at least every 500 ms.
- **FUN-016:** **Garden notes.** The system MUST list the garden's current needs, including at least: withered plants, thirsty plants (hydration < 25), overgrown plants (overgrowth ≥ 60), weed count, leaf count, and low raking (R < 0.6). The list MUST be ordered: withered plants, thirsty plants, overgrown plants, weeds, leaves, low raking. It MUST show at most 7 entries, followed by "N more needs." when there are more. When the garden has no objects, the first entry MUST say the garden is empty and suggest Place and then Rake. When nothing applies it MUST say the garden is in balance.
- **FUN-017:** **Inspect.** Hovering over a plant with a mouse, or pressing or tapping it with any tool, MUST show its kind, stage, hydration, health, and overgrowth; for a withered plant it MUST say it has withered. On touch the shown details MUST stay after the finger lifts, until another object is touched.
- **FUN-018:** **Visible condition.** Plants MUST visibly differ when thirsty, overgrown, and withered; maples MUST visibly reflect the season; Night MUST darken the garden and lanterns MUST glow at Night.
- **FUN-019:** **Starting state.** On load the system MUST open a sample garden, named by FUN-024, at day index 12 (Autumn) containing stones, moss, a lantern, a water basin, a maple, a pine, an azalea, a bamboo, some raked sand, some fallen leaves, and a few weeds, with at least one plant thirsty and one overgrown.
- **FUN-020:** **New garden.** The Gardener MUST be able to start a new, empty garden at day index 0 with smooth sand, named by FUN-024, after an in-page confirmation that warns the current garden will be lost unless saved.
- **FUN-021:** **Keyboard shortcuts.** As a desktop convenience, keys 1–9 MUST select the tools in FUN-002 order and Space MUST toggle between Paused and the last running speed, except while a text field has focus. No action MAY require the keyboard.
- **FUN-022:** **Mouse and touch.** Every garden action MUST be possible with a mouse alone and with a single finger on a touchscreen, except that Move is offered only outside the portrait phone layout (FUN-027). No action MAY depend on hover, a right click, a multi-finger gesture, or a keyboard. On touch-first devices (no hover), the system MUST NOT show keyboard shortcut hints, and every control (tools, option buttons, view-switch segments, the badge, the menu button and items, and sheet and dialog buttons) MUST offer a touch target at least 40 × 40 CSS px; the visible mark MAY be smaller than its touch target.
- **FUN-023:** **Name on save.** Choosing Save garden MUST first open an in-page dialog with a name field prefilled with the current garden name (1–40 characters). Confirming MUST set the garden's name to the entered text (blank → "Untitled garden") and then save it (INT-001/INT-002). Cancelling MUST save nothing and leave the name unchanged. The dialog MUST work with mouse and touch; on devices with a keyboard, Enter confirms.
- **FUN-024:** **Suggested name.** Each new game (page load and New garden) MUST name the garden with an entry chosen uniformly at random from Table N, written in the kanji-then-English form. New garden MUST choose an entry different from the current garden's name when that name is itself a Table N entry. The Gardener MAY rename it (FUN-009, FUN-023).
- **FUN-027:** **Portrait phone toolbar.** When the viewport is at most 640 CSS px wide, the tool palette MUST show exactly 8 tools, Rake, Ripple, Smooth, Place, Water, Prune, Hand, Remove, as two rows of 4 in that order, with no scrolling. Move MUST NOT be shown there; if Move is active when this layout applies, the active tool MUST become Rake. Other layouts keep all 9 tools. In this layout the header's first row MUST hold, left to right: the title, the garden name, the Tranquility badge, and the garden menu button at the right edge; below 360 CSS px the garden name moves to its own second row (FUN-043). At viewport widths up to 400 CSS px the badge MAY show only its ring and score. The garden view switch MUST sit centered in its own row directly above the garden, and MUST stay visible in the notes view.
- **FUN-029:** **Menu position.** In every layout the garden menu button MUST be at the right end of the header's first row. Exception: a phone held sideways after the Gardener dismissed the FUN-031 notice, whose layout is not specified. When space is short, the garden name's text MUST shrink (to no less than 12 CSS px) rather than be cut off or push the menu to another row. In every layout the header title MUST show only the kanji 枯山水, with no Latin "Karesansui" beside it.
- **FUN-030:** **Portrait phone litter.** While the portrait phone layout (FUN-027) applies: (a) weeds and fallen leaves MUST be drawn 1.5× their normal size, centered on the same positions (their positions, pickup radius, and saved data are unchanged); (b) every leaf count in §3.4 step 7 and in the starting garden (FUN-019) MUST be multiplied by 0.6, with any fractional part becoming the probability of one extra leaf (for example 6 becomes 3 or 4, averaging 3.6); (c) the leaf cap is 120 (INV-005). Tranquility (§3.6) still counts the actual leaves.
- **FUN-031:** **Portrait preferred on phones.** The system MUST NOT provide a dedicated layout for phones held sideways. When the device is a phone (coarse pointer and the screen's shorter side at most 500 CSS px) and the device orientation is landscape, the system MUST cover the page with a notice asking the Gardener to hold the phone upright. Orientation MUST be read from the device (`screen.orientation`, then `window.orientation`), falling back to the viewport's shape only when neither exists, because an embedding frame may not resize on rotation. The notice MUST offer a "Continue anyway" control from the moment it appears; continuing hides it until the phone has been upright again and turns sideways again. The notice MUST NOT pause, reset, or otherwise change the garden.
- **FUN-028:** **Tool options on long press.** Pressing and holding a tool button for 500 ms (mouse or touch, moving less than 10 CSS px) MUST make it the active tool and open an in-page sheet showing its name, description, and every option that tool has: Rake tines 3/5/7; Ripple rings 3/5/7; Smooth brush Small/Medium/Large; Place object kind (Table K) and, for Stone, size S/M/L. Choices take effect at once and stay in step with any other options view. A Done control closes the sheet. The long press MUST NOT also trigger the tool button's click or the OS context menu. On touch-first devices the details line SHOULD mention that a long press opens a tool's options.
- **FUN-032:** **Layouts.** The system MUST choose the layout by viewport width as in Table L, and in the Web layout MUST show every element listed there at once, without page scrolling, at viewport heights of 700 CSS px and more.
- **FUN-033:** **Phone views.** In the Phone layout the system MUST show one of two views. The garden view holds the tools (FUN-027), the view switch, the garden, and the details line. The notes view holds only the garden notes (FUN-016); the tools and garden are hidden. The view switch MUST show which view is active; tapping a segment MUST open that view, and a horizontal swipe of at least 30 CSS px on the switch MUST open the notes view (leftward) or the garden view (rightward). The garden view is shown on load. Switching views MUST NOT change the garden or the clock.
- **FUN-034:** **Garden menu.** The garden menu MUST offer, in this order: New garden, Open garden, Save garden, Save picture. In the Phone layout it MUST also offer the speed control (Paused, 0.5×, 1×, 4×) at its top; in the other layouts the speed control is in the header instead. The menu MUST close when an item is chosen, when the Gardener clicks or taps outside it, and on Escape.
- **FUN-035:** **Tranquility on narrow screens.** In the Tablet and Phone layouts the Tranquility card MUST be replaced by a badge showing a progress ring, the score, and the band label (the label MAY be dropped at 400 CSS px and narrower, FUN-027). Clicking or tapping the badge MUST open an in-page sheet with the score, band label, and the four components R, P, C, G; closing it returns to where the Gardener was.
- **FUN-036:** **Tool options on wider screens.** In the Web and Tablet layouts the active tool's name, one-sentence description, and options (as listed in FUN-028) MUST always be visible in the panel, without a long press. Choosing an option there MUST take effect at once. The long press (FUN-028) MUST work in these layouts too.
- **FUN-037:** **Guidance.** Every tool MUST have the one-sentence description given in Table T. When no object is shown in the details line, the line MUST say how to see a plant's details: "Hover a plant to see how it is doing." with a mouse, or "Tap a plant to see how it is doing. Long-press a tool for its options." on touch-first devices. The empty-garden note of FUN-016 is the only other guidance; there is no tutorial.
- **FUN-038:** **Action feedback.** Place, Move, Water, Prune, Hand (when it picks something up), Remove, New garden, Open garden, Save garden, Save picture, and a cancelled save MUST each confirm with a short message that names what happened (for example "Maple watered", "Picked 3 items", "Saved 寂庭 Garden of Silence.garden"). The message MUST NOT block play and MUST disappear by itself within about 2 seconds. Rake, Ripple, and Smooth show their effect in the sand and need no message. Refusals follow FUN-006, FUN-007, and FUN-011.
- **FUN-039:** **Replacing the garden.** New garden MUST ask for confirmation first (FUN-020). The Open garden dialog MUST state that the current garden will be replaced before the Gardener confirms. There is no other protection against losing an unsaved garden (see §6).
- **FUN-040:** **Time and dialogs.** Simulated time MUST keep running while menus, dialogs, sheets, and the FUN-031 notice are open. Only the Paused speed and a hidden page stop it (FUN-013).
- **FUN-041:** **Pointer affordances.** With Water, Prune, Remove, or Move active, the object under the mouse, or the last object tapped, MUST be outlined so the Gardener sees what the tool will act on; Remove's outline MUST differ from the others. On devices with a keyboard, Escape MUST close an open dialog, sheet, or menu.
- **FUN-042:** **Reduced motion.** When the Gardener's system asks for reduced motion, falling rain and snow MUST be drawn still, lanterns MUST NOT flicker, and interface transitions MUST be instant. The simulation is unchanged.
- **FUN-043:** **Small screens.** The Phone layout MUST work down to 320 CSS px wide. (a) From 320 px up there is no horizontal page scrolling, the menu stays at the top-right (FUN-029), all four clock chips sit on one row, and all eight tools show in two rows of four. (b) From 360 px up the garden name shares the first header row; from 320 to 359 px it moves to its own row under the title, badge, and menu, so every Table N name fits at 12 CSS px or larger. (c) The garden view (header, tools, view switch, garden, and details line) MUST fit without vertical scrolling at heights of 480 CSS px and more when the viewport is 360 px wide or wider, and at heights of 520 px and more when it is 320–359 px wide (the name's extra row needs the room); when the height is short, the garden MUST shrink, keeping 3:2, but not below 240 CSS px wide, and only then MAY the page scroll vertically. (d) Dialogs and sheets fit within the viewport width and scroll inside themselves when taller than the viewport. (e) Below 320 px wide the layout is not specified beyond having no horizontal page scrolling.
- **FUN-044:** **Wording.** Every text listed in Table M and Table T MUST appear exactly as written there, with only the placeholders filled in. Texts not listed there are free but SHOULD follow the same plain, calm tone.

### 4.3 Data (DATA)

- **DATA-001:** **Garden.** A garden consists of: name, simulated time, today's weather (clear, rain, snow), objects, weeds, leaves, the 60 × 40 raked-cell grid, and the sand pattern image.
- **DATA-002:** **Object.** An object has a unique positive integer id, kind, center (x, y) in u, size (stones only), a shape seed (integer used for stable procedural appearance), and, for plants, the attributes of §3.3.
- **DATA-003:** **Weed.** A weed has a position and a shape seed.
- **DATA-004:** **Leaf.** A leaf has a position, a rotation in radians, and a color index 0–4.
- **DATA-005:** **Memory only.** All garden state MUST live in page memory only. Reloading the page MUST return to the sample garden (FUN-019).

### 4.4 Identity and access (AUTH)

- **AUTH-001:** **No identity.** The system MUST NOT require, request, or store any account, sign-in, or identity. Whoever has the page open is the Gardener and has full control over that page's garden.
- **AUTH-002:** **No sharing.** The system MUST NOT make one Gardener's garden visible to anyone else except through a file the Gardener exports and shares themselves.

### 4.5 Security and privacy (SEC)

- **SEC-001:** **Sandbox profile.** The system MUST work inside an iframe sandboxed with only `allow-scripts allow-forms allow-downloads` at an opaque origin, as one self-contained HTML document. It MAY load public, anonymous CDN resources (scripts, fonts); it MUST NOT depend on any other file.
- **SEC-002:** **Untrusted import.** Text from an imported file (including the garden name) MUST be rendered as plain text and MUST NOT be interpreted as markup or script.
- **SEC-003:** **Import validation.** Import MUST reject any file larger than 8 MB, any value outside the ranges of INT-003, unknown kinds, duplicate ids, overlapping or out-of-bounds footprints, more than the caps of INV-005, and a sand image that is not a 1200 × 800 PNG data URI.
- **SEC-004:** **No data egress.** The system MUST NOT send garden data or any user input over the network.

### 4.6 Interfaces (INT)

- **INT-001:** **Downloads shim.** To save a file, the system MUST first call `window.claude.use('downloads').save({ filename, data })` when it exists, with `data` as a string or Blob, accepting either a direct or a promised return from `use` and `save`. If `save` rejects with code `declined` or `rate_limited`, the Gardener chose not to save: the system MUST NOT show the INT-002 dialog and SHOULD say the save was cancelled. Any other throw or rejection (for example `rejected_extension`), or `use` returning `null`, counts as failure.
- **INT-002:** **Save fallback.** If the shim is absent or fails (not when cancelled, INT-001), the system MUST show the saved content in an in-page dialog: for a garden file, the text with a Copy button; for a picture, the image. The dialog MAY also offer a Blob-URL download link.
- **INT-003:** **Garden file.** "Save garden" MUST produce a UTF-8 file whose content is JSON in the format below, named `<file-name>.garden`. `<file-name>` is the garden name with the characters `/ \ : * ? " < > |` and control characters replaced by `-`, surrounding spaces and dots trimmed, and "Untitled garden" used if nothing remains. "Open garden" MUST accept the same format from a chosen file or from pasted text; the file picker MUST offer `.garden` files and MAY also offer `.json`. The file extension is not validated; the content is (SEC-003).

  **Normative — garden file, version 1**

  | Field | Type and range |
  |---|---|
  | `format` | the string `"karesansui-garden"` |
  | `version` | the integer `1` |
  | `name` | string, 0–40 characters |
  | `day` | number, 0 ≤ day < 100000 |
  | `weather` | `"clear"`, `"rain"`, or `"snow"` |
  | `objects` | array, ≤ 200 items: `{ id` integer ≥ 1, `kind` (Table K), `x`, `y` numbers, `size` `"S"`/`"M"`/`"L"` (rock only), `seed` integer 0–2³²−1; plants also `hydration`, `health`, `overgrowth` numbers 0–100, `stage` 0–2, `stageDays` integer 0–10, `withered` boolean `}` |
  | `weeds` | array, ≤ 30 items: `{ x` 0–1200, `y` 0–800, `seed }` |
  | `leaves` | array, ≤ 200 items: `{ x` 0–1200, `y` 0–800, `rot` number, `c` integer 0–4 `}` |
  | `coverage` | string of exactly 2400 characters `0`/`1`, row-major raked-cell grid |
  | `sand` | PNG data URI of the 1200 × 800 sand pattern |

- **INT-004:** **Picture.** "Save picture" MUST produce a PNG named `<file-name>.png` (file name as in INT-003) of the garden as currently shown, via INT-001/INT-002.

## 5. Interfaces and External Contracts

This section describes each external interface for independent interoperability. It adds no obligations of its own: the cited requirement IDs are normative, and where this summary and a cited requirement differ, the requirement wins.

### Host Downloads API

- **Purpose:** hand a file the Gardener chose to save (garden file or picture) to the host, which delivers it to the Gardener (INT-001).
- **Data authority:** the page owns the file content; the host owns delivery and may ask the Gardener to confirm.
- **Identity:** none. No account or identity is sent (AUTH-001).
- **Operations:** `window.claude.use('downloads')` then `save({ filename, data })`, where `data` is a string or Blob; either call may return directly or as a promise (INT-001). No other host call exists.
- **Failure behavior:** `declined` or `rate_limited` means the Gardener chose not to save: no fallback, a "Save cancelled" message. Any other rejection or throw, or no API at all, shows the in-page fallback dialog (INT-002). A save the host reports as successful is trusted (OI-001).
- **Security and privacy:** only files the Gardener chose to save are passed; nothing is sent otherwise (SEC-004, INV-001).
- **Known hosts (informative):**

| Host | How the API arrives | `.garden` save | `.png` save |
|---|---|---|---|
| Host that injects the shim | Shim injected before page scripts; saves any filename without a prompt | Direct | Direct |
| claude.ai artifact viewer | `use('downloads')` resolves asynchronously when the page declares the downloads capability; the viewer confirms each save; filenames must end in an allowlisted extension, which excludes `.garden` | Rejected (`rejected_extension`) → INT-002 dialog | Direct after confirmation |

The `.garden` extension is kept on every host (INT-003), accepting the claude.ai fallback.

### Garden File

- **Purpose:** the interchange format between sessions and between Gardeners; the only way a garden outlives a reload (DATA-005).
- **Data authority:** the file is authoritative for the garden it describes; opening it replaces the current garden whole (INV-006).
- **Operations:** Save garden writes `<file-name>.garden` after the naming dialog (FUN-023, INT-003); Open garden reads a chosen file or pasted text (INT-003).
- **Data mapping:** the version 1 schema in INT-003 (name, day, weather, objects, weeds, leaves, raked-cell grid, sand PNG).
- **Failure behavior:** any invalid content (size over 8 MB, out-of-range value, unknown kind, duplicate id, overlap, over the caps, bad sand image) is rejected with a reason, and the current garden is unchanged (SEC-003, INV-006).
- **Security and privacy:** file text is data, never markup or script (SEC-002).
- **Version compatibility:** only `version: 1` is accepted; other versions are rejected, with no migration (§6).

### Garden Picture

- **Purpose:** a PNG of the garden as currently shown, for the Gardener to keep or share (INT-004).
- **Operations:** Save picture writes `<file-name>.png` through the Host Downloads API (INT-001).
- **Failure behavior:** as for the Host Downloads API: cancelled saves show a message; other failures show the image in the fallback dialog (INT-002).

### Embedding Frame

- **Purpose:** the sandboxed iframe the page runs in (SEC-001).
- **Supported operations:** scripts, forms, and downloads only.
- **Prohibited operations:** storage, parent or top frame access, popups, browser dialogs, fullscreen, and pointer lock (INV-001).
- **Failure behavior:** the frame may not resize when a phone rotates; orientation is therefore read from the device (FUN-031).

### Gardener Input

- **Purpose:** how the Gardener plays: mouse, single-finger touch, and optional keyboard shortcuts (FUN-022, FUN-021).
- **Operations:** tool selection and options (FUN-002, FUN-028); surface actions: rake, ripple, smooth, place, move, water, prune, hand, remove (FUN-003 to FUN-012); speed control (FUN-013); naming, new, open, and save (FUN-009, FUN-020, FUN-023).
- **Failure behavior:** an invalid placement or move creates or changes nothing and says why (FUN-006, FUN-007); tools that do not apply to the touched object say so or do nothing (FUN-010, FUN-011).

## 6. Constraints and Non-Goals

- No persistence across reloads other than exported files (DATA-005, INV-001).
- No undo or redo.
- No sound.
- No multiplayer, sharing service, leaderboard, or accounts.
- No migration of garden files from other formats or versions; a file whose `version` is not 1 is rejected.
- No use of browser dialogs; all confirmations are in-page.
- No tracking of unsaved changes and no warning on reload or close: the sandbox blocks the browser's leave-page prompt (INV-001), so saving is the Gardener's job (FUN-039).
- No tutorial or guided first run beyond FUN-037.
- Keyboard-only and screen-reader operation of the garden surface is not required. Shortcuts (FUN-021) are a convenience only.
- Grooves are not required to deform around objects; they are only required not to alter sand inside footprints (INV-007).

## 7. Verification and Acceptance

Verification method: T = Test, D = Demonstration, I = Inspection.

- **V-01 — FUN-019, FUN-016:** Given page freshly loaded. Then sample garden named with a Table N entry in kanji-then-English form, Day 13, Autumn, at least one thirsty and one overgrown plant listed in notes (D)
- **V-02 — FUN-003, FUN-015:** Given rake with 5 tines. When gardener drags across smooth sand. Then 5 parallel grooves appear, R rises, leaves on the path disappear (D)
- **V-03 — FUN-004, INV-007:** Given a stone. When gardener clicks with Ripple next to it. Then rings center on the stone, start outside it, and leave the stone's footprint untouched (D)
- **V-04 — FUN-005:** Given raked area. When gardener drags Smooth over it. Then grooves vanish under the brush; R falls (D)
- **V-05 — FUN-006, INV-002:** Given place: Stone L. When mouse hovers, or finger presses and drags, over another object or the edge. Then preview shows invalid cue (dashed ring and color); release creates nothing and explains why (D)
- **V-06 — FUN-007, INV-002:** Given an object. When gardener drags it with Move onto another object and releases. Then it returns to its start position (D)
- **V-07 — FUN-010, FUN-017:** Given a thirsty maple. When gardener clicks it with Water. Then inspect shows water 100 % and it leaves the notes list (D)
- **V-08 — FUN-011:** Given a mature overgrown pine. When gardener clicks it with Prune. Then overgrowth 0; G rises. Clicking a sprout shows the "only mature plants" message (D)
- **V-09 — FUN-012:** Given weeds and leaves. When gardener clicks them with Hand. Then those within 16 u vanish; C rises (D)
- **V-10 — FUN-008, FUN-018, INV-003:** Given plant at hydration 20, health 15, clear Autumn day. When one day passes. Then health 0, plant withered and shown withered; Remove clears it (T) Also checks §3.4.
- **V-11 — INV-005:** Given garden with 30 weeds / 200 leaves. When days pass. Then counts never exceed the caps (T)
- **V-12 — INV-004, FUN-015:** Given any state. When score computed. Then equals §3.6 formula from current state, integer 0–100 (I/T)
- **V-13 — FUN-013, FUN-021:** Given speed 1×. When gardener presses Space, then Space. Then time stops, then resumes at 1× (D)
- **V-49 — FUN-013, FUN-021:** Given a fresh page. Then 1× is active. When the Gardener chooses 0.5×. Then a day takes about 90 s (half the pace of 1×), 0.5× is marked active in both the header and the phone menu, and Space pauses and then resumes at 0.5×; opening or starting a garden keeps 0.5× (T)
- **V-14 — FUN-023, INT-001, INT-003:** Given shim present. When gardener clicks Save garden, enters "Stones: Autumn?", confirms. Then garden name becomes "Stones: Autumn?"; `save` is called once with filename `Stones- Autumn-.garden` and a JSON string (T)
- **V-25 — FUN-023:** Given save dialog open. When gardener cancels. Then nothing saved; name unchanged (D)
- **V-15 — INT-002, INT-004:** Given shim absent or throwing. When gardener clicks Save garden / Save picture. Then in-page dialog shows the JSON with Copy / the image (D)
- **V-27 — INT-001:** Given shim whose `save` rejects `{code: "declined"}`. When gardener saves a garden or picture. Then no dialog; "Save cancelled" shown (T)
- **V-16 — DATA-001, DATA-002, DATA-003, DATA-004, INT-003:** Given saved garden file. When gardener opens it after reload. Then name, clock, objects, plant states, weeds, leaves, sand, and score match the saved garden (T)
- **V-17 — SEC-002, SEC-003, INV-006:** Given file with overlapping objects, bad kind, `<img onerror>` name, or 9 MB size. When gardener opens it. Then garden unchanged and a reason shown; a valid markup-laden name displays literally (T)
- **V-18 — INV-001, SEC-001, SEC-004, AUTH-001, AUTH-002, DATA-005:** Given page source. When inspected. Then no storage, parent/top, postMessage, window.open, alert/confirm/prompt, fullscreen, pointer-lock, or network calls with garden data (I)
- **V-19 — FUN-018:** Given garden at Night with a lantern. Then garden darker, lantern glowing (D) Also checks §3.5.
- **V-20 — FUN-002, FUN-021:** Given any tool. When gardener presses 1–9 outside text fields. Then matching tool becomes active and is visibly indicated (D)
- **V-21 — FUN-020, FUN-024:** Given any garden. When gardener chooses New garden, confirms. Then empty Day 1 Spring garden with smooth sand, named with a Table N entry other than the previous name; cancel leaves it unchanged (D)
- **V-26 — FUN-024:** Given page reloaded or New garden chosen 20 times. Then every name is a Table N entry in kanji-then-English form; more than one distinct name appears (T)
- **V-33 — FUN-009:** Given any garden. When the Gardener clears the name field and leaves it, then types 41 characters. Then the name becomes "Untitled garden", and only the first 40 characters are kept (D)
- **V-34 — FUN-014:** Given the page at 375 × 812 and at 1280 × 800. When time passes through Dawn, Day, Dusk, and Night and the weather changes. Then the day number, season, time-of-day phase, and weather are all shown and update (D)
- **V-35 — FUN-032:** Given viewports 1280 × 800, 1440 × 900, and 1100 × 700. Then header, tools, garden, tool options, Tranquility card, notes, speed control, and menu are all visible and the page does not scroll (T)
- **V-36 — FUN-033:** Given the Phone layout in the garden view. When the Gardener taps the notes segment, then swipes right on the switch. Then only the notes show, then the garden view returns; day, score, and objects are unchanged (D)
- **V-37 — FUN-034:** Given each layout. When the Gardener opens the menu. Then it lists New, Open, Save garden, Save picture in that order, with the speed control first only in the Phone layout; it closes on a choice, an outside tap, and Escape (D)
- **V-38 — FUN-035:** Given the Tablet or Phone layout. When the Gardener taps the badge. Then a sheet shows the score, band, and R, P, C, G matching the formula (D)
- **V-39 — FUN-036:** Given the Web layout. When the Gardener selects Ripple, then Place. Then the panel shows each tool's name, description, and options without a long press, and choosing 7 rings or Stone L takes effect at once (D)
- **V-40 — FUN-037, FUN-016:** Given a new, empty garden. Then the first note says the garden is empty and suggests Place and Rake, and the details line shows the hover or tap hint for the device (D)
- **V-41 — FUN-038:** Given each discrete action in FUN-038. When it is performed. Then a matching message appears and fades within about 2 seconds without blocking play; raking shows no message (D)
- **V-42 — FUN-039, FUN-040:** Given speed 1×. When the Gardener opens the Open garden dialog and waits. Then the dialog states the garden will be replaced, and the clock keeps advancing while it is open (D)
- **V-43 — FUN-041, FUN-007, FUN-012:** Given Water, then Remove, then Move active. When the pointer is over a plant, and an object is dragged onto another. Then the plant is outlined (Remove differently), the dragged object shows the invalid cue, returns on release, and the reason is shown; dragging Hand across weeds removes them all (D)
- **V-44 — FUN-042:** Given the system set to reduce motion and a rainy night with a lantern. Then rain is drawn still, the lantern does not flicker, and messages appear without fading (D)
- **V-46 — FUN-044:** Given each dialog, sheet, message, refusal, note, and details line in Table M. When it is shown. Then its text matches Table M exactly, with the placeholders filled (I/D)
- **V-47 — FUN-043, FUN-027, FUN-022:** Given viewports 320 × 520, 320 × 568, 360 × 480, 360 × 640, 375 × 667, and 390 × 500, with the name "寂庭 Garden of Silence". Then there is no horizontal or vertical page scrolling, the menu is at the top-right, the four chips share one row, the name fits at 12 px or larger (on its own row at 320 px), the garden and details line are fully visible, and every control has a 40 × 40 touch target (T)
- **V-48 — FUN-028, FUN-038, FUN-033, FUN-007:** Given the timing in Table Z. Then a 450 ms press does not open the options sheet and a 550 ms press does; a message is gone about 2 s after it appears; a 25 px swipe does not switch views and a 35 px swipe does; a 2 u drag with Move does not move the object (T)
- **V-45 — FUN-003, FUN-004, FUN-005:** Given a single rake stroke whose band just misses a cell's center. Then that cell is not raked and R does not count it; a band over the center marks it raked (T)
- **V-22 — FUN-001:** Given page at 320 px wide. Then whole garden visible, no horizontal page scroll, touch maps to the touched spot (D)
- **V-31 — FUN-030, INV-005:** Given portrait phone layout active from page load. Then starting garden has about 40% fewer leaves than on desktop (about 13–16 instead of 20–26); weeds and leaves are drawn 1.5× larger; daily drops average 60% of the §3.4 counts (T/D)
- **V-32 — FUN-031:** Given phone (coarse pointer, 390 × 844 screen). When loaded sideways; Continue anyway; still sideways; turned upright; turned sideways again. Then notice shown, hidden, stays hidden, hidden, shown again; garden unchanged throughout (T)
- **V-28 — FUN-027:** Given phone portrait, 375 × 812. Then tools show as 2 rows of 4 without Move; first header row is title, name, badge, menu (menu at the right edge); view switch centered just above the garden, still shown in notes view (D)
- **V-30 — FUN-029, FUN-027:** Given viewports 375 × 812, 412 × 915, 1280 × 800; name "寂庭 Garden of Silence". Then menu button at the right end of the first header row; the whole name visible at 12 px or larger; no horizontal scroll (T)
- **V-29 — FUN-028:** Given any layout. When gardener long-presses Ripple, picks 7; long-presses Place, picks Stone and Large. Then a sheet opens each time with the tool's options; the choices are active afterwards; no context menu appears (D/T)
- **V-23 — FUN-022, FUN-017, FUN-006:** Given phone with touch only. When gardener rakes, ripples, smooths, places (press, drag, release), waters, prunes, picks, removes. Then each works with one finger; the page does not scroll while touching the garden; tapped plant details stay after lifting; no shortcut hints shown (D)
- **V-24 — FUN-022, FUN-017:** Given desktop with mouse only. When gardener performs every tool action and uses every button. Then each works without the keyboard; hovering a plant shows its details (D)

## Open Issues

### OI-001 — Detecting silent download failure

Status: TBD

Some hosts accept `save()` without error but never deliver the file. Whether the page should always also show the INT-002 dialog, or trust a non-failing shim, is undecided. Current requirement: trust the shim (INT-001).

## 8. Notes and Rationale

This section is informative.

- Versioning policy for this spec: while `status` is `draft`, a change that can make a previously conforming implementation non-conforming raises MINOR (1.x.0) instead of MAJOR, and additive or clarifying changes raise PATCH. This deliberately departs from SPEC.md Core §12, which calls such changes MAJOR. Core's MAJOR rule applies from the first release with `status: production`. Versions 2.0.0–13.0.0 were renumbered 1.2.0–1.13.0 under this policy (4.1.0 became 1.4.1); nothing else in them changed.
- 1.16.0 is an additive change by content (a new 0.5× speed; 1× stays the default), recorded as a MINOR bump because the speed control's options and wording (Table M) changed, so a 1.15.0 build no longer matches FUN-013.
- 1.15.0 was a breaking change: it fixes the wording of every text the Gardener sees (Table M, Table T now normative, FUN-044), the timing of long presses, messages, swipes, and drags (Table Z), and the small-screen rules (FUN-043): support down to 320 px wide (fitting without scrolling from 480 px tall, or 520 px below 360 px wide), the name on its own row below 360 px, a garden that shrinks on short screens, and 40 × 40 touch targets for every control (FUN-022). It adds an informative first-minute walkthrough (§3.8).
- 1.14.0 was a breaking change: it specifies the experience that implementations had been left to guess. It adds the layouts and the one-screen web intent (§3.7, FUN-032), phone views (FUN-033), the garden menu (FUN-034), the Tranquility badge (FUN-035), always-visible options on wider screens (FUN-036), guidance (FUN-037), action feedback (FUN-038), replacement warnings (FUN-039), time during dialogs (FUN-040), pointer affordances (FUN-041), and reduced motion (FUN-042). It also defines when a cell is covered, which affects Tranquility, and amends FUN-007, FUN-012, and FUN-016. Choices that were not already made in the game: the empty-garden note (FUN-016, FUN-037), time running during dialogs, and no unsaved-changes warning.
- 1.13.4 was a PATCH: Table K's header separator row, lost in the 1.13.2 markup conversion, was restored so the table renders. No requirement changed.
- 1.13.3 was a PATCH: acceptance checks V-33 (FUN-009) and V-34 (FUN-014) were added, and V-10 now cites INV-003, which it already exercised. No requirement changed.
- 1.13.2 was a PATCH: requirement and invariant IDs now use the `**ID:**` form and acceptance checks the `**V-xx — IDs:**` form, so tools can read them; §5 is split into one subsection per external interface, and its notes heading drops "(informative)" (the section still says it is informative). No requirement changed.
- 1.13.1 was a PATCH: the informative host table in §5 no longer names a specific host site. No requirement changed.
- 1.13.0 was a breaking change because the phone layout (FUN-027) now applies to every viewport up to 640 px wide, with no portrait or height condition, removing the last landscape distinction; a 1.12.0 implementation could differ on a small phone held sideways after its notice is dismissed.
- 1.12.0 was a breaking change because phones held sideways no longer get their own layout; they get a turn-upright notice instead (FUN-031).
- 1.11.0 was a breaking change because the portrait phone layout drops 40% fewer leaves and caps them at 120 (FUN-030, INV-005), and draws weeds and leaves 1.5× larger. The header title no longer shows the Latin "Karesansui" (FUN-029).
- 1.10.0 was a breaking change because the portrait header changed: the badge and menu sit top-right and the view switch moved to a centered row above the garden (FUN-027); FUN-029 fixes the menu at the top-right in every layout.
- 1.9.0 was a breaking change because the portrait phone layout no longer offers Move (FUN-002, FUN-022, FUN-027); a long press on a tool now opens its options sheet (FUN-028).
- 1.8.0 was a breaking change that removed FUN-025 (below) because it trapped Gardeners whose phones had rotation lock on.
- 1.7.0 was a breaking change that let the FUN-025 prompt be dismissed after 5 seconds. (Removed in 1.8.0.)
- 1.6.0 was a breaking change that added FUN-025, a prompt asking portrait phones to rotate to landscape. (Removed in 1.8.0; FUN-025 is retired.)
- P and G default to 1 when there is nothing to measure, so a classic stones-and-sand garden with no plants can reach full Tranquility.
- The garden file embeds the sand as an image rather than a stroke history so its size stays bounded no matter how long the Gardener rakes.
- The sample garden opens in Autumn so the first view shows seasonal color, falling leaves, and real work to do.
- 1.5.0 was a breaking change because declining the host's save prompt no longer opens the fallback dialog (INT-001), and pictures are named like garden files, keeping kanji (INT-004).
- 1.4.1 was an additive change: the Suggest another control is no longer required (FUN-023). Implementations that still offer it remain conforming.
- 1.4.0 was a breaking change because new and sample gardens are now named from Table N instead of fixed names (FUN-019, FUN-020, FUN-024).
- 1.3.0 was a breaking change because saved files now end in `.garden` instead of `.zengarden.json`, and saving asks for a name first (FUN-023).
- 1.2.0 was a breaking change because FUN-006 now places on release instead of on press, so a 1.0.0 implementation could stop conforming. OI-002 is closed: keyboard-only play is a non-goal, and touch and mouse are required (FUN-022).
