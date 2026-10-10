# Plume — the Nidus design language

**Version: Plume 2.6** · Base: Material 3 Expressive, and past it

This file defines how Nidus should look, move and feel. It is the target, not
a description of the app as it is today. Most of the current UI was built
before Plume 2 and is **legacy**: it sets no standard, and nothing in it is a
model to copy. Each surface moves to Plume when it is rebuilt, one at a time,
when the user asks for it. The ledger at the end lists what is done.

> A nidus is a nest. A plume is the feather it is built from: light,
> exact, and alive to the smallest touch of air.

---

## 0. What Plume is

Plume starts from Material 3 Expressive (dynamic colour, the shape library,
spring physics, emphasized type, the new components) and **aims past it**.
Stock M3 Expressive is the floor. The bar is an app whose controls feel more
fluid than any M3E app the user has seen.

Plume 1 got the grammar right and still looked dated: the light theme read
as a photo negative of the dark one, the type was 2021 Material You, card
corners were small and buttons thin, and following a rulebook to the letter
produced surfaces that were correct and stiff. Plume 2 fixes the cause of
each one. §1 says what made the difference; the rest of the file builds on
it.

### How to read this file

Plume has three kinds of content, and each one binds differently:

- **Goals** (§1, §2, and the first lines of every section) say what a
  surface must *feel* like. They are the point. Meet them.
- **Hard rules** are marked **Rule** or sit in a "never" list. They come
  from real bugs and from the user's corrections. Don't break them, and
  don't re-learn them.
- **Defaults** are the numbers: radii, sizes, spring constants, tones. Use
  them unless you have a better answer. A surface **may depart from a
  default** when the result feels better *and* passes the quality bar (§13).
  When it does, write down what you invented in §14 so it becomes reusable.

**Be inventive in how a thing is built.** The best motion in the user's
apps came from a model that was given a clear goal and free rein on the
implementation, not a recipe. Read the goal, then design the whole moment:
what moves, what it is made of, what caused it, which way it goes, what
the finger feels. A surface that only ticks the boxes is not done.

---

## 1. What made the difference

These are the findings behind Plume 2, measured on the real apps
(2026-10-10). Each one is now a section of this file.

1. **Light comes from above, in both themes** (§3). A surface closer to the
   user is *lighter*: the page is lowest, cards float above it, menus and
   sheets above those. Plume 1 used M3's stock surface ladder, where a
   "higher" container in the light theme is *darker*, so cards sank into a
   near-white page while in dark they rose. The depth flipped between
   themes and the light theme read as inverted, glaring and hard on the
   eyes. The measured gap was also tiny (card vs page: −0.017 lightness in
   light, +0.036 in dark). Plume 2 keeps one physical model in both themes
   and a clear gap.
2. **Big areas are calm; colour is spent on small things** (§3). Plume 1
   filled its hero card with `primary-container`: in light, a large
   pastel-blue block. Colour on large surfaces is what makes a light theme
   loud. Hero surfaces are neutral and lifted; colour lives on the action,
   the indicator, the shape.
3. **The typeface is the era** (§4). Roboto Flex says 2021. Google Sans
   Flex, with its width and roundness axes, is the current Android voice.
   Expressive numbers (wider, rounded, heavier, tightly tracked) carry
   more character than any decoration.
4. **Generous shape and real button sizes** (§5). Card-sized surfaces with
   16px corners and 40px buttons as a screen's main action read as old.
   Cards take 24px, hero cards 32px, and a screen's main actions are 56px
   tall.
5. **Motion is an ensemble with one cause** (§6). The most fluid control the
   user has seen (described in §2) is not one animation: five layers
   answer the same tap together, move the same way, and settle on real
   springs with a barely visible overshoot.
6. **The first frame never waits** (§6). That same control lags because the
   app builds the new page *before* the pill can move. Motion starts on the
   frame the finger lands; heavy work comes after.

---

## 2. The feel: the reference moment

The user's north star is a two-option view switcher (a pill that slides
between "Insights" and "Anki" in their flashcard app). It is a reference
for **feel only**. Nothing in Nidus should copy its code or look, and it is
not perfect. Filmed frame by frame, this is what it does:

1. **The pill is a physical object.** It travels on a spring generated from
   stiffness and damping (380 / 0.8): it shoots across, passes its target
   by about 1%, and settles back, all in ≈430ms. The overshoot is barely
   visible and it is what makes the motion feel alive.
2. **It gives under the finger.** While pressed, its corners square off
   (24 → 14px) on a fast spring, and spring back on release.
3. **The labels change because the pill reached them.** The label's colour
   and its icon's fill (outline → filled) change as the pill slides under
   them, on an effects spring. They don't change on their own schedule.
4. **The page follows the pill.** The incoming content slides 28px in from
   the side the pill moved toward and fades in. Go back and both move the
   other way. One direction for the whole gesture.
5. **Data arrives alive.** The numbers count up, and a short haptic tick
   (6ms) confirms the commit.

And the flaw: tapping rebuilds the whole statistics page synchronously,
*then* the browser can paint, so the pill freezes for the build time (40ms
with an empty collection, more with a real one) before it moves.

What to take from it, for every Plume surface:

- **One cause, many answers.** A single input drives several layers at
  once: the thing touched, its label, its icon, the content it controls,
  the haptic. Design the ensemble, not one tween.
- **Every change has a direction and a source.** Content enters from the
  side the indicator went, grows out of what was tapped, returns into it.
- **Physical, not scripted.** Real springs, small overshoot on travel,
  none on colour.
- **Immediate.** The first moving frame is the one after the finger lands.

### Questions to ask before building anything

- If this were a physical object, what would it be made of and how would
  it move?
- What caused this change, and from which direction did it come?
- What does the finger feel in the first 16ms?
- What stays perfectly still so that the moving part reads?
- What else on screen should answer the same touch?
- Does it look like the same object in light and in dark?

---

## 3. Light and colour

**Goal:** both themes look like the same object under different light.
Calm at rest, easy on the eyes for hours of reading, with colour that
means something wherever it appears.

### The lighting model (Rule)

Light comes from above. **A surface closer to the user is lighter, in both
themes.** Recessed things are darker than what holds them.

| Level | Plume role | What lives there | Light (OKLCH L) | Dark (OKLCH L) |
|---|---|---|---|---|
| Page | `--p-page` | The screen background, behind everything | 0.955–0.965, a hint of the seed hue (chroma ≈ 0.01) | 0.13–0.15 |
| Bar | `--p-bar` | Navigation bar, rail | between page and card | between page and card |
| Card | `--p-card` | Cards, list rows, hero card, stat tiles | 0.99–1.0 | 0.20–0.22 |
| Card, raised | `--p-card-hi` | A raised thing inside a card (a selected tile, a pressed row's lift) | 1.0, separated by shadow or edge | 0.25–0.27 |
| Well | `--p-well` | Recessed things: text fields, progress and slider tracks, chart backgrounds, a switch track, inactive segments of a button group | slightly below its holder (≈ page tone) | slightly below its holder (≈ 0.16–0.18) |
| Float | `--p-float` | Menus, sheets, dialogs, snackbars excluded (they are inverse) | 1.0 + Plume shadow | 0.27–0.30 + dark shadow |

- **Minimum gap** between page and card: **ΔL ≥ 0.035 in light, ≥ 0.06 in
  dark**, measured on computed colours. Below that, cards dissolve.
- The page is **never pure white** in light. White is reserved for what
  floats on it. Less white area means less glare.
- The page carries a faint tint of the seed hue in light; neutrals in dark
  stay nearly grey. Text is `on-surface` (≈ 0.22 light / 0.92 dark), never
  pure black or pure white.
- These roles are **generated by the colour engine** (`M3C`, which already
  writes the `--md-*` roles from one seed hue and the contrast setting),
  from the same neutral palette. Never hand-typed per theme.
- `--md-surface-container-*` keep their M3 meaning for the engine, but a
  Plume surface uses the **lighting roles** above for depth, because in
  M3's light scheme a "higher" container is darker, which is the
  inversion Plume 2 removes.

### Colour budget

- **Large surfaces are neutral.** The page, cards, the hero card, sheets.
  A hero earns attention with size, shape, type and the one filled
  action, not with a coloured fill.
- **Colour lives on small things:** the hero action (`primary`), the
  travelling indicator, a selected chip or segment (`secondary-container`),
  a leading icon's tonal shape, a progress ring, a verdict badge.
- A **container fill** (`primary-container`, `tertiary-container`, …) may
  cover something the size of a chip, a button or a list row's leading
  shape. Never a whole card in light.
- Leading icons in a list may carry meaning by hue (one category, one
  hue), drawn as a tonal shape generated by the engine from that hue,
  readable in both themes. Optional, and only where the hue means
  something.
- `tertiary` is for celebration only (§10). Verdict colours (`success`,
  `error`, `warning`) are for verdicts and status only.
- **Rule: never fill a card that holds reading text with a verdict
  container.** `error-container` (deep red in dark, loud salmon in light)
  is painful to read through. A graded option stays a reading surface in
  `on-surface` text: `color-mix(in oklab, var(--md-error) 9%, <the card>)`
  plus a 1.5px inset edge of `error` at about 42%. The badge keeps the full
  colour. (Player wrong option, v6.39, after the user found it painful.)
- **Rule: no theme swaps by hand.** A `[data-theme="light"]` override that
  maps a role to its opposite (a dark container as a light-theme fill, a
  light logo glyph in dark) is how an inverted light theme is born. A role
  means the same thing in both themes; only the engine changes its tone.

### State layers and contrast

- State layers are the element's own "on" colour at fixed opacity: hover
  8%, focus 10%, press 10%, drag 16%. Never invent a hover colour.
- Body text at least 4.5:1, large text and icons at least 3:1, in both
  themes, every seed hue and every contrast setting.

### Shadows

- Depth comes from **tone** first. Shadows belong only to what truly floats
  (FAB, menus, sheets, dialogs, a dragged item, a floating toolbar).
- A **Plume shadow** is two soft layers tinted with the primary hue in
  light: `0 1px 2px color-mix(in oklab, var(--md-shadow) 22%, transparent),
  0 6px 20px -4px color-mix(in oklab, var(--md-primary) 16%,
  color-mix(in oklab, var(--md-shadow) 26%, transparent))`. In dark it is
  darker and untinted, and tone does most of the work.
- A lifted or dragged item rises one level: fade between two pre-drawn
  shadow layers, scale 1.02.
- Scrims: `--md-scrim` at 32% (50% in dark), fading on `--p-effect-slow`. A
  backdrop blur of 2–6px is allowed behind sheets and dialogs if it stays
  smooth.

---

## 4. Type

**Goal:** the type itself says "now". Calm and very readable for long
text, with numbers that have real character.

- **Typeface: Google Sans Flex** (variable: `wght`, `wdth`, `opsz`, `ROND`),
  loaded from Google Fonts with all four axes, `display=swap`, falling back
  to Roboto Flex, then the system UI font. It is the UI face everywhere,
  including bars, buttons, chips and titles.
- **Mono** (Roboto Mono) only for things that are codes: IDs, hashes, the
  copyable image ID. Counts, timers and scores use the UI face with
  `font-variant-numeric: tabular-nums`, so they keep its character.
- **Expressive numbers** (a hero count, a score, a streak, a stat tile's
  value): weight 500–560, `"wdth"` 110–120, `"ROND"` 100, tracking −0.02 to
  −0.035em, line-height ≈ 1.02, tabular figures. A unit beside the number
  ("d", "min", "%") is ≈ 0.42–0.6em, back at `"wdth"` 100, and a step quieter
  in colour. Numbers count up when they arrive (§6).
- **Screen titles** are large and light: 32–45px, weight 450–500, tracking
  −0.01em. Emphasis is weight and width, not more size.
- **Labels** (buttons, tabs, chips): weight 550–600, 14–15px. A selected
  item's label may animate its weight or width (`font-variation-settings`
  on `--p-effect`).
- **Section headings** are rare. Groups of cards separate by space (24–32px)
  first. When a heading is needed it is Title S in `on-surface-variant`,
  sentence case. Never a small coloured (primary) label over every group:
  that is the 2021 settings look.
- **Reading text follows the reader's settings** (`--qfont`, `--qlh`,
  `--qfam`, `--measure`). Plume never overrides them. The "Sans" reading
  family is the UI face. Measure stays within 60–75ch.

| Role | Size / line | Weight | Use |
|---|---|---|---|
| Display | 45–64 / 1.02–1.15 | 450–560, expressive axes | One hero number or title per screen |
| Headline L / M / S | 32/40 · 28/36 · 24/32 | 450–500 | Screen and sheet titles |
| Title L / M / S | 22/28 · 16/24 · 14/20 | 500–550 | Card titles, row titles |
| Body L / M / S | 16/24 · 14/20 · 12/16 | 400 | UI text |
| Label L / M / S | 15/20 · 12/16 · 11/16 | 550–600 | Buttons, chips, tabs |

Sentence case everywhere. No ALL-CAPS except tiny overlines (+0.08em).

---

## 5. Shape

**Goal:** soft, generous, confident shapes, and shape that changes with
state, so a pressed or selected thing looks different even in greyscale.

### Scale

| Token | Value | Use |
|---|---|---|
| `--p-r-xs` | 4px | Inner joints of a grouped list or button group, tags |
| `--p-r-s` | 8px | Chips, small inputs, tooltips |
| `--p-r-m` | 12px | What a control squares to while pressed, fields, menu rows |
| `--p-r-l` | 16px | Things inside a card: an image, an inner tile |
| `--p-r-lx` | 20px | Floating toolbars, small cards in a dense grid |
| `--p-r-card` | 24px | **Cards and list groups** (the default for any card) |
| `--p-r-xl` | 28px | Sheets, dialogs, menus |
| `--p-r-hero` | 32px | The hero card of a screen |
| `--p-r-xxl` | 48px | Display surfaces, hero imagery |
| `--p-r-full` | 999px | Buttons at rest, FABs, pills, indicators, search fields |

- **Rule: no card-sized surface below 20px.** 16px corners on a whole card
  are the Plume 1 look.
- **Concentric corners.** Something nested in a rounded box with an inset
  of *p* takes a radius of about *outer − p* (never below 8px), so the
  two curves run parallel.
- **Grouped lists:** rows are separate cards, `--p-r-card` at the ends of a
  group and `--p-r-xs` between them, with 2px gaps of page tone, never
  divider lines. While a row is pressed, its corners round up.
- **Connected button groups:** outer corners full, inner `--p-r-xs`. The
  pressed or selected member takes full corners and pushes its neighbours
  by a few pixels (`transform` only).

### Shape in motion

- **Press squares the corners.** A pill on `pointerdown` morphs its radius
  toward `--p-r-m` (fast effects spring), and springs back on release.
  Small icon buttons go from round to `--p-r-m`. Never to 0.
- **Selected changes form.** A toggle that turns on goes round → squircle
  (or the reverse, consistently across the app).
- **Expressive shapes** (the M3E library: cookie, sunny, clover, gem, pill,
  soft burst, …) are reserved for meaning: avatars, an empty state, the
  loading indicator, a correct-answer badge, a celebration. Draw them as
  SVG paths with matching command counts so they can morph, or as
  `clip-path: path()`.
- Where supported, `corner-shape: squircle` may refine large cards and
  sheets (`@supports`), with the plain radius as fallback.
- **Radius never takes a spring that overshoots.** An overshooting radius
  can dip below its target. Animate corners on an effects spring.
- **Rule: a shape whose corners morph rests at its real radius, never at
  `--p-r-full`.** A pill's radius is half its height (a 40px pill is 20px,
  a 30px dot 15px). From 999px, nothing visible changes until the radius
  drops below half the height, so a press squares the corners in the last
  two frames and a release snaps them back in one. `--p-r-full` is only for
  shapes whose corners never animate. (The Settings button groups, seeds,
  steppers and icon buttons, v6.58; the user found the long-press on the
  chosen theme option jerky. Fixed in v6.60.)
- **The pressed choice presses its indicator too.** In a button group, the
  indicator under the chosen option squeezes with its label (`scale`, as
  the touch contract) while its corners square; the label never shrinks
  inside a pill that stays still.

---

## 6. Motion: the Plume physics

**Goal:** every touch answers at once, physically, and every change shows
where it came from. Quiet at rest, alive in touch: nothing idles or loops
except a loading indicator while something really loads.

All motion is **spring physics**, written in CSS as `linear()` easings
generated from stiffness and damping (mass 1), with the duration set to
the spring's settling time.

### Butter: nothing changes plainly (Rule)

The whole app should feel like butter. **No visible change happens in a
single frame.** Anything that appears, leaves, moves, resizes, changes
colour, changes value or swaps its content reaches its new state through a
transition that shows where it came from and where it went: a number
rolls or counts to its new value, a label crossfades, a list makes room
before an item lands, a panel opens out of its trigger, a removed item
collapses its gap, a badge grows in, a disabled button eases into
being enabled. If you can see the before and the after, there is motion
between them.

- This includes the small things that are easiest to forget: a counter's
  text, a toggled icon, a chip's checked state, an error line appearing
  under a field, a button's label changing, a re-sorted list.
- Being smooth never means being slow. Small changes settle in 130–330ms;
  the user should never wait for an animation to finish before acting
  (every motion retargets mid-flight).
- The only things that don't move: text while it is being read (motion
  never shifts a paragraph under the eye) and anything under reduced
  motion beyond a short crossfade (which is still a transition).

### Two kinds of spring

- **Spatial springs** move things: position, scale, size, rotation. They
  may overshoot a little, which is what makes things feel physical.
- **Effects springs** change appearance: opacity, colour, corner radius,
  blur, shadow. They never overshoot.

### The six springs (defaults)

| Token | Stiffness / damping ratio | Settles in | Overshoot | Use |
|---|---|---|---|---|
| `--p-spatial-fast` | 800 / 0.6 | 330ms | 9.5% | Small things: switch thumbs, checkmarks, press release, chips, badges |
| `--p-spatial` | 380 / 0.8 | 380ms | 1.5% | Medium things: indicators that travel, menus, cards, FAB menu |
| `--p-spatial-slow` | 200 / 0.8 | 510ms | 1.5% | Large things: sheets, dialogs, full-screen transitions |
| `--p-effect-fast` | 3800 / 1.0 | 130ms | 0 | Press state, state layers, radius under the finger |
| `--p-effect` | 1600 / 1.0 | 190ms | 0 | Colour and opacity changes on selection |
| `--p-effect-slow` | 800 / 1.0 | 270ms | 0 | Fades of large surfaces, scrims |

Each token is a pair: `--p-spatial` (the `linear()` curve) and
`--p-spatial-d` (its settling time, to within 0.5%). They live on `:root`
in the `Plume 1.0 · tokens` block of `index.html`, generated by a spring
sampler (never hand-typed). Script reads them with `PL('spatial')` →
`{e: easing, d: ms}`, so CSS and JS can never drift apart. Use the
**standard scheme** (stiffness 1400 / 700 / 300 at damping 0.9) for utility
motion that should not draw the eye, like a scroll-linked header or a
reflowing list. A surface may tune its own spring when a default doesn't
feel right; generate it with the same sampler and record it in §14.

### The ensemble (Goal)

A meaningful touch is answered by **several layers at once**, all caused by
the same input and moving the same way:

- the thing touched (squeeze on press, spring on release);
- the indicator that travels to it (one shape, never two crossfading);
- its label and icon (colour and fill change as the indicator arrives);
- the content it controls (enters from the side the indicator went, or
  grows out of what was tapped);
- data inside that content (counts up, draws itself);
- a haptic tick on commit, where supported.

Not every touch needs all six. A view switch, a tab, choosing an answer,
opening a card and finishing a block do.

### The first frame (Rule)

**The first moving frame is the one after the finger lands.** Nothing a
touch triggers may wait for rendering, data or layout.

- On `pointerdown` the press response starts (touch contract below). On
  commit, write the state that drives the motion (a class or attribute)
  and **let the browser paint before any heavy work**: yield
  (`await new Promise(requestAnimationFrame)` then a task, or
  `scheduler.yield()`), then build.
- **Never re-render the control that is animating.** It stays in the DOM
  and its transition runs; only the content it controls is rebuilt.
- The incoming content may be built during the motion and arrive when it
  is ready, with its own entrance. The indicator never waits for it.
- Build only what changed. A large list or a page of statistics renders in
  slices across frames, or once, off screen, before it is needed.
- Measured, not felt: on a seeded bank of realistic size (≥ 2,000
  questions), sample the indicator's transform every animation frame after
  the tap. It must already have moved in the first or second frame
  (`CLAUDE.md`, Testing).

### The touch contract (every interactive element)

| Moment | Response |
|---|---|
| `pointerdown` | Within one frame: scale to **0.96** (0.92 for small icon buttons, 0.98 for cards), corners square toward `--p-r-m`, press state layer at 10%. `--p-effect-fast` |
| Hold | Stays compressed. No timers |
| Release / click | Springs back to 1 on `--p-spatial-fast` (the small overshoot *is* the feel), and the action starts in the same frame |
| Hover (`@media (hover:hover)` only) | 8% state layer and, on cards, one tonal step lighter. `--p-effect` |
| Focus-visible | 3px `primary` ring offset 2px, following the element's current shape |
| Cancelled press (finger slides off) | Springs back with no action |
| Disabled | 38% content opacity, no response, no cursor change |

Pressing never shifts layout: `transform` only, never margin, padding or
size.

### Icons move by their own logic (Rule)

**Every icon has a logic you can read from its shape, and its motion acts
that logic out.** An icon is a picture of a thing; when its action
happens, the thing does what it would really do. A bounce, a squash, a
stretch, a wobble or a spin of the whole glyph is not icon motion: it is
the same gesture stuck on every icon, it reads as childish and it says
nothing. (That is what the flashcard app does today: one table hands out
a whole-glyph tilt, scale or nudge by icon category, so a bin, a pin and
a flag all just lean. Don't repeat it.)

How to design one:

1. **Read the glyph.** What object is it? What are its parts (a lid and a
   body, two sheets, an arrow and a tray, a knob on a track, a hand on a
   dial)? Which part is fixed and which moves, and around what pivot?
2. **Say the motion in one sentence, as what the object does.** "The lid
   lifts on its hinge and drops shut." "The front sheet slides off the
   one behind." If the only sentence you can write is "it bounces", it
   fails.
3. **Move the parts, not the picture.** Draw the icon as inline SVG with
   each moving part its own path or group, `transform-box: fill-box`, and
   `transform-origin` at the real pivot (the hinge, the axle, the centre
   of the arc). Strokes that appear draw along their own direction
   (`stroke-dashoffset`); shapes that change form morph between paths
   with matching commands.
4. **Tie it to the meaning.** It plays when its action happens or its
   state changes (tap, start, finish, on/off), never idly. A state icon
   moves to its new state and stays there; an action icon plays its
   action and comes to rest.
5. **Keep it adult.** Small amplitude, physical springs (spatial for
   parts that travel, effects for strokes and fills), overshoot only
   where the real object would have it (a lid settling, a needle). Most
   icon motions finish within 250–450ms.

Examples from Nidus's own icons (directions, not specs; find the best
version of each):

| Icon | Its logic, acted out |
|---|---|
| Sync (arrows on a circle) | While syncing, the arrows travel along their own arc, which is what they depict; on finishing they glide to rest at their drawn position. Sync off: the slash draws across the cloud along its length, and draws back off when sync returns |
| Import / download (arrow into a tray) | The arrow drops into the tray, the tray dips slightly under it and recovers, and a fresh arrow slides in from above |
| Copy (two sheets) | The front sheet slides off the back one along their offset, then the pair becomes a check that draws itself |
| Delete (bin) | The lid lifts on its hinge, tilts open, and closes with a small settle |
| Review (clock with an arrow running back) | The arrow sweeps backwards around the dial and the hands turn back with it |
| Bank (archive box) | The lid lifts off the box and settles back |
| Images (frame with mountains and a sun) | The sun rises a little behind the mountains |
| Progress (bars) | The bars grow from their baseline one after another, to their own heights |
| Settings / tune (knobs on tracks) | Each knob slides along its own track to a new position, in opposite directions |
| Search (magnifier) | The lens sweeps a short arc, as if scanning, pivoting at the handle's end |
| Theme (moon ↔ sun) | The moon's shadow slides off to reveal the sun's disc and the rays extend outward one by one; the reverse eclipses it |
| Expand / collapse (chevron) | It turns 180° about its centre, in step with the panel it opens |
| Show / hide (eye) | The lid closes over the eye and the slash draws in |
| Play ↔ pause | The two bars morph into the triangle and back (matched paths) |
| Check | The stroke draws in its writing direction, short leg first |
| Focus mode (four corners) | Each corner turns 180° about its own centre (the pattern already built for `#pFull`, §14) |

- **A noun with no action** (a static label icon) does not invent motion;
  its motion is its state change: outline → filled on selection.
- **One icon, one motion, everywhere.** Each designed icon motion is
  recorded in §14 and reused; the same icon never moves two ways.
- Interruptible like everything else. Under reduced motion, an icon keeps
  its fill and stroke changes and drops the travel.

### Choreography

- **Arrival:** surfaces grow from their origin with `transform-origin` set
  to the anchor: scale 0.85→1 plus fade on `--p-spatial`. Their contents
  follow 30–40ms later, staggered by 25–35ms per item, six steps at most,
  in reading order.
- **Departure:** about 30% faster than arrival, with no stagger, shrinking
  back toward the origin. Remove the element only after the exit animation
  has actually finished (measured, never a guessed timer).
- **Rule: a surface returning into its origin hands off to it; it never
  covers it.** As it lands it fades out so the real origin shows through,
  and it is removed as soon as it is invisible, not when a spring's long
  tail settles. (Settings, v6.58: the closing section landed on its row as
  a blank card-coloured tile and waited out the spring, so the row the user
  had tapped looked missing for about a quarter of a second, then popped
  in. Fixed in v6.59.)
- **Peer screens:** fade-through. The old screen fades out (≈90ms), then the
  new one fades in with a 12px rise on `--p-spatial`. They never overlap.
- **A screen opened from a card** (Resume, Companion on Home) grows out of
  that card (`plumeOpen`). The old screen stays visible underneath until the
  new one has landed. **Never a blank frame**, and the new screen arrives
  as one piece: its own entrance animations must not trickle in parts
  (bar first, then content, then a bottom bar) under the transform.
- **Messages wait for motion.** A snackbar triggered by opening a screen
  appears after the screen has landed, never mid-transition.
- **Hierarchy (list → detail): the page slides over (Rule).** The detail
  page comes in from the right edge as a whole page with its leading
  corners rounded while it travels and a soft shadow on its edge; the list
  underneath drifts left about a fifth of the width and dims under a scrim.
  Back reverses it, a third quicker. One spring stepped per frame drives
  every layer, so a tap, Back or a finger dragging the page to the right
  (1:1, close past ~40% or on a flick, else spring home) takes it over
  mid-flight. **Never a container transform or a grow-from-the-row reveal
  for a page:** the user found it dated ("from 2017") and its return
  covered the row (Settings, v6.58–6.60). Existing grow-from-card openings
  (`plumeOpen` on Home) stay until the user asks about them.
- **Shared indicators travel.** One active pill in a nav bar, tabs or
  segmented control slides and stretches to the new item
  (`translateX` + `scaleX` on `--p-spatial`). Never crossfade two
  indicators.
- **Layout changes glide.** When an item is added, removed or reordered, the
  rest move to their new places with FLIP (WAAPI, `--p-spatial`). Nothing
  jumps.
- **Data draws itself.** Numbers count up (tabular figures), rings run,
  bars grow from their base. On a re-render, only the difference animates.

### Gestures

- Sheets, the drawer and swipe-able rows follow the finger 1:1. Turn off
  transitions while dragging.
- On release, the target is chosen by position *and* velocity (a flick
  counts), and the motion continues with that velocity on `--p-spatial`.
- Past a threshold: resistance (rubber band, 0.55× movement), then a light
  haptic tick where supported.
- **Pinch to zoom a picture** (the figure viewer): the stage takes every
  touch (`touch-action:none`) and the picture moves by `transform` alone.
  Two fingers scale about their midpoint and pan at once, exactly under
  the fingers; one finger pans only once zoomed (at fit it is the swipe to
  the next picture). Past an edge, under fit or over the ceiling it gives
  at 0.55×. Every settle is a spring stepped per frame with the
  `--p-spatial` values, never a CSS transition, so a finger can catch it
  mid-flight. Double tap zooms into the tapped point and back.
- Haptics are optional and subtle: `navigator.vibrate(6–10)` on a commit
  (toggle on, gesture past its threshold, answer locked). Behind a setting,
  and a silent no-op where unsupported.

### Mechanics (non-negotiable)

- Animate only `transform`, `opacity`, `filter`, `clip-path`, and colours.
  Never `width`, `height`, `top`, `left`, `margin` or `padding`. Height
  reveals use `grid-template-rows: 0fr → 1fr` with a content fade.
- Name properties in `transition`. Never `transition: all`.
- State that can flip back and forth uses **transitions** (they retarget).
  Keyframes are only for one-shot arrivals and exits.
- Never hide with `display:none` directly. Animate out, then hide. Use
  `inert` on whatever is closed or leaving.
- No `setTimeout` guessing animation lengths. Use `animationend` /
  `transitionend` or `Animation.finished`.
- No linear easing on movement (only on spinners and progress).
- Hover effects live in `@media (hover:hover)` so they don't stick on touch.
- `will-change` only while something is animating, removed afterwards.
- 60fps on a mid-range Android phone is the bar. Large blurs and box-shadow
  animations are suspects. Animate the opacity of a shadow layer instead.

### Layers and keys

- Escape closes **only the top layer**: menu, then dialog or figure viewer,
  then the reader, then nothing. One key press never closes two layers.
- A Plume layer's key handler runs in the **capture phase** and returns
  early while a dialog (`#mask.on`) or the figure viewer (`#lbx` not
  `hidden`) is open. In bubble phase it runs after the dialog has already closed itself,
  and closes the layer underneath too.
- Arrow keys move between items only when focus is not in a text field.
- **Back closes the top full-screen layer, never the app.** Nidus is often
  the only page in its tab, so a phone's Back with no history entry of the
  app's own leaves the app. A full-screen layer (the figure viewer) pushes
  one history entry when it opens (`{lbx:1}` in `history.state`), closes
  on `popstate` when the entry it lands on lacks that key, and takes the
  entry back off (`history.back()`) when it is closed any other way. Decide
  from the state of the entry, never from a counter, so a late Back after a
  quick close and reopen cannot close the wrong thing. A stale key found at
  start-up (a reload while open) is cleared with `replaceState`.
- **Close is always on screen.** Every full-screen layer has its Close (×)
  as the first item of its bar, top left, and the bar keeps two or three
  actions at most; the rest go into a ⋮ menu. Measure at 320–430px that no
  bar control's right edge passes `innerWidth`. (The legacy figure viewer
  put six buttons in a row, and on a 390px phone Close sat at x=405, off
  screen: the user could only leave with Back, which closed the tab.)

### Reduced motion

With `prefers-reduced-motion: reduce`: keep fades, colour changes and the
press state layer. Replace movement, scale and overshoot with a short
crossfade (≈150ms). The app stays fully alive in feel. It just stops
travelling. Never set all durations to 0: that removes the feedback too.

---

## 7. Layout and spacing

**Goal:** airy, ordered and easy to scan; content always readable, nothing
covered, nothing cramped.

- **4px grid.** Spacing steps: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64.
- Screen side margins: 16px compact, 24px medium, 24–32px expanded.
- Inside cards: 20–24px (16px only in dense tiles). Nothing touches an edge.
- Groups of cards separate by 24–32px of page; rows inside a group by 2px.
- Touch targets are at least 48×48px, even when the visual is smaller.
- Window classes (M3): compact < 600px, medium 600–839px, expanded 840px+.
  Navigation: bottom nav bar on compact, rail on medium, rail or drawer on
  expanded.
- **A list → detail screen**: on compact (≤ 860px in Nidus) the detail is a
  full-screen layer that slides over the list (§6, Hierarchy), with its own
  Back entry; wider, both panes show and one marker travels in the list.
- Always check **390px** (phone) and a desktop width. No horizontal scroll,
  ever. Respect `env(safe-area-inset-*)`.
- **Rule: floating things never cover content at rest.** A screen with a
  FAB or floating toolbar ends with bottom padding of at least its height
  plus 16px, so the last card can scroll fully clear. (Home's Import FAB
  sat over the "Import a batch…" card and cut its text, seen 2026-10-10.)
- **A pill beside text is centred on that text.** Its vertical centre sits
  on the centre of the line it shares (the glyph box: ascent plus
  descent), measured, never nudged with `vertical-align: 2px` or aligned
  by baseline. A small label next to large text aligned by baseline sits
  low, and an inline-flex pill whose first child is an icon rides high.
  The text inside a pill is centred in it too. Two pills that label rows
  of one list share one column, so their sentences start on the same
  line. (The lure and "your answer" tags, the Not here / Right in labels
  and the verdict's chips, v6.50, after the user found all of them off.)
- **A highlight keeps the column.** A tinted tile, chip or box inside a
  list or stepper starts on the same left edge as the text around it, and
  its padding goes inward. Never pull it out with a negative margin to
  keep its text aligned: the box edge is what the eye reads, and one that
  juts toward the markers breaks the column. (The chain's answering step,
  v6.48, after the user found it misaligned.)

---

## 8. Components: the Plume standard

When a component is built or rebuilt, it meets this anatomy (defaults; the
goals and rules above win). All follow the touch contract (§6).

| Component | Plume anatomy |
|---|---|
| **Buttons** | Sizes XS 32 / S 40 / M 56 / L 96 / XL 136px high. **A screen's main actions are M (56px)**; buttons in one row share a height. S (40px) only in dense places: inside list rows, toolbars, small cards. Pill at rest, squares on press. Variants: filled (the hero only), tonal, outlined, text, elevated. Leading icon 20px with 8px gap. A loading button morphs its label into the loading indicator without changing width |
| **Icon buttons** | Round 40px visual in a 48px target. Toggle variants change shape and fill when selected. The icon may animate its fill axis (outline → filled) |
| **Button group / view switcher** | One `--p-well` track, `--p-r-full`, 4px inset, and **one** `primary` indicator that travels between segments on `--p-spatial` and squares a little while pressed. Labels and icons change as it arrives. Content follows its direction (§2). Replaces old segmented controls |
| **Split button** | Leading action plus trailing chevron. The chevron half turns into a circle and its icon rotates 180° while its menu is open |
| **FAB / FAB menu** | Only for the screen's hero action, never covering content (§7). FAB menu: the FAB morphs into a close button (rotate and shape morph) while items unfold upward with a stagger, each a `primary-container` pill |
| **Navigation bar** | `--p-bar`, separated from the page by tone, never a line. The active item has a pill indicator that slides between items and stretches mid-travel. Label weight animates. Icons switch outline → filled |
| **Navigation rail / drawer** | Same travelling indicator. The rail can expand into a drawer with a container transform |
| **Top app bar** | Flat on the page at rest, with a large title that may collapse on scroll. Scrolled: one level up in tone with an effects-spring colour transition. No shadow, no line |
| **Floating toolbar** | `--p-float`, `--p-r-full` or `--p-r-lx`, Plume shadow. Hides on scroll down and returns on scroll up (`--p-spatial`, translate only) |
| **Hero card** | One per screen at most. `--p-card`, `--p-r-hero`, neutral fill. Its focal element is an expressive number, a shape or a ring; its colour is the one filled action |
| **Cards** | `--p-card` on `--p-page`, `--p-r-card`. Press compresses to 0.98. A card that opens something uses the container transform. Stat tiles are cards with an expressive number |
| **Lists** | Grouped cards (§5), 56–72px rows, leading icon in a 40px tonal shape, title + one supporting line, trailing meta or chevron |
| **Menus** | `--p-float`, `--p-r-xl`, 4px inner padding, rows `--p-r-m` 48px high, grouped sections with 2px gaps instead of dividers. Grows from its anchor (scale 0.85 + fade, `transform-origin` at the anchor) on `--p-spatial`. Selected row: `secondary-container` with a check |
| **Bottom sheet** | `--p-float`, `--p-r-xl` top corners, a 32×4px drag handle, draggable with velocity and snap points, scrim fade. Rises on `--p-spatial-slow`. Closes on scrim tap, Escape, back gesture, or flick down |
| **Dialog** | `--p-float`, `--p-r-xl`, 24px padding, actions bottom-right as text or tonal buttons. Arrives with scale 0.9 + fade from the centre (or from its trigger). On compact screens prefer a bottom sheet |
| **Snackbar** | `inverse-surface`, `--p-r-s`, slides up 16px + fade, one optional action, auto-dismiss 4–6s, swipe to dismiss. One at a time; a new one replaces the old with a quick crossfade |
| **Chips** | 32px, `--p-r-s`. Selected: `secondary-container`, a leading check that draws itself (stroke-dashoffset), and the chip widens smoothly to fit it |
| **Switch** | 52×32 `--p-well` track. The thumb grows from 16 to 24px when on (28px while pressed), slides on `--p-spatial-fast`, and shows a check icon when on |
| **Checkbox / radio** | Check strokes draw in. The radio dot scales in with a spring. 48px target |
| **Slider** | M3E slider: a tall track with a gap around the handle. The handle is a vertical bar that narrows while dragged. A value label pops above it while dragging |
| **Text field / search** | Filled on `--p-well`, or outlined. Search is a full pill. The label floats on `--p-spatial-fast`. The focus indicator grows from the centre outward |
| **Progress** | Wavy (M3E) for determinate progress worth celebrating (session progress). Flat for utility. Indeterminate: the M3E loading indicator, a shape that morphs through the expressive shapes inside a soft container |
| **Tabs** | Indicator travels (§6). Content follows its direction, or swipes horizontally with the finger on touch |
| **Tooltip** | `inverse-surface`, `--p-r-xs`, appears after 500ms hover or a long press, scale 0.9 + fade |
| **Empty states** | One expressive shape as illustration (a tonal container), one short line, one action |

**Icons:** Material Symbols Rounded geometry on a 24px grid, outline at
rest and filled when selected or active, with the change animated (the
font's `FILL` axis, or matching SVG paths). An icon that moves is drawn as
inline SVG with its moving parts separate, and moves by its own logic
(§6). Stroke icons drawn as inline
SVG keep one consistent weight. No emoji as icons. Icon morphs (copy →
check, play → pause) use paths with matching commands, or a rotate + scale
crossfade.

---

## 9. Rolling out Plume 2

The user moves Nidus to Plume 2 **one surface at a time** (Settings, the
gallery, Home, the player, …). Rebuild only what is asked for.

- **The foundation is app-wide and lands first.** The lighting roles (§3),
  the typeface (§4) and the new radius tokens (§5) are one change across
  the whole app: a lit card on an unlit page can't look right. If the
  ledger (§15) shows the foundation as not landed, the first Plume 2
  request lands it in the same change (and the report says so). It
  includes: the engine generating the lighting roles; `body` and the
  browser chrome colour on `--p-page`; Google Sans Flex loaded as
  `--m3-font`; `--p-r-card` and `--p-r-hero` added; the Plume 1.x
  surfaces re-pointed from `surface-container-*` to the lighting roles;
  every `[data-theme="light"]` role swap removed. Legacy screens get the
  page, font and colour change for free; their layout stays as it is.
- **A surface rebuilt under Plume 2** meets every goal and rule in this
  file, not just its own row in §8, and is filmed and measured (§13).
- Plume 1.x surfaces in the ledger stay valid for their motion and
  behaviour; their colours and type follow the foundation.

---

## 10. Nidus moments (where Plume is most expressive)

These carry the app's character. Every other surface stays calm so these
can shine.

- **Choosing an answer:** the option card compresses, and the letter badge
  morphs circle → squircle and fills with `primary` (`--p-spatial-fast`).
  Changing the choice moves the selection to the new card (one indicator
  that travels where the layout allows).
- **The verdict:** correct: the key badge morphs into the expressive
  "soft burst" shape in `success`, with a single small scale pulse
  (1 → 1.08 → 1). Wrong: the chosen card gives one damped horizontal
  nudge (≈6px, two cycles, `--p-spatial-fast`), its badge turns `error`,
  and the correct option lights up 80ms later. The explanation then rises
  in as a stagger of its sections.
- **Next question:** the content moves out and in as a shared horizontal
  motion following the direction of travel (≈24px + fade). The chrome
  (bars, timer) never moves.
- **Finishing a block:** the score counts up on a Display number. A ring
  runs to its value. If the result earns it, a `tertiary` celebration:
  an expressive shape blooms behind the number, once, never on loop.
- **Streaks and milestones:** the only place `tertiary` is used as a fill.
- **Spoilers stay sealed.** When an answer is hidden (the companion's
  read-along), nothing outside the spoiler may hint at it. No green or red
  chip, badge or tint says whether it was right. "Answered there" is
  neutral, and the chosen option is marked but never graded outside it.

---

## 11. Copy and tone

Plain, calm, second person. Short labels that explain themselves. No
exclamation marks. A helper line only when a control can't explain itself.
Errors say what happened and what is safe ("Nothing was lost").

---

## 12. Never

- Copying a legacy component's styling as a "reference" for a Plume surface,
  or copying another app's code or look (the flashcard app is a reference
  for feel, never a source).
- A card darker than the page it sits on, in either theme. A pure-white
  page in light.
- A large surface filled with colour; a hero card in a container colour.
- A role swapped by hand for one theme.
- Raw colour values, fonts outside §4, one-off radii outside the scale, or
  16px corners on a card-sized surface.
- A 40px button as a screen's main action.
- A small coloured heading over every group of a list.
- A first frame that waits for rendering; re-rendering the control that is
  animating.
- `transition: all`, animating layout properties, timers instead of events.
- Two indicators crossfading where one should travel.
- Things popping in or out without an origin or a direction.
- Shadows used to separate ordinary cards.
- Bounces on colour, opacity or radius (effects never overshoot).
- Idle or looping decoration.
- Any visible change in a single frame: a value, label, icon, colour or
  item that just swaps.
- An icon that bounces, squashes, stretches, wobbles or spins as a whole
  instead of moving its parts by their logic; one generic motion shared
  by many icons.
- Feedback that waits for `click` while the finger is already down.
- A floating button covering content at rest.
- A redesign of a surface the user did not ask for.
- A blank frame, or parts arriving at different times, between two screens.
- One key press closing two layers.
- A colour that gives away a hidden answer.

---

## 13. Definition of done (every Plume change)

The quality bar. A surface is done when it *feels* like §2, and:

- [ ] **Light:** in both themes the page is lower than the cards and cards
      are lighter than the page; the gap is measured on computed colours
      (ΔL ≥ 0.035 light, ≥ 0.06 dark). No large coloured fill.
- [ ] **Parity:** screenshots at 390px in light and dark look like the same
      object in different light, not a negative of each other
      (`htmlcheck --shot`, and a desktop width).
- [ ] **Type and shape:** UI text in Google Sans Flex; numbers expressive
      where they lead; card radii from the scale; main actions 56px.
- [ ] Uses only `--md-*` roles, the lighting roles and `--p-*` tokens. No
      raw values.
- [ ] Every interactive element follows the touch contract (press, release
      spring, hover, focus, cancel, disabled).
- [ ] **Butter:** nothing on the surface changes in a single frame
      (filmed); every value, label, icon and item transitions.
- [ ] **Icons:** every icon that moves passes the one-sentence test (§6)
      and moves its parts, not the whole glyph.
- [ ] **Ensemble:** each meaningful touch is answered by its layers together,
      in one direction.
- [ ] **First frame:** on a bank of ≥ 2,000 questions, the indicator or
      pressed element has moved within two animation frames of the tap
      (logged per frame, `CLAUDE.md` Testing).
- [ ] Arrivals have an origin. Exits finish before removal. Mid-animation
      taps retarget smoothly.
- [ ] No layout jump during or after any animation, and none on first paint.
- [ ] Reduced motion: still alive in feel, no travel.
- [ ] Every new or changed transition **filmed frame by frame** on its real
      trigger (see `CLAUDE.md`, Testing): no blank frame, no part snapping
      in on its own, and it lands where it should.
- [ ] Keyboard: Tab order, `:focus-visible`, Escape closes, Enter/Space
      activates. ARIA roles and states are correct.
- [ ] `htmlcheck` passes with no console errors.
- [ ] The CSS block is marked `/* Plume <ver> · <surface> */`, anything
      invented is written into §14, and the ledger (§15) is updated.

---

## 14. Where Plume lives in `index.html` (reuse before writing new)

- **Foundation (landed in v6.58):** the colour engine (`M3C.scheme`, also
  `window.M3C`) writes the lighting roles `--p-page`, `--p-bar`, `--p-card`,
  `--p-card-hi`, `--p-well`, `--p-float` by OKLCH lightness on the neutral
  hue, and retunes the M3 surface ladder to the same model (`--md-surface`
  = the page, `container-low` = a card, `container-highest` = a well), so
  legacy screens get the same depth without being rebuilt. It also writes
  `--p-cat-a|b|c` / `--p-on-cat-a|b|c`: three category hues (the seed, +75°,
  −105°) as tonal containers for leading icon shapes. The static fallback
  palette at the top of the file is generated from the engine (hue 262,
  tonal, standard contrast); regenerate it whenever the engine changes.
  Google Sans Flex is `--m3-font`; `--p-r-card` (24) and `--p-r-hero` (32)
  are in the tokens block.
- **Tokens:** CSS block `Plume 1.0 · tokens` — radii `--p-r-*`, the six
  springs `--p-*` / `--p-*-d`, `--p-press*`, `--p-stagger`, `--p-exit`
  (the exit curve), `--p-shadow`.
- **Primitives:** CSS block `Plume 1.0 · primitives` — `.pl-ib` (icon
  button), `.pl-btn` (+ `--m`, `--filled`, `--tonal`, `--text`),
  `.pl-chip`, `.pl-tb` (floating toolbar), `.pl-split` (split button),
  `.pl-menu` / `.pl-menu__item`, `.pl-field` + `.pl-help`.
- **Touch contract:** add class `pl-press` to anything tappable. A
  document-level handler sets `.is-down` on `pointerdown` (held at least
  90ms, cancelled when the finger slides off, also on Enter/Space). Style
  the squeeze on `.is-down`; the release spring is the element's own
  base transition.
- **Script helpers:** `PL(name)` reads a spring; `PL_EXIT` is the exit
  curve. `Motion.reduced` is the live reduced-motion flag. Animate with
  WAAPI and wait on `Animation.finished`, never a timer.
- **Patterns built so far** (copy their technique): the container
  transform out of a tile and back into it (`cpqShow` / `cpqHide`, a
  `clip-path: inset()` morph plus a fading tint of the tile's colour);
  the direction-aware page swap (`cpqSwap`); the menu that grows from its
  anchor (`cpqMenu`); the wavy progress drawn to real width (`compWave`);
  count-up figures (`compCount`); the toolbar that tucks away on scroll.
- **Opening a whole screen from a card:** `plumeOpen(dest, card, build,
  after, settle)` pins the screen over Home, grows it out of the card
  (clip-path + the card's colour fading out), brings its content in as one
  piece, and only then hides Home and scrolls to the top. `settle` finishes
  a legacy screen's own CSS entrances so they don't trickle in underneath.
  Anything that closes the screen calls `plumeAbort(dest)` first.
- **A switch inside a menu** (the player's ⋮ menu, "Show clock"): the row
  is a `<label class="tool tmsw">` holding the app's own switch
  (`<input type="checkbox" role="switch">`). The switch shows the state, so
  the row is not filled, and tapping it keeps the menu open so the thumb
  can be seen moving. It writes a `SETTINGS_SPEC` key, so Settings and the
  in-session sheet show the same switch.
- **Icon morph by turning its parts** (focus mode, `#pFull`): when two
  glyphs are the same strokes in different places (expand ↔ shrink are the
  same four corners, each turned 180° about its own centre), draw the icon
  inline and rotate each path (`transform-box: fill-box`) on `--p-spatial`,
  staggered 15ms. Never swap the `<use>`: that cuts. When the icon itself
  shows the state, the button stays unfilled in every state: the user did
  not want the focus button highlighted while fullscreen is on.
- **The compact session bar** (`Plume 1.4 · compact session bar`): from
  520px down the bar's icon buttons are 44px wide (40px below 380px), the
  height stays 48px; the Tutor/Exam chip hides; from 420px the counter's
  chevron hides. Anything added to the bar is measured at 320–430px with
  the widest counter ("100 / 120") before it ships.
- **Pinch, pan and double-tap zoom** (`Plume 1.5 · figure viewer zoom`,
  `FZ` in script): a per-frame spring (stiffness 380, damping 0.8, the
  `--p-spatial` source values; 3800 / 1 under reduced motion) that keeps
  its velocity, so gestures retarget without a jump. Each gesture restarts
  from what is on screen whenever a finger is added or lifted, un-doing
  the rubber band first. `will-change` is on only while moving, so the
  picture redraws sharp once still. A click after a drag or a tap on the
  picture is swallowed (`FZ.swallow()`); only a tap on the scrim closes.
  Reuse it for any other zoomable picture.
- **A sheet over a dialog** (the gallery picker, `Plume 1.6 · gallery
  picker`, `#fpk`): its own layer at z-index 110, above `#mask` (100) and
  below the figure viewer (120) and the snackbar. Bottom sheet below
  600px, dialog from 600px. Open and close flip one class, `is-open`, so
  every motion is a transition that retargets; it is hidden only after
  `getAnimations()` have finished. Its keydown handler runs in the capture
  phase, registered before the companion's, and stops every key while it
  is open, so Escape closes the sheet and nothing under it. The handle and
  title drag it down 1:1 (0.55× upward); past a third of its height or on
  a flick it closes, otherwise it springs back. Filter chips widen to show
  a check that draws itself (`grid-template-columns: 0fr → 1fr`,
  `stroke-dashoffset`).
- **A full-screen picture viewer** (`Plume 1.10 · figure viewer`, `#lbx`):
  bar with Close, title (slot and "2 of 5"), zoom and ⋮ (Replace, Remove;
  Remove asks twice through `figArm`, the row turning `error`, and the menu
  stays open between taps). The picture opens out of the thumbnail tapped
  (`lbxSource()` finds it, `lbxAt()` gives the transform plus a
  `clip-path: inset(… round r)` that covers the thumbnail's box, so a
  cropped gallery tile morphs too); the thumbnail is hidden while its
  picture is in flight and shown again when it lands or goes back. Opening
  uses `--p-spatial-slow`, closing `--p-spatial` back into the thumbnail,
  or a 150ms fade and shrink on the exit curve when it has scrolled away or
  the picture is zoomed. The picture's fitted box is set in pixels
  (`lbxFit()`) from its known shape (`f.w/f.h`), so the morph and the zoom
  measure the picture, not a letterbox; the thumbnail's pixels stand in
  until the full one loads. Two layers, two transforms: `.lbx__frame`
  carries the morph and the swipe, the `<img>` carries `FZ`. On touch at
  fit the frame follows the finger: sideways to step (0.55× past the ends),
  down to close with the scrim and chrome thinning as it goes; a third of
  the way (a fifth, down) or a flick commits. A step's new picture slides
  in only once it has loaded (its animation waits paused), and the
  neighbours' full pictures are fetched ahead. Several pictures show as
  dots with one travelling pill (12 at most), and, for a mouse, arrows
  beside the picture. Focus goes to the dialog itself on open (focusing
  Close showed its focus layer at rest) and back to where it was on close.
- **Settings** (`Plume 2.2 · settings`, rebuilt from scratch): a hero
  whose focal element is a live **specimen** (`sxSpecHTML`, a question drawn
  with `--qfont`/`--qlh`/`--qfam`/`--efont`, so every reading choice is seen
  on real text), with the theme button group and the accent seeds under it;
  the sections as grouped cards with a 40px tonal leading shape per category
  (`cat` in `SETTINGS_SPEC`) and a summary line that crossfades when it
  changes. Reuse its parts:
  - **Button group with one travelling indicator** (`.pl-seg`, `sxSegHTML`,
    `sxSegMark`): equal columns, the indicator at a real 20px radius that
    squares to 12px and squeezes to 0.96 with its label while pressed, the indicator moved by `--i` on a
    transition (so it retargets), the label it lands on takes its colour
    after `--lag`, scaled by the distance travelled. The colour style is the
    same group with swatches generated by the engine (`.pl-seg--sw`).
  - **A control inside a theme change**: give it `view-transition-name:
    sx-ctl` for the length of `Motion.morph`; `::view-transition-old(sx-ctl)`
    is hidden and the new one is live, so the indicator keeps sliding in the
    new colours instead of crossfading with its old self. `html.m-snap`
    exempts `.pl-seg__ind` and `.sx-seeds__ring` from the transition kill.
  - **Seeds with a travelling ring** (`.sx-seeds`): colours from
    `M3C.css`, never typed; the last seed opens Appearance and shows the
    current hue when it is not a preset.
  - **Switch** (`.pl-sw`: input + `.pl-sw__t` thumb): 52×32 well track,
    the thumb 16 → 24px (28 while the row is pressed) by `scale`, a check
    that draws in after 60ms. Overrides the legacy global checkbox.
  - **M3E slider** (`.pl-slider` + `sxSlider`): 16px track, a 12px gap
    round a 4px bar handle, a stop dot, and a value label that pops above
    the handle while dragging (`.is-drag`).
  - **Stepper** (`.sx-step`, `sxRoll`): the old number leaves the way the
    count went while the new one comes in from the other side.
  - **Text swap** (`sxSwap`): any text that changes on a live screen fades
    out in 80ms and the new text fades in, retargetable.
  - **List → detail on a phone** (`sxOpen` / `sxClose` / `sxMove` /
    `sxPaint`, v6.61): one progress value `SXP.p` (0 list, 1 section) on a
    per-frame spring (340 / 0.9 going, 520 / 1 coming back) paints the
    layer's `translate3d`, its leading corner radius (28px while travelling,
    square once landed), the list, app bar and nav bar drifting −20% and the
    scrim (`#sxScrim`, z 79). The layer has `touch-action: pan-y`, and so
    does its scroller (touch-action is read only up to the nearest
    scroller), so a sideways pull reaches the drag-back handler; it ignores
    pulls that start on a slider or a field. A tap while it is closing
    reopens it from where it is. Its history key is `{sxd:1}`.
  - **List → detail on a large screen** (`sxSwitch`, `sxMark`): a 4px
    marker travels to the open row (`translate` transition) and stretches
    in proportion to the distance (a WAAPI `scale` on top); the old section
    steps out 12px the other way (absolutely placed so nothing jumps) and
    the new one enters 24px from the side the marker went.
  - **Search that finds settings, not sections** (`sxIndex`, `sxSearch`):
    every spec row and every static row (`SX_STATIC`); a hit opens the
    section, scrolls the row to the centre and flashes it (`.sx-flash`).
    The magnifier sweeps its arc about the handle's end on focus.
  - **Sync status icon**: the arrows travel round their own arc while a
    sync runs (`sxSyncSpin`, the only loop, and only while it loads) and
    glide on to their drawn place when it ends; off draws the slash.
  - **Static icons** in Settings are inline copies of the sprite (`sxIc`,
    `svg[data-ic]` + `sxHydrate`), so the legacy icon-motion layer, which
    only hydrates `<use>`, gives them no generic press motion.
- **A copyable ID** (`.lbid` in the viewer): a mono chip with the copy
  icon; the snackbar says what was copied and where to use it.
- **A mark that arrives with the verdict** (the lure tag, `Plume 1.8 ·
  lure tag`, `.pl-lure` from `lureTag()`): a pill with the metrics of the
  card's own "your answer" tag (11px/16px, 600, full radius) so the two sit
  side by side, coloured by meaning, not by the card. It grows in on the
  reveal (`--p-spatial-fast` scale 0.6 → 1, fade on `--p-effect`), one
  stagger step after its card's key has turned. A dimmed card that still
  matters keeps its mark at full strength and steps back its text only.
  Draw the same component on every surface that shows the key (player,
  grouped layout, bank preview, companion), never a per-surface copy.
- **A pill on a line of text** (`Plume 1.9 · pill on a line of text`):
  inside running text, wrap the pill in `.pl-inl`, a slot the height of
  the text's own glyph box (`vertical-align: text-top`, `line-height:
  normal`, `height: 1lh`) with the pill centred in it. Beside a sentence in
  a grid, wrap the label in `.sp-wlc` (`height: 1lh` at the sentence's
  line-height, label centred, rows `align-items: start`). Both follow any
  reading font, size and line-height. A chip row with mixed fonts (mono
  digits) sets `line-height: 1` on the mono part so it cannot make one chip
  taller and push its text off centre. Check with Range rects (CLAUDE.md,
  Testing): the offset must be 0.
- **Dimming text inside a card:** step back its colour
  (`color-mix(in oklab, currentColor 72%, transparent)`), never `opacity`
  on an inline span. Opacity on an inline that wraps paints a faint
  rectangle behind the first line (seen on the lure card, v6.49). Opacity
  on a block is fine.
- **Hiding something without changing the layout** (the session clock,
  `.player.clock-off`): fade and shrink it out on the exit curve, then flip
  `visibility` to hidden after the fade (a delayed `visibility` transition).
  It keeps its space, so nothing beside it jumps, and it can't be tapped or
  read by a screen reader (`aria-hidden`) while hidden.

---

- **Figure plate** (`Plume 2.6 · figure plate`, v6.62): a captioned
  picture is one object. `figCardHTML` gives `.figfig.has-cap` a
  `--p-r-l` frame that clips the photo, with the caption on a strip tinted
  `on-surface` 5% (the same plate on the page and in a card, both themes).
  The plate's width is the photo's (`width:fit-content`, and
  `contain:inline-size` on the caption so text never widens it).
  `figCapSplit()` splits an ARCHIVIST caption at its first " — ": the name
  leads in Title S, the reading line follows in Body M. A caption that
  arrives late unfolds from zero height (`grid-template-rows` 0fr → 1fr on
  `--p-spatial`, padding on an inner `.figcap__pad` so nothing jumps) with
  its text fading in after (`.cap-in`).

## 15. Ledger: surfaces rebuilt under Plume

A surface that is not listed here is legacy.

| Surface | Plume version | App version | Notes |
|---|---|---|---|
| Companion (session screen + question reader) | 1.0 | 6.36 | First Plume surface. Tokens and primitives added. Inside the reader, the figure blocks (`figHTML`) and the referenced-file buttons (`linkBtns`) are still legacy components |
| Opening the player and the companion from Home's session card (Resume, Companion) | 1.0 | 6.38 | The transition only. The player itself is still legacy |
| Player: "Show clock" switch in the ⋮ menu, and the clock's hide/show | 1.3 | 6.40 | The menu around it is still legacy |
| Player: focus mode button in the bar, and the compact bar for phones | 1.4 | 6.41 | Moved out of the ⋮ menu. Never highlighted since 6.43: the icon carries the state. The other bar buttons keep their legacy press style |
| Figure viewer: pinch, pan, double-tap and wheel zoom | 1.5 | 6.45 | The gesture and its springs only. The viewer's bar and buttons are still legacy |
| Gallery picker ("From gallery" in every figure block) and the image ID chip in the viewer | 1.6 | 6.46 | The "From gallery" and "Gallery" buttons sit in legacy figure rows and match them; the sheet itself is Plume |
| Figure viewer, whole surface: bar, ⋮ menu, open and close from the thumbnail, swipe to step and drag down to close, dots, side arrows, Back closes it | 1.10 | 6.57 | Rebuilt from scratch; the zoom (1.5) and the ID chip (1.6) are kept as they were. The gallery rail's buttons kept their look and took the touch contract |
| Alignment of every pill beside text in the answer cards (lure, "your answer", Not here / Right in in both layouts) and the verdict's chips | 1.9 | 6.50 | Alignment only; those legacy components keep their look |
| Lure tag on the answer cards (player, grouped "Choice by choice", bank preview, companion) and the dimmed lure card's edge | 1.8 | 6.49 | The tag only. The cards around it are still legacy; the editor's "This is the lure" switch and the report's "Took the lure" chip use the legacy controls beside them |
| **Foundation: lighting roles, Google Sans Flex, card and hero radii (§9)** | 2.0 | 6.58 | Landed with Settings. Engine writes the lighting roles and retunes the M3 surface ladder (legacy screens follow the lighting model without being rebuilt); Plume 1.x surfaces re-pointed to the roles; the `.card` and logo light-theme swaps removed. Measured ΔL card − page: 0.038 light, 0.065 dark. Legacy screens keep their own layout and colour fills (Home's hero is still a container fill) |
| Settings, whole screen (list, hero, every section, search, phone layer, desktop list-detail) and the controls in the in-session settings sheet | 2.2 | 6.58 | Rebuilt from scratch. The in-session sheet's dialog around the rows is still legacy |
| Figure captions (`figCardHTML`): the plate in the player, explanation, bank and editor, and the name / line in the viewer | 2.6 | 6.62 | The figure blocks' edit buttons, empty-slot plates and "Add a figure" row are still legacy |

## 16. Language changelog

- **Plume 1.0** (2026-10-07): first definition. Based on M3 Expressive,
  extended with the touch contract, continuity rules, travelling
  indicators, gesture physics and the Nidus moments. Amended the same day
  with the first surface (Companion): spring durations replaced by the
  generated settling times, and §13 now maps where Plume lives in the code.

- **Plume 1.1** (2026-10-07): rules learned from the first fixes. A screen
  opened from a card never shows a blank frame and arrives as one piece.
  Snackbars wait for motion. Escape closes only the top layer (capture-phase
  key handlers). Spoilers stay sealed: no colour hints at a hidden answer.
  Transitions are filmed frame by frame before they ship.

- **Plume 1.2** (2026-10-07): verdict containers never fill a card that
  holds reading text; the wrong option in the player became a faint veil
  with a thin red edge.
- **Plume 1.3** (2026-10-07): a switch inside a menu (row is a label,
  the menu stays open), and hiding an element in place without a layout
  jump (fade out, then `visibility`).
- **Plume 1.4** (2026-10-07): icons that morph by rotating their own
  strokes; the compact session bar, measured with the widest counter.
- **Plume 1.5** (2026-10-07): pinch-to-zoom for pictures, driven by a
  per-frame spring that a finger can catch mid-flight.
- **Plume 1.6** (2026-10-07): a sheet that stacks over a dialog (its
  layer, its keys, drag to dismiss), filter chips with a drawn check, and
  the copyable ID chip.
- **Plume 1.7** (2026-10-07): a highlight keeps the column; a tinted tile
  in a list never juts out toward the markers.
- **Plume 1.8** (2026-10-09): a mark that arrives with the verdict (the
  lure tag), in `warning` because it is a caution, not a verdict; dim text
  by colour, never by opacity on an inline.
- **Plume 1.9** (2026-10-09): a pill beside text is centred on that text
  (`.pl-inl`, `.sp-wlc`), never baseline-aligned or nudged by hand.
- **Plume 1.10** (2026-10-10): Back closes the top full-screen layer
  through a history entry of its own; Close is always on screen, top left,
  with a bar of two or three actions and a ⋮ menu for the rest; the
  full-screen picture viewer (container morph out of a cropped thumbnail,
  follow-the-finger step and dismiss).

- **Plume 2.0** (2026-10-10): change of direction, from measuring Nidus
  against the feel the user wants. Light comes from above in both themes
  (lighting roles generated by the engine; cards lighter than the page;
  measured gap; no pure-white page; no hand-swapped roles). Large surfaces
  neutral, colour spent on small things. Google Sans Flex with expressive
  numbers replaces Roboto Flex. Card and hero radii (24 / 32), 56px main
  actions, no small coloured group headings. Motion as an ensemble with
  one cause and one direction; the first frame never waits for rendering.
  The file now separates goals, hard rules and defaults, and invites
  inventive implementation measured against a quality bar. The reference
  moment (§2) records what made the user's favourite control fluid, and
  its lag.

- **Plume 2.1** (2026-10-10): the user's two clarifications. Butter:
  no visible change happens in a single frame; everything that changes
  transitions. Icons move by their own logic: read the object the glyph
  depicts and move its parts as that object would, never a generic bounce,
  stretch or spin of the whole glyph; examples for Nidus's icons.

- **Plume 2.2** (2026-10-10): Settings, the first Plume 2 surface, and the
  foundation with it. New components: the button group with one travelling
  indicator and a distance-scaled label lag, the Plume switch, the M3E
  slider with a value label, the rolling stepper, accent seeds with a
  travelling ring, the live reading specimen, the list → detail layer that
  grows from its row (phone) and the travelling list marker (large screens),
  a control lifted out of a theme change (`sx-ctl`), and search that finds
  single settings. Category hues for leading shapes come from the engine.

- **Plume 2.3** (2026-10-10): a surface returning into its origin hands
  off to it (fades out as it lands, removed once invisible) and never
  covers it as a blank tile; found on the Settings close.

- **Plume 2.4** (2026-10-10): a shape whose corners morph rests at its real
  radius (half its height), never 999px; the chosen option's indicator
  squeezes with its label when pressed.

- **Plume 2.5** (2026-10-10): list → detail pages slide over the list
  (rounded leading edge, list drifting and dimming, drag back with the
  finger) on one per-frame spring; no container transform or grow-from-row
  reveal for a page. From the user's correction on Settings.

- **Plume 2.6** (2026-10-10): the figure plate. A caption shares the
  photo's frame on a tinted strip, name first, reading line after, and a
  caption that arrives late unfolds from the photo's edge. From the user
  finding the bare grey caption line badly designed.

Version rules: a clarification or a new component spec bumps the minor
version (1.0 → 1.1). A change of direction (palette philosophy, motion
system) bumps the major version, and the ledger records which surfaces
were built under which version.
