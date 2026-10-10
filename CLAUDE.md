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
  §14 (where it lives) and the §15 ledger, with a Plume version bump if it
  adds a rule;
- a bug found and its cause, or a correction from the user → the rule that
  prevents it (`DESIGN.md` for design and motion, this file for workflow
  and code conventions);
- a new house convention, shared function or testing technique → this file.

If nothing was learned, nothing changes; say so in the report.

## Design: Plume

Nidus is being redesigned in **Plume**, its own design language built on
Material 3 Expressive and aiming past it. `DESIGN.md` defines it (the
current version is in its header). **Read `DESIGN.md` before any visible
change**, without waiting to be asked. It has three kinds of content:
**goals** (what a surface must feel like: meet them), **hard rules** (from
real bugs and the user's corrections: never break them) and **defaults**
(the numbers: use them unless something better passes the quality bar in
§13, then record it in §14). Be inventive in how a surface is built; the
user's favourite motion came from a model given a goal and free rein, not
a recipe.

- **The foundation lands first.** Plume 2's lighting roles, typeface and
  radius tokens are app-wide (`DESIGN.md` §9). If the ledger shows them as
  not landed, the first Plume 2 request lands them in the same change and
  the report says so.

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
- **Plume's definition of done** (`DESIGN.md` §13) is the checklist for
  every Plume change, including `htmlcheck --shot` screenshots at 390px in
  light and dark.
- If a change needs a pattern Plume doesn't cover yet, design it within the
  principles, then add it to `DESIGN.md` in the same commit and bump the
  Plume minor version.
- The **frontend-design** plugin is installed. Use it for aesthetic
  direction on visual work, but `DESIGN.md` wins whenever they disagree.
- **The flashcard app (engram-app) is a reference for feel, never a
  source.** The user pointed at it to show what they mean; don't copy its
  code, tokens or look into Nidus.
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
- **The lure travels with the key.** ARCHIVIST §6B marks the wrong option
  built to pull in a reader who knows most of the topic: `"lure": true` on
  that choice, one per item, two at most, never the key (`lureTidy()`
  enforces this on import and in the editor; a choice that is not a lure
  has no `lure` field at all). It is answer-side, so it shows exactly where
  the key shows and nowhere else: the player once submitted, the grouped
  "Choice by choice", the bank preview, the companion's spoiler, the
  Markdown copy (`[lure]` mark, "incorrect, took the lure") and the missed
  sheet. Picking it says so (`tookLure()`): the verdict reads "Took the
  lure", the report adds a "Took the lure" chip, and the companion's
  "Answered there" line says it. Never on the companion's read-along
  options or the question-only copy. A new surface that shows the key
  shows the lure with `lureTag()`. Questions imported before ARCHIVIST v3.1
  have no lure; Export for re-run offers just those, and the stale-prompt
  note flags a saved prompt without section 6B.
- **Changing the extractor prompt** (`#defaultPrompt`): bump its ARCHIVIST
  version, add the new rule to the final gate, give `renderPrompt()` a gap
  check for a saved copy that lacks it, keep the re-run export carrying
  the new field, and update the `nidus-archivist` skill to match (it is
  the same protocol, used outside the app). The skill also ships
  `scripts/figures.py` (overview, pages, find, crop, sheet, embed). It
  must run offline: claude.ai's sandbox blocks pip, so it reads PDFs with
  whatever is installed (PyMuPDF, then pypdfium2, then poppler's
  `pdftoppm`), forced with `FIGURES_BACKEND=` for testing. Test a change on
  all three (here: apt `python3-pymupdf`, `poppler-utils`, pip
  `pypdfium2`) and compare the crops on one sheet. A change to how
  figures travel changes the script, the prompt text and `figAbsorb()`
  together.
- **Pictures can arrive inside the JSON** (ARCHIVIST 3.2, section 9A): the
  extractor crops each figure from the source PDF with the skill's
  `scripts/figures.py` and carries it in `src` as a JPEG data URI. Every
  import goes through `importText()`, never `doImport()` alone:
  `figAbsorb()` first sends each data URI through `MEDIA.ingest` (stored
  once by hash, the same picture twice in a batch stored once, `src`
  emptied), then `doImport()`, then `figAnnounce()` queues the bytes and
  calls `V3.photoAttached` for each touched question, like a hand-picked
  photo. A question must never keep megabytes of base64 in the data repo;
  only a browser without IndexedDB keeps the data URI, as `figTake` does.
- **Figure fill** (ARCHIVIST section 12): **Export missing figures** writes
  `kind: "figures"` with each question's id, source, tags, wording and its
  empty, non-optional slots (fid, slot, caption). The reply carries only id
  and images (fid, slot, src). `figFillImport()` matches by id, then fid,
  then the first empty slot of the same kind, and fills **only empty
  slots**: a photo already attached is never replaced and nothing else in
  the question changes. It reports filled / already had / not found / gone.
  When the reply puts the same picture in two empty slots of one question,
  the second slot was a duplicate placeholder (two extractor runs each left
  one, with slightly different captions, so `figDedupe` kept both): it is
  deleted with `figBury`, like Delete placeholder, and reported as a removed
  duplicate. Before v6.54 it stayed empty and counted as "already had a
  photo", so it came back in every later export.
- **The file tag carries the question's number** (`cardiovascular system v5
  #11`, ARCHIVIST 3.2 section 7). The Tags filter groups by file through
  `fileTagBase()`, which drops the ` #N`; the question and search keep it.
  A file the app exported (re-run, figures, backup) is never a source:
  until ARCHIVIST 3.3 a re-run tagged every question with the re-run file's
  name ("nidus rerun 2026 10 07") and the real file tag was lost.
  `isExportTag()` drops such tags on every import, and a **tag fix** file
  (`kind: "tags"`, items of `{id, tags}`) replaces only the tags of the
  questions it names (`tagFixImport()`), for repairs in bulk.
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
- **Sessions across devices.** Finishing is terminal and wins every merge
  (`richer()`), and three rules keep the screens honest about it:
  - *Finished elsewhere closes the player.* A merge that turns the session
    open in this device's player into `done` calls `sessEndedElsewhere()`:
    the player closes without flushing time, Home shows it in Recent
    sessions, and a toast says where it went (the sync's generic "merged
    in" toast is held back for 8 s so it cannot replace it). Before v6.51
    the phone kept the graded session open with Submit live.
  - *Deleted stays deleted.* Deleting a session writes a stamped rule in
    `purged.sessions` but leaves its file in the store, so every path that
    adds a session checks `sessPurged()` first (`mergeSessionFile`, the
    legacy `mergeProgress`), rules arriving from another device run
    `sessDropPurged()`, and so does start-up. Before v6.51 a full pull
    brought deleted sessions back.
  - *Lists of sessions sort by time, never by array position.* The array
    holds sessions in the order they reached this device, which after a
    pull is file order. Recent sessions sorts by `finishedAt`.
  The companion's "on question N" line clears when that device's beacon
  stops carrying a session (`COMP.live.dev`).
- **The sync pointer is written last.** `c.sha` (localStorage, written at
  once) promises "everything up to here is merged on this device". So the
  merged data goes to disk (`doSave()`) before the pointer moves, in every
  pass and in `fullResync`. Until v6.51 the database was written only after
  the push, seconds later; a phone that went to the background (which
  itself starts a pass) and was frozen or killed in that window kept the
  pointer, lost the merge, and never fetched those files again: a session
  finished on the tablet stayed "Live elsewhere" on the phone for good.
  Bumping `REPAIR_STAMP` makes every device do one full re-read on its next
  sync; v6.52 did that to recover what the old order lost. The manifest
  pull reads at the branch tip, never at an older `d.head`, so a file is
  never marked as delivered in a newer form than the one merged.
- **Back must never leave the app from an overlay.** Nidus usually is the
  only page in its tab, so Back with no entry of the app's own closes the
  tab (the user lost the app that way from the figure viewer until v6.57).
  The figure viewer pushes `{lbx:1}` on open and listens for `popstate`
  (`lbxHistPush` / `lbxHistBack`, DESIGN.md §6 Layers and keys). Any new
  full-screen layer follows the same pattern with its own key. Dialogs
  (`#mask`), sheets and the companion don't have one yet. The Settings
  section layer on phones has one: `{sxd:1}` (`sxOpen` / `sxClose`).
- **Preferences** are declared once in `SETTINGS_SPEC` (the SETTINGS ENGINE
  section). The Settings screen and the in-session sheet both render from
  it. A section has `cat` (a, b, c, n: the hue of its leading shape on the
  list, by what it is about) and is listed in `SX_GROUPS`; its summary line
  is `SX_SUMMARY[id]`, its search words `SX_KEYS[id]`. A row has `label`,
  an optional `sub` (one short line printed under it) and an optional
  `hint` (the longer explanation, shown only in the section's (i) note).
  A row in a static panel that search should reach goes in `SX_STATIC`.
  Any preference change ends in `applySettings()`, which calls
  `sxRefresh()` to repaint the list, the seeds and the sync card; never
  re-render the Settings screen to show a changed value (that cuts the
  motion of the control under the finger).
- **Inline icons in Plume surfaces**: `sxIc(id)` (or `<svg data-ic>` +
  `sxHydrate`) copies a sprite symbol inline. The legacy icon-motion layer
  hydrates every `<use>` and gives it a generic press motion, which Plume
  forbids; inline copies are left alone.
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
- **Test pictures that arrive in the JSON** with a real JPEG data URI (make
  one with ImageMagick, `base64 -w0`), put the same one in two items, and
  call `importText(json,true)`: both questions get one hash, `src` is empty,
  `IDB.sAll('media')` grows by exactly 2 (full + thumb), and
  `JSON.stringify(DB.questions)` holds no `data:image/`. For a fill, seed a
  question with one filled and one empty slot plus an optional one, replace
  `window.download` to catch the export, and import a reply that also
  targets the filled slot, an unknown id and a null src: only the empty slot
  fills. `setInputFiles('#fileInput', …)` drives the real file path.
- **Seed a lure** with `lure:true` on a wrong choice (seeded questions skip
  `normalizeQ`, so a stray `lure:true` on the key is drawn as nothing,
  not as a tag). To test the import rules, call `doImport(json, true)`
  with a lure on the key and three on wrong options: the key loses it and
  only the first two stay.
- **Measure alignment, don't eyeball it.** For a pill beside text, take
  the pill's `getBoundingClientRect()` centre and the centre of the text
  line it shares (`Range.selectNodeContents(textEl).getClientRects()`, the
  rect whose centre is nearest the pill). The difference must be 0. Run it
  at the default reading settings and at a large size, line-height 2 and
  each family (`Object.assign(DB.settings,{font,efont,lh,family});
  applySettings()`; setting the CSS variables by hand is overwritten). A
  pill that wrapped onto a line of its own has no text to align with, so
  skip it. Then look at a 4× clip (`deviceScaleFactor: 4`).
- **Test sync with two devices and a fake GitHub.** Sync bugs need two
  devices, so drive two browser contexts (separate storage) against one
  in-memory GitHub installed with `context.route('https://api.github.com/**')`.
  It needs: `GET /repos/O/R/commits/main` (sha text, ETag, 304), `compare/A...B`
  (files that differ), `contents/PATH` (raw file, JSON dir listing, ETags,
  404), `POST /graphql` (the `readMany` `object(expression:"ref:path")`
  query and `createCommitOnBranch` with `expectedHeadOid` → `STALE_DATA` on a
  moved head), and the `git/blobs|trees|commits|refs` chain as fallback.
  Seed it with a read-only copy of the real `nidus-data` tree
  (`gh api …/git/trees/main?recursive=1`, then each file raw) and give each
  context a `nidus_sync_cfg` in `localStorage` (token, owner, repo, mrepo,
  branch, dir `nidus`, its own `deviceId`). `SYNC.now()` runs a pass; the
  other device also syncs by itself when it sees the beacon, so install any
  spies before the action that triggers it. Never point a test at the real
  repositories. To test a tab killed mid-sync, let the fake hang one
  device's commit (`createCommitOnBranch` never answers for that device's
  headline), close the page, and open a new page in the same context. To
  test recovery of a device the old build broke, run the old file
  (`git show HEAD:index.html`) first, then reopen with the new one.
- **Test Back with `page.goBack()`.** It fires `popstate` like a phone's
  Back. After it, `location.href` must be unchanged, the layer closed and
  the one underneath (`#mask.on`) still open. Also close with ×, Escape
  and a close then a reopen 30ms later, then press Back once more: each
  time `history.state` must end without the layer's key.
- **Measure that Close is on screen** for any full-screen layer: at 320 /
  390 / 430px, every bar button's `getBoundingClientRect().right` is at
  most `innerWidth`. A screenshot hides it: the legacy viewer looked fine
  with Close cut off at x=405.
- **Measure the lighting, don't eyeball it.** In light and dark, read
  `getComputedStyle` background of the page (`body`) and of a card on it
  (both are `oklch(L C H)` strings, since the engine writes OKLCH; convert
  anything else through a canvas). The card's L minus the page's L must be
  at least 0.035 in light and 0.06 in dark, and never negative. Then put
  the two screenshots side by side: they must look like the same object,
  not a negative.
- **Measure the first frame.** Seed a bank of at least 2,000 questions, then
  in the page dispatch `pointerdown` and `click` on the control and log
  the moving element's computed `transform` (a `DOMMatrix`, `m41` for x)
  on every `requestAnimationFrame` for ~900ms, with the click handler's own
  duration. The element must have moved by the first or second frame. A
  long run of identical first values means the motion is waiting for a
  render. (This found the 40ms freeze in the flashcard app's view switcher,
  2026-10-10.) The same log shows the spring's overshoot and settling time.
- **Screenshot a fixed full-screen layer at full length**: set its
  scroller's `overflow` to visible and the layer to `position:absolute;
  bottom:auto` before `fullPage` (undo after). Film the open with the
  screencast too: frames taken mid-grow show the list around the growing
  window, which is correct.
- **Measure the first frame of a button group**: log the indicator's
  `DOMMatrix.m41` per frame after `pointerdown` + `click` on a 2,000-question
  bank (Settings, v6.58: handler 11ms, moving by the second painted frame).
- **Check copies:** replace `window.copy` with a function that records the
  text, then click the copy buttons and compare the strings.
