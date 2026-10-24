# README — AHCPCM303 Interactive Learning Resource (3D Edition)

# AHCPCM303 — Identify Plant Specimens (Interactive Learning Resource)

A self-contained HTML learning resource for the Australian vocational unit
**AHCPCM303 Identify Plant Specimens**, contextualised for Tasmania.

The entire resource — text, quizzes, database, glossary, CSS, JavaScript,
and Three.js 3D scenes — lives in **one `.html` file**. There is no build
step, no npm, no framework, and no server requirement.

---

## 1. Quick start

1. Save `3D Plants.html` anywhere on your computer.
2. Double-click it, or drag it into any modern browser
   (Chrome, Edge, Firefox, Safari — Edge/Chrome recommended).
3. That's it. The resource runs entirely client-side.

**Internet required for first load** because two external CDNs are used:

- Font Awesome 6.4.0 (icons) — `cdnjs.cloudflare.com`
- Three.js 0.160.0 (3D scenes) — `unpkg.com`
- Google Fonts (Inter) — fetched automatically

If the CDNs are blocked or you're offline, **the entire text-based
resource still works**. Only the 3D scenes degrade — they display a
friendly fallback message inside their scene container. See
*Degradation behaviour* below.

---

## 2. File structure

The file is organised in a strict order. Editing requires respecting
this order, especially around the head:

```
<!DOCTYPE html>
<html>
<head>
  <meta charset>, <meta viewport>, <title>
  <link> Font Awesome
  <script type="importmap">          ← ONLY ONE. Do not add a second.
  <script type="module">             ← ONLY ONE. Bootstraps THREE on window.
  <style> … all CSS … </style>
</head>
<body>
  <a skip-link>
  <header>
  <div class="mobile-overlay">
  <div class="container">
    <aside class="sidebar">          ← TOC + search + save/load
    <main class="main-content">      ← all tab-content divs
      #welcome
      #section1
      #section2
      #leaf3d                        ← NEW 3D SCENE 1
      #section3
      #section4
      #section5
      #forest3d                      ← NEW 3D SCENE 2
      #section6
      #section7
      #section8
      #press3d                       ← NEW 3D SCENE 3
      #section9
      #section10
      #database
      #glossary
      #appendices
    </main>
  </div>
  <footer>
  <script> … all JS … </script>
</body>
</html>
```

### Important structural rule for future edits

Each `#leaf3d`, `#forest3d`, `#press3d` block **must be a sibling** of
its parent section's `tab-container` div, not a child of it. If a 3D tab
is nested inside its parent section, it collapses to zero height when
the parent tab is hidden — and the scene renders into a 0×0 canvas that
is invisible on screen but reports no errors.

This is the single most common way this file breaks. See section 8
(*Troubleshooting*) below.

---

## 3. What's inside

### Content modules

| Tab | Content |
|---|---|
| Welcome | Overview, Tasmanian flora intro, flashcards |
| Section 1 | Taxonomic hierarchy, binomial nomenclature, divisions, family matching game, quiz |
| Section 2 | Leaf morphology terms, flashcards, interactive leaf key, quiz |
| **Section 2 → 3D Leaf** | **3D leaf cross-section model (three species)** |
| Section 3 | Flower structure, fruit types, flower parts quiz |
| Section 4 | Stem types, bark types, root systems |
| Section 5 | Cones, spores, gymnosperms, ferns, quiz |
| **Section 5 → 3D Forest** | **3D forest strata & disturbance simulation** |
| Section 6 | Dichotomous keys, two key simulations, key construction activity |
| Section 7 | Conservation legislation, 30+ threatened species profiles |
| Section 8 | Collection protocols, pressing techniques, herbarium template |
| **Section 8 → 3D Pressing** | **3D specimen pressing simulator** |
| Section 9 | 3 practice exams, field assessment checklists |
| Section 10 | Reporting templates, workplace checklists, JSA |
| Master Database | 130+ Tasmanian native species, filterable, sortable, paginated |
| Glossary | 120+ botanical terms with definitions and Tasmanian examples |
| Appendices | Useful resources, video library, unit mapping matrix |

### Data persistence

Two independent localStorage keys are used:

- **`ahcpcm303Progress`** — stores `currentTab`, `completedSections`,
  and every field with a `data-question` attribute. Cleared via the
  **Save Progress** / **Load Progress** buttons in the sidebar.
- **`herbariumRecord`** — stores only the herbarium template fields
  (Section 8). Cleared via the **Save Record** / **Load Last** buttons
  inside that template.

Neither key expires. If you need to reset everything, open DevTools →
Application → Local Storage → delete both keys.

### Persistence contract — important for future edits

**Any input, select, or textarea that should survive a Save/Load cycle
MUST have a unique `data-question="..."` attribute.** The save and load
functions iterate `[data-question]` elements. If you add a new field
and forget the attribute, students will fill it in and silently lose
their answer on reload.

Checkboxes need extra care — see the `saveProgress` function; it
detects `input.type === 'checkbox'` and stores `.checked` instead of
`.value`.

---

## 4. The three 3D scenes

Each 3D scene lives in its own `tab-container` (`#leaf3d`, `#forest3d`,
`#press3d`) and initialises on demand via a lazy-init pattern.

### 4.1 Leaf Cross-Section Model (`#leaf3d`)

- **File location:** between `#section2` and `#section3`
- **Init function:** `initLeafScene()`
- **Container ID:** `#leaf-canvas-wrap`
- **Canvas wrap height:** 460px desktop, 340px mobile (`@media max-width: 700px`)
- **Interaction:**
  - Drag to rotate (custom pointer-event orbit — no external dependency)
  - Scroll to zoom
  - Toggle species: `leafSetModel('richea' | 'bedfordia' | 'eucalyptus')`
  - Slice-depth slider scrubs an orange plane through the leaf model
- **Readout elements:** `#leaf-readout-species`, `#leaf-readout-boundary`,
  `#leaf-readout-waterbar`, `#leaf-readout-water`
- **Scene state:** `window.leafSceneRef` — holds `{ scene, camera,
  renderer, leafGroup, slicePlane, buildLeafModel, setReadouts,
  variant, slicePercent }`

### 4.2 Forest Strata & Disturbance (`#forest3d`)

- **File location:** between `#section5` and `#section6`
- **Init function:** `initForestScene()`
- **Container ID:** `#forest-canvas-wrap`
- **Interaction:**
  - Drag to rotate
  - Scroll to zoom
  - `forestFellCanopy()` — fells the first visible canopy tree
  - `forestResetScene()` — restores all trees and resets sliders
  - Time slider (0–100 years) scales up shrubs and ferns
- **Readout elements:** `#forest-readout-cover`,
  `#forest-readout-lightbar`, `#forest-readout-light`,
  `#forest-readout-gap`
- **Scene state:** `window.forestSceneRef`

### 4.3 Specimen Pressing Simulator (`#press3d`)

- **File location:** between `#section8` and `#section9`
- **Init function:** `initPressScene()`
- **Container ID:** `#press-canvas-wrap`
- **Interaction:**
  - Drag to rotate
  - Scroll to zoom
  - `pressFlipLeaf()` — flips the next unflipped leaflet
  - `pressToggleFold('top' | 'bottom')` — raises leaflets in that half
  - `pressDoPress()` — flattens leaflets and evaluates the arrangement
  - `pressResetScene()` — restores all leaves and fold state
- **Readout elements:** `#press-readout-features`,
  `#press-readout-fit`, `#press-readout-mould`
- **Scene state:** `window.pressSceneRef`

### 4.4 Lazy initialisation

Scenes are only built when their tab is first opened. The logic lives
in `switchTab()`:

```js
if (!sceneState.initialised.has(tabId)) {
    const initialisers = {
        'leaf3d': initLeafScene,
        'forest3d': initForestScene,
        'press3d': initPressScene
    };
    const initFn = initialisers[tabId];
    if (initFn) {
        sceneState.initialised.add(tabId);
        waitForTHREE(initFn, 60, 100);
    }
}
```

`waitForTHREE(callback, retries, delay)` polls `window.THREE` until it
exists or retries are exhausted. This is necessary because the
`importmap`-based module script loads asynchronously, and a user could
click a 3D tab within milliseconds of page load — before THREE is ready.

If THREE never arrives (offline, blocked CDN), the scene's own
`try/catch` calls `showSceneFallback()` and displays a polite message
inside the scene container. **The rest of the resource still works.**

### 4.5 Adding a new 3D scene

If you need a fourth scene, follow this recipe exactly:

1. Add a new `tab-container` div as a **sibling** of its parent
   section — not a child.
2. Add a `<div class="scene-canvas-wrap" id="your-canvas-wrap">`
   inside it. The CSS class already gives it explicit height and
   `position: relative`.
3. Add a `waitForTHREE`-wrapped init function `initYourScene()` that
   follows the pattern of `initLeafScene`.
4. Register it in the `initialisers` object inside `switchTab()`.
5. Add `console.log('[your-scene] 3D scene initialised')` at the end
   of successful init and `console.warn('[your-scene] …', err)` in
   the catch.
6. Verify with the diagnostic in section 8.

---

## 5. CSS architecture

All scene-related classes are grouped at the bottom of the `<style>`
block, immediately before `</style>`. The full contract:

```css
.scene-container        /* outer card */
.scene-header           /* title + subtitle bar */
.scene-canvas-wrap      /* CRITICAL: has explicit height */
.scene-hint             /* animated "drag to rotate" badge */
.scene-readout          /* live data strip below canvas */
.scene-toggles          /* button row */
.scene-toggle           /* individual toggle button */
.scene-callout          /* explanatory callout below scene */
.scene-slider-row       /* slider + value label */
.scene-slider-value     /* numeric value beside slider */
.scene-placement-help   /* "reinforces Section X" note */
.scene-fallback         /* shown when init fails */
.scene-spinner          /* shown before THREE loads */
```

**Critical rule:** `.scene-canvas-wrap` MUST have an explicit `height`
(non-zero). Without it, `wrap.clientHeight` is `0`, the Three.js
renderer is created at `0×0`, and the scene appears blank with no
console errors. The current value is `460px` desktop / `340px` mobile.
Do not remove or override these.

The canvas itself is styled by:
```css
.scene-canvas-wrap canvas {
    width: 100%;
    height: 100%;
    display: block;
}
```

---

## 6. JavaScript architecture

The `<script>` block at the bottom is organised into labelled sections:

```
GLOBAL STATE               currentTab, tabs[], userProgress
SPECIES DATABASE           speciesDatabase[] (130+ entries)
PAGINATION STATE           currentPage, rowsPerPage, filteredSpecies
3D SCENE LAZY-INIT         sceneState, waitForTHREE, showSceneFallback
TAB SWITCHING              switchTab (with lazy-init hook)
PROGRESS                   updateProgress, markSectionComplete
FLASHCARDS                 flipCard, showNextCard, showPrevCard
SECTION 1: FAMILY MATCHING checkFamilyMatch
SECTION 2: KEY SIMULATION  keyChoice, resetKey
3D SCENE 1: LEAF           initLeafScene, leafSetModel
3D SCENE 2: FOREST         initForestScene, forestFellCanopy, forestResetScene
3D SCENE 3: PRESSING       initPressScene, pressFlipLeaf, pressToggleFold,
                           pressDoPress, pressResetScene
SECTION 2/3/5 QUIZ HELPERS checkLeafQuiz, checkFlowerQuiz, checkReproQuiz
SECTION 6 KEYS             rainforestKeyChoice, eucalyptKeyChoice,
                           resetRainforestKey, resetEucalyptKey
SECTION 6/7/8/10 QUIZZES   checkKeyQuiz, showKeyModelAnswers,
                           checkThreatenedQuiz, showThreatenedModelAnswers,
                           checkHerbQuiz, showHerbModelAnswers,
                           checkWorkplaceQuiz, showWorkplaceModelAnswers
EXAMS (SECTION 9)          showExamAnswers, checkExam, clearExamAnswers
HERBARIUM                  saveHerbariumRecord, loadHerbariumRecord,
                           printTemplate
SPECIES DB TABLE           renderSpeciesTable, filterSpeciesTable,
                           sortTable, changePage
SAVE/LOAD PROGRESS         saveProgress, loadProgress
SEARCH                     setupSearch, searchGlossary
BOOTSTRAP                  DOMContentLoaded listener → wires all buttons
```

### Naming conventions

- `initXxxScene()` — scene initialisers, one per scene
- `xxxSceneRef` — global state object for a scene
- `xxxFunction()` — event handler exposed to `onclick` attributes
- `checkXxx()`, `showXxx()`, `resetXxx()` — quiz and sim handlers

### Conventions in the 3D init functions

Every `initXxxScene()`:

1. Finds its canvas wrap element and bails early if absent (with
   `console.warn`).
2. Wraps its whole body in `try { … } catch (err) { … }`.
3. Writes a spinner into `wrap.innerHTML` **before** any async work.
4. Writes `wrap.innerHTML = ''` **immediately before**
   `wrap.appendChild(renderer.domElement)` so the spinner doesn't sit
   under the canvas.
5. Uses a `ResizeObserver` to respond to layout changes.
6. Ends with `console.log('[scene-name] 3D scene initialised')`.
7. On error, calls `showSceneFallback(container, message)` and
   `console.warn('[scene-name] initialisation failed:', err)`.

Preserve these invariants in any new scene.

---

## 7. Verification checklist

Before shipping any edit, open DevTools console and run these:

```js
// 1. Exactly one importmap
document.querySelectorAll('script[type="importmap"]').length
// → 1

// 2. Exactly one module script (bootstrap)
document.querySelectorAll('script[type="module"]').length
// → 1

// 3. No <script> inside <style>
document.querySelectorAll('style script').length
// → 0

// 4. THREE loaded
typeof window.THREE
// → "object"

// 5. No duplicate IDs anywhere
[...document.querySelectorAll('[id]')]
    .map(el => el.id)
    .filter((id, i, arr) => arr.indexOf(id) !== i)
// → []

// 6. No duplicate data-question values
[...document.querySelectorAll('[data-question]')]
    .map(el => el.dataset.question)
    .filter((q, i, arr) => arr.indexOf(q) !== i)
// → []

// 7. Every Section 9 field has a data-question
[...document.querySelectorAll('#section9 select, #section9 textarea, #section9 input[type="text"]')]
    .filter(el => !el.dataset.question).length
// → 0
```

Manual smoke test — for each of the three scenes:

1. Open the tab.
2. Confirm console shows `[scene-name] 3D scene initialised`.
3. Confirm the canvas is visible (not blank, not 0-height).
4. Drag to rotate — the model should move.
5. Scroll — the model should zoom.
6. Exercise every toggle and slider once.
7. Switch to a different tab and back — the scene should persist
   (it does not re-initialise; `sceneState.initialised` prevents that).

---

## 8. Troubleshooting

### Scene is blank, but console says "3D scene initialised"

Most common cause: the `.scene-canvas-wrap` height has collapsed to 0
because an ancestor is `display: none`. Run this in the console while
the scene tab is open:

```js
(() => {
    const ids = ['leaf-canvas-wrap','forest-canvas-wrap','press-canvas-wrap'];
    return ids.map(id => {
        const w = document.getElementById(id);
        return { id, wrapHeight: w && w.clientHeight,
                 canvasExists: !!(w && w.querySelector('canvas')) };
    });
})()
```

If `wrapHeight` is `0` but the canvas exists, the scene is nested inside
a hidden ancestor. Open the Elements panel and walk up from
`#leaf-canvas-wrap` until you find a `display: none` — that ancestor is
the problem. The 3D `tab-container` divs must be **siblings** of their
parent section, not children.

### Scene is blank and console says "Failed to load Three.js"

The CDN is blocked. Either allow `unpkg.com` and reload, or accept that
the 3D is unavailable — the rest of the resource works fine. The scene
will show its own fallback message.

### A quiz doesn't respond to Check Answers

Open the console and look for a specific error. Common causes:

- Duplicate `id` on a question container (Section 1 had this — see the
  audit notes). Run the duplicate-ID check (item 5 above).
- A `data-question` value that doesn't match what the check function
  is looking for. Search the JS for the specific `getElementById` call
  and confirm the ID exists exactly once in the HTML.

### Save Progress doesn't persist a field

The field is missing `data-question`. Run item 7 above. Every persistent
field must have a unique `data-question` value.

### Exam answers come back blank after Load Progress

Same cause — the exam fields in Section 9 need `data-question="examN-qX"`
attributes. See item 7 above.

### The 3D model is visible but looks wrong / is cut off

Scroll-zoom may have been pushed too far in or out. Click anywhere
outside the canvas and try dragging to reset the view, or reload the
page. If the model still looks wrong, the camera setup or the model
geometry has been edited. Check the initial `camera.position.set(...)`
values in the relevant `initXxxScene` function.

### The model renders but is completely black

Possible WebGL/GPU issue. Try:

1. Reload the page.
2. Test in a different browser.
3. If it persists across browsers, the lighting setup has been edited —
   check that `AmbientLight` and `DirectionalLight` instances are still
   added to the scene in the init function.

---

## 9. Known limitations

- **Offline mode:** 3D scenes and Font Awesome icons don't work.
  Everything else does.
- **Older browsers:** The 3D requires WebGL2 support and ES modules
  with import maps. IE11 and pre-2020 Safari will fail entirely on the
  module script — the whole page still renders though, because script
  module failures are silent.
- **Printing:** 3D scenes are hidden in print view (`@media print`
  hides `.scene-container`).
- **Mobile:** Canvas wrap reduces to 340px height below 700px width.
  Touch gestures work (drag = rotate), but pinch-zoom is not
  implemented — scroll-to-zoom works with mouse or trackpad only.
- **Persistence:** localStorage is per-origin. Opening the file from
  `file://` in different browsers gives separate progress. Opening from
  a different path (e.g. moving the file) may also reset progress,
  depending on browser behaviour.

---

## 10. Editing this file — golden rules

If you take nothing else away, take these:

1. **Never add a second `<script type="importmap">`.** The browser
   silently ignores all but the first, and the module graph breaks with
   no error.

2. **Never add a second bootstrap `<script type="module">`** that
   re-imports THREE. One is enough.

3. **Never remove the explicit height from `.scene-canvas-wrap`.**
   Without it, the canvas is 0×0 and nothing is visible, with no
   console error.

4. **When adding a new 3D `tab-container`, make it a sibling of the
   parent section, not a child.** See the audit notes for the exact
   mistake and its symptom.

5. **When adding or changing an attribute on an existing input,
   replace the entire opening tag.** Do not paste a new tag in front
   of the old one — nested form controls break everything downstream.

6. **Every persistent field needs `data-question="unique-id"`.** No
   exceptions. Uniqueness across the whole file matters — a duplicate
   `data-question` value means one field silently shadows another.

7. **Test the whole file after any edit.** Use the checklist in
   section 7. It takes 30 seconds and saves hours.

---

## 11. Credits and provenance

- **Unit:** AHCPCM303 Identify Plant Specimens
- **Sector:** Conservation and Ecosystem Management
- **Contextualisation:** Tasmania, Australia
- **Content verification:** Tasmanian Herbarium records, EPBC Act
  listings (2025), TSP Act 1995 schedules
- **Original author:** Jay Rowley © 2026
- **3D scenes and structural fixes:** added 2026

---

## 12. Version history

### v2 (3D edition) — current

- Added three interactive 3D scenes: Leaf cross-section, Forest strata
  & disturbance, Herbarium pressing simulator
- Fixed duplicate IDs in Section 1 (`q1-2` / `q1-3` collision between
  the matching game and the quiz)
- Fixed the stray `</li>` in the sidebar TOC
- Added `data-question` attributes to all Section 9 exam fields
- Added checkbox support to `saveProgress` / `loadProgress`
- Corrected `aria-label` on Exam 2 and Exam 3 "Clear Answers" buttons
- Extended `switchTab()` with lazy-init hooks for 3D scenes
- Added `waitForTHREE()` and `showSceneFallback()` helpers

### v1 — original

- Text, quizzes, species database, glossary, exams, herbarium template,
  workplace tools
- Ten content sections
- Progress save/load via localStorage
- Font Awesome icons and Inter webfont
- The three structural issues listed above
```

---

Save that as `README.md` in the same folder as `3D Plants.html`. If you'd rather have it inside the HTML file itself as a hidden developer comment (so it can never be separated from the resource), let me know and I'll give you a version wrapped in `<!-- … -->` that you can paste just after the opening `<html>` tag — but I'd recommend the separate file. It's easier to read, edit, and version-control.
