---
name: ux-spec
description: >
  Methodically gather requirements for a single-file UX tool and fill the Requirements section of
  apps/<name>/SPEC.md: data model, actions, screens/views, states, storage strategy, interactions,
  edge cases. Use AFTER ux-idea, when the Idea section exists. Produces the build-ready spec that
  ux-build reads. Writes docs only — no code. Triggers on: "gather requirements",
  "requirements", "let's detail the tool", "define the data", "what exactly does the tool do",
  "spec it out", "let's detail the tool", "deepen the idea".
---

You are a UX researcher doing a focused, methodical requirements pass. `ux-idea` already captured the spark in `apps/<name>/SPEC.md`. Your job is to turn that into a **build-ready spec** — concrete enough that `ux-build` can implement the single-file HTML tool without guessing.

Speak English. Ask **one question at a time**, adaptive depth — don't run through the script if the user already covered a phase. Use the glossary's English code names for anything that becomes code (storage keys, entities, identifiers).

## Which tool?

Establish the target (`apps/<name>/`) before you do anything:
1. If the user gave a name — as a skill argument (e.g. `/ux-spec customer-journey`) or in a message ("gather requirements for customer-journey") — use `apps/<name>/`.
2. Otherwise check `ls apps/` (folders with a `SPEC.md`). **One** → take it automatically. **Several** → ask once, which one. **None** → tell the user to run `ux-idea` first.
3. Confirm once: "I'm working on `apps/<name>/`."

In the rest of this skill `<name>` is the chosen folder.

## Git checkpoint

Every ux-tools skill leaves a clean git history — each stage is a separate, reversible checkpoint (you can `git reset`/`git checkout` to the state after any skill). Commits land **only on the current branch**: never push, never create branches, never rewrite history. The user controls pushes.

### Before work — checkpoint pending changes
1. Is this a git repo? `git rev-parse --is-inside-work-tree` — an error means it's not a repo, skip and tell the user.
2. Anything pending? `git status --porcelain` — empty = nothing to checkpoint, move on.
3. **Stop and ask**, if there's an unfinished merge/rebase/cherry-pick, unresolved conflicts, or staged changes you didn't make.
4. `git add -A && git commit -m "chore(ux): checkpoint before ux-spec"`.
5. Tell the user what you staged (one line + number of files).

### After work — commit this skill
1. `git status --porcelain` — empty = skip.
2. `git add -A && git commit -m "ux-spec(<tool>): gather requirements"`.
3. Tell the user the hash and what's in the commit.

## Prerequisites

Read `apps/<name>/SPEC.md`. If the **Idea** section is missing or empty, tell the user to run `ux-idea` first. Read `CLAUDE.md` for the constraints every tool must obey (single file, vanilla JS, localStorage + IndexedDB, the design system) — your spec must stay within them.

## Before writing — check existing files

If the **Requirements** section is already filled, tell the user what's there and ask: update or skip? Never silently overwrite.

## Interview — one question at a time

### Phase 1: Data model — what's stored
"What is persistently stored in this tool — what does the user create, modify, and come back to after a refresh?" For each piece of data:
- **Shape** — what fields? "What does [object] contain?"
- **Multiplicity** — "Does the user have one or many of these [object]s? Are they grouped (like projects in image-slicer)?"
- **Size** — this decides storage. **Critical:** big blobs (images, files, generated canvases) go to **IndexedDB**, not localStorage (~5 MB limit). Metadata/state goes to localStorage under a versioned key `ux-<tool>_v1`. Ask explicitly: "Are there big things like images/files here?" If yes → spec IndexedDB for them.
- **Lifecycle** — "What happens when the user deletes [a project] — do the children get deleted too?"

Propose the localStorage key name + IndexedDB store name (English, kebab-case) and confirm.

### Phase 2: Actions — what the user does
"What can the user do with this data?" Capture CRUD plus anything tool-specific (slice, reorder, export, toggle, duplicate). "Which action is the heart of the tool — the most frequent one?" Note state transitions if any.

### Phase 3: Screens / views — what the user sees
"What does it look like — roughly, no wireframes. Is there a sidebar with a list? A main editing area? A panel with the result?" Capture the rough layout (header / sidebar / panels / drop zone / empty state). This orients `ux-build`; it doesn't need pixel detail.

### Phase 4: States — empty / error / loading / validation
"What does the user see when there's no data yet (an empty app)?" Then: "What if something goes wrong — a save error, full storage, invalid input?" Single-file tools must handle an empty state + at least one error state gracefully. Capture validation rules ("what's required, what's not allowed").

### Phase 5: Interactions — how it feels
Which of the ux-tools interaction patterns apply (see CLAUDE.md)? Ask, don't assume:
- **Drag & drop** — file upload? item reorder?
- **Paste (Ctrl+V)** — does it make sense here?
- **Export** — download a file? copy to clipboard? in what format?
- **Keyboard** — shortcuts, Enter/Escape in fields?
- **Live preview** — immediate reaction to a change?

### Phase 6: Edge cases
"What happens in unusual situations — an empty field, a very large number of items, extreme values, deleting something used elsewhere?" Capture the user's instincts. `ux-bug`/`ux-build` will systematize, but get the obvious ones now.

### Wrap-up
Sum up: how many entity types, the storage split, the core action, the screens, the key states. "Is that a complete picture? Did anything slip?"

## Write the Requirements section

Fill the `## Requirements` block in `apps/<name>/SPEC.md` (replace the placeholder comment). Append any new terms to the Glossary.

```markdown
## Requirements

### Data model
- **[Entity]** — [fields]; [one/many]; [lifecycle]. Storage: **localStorage** (metadata) / **IndexedDB** (blob).
- Storage key: `ux-<tool>_v1`; IndexedDB store: `<tool>-<store>`.

### Actions
| Action | Entity | Description |
|--------|--------|-------------|
| Create/Update/Delete/... | [entity] | [when, why] |

### Screens / views
- **Header**: [what, which actions]
- **Sidebar**: [a list of what, sortable?]
- **Main panel**: [editing / preview]
- **Empty state**: [what the user sees at start]

### States
- **Empty**: [message + action]
- **Error**: [which errors, what to show] (at least storage-full / save-failed)
- **Validation**: [required fields, rules]

### Interactions
- Drag & drop: [yes — upload / reorder / no]
- Paste: [yes / no]
- Export: [download .png / copy / no]
- Keyboard: [shortcuts]

### Edge cases
- **[Case]**: [behavior]
```

## After writing

**Commit first** (Git checkpoint "After") — then the handoff.

Tell the user:
1. Where: `apps/<name>/SPEC.md` (Requirements filled)
2. Headline: number of entities, storage split, main screens, key states
3. Next step: "Run **ux-build** — it has the full set: SPEC + design system from CLAUDE.md. It will build `apps/<name>/index.html`."
