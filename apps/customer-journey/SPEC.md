# Customer Journey

> Structured customer journey maps — a grid of steps × slices, grouped into stages — replacing the freeform Axure canvas.

## Idea

### Core concept
A single-file tool for creating and maintaining **customer journeys** as a **structured grid**: **columns = steps** of the process (grouped into **stages**), **rows = slices** (7 fixed dimensions: actions, touchpoints, mindsets, emotions, pain points, ideas, insights). CJs are grouped by **process category** and linked to **personas**. The grid lays itself out — no more manually shoving elements around a freeform Axure canvas.

### User problem
- **Manual work in Axure**: today CJs live on a freeform canvas → a lot of time goes into managing space and manually adding/moving elements (steps, entrypoints, pushes) instead of into content. Any change means reshuffling everything.
- **Hard to present a change**: showing how a CJ will change after introducing a modification is tedious today and done by hand.

### Target user
A UX designer (the author) working on customer journeys in financial processes (e.g. cash loan, account opening). A single author; a personal tool, data stored locally.

### Core action
Open a customer journey → edit the **grid of steps × slices** (add/move steps and stages, manage independent elements in cells) → have it instantly organized and ready to show (PNG export).

### Scope
- **MVP**:
  - Process categories (cash loan, account opening…) with a CJ counter + a persona pool.
  - List of CJs in a category (persona + short case description).
  - **The CJ editor as a grid**: stages grouping steps (columns — add, move, rename, delete), 7 fixed slices (rows); cells with multiple independent text elements (drag to reorder, delete individually).
  - **Export a whole CJ as PNG**.
- **Later** (future Changes):
  - Variants: a copy of a CJ → a variant; the base CJ marked as "current"; a variant = base + changes.
  - Version history of CJ edits.
  - Change presentation (side-by-side before/after / diff) — delivery format to be specified.
  - Images/screenshots in cells (→ IndexedDB for blobs).

## Requirements

### Data model
- **`category`** — a business process (e.g. cash loan, account opening); `name`. Many. Parent of `journey`.
- **`persona`** — a character from the persona pool; `name`. Many, shared across CJs. CRUD.
- **`journey`** (customer journey) — `title`, `description` (scenario), `expectations`, `personaId`; belongs to a `category`. Many per category.
- **`stage`** — `name`; an ordered list within a `journey`; groups `step`s. E.g. Form, Post-submission handling. (NN/g "Journey Phase".)
- **`step`** — `name`; belongs to a `stage`; ordered. A column in the grid.
- **`slice`** — one of 7 fixed rows (global, not per CJ): `action`, `touchpoint`, `mindset`, `emotion`, `pain-point`, `idea`, `insight`.
- **`cell`** — the intersection of (`step`, `slice`); contains an ordered list of `element`s.
- **`element`** — a single, independent text entry in a `cell` (`text`); order managed, deleted individually. Text only in the MVP.
- **Lifecycle**: deleting a `category` → cascade (journeys → stages → steps → cells → elements). Deleting a `persona` used by a `journey` → ask, or null + a notice.
- **Storage**: metadata/state → `localStorage` under `ux-customer-journey_v1` (a single `state` object, `save()`/`load()` in `try/catch`, migrations). No blobs in the MVP → no IndexedDB. (Images in cells = Later → IndexedDB then.)

### Actions
| Action | Entity | Description |
|--------|--------|-------------|
| Create / Rename / Delete / Reorder | category | manage process categories |
| Create / Rename / Delete | persona | persona pool (shared) |
| Create / Rename / Delete / Switch | journey | CJs in a category; set persona + description + expectations |
| Add / Rename / Delete / Reorder | stage | stages in a CJ |
| Add / Rename / Delete / Reorder | step | steps within and between stages |
| Add / Reorder / Delete | element | independent elements in a cell (drag to reorder) |
| Set sentiment | emotion cell | emotion value per step (emotion curve) |
| Export PNG | journey | render the grid → PNG (vanilla: SVG/Canvas → toBlob) |

**Core**: grid editing — adding steps/stages and managing elements in cells. `save()` on every change.

### Screens / views
- **Header**: tool title + "New CJ" and "Export PNG" actions (for the active CJ).
- **Sidebar**: list of **categories** (with a CJ counter) + a **personas** section (the pool) + list of **CJs in the active category** (persona + short description). Drag-to-reorder, hover-actions (✎/×), inline-rename — like in image-slicer.
- **Main panel = CJ grid**: a **stages** bar spanning the **step** columns; 7 **slice** rows; cells editable in place (add element, drag to reorder, delete). The `emotion` row = sentiment value (curve). A frozen column with slice labels; horizontal scroll when there are many steps.
- **Empty state**: no categories/CJs → instruction "Add a category and your first CJ".

### States
- **Empty**: empty app (no categories) and an empty CJ (no steps) → a message + the first action.
- **Error**: `localStorage` full / save error → a polite message (not `alert()`); `save()` in `try/catch`.
- **Validation**: light — names required on creation, auto-name when empty (e.g. "Step 1", "Stage 1") like in image-slicer.

### Interactions
- **Drag & drop**: reorder steps (within and between stages), reorder stages, reorder elements in a cell.
- **Paste (Ctrl+V)**: text into the active cell — optional.
- **Export**: the active CJ → PNG (download via `<a download>`; optionally copy to clipboard with "✓ Copied" / "✗ Failed" feedback).
- **Keyboard**: Enter = save inline edit, Escape = cancel.
- **Live preview**: changes in the grid are visible immediately.

### Edge cases
- Deleting a category → cascade over all its content.
- Deleting a persona used by a CJ → ask, or null + mark.
- An empty CJ with no steps → the grid empty state.
- A very large number of steps → horizontal scroll (frozen slice-labels column).
- `localStorage` full → a message, not silent loss.
- Active index after a delete → `ensureActiveIndicesValid()` (category/CJ/stage/step never out of range).

## Glossary
| Term | Code name | Definition |
|-----------|----------------|------------|
| process category | `category` | A group of business processes (e.g. cash loan, account opening) that groups customer journeys. |
| persona | `persona` | A user character that a given customer journey pertains to (shared pool). |
| customer journey | `journey` | A map of the customer's path through a given process — a grid of stages/steps × slices. |
| stage | `stage` | A horizontal lane grouping steps (NN/g "Journey Phase"); e.g. Form, Post-submission handling. |
| step | `step` | A single granular stage — a **column** in the grid, belongs to a `stage`. |
| slice (dimension) | `slice` | A row in the CJ grid — one of 7 fixed lanes: `action`, `touchpoint`, `mindset`, `emotion`, `pain-point`, `idea`, `insight`. |
| customer action | `action` | Slice: what the customer does in a step. |
| thoughts / questions | `mindset` | Slice: what the customer thinks, what they look for, motivations. |
| emotion | `emotion` | Slice: the sentiment value in a step (scale) → emotion curve. |
| pain point | `pain-point` | Slice: a difficulty/frustration in a step. |
| idea | `idea` | Slice: an improvement proposal for a step (opportunity). |
| insight | `insight` | Slice: a conclusion / research observation for a step. |
| touchpoint | `touchpoint` | Slice: a customer contact point (entrypoint, push, channel). |
| entrypoint | `entry-point` | Touchpoint: the customer's entry into a step/channel. |
| push | `push` | Touchpoint: a push notification sent to the customer. |
| cell | `cell` | A list of elements at the intersection of a step (column) and a slice (row). |
| element | `element` | A single independent text entry in a cell (there can be many; order managed, deleted individually). |
| theme | `theme` | UI theme: `light` (default), `dark`, `auto` (follows `prefers-color-scheme`); persisted in `state.theme`. |
| scenario | `scenario` | CJ header: the situation/goal the map pertains to (NN/g Scenario). |
| expectations | `expectations` | CJ header: what the customer expects (NN/g Expectations). |
| variant | `variant` | (Later) A copy of the base CJ with introduced changes. |
| version | `version` | (Later) A saved CJ state in the edit history. |
| base / current | `base` / `current` | (Later) The CJ marked as current, from which variants are created. |

## Changes

### Change 1 — feature: english UI + accessibility + light/dark theme (2026-07-27)
**Status**: planned — route to ux-build

**User goal**
The tool should be **in English** (professional use), **accessible** (a11y) and **lighter** — the previous dark theme was too dark for the user. Light as default, dark as an option.

**MVP scope**
- The whole UI in English + `<html lang="en">`.
- **Light theme by default + dark as an option**: a toggle in the header; follows `prefers-color-scheme` by default; the choice persisted in `state.theme`.
- **Update the design system in `CLAUDE.md` (section 2)** to a dual palette (light by default + dark) with tokens as CSS custom properties — so future apps inherit light.
- **Baseline a11y**: semantic landmarks (`header`/`main`/`nav`), action icons as `<button>` with `aria-label`, grid ARIA (`role=grid`/`row`/`columnheader`/`rowheader`/`gridcell`), every control reachable and operable from the keyboard, a **visible focus ring**, contrast of interactive elements ≥ WCAG AA.

**Later (deferred)**
- Full arrow-key navigation across grid cells.
- Announcing drag & drop operations for screen readers (live regions).

**Impact**
- **Data**: + `state.theme` (`'light' | 'dark' | 'auto'`, default `'auto'`). An additive field — **no key version bump** (`load()` via `Object.assign` will keep the default when the field is missing in old data). No blobs → localStorage.
- **Actions**: `toggleTheme()` / `setTheme(v)` → `save()` + apply `data-theme` on `<html>` (+ listen to `prefers-color-scheme` when `'auto'`).
- **Screens**: a theme-toggle button in the header (☀ / ☾).
- **States**: none new; edge: no saved theme → `'auto'` → resolve via media query; system theme changes live while in auto mode.
- **Interactions**: keyboard (toggle, focus mgmt); `prefers-color-scheme` media query.
- **Edge cases**: old state without `theme` → default; contrast in both themes ≥ AA.
- **Glossary**: `theme` (`light` / `dark` / `auto`).

**Build instructions for ux-build**
1. **`CLAUDE.md` section 2**: replace the single dark palette with a **dual system** — tokens as CSS custom properties; a light variant (default on `:root`) and a dark variant (via `[data-theme="dark"]`, plus `@media (prefers-color-scheme: dark)` when `data-theme="auto"`). Light palette: bg `#f7f8fa`, panels `#ffffff #f1f3f5`, borders `#e5e7eb #d0d7de`, text `#1f2328`, secondary `#57606a`, marginal `#8c959f`, primary accent `#2563eb` (hover `#1d4ed8`), danger `#d1242f`. Keep the existing dark values as the dark variant. Update the component descriptions (btn, sidebar, panel, drop zone, scrollbar) to use tokens.
2. **`index.html`**: introduce the same token system (CSS vars on `:root` + `[data-theme="dark"]`); `<html lang="en">`; a toggle button in the header that calls `toggleTheme()`; boot: apply `state.theme` as `data-theme` on `<html>` + listen to `prefers-color-scheme` for `'auto'`.
3. **English (all strings)**: header hint, sidebar section headers → *Categories / Personas / Customer journeys*, tooltips ✎/×/⠿ → *Rename / Remove / Drag*, journey meta → *Title / Persona / Scenario (description) / Expectations*, `+ Etap` → *+ Stage*, `+ krok` → *+ Step*, `+ element` → *+ element*, empty states, export → *Export PNG / ✓ Copied / ✗ Failed*, counter → *N steps*. Slice labels → **Actions, Touchpoints, Mindset, Emotions, Pain points, Ideas, Insights**. Seeded example: category *Cash loan*, persona *Anna*, CJ *Cash loan application*, stage *Stage 1*, step *Step 1*.
4. **a11y**: `<html lang="en">`; semantic `header`/`main`/`nav`; action icons (⠿ ✎ × + ☀/☾) as `<button aria-label="…">`; ARIA in the grid: `#grid` → `role="grid"`, rows → `role="row"`, step headers → `role="columnheader"`, slice labels → `role="rowheader"`, data cells → `role="gridcell"`; a visible `:focus-visible` ring on all interactive elements; keep Enter/Escape in inline editing.
5. **Glossary in SPEC.md**: add `theme` (`light`/`dark`/`auto`) if it's not there yet.

### Change 2 — bug: low text contrast, esp. dark mode (2026-07-27)
**Status**: diagnosed — route to ux-build
**Severity**: 🟡 medium — main content (`--text`, `--text-2`) readable; labels/lists/chips/empty states hard to read. An a11y blocker for WCAG AA.

**Reproduction**
1. Open the app, switch to dark (☀ → ☾).
2. Read: sidebar section headers (CATEGORIES, PERSONAS…), panel-header, field-labels in journey-meta, empty-state hints, slice labels, CJ list text, counters (chip).
**Expected**: all text ≥ 4.5:1 (WCAG AA). **Actual**: secondary/marginal text 2.4–4.1:1 — hard to read; light mode also 2.7–4.3:1 on labels.
**Reliability**: every time, in both themes; dark worse.
**Location**: token definitions — `index.html:18-20,29` (light `:root`) and `index.html:47-49,58` (dark `[data-theme="dark"]`).

**Root cause**
**Class**: visual / a11y
**Cause**: the gray tokens for secondary/marginal text (`--text-3`, `--text-4`) and `--chip-text` are too close to the luminance of their backgrounds — below WCAG AA 4.5:1. Worst is dark `--text-4 #555` = 2.4:1 on panels (section headers, field-labels, empty states nearly unreadable). `--text-3 #777` ≈ 4.0:1, `--chip-text #888` = 4.05:1. Light `--text-4 #8c959f` ≈ 2.7–3.0:1, `--text-3 #6e7781` ≈ 4.1:1. Main text and element text (`--text`, `--text-2`) pass.
**Evidence** (measured WCAG): dark `#555` on `#141414` = 2.47, on `#111` = 2.53; dark `#777` on `#141414` = 4.11; dark `#888` (chip) on `#2a2a2a` = 4.05; light `#8c959f` on `#fff` = 3.04, on `#f1f3f5` = 2.73; light `#6e7781` on `#f1f3f5` = 4.09. Spec intent: SPEC.md §Requirements→States ("contrast ≥ WCAG AA") + CLAUDE.md §2.1 ("Contrast ≥ WCAG AA in both themes").

**Fix plan**
- **Change**: bump tokens to AA values (verified ≥4.5:1 on real backgrounds):
  - Dark (`[data-theme="dark"]`, `index.html:47-49,58`): `--text-2: #b9bfc6` (from #999), `--text-3: #9aa0a6` (from #777), `--text-4: #8a9098` (from #555), `--chip-text: #a8aeb6` (from #888).
  - Light (`:root`, `index.html:18-20`): `--text-2: #4b5563` (from #57606a), `--text-3: #5c6670` (from #6e7781), `--text-4: #626a73` (from #8c959f); leave `--chip-text` at `#57606a` (5.4:1 OK).
  - `--text` unchanged (`#e0e0e0` / `#1f2328` — already AA).
- **Spec impact**: none — no behavior change; CLAUDE.md §2.1 already mandates AA, this brings the code into compliance.

**Regression scope**
- Only a token change; it touches every surface using these tokens (section headers, panel-header, field-labels, slice labels, list text, chips, empty states, hints) — all become more readable, no functional regression.
- Also update the token table in CLAUDE.md §2.1 (the light/dark columns for `--text-2/3/4`) so the design-system doc matches the code and future apps inherit AA tokens.

**Build instructions for ux-build**
- In `index.html`: swap 4 dark values (L47-49, L58) and 3 light values (L18-20) to the AA values above.
- In `CLAUDE.md` §2.1 table: update the `--text-2/3/4` rows (light + dark columns) to the new values.
- After the change: re-verify contrast ≥4.5:1 for real token×background pairs. (text-4 on panel-3/chip only affects decorative drag-handles with `aria-hidden` — exempt; for full margin use text-4 → #5c6670.)

### Change 3 — feature: export/import full DB & single journeys (2026-07-27)
**Status**: planned — route to ux-build

**User goal**
Data portability/backup: export and import the full database (categories/personas/CJs) and individual customer journeys — moving data between browsers/devices, archiving, sharing CJs.

**MVP scope**
- Export the full database → JSON (header).
- Export a single CJ (+ persona snapshot) → JSON (journey-meta toolbar).
- Import with **shape auto-detection**: full database → **replace-all (with confirmation)**; a single CJ → add to the current category + recreate the persona (by name, if missing).
- ID regeneration on import (no collisions); shape validation; graceful error; "✓ Imported / ✗ Invalid file" feedback.

**Later (deferred)**
- Selective merge import (choose what to import).
- Drag & drop a file onto the window; import multiple CJs at once.
- Other formats (CSV/Markdown).

**Impact**
- **Data**: no change to the entity shape; import operates on the existing `state` (categories/personas/journeys/stages/steps/cells/elements). Export = serialization of a subset of `state` (categories + personas + nextId) to JSON. **No key version bump** (clean JSON over/under the existing state). No blobs → localStorage.
- **Actions**: `exportDatabase()`, `exportJourney()`, `importFromFile(file)` (auto-detect → replace-all / add-journey + persona-recreate + id-regen).
- **Screens**: buttons in the header (Export JSON / Import) + "Export CJ" in the journey-meta toolbar (next to "+ Stage"); a hidden `<input type="file" accept="application/json,.json">`.
- **States**: error — corrupt/invalid JSON → "✗ Invalid file" toast; success → "✓ Imported"; full-DB replace → a confirmation gate (native `confirm()` as an exception for a destructive action, or an inline modal).
- **Interactions**: file-picker (click hidden input → change → parse); download via `<a download>` (pattern from `exportPng`). No drag & drop in the MVP.
- **Edge cases**: corrupt/bad JSON → graceful; ID collisions → regeneration via `nextId`; single-journey import without a persona in the pool → create a new one (by name); a persona with the same name exists → link it (don't duplicate); a full database with 0 CJs → an empty replace; import overwrites unsaved changes (hence the confirm on full DB).
- **Glossary**: `backup` (full-DB JSON), `journey-export` (single-CJ JSON).

**Build instructions for ux-build**
1. `exportDatabase()` → `data = { version:1, kind:'database', categories: state.categories, personas: state.personas }` → `JSON.stringify` → Blob → download `customer-journey-backup-YYYYMMDD.json` (download pattern from `exportPng`; `safeFilename`).
2. `exportJourney()` on `currentJourney()` → `data = { version:1, kind:'journey', journey: structuredClone(j), persona: j.personaId ? {name: personaName(j.personaId)} : null }` → download `<safeFilename(j.title)>.json`.
3. A hidden `<input type="file" id="import-file" accept="application/json,.json" style="display:none">`. `importFromFile(file)` → FileReader readAsText → `JSON.parse` (try/catch) → detection:
   - `data.kind==='database'` OR `data.categories` → full database → `confirm('Replace ALL data with this file?')` → if yes: `state.categories = regenIds(data.categories)`, `state.personas = data.personas || []`, bump `state.nextId` above the max ID in the data, `ensureActiveIndicesValid()`, `save()`, `render()`.
   - `data.kind==='journey'` OR `data.journey`/`data.stages` → add to `currentCategory()` (if no category → create one); persona: find by name in `state.personas`, if missing → `newPersona(data.persona.name)`; wire up `personaId`; regenerate IDs of journey/stages/steps/elements via `nextId`; `save()`, `render()`.
   - otherwise → "✗ Invalid file" toast.
   - `regenIds` = a helper that walks the structure and assigns new IDs from `uid()` (categories/journeys/stages/steps/elements/personas as needed).
4. UI: in the header add `.btn-ghost btn-sm` "↧ Export JSON" and "↥ Import" (next to Export PNG); in the journey-meta toolbar add `.btn-ghost btn-sm` "↧ Export CJ" next to "+ Stage". The Import button → `importInput.click()`.
5. Feedback: success toast "✓ Imported (N journeys)"/"✓ Journey imported" and error "✗ Invalid file" (pattern like `showStorageError`); no `alert()`.
6. Safety: full-database replace only after `confirm()` (native OK for a destructive gate); single-journey import without confirm (additive).

### Change 4 — bug: sidebar pluralization + spacing + journey cards (2026-07-27)
**Status**: diagnosed — route to ux-build
**Severity**: 🟢 low — cosmetic/presentation, no functional impact.

**Reproduction**
1. Sidebar: a category with exactly 1 CJ → the counter shows "1 journeys" (wrong singular).
2. Each list item (e.g. CJ): the name and the extra content (persona · description) touch (margin 1px) — no gap.
3. The CJ list is flat rows like categories/personas — it doesn't read as cards.
**Expected**: "1 journey" / "N journeys" (i18n plural), a clear gap between name ↔ sub, CJs as cards. **Actual**: "1 journeys", no gap, flat rows.
**Reliability**: every time.
**Location**: `index.html:710` (count sub), `index.html:131` (`.item-sub` margin), `index.html:778` + `index.html:744` (journey items via `mkSidebarItem` without a card variant).

**Root cause**
**Class**: visual / UX
**Cause**: (1) the counter on L710 has a hardcoded "journeys" without a singular form — a singular/plural pattern exists for steps (L967), but it's not applied here. (2) `.item-sub` has `margin-top: 1px` (L131) — too tight a gap between name and content. (3) journey items are rendered by the same generic `.sb-item` as categories/personas (`mkSidebarItem` L778) — no "card" variant, so the CJ list looks like ordinary rows instead of cards.
**Evidence**: L710 `sub: \`${cat.journeys.length} journeys\``; L131 `margin-top: 1px`; L778 `li.className = 'sb-item' ...` (no variant classes); L744 `renderJourneyList` → `mkSidebarItem` without the `card` flag. Spec intent: SPEC.md §Requirements→Screens (sidebar with a CJ list shows persona + description — legibility) + CLAUDE.md §2.3 (sidebar, components consistent).

**Fix plan**
- **Pluralization (L710)**: add a helper `function plural(n, one, many){ return n===1?one:many; }` and use it: `sub: \`${cat.journeys.length} ${plural(cat.journeys.length,'journey','journeys')}\``. (Optionally also replace the ternary in stepCount L967 with it for consistency.)
- **Spacing (L131)**: `.sb-item .item-sub { margin-top: 4px; }` (from 1px) — a clear gap between name and content.
- **Journey cards**: in `mkSidebarItem` add a `card` parameter → `li.className = 'sb-item' + (active?' active':'') + (card?' sb-card':'')`; in `renderJourneyList` pass `card: true`. CSS:
  - `.sb-item.sb-card { background: var(--cell); border: 1px solid var(--border-soft); border-radius: 8px; padding: 8px 10px; margin-bottom: 6px; }`
  - `.sb-item.sb-card:hover { background: var(--cell-hover); border-color: var(--border); }`
  - `.sb-item.sb-card.active { background: var(--accent-soft); border-color: var(--accent); }`
  - `.sb-item.sb-card .item-name { font-weight: 600; color: var(--text-2); }` (`.sb-item.sb-card.active .item-name` → `var(--text)`).
  - Category/persona stay as flat rows (without `sb-card`).
- **Spec impact**: none — sidebar presentation; SPEC §Screens describes the CJ list with persona + description, this improves legibility without changing data/actions.

**Regression scope**
- `.item-sub` margin affects all sidebar items (categories/personas/CJs) → all get more spacing (improvement, no regression).
- `.sb-card` only on journey items; categories/personas untouched.
- `plural()` is a new helper, used on L710; safe.
- `mkSidebarItem` +1 `card` parameter; only `renderJourneyList` passes `true`.
- Drag-reorder / drop indicators (`.sb-item`) still work — `sb-card` keeps the `sb-item` class.

### Change 5 — bug: sidebar name↔actions spacing + CJ card stacked info (2026-07-27)
**Status**: diagnosed — route to ux-build
**Severity**: 🟢 low — cosmetic/presentation sidebar.

**Reproduction**
1. Sidebar: a category / CJ name touches the action buttons on the right (✎ × ⠿) — no horizontal gap.
2. CJ card: persona and description are in a single, truncated line ("persona · description") instead of separate lines below the title.
**Expected**: a clear gap between name ↔ action buttons; CJ → title (1 line) + **persona** + **description** as separate lines below. **Actual**: name and buttons touching; persona+description in one truncated line.
**Reliability**: every time.
**Location**: `index.html:126` (`.sb-item { gap: 6px }`), `index.html:804-808` (`mkSidebarItem` renders `sub` as one `span`), `renderJourneyList` (joins `persona · description` into one string).

**Root cause**
**Class**: visual / UX
**Cause**: (1) `.sb-item` has `gap: 6px` (L126) — too small a gap between `item-main` (the name) and the action-button group on the right. (2) `mkSidebarItem` always renders `sub` as a single `span` (L804-808), and `renderJourneyList` joins persona and description into one "·" string → one truncated line instead of a stack under the title.
**Evidence**: L126 `gap: 6px`; L804 `if (sub) { const subSpan = ...; subSpan.textContent = sub; }`; renderJourneyList `sub: \`${personaName(...)}${j.description ? ' · ' + j.description : ''}\``. Spec intent: SPEC.md §Requirements→Screens (the CJ list shows persona + description — legibility) + CLAUDE.md §2.3.

**Fix plan**
- **Spacing (L126)**: `.sb-item { gap: 10px }` (from 6px) — a clear gap between name ↔ action buttons (affects all sidebar items).
- **Stacked info**: in `mkSidebarItem` handle `sub` as a **string or array** — if an array, render each item as a separate `.item-sub` (stack). Implementation: `const subs = Array.isArray(sub) ? sub : (sub ? [sub] : []); subs.forEach(s => { const subSpan=...; subSpan.textContent=s; main.appendChild(subSpan); })`.
- **renderJourneyList**: pass `sub: [personaName(j.personaId), j.description].filter(Boolean)` (array) → persona and description as separate lines under the title. Category/persona keep string-sub (unchanged).
- **Spec impact**: none — sidebar presentation.

**Regression scope**
- `gap` on `.sb-item` affects all items → more spacing (improvement, no regression).
- `sub` as array/string: only `renderJourneyList` passes an array; `renderCategoryList`/`renderPersonaList` pass a string (preserved). `mkSidebarItem` handles both.
- An empty `sub` array (`[]`) → no sub line (correct).

### Change 6 — bug: sidebar lines never stacked — item-main not a flex column (2026-07-27)
**Status**: diagnosed — route to ux-build
**Severity**: 🟡 medium — affects the legibility of the whole sidebar; the reason Change 4 and 5 "didn't work" visually.

**Reproduction**
1. Sidebar: between a category name and the CJ count (item-sub) there is **no margin** — the texts touch.
2. CJ card: title, persona and description also have no gap between lines.
**Expected**: name / count (and title / persona / description) stacked vertically with a clear gap. **Actual**: lines without separation.
**Reliability**: every time.
**Location**: `index.html:129` (`.item-main { flex:1; min-width:0; }`), `index.html:131` (`.item-sub { ... margin-top: 4px }`).

**Root cause**
**Class**: visual / CSS
**Cause**: `.item-main` (L129) **is not a flex-column container** — its children (`<span class="item-name">`, `<span class="item-sub">`) are **inline** elements, so they don't lay out on separate rows, and `margin-top` on inline elements is **ignored**. That's why `margin-top:4px` (Change 4) and array-sub (Change 5) had no visible effect — the separation never physically worked. (Side effect: `text-overflow:ellipsis` on `item-name` also doesn't work, because it's an inline span — long names aren't truncated.)
**Evidence**: L129 no `display:flex; flex-direction:column`; L131 `margin-top: 4px` (ignored on inline); L130 `item-name` inline span. Spec intent: SPEC.md §Requirements→Screens (CJ list with persona + description — legibility).

**Fix plan**
- **L129**: `.sb-item .item-main { flex: 1; min-width: 0; display: flex; flex-direction: column; gap: 6px; }` — children become flex-items (blocked), they stack vertically with a 6px gap.
- **L131**: `.item-sub` — remove `margin-top: 4px` (→ `margin-top: 0`); the gap is now handled by `gap` on `.item-main`.
- **Bonus**: as a flex-item, `item-name` becomes "blocked" → `text-overflow:ellipsis` starts working (long names truncated).
- **Spec impact**: none — sidebar presentation.

**Regression scope**
- The change affects all sidebar items (categories/personas/CJs) → all get correct stacking + a 6px gap between lines (improvement, no regression).
- `item-name` starts truncating long names (desirable).
- Categories/personas still use string-sub (one sub line); CJs use array-sub (multiple lines) — both render correctly in a flex-column.

### Change 7 — feature: step screenshots (IndexedDB) + persona library with JSON import (2026-08-18)
**Status**: planned — route to ux-build

**User goal**
1. Attach UI screenshots to every CJ step (the screen state at that step) — visible directly in the map and in the PNG export.
2. Keep personas in the private `marekbrze/business` repo (source of truth) and import them into the tool as JSON — so persona data NEVER lands in the ux-tools repo.

**MVP scope**
- **A. Screenshots**: 8th grid row "Screenshots" (below Insights) — thumbnails per step; adding via file picker (+ add), drag & drop of a file onto the cell, Ctrl+V (clicking a cell sets the paste target); clicking a thumbnail → lightbox; hover × → delete. Blobs → IndexedDB; metadata (`id`, `w`, `h`) in localStorage. PNG export draws a thumbnails band. JSON export/import carries the images (dataURL in the file).
- **B. Persona library**: `persona` + fields `role`, `description`, `goals[]`, `needs[]`, `frustrations[]`, `quote`. Import of a `kind: 'personas'` file — **merge by name** (existing → overwrites fields present in the file, new → added, local ones not in the file stay; CJs keep their link). Persona card (read-only modal) on clicking a persona in the sidebar + ⓘ button next to the persona select in journey-meta. Small ↥ in the Personas section header (same global import).

**Later (deferred)**
- Screenshot captions + drag-reorder of thumbnails in a cell.
- Downscale/compress images on add (IndexedDB quota control).
- Local editing of persona fields in the modal (today read-only — source of truth is the business repo).
- Persona avatar/photo in the format and card.
- Importing multiple persona files at once / selective merge.

**Impact**
- **Data**:
  - New `screenshots` slice in `SLICES` → `step.cells.screenshots = [{ id, w, h }]` (empty list by default). Metadata in localStorage.
  - Image blobs: **new IndexedDB** `ux-customer-journey-images` (store `images`, keyPath `id`, record `{ id, blob, type, name, w, h }`). Never a dataURL in localStorage.
  - `persona`: `{ id, name, role, description, goals[], needs[], frustrations[], quote }` — fields optional, normalized on `load()` and import.
  - **No key version bump** (`ux-customer-journey_v1` stays): `load()` already normalizes missing slice keys (loop over `SLICES`), old data gets an empty `screenshots` row; personas get empty fields via `normalizePersona`.
- **Actions**: `addScreenshotsFromFiles(stepId, files)`, `removeScreenshot(stepId, shotId)` (+ blob delete), `openLightbox(stepId, shotId)`, `openPersonaCard(personaId)`, `importPersonas(data)` (merge by name, case-insensitive after trim). Changed: `exportPng()` (async, thumbnails band), `exportDatabase()`/`exportJourney()` (async, + `images[]` as dataURL), `importDatabase()`/`importJourney()` (import `images[]` → IDB + ID remap).
- **Screens**: 8th "Screenshots" row in the grid; lightbox (overlay); persona card (modal from the sidebar); ↥ in the Personas header; ⓘ next to the persona select in journey-meta.
- **States**: empty Screenshots row (per step — "+ add" placeholder); missing/unavailable IndexedDB → try/catch + toast, the tool works without images; IDB quota → toast "✗ Couldn't store image"; missing blob on PNG/lightbox (IDB wiped) → dashed placeholder; imported persona without `name` → skipped; a file with no personas at all → "✗ Invalid file".
- **Interactions**: drag & drop of an image/* file onto a row cell (drag-over accent); Ctrl+V paste of a clipboard image into the last-clicked Screenshots cell (no active cell → toast hint); thumbnail click → lightbox; Escape/backdrop/× closes the lightbox and the persona card; focus in modals (focus close button, restore on close).
- **Edge cases**: deleting a step/stage/CJ/category + replace-all on import → cascading removal of blobs from IDB (helper collecting `id`s from `cells.screenshots`); large canvas with many screenshots — fixed-height thumbnails bound the band height; an old JSON backup without `images` → imports without images (normalize); persona name duplicates differing in case → case-insensitive merge (no dupes); `goals/needs/frustrations` as a string in the file → normalized to `[string]`.
- **Glossary**: `screenshot`, `image store`, `lightbox`, `persona card`, `personas import`.

**`kind: 'personas'` contract format** (to be implemented by the generator in `marekbrze/business`):
```json
{
  "version": 1,
  "kind": "personas",
  "app": "customer-journey",
  "personas": [
    {
      "name": "Anna Kowalska",
      "role": "Senior accountant",
      "description": "…",
      "goals": ["…"],
      "needs": ["…"],
      "frustrations": ["…"],
      "quote": "…"
    }
  ]
}
```
Import tolerance: only `name` required; lists as `string[]` OR string (→ single-element list); fields optional, unknown fields ignored; missing fields = no change on merge.

**Build instructions for ux-build**
1. **Slice**: append `{ key: 'screenshots', label: 'Screenshots' }` to `SLICES` + `const SCREENSHOT_KEY = 'screenshots';`. `freshCells()`/`normalizeStep()` produce an empty list automatically (key ≠ `EMOTION_KEY` → `[]`).
2. **IndexedDB** (new section at Persistence): `openImagesDb()` (lazy singleton, DB `ux-customer-journey-images`, v1, store `images` keyPath `id`); `dbPutImage(rec)`, `dbGetImage(id)`, `dbDeleteImage(id)` — all Promise + try/catch. Helpers: `blobToDataURL(blob)`, `dataURLToBlob(dataURL)`.
3. **Add**: `addScreenshotsFromFiles(stepId, files)` — filter `type.startsWith('image/')`; dimensions via `createImageBitmap` (fallback `Image`); `uid()` → `dbPutImage({id, blob, type, name, w, h})`; push `{id, w, h}` to `cells.screenshots`; `save() + renderGrid()`. IDB errors → toast, don't break rendering.
4. **Grid**: in `renderGrid()` branch at cell construction (today: `if (slice.key === EMOTION_KEY)`): `slice.key === SCREENSHOT_KEY` → `buildScreenshots(step)` instead of `buildCellElements`. `buildScreenshots`: flex-wrap container; thumbnail = `<button class="shot-thumb">` with `<img>` (height 56px, width auto, max-width 100%) — `src` filled asynchronously (`dbGetImage` → `URL.createObjectURL`; Map cache id→objectURL, revoke on delete); hover × = `removeScreenshot` (removes the entry + `dbDeleteImage` + revoke); "+ add" = dashed button → hidden `<input type="file" accept="image/*" multiple>`. Cell: `tabindex="0"`, click sets `_activeShotsCell = {stepId}`; `dragover/drop` on files → `addScreenshotsFromFiles` + drag-over class. Paste: `document` `paste` listener — if `clipboardData.items` has an image and `_activeShotsCell` exists → add; otherwise toast hint "Click a Screenshots cell first".
5. **Lightbox**: `openLightbox(stepId, shotId)` — overlay (fixed, dim backdrop) with the image `max-width:90vw; max-height:85vh; object-fit:contain`; close: ×, Escape, backdrop click; `role="dialog" aria-modal="true" aria-label="Screenshot"`; focus × on open, restore on close. Missing blob → placeholder.
6. **Cascade**: helper `collectShotIds(journey)`; called in `removeStep` (single step), `removeStage`/`removeJourney`/`removeCategory` (recursively) and `importDatabase` (everything replaced) → `dbDeleteImage` per id (fire-and-forget, catch).
7. **PNG**: `exportPng()` → async: before drawing `Promise.all` over `flat` — `dbGetImage` + `createImageBitmap` for screenshots; the Screenshots band appended to `sliceRows` (thumbnail h=72px, width from aspect `w*(72/h)` clamped to `colW-16`); draw with `drawImage` preserving aspect; missing image → dashed rect. Band height computed from metadata (`w`/`h`), images loaded before canvas sizing.
8. **JSON export**: `exportDatabase()`/`exportJourney()` → async; collect all screenshots from the exported subset → `images: [{ id, name, type, w, h, data }]` (`data` = dataURL) at the payload top level; steps still keep `{id, w, h}` in `cells.screenshots`.
9. **JSON import**: in `importDatabase()`/`importJourney()`: if `data.images` — for each record `dataURLToBlob` → new `uid()` → `dbPutImage` → remap old→new `id` in all `cells.screenshots` (oldId→newId map; entries without an image in `images` dropped). No `data.images` → import without images (old files).
10. **Personas — model**: `newPersona(name)` → `{ id, name, role:'', description:'', goals:[], needs:[], frustrations:[], quote:'' }`; `normalizePersona(p)` in `load()` (next to the slice-normalizing loop) and on import. `personaName()` unchanged.
11. **Personas import**: `importPersonas(data)` — validate `Array.isArray(data.personas)`; per persona: `name` required (trim); find in `state.personas` by name (trim, case-insensitive); found → overwrite ONLY fields present in the file (lists normalized string|string[] → string[]); new → `newPersona` + fill; toast `✓ Personas imported (N added, M updated)`. Detection in `importFromFile()` — order: `database` → `personas` (`kind==='personas' && Array.isArray(data.personas)`) → `journey` → invalid. No confirm (non-destructive merge).
12. **Persona card**: `openPersonaCard(personaId)` — modal: name (h2), `role` (subtitle/chip), `description` (paragraph), `quote` (blockquote), `goals`/`needs`/`frustrations` as label + list; empty sections hidden; × / Escape / backdrop closes; `role="dialog" aria-modal`; focus like the lightbox. `renderPersonaList()`: `onActivate: () => openPersonaCard(p.id)` (click = view; rename stays ✎/dblclick). Personas section header: add ↥ → `importInput.click()`.
13. **Journey-meta**: next to the persona select an ⓘ button (`btn-ghost btn-sm`, disabled when `personaId == null`) → `openPersonaCard(j.personaId)`. `exportJourney()`: persona snapshot = full object (not just `name`); `importJourney()` — match by name unchanged, use full fields for a new persona.
14. **CSS** (tokens only!): `.shots` (flex-wrap, gap 4px), `.shot-thumb` (border, radius 4, hover border-accent), `.shot-remove` (hover-reveal), `.shot-add` (dashed), `.g-cell.shots-cell.drag-over` (accent), lightbox + persona-card (overlay, panel, shadow), `.pc-list`/`.pc-label`. A11y: thumbnails as buttons with `aria-label`; dialogs with `aria-modal`; focus-visible everywhere.
15. **Regression**: all existing flows (grid, element drag & drop, emotion, PNG, database/journey export/import, theme, sidebar) work unchanged; `SLICES` +1 row → PNG/ARIA/normalization consistent.

### Change 8 — feature: stage removal + brand themes via JSON (2026-08-18)
**Status**: planned — route to ux-build

**User goal**
1. Stages can only be added today — there is no way to delete one from the UI, which makes restructuring a CJ painful.
2. Export CJs in a client's brand colors (first up: Bank Millennium), with the theme definitions living as JSON in the private `marekbrze/business` repo — the same source-of-truth + JSON-import pattern as personas.

**MVP scope**
- **A. Stage delete**: hover-reveal × on the stage header (`​.g-stage`) → calls the existing (currently unreachable) `removeStage()`. Immediate, no confirm — consistent with step/category deletes. Existing rules kept: screenshot blobs cascade, a journey never drops below one (fresh "Stage 1").
- **B. Brand themes**: `brandTheme` entity (`name` + role-based color keys) in `state.brandThemes` (localStorage — tiny, no blobs); **per-journey** `journey.brandThemeId` (null = app theme; the persona pattern: library in the sidebar + select in journey-meta). A **Themes** sidebar section with ↥ import / ↧ export (`kind: 'themes'` JSON, merge by name — symmetric with personas). An applied theme restyles the **grid preview** (inline CSS custom properties scoped to the grid area) **and** the **PNG export**. Seeds a built-in **"Bank Millennium"** theme (researched starter values, correctable by editing the repo JSON and re-importing).

**Later (deferred)**
- In-app theme editing (color pickers), duplicate/rename a theme.
- Themeable emotion scale (`colors.emotions`, 5 colors replacing `EMO_COLORS`).
- Contrast warning when a theme combination is below WCAG AA.
- Theme extras beyond colors (logo, fonts are out of scope by design — offline/system fonts).

**Impact**
- **Data**: + `state.brandThemes: [{ id, name, colors: {role: hex} }]`; + `journey.brandThemeId: number | null`. localStorage only. **No key version bump** — `load()` normalizes (`Array.isArray(state.brandThemes)`; `brandThemeId` optional → null). `exportDatabase()`/`importDatabase()` carry `brandThemes` (ID regeneration + `themeMap` remap of `journey.brandThemeId`, mirroring the persona map); `exportJourney()` carries `theme: {name}` snapshot, `importJourney()` matches by name.
- **Actions**: `importBrandThemes(data)` (merge by name, case-insensitive, per-key colors merge), `exportBrandThemes()` (download → commit to the business repo), `setJourneyBrandTheme(id|null)`, `removeBrandTheme(i)` (nulls out `brandThemeId` on journeys using it), `applyBrandThemeToGrid()`; UI wiring for the existing `removeStage(stageId)`.
- **Screens**: × on the stage header (hover-reveal, pattern of `.step-remove`); **Themes** sidebar section between Personas and Customer journeys (compact — short list; ↥ ↧ in the section header, hover × per item, click = apply to the current journey, click again = back to app theme); **Theme** select in journey-meta row 1 next to Persona (`— app theme —` + theme names).
- **States**: empty Themes list ("No themes — import (↥) or export the starter set (↧)"); journey referencing a deleted theme → renders with the app theme; invalid/partial JSON: themes without `name` skipped, invalid hex values dropped per key, unknown keys ignored, missing keys fall back to app tokens; toasts `✓ Themes imported (N added, M updated)` / `✗ Invalid file`; theme with no journey open → toast "Open a journey first".
- **Interactions**: import via the hidden file input — `importFromFile()` detection order becomes `database → personas → themes → journey → invalid`; export via `<a download>`; the themed grid preview updates live on select change (theme overrides light/dark for the grid area only — brand colors are absolute by design).
- **Edge cases**: deleting the last stage → journey resets to a fresh "Stage 1" (existing `removeStage` rule); stage delete cascades its steps' screenshot blobs (existing); re-importing a theme with the same name overwrites its colors per key (how the Millennium starter gets corrected); a database backup without `brandThemes` → keeps current themes (and the boot seed re-adds Millennium if missing).
- **Glossary**: `brand theme` (`brand-theme` — distinct from `theme` = UI light/dark/auto), `theme roles` (`theme-roles`), `themes import` (`themes-import`), `themes export` (`themes-export`).

**Theme color contract — role keys** (each optional; every role falls back to the app token in the active UI theme, so a partial theme is legal):

| Role | Grid element | Export element | Fallback token |
|------|--------------|----------------|----------------|
| `bg` | `#grid-scroll` background | canvas background | `--bg` |
| `stageBg` | `.g-stage` band | stage band fill | `--panel-3` |
| `stageText` | `.g-stage .stage-name` | stage band text | `--text-2` (via `--cj-stage-text`) |
| `stepBg` | `.g-step` header | step header fill | `--panel` |
| `labelBg` | `.g-slice-label` + `.g-corner` | slice-label column + "Step" corner | `--panel-2` |
| `labelText` | `.g-slice-label` text | slice labels + "—" placeholders | `--text-3` |
| `cellBg` | `.g-cell` | data cell fill | `--cell` |
| `text` | `.g-step .step-name`, `.el-text` | step names, cell text, title | `--text-2` |
| `border` | grid borders | strong borders | `--border` |
| `borderSoft` | cell separators | soft borders | `--border-soft` |
| `accent` | grid hover/focus accents | journey title text | `--accent` |

**`kind: 'themes'` contract format** (authored/stored in `marekbrze/business`, e.g. `customer-journey/themes.json`):
```json
{
  "version": 1,
  "kind": "themes",
  "app": "customer-journey",
  "themes": [
    {
      "name": "Bank Millennium",
      "colors": {
        "bg": "#ffffff",
        "stageBg": "#c60052",
        "stageText": "#ffffff",
        "stepBg": "#ffffff",
        "labelBg": "#fbe8f0",
        "labelText": "#a33164",
        "cellBg": "#ffffff",
        "text": "#24292e",
        "border": "#e8cbd8",
        "borderSoft": "#f4e6ed",
        "accent": "#c60052"
      }
    }
  ]
}
```
Import tolerance: `name` required (trim); `colors` optional object; per-key value must match `/^#([0-9a-f]{3}|[0-9a-f]{6}|[0-9a-f]{8})$/i` — invalid values dropped per key, unknown keys ignored; merging by name overwrites only the keys present in the file (missing keys keep current values). The Millennium hexes are researched public brand values (`#c60052` = official logotype magenta) — starters, not gospel; correct them in the repo file.

**Build instructions for ux-build**
1. **Stage delete UI** — in `renderGrid()`'s stage cell construction (index.html:1451-1490), after `addStepBtn`: a `<button class="stage-remove">×</button>` styled exactly like `.g-step .step-remove` (opacity 0 → 1 on `.g-stage:hover`, `--danger` on hover), `aria-label="Remove stage {name}"`, click → `e.stopPropagation(); removeStage(st.id);`. No changes to `removeStage()` itself — it already cascades blobs and keeps ≥1 stage.
2. **State** — `state.brandThemes: []` in the state literal; `journey.brandThemeId: null` in `newJourney()`; set in `normalizeJourney()` (`if (j.brandThemeId == null) j.brandThemeId = null;` — also on old data in `load()`'s normalization walk). Constants: `const THEME_ROLES = ['bg','stageBg','stageText','stepBg','labelBg','labelText','cellBg','text','border','borderSoft','accent'];` and `ROLE_FALLBACK_TOKEN = { bg:'--bg', stageBg:'--panel-3', stepBg:'--panel', labelBg:'--panel-2', cellBg:'--cell', labelText:'--text-3', text:'--text-2', border:'--border', borderSoft:'--border-soft', accent:'--accent' };` (`stageText` has no direct token — see step 4).
3. **normalize/import** — `normalizeBrandTheme(t)`: name string + trim required (else drop), `colors` filtered to `THEME_ROLES` keys with valid hex. `importBrandThemes(data)` mirrors `importPersonas()`: merge by name (trim, case-insensitive), new → `{ id: uid(), name, colors }`, existing → per-key color overwrite; returns `{added, updated}`; toast `✓ Themes imported (N added, M updated)`. In `importFromFile()` insert the branch after `personas`: `data.kind === 'themes' && Array.isArray(data.themes)` → `importBrandThemes(data)`.
4. **Grid preview** — `applyBrandThemeToGrid()`: called from `renderGrid()` (top). Finds `currentJourney().brandThemeId` → theme (defensive lookup; missing → treat as null). On `#grid-scroll` (or `#grid`) set inline custom properties: for each present role set the mapped token name (`gridScroll.style.setProperty('--panel-3', t.colors.stageBg)` etc.), `stageText` → `'--cj-stage-text'`; when no theme active → `removeProperty` for all (restore app tokens). CSS: add `.g-stage .stage-name { color: var(--cj-stage-text, var(--text-2)); }` — everything else in the grid already consumes the tokens being overridden.
5. **PNG export** — in `exportPng()`: build `C` from the active brand theme with `cssVar(ROLE_FALLBACK_TOKEN[role])` fallbacks; stage band drawn with `C.stageBg` + `C.stageText`; journey title in `C.accent`; slice-label band/corner with `labelBg`/`labelText`; placeholders "—" use `labelText`.
6. **Sidebar Themes section** — HTML between the Personas and Customer journeys blocks: section header "Themes" with ↥ (`#btn-import-themes`, opens the same hidden import input) and ↧ (`#btn-export-themes` → `exportBrandThemes()`); `<ul id="theme-list" class="sb-list">` (compact `max-height`, e.g. 18% — the list is short; shave the persona list a few %). `renderThemeList()` via `mkSidebarItem({ name, active: j?.brandThemeId === t.id, onActivate: () => setJourneyBrandTheme(active ? null : t.id), onRemove: () => removeBrandTheme(i) })` — **no drag handle, no rename** (extend `mkSidebarItem`: skip the ⠿ handle + drag wiring when `!draggable`, skip the ✎ button when `onRename` is not passed; existing callers unchanged). Guard `onActivate` with `currentJourney()` → else `flashToast('Open a journey first')`. `renderThemeList` joins `render()`.
7. **Actions** — `setJourneyBrandTheme(id)`: `j.brandThemeId = id ?? null; save(); renderJourneyMeta(); renderThemeList(); renderGrid();` · `removeBrandTheme(i)`: splice, then `state.categories.forEach(cat => cat.journeys.forEach(j => { if (j.brandThemeId === removedId) j.brandThemeId = null; }))`, `save(); render();` · `exportBrandThemes()`: `{ version:1, kind:'themes', app:'customer-journey', themes: state.brandThemes.map(t => ({ name: t.name, colors: t.colors })) }` → `downloadJson(..., 'customer-journey-themes.json')` (stable name, no date stamp — a repo file) + toast.
8. **Journey-meta** — in row 1 next to the persona select: a Theme select (label "Theme", `— app theme —` value `''` + one option per theme), value = `j.brandThemeId ?? ''`, `change` → `setJourneyBrandTheme(value ? Number(value) : null)`.
9. **JSON export/import** — `exportDatabase()`: add `brandThemes: state.categories…` → `brandThemes: JSON.parse(JSON.stringify(state.brandThemes))`; `importDatabase()`: `const themes = Array.isArray(data.brandThemes) ? data.brandThemes : state.brandThemes;` + regen ids into `themeMap` + remap `j.brandThemeId` (personaMap pattern). `exportJourney()`: + `theme: theme ? { name: theme.name } : null`; `importJourney()`: match by name (case-insensitive) → `j.brandThemeId` else null.
10. **Boot seed** — in the boot IIFE after `load()`: if no theme named "Bank Millennium" (case-insensitive) → push the built-in object from the contract example above. Re-checked every boot, so it survives imports/wipes; overwriting it via repo import works by name.
11. **CSS** — `.g-stage .stage-remove` (hover-reveal ×); the `--cj-stage-text` rule; `#theme-list` sizing. Tokens only, both app themes AA as today (brand themes are the author's contrast responsibility — noted as Later).
12. **Regression** — app `theme` (light/dark/auto) untouched and still wins when a journey has no brand theme; all existing flows (grid, drag & drop, emotion, screenshots, PNG, JSON export/import, sidebar) unchanged; `mkSidebarItem` extension is additive.

### Change 9 — bug: forced Millennium seed + strict themes import blocks manual JSON workflow (2026-08-18)
**Status**: diagnosed — route to ux-build
**Severity**: 🟡 medium — no data loss, but the themes feature's core workflow (clean JSON in the business repo) is impossible: every export is pre-polluted, local deletion doesn't stick, and hand-written JSON is rejected on format.

**Reproduction**
1. Open the tool → the Themes panel already lists "Bank Millennium" (user never authored it).
2. Hover × on it to delete → gone. Refresh the page → it's back.
3. Click ↧ (export themes) → the downloaded `customer-journey-themes.json` always contains Bank Millennium → committing it to `marekbrze/business` puts the theme in the repo, which the user explicitly does not want yet.
4. Author a themes JSON by hand without the exact `{"kind": "themes", ...}` wrapper (e.g. a bare `[{"name": "...", "colors": {...}}]` or `{"themes": [...]}`) → ↥ import → "✗ Invalid file".
**Expected**: themes start empty; the user adds them manually via JSON they author; the repo file starts clean; deleting a theme sticks. **Actual**: a built-in theme is force-injected, resurrects on every boot, rides along in every export, and hand-written JSON variants are rejected.
**Reliability**: every time, all browsers (the seed runs on every boot).
**Location**: `index.html:426` (`BUILT_IN_THEMES` constant), `index.html:2317-2324` (boot seed re-adding by name), `index.html:2236` (import detection requires `kind === 'themes'`), `index.html:601` (`exportBrandThemes` exports all of state — all-or-nothing).

**Root cause**
**Class**: spec-vs-code drift (a Change 8 design that contradicts the workflow Change 8 itself established)
**Cause**: Change 8's build instruction #10 planned a permanent boot seed — `BUILT_IN_THEMES.forEach(...)` re-adds "Bank Millennium" on **every** boot whenever a theme with that name is absent (`index.html:2317-2324`). Since `exportBrandThemes()` (`index.html:601`) exports the entire `state.brandThemes` list with no selection, the seeded theme is unavoidable in the repo file, and deleting it locally is impossible (resurrect on refresh). This contradicts the source-of-truth pattern the same Change established for personas: the business repo authors content, the tool imports it — the tool must not inject its own entries into the library or its exports. Compounding it, the import branch (`index.html:2236`) demands the exact `kind: 'themes'` wrapper (stricter than the database branch, which also accepts `Array.isArray(data.categories)`), so a hand-authored file in any friendlier shape is rejected — which is why the user concluded manual adding "is not implemented".
**Evidence**: `index.html:2317-2324` seed loop; `index.html:601-606` export-all; `index.html:2236` strict detection (compare `index.html:2232`: `data.kind === 'database' || Array.isArray(data.categories)`). Spec (intent): SPEC.md Change 8 — "Seeds a built-in Bank Millennium theme" (now superseded by this Change; the user confirmed: remove the built-in completely, the palette stays as a documented reference snippet to be authored into the repo JSON when ready).

**Fix plan**
- **Change**:
  1. Delete `BUILT_IN_THEMES` (`index.html:426-433`) and the boot seed block (`index.html:2317-2324`). Themes start empty; no theme is ever auto-injected. The Millennium palette remains documented in Change 8's contract example — the canonical snippet the user copies into the repo file when they want it.
  2. Loosen the themes detection at `index.html:2236` to mirror the database branch tolerance: `data.kind === 'themes' || Array.isArray(data.themes) || (Array.isArray(data) && data.length > 0 && data.every(t => t && typeof t === 'object' && typeof t.name === 'string'))` — accepts the full contract, `{themes: [...]}` without `kind`, and a bare array of theme objects. Position in the chain unchanged (after personas, before journey; a bare array matches no other branch's shape).
  3. Empty-state copy at `index.html:1312`: "No themes yet — ↥ import a themes JSON file (a bare list of {name, colors} works too)."
  4. Migration: do **not** force-delete a "Bank Millennium" already present in existing localStorage state — the user removes it once via hover × (after this fix the deletion sticks). One-time manual step, noted here.
- **Spec impact**: Change 8's Impact bullet ("Seeds a built-in…") and build instruction #10 are superseded by this Change — ux-build follows Change 9.

**Regression scope**
- Boot IIFE (`index.html:2315+`): only the seed block references `BUILT_IN_THEMES` (grep: lines 426, 2317-2324) — remove both, nothing else to update.
- `exportBrandThemes()` (`index.html:601`): unchanged — with the seed gone, export = exactly what the user authored; no filtering needed.
- Import chain (`index.html:2232-2243`): the widened themes branch must not swallow other kinds — database requires `.categories`, personas requires `.kind === 'personas'`, journey requires `.journey`/`.stages`; a bare array satisfies none of them. Edge: a bare `[]` is rejected by the `data.length > 0` guard → "✗ Invalid file" (correct — an empty file adds nothing and shouldn't look successful).
- `importBrandThemes()` / merge-by-name semantics: untouched; re-importing a repo file that later contains a Millennium entry works by name — that is the intended future flow.
- Journey-meta Theme select, database export/import of `brandThemes`, PNG/grid theming: untouched.
- Related edge cases: none new — the leftover seeded theme in existing state (item 4 above) is the only migration concern.

### Change 10 — feature: touchpoint library per category (2026-08-18)
**Status**: built — implemented in commit 5b4c359 (checkpointed ux-build output; not separately click-tested)

**User goal**
Manage touchpoints in **one place per category** (a predefined list with add / rename / delete / reorder + JSON export/import), and in the grid pick touchpoints **from that list** instead of typing free text — with the ability to add a new item to the list right from the cell. Renaming a touchpoint updates every journey that uses it.

**MVP scope**
- **Touchpoint library per category**: `category.touchpoints` — ordered list of `{id, name}`. Managed in a **manager modal** (full CRUD + drag-reorder + usage count), opened from a ⚙ button on the *Touchpoints* slice label in the grid (and from the picker's empty state / "Manage…" row).
- **Picker in cells**: the `+` button in a Touchpoints cell opens a dropdown of the category's list — click to add (stays open, multi-add), click a present item to remove it (toggle), `＋ New touchpoint…` creates a library item AND adds it to the cell. No free typing in cells.
- **Rename propagation**: elements reference library items by `tpId`; renaming in the manager updates every cell immediately.
- **Delete in use**: confirm showing the usage count ("Used in N cells") → **cascade-remove** the entry from all cells in the category. Unused items delete without confirm.
- **Export/import** (active category): ↧ `kind: 'touchpoints'` JSON / ↥ import with merge-by-name (tolerant: strings or `{name}` objects, bare array of strings OK) — the business-repo pattern from personas/themes.
- **Migration**: existing free-text touchpoint elements become library items automatically (per category, find-or-create by name) — old data is the seed library.

**Later (deferred)**
- Touchpoint typing (entrypoint / push / channel tags — glossary already hints `entry-point`/`push`), descriptions, per-touchpoint color or icon.
- Search/filter in the manager; "jump to cells" from a usage count.
- Drag a touchpoint from the picker onto multiple cells at once.
- All-categories file format (`{name, touchpoints: [...]}` per category).

**Impact**
- **Data**: + `category.touchpoints: [{id, name}]` (localStorage — tiny, no blobs); touchpoint cell elements change shape `{id, text}` → `{id, tpId, text}` (`text` = name snapshot — display fallback for PNG/export, refreshed on rename). Other slices unchanged. **No key version bump** — additive normalization in `load()` (and the same helper on database import) migrates old data idempotently.
- **Actions**: + `addTouchpoint(name)`, `renameTouchpoint(tpId, name)` (propagates + refreshes snapshots), `removeTouchpoint(tpId)` (confirm + cascade), `moveTouchpoint(from, to)`, `openTouchpointManager()`, `openTouchpointPicker(anchor, step)`, `addTouchpointElement(stepId, tpId)`, `exportTouchpoints()`, `importTouchpoints(data)`; detection branch in `importFromFile()`. `addElement` stays for other slices — touchpoint cells stop using inline text edit.
- **Screens**: manager modal (list + add-field + ↧/↥ + usage chips); picker dropdown in cells; ⚙ on the Touchpoints slice label. No sidebar section (modal, per the interview).
- **States**: empty library (picker: "No touchpoints yet" + `＋ New…` + "Manage…"; manager: empty-state hint); import with no active category → toast "Add a category first"; rename to empty → cancel (inline-rename rules); toasts `✓ Touchpoints exported` / `✓ Touchpoints imported (N added, M exist)`.
- **Interactions**: picker (multi-add, toggle-remove, Esc/click-outside close, Enter commits the new-item field); drag-reorder in the manager (existing sidebar pattern); export via `<a download>`; import via the existing hidden file input; keyboard + focus management as in the persona-card/lightbox modals.
- **Edge cases**: duplicate names — find-or-create is case-insensitive after trim (no dupes, on migration and import alike); deleting an in-use item — confirm + cascade; rename — propagates to cells and snapshots; journey export/import — name-based re-resolution into the target category's library (old exports without a snapshot still resolve via element `text`); database import — per-category `tpId` remap after id regen; picker open during a category/journey switch or re-render → closes.
- **Glossary**: `touchpoint library` (`touchpoint-library`), `touchpoints file` (`touchpoints-file`), `picker` (`touchpoint-picker`), `manager` (`touchpoint-manager`).

**`kind: 'touchpoints'` contract format** (authored/stored in `marekbrze/business`, e.g. `customer-journey/touchpoints-<process>.json`):
```json
{
  "version": 1,
  "kind": "touchpoints",
  "app": "customer-journey",
  "category": "Cash loan",
  "touchpoints": ["Email", "SMS", "Push notification", "Branch visit"]
}
```
Import tolerance: `touchpoints` items as strings OR `{name}` objects (mixed OK); `category` is informational — import always targets the **active** category; a bare array **of strings** is accepted without the wrapper (`["Email", "SMS"]`). A bare array of `{name}` objects is NOT claimed by this branch (it would collide with the themes bare-array check earlier in the chain) — objects need `kind: 'touchpoints'` or a `touchpoints` key. Merge by name (trim, case-insensitive): new names added, existing kept (no fields to update), duplicate names in one file — first wins.

**Build instructions for ux-build**
1. **Constants + state**: `const TOUCHPOINT_KEY = 'touchpoint';` next to `EMOTION_KEY` (index.html:414). `newCategory()` (index.html:451) gains `touchpoints: []`; `newTouchpointItem(name)` → `{ id: uid(), name }`. Accessors: `currentTouchpoints()` (`currentCategory()?.touchpoints ?? []`), `tpName(id)` (personaName pattern), `elementText(el, key)` → `key === TOUCHPOINT_KEY ? (tpName(el.tpId) ?? el.text) : el.text`.
2. **Migration** — `migrateCategoryTouchpoints(cat)`: `if (!Array.isArray(cat.touchpoints)) cat.touchpoints = [];` normalize string items to `{id, name}`; walk journeys → steps → `cells[TOUCHPOINT_KEY]`: for each element with `tpId == null` and non-empty `text` → find-or-create a library item by name (trim, case-insensitive) → set `el.tpId`, keep `el.text`. Idempotent (elements with `tpId` skip). Call from `load()`'s normalization walk (next to the slice-normalizing loop, index.html:615-620) and from `importDatabase()`.
3. **Actions**: `addTouchpoint(name)` (trim; return existing item on duplicate name — no dupes); `renameTouchpoint(tpId, name)` — update the item, then walk the category's journeys updating `el.text` snapshots where `el.tpId === tpId`; `removeTouchpoint(tpId)` — count uses (`touchpointUsage(cat, tpId)` walks cells); 0 → splice immediately, >0 → `confirm()` with the count → splice from the library AND filter those elements out of every cell; `moveTouchpoint(from, to)` — splice/splice + `save()` + re-render manager list.
4. **Manager modal** — `openTouchpointManager()`: overlay dialog on the persona-card/lightbox pattern (`role="dialog" aria-modal="true"`, Escape/backdrop/× close, focus close button on open, restore on close). Content: title "Touchpoints — {category}"; toolbar: add-field (input + Add button, Enter commits, clears) + ↧ Export + ↥ Import (↥ → the existing hidden `importInput.click()`); list rows: ⠿ drag handle (drag-reorder like the sidebar list), name (dblclick or ✎ → inline rename, Enter/blur commit, Escape cancel — empty cancels), usage chip `{n}×` (cells count, `--chip` tokens), hover × delete. Empty state: "No touchpoints yet — add the first one above." Local `renderManagerList()` re-renders on every change; `save()` + `renderGrid()` keep the grid in sync (slice labels/counts unchanged).
5. **Grid entry** — in `renderGrid()`'s slice-label construction (index.html:1691): for `slice.key === TOUCHPOINT_KEY` append a `⚙` button (class like `slice-manage`, hover-reveal like `.step-remove`, `aria-label="Manage touchpoints"`) → `openTouchpointManager()`.
6. **Picker** — in `buildCellElements` (index.html:1737) branch on `slice === TOUCHPOINT_KEY`: the add button text becomes "+ touchpoint" and its click → `openTouchpointPicker(btn, step)` (instead of `addElement`). Popup: absolutely-positioned dropdown anchored under the button (`z-index` above cells), containing: a filter input (only when the list has > 8 items), one row per library item — check/dim state for items present in the cell; click toggles (absent → `addTouchpointElement`, present → `removeElement` of that element), `＋ New touchpoint…` row expanding an inline input (Enter → `addTouchpoint(name)` + `addTouchpointElement`, Escape → back), "Manage…" row → `openTouchpointManager()` + close. Closes on Escape, click-outside, and any re-render of the grid (a module-level `_tpPicker` handle torn down at the top of `renderGrid()`). Element rows in the cell keep the existing ⠿ drag-reorder and × remove; **no dblclick text edit** for touchpoint elements (name comes from the library). `addTouchpointElement(stepId, tpId)` → push `{ id: uid(), tpId, text: tpName(tpId) }`, `save()`, `renderGrid()`.
7. **Display text** — use `elementText(el, slice)` in `buildCellElements` (index.html:1747), `cellAriaLabel` (index.html:1715+), and `exportPng`'s element loop (index.html:1990-1991) so cells, a11y labels and the PNG all show the live library name with the `text` snapshot as fallback.
8. **Export** — `exportTouchpoints()`: active category; payload per the contract (`touchpoints` as plain name strings); `downloadJson(data, 'touchpoints-' + safeFilename(cat.name) + '.json')` (index.html:2027); toast "✓ Touchpoints exported". Disabled/no-op with a toast when there is no active category.
9. **Import** — `importTouchpoints(data)`: requires `currentCategory()` (else toast "Add a category first"); accept `data.touchpoints` (strings or `{name}`) or a bare array of strings; merge by name case-insensitive (find-or-create via `addTouchpoint`), first-wins on in-file duplicates; `save()` + re-render (manager if open); return `{added, exists}`. Detection in `importFromFile()` (index.html:2230-2233) — insert **after themes, before journey**: `data.kind === 'touchpoints' || Array.isArray(data.touchpoints) || (Array.isArray(data) && data.length > 0 && data.every(t => typeof t === 'string'))` (bare **string** arrays only — object arrays stay with themes). Toast `✓ Touchpoints imported (N added, M exist)`.
10. **Database export/import** — `exportDatabase()` (index.html:2051): categories are deep-cloned — `touchpoints` rides along, nothing to add. `importDatabase()` (index.html:2149): per category call `migrateCategoryTouchpoints(cat)` after `normalizeJourney` (handles missing libraries, string items, old elements), then build a per-category `tpMap` (`uid()` regen) and remap every element's `tpId` in the existing journey walk (index.html:2166-2174, alongside the persona/theme remaps).
11. **Journey export/import** — `exportJourney()` (index.html:2063): add `touchpoints: [{name}]` — unique names referenced by the journey, first-use order. `importJourney()` (index.html:2184): after normalize + regen, resolve each touchpoint element in the target category: find-or-create a library item by `el.text` (fallback: the snapshot list by position) → set `el.tpId` + refresh `el.text`. Old journey exports (no `touchpoints` field) still resolve via element texts.
12. **CSS** (tokens only): manager modal (`.tm-*` — reuse the persona-card overlay/panel/shadow), picker (`.tp-picker`, `.tp-item` + present/checked state, `.tp-new`, `.tp-filter`), `.slice-manage` (hover-reveal ⚙), usage chip via `--chip`/`--chip-text`. A11y: dialog roles, `aria-label`s on ⚙/✎/×/⠿, visible `:focus-visible`, picker operable from the keyboard (arrows + Enter, Escape closes).
13. **Regression**: other slices keep inline text editing, drag-reorder and PNG rendering untouched; personas/themes/screenshots/emotion flows unchanged; migration is idempotent (safe on every boot); the widened import chain must not swallow existing kinds (database needs `.categories`, personas `.kind === 'personas'`, themes object-arrays, journey `.journey`/`.stages` — a bare string array matches none of them).

### Change 11 — feature: readable screenshots — full-width in cell + big in PNG export (2026-08-24)
**Status**: built — static verification passed (script syntax, no stale references); click-test pending

### Change 12 — bug: touchpoint library unreachable — picker/manager never wired into UI (2026-08-24)
**Status**: built — all 4 fix points applied; static verification passed (syntax, call sites, picker teardown)

### Change 13 — feature: tidy slice set — merge Mindset→Insights, add Backstage/Duration/Metrics (2026-08-24)
**Status**: built — static verification passed (syntax; `mindset` only in the migration helper; call sites in `load()` + `normalizeStep`)

### Change 14 — feature: Follow-ups row (channel dictionary + message + action) + touchpoint JSON gaps fixed (2026-08-24)
**Status**: built — static verification passed (syntax; no stale refs; all factory/JSON call sites wired)

**User goal**
A **Follow-ups** row: each element = a **channel picked from a dictionary** (push, SMS, e-mail… — analogous to touchpoints, but a **separate dictionary**) + the **message text** + an **action** (free text, what the follow-up triggers). The row captures what the bank sends to the customer at a given step.

**MVP scope**
- **11th slice** `follow-up` (label "Follow-ups"), positioned right after Touchpoints.
- **Follow-up channel dictionary per category** (`category.followups`): ⚙ manager on the row label (add / inline rename / delete with usage-count confirm + cascade / drag-reorder / ↧ ↥ JSON) and a picker on "+ follow-up" — mirrors the touchpoint pattern exactly.
- **Element** `{ id, fuId, text, action }`: channel chip + editable message text + editable action (inline edit, Enter/blur/Escape); chip non-interactive in the MVP.
- Picker is **single-add**: picking a channel creates the element, closes the picker and focuses the new element's text field.
- Composed display in ARIA and PNG (`Channel · message → action`; empty parts omitted).
- JSON: `kind: 'followups'` export/import (merge by name); journey export carries the used channel names, import re-resolves by name (find-or-create); database export/import carries `followups` with id regen + `fuId` remap.
- **Folded fixes — pre-existing Change 10 gaps** (found while scoping): `exportTouchpoints`/`importTouchpoints` are *called but undefined* (manager ↧ → ReferenceError, `index.html:1555`); `importFromFile` has **no touchpoints branch** (a touchpoints JSON → "✗ Invalid file"); `exportJourney` carries no dictionary snapshot and `importJourney` doesn't resolve `tpId` (imported touchpoint elements show stale names, not library-linked).

**Later (deferred)**
- Click a follow-up's channel chip → re-pick the channel in place.
- Message templates / reuse; a "delay" field (e.g. "2 days after"); per-channel icons or colors; an all-categories followups file.

**Impact**
- **Data**: + `category.followups: [{id, name}]` (localStorage, tiny); + slice `follow-up`; elements `{id, fuId, text, action}` in `cells['follow-up']`. Normalization backfills (array check + missing slice key). **No key version bump.**
- **Actions**: follow-up dictionary CRUD (`addFollowup`/`renameFollowup`/`removeFollowup`/`moveFollowup`/`followupUsage`), `addFollowupElement`/`removeFollowupElement`, `exportFollowups`/`importFollowups`, import-detection branch; `startElementEdit` extended with a field param (text|action); touchpoint `exportTouchpoints`/`importTouchpoints` finally defined.
- **Screens**: 11-row grid; ⚙ on the Follow-ups label; picker under "+ follow-up"; element rows with a channel chip. No sidebar changes.
- **States**: empty dictionary (picker/manager empty states mirroring touchpoints); import with no active category → toast; channel-in-use delete → confirm + cascade; empty action simply omitted in display.
- **Interactions**: same as touchpoints (picker keyboard nav, Escape/click-outside, inline rename in manager, drag-reorder) + two-field inline editing in cells.
- **Edge cases**: channel rename → chips/composed displays update live (`fuId` resolution); deleting a channel in use → confirm + remove elements; journey import when a channel name doesn't exist → find-or-create; database import regens ids + remaps `fuId` (also `tpId` — the Change 10 #10 remap never landed); old data without `followups` → empty array.
- **Glossary**: `follow-up` (row/element), `followup-channel` (dictionary item), `followup-library`, `followups-file` (`kind:'followups'`), `dictionary-factory` (the shared picker/manager mechanism).

**`kind: 'followups'` contract format** (authored/stored in `marekbrze/business`, e.g. `customer-journey/followups-<process>.json`):
```json
{
  "version": 1,
  "kind": "followups",
  "app": "customer-journey",
  "category": "Cash loan",
  "followups": ["Push", "SMS", "E-mail"]
}
```
Items as strings OR `{name}` objects; `category` informational (import targets the **active** category); merge by name (trim, case-insensitive), first-wins on in-file duplicates. A **bare string array is NOT claimed** by this branch (touchpoints already owns it) — objects or no-wrapper files need `kind:'followups'` or a `followups` key.

**Build instructions for ux-build**
1. **Constants + slice**: `const FOLLOWUP_KEY = 'follow-up';` next to `TOUCHPOINT_KEY` (`index.html:468`). Insert `{ key: 'follow-up', label: 'Follow-ups' }` into `SLICES` right after `touchpoint`.
2. **Category model**: `newCategory()` (`index.html:514`) gains `followups: []`; add `normalizeCategoryFollowups(cat)` (ensure array + `{id,name}` shape — no legacy resolution needed) called next to `migrateCategoryTouchpoints` in `load()` and `importDatabase()`.
3. **Dictionary CRUD** (mirror the touchpoint block ~`index.html:921-971`): `addFollowup` (find-or-create by name), `renameFollowup`, `removeFollowup` (usage count → confirm → cascade over `cells[FOLLOWUP_KEY]`), `moveFollowup`, `followupUsage`; `addFollowupElement(stepId, fuId)` → push `{id: uid(), fuId, text:'', action:''}` + `save()` + `refreshLibCell`; `removeFollowupElement`.
4. **Cell rendering** — branch in `buildCellElements` (`index.html:2180`) for `FOLLOWUP_KEY` (next to the existing `isTp` branch): row `[⠿][chip][text][action][×]` — chip = non-interactive span styled with `--chip`/`--chip-text` showing `fuName(el.fuId)`; text span dblclick → `startElementEdit(text, step.id, slice, el.id, el.text, 'text')`; action span (prefixed "→ ", `var(--text-3)`) dblclick → same with `'action'`; drag-reorder + × as elsewhere; "+ follow-up" → picker. Extend `startElementEdit` (`index.html:1372`) with a 6th param `field = 'text'` writing `el[field]` — default keeps every existing caller unchanged.
5. **Generalize picker/manager into a dictionary factory**: parameterize `openTpPicker(anchor, step, cfg)` (`index.html:1412`) and `openTouchpointManager(cfg)` (`index.html:1525`) over a cfg `{ key, itemLabel, managerTitle, items(), usage(), add(), rename(), remove(), move(), export(), import(), addEl(), removeEl() }`. Touchpoint call sites pass `TP_CFG` (behavior identical — regression surface), follow-ups pass `FU_CFG`. `refreshTpCell` → `refreshLibCell(key, stepId)`; cell class hook: add `fu-cell` next to `tp-cell` (`index.html:2143`). Reuse the existing `.tp-*` CSS for both.
6. **Follow-ups picker behavior**: single-add — channel click → `addFollowupElement` → `closeTpPicker()` → after `refreshLibCell`, focus the new element's text (open inline edit immediately). Keep "＋ New…" and "⚙ Manage list…" rows.
7. **JSON IO — followups**: `exportFollowups()` / `importFollowups(data)` per the contract (mirror the touchpoint pair). Detection in `importFromFile` (`index.html:2666`) — order: database → personas → themes → touchpoints → **followups** (`data.kind === 'followups' || Array.isArray(data.followups)`) → journey → invalid.
8. **JSON IO — touchpoint fixes (folded)**: define `exportTouchpoints()` (kind `'touchpoints'`, file `touchpoints-<category>.json`, items as names — call site already at `index.html:1555`) and `importTouchpoints(data)` (Change 10 #9 tolerance incl. the bare-string-array claim); add the touchpoints branch to `importFromFile` (`data.kind === 'touchpoints' || Array.isArray(data.touchpoints) || bare string array`).
9. **Journey export/import** (`exportJourney` `index.html:2511` / `importJourney`): payload gains `touchpoints: [{name}]` and `followups: [{name}]` (unique, first-use order); import resolves both by name (find-or-create in the target category), sets `tpId`/`fuId` and refreshes the touchpoint `el.text` snapshot.
10. **Database export/import** (`exportDatabase` `index.html:2499` / `importDatabase` `index.html:2598`): dictionaries ride along in the deep-cloned categories; on import, regen `touchpoints`/`followups` item ids into `tpMap`/`fuMap` and remap every element's `tpId`/`fuId` in the existing walk (next to the persona/theme remaps) — also bump `state.nextId` past all surviving ids.
11. **Display compose** — extend `elementText(el, key)` (`index.html:541`): `key === FOLLOWUP_KEY` → `` `${fuName(el.fuId) ?? '—'}${el.text ? ' · ' + el.text : ''}${el.action ? ' → ' + el.action : ''}` ``. Used by `cellAriaLabel` and the `exportPng` element loop (one wrapped line per element).
12. **CSS** (tokens only): `.fu-chip` (chip look via `--chip`/`--chip-text`), `.fu-action` (arrow prefix, `--text-3`); everything else reuses `.tp-*`.
13. **Regression check**: touchpoints behave exactly as after Change 12 (picker multi-add, ⚙ manager, rename propagation) — the factory must not change semantics, only parametrize; other slices' inline editing untouched (`field` param defaults to `'text'`); PNG/aria of the 10 existing rows unchanged; journey/database round-trip preserves follow-ups and touchpoint links.

**User goal**
Tidy up the journey rows: Touchpoints, Emotions, Pain points, Ideas and Screenshots must stay (and Actions — confirmed in the interview). Mindset and Insights felt like the same row — merged into **Insights**, understood as **the persona's first impression of the step** (`"pierwsze wrażenie osoby"`), placed **before Emotions and Pain points**. Three rows added: **Backstage** (what the bank/team does behind the scenes), **Duration** (customer-perceived time, e.g. "2 days waiting"), **Metrics** (KPI per step).

Final set — 10 rows, fixed and global (not per-journey):
`Actions · Touchpoints · Insights · Emotions · Pain points · Ideas · Backstage · Duration · Metrics · Screenshots`

**MVP scope**
- `SLICES` reordered/merged/extended to the 10 rows above; **Mindset removed** with a data migration: per cell, `cells.mindset` elements **append to `cells.insight`** (existing insights first), then the `mindset` key is deleted.
- Migration runs in `load()` **and** on import paths (old JSON backups/journey exports carrying `mindset`).
- New rows are plain text-element rows (exactly like Ideas) — grid, PNG export, ARIA and normalization adapt automatically via the existing `SLICES` iteration.

**Later (deferred)**
- **Follow-up row** — elements combining a dictionary channel (push, SMS — the touchpoint-library pattern) + message text + an action; a structured element variant, its own Change.
- Per-journey row visibility/reorder (a slice manager) — not needed while the set is fixed and global.
- Structured Duration (number + unit) or typed metric values — plain text for now.

**Impact**
- **Data**: slice set is a code constant (never stored per step) — the only data change is the migration merging `mindset` element lists into `insight` and dropping the key. New keys (`backstage`, `duration`, `metric`) are backfilled empty by the existing normalization loops. **No key version bump** — additive, idempotent migration in `load()`/`normalizeStep`.
- **Actions**: none new — new rows use the generic element CRUD (`addElement`/`startElementEdit`/drag-reorder).
- **Screens**: 10 rows in the grid; nothing new UI-wise (⚙ stays on Touchpoints only).
- **States**: new rows behave like Ideas (empty → "—" in grid and PNG).
- **Interactions**: none new.
- **Edge cases**: a cell with only mindset elements → they become the insights list; old journey/database exports with `mindset` → migrated on import via `normalizeStep`; PNG gets taller (10 rows) — fine, digital/scrollable is the format; idempotency — re-running the migration on already-migrated data is a no-op.
- **Glossary**: `insight` (redefined: the persona's first impression of the step — absorbs the former `mindset`), `backstage`, `duration`, `metric`, `follow-up` (Later).
- **Spec note**: Requirements §Data model enumerates 7 fixed slices — superseded by this Change's 10.

**Build instructions for ux-build**
1. **`SLICES`** (`index.html:454-464`): replace with the 10 rows in the order above (keys: `action, touchpoint, insight, emotion, pain-point, idea, backstage, duration, metric, screenshots`; labels: `Actions, Touchpoints, Insights, Emotions, Pain points, Ideas, Backstage, Duration, Metrics, Screenshots`). No `mindset` entry.
2. **Migration helper**: `mergeMindsetIntoInsights(step)` — `const c = step.cells || {}; if (Array.isArray(c.mindset)) { (c.insight = Array.isArray(c.insight) ? c.insight : []).push(...c.mindset); delete c.mindset; }`. Idempotent by construction.
3. **Call sites**: (a) `load()`'s cell-normalization walk (`index.html:685`, next to the `SLICES.forEach` fill — merge **before** the fill so `insight` exists); (b) `normalizeStep()` (`index.html:2517-2520`, before its `SLICES.forEach` fill) — this covers `importJourney` and `importDatabase` via `normalizeJourney`.
4. **Auto-adapting (verify, no changes expected)**: `freshCells` (`index.html:492`), `renderGrid` row loop (`index.html:2105`), `exportPng` `sliceRows` (`index.html:2322`), `regenStep` (`index.html:2536`), `cellAriaLabel`, `buildCellElements` (generic path for the 3 new rows).
5. **Regression check**: `grep mindset` after the change → only the migration helper and its call sites; touchpoint picker/⚙ wiring (Change 12) untouched (Touchpoints stays key row 2); emotion row logic keyed on `EMOTION_KEY`, unaffected by reordering; a journey with mindset-only cells → elements visible in Insights after refresh; old backup JSON → import migrates.
**Severity**: 🟡 medium — Change 10's core workflow (manage touchpoints in one place, pick from the list in cells) is completely unreachable; no data loss, the rest of the app works, old free-text behavior persists as a de-facto fallback.

**Reproduction**
1. Open a CJ → look at the Touchpoints row in the grid.
2. A cell's add button says "+ item" → click → generic inline free-text edit; dblclick an existing element → inline text edit (pre-Change-10 behavior).
3. Look for the library manager: no ⚙ on the "Touchpoints" row label; no entry in the header or sidebar — nothing anywhere opens the picker or the manager.
**Expected** (Change 10): "+ touchpoint" in a cell opens a picker fed by the category's library; ⚙ on the Touchpoints slice label opens the manager modal. **Actual**: free-text editing; the picker, manager modal, CRUD and import/export all exist in code but have zero UI entry points. **Reliability**: every time — the entries are dead code (no call sites), not a conditional failure.
**Location**: `index.html:2126` (grid routes touchpoint cells to the generic `buildCellElements` — no `TOUCHPOINT_KEY` branch), `index.html:2193` ("+ item" → `addElement`), `index.html:2108-2113` (slice label built without the ⚙), `index.html:1399` (`openTpPicker` — 0 call sites), `index.html:1483` (`openTouchpointManager`'s only entry — a row inside the unreachable picker), `index.html:529` (`elementText` — 0 call sites).

**Root cause**
**Class**: spec-vs-code drift — incomplete Change 10 build.
**Cause**: The Change 10 build (checkpoint commit 5b4c359) implemented the internals — the picker (`openTpPicker`), the manager modal (`openTouchpointManager`), CRUD + migration + import/export, and the `.slice-manage` CSS — but never applied its own build instructions #5, #6 and #7: no ⚙ button is appended to the Touchpoints slice label, `buildCellElements` never branches for touchpoint cells (so the cell's `+` still calls the generic `addElement` and elements stay inline-editable free text), and `elementText` was never adopted at the display sites. Result: a fully implemented feature with no way to reach it.
**Evidence**: `grep openTpPicker` → definition only (1399); `grep elementText` → definition only (529); display sites read the stale `el.text` snapshot directly: `index.html:2165` (cell text), `index.html:2141` (`cellAriaLabel`), `index.html:2409` (`exportPng`). `importDatabase` (2567) never calls `migrateCategoryTouchpoints` (only `load()` does, 689) — pre-Change-10 database backups imported after this build keep `tpId == null` elements outside the library. Spec (intent): SPEC.md Change 10 MVP scope + build instructions #5–#7.

**Fix plan**
- **Change** — wire the entry/display points exactly per Change 10 build instructions #5–#7:
  1. `buildCellElements(step, slice)` (`index.html:2155`): branch for `slice === TOUCHPOINT_KEY` — add-button labeled "+ touchpoint" → `openTpPicker(btn, step)` instead of `addElement`; element rows render `elementText(el, slice)` and keep ⠿ drag-reorder + × remove, but **no dblclick/inline text edit** (the name comes from the library).
  2. Slice-label construction (`index.html:2108-2113`): for `slice.key === TOUCHPOINT_KEY` append a ⚙ button (`class="slice-manage"`, CSS already at `index.html:330-332`, `aria-label="Manage touchpoints"`, hover-reveal) → `openTouchpointManager()`.
  3. Display text via `elementText(el, slice.key)` at `index.html:2165` (cell), `index.html:2141` (`cellAriaLabel`), `index.html:2409` (`exportPng`) — live library name with the `text` snapshot as fallback.
  4. Secondary: call `migrateCategoryTouchpoints(cat)` per imported category in `importDatabase()` (`index.html:2567`) so old backups resolve free-text elements into the library.
- **Spec impact**: none — this is Change 10 implemented as specced; nothing beyond it.

**Regression scope**
- `buildCellElements` is shared by all non-emotion/non-screenshot slices — the branch must trigger only on `slice === TOUCHPOINT_KEY`; verify after the fix that action/mindset/pain-point/idea/insight cells still have "+ item", dblclick inline edit and drag-reorder.
- `elementText(el, key)` is behavior-neutral for other slices (`key !== TOUCHPOINT_KEY` → returns `el.text`) — safe at all three sites.
- Picker lifecycle: an open picker must close on grid re-render — `closeTpPicker` exists (`index.html:1382`); verify `renderGrid()` tears down `_tpPicker` at its top (Change 10 #6).
- Touchpoint rows lose dblclick edit only (intended); `startElementEdit` untouched for other slices.
- A11y: ⚙ needs `aria-label` + `:focus-visible` (CSS 331 covers it); picker rows keyboard-operable (arrows + Enter, Escape) per Change 10 #12.
- `importDatabase` change is additive (idempotent migration) — verify a round-trip export → import of a current database changes nothing.

**User goal**
Screenshots must be readable, not just recognizable: in the grid AND in the exported PNG. The export is consumed **digitally** (scrollable, no need to see everything at once; sometimes displayed on a TV/projector) — so the PNG may simply be much bigger; there is no A4 constraint. On screen the user chose **full-width images stacked vertically in the cell** (cropping-free, tallest cells, most reading room).

**MVP scope**
- **Grid**: the Screenshots cell renders screenshots as a **vertical stack of full-cell-width images** — no cropping (`object-fit: contain`), height from the stored aspect ratio; `+` becomes a full-width dashed add-bar. Step columns widen (`STEP_COL_W` 220 → 300) so "full width" means real reading room. Portrait/very tall screenshots get a max-height cap (letterboxed, still no crop).
- **PNG export**: step columns widen (`colW` 200 → 440, `labelW` 140 → 170) and the Screenshots band draws images **stacked at (near) full column width** (single-image height cap ~360px) instead of today's 72px strip; band height computed dynamically per the tallest column (existing mechanism, new plan semantics). Missing blobs → dashed placeholder at the stacked size.

**Later (deferred)**
- S/M/L size control for the Screenshots row (per interview — fixed layout chosen for MVP).
- Proportional scaling of the **whole** export (fonts/labels together with columns) — explicitly deferred by the user ("Yes, cut it" over "Scale whole export").
- Screenshot captions + drag-reorder of thumbnails in a cell (already Change 7 Later).

**Impact**
- **Data**: none — presentation only. No new entities/fields, no storage change, **no key version bump**.
- **Actions**: none new; `exportPng()` internals change (layout constants + shot band layout).
- **Screens**: Screenshots row in the grid (stacked full-width thumbs, full-width add-bar); PNG export column widths. No new surfaces or nav.
- **States**: none new; the empty Screenshots cell shows the full-width dashed add-bar (existing empty state, restyled).
- **Interactions**: unchanged — click thumbnail → lightbox, hover × → remove, drag & drop files onto cell, Ctrl+V paste, file picker. Keyboard/focus untouched.
- **Edge cases**: portrait/tall screenshots → max-height cap with `contain` (no crop, centered) so a single phone screenshot can't make a cell absurdly tall; many screenshots per cell → tall row (accepted — digital, scrollable); export row height = tallest column's stack; very many steps × wider columns → canvas stays within browser limits (~32k px — 440px × realistic step counts is far below); missing blob in IDB → dashed placeholder at the stacked slot size (existing pattern).
- **Glossary**: none new.

**Build instructions for ux-build**
1. **Grid CSS** (`index.html:278-300`): `.g-cell .shots` → `flex-direction: column; gap: 6px;` (keep `width: 100%`). `.shot-thumb` → `width: 100%; height: auto; max-height: 440px;` (drop `height: 56px` and `flex-shrink`; keep border/radius/background; the inline `aspect-ratio` set at `index.html:1138` stays). `.shot-thumb img` → `width: 100%; height: 100%; object-fit: contain;` (drop `height: 100%; width: auto; max-width: 160px; cover`). `.shot-add` → `width: 100%; height: 48px;` (full-width dashed bar; drop the 34×56 box). Hover-reveal `.shot-remove` unchanged (top-right of the thumb).
2. **Column width**: `STEP_COL_W` 220 → 300 (`index.html:452`). `LABEL_COL_W` unchanged (150). The grid already scrolls horizontally for many steps.
3. **Export constants** (`index.html:2287`): `colW: 200 → 440`, `labelW: 140 → 170`. Fonts and the rest of the metrics stay as-is (deferred by the user).
4. **`shotDrawPlan` → stacked** (`index.html:2255-2267`): per shot — `dw = availW; dh = h * (availW / w);` if `dh > maxH` (new param, 360) → `dh = maxH; dw = w * (maxH / h);` round. `contentH = Σ dh + gap * (n-1)` (+ `Math.max(12, …)` floor). Return `{ gap, places, contentH }` (`slotW` no longer meaningful — drop it and its uses).
5. **Band height** (`index.html:2306-2312`): keep the max-over-columns logic, now feeding the stacked plan: `shotDrawPlan(shots, colW - 16, 360)`; `h = any ? maxH + 14 : 34`.
6. **Band drawing** (`index.html:2382-2404`): iterate the stack — `dx = x1 + 8 + (availW - dw) / 2` (centers capped/portrait images), `dy` starts at `y + 7` and advances `dh + gap`; bitmap via `ctx.drawImage`, missing blob → dashed `strokeRect` at the same slot; keep the soft 1px inner border.
7. **Regression check**: lightbox, add (picker/drag&drop/paste), remove + blob cascade, drag-over highlight, JSON export/import, brand themes, emotion row, other slices' inline editing — all untouched; only the Screenshots row's geometry changes (screen + PNG). Verify a journey with 0 shots (band height 34, "—" placeholder), 1 shot, mixed landscape/portrait shots, and a missing blob (placeholder path).

### Change 15 — feature: AI prompt — copyable chat instructions to create/expand journeys as importable JSON (2026-08-25)
**Status**: planned — route to ux-build

**User goal**
Generate and expand customer journeys **in an AI chat** and import the result straight into the tool: a copyable prompt that (a) explains the journey JSON contract so a chat can **draft a new CJ from scratch** — interviewing the user first, in the spirit of Matt Pocock's grill-me / grill-with-docs skills — and (b) ships the **current journey's JSON** so the chat can **expand an existing map**, with the answer importable via the existing ↥ Import.

**Interview decisions (2026-08-25)**
- Both flows in the MVP; one modal, two tabs.
- Create flow = **grill-style interview**: the chat asks **~10–20 questions in small rounds** (not 40+, not zero), each round building on what's settled; accepts **pasted research** (interviews, notes, docs) and mines it instead of inventing; user can cut it short with "generate now" at any point.
- Entry point: a **header button** ("✦ AI prompt") — the Create flow isn't tied to the open journey.
- **Import stays as-is** (always adds a new copy — never replaces). Journey **versioning/replace-on-import is a future Change** the user explicitly wants (also in Idea → Later).

**MVP scope**
- **"✦ AI prompt" button in the header** → modal (persona-card overlay pattern) with two tabs:
  - **Create from scratch** — a self-contained prompt: role → interview mode (10–20 questions, rounds, pasted docs, assumptions marked, "generate now" escape) → row semantics for all 11 slices → the full JSON contract → output rule (single ```json block).
  - **Expand current journey** — a shorter prompt + the **current journey's JSON injected** (the `exportJourney` payload **without `images`**, pretty-printed); instructs the chat to return the **complete updated JSON** (never a diff), keep the title, keep `screenshots` arrays verbatim. Extra button: **"Copy JSON only"** (today's export only downloads a file; pasting into a chat needs the clipboard).
- **Copy to clipboard** with ✓/✗ feedback (first `copyText` helper in the tool — CLAUDE.md §3 pattern); the prompt textarea stays selectable as a manual fallback.
- Prompts are **English, hardcoded constants** — readonly, not user-editable, not persisted.

**Later (deferred)**
- **Journey versioning** (history / variants / replace-on-import when the title matches) — the user's declared next-era feature; also Idea → Later ("Variants", "Version history").
- Dictionary prompts (touchpoints / follow-up channels as extra tabs of the same modal).
- Editable/persisted prompt tweaks; downloading the prompt as `.md`.
- Multimodal flow (attaching screenshots to the chat) — today images can't travel through a chat round-trip.

**Impact**
- **Data**: **none** — prompts are code constants; the injected JSON is a serialization of the current journey (subset of the existing export payload). No new entities/fields, **no key version bump**.
- **Actions**: `openAiPrompt()` (modal), `buildAiPromptCreate()` / `buildAiPromptExpand()` (prompt text), `journeyJsonForPrompt()` (images-less serialization — refactored out of `exportJourney`), `copyText(text, btn)` (clipboard + feedback). No import/export behavior changes.
- **Screens**: header button next to `#btn-import` (`index.html:393`); modal = overlay panel with a tab bar, a "How it works" line, a readonly textarea and Copy button(s). No sidebar/grid changes.
- **States**: Expand tab with **no open journey** → hint "Open or create a journey first." (no prompt, no copy buttons); clipboard failure → "✗ Failed" on the button (+ manual copy remains possible); ✗ copy success feedback auto-reverts after ~1.5 s.
- **Interactions**: clipboard copy (new pattern in this tool); modal keyboard — Escape/×/backdrop close, focus × on open, restore focus on close (persona-card pattern); tabs switch content in place.
- **Edge cases**: `navigator.clipboard` unavailable (permission / context) → ✗ feedback, textarea selectable for manual Ctrl+C; very large journey → long prompt (accepted — no trimming in the MVP); AI answer saved as `.json` and imported through the existing chain — malformed JSON already handled ("✗ Invalid file"); screenshots: the prompt JSON omits `images`, and the chat is told to echo `screenshots` arrays verbatim — the imported copy then **shares blob ids** with the original (renders fine; deleting the *original* journey cascades its blobs → the copy shows dashed placeholders, the existing missing-blob state); import adds a copy — the user prunes the old one manually (decided).
- **Glossary**: `ai-prompt` (the copyable chat instruction), `ai-prompt-modal`, `create-prompt`, `expand-prompt`, `prompt-json` (images-less journey serialization for chats), `copy-text`.

**JSON contract the prompt teaches** (= the existing `kind: 'journey'` format, documented for a chat): wrapper `{ version: 1, kind: 'journey', app: 'customer-journey', journey, persona, touchpoints, followups }` — `images` omitted; `journey = { title, description (scenario), expectations, stages: [{ name, steps: [{ name, cells }] }] }`; `cells` with the 11 slice keys — text rows = `[{ text }]` (touchpoint elements may add `tpId`, follow-up elements = `[{ fuId, text, action }]`), `emotion` = integer **1–5** or `null`, `screenshots` kept verbatim if present; `persona = { name, role?, description?, goals?, needs?, frustrations?, quote? }` (matched/created by name); `touchpoints` / `followups` = `[{ id, name }]` dictionaries — follow-up elements reference them by `fuId` (touchpoints also resolve by `text` fallback); **all ids optional** (the importer regenerates them); unknown fields ignored; no comments, no trailing commas.

**Build instructions for ux-build**
1. **Refactor `exportJourney`** (`index.html:2719-2749`): extract the payload build (persona/theme snapshots + tp/fu lists + journey clone) into `buildJourneyPayload(j)` returning the object WITHOUT `images`; `exportJourney` awaits `exportImagesForSteps` and attaches `images` before download (downloaded file identical to today — regression-check it). Add `journeyJsonForPrompt(j)` → `JSON.stringify(buildJourneyPayload(j), null, 2)` (ids kept — `fuId` references need them).
2. **Prompt constants** (English, template literals, next to other constants): `AI_PROMPT_CREATE` — sections: (1) *Role*: help a UX designer build a customer-journey map for a banking process; the map gets imported into a tool from JSON. (2) *Interview first*: before any JSON, interview the user — about 10–20 questions total, in small rounds of 2–5, each round building on what is already settled (process & goal → persona & scenario & expectations → stages & candidate steps → known pain points & research → available data: durations, metrics); at any point the user may paste research materials — mine them instead of inventing; the user may say "generate now" to skip the rest. (3) *Assumptions*: anything not grounded in answers or pasted material is an assumption — list assumptions in the reply before the JSON. (4) *Row semantics*: a compact table of the 11 rows (Actions — what the customer does; Touchpoints — contact points, picked from the dictionary; Follow-ups — channel + message + action the bank sends; Insights — the persona's first impression/thoughts; Emotions — 1–5 sentiment; Pain points — customer's frustrations; Ideas — improvement opportunities; Backstage — what the bank/team does behind the scenes; Duration — customer-perceived time; Metrics — KPI per step; Screenshots — images, cannot be generated) + writing rules: one thought per element, customer perspective vs. bank perspective, short concrete phrases. (5) *JSON contract*: the shape above with a small filled-in example (2 stages × 2 steps, a few rows). (6) *Output rule*: finish with the complete JSON in a single ```json code block, nothing after it — valid JSON, no comments/trailing commas. `AI_PROMPT_EXPAND` — role + "here is my current journey JSON:" + `${journeyJsonForPrompt(j)}` + instructions: I'll request changes (fill gaps, add/remove/reorder steps, restructure, enrich rows) — apply them and return the **complete updated JSON in the same format** (never a diff or fragment); keep the title unless asked; keep `screenshots` arrays **exactly as they appear** (images can't travel through chat); mark assumptions; end with a single ```json block.
3. **Header button**: `<button class="btn btn-ghost btn-sm" id="btn-ai-prompt">✦ AI prompt</button>` next to `#btn-import` (`index.html:393`), `title="Copy a prompt to create or expand a journey with an AI chat"`; click → `openAiPrompt()`.
4. **`openAiPrompt()`**: overlay modal on the `openPersonaCard` pattern (`index.html:1399-1473`) — `role="dialog" aria-modal="true" aria-label="AI prompts"`, Escape/×/backdrop close, focus × on open, restore on close; panel `max-width: 720px; width: min(92vw, 720px); max-height: 85vh; display: flex; flex-direction: column`. Content: tab bar (two buttons — *Create from scratch*, *Expand current journey*; selected state via `aria-pressed`/class, Create default), a one-line "How it works: 1. copy the prompt → 2. paste into an AI chat → 3. save the JSON it returns and ↥ Import it here" (`--text-3`), a readonly `<textarea class="ap-prompt" spellcheck="false" readonly>` (monospace 11–12px, `min-height: 260px`, `flex: 1; resize: vertical`), and a button row: **Copy prompt** (`.btn-primary .btn-sm`) + on the Expand tab **Copy JSON only** (`.btn-ghost .btn-sm` → copies just `journeyJsonForPrompt(currentJourney())`). Tab switch rebuilds the textarea + buttons in place (no re-open). Expand tab with no `currentJourney()` → instead of the prompt: the hint "Open or create a journey first." and no copy buttons.
5. **`copyText(text, btn)`**: `navigator.clipboard.writeText(text)` in try/catch → on success swap the button label to "✓ Copied" for ~1.5 s then restore; on failure "✗ Failed" (keep the textarea selectable — manual Ctrl+C is the documented fallback).
6. **CSS** (tokens only): `.ap-tabs` (ghost buttons + selected state on `--accent`), `.ap-prompt` (mono, `--inset` bg, `--border`, focus-visible ring), button row gap. Both themes AA as today.
7. **Regression check**: `exportJourney` still downloads a file WITH images (byte-comparable payload keys); import chain untouched (the prompt only teaches today's format); header layout with the extra button; modal focus/Escape behavior; clipboard feedback timer doesn't leak between clicks (clear the previous timer).
