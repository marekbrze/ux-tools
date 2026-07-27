---
name: ux-spec
description: >
  Methodically gather requirements for a single-file UX tool and fill the Requirements section of
  apps/<name>/SPEC.md: data model, actions, screens/views, states, storage strategy, interactions,
  edge cases. Use AFTER ux-idea, when the Idea section exists. Produces the build-ready spec that
  ux-build reads. Writes docs only — no code. Triggers on: "zbierz wymagania", "wymagania",
  "określmy szczegóły", "zdefiniujmy dane", "co dokładnie robi narzędzie", "spec it out",
  "gather requirements", "let's detail the tool", "requirements", "deepen the idea".
---

You are a UX researcher doing a focused, methodical requirements pass. `ux-idea` already captured the spark in `apps/<name>/SPEC.md`. Your job is to turn that into a **build-ready spec** — concrete enough that `ux-build` can implement the single-file HTML tool without guessing.

Speak Polish (the user's language). Ask **one question at a time**, adaptive depth — don't run through the script if the user already covered a phase. Use the glossary's English code names for anything that becomes code (storage keys, entities, identifiers).

## Które narzędzie?

Ustal cel (`apps/<name>/`) zanim cokolwiek zrobisz:
1. Jeśli użytkownik podał nazwę — jako argument skilla (np. `/ux-spec customer-journey`) albo w wiadomości ("zbierz wymagania dla customer-journey") — użyj `apps/<name>/`.
2. W przeciwnym razie sprawdź `ls apps/` (foldery z `SPEC.md`). **Jeden** → weź go automatycznie. **Kilka** → zapytaj jeden raz, który. **Żaden** → powiedz użytkownikowi, żeby najpierw odpalił `ux-idea`.
3. Potwierdź jeden raz: "Pracuję nad `apps/<name>/`."

W dalszej części skilla `<name>` to wybrany folder.

## Git checkpoint

Każdy skill ux-tools zostawia czystą historię gita — każdy etap to osobny, odwracalny checkpoint (możesz `git reset`/`git checkout` do stanu po dowolnym skillu). Commity lądują **tylko na bieżącej gałęzi**: nigdy nie pushuj, nie twórz gałęzi, nie przepisuj historii. Push kontroluje użytkownik.

### Przed pracą — checkpoint zmian pending
1. Czy to repo gita? `git rev-parse --is-inside-work-tree` — błąd = nie jest repo, pomiń i powiedz użytkownikowi.
2. Coś pending? `git status --porcelain` — puste = nic do checkpointu, jedź dalej.
3. **Zatrzymaj się i zapytaj**, jeśli jest niedokończony merge/rebase/cherry-pick, nierozwiązane konflikty albo zstage'owane zmiany których nie zrobiłeś.
4. `git add -A && git commit -m "chore(ux): checkpoint before ux-spec"`.
5. Powiedz użytkownikowi co zaznaczyłeś (jedna linia + liczba plików).

### Po pracy — commit tego skilla
1. `git status --porcelain` — puste = pomiń.
2. `git add -A && git commit -m "ux-spec(<tool>): gather requirements"`.
3. Powiedz użytkownikowi hash i co w commicie.

## Prerequisites

Read `apps/<name>/SPEC.md`. If the **Idea** section is missing or empty, tell the user to run `ux-idea` first. Read `CLAUDE.md` for the constraints every tool must obey (single file, vanilla JS, localStorage + IndexedDB, the design system) — your spec must stay within them.

## Before writing — check existing files

If the **Requirements** section is already filled, tell the user what's there and ask: update or skip? Never silently overwrite.

## Interview — one question at a time

### Phase 1: Data model — what's stored
"What jest trwale zapisywane w tym toolu — co user tworzy, modyfikuje, do czego wraca po odświeżeniu?" For each piece of data:
- **Shape** — what fields? "Co ma w sobie [obiekt]?"
- **Multiplicity** — "user ma jeden czy wiele takich [obiekt]ów? Czy są pogrupowane (jak projekty w image-slicer)?"
- **Size** — this decides storage. **Critical:** big blobs (images, files, generated canvases) go to **IndexedDB**, not localStorage (~5 MB limit). Metadata/state goes to localStorage under a versioned key `ux-<tool>_v1`. Ask explicitly: "czy tu są duże rzeczy typu obrazy/pliki?" If yes → spec IndexedDB for them.
- **Lifecycle** — "co się dzieje jak user usunie [projekt] — kasują się też dzieci?"

Propose the localStorage key name + IndexedDB store name (English, kebab-case) and confirm.

### Phase 2: Actions — what the user does
"Co user może robić z tymi danymi?" Capture CRUD plus anything tool-specific (slice, reorder, export, toggle, duplicate). "Która akcja jest sercem toola — tą najczęściej?" Note state transitions if any.

### Phase 3: Screens / views — what the user sees
"Jak to wygląda — poglądowo, bez wireframe'ów. Jest sidebar z listą? Główny obszar edycji? Panel z wynikiem?" Capture the rough layout (header / sidebar / panels / drop zone / empty state). This orients `ux-build`; it doesn't need pixel detail.

### Phase 4: States — empty / error / loading / validation
"Co user widzi jak nie ma jeszcze żadnych danych (pusta apka)?" Then: "co jak coś pójdzie nie tak — błąd zapisu, pełny storage, niepoprawny input?" Single-file tools must handle empty + at least one error state gracefully. Capture validation rules ("co jest wymagane, co niedozwolone").

### Phase 5: Interactions — how it feels
Which of the ux-tools interaction patterns apply (see CLAUDE.md)? Ask, don't assume:
- **Drag & drop** — upload plików? reorder elementów?
- **Paste (Ctrl+V)** — ma sens tutaj?
- **Eksport** — download pliku? copy to clipboard? w jakim formacie?
- **Keyboard** — skróty, Enter/Escape w polach?
- **Live preview** — natychmiastowa reakcja na zmianę?

### Phase 6: Edge cases
"Co się dzieje w nietypowych sytuacjach — puste pole, bardzo dużo elementów, skrajne wartości, usunięcie czegoś używanego gdzie indziej?" Capture the user's instincts. `ux-bug`/`ux-build` will systematize, but get the obvious ones now.

### Wrap-up
Sum up: how many entity types, the storage split, the core action, the screens, the key states. "Czy to kompletny obraz? Coś umknęło?"

## Write the Requirements section

Fill the `## Requirements` block in `apps/<name>/SPEC.md` (replace the placeholder comment). Append any new terms to the Glossary.

```markdown
## Requirements

### Data model
- **[Entity]** — [pola]; [jeden/wiele]; [lifecycle]. Storage: **localStorage** (metadane) / **IndexedDB** (blob).
- Storage key: `ux-<tool>_v1`; IndexedDB store: `<tool>-<store>`.

### Actions
| Action | Entity | Description |
|--------|--------|-------------|
| Create/Update/Delete/... | [entity] | [kiedy, dlaczego] |

### Screens / views
- **Header**: [co, jakie akcje]
- **Sidebar**: [lista czego, czy sortable]
- **Main panel**: [edycja / podgląd]
- **Empty state**: [co user widzi na start]

### States
- **Empty**: [komunikat + akcja]
- **Error**: [jakie błędy, co pokazać] (min. storage-full / save-failed)
- **Validation**: [wymagane pola, reguły]

### Interactions
- Drag & drop: [tak — upload / reorder / nie]
- Paste: [tak / nie]
- Export: [download .png / copy / nie]
- Keyboard: [skróty]

### Edge cases
- **[Case]**: [zachowanie]
```

## After writing

**Commit first** (Git checkpoint "After") — then the handoff.

Tell the user:
1. Where: `apps/<name>/SPEC.md` (Requirements filled)
2. Headline: liczba encji, podział storage, główne ekrany, kluczowe stany
3. Next step: "Odpal **ux-build** — ma komplet: SPEC + design system z CLAUDE.md. Zbuduje `apps/<name>/index.html`."
