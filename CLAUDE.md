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
- **A question's own figure keeps its caption until the answer.** The
  caption names what the picture shows, so it can give the key away. In a
  session, `figStemHTML` passes `play`, and `figHeld(qid)` holds the stem
  captions (and an empty slot's caption) until the explanation is on
  screen (tutor and submitted; never during an exam). Opened from a held
  block, the viewer shows only the stem pictures, with no caption and a
  neutral alt. On reveal the caption unfolds in. Any new surface that draws the stem
  before the answer holds the caption the same way.
- **No "attach" orders on figures.** The user asked for the alt note
  ("Attach the 12-lead ECG printed with this question") to go everywhere
  (v6.67, ARCHIVIST 3.5): the prompt no longer asks for `alt`, the
  figure-fill and re-run exports no longer send it, and `figCapPair()`
  never returns a note. Old banks still store `alt`; `figAltText()` lets
  it stand in for a missing caption only when it is a description (the
  auto slot holds `imageDescription` there), never an order to attach.
  Captions and `imageDescription` are untouched. Never reintroduce a
  reader-facing instruction to attach, add or find a picture.
- **Options that are not a line of text** (v6.68, ARCHIVIST 3.7 section
  4B). An *option table* stores its headings in `q.choiceColumns` ("" for
  an unheaded column, null when the options are ordinary) and one string
  per heading in `c.cells` on every option; `c.text` is always the cells
  joined by " · " (the importer and the editor both write it), so search,
  the fingerprint, copies and reports keep working. A *picture option* is a
  figure in `q.images` with slot `choice:<label>` (`figChSlot`,
  `figIsCh`, `figChLab`; the extractor writes `{"slot":"choice","choice":"B"}`
  and `figSlotIn()` reads it), so storage by hash, sync, Export missing
  figures, the gallery, the viewer and cleanup need no second path. Its
  `c.text` is only the words printed beside it, often "". Rules:
  - Every surface draws an option through `chLine` (text or cells),
    `chPicHTML` (pictures) and `chHeadRow` (headings in an invisible copy of
    that surface's own option row, `.chx-th`, so columns line up), and names
    one with `chSay` (HTML) / `chPlain` / `chPlainMD`. Never `aFmt(c.text)`
    for an option again.
  - An option picture's caption gives the answer away: it is drawn only
    where the key is (`cap`), and the viewer opened from a held option
    shows no caption. Tapping the picture in a session chooses the option;
    the corner button (`data-chzoom`) opens it full screen.
  - Option letters name picture slots, so anything that relabels options
    moves the pictures in one pass (`edRelabel` in the editor); removing an
    option buries its pictures first.
  - `figMerge` (import with update) walks every slot either copy holds; a
    fixed `['stem','expl']` list dropped option pictures.
  - `chHoist(raw)` moves a picture hung on the option itself into images.
    It runs in `figAbsorb` *and* `normalizeQ`: run only in `normalizeQ`, the
    picture skipped the media store and stayed in the bank as base64.
  - A picture option with no words takes its picture's hash into the
    fingerprint (`chFpText`) and never matches by the loose wording key in
    `findExisting`, or two papers' "which of these images…" would merge.
  - `hasImage` means a picture printed with the question; an item whose only
    pictures are options has it false (else `figList` makes an empty stem
    slot).
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
  - *Deleting a session removes everything it wrote* (v6.80). Every delete
    button goes through `sessDelete(id)` → `purgeData({sessions:[id]})`:
    its answers (so every count, chart, streak and Unused that reads them),
    the schedules they moved (replayed from what is left), the session, and
    the stamped rule. Until v6.80 the Progress list only spliced the session
    out of `DB.sessions`, so its answers kept counting. A session's rule
    takes all its answers, even ones written after the delete
    (`purgedAtt`; ids are never reused). A rule arriving by sync replays every
    schedule (`replayAll()`) and closes the player if its session is the one
    deleted (`sessGoneElsewhere`). Notes, flags, highlights and tags are the
    user's own writing and stay. Never remove a session any other way.
  The companion's "on question N" line clears when that device's beacon
  stops carrying a session (`COMP.live.dev`).
- **Only a question that was on screen counts as studied** (v6.79).
  Finishing a session grades what is left as omitted, but `finishSession`
  writes an attempt only for an omitted question with time on screen
  (`it.timeMs`), and none at all when nothing in the session was answered.
  Until v6.79 it wrote one for every unanswered question: four blocks of 20
  opened and finished on question 1 turned a day of 20 into "81 studied",
  emptied Unseen and put never-seen questions on the review schedule.
  `attGhosts()` names such attempts (nothing chosen, and either
  `studyMs === 0` or a finished session with no answer whose attempts were
  all written within 2 s), and `attDropGhosts()` removes them and replays
  their schedule at start-up and after every pull, because another device's
  log can carry them back. Any new path that writes attempts follows the
  same rule.
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
- **A rebuild keeps every measure unless the user drops it.** The first
  Progress rebuild (v6.69) cut about half the old statistics to a shorter
  list I judged better (completion by area, memory health, time split,
  sighting ladder, like-for-like comparisons, weekday accuracy, months), and
  the user found it "significantly less". When rebuilding a surface, carry
  over everything it measured, restructured where that helps; propose cuts,
  don't make them.
- **Every Progress figure is interactive (functional interactivity, v6.74,
  `DESIGN.md` §6).** A new figure, bar, legend or chart on Progress ships
  with its answer in the same change: a `data-pk` and a `PG_PK` builder for
  a figure, `data-pgf` inside a `data-pgfg` holder for a part, a peek of its
  period for a mark. A peek reads the same data as its card (refactor the
  card's maths into a shared function, as `pgCauses` and `pgCmpWin` are,
  rather than writing it twice), and its act practises exactly the
  questions it counts through `startIds(ids,label)` (pool `ids`; the user's
  block size and order; the builder's pool is restored after).
- **Progress measures what a session records, and says its window**
  (v6.69, the PROGRESS section; Areas since v6.70). `pgData()` joins every counted attempt
  (`allAttempts()`: ts, chosen, correct, answer time, question time, conf,
  miss, sess, seq) to its session item once per render: position in the
  block, block size and mode, `struckKey` (the key was crossed out) and,
  through the question, `hasLure` / `lure`. A new field a session starts
  recording is added there, and every view can use it. The definitions:
  - *Form* is accuracy on the last 100 first sightings (`PG_FIRST`), with a
    Wilson 95% range and the change on the 100 before (under 2.5 points is
    called level). Never a lifetime figure as the headline.
  - *Where the misses came from* puts each miss in one cause: left blank →
    time; else the user's tag; else, measured, the lure taken → distractor,
    the key crossed out, or an answer under half the median correct answer
    time (and under 8s floor) → rushed; else "not named". The card shows how
    many of each were tagged and how many came from the record.
  - *Where the marks are* ranks areas by marks at stake: miss rate × the
    bank's questions in that area, eight answers minimum in the range.
  - *Time* (v6.72): question time is `studyOf()` (everything spent on a
    question, every visit), answer time `ansOf()` (to the commit; none when
    left blank). The averages the user watches are **means**, question time
    over every attempt in the period and answer time over the answered ones,
    by day, week, month and all time (`pgTsum`, `pgTbuckets`). Session time is
    the session's own `elapsed`, shown as is, never recomputed. The user
    reads average question time as their pace and strength: keep it the
    lead figure of the Time view, and never change how the clocks are
    recorded without asking.
  - *Pace* uses 90 s as exam pace (`PG_PACE`); *stamina* reads position in
    blocks of 20 or more.
  - *Areas* reads any facet in `FACETS` (except answer confidence) through
    `facetValues`, so a new facet in `FACETS` only needs a row in
    `PG_AREA`. A question counts as done once it has any counted attempt;
    its state is its last answer (`pgQState`, cached per render with
    `PG.QS`, cleared wherever `PG.E` is). Practise sets `F.pool` to
    `unused` or `incorrect` plus that one facet value and calls
    `startSession`, the same path as the builder.
  Entry points stay `renderStats()` (data changed: rebuild the open view)
  and `renderHistory()`; `go('stats')` calls `renderStats(true)` (arrival:
  counts start from zero) and `showPane('stats', …)` hands over to
  `pgShow`, so the `history` alias still lands on Sessions. Readiness is
  still `readiness()`, shared with Home.
- **Home** (v6.77, the HOME section of the script, `DESIGN.md` §14):
  `renderDash(true)` is an arrival (figures from zero, cards rise in, the
  lower half one frame later); `renderDash()` is a live update and repaints
  in place, gliding keyed marks (`data-hk`) from their old value. Off Home it
  only refreshes the nav counts (Home rebuilds on `go('dash')`), so the many
  callers after imports and syncs stay cheap. It reads the Progress record
  (`pgData`, `pgQState`, `pgForm`) and clears `PG.E`/`PG.QS`/`PG.QM` first,
  so the two screens can never disagree. Every figure opens a peek through
  `pgPeek`; Home's own builders (`hz`, `h14`, `hlev`) live after `PG_PK` is
  declared (assigning to it earlier throws at start-up). `pgxGo` leaves for
  Progress when a peek's act is pressed elsewhere. The open-session card
  keeps `data-sessresume` / `data-compopen` / `data-sesspause` /
  `data-sessfinish`, and `plumeOpen` grows the player and the companion
  out of `.hm-sess`.
- **The session builder's setup is a setting** (v6.76): `DB.settings.build`
  (`bld()` fills it from `BLD_DEF`) holds mode, count (0 = every match),
  timing (`off` / `q` / `block`), secs, block (`pace` = 90s × questions, or
  `fixed` minutes in `mins`), onEnd, order, shuffle, conf. It is this
  device's own (not in `SHARED_SETTINGS`) and syncs in the device file.
  `startSession()` reads `bld()`, never the screen, so every way in (Due
  review, Weakest system, `startIds`, Progress's Practise) uses the same
  block size, timing and order; `pgBlockN()` reads it too. The pool and the
  filters (`F.pool`, `F.sets`) stay in memory as before.
- **A timed block** (v6.76): a session's `config.limit` (seconds for the
  whole block) and `config.onEnd` (`end` | `over`). The bar counts it down
  from `S.elapsed` plus the current visit (`blockLeft()`), so a pause stops
  it and another device resumes it where it stood; it replaces the
  per-question clock for the whole session, explanations included. At zero,
  `end` runs `blockUp()` (closes the viewer, dialog and menu, then
  `endSession()`), `over` turns the clock into overtime. Warnings at 5 and 1
  minutes fire once, only when crossed. Review clones set `limit:0`; a
  retry from the report never carries it. The report shows the allowance and
  how it ended (`blockChip`), the companion the time left. Sessions without
  `limit` behave exactly as before.
- **Images (the gallery)** (v6.81, THE GALLERY section): a derived index,
  rebuilt by `renderGallery(reset, arrive)` (`go('gallery')` passes
  `arrive`); taps change their own control at once and redraw a frame later
  through `galSoon`. `galIndex()` is shared with the gallery picker; keep its
  shape. Filters live in `GAL.sets` (`fmt`, with `__other` for none of the
  five formats; `slot`; `st`; `need` is the Waiting mode) and are tested in
  one place, `galTest(e, term, skipGroup)`, which the chip counts reuse.
  The viewer steps through `GAL.view` (the entries in drawn order, groups
  included), and `lbxSource` finds a tile by `#galBody [data-galidx]`.
  Gallery peeks are `PG_PK.gal` (installed by `galPkInstall()` on the first
  render, since `PG_PK` is declared later in the file). Group, layout and
  size, and which of the two panels (Arrange and filter, About your
  pictures; `galPanel`) are open, are this device's own, in
  `localStorage.nidus_gal_view`. The pictures come first (v6.82, the
  user's correction): never put statistics or controls back above the
  grid; add them to a panel.
- **The bank, the question page and the editor** (v6.84, the BANK, THE
  PAGE LAYER, THE QUESTION PAGE and THE EDITOR sections, `DESIGN.md` §14):
  - `renderBank(reset, arrive)`; `go('bank')` passes `arrive`. Taps flip their
    own control and redraw a frame later through `bkSoon`, gliding
    (`bkSnap`/`bkFlip`). Filters live in `BK.sets` (rec, key, lib: keys of
    `BK_T`; sys, sub, dif, lab: values) and are tested in one place,
    `bkTest(q, T, skipGroup)`, which the chip counts reuse. A new filter is a
    predicate in `BK_T` plus a row in `BK_CHIPS`. The sort and which panels
    are open are this device's own (`nidus_bank_view`).
  - `previewQuestion(id, seq)` opens the question page from anywhere; `seq`
    (the ids of the list it came from) turns on stepping. It closes an open
    dialog first, since the page sits under `#mask`. `qpHTML(q, {qid, mode})`
    draws a question in spine order and is shared with the editor's Preview
    (`qid:'@edit'`); change the page there, never twice.
  - `editQuestion(id|null)` opens the editor. Its option cards are tied to
    their choice objects (`QE.card`) and `qeSync()` writes each card into its
    own choice, keeping any other field the choice carries. Anything that
    moves options calls `edRelabel()` then `qePaintOpts()`. `qeCheck()` is
    the one list of what stops a save; the save path itself is unchanged
    (figure merge, permanent ID, fingerprint for a new question).
  - **Peek kinds share one namespace.** `PG_PK.bank` already belonged to
    Progress's Areas, so the bank's own lazy install found it taken and
    skipped itself, and every bank figure opened nothing (caught by the
    functional test). The bank's kind is `bnk`. Before adding a kind, grep
    `PG_PK.<name>`.
  - **A dialog that closes leaves focus on `body`.** A key handler bound to a
    page's own element then never hears the key (Ctrl+S after "Keep
    editing"). Shortcuts for a page go on `document` in the capture phase,
    guarded by `plyOpenNow(id)` and `#mask.on`.
- **Storage:** `nidus_db`, `nidus_base_<kind>`, `nidus_media_budget`,
  `nidus_straggler`, `nidus_gal_view` and `nidus_bank_view` are live keys. Never rename one without migrating the
  saved data. Any change to data shape must keep sync (CLOUD SYNC section)
  and import/export working.
- Ship with `ship ~/projects/nidus-app "<message>"` (from the global
  instructions). Never push code that fails `htmlcheck`.

## Testing locally

- **Test on the user's two devices, tablet first.** Nidus is studied mostly
  on a Galaxy Tab S6 Lite in **portrait** (viewport about 800×1220,
  `deviceScaleFactor` 1.5), then a 6.7" phone (about 412×860, 2.625). Shoot
  every visible change at both; 390px alone missed that the 2.7 viewer card
  left a zoomed picture no room.

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
- **Measure every well against what it sits on, not against the page.**
  For each element whose computed background has alpha (the `--p-well`
  veil), find the nearest ancestor with an opaque background, composite
  the two through a 1×1 canvas and take the OKLab L of each: |ΔL| must be
  at least 0.05 for every well on Progress and Settings, in both themes and
  at contrast 0, 1 and 2 (`DB.settings.contrast`). Measuring only the
  page-to-card gap missed that the old fixed-tone well was the page's own
  colour wherever it sat on the page (v6.73). The engine writes the roles
  as inline styles on `<html>`, so a CSS experiment needs `!important`.
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
- **Test a drag on a layer with CDP touch events, and log the pointer
  events first.** A `pointercancel` right after the first move means the
  browser took the gesture: `touch-action` is read from the touched element
  up to the nearest scroller only, so a scroller inside the layer needs
  `touch-action: pan-y` too (Settings drag-back, v6.61). Check that a short
  drag springs home, a long one or a flick closes, a vertical swipe still
  scrolls, and a slider inside still drags.
- **Film motion at the tablet's size with the CPU throttled.** Per-frame
  `m41` logs at 800×1220 (dpr 1.5) under `Emulation.setCPUThrottlingRate`
  4 show gaps of 33–67ms that the phone size never shows: the Settings
  slide (a spring stepped per frame) looked perfect on the phone and
  glitchy on the tablet (fixed in v6.66 by moving it to the compositor).
  Once a motion runs on the compositor, the rAF log only shows main-thread
  samples; judge it by whether the values follow the curve, not by gaps.
- **Measure a corner morph, don't eyeball it.** Log the element's computed
  `borderTopLeftRadius` (and `scale`) on every animation frame through
  `pointerdown`, a hold and `pointerup`. Every frame should step; a value
  that sits above half the element's height and then lands in one or two
  frames means it rests at 999px (fixed in v6.60 for Settings).
- **Test option tables and picture options through the real importer.**
  Build a bank with a two-column table, a five-column table (it must turn
  into heading/value pairs on the phone and stay columns on the tablet), a
  question whose options are all pictures (one hung on the option itself),
  and one with an empty picture slot; `importText` it and check
  `JSON.stringify(DB.questions)` holds no `data:image/`. Then, with real
  touch: a tap on an option picture chooses it and the viewer stays shut,
  the corner button opens the viewer with the caption held, Back closes
  it; after Submit the captions show. Round-trip the editor (edit a cell,
  remove option A of the picture question: B's picture becomes A's), Export
  missing figures (`slot: "choice"`, `choice: "B"`) and fill it, re-import
  with update (option pictures kept) and re-run export (columns, cells,
  slots carried).
- **Profile a view's build per card.** In the page, time each builder five
  times (`pgFormHTML`, `pgHeatCard(...)`, …) on the seeded bank and fix the
  outlier: a `toLocaleDateString` per heat-map cell cost 20ms (cache
  `Intl.DateTimeFormat`, `pgDate`), and a linear scan per chart point was
  quadratic (binary search in `pgForm`). Then log long tasks
  (`PerformanceObserver` `longtask`) with the indicator's `m41` per frame at
  `setCPUThrottlingRate` 4, waiting 2.5s between taps so the last view's
  count-ups are not measured as the next one's cost.
- **Measure text collisions, don't trust one screenshot.** For every card in
  every Progress view, at 320 / 360 / 390 / 412 / 800px and in both themes,
  collect the client rects of each text node (clipped to any ancestor with
  overflow, so ellipsis and closed unfolds don't count) and report pairs
  that overlap by more than a third of a line, plus text past the stage's
  edge. Zero is the bar. (The scratch `overlap.js`; it found the hours,
  week-column and stamina collisions in v6.69 that the 412px shots hid.)
- **Drive every interactive element on Progress.** In the page, for each
  view, click every `[data-pk]` in turn and record `pgxIsOpen()`, the
  peek's title, its text length and its acts, then `pgxClose()`; click the
  first part of every `[data-pgfg]` twice (isolated, then cleared); finally
  run one act and check a session started with the right label and that
  `history.state` holds no `pgx`. A peek that stays shut means a key with
  no builder (`data-pk="ta"` did, v6.74). Then take full-page shots of
  every view in both themes: a `div` turned into a `button` re-matches old
  descendant selectors (the scratch `t.js`).
- **A seed for Progress** needs months of history to show anything: the
  scratch generator writes ~3,500 questions, ~80 sessions over 75 days with
  rest days, confidence, miss tags, lures on 60% of questions and crossed-out
  options in the session items. Fewer questions than answers seen leaves
  coverage near 100%, which hides the pace and coverage cards.
- **Audit durations by rendered text.** After seeding, read each Progress
  view's `innerText` and look for any time outside the DESIGN.md duration
  rule (a lone `65m`, `90s` in prose, `0m 45s`, `1h 0m`, `12:34`); a
  duration can also hide in a `pgN` tile with a time unit, an aria-label or
  a hard-coded bucket label (v6.75).
- **Test the block clock without waiting.** Start a session through
  `bsSet({...});startSession()`, then `flushTime();qEnter=Date.now();
  S.elapsed=(limit-301)*1000` and wait ~1.5s for the 5-minute warning, or
  set it past the limit for the end or the overtime. Read `#pTimer`'s text
  and class, `S.status`, `#resultsPanel.on` and the report's chips. The
  clock's text uses a no-break space (`fmtT`), so match `\s`-free patterns.
- **The bar's counter is already a few pixels tight** with the fallback font
  in headless Chromium (`.sp-count` 4–8px over at 360/390/430 with the
  per-question clock, before v6.76 too). Anything added to the bar must not
  make it worse than that baseline; measure both.
- **Test a live update on Home, not just the arrival.** Push a few
  attempts for today into `DB.stats`, call `renderDash()` and log, per
  animation frame, the ring's computed `strokeDashoffset`, today's pebble
  `DOMMatrix.d` and the count's text: all three must travel. A value that
  sits at its final number from the first frame means a layout read ran
  before the old style was set (DESIGN.md §6, 2.17).
- **Profile a screen's build per builder**, best of five (`hmToday`,
  `hmVitals`, … for Home), then time the whole `go()` to the next
  `requestAnimationFrame`. The builders summed to 28ms while the render
  took 65ms: the rest was parsing and layout, fixed by building the lower
  half a frame later, not by tuning the maths.
- **Seeding the gallery:** `figTake` re-runs the classifier on the
  question (`figTouch`), so set `q.resources` *after* attaching pictures, and
  spread formats across the questions that actually got one, or a format
  filter reads 0 and looks broken. Drive the screen with real taps: chips,
  the three switches, Select then two tiles, Waiting, every `[data-pk]`
  (each must open with acts), a tile into the viewer and `page.goBack()`.
- **Audit a surface's alignment by script, every state.** The scratch
  `m.js` (Images, v6.83) walks twelve states (first, filters open, filtered,
  summary, grouped, true shape, three sizes, waiting, selecting, search) at
  360 / 412 / 800px and prints, against the content box: the left and right
  edge of each row *by its drawn text or glyph* (Range rects, an icon's
  `svg` box), the vertical centre of every item in a row, the first chip of
  each chip row, anything past the viewport not inside a scroller, and text
  collisions clipped to overflow boxes, skipping rects of the same text node
  (an ellipsised line returns two). Every edge must read 0 (or the intended
  inset); one screenshot hid a 10px search offset, a 6px header indent and a
  3px grid inset.
- **Drive the bank, the page and the editor end to end** (the scratch
  `func.js`, `func2.js`): chips (a group ORs, groups AND, the chip's count
  equals the list it brings in), each sort's order, search reaching labels
  and notes with the passage shown, every About peek opening with acts and
  Show only these filtering, long-press (480 ms with the mouse held) selecting
  without opening, each bulk act, the armed delete leaving tombstones, the
  page stepping, flagging, labelling and saving its note, Escape closing only
  the menu, the editor's key / lure ceiling / move (key and pictures follow) /
  two-tap remove / add / blocked save / preview, Back with changes asking
  first, Ctrl+S, a new question, the editor over the player, and
  `history.state` after each close. Wait out a dialog's exit (about 1 s)
  before the next key: while `#mask.on` the page ignores shortcuts on
  purpose. Attach a real picture to an option with
  `figTake('@edit', null, 'choice:B', [file])` before moving it.
- **Check copies:** replace `window.copy` with a function that records the
  text, then click the copy buttons and compare the strings.
