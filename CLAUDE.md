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
- **Check copies:** replace `window.copy` with a function that records the
  text, then click the copy buttons and compare the strings.
