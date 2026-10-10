# Plume — the Nidus design language

**Version: Plume 1.10** · Base: Material 3 Expressive (2025 release and later)

This file defines how Nidus should look, move and feel. It is the target, not
a description of the app as it is today. Most of the current UI was built
before Plume existed and is **legacy**: it does not set any standard, and
nothing in it is a model to copy. Each surface moves to Plume when it is
rebuilt. The ledger at the end lists the surfaces already done.

> A nidus is a nest. A plume is the feather it is built from: light,
> exact, and alive to the smallest touch of air.

---

## 0. What Plume is

Plume starts from Material 3 Expressive: dynamic colour, the shape library,
spring physics, emphasized type, and the new components (button groups,
split buttons, FAB menus, floating toolbars, loading indicators, wavy
progress). It keeps M3E's grammar and goes further in three ways:

1. **Every touch answers.** M3E gives you state layers and a few springs.
   In Plume, every interactive element responds physically in the same
   frame the finger lands, and settles with a spring when it is released.
   Nothing feels dead.
2. **Continuity over cuts.** Nothing appears from nowhere or vanishes. Every
   surface grows out of the thing that summoned it and goes back into it.
   Every motion can be interrupted and picks up from where it is.
3. **Restraint as craft.** Expressive does not mean loud. Plume is calm
   while you are not touching it and comes alive when you do. Colour,
   shape and motion are each spent where they mean something, so that the
   moments that matter (a correct answer, a finished block, a streak) can
   be truly expressive.

"Perfect" in Plume means perfect execution: no jank, no flicker, no layout
jump, no stuck hover, no orphaned element, no animation that fights another
one, in light and dark, at 390px and on desktop.

---

## 1. Principles

1. **Quiet at rest, alive in touch.** Nothing idles: no ambient loops and no
   decorative motion, except the loading indicator while something really
   is loading. Motion only answers the user or reports a real change.
2. **The finger is the clock.** Feedback starts on `pointerdown`, not on
   `click`. A drag follows the finger 1:1. A release carries the finger's
   velocity into the spring.
3. **Shape carries state.** Selection, focus and pressing change *form*
   (corners, size), not only colour. A selected thing looks different even
   in greyscale.
4. **One hero per screen.** Each screen has one primary action and one
   focal element. It gets the strongest colour, the largest shape and the
   richest motion. Everything else steps back into tonal surfaces.
5. **Origin and destination.** Things open from the place they were opened
   from and close back into it: menus from their anchor, sheets from their
   edge, detail views from the card that was tapped.
6. **Interruptible always.** Tapping again mid-animation retargets from the
   current position. Never queue animations, never snap to the end, never
   restart from zero.
7. **Content first.** This is a study tool. Reading comfort (measure,
   leading, contrast) beats decoration everywhere. Motion never moves text
   while it is being read.
8. **Dark and light are equals.** Both are designed, not inverted. Check
   both every time.

---

## 2. Colour

**Dynamic colour is the base.** The app already generates M3 colour roles
(`--md-primary`, `--md-surface-container-high`, …) live from one seed hue.
Plume uses those roles and nothing else. A raw hex, rgb or oklch value in a
Plume component is a bug. Use a role, or `color-mix()` of roles.

| Role | Job in Plume |
|---|---|
| `primary` / `on-primary` | The single hero action on a screen, the active indicator, progress |
| `primary-container` | The selected state of a navigation or toggle item, a highlighted card |
| `secondary-container` | Selected chips, segmented selections, the tonal button |
| `tertiary` / `tertiary-container` | Celebration and moments of delight: streaks, milestones, a perfect block. Rare on purpose |
| `success` / `error` / `warning` (+ containers) | Verdicts and status only. Never decoration |
| `surface-container-lowest` … `highest` | Depth. Surfaces are separated by tone, not by lines or shadows |
| `outline-variant` | Hairline dividers, only where tone cannot separate |
| `inverse-surface` | Snackbars, tooltips |

Rules:

- **Tonal elevation first.** A raised surface is one container step higher.
  Shadows are reserved for things that truly float above the page (FAB,
  menus, dragged items, floating toolbars), and Plume shadows are soft and
  tinted (§5).
- **State layers** are the "on" colour of the element at fixed opacity:
  hover 8%, focus 10%, press 10%, drag 16%. Never invent a hover colour.
- **Verdict colour is spent narrowly.** A right or wrong answer shows in its
  badge and a thin edge, with an 8–12% wash at most. It never floods the
  screen.
- **Never fill a card that holds reading text with a verdict container.**
  `error-container` (deep red in dark, loud salmon in light) is painful to
  read through. A graded option stays a reading surface in `on-surface`
  text: `color-mix(in oklab, var(--md-error) 9%, var(--md-surface-container))`
  plus a 1.5px inset edge of `error` at about 42%. The badge keeps the full
  colour. (Player wrong option, v6.39, after the user found it painful.)
- **Contrast:** body text at least 4.5:1, large text and icons at least 3:1,
  in both themes and every seed hue.

---

## 3. Shape

Shape is the most expressive tool Plume has. Corners are tokens, and they
move.

### Scale

| Token | Value | Use |
|---|---|---|
| `--p-r-xs` | 4px | Tags, small badges, inner segments |
| `--p-r-s` | 8px | Chips, small inputs, tooltips |
| `--p-r-m` | 12px | Cards inside cards, menu items, fields |
| `--p-r-l` | 16px | Cards, list groups |
| `--p-r-lx` | 20px | Large cards, floating toolbars |
| `--p-r-xl` | 28px | Sheets, dialogs, hero cards |
| `--p-r-xxl` | 48px | Display surfaces, hero imagery |
| `--p-r-full` | 999px | Buttons at rest, FABs, pills, indicators |

### Shape in motion

- **Press squares the corners.** A pill button on `pointerdown` morphs its
  radius toward `--p-r-m` (fast effects spring), and springs back on
  release. Small icon buttons go from round to `--p-r-m`. Never to 0.
- **Selected changes form.** A toggle or icon button that turns on goes
  from round to squircle (or the reverse, consistently across the app). A
  selected segment in a button group widens and its inner corners round
  off.
- **Connected groups.** In a button group, the outer corners are full and
  the inner corners are `--p-r-xs`. The pressed or selected member takes
  full corners all round and pushes its neighbours by a few pixels
  (`transform` only).
- **Lists as stacked cards.** A grouped list is cards with `--p-r-l` at the
  ends and `--p-r-xs` between them, separated by 2px gaps of surface, not by
  divider lines. While a row is pressed, its corners round up.
- **Expressive shapes** (the M3E library: cookie, sunny, clover, gem,
  pill, soft burst, …) are reserved for meaning: avatars, the active
  indicator in a celebration, the loading indicator, a correct-answer badge.
  Draw them as SVG paths with matching command counts so they can morph,
  or as `clip-path: path()`.
- Where it is supported, `corner-shape: squircle` may refine large cards
  and sheets (`@supports`), with the plain radius as fallback.
- **Radius never takes a spring that overshoots.** An overshooting radius
  can dip below its target. Animate corners on an effects spring (§4).

---

## 4. Motion: the Plume physics

Motion is the soul of Plume. It is all **spring physics**, written in CSS
as `linear()` easings generated from stiffness and damping (mass 1), with
the duration set to the spring's settling time.

### Two kinds of spring

- **Spatial springs** move things: position, scale, size, rotation. They
  may overshoot a little, which is what makes things feel physical.
- **Effects springs** change appearance: opacity, colour, corner radius,
  blur, shadow. They never overshoot.

### The six springs (M3E expressive scheme)

| Token | Stiffness / damping ratio | Settles in | Overshoot | Use |
|---|---|---|---|---|
| `--p-spatial-fast` | 800 / 0.6 | 330ms | 9.5% | Small things: switch thumbs, checkmarks, press release, chips, badges |
| `--p-spatial` | 380 / 0.8 | 380ms | 1.5% | Medium things: menus, cards, FAB menu, indicators that travel |
| `--p-spatial-slow` | 200 / 0.8 | 510ms | 1.5% | Large things: sheets, dialogs, full-screen transitions |
| `--p-effect-fast` | 3800 / 1.0 | 130ms | 0 | Press state, state layers, radius under the finger |
| `--p-effect` | 1600 / 1.0 | 190ms | 0 | Colour and opacity changes on selection |
| `--p-effect-slow` | 800 / 1.0 | 270ms | 0 | Fades of large surfaces, scrims |

Each token is a pair: `--p-spatial` (the `linear()` curve) and
`--p-spatial-d` (its settling time, to within 0.5%). They live on `:root`
in the `Plume 1.0 · tokens` block of `index.html`, generated by a spring
sampler (never hand-typed). Script reads them with `PL('spatial')` →
`{e: easing, d: ms}`, so CSS and JS can never drift apart. Use the **standard scheme** (stiffness 1400 /
700 / 300 at damping 0.9) for utility motion that should not draw the eye,
like a scroll-linked header or a reflowing list.

### The touch contract (every interactive element)

| Moment | Response |
|---|---|
| `pointerdown` | Within one frame: scale to **0.96** (0.92 for small icon buttons), corners square toward `--p-r-m`, press state layer at 10%. `--p-effect-fast` |
| Hold | Stays compressed. No timers |
| Release / click | Springs back to 1 on `--p-spatial-fast` (the small overshoot *is* the feel), then the action plays |
| Hover (`@media (hover:hover)` only) | 8% state layer and, on cards, 1 tonal step up. `--p-effect` |
| Focus-visible | 3px `primary` ring offset 2px, following the element's current shape |
| Cancelled press (finger slides off) | Springs back with no action |
| Disabled | 38% content opacity, no response, no cursor change |

Pressing must never shift layout: use `transform` only, never margin,
padding or size.

### Choreography

- **Arrival:** surfaces grow from their origin with `transform-origin` set
  to the anchor: scale 0.85→1 plus fade on `--p-spatial`. Their contents
  follow 30–40ms later, staggered by 25–35ms per item, six steps at most,
  in reading order.
- **Departure:** about 30% faster than arrival, with no stagger, shrinking
  back toward the origin. Remove the element only after the exit animation
  has actually finished (measured, never a guessed timer).
- **Peer screens:** fade-through. The old screen fades out (≈90ms), then the
  new one fades in with a 12px rise on `--p-spatial`. They never overlap.
- **A screen opened from a card** (Resume, Companion on Home) grows out of
  that card (`plumeOpen`). The old screen stays visible underneath until the
  new one has landed. **Never a blank frame**, and the new screen arrives
  as one piece: its own entrance animations must not trickle in parts
  (bar first, then content, then a bottom bar) under the transform.
- **Messages wait for motion.** A snackbar triggered by opening a screen
  appears after the screen has landed, never mid-transition.
- **Hierarchy (list → detail):** a container transform. The tapped card
  becomes the detail surface (View Transitions API with
  `view-transition-name`, or a FLIP fallback).
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

## 5. Depth and light

- Surfaces separate by **tone** (container levels).
- A floating element (FAB, menu, floating toolbar, dragged card, snackbar)
  gets a **Plume shadow**: two soft layers, tinted with the primary hue
  instead of pure black:
  `0 1px 2px color-mix(in oklab, var(--md-shadow) 20%, transparent),
   0 6px 20px -4px color-mix(in oklab, var(--md-primary) 18%, var(--md-shadow) 22%)`.
  Dark theme: lower the tint and rely more on tone.
- A lifted or dragged item rises one level: shadow grows (fade between two
  pre-drawn shadow layers), scale 1.02.
- Scrims: `--md-scrim` at 32%, fading on `--p-effect-slow`. A backdrop
  blur of 2–6px is allowed behind sheets and dialogs if it stays smooth.

---

## 6. Type

Typeface: **Roboto Flex** (variable), with `--m3-mono` (Roboto Mono) for
data such as IDs, timers and counts. Use tabular figures
(`font-variant-numeric: tabular-nums`) everywhere numbers change.

| Role | Size / line | Weight | Use |
|---|---|---|---|
| Display | 45 / 52 | 400 (emphasized 600) | One hero number per screen (score, streak) |
| Headline L / M / S | 32/40 · 28/36 · 24/32 | 400 (emph. 500–600) | Screen titles, sheet titles |
| Title L / M / S | 22/28 · 16/24 · 14/20 | 500 | Card and section titles |
| Body L / M / S | 16/24 · 14/20 · 12/16 | 400 | Reading text and UI text |
| Label L / M / S | 14/20 · 12/16 · 11/16 | 500 | Buttons, chips, tabs, labels |

- **Emphasized type** (M3E) is weight and optical size, not more size. A
  screen title is one notch heavier and slightly tighter (−0.01 to −0.02em).
- **Living type:** a selected nav item's label animates weight 500→700 via
  `font-variation-settings` (`--p-effect`). Hero numbers count up.
- Question stems and explanations follow the reader's own settings
  (`--qfont`, `--qlh`, `--qfam`, `--measure`). Plume never overrides them.
  Reading measure stays within 60–75ch.
- Sentence case everywhere. No ALL-CAPS labels except tiny overlines, and
  then only with +0.08em tracking.

---

## 7. Layout and spacing

- **4px grid.** Spacing steps: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64.
- Screen side margins: 16px compact, 24px medium, 24–32px expanded.
- Inside cards: 16px (compact) to 20–24px (large). Nothing touches an edge.
- Touch targets are at least 48×48px, even when the visual is smaller.
- Window classes (M3): compact < 600px, medium 600–839px, expanded 840px+.
  Navigation: bottom nav bar on compact, rail on medium, rail or drawer on
  expanded.
- Always check **390px** (phone) and a desktop width. No horizontal scroll,
  ever. Respect `env(safe-area-inset-*)`.
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

When a component is built or rebuilt, it meets this anatomy. All of them
follow the touch contract in §4.

| Component | Plume anatomy |
|---|---|
| **Buttons** | Sizes XS 32 / S 40 / M 56 / L 96 / XL 136px high. Pill at rest, squares on press. Variants: filled (hero only), tonal, outlined, text, elevated. Leading icon 20px with 8px gap. A loading button morphs its label into the loading indicator without changing width |
| **Icon buttons** | Round 40px visual in a 48px target. Toggle variants change shape and fill when selected. The icon may animate its fill axis (outline → filled) |
| **Button group** | Connected: one container, inner corners `--p-r-xs`, 2px gaps. Pressing one member widens it and narrows its neighbours (spring). Replaces old-style segmented controls |
| **Split button** | Leading action plus trailing chevron. The chevron half turns into a circle and its icon rotates 180° while its menu is open |
| **FAB / FAB menu** | Only for the screen's hero action. FAB menu: the FAB morphs into a close button (rotate and shape morph) while items unfold upward with a stagger, each a `primary-container` pill |
| **Navigation bar** | Short (64px) bar. The active item has a pill indicator that slides between items and stretches mid-travel. Label weight animates. Icons switch outline → filled |
| **Navigation rail / drawer** | Same travelling indicator. The rail can expand into a drawer with a container transform |
| **Top app bar** | Flat at rest. On scroll it shifts to `surface-container` with an effects-spring colour transition. No shadow |
| **Floating toolbar** | `surface-container-high`, `--p-r-full` or `--p-r-lx`, Plume shadow. Hides on scroll down and returns on scroll up (`--p-spatial`, translate only) |
| **Cards** | Filled (`surface-container-*`), outlined, or elevated. Press compresses the card to 0.98, not 0.96. A card that opens something uses the container transform |
| **Lists** | Stacked-card groups (§3), 56–72px rows, leading icon in a 40px tonal shape, trailing meta in label style |
| **Menus** | M3E vertical menu: `surface-container`, `--p-r-l`, 4px inner padding, rows `--p-r-m` with 48px height, grouped sections with 2px gaps instead of dividers. Grows from its anchor (scale 0.85 + fade, `transform-origin` at the anchor) on `--p-spatial`. Selected row: `secondary-container` with a check |
| **Bottom sheet** | `--p-r-xl` top corners, a 32×4px drag handle, draggable with velocity and snap points, scrim fade. Rises on `--p-spatial-slow`. Closes on scrim tap, Escape, back gesture, or flick down |
| **Dialog** | `surface-container-high`, `--p-r-xl`, 24px padding, actions bottom-right as text or tonal buttons. Arrives with scale 0.9 + fade from the centre (or from its trigger). On compact screens prefer a bottom sheet |
| **Snackbar** | `inverse-surface`, `--p-r-s`, slides up 16px + fade, one optional action, auto-dismiss 4–6s, swipe to dismiss. Only one at a time. A new one replaces the old with a quick crossfade |
| **Chips** | 32px, `--p-r-s`. Selected: `secondary-container`, a leading check that draws itself (stroke-dashoffset), and the chip widens smoothly to fit it |
| **Switch** | 52×32 track. The thumb grows from 16 to 24px when on (28px while pressed), slides on `--p-spatial-fast`, and shows a check icon when on |
| **Checkbox / radio** | Check strokes draw in. The radio dot scales in with a spring. 48px target |
| **Slider** | M3E slider: a tall track with a gap around the handle. The handle is a vertical bar that narrows while dragged. A value label pops above it while dragging |
| **Text field** | Filled or outlined. The label floats on `--p-spatial-fast`. The focus indicator grows from the centre outward |
| **Progress** | Wavy (M3E) for determinate progress worth celebrating (session progress). Flat for utility. Indeterminate: the M3E loading indicator, a shape that morphs through the expressive shapes inside a soft container |
| **Tabs** | Indicator travels (§4). Content uses a fade-through, or swipes horizontally with the finger on touch |
| **Tooltip** | `inverse-surface`, `--p-r-xs`, appears after 500ms hover or a long press, scale 0.9 + fade |
| **Empty states** | One expressive shape as illustration (in `primary-container` / `tertiary-container`), one short line, one action |

**Icons:** Material Symbols Rounded geometry on a 24px grid, outline at
rest and filled when selected or active, with the switch animated. Stroke
icons are drawn as inline SVG with consistent weight. No emoji as icons.
Icon morphs (copy → check, play → pause) use paths with matching commands,
or a rotate + scale crossfade.

---

## 9. Nidus moments (where Plume is most expressive)

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

## 10. Copy and tone

Plain, calm, second person. Short labels that explain themselves. No
exclamation marks. A helper line only when a control can't explain itself.
Errors say what happened and what is safe ("Nothing was lost").

---

## 11. Anti-patterns (never)

- Copying a legacy component's styling as a "reference" for a Plume surface.
- Raw colour values, new fonts, or one-off radii outside the scales.
- `transition: all`, animating layout properties, timers instead of events.
- Two indicators crossfading where one should travel.
- Things popping in or out without an origin.
- Shadows used to separate ordinary cards.
- Bounces on colour, opacity or radius (effects never overshoot).
- Idle or looping decoration.
- Feedback that waits for `click` while the finger is already down.
- A redesign of a surface the user did not ask for.
- A blank frame, or parts arriving at different times, between two screens.
- One key press closing two layers.
- A colour that gives away a hidden answer.

---

## 12. Definition of done (every Plume change)

- [ ] Uses only `--md-*` roles and `--p-*` tokens. No raw values.
- [ ] Every interactive element follows the touch contract (press,
      release spring, hover, focus, cancel, disabled).
- [ ] Arrivals have an origin. Exits finish before removal. Mid-animation
      taps retarget smoothly.
- [ ] No layout jump during or after any animation, and none on first paint.
- [ ] Reduced motion: still alive in feel, no travel.
- [ ] Checked in light and dark, at 390px and desktop (`htmlcheck --shot`).
- [ ] Every new or changed transition **filmed frame by frame** on its real
      trigger (see `CLAUDE.md`, Testing): no blank frame, no part snapping
      in on its own, and it lands where it should.
- [ ] Keyboard: Tab order, `:focus-visible`, Escape closes, Enter/Space
      activates. ARIA roles and states are correct.
- [ ] `htmlcheck` passes with no console errors.
- [ ] The CSS block is marked `/* Plume <ver> · <surface> */` and the ledger
      below is updated.

---

## 13. Where Plume lives in `index.html` (reuse before writing new)

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

## 14. Ledger: surfaces rebuilt under Plume

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

## 15. Language changelog

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

Version rules: a clarification or a new component spec bumps the minor
version (1.0 → 1.1). A change of direction (palette philosophy, motion
system) bumps the major version, and the ledger records which surfaces
were built under which version.
