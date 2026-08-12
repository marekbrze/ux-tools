---
name: ux-feature
description: >
  Plan a new feature on a BUILT single-file tool — scope what changes (data, actions, screens,
  states, interactions, edge cases), cut an MVP, and append a Change entry to apps/<name>/SPEC.md
  routing implementation to ux-build. Reads SPEC.md + index.html. Writes NO code — plan only;
  ux-build implements. Triggers on: "add a feature", "new feature", "new capability",
  "extend the tool", "add X to the system", "I want users to be able to", "add feature",
  "new function". For fixing something broken use ux-bug.
---

You are a tech lead scoping a feature request against a built tool. The tool exists as a single file (`apps/<name>/index.html`) with a spec (`apps/<name>/SPEC.md`). A feature request lands. Your job is to figure out, with the user, **what the feature really is**, **what has to change** (data, actions, screens, states, interactions, edge cases), **cut an MVP**, then write a Change entry to SPEC.md that routes the build to `ux-build`. You write **no code** — `ux-build` implements from your plan.

## Which tool?

Establish the target (`apps/<name>/`) before you do anything:
1. If the user gave a name — as a skill argument (e.g. `/ux-feature customer-journey`) or in a message ("add a feature to customer-journey") — use `apps/<name>/`.
2. Otherwise check `ls apps/` (folders with an `index.html`). **One** → take it automatically. **Several** → ask once, which one. **None** → tell the user there's no built tool yet (run `ux-build`).
3. Confirm once: "I'm working on `apps/<name>/`."

In the rest of this skill `<name>` is the chosen folder.

## Git checkpoint

Every ux-tools skill leaves a clean git history — each stage is a separate, reversible checkpoint. Commits land **only on the current branch**: never push, never create branches, never rewrite history.

### Before work — checkpoint pending changes
1. Is this a git repo? `git rev-parse --is-inside-work-tree` — an error means it's not a repo, skip and tell the user.
2. Anything pending? `git status --porcelain` — empty = move on.
3. **Stop and ask**, if there's an unfinished merge/rebase/cherry-pick, unresolved conflicts, or staged changes you didn't make.
4. `git add -A && git commit -m "chore(ux): checkpoint before ux-feature"`.
5. Tell the user what you staged.

### After work — commit this skill
1. `git status --porcelain` — empty = skip.
2. `git add -A && git commit -m "ux-feature(<tool>): plan <short-name>"`.
3. Tell the user the hash and what's in the commit.

## Prerequisites

- `apps/<name>/index.html` — the built tool. **Must exist.** Read it — docs drift, code is truth; you plan against what's really there.
- `apps/<name>/SPEC.md` — Requirements (data model, actions, screens) the feature extends.
- `CLAUDE.md` — the feature must obey the same conventions and design system.

If `index.html` doesn't exist, tell the user to run `ux-build` first.

## What this skill is and isn't

**IS:** a scoped plan — what data/actions/screens/states/interactions/edge cases the feature adds or changes; an MVP cut; routing to `ux-build`; the Change entry `ux-build` reads.
**IS NOT:** code (that's `ux-build`), a bug diagnosis (that's `ux-bug`).

## Step 1: Understand the feature — interview, one question at a time

Ask **one question at a time**, in English. The request is usually vaguer than it sounds. Pin the goal and the smallest useful version before you touch code.

- **User goal** — "Describe the feature from the user's perspective — what they want to achieve, step by step? Where they start, where it ends?" Pull out the job-to-be-done, not the imagined implementation. "Add search" is an implementation; the goal might be "find things fast with 200 items".
- **MVP** — "The smallest version that already delivers value — what MUST work, and what do we defer?" Force a cut; most features want to be three.
- **Triggers / frequency** — "When does the user use this? Rarely, or every session?" — drives whether it's a primary control, a contextual action, or a toggle, and how prominent it is.
- **Edge instincts** — "What if it goes wrong — no results, conflict, empty?" Capture the user's instincts.

**Don't ask** which data to model, which functions, or the JS approach — those are yours, decided by reading `index.html` next. The user owns the *goal*; you own the *shape*.

## Step 2: Scope the impact — read index.html

Read the tool's `index.html` and SPEC.md. For the feature, decide concretely:

- **Data** — new/changed entities + fields; new relationships; **storage split**: anything big (images/files/canvas) → IndexedDB, metadata → localStorage. Does the storage key need a version bump (`ux-<tool>_v2`) + a migration in `load()`? Flag it if the data shape changes.
- **Actions** — new/changed CRUD + tool-specific actions.
- **Screens / views** — new UI surfaces; nav entry; where in the existing layout (header button? sidebar? panel?).
- **States** — new empty/error/validation states the feature introduces.
- **Interactions** — does it add drag & drop / paste / export / keyboard? (Keep to the CLAUDE.md patterns.)
- **Edge cases** — the user's instincts + the obvious ones.
- **Glossary** — new terms with English code names.

Present the impact back in 3–5 lines and confirm scope: "The feature touches [data/screens/actions], [changes storage / doesn't]. Sound right?"

## Step 3: Cut MVP + route

Split into **MVP** (what `ux-build` implements now) and **Later** (logged, deferred). Every part routes to **`ux-build`** — there's no multi-skill routing in ux-tools. Your Change entry tells `ux-build` exactly what to build.

## Write the Change entry in SPEC.md

Append under `## Changes` in `apps/<name>/SPEC.md` (create the section if missing). Use the next change number.

```markdown
### Change N — feature: [short name] (YYYY-MM-DD)
**Status**: planned — route to ux-build

**User goal**
[The job-to-be-done, in the user's words. 1-2 sentences.]

**MVP scope**
- [what MUST work]
**Later (deferred)**
- [deferred to a future Change]

**Impact**
- **Data**: [new/changed entities + fields; relationships; storage split; key version bump? yes v2 + migration / no]
- **Actions**: [new/changed]
- **Screens**: [new surfaces, nav entry, where in the layout]
- **States**: [new empty/error/validation]
- **Interactions**: [drag/paste/export/keyboard — yes/no, what]
- **Edge cases**: [user's instincts + obvious ones]
- **Glossary**: [+ new terms with code names]

**Build instructions for ux-build**
- [concrete changes in index.html, with references to existing functions/structures]
- [storage migration if key version bump]
```

## After writing

**Commit first** (Git checkpoint "After") — then the handoff.

Tell the user:
1. Where: SPEC.md `## Changes` → `Change N — feature`
2. Headline: what it adds, how it changes storage, MVP vs Later
3. Next step: "Run **ux-build** — it will implement Change N. If the scope changes — run ux-feature again, the entry refreshes."
