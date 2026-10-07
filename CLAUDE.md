# Nidus

A personal USMLE question bank (study sessions, review queue, bank, images,
progress), built as a single-file app: everything lives in `index.html`,
served by GitHub Pages from `main`. Its data syncs to the private repos
`nidus-data` / `nidus-media`. Never edit those repos.

## Keep these files current, unprompted

Updating `CLAUDE.md` and `DESIGN.md` is part of every change, in the same
commit, without the user asking. Before shipping, check what the work
taught and write it down:

- a new or changed Plume pattern, helper, component or token → `DESIGN.md`
  §13 (where it lives) and the §14 ledger, with a Plume version bump if it
  adds a rule;
- a bug found and its cause, or a correction from the user → the rule that
  prevents it (`DESIGN.md` for design and motion, this file for workflow
  and code conventions);
- a new house convention, shared function or testing technique → this file.

If nothing was learned, nothing changes; say so in the report.

## Design: Plume

Nidus is being redesigned in **Plume**, its own design language built on
Material 3 Expressive. `DESIGN.md` defines it (the current version is in its
header). **Read `DESIGN.md` before any visible change and follow it
exactly**, without waiting to be asked.

- **The current UI is legacy.** It was built before Plume and is not a
  reference. Don't copy its colours, shapes, spacing or motion into new
  work. Reuse legacy code only for *logic* (data, sync, the `Motion`
  engine's lifecycle helpers such as `Motion.leave`), and only if it meets
  Plume's mechanics rules.
- **Scope is the user's call.** Rebuild only the surfaces that are asked
  for. If a Plume change touches a component that legacy screens share,
  scope the new styles so legacy screens don't break, and mention it.
  Don't restyle surfaces nobody asked about.
- **Every surface built or rebuilt in Plume** gets:
  1. a CSS block marked `/* Plume <ver> · <surface> */`,
  2. a row in the ledger in `DESIGN.md` §14,
  3. the Plume label in the commit message (see below).
- **Plume's definition of done** (`DESIGN.md` §12) is the checklist for
  every Plume change, including `htmlcheck --shot` screenshots at 390px in
  light and dark.
- If a change needs a pattern Plume doesn't cover yet, design it within the
  principles, then add it to `DESIGN.md` in the same commit and bump the
  Plume minor version.
- The **frontend-design** plugin is installed. Use it for aesthetic
  direction on visual work, but `DESIGN.md` wins whenever they disagree.
- **Motion complaints:** when the user says a motion looks wrong or buggy,
  film it frame by frame on the real trigger first, name exactly what
  happens ("Home vanishes in one frame, then a blank screen, then the
  options appear one by one"), and fix that motion only. Don't add visual
  elements nobody asked for.

## Versions and commits

- The app version lives in four places. Bump them together (6.35 → 6.36):
  the comment on line 2, `APP_VERSION`, `#buildChip`, and `#verChip`
  (`vX.YY · sharded store`).
- Commit message format: `<Area>: <what changed> (v6.36)`. For Plume work,
  add the language version: `<Area>: <what changed> (v6.36 · Plume 1.1)`,
  using the version in the header of `DESIGN.md`.

## House rules

- **The question spine** (comment at the top of `index.html`): one block
  order for every surface that draws a question (player, bank preview,
  companion, Markdown copy via `qToMarkdown()`). A surface may omit a
  block but never reorder one.
- **Copying a question:** the study session defines what Copy writes and in
  what order, through `sessCopyOpts(scope)` (no tags line, explanation
  figures last). Any other surface that copies a question from a session
  (the companion does) calls the same function, so the copies stay
  identical. Change copy behaviour there, never per surface.
- **Confidence travels with the answer.** The "How sure are you?" control
  hides once the answer is submitted, so every place that reports an
  answer (block 7) also reports its confidence (`it.conf`): the verdict's
  meta row (a pill with the control's own icon, worded by `CONF_IC` /
  `CONF_SAY`), the Markdown copy ("Confidence before answering: …") and
  the companion's "Answered there" line. A new surface that shows an
  answer shows its confidence too.
- **Adding a picture** always offers three ways in: choose a file
  (`figPick`), **Paste** (`figPaste`) and **From gallery** (`fpkOpen`,
  which reuses a picture already in the bank). A phone has no Ctrl+V and no
  field to long-press, so a copied image needs its own button. `figPaste`
  calls `navigator.clipboard.read()` first thing in the tap (iOS shows its
  Paste bubble, Chrome asks once); if that is blocked it opens a paste box
  (`figPastePad`) in the figure block. Every figure block drawn by
  `figHTML` (companion, bank preview, editor, figure manager) gets all
  three, and an empty placeholder (`figPlateHTML`) gets Paste and Gallery.
- **A picture is its hash.** `f.media` is the SHA-256 of the bytes and the
  media repository stores each hash once, so two questions share a picture
  by pointing at the same hash. Its ID for people is `figImgId(media)`
  (`img-` plus the first 8 characters), read back with `figIdTerm()` as a
  prefix. It is shown and copied in the figure viewer, and accepted by the
  gallery search and the picker. Reuse always goes through `figLink`, which
  writes a reference, never bytes, and calls `V3.photoAttached` like a new
  photo so a reference never lands ahead of its bytes. Never add a second
  ID field: the hash is already the same on every device.
- **Focus mode has two forms**: real fullscreen and the in-page fallback
  (`.zen`, used when the browser refuses fullscreen). Its state is
  `focusOn()` (either one), and a tap always flips that. Never decide from
  the fullscreen flag alone: that left the fallback stuck on with the icon
  saying off (v6.43 and earlier). Every fullscreen call is awaited and
  caught, a tap during a pending request is ignored (`fsBusy`), and hiding
  the tab exits fullscreen while coming back repaints the button.
- **Preferences** are declared once in `SETTINGS_SPEC` (the SETTINGS ENGINE
  section). The Settings screen and the in-session sheet both render from
  it.
- **Storage:** `nidus_db`, `nidus_base_<kind>`, `nidus_media_budget` and
  `nidus_straggler` are live keys. Never rename one without migrating the
  saved data. Any change to data shape must keep sync (CLOUD SYNC section)
  and import/export working.
- Ship with `ship ~/projects/nidus-app "<message>"` (from the global
  instructions). Never push code that fails `htmlcheck`.

## Testing locally

`htmlcheck` only loads the page empty. To see real screens, drive the app
with playwright-core (Chromium at `~/.cache/ms-playwright`, as `htmlcheck`
does) and work in the scratchpad:

- **Seed data:** in `addInitScript`, write a JSON database to
  `localStorage.nidus_db`, guarded by a `sessionStorage` flag so a reload
  doesn't overwrite it. It needs `questions` (id, stem, leadIn, choices
  with label / text / isCorrect / explanation, correctAnswer, explanation…),
  `stats`, `sessions` and `settings` (`{theme:'light'}` gives the light
  theme; setting `data-theme` by hand is overwritten by the app).
- **Session items must be complete:** `{order, chosen, struck:[], conf,
  timeMs, ansMs, submitted, correct}`. A missing `struck` makes the player
  throw, which looks like an app bug but isn't.
- **Open screens:** `compOpen(id)`, `compQuestion(i, tileEl)`,
  `resumeSession(sessFind(id), buttonEl)`, or tap the real buttons
  (`[data-compopen]`, `[data-sessresume]`) with `hasTouch:true`.
- **Film a transition:** `Page.startScreencast` over CDP while tapping the
  real trigger, save the frames with their times, and make a contact sheet
  with ImageMagick (`montage -label '%t' … -tile 8x3`). Read the sheet
  before and after the fix.
- **Test that a setting survives a reload without the seed.** The real
  store is IndexedDB; the `localStorage.nidus_db` seed can win again on
  reload and make a working setting look lost. Load the page empty, set
  the value, `await doSave()`, reload, then read `DB.settings`.
- **Measure the session bar, don't eyeball it.** For anything added to
  `.sp-top`, set the session to 120 questions at index 0 and 99 and, at
  320 / 360 / 375 / 390 / 412 / 430px, read `scrollWidth - clientWidth`
  of `.sp-top` and `.sp-count` (both must be 0) and the gaps between the
  counter, clock and tools.
- **Narrow-width rules go after the `max-width:520px` player block.** It
  comes late in the file and silently overrides an earlier, narrower media
  query with the same specificity.
- **Test the clipboard over http, not file://.** Serve the repo with
  `python3 -m http.server`, `grantPermissions(['clipboard-read',
  'clipboard-write'])` on the context, write a PNG with
  `navigator.clipboard.write`, then tap Paste. For the fallback, make
  `navigator.clipboard.read` reject and dispatch a `ClipboardEvent('paste')`
  carrying a `DataTransfer` file on `.figpad`.
- **Test focus mode by stubbing the browser**, since headless Chromium
  keeps fullscreen across tabs: make `requestFullscreen` reject (refused),
  slow it down and tap twice (pending), and fake `visibilityState` hidden →
  visible (another tab). After every tap, fullscreen, `.zen` and the icon
  must agree.
- **Test pictures with real bytes.** A seeded `media` hash has no bytes
  behind it, so thumbnails stay blank. Draw images on a canvas in the page,
  turn them into `File`s and call `figTake(qid, null, slot, [file])`: that
  runs the real ingest and gives real hashes. Count `IDB.sAll('media')`
  rows before and after a reuse to prove nothing was stored twice.
- **Test touch gestures with real touch points.** Playwright's `tap` sends
  one finger only. For pinch, pan and double tap, open a CDP session and
  send `Input.dispatchTouchEvent` (`touchStart` with two points, a series
  of `touchMove`, then `touchEnd` with none), with `hasTouch` and
  `isMobile` on the context. Read the picture's `style.transform` and
  `visualViewport.scale` (it must stay 1: the page itself never zooms).
  To check a spring, log the transform on every animation frame.
- **Check copies:** replace `window.copy` with a function that records the
  text, then click the copy buttons and compare the strings.
