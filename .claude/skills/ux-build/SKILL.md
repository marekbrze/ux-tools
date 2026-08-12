---
name: ux-build
description: >
  Build or extend a single-file HTML tool from apps/<name>/SPEC.md, obeying the design system and
  conventions in CLAUDE.md (one file, vanilla JS, no dependencies, localStorage + IndexedDB,
  dark theme, drag&drop, paste, export, built-in edge-case states). Produces apps/<name>/index.html.
  On first run builds the whole app from the Requirements section; on later runs implements the
  latest pending Change entry in SPEC.md (added by ux-feature or ux-bug). This is the ONLY
  code-writing skill in ux-tools. Triggers on: "build", "implement", "make this tool",
  "write the app", "build the tool", "implement the spec", "apply the change from the spec",
  "implement the fix/feature". Reads SPEC.md + CLAUDE.md.
---

You are a builder. You implement single-file HTML tools that match `image-slicer` in quality and structure. You write **one file**: `apps/<name>/index.html`. Everything — HTML, CSS, JS — lives in it. No frameworks, no build step, no external resources.

## Which tool?

Establish the target (`apps/<name>/`) before you do anything:
1. If the user gave a name — as a skill argument (e.g. `/ux-build customer-journey`) or in a message ("build customer-journey") — use `apps/<name>/`.
2. Otherwise check `ls apps/` (folders with a `SPEC.md` or `index.html`). **One** → take it automatically. **Several** → ask once, which one. **None** → tell the user to run `ux-idea` + `ux-spec` first.
3. Confirm once: "I'm working on `apps/<name>/`."

In the rest of this skill `<name>` is the chosen folder.

## Git checkpoint

Every ux-tools skill leaves a clean git history — each stage is a separate, reversible checkpoint. Commits land **only on the current branch**: never push, never create branches, never rewrite history.

### Before work — checkpoint pending changes
1. Is this a git repo? `git rev-parse --is-inside-work-tree` — an error means it's not a repo, skip and tell the user.
2. Anything pending? `git status --porcelain` — empty = move on.
3. **Stop and ask**, if there's an unfinished merge/rebase/cherry-pick, unresolved conflicts, or staged changes you didn't make.
4. `git add -A && git commit -m "chore(ux): checkpoint before ux-build"`.
5. Tell the user what you staged.

### After work — commit this skill
1. `git status --porcelain` — empty = skip.
2. `git add -A && git commit -m "ux-build(<tool>): <short summary>"` — e.g. `ux-build(customer-journey): build initial app`.
3. Tell the user the hash and what's in the commit.

## Prerequisites

Read, in order:
1. `apps/<name>/SPEC.md` — the spec. If it has no **Requirements** section, tell the user to run `ux-spec` first.
2. `CLAUDE.md` (repo root) — **mandatory**. It defines the design system (palette, fonts, components) and the hard conventions. Every tool you build obeys it. Do not re-derive the palette from memory — read it.
3. The `image-slicer` `index.html` (via `gh api repos/marekbrze/image-slicer/contents/index.html` if you need the canonical pattern for state/save/load, sidebar CRUD, drag&drop, drop zone, canvas export). It is the reference implementation.

## Context detection — what am I building?

- **No `apps/<name>/index.html` yet** → build the whole app from the **Requirements** section.
- **`index.html` exists** + a pending `### Change N — ...` entry under `## Changes` in SPEC.md → implement **that change** (the latest un-implemented one). Confirm with the user which change if several are pending.
- If neither, ask the user what they want built.

## Build rules — every tool obeys these (from CLAUDE.md)

**Single file, zero dependencies**
- One `index.html`: `<!DOCTYPE html>` + `<head>` (viewport, `<style>`) + `<body>` + `<script>`.
- Vanilla JS only. No CDNs, no imports, no fonts/icons fetched from the network. System font stack, inline SVG icons.

**Persistence**
- One `state` object as source of truth. `save()` writes it to `localStorage` under a **versioned key** (`ux-<tool>_v1`) inside `try/catch`.
- **Big blobs (images, files, generated canvases) → IndexedDB**, never localStorage. Strip blobs out of `state` before saving to localStorage; rehydrate them on `load()`.
- `load()` runs before first render; wrap in `try/catch`; handle missing/legacy keys. `await load()` in the boot IIFE.
- Call `save()` on every state change.

**Code structure (mirror image-slicer)**
- `state` object at top; `current*()` accessors; `ensureActive*()` after deletes so indices never go out of range.
- CRUD functions: `create*`/`add*`/`switch*`/`rename*`/`remove*`/`move*` — each calls `save()` + `render()`.
- One `render()` coordinator + focused `render*()` functions. Build DOM imperatively (`createElement` + `appendChild` + event listeners), not `innerHTML` strings.
- Stable IDs via a `nextId` counter in state.

**Design system (use exactly, from CLAUDE.md)**
- Backgrounds `#111` / panels `#141414 #161616 #1a1a1a` / borders `#222 #2a2a2a` / text `#e0e0e0`, secondary `#999 #777`, marginal `#555 #444`.
- Accents: primary `#3b82f6` (hover `#2563eb`), danger `#f43f5e`.
- Font: `-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`.
- Components: `.btn`/`.btn-primary`/`.btn-ghost`/`.btn-sm`, sidebar with hover-reveal actions + drag-to-reorder + inline rename, `.panel-header` + `.panel-body`, drop zone / empty state (dashed border), thin custom scrollbars, resizable sidebar persisted in state.

**Interactions — include the ones the spec calls for**
- Drag & drop (file upload + list reorder), paste (Ctrl+V), export (download via `<a download>` + copy via `navigator.clipboard` with "✓ Copied"/"✗ Failed" feedback), keyboard (Enter commit / Escape cancel in fields).

**States — build them in, not as afterthoughts**
- Empty state for every list/panel (what the user sees at first run, with the next action).
- At least one error path handled gracefully (storage full / save failed) — never a silent failure or a raw `alert()`.

## Writing the file

Implement the whole tool in `apps/<name>/index.html`. Cover every section of the Requirements: data model (correct storage split), all actions, all screens, the states, the interactions, the edge cases. If the spec is ambiguous on a detail, pick the option closest to `image-slicer`'s behavior and note the choice in one line to the user — don't block on minor unknowns.

When implementing a **Change entry** (from `ux-feature`/`ux-bug`): read the change's plan/diagnosis, apply exactly that to the existing `index.html`, respect the regression sites the bug entry flagged, and don't drift the rest of the app.

## Verification (do this, report results honestly)

1. Open the file: point the user to open `apps/<name>/index.html` directly in a browser (or note it'll be at `.../ux-tools/apps/<name>/` once deployed).
2. Self-check the code: storage split correct (no blobs in localStorage), `save()`/`load()` wired, empty state present, no `alert()`, no external network calls, palette matches CLAUDE.md, stable IDs.
3. If you can run a quick check (e.g. a node/html parse), do; otherwise state what you verified statically and what the user should click-test.
4. Report failures plainly — if something is unimplemented or uncertain, say so, don't claim it works.

## After writing

**Commit first** (Git checkpoint "After") — then the handoff.

Tell the user:
1. Where: `apps/<name>/index.html`
2. What's implemented (map to Requirements sections) and what's deferred
3. How to open it, and the one thing to click-test first (usually: add data → refresh → confirm it persisted)
4. Next step: "If you find a bug — **ux-bug** will diagnose and file a Change. If a new feature — **ux-feature** will plan it and route it to me."
