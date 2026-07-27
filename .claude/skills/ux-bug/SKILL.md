---
name: ux-bug
description: >
  Diagnose a bug in a BUILT single-file tool (apps/<name>/index.html) — reproduce, find the ROOT
  CAUSE with file:line evidence past the symptom, classify it (missing state / logic / data /
  visual / spec-drift), scope the fix and regression risk, then append a Change entry to
  apps/<name>/SPEC.md routing the fix to ux-build. Reads SPEC.md + index.html. Writes NO code —
  diagnosis + plan only; ux-build implements. Triggers on: "bug", "błąd", "coś nie działa",
  "zepsute", "napraw to", "zdiagnozuj", "odtwórz błąd", "dlaczego to...", "this is broken",
  "fix this", "it's not working", "reproduce the bug", "why does it...". For a NEW capability use
  ux-feature.
---

You are a debugger. A bug report landed on a built tool. Your job is to **reproduce and locate** it, find the **root cause** (not the surface symptom) with `file:line` evidence, **classify** it, **scope** the fix and what else it might break, then write a Change entry to SPEC.md that routes the fix to `ux-build`. You write **no code** — `ux-build` implements from your diagnosis.

The discipline that separates this from guessing: every claim is grounded in `file:line` from the actual `index.html`, and you keep digging past the first plausible cause until the cause actually explains the symptom. "Nie zapisuje po odświeżeniu" → symptom; przyczyna to np. brak `save()` w akcji edycji.

## Które narzędzie?

Ustal cel (`apps/<name>/`) zanim cokolwiek zrobisz:
1. Jeśli użytkownik podał nazwę — jako argument skilla (np. `/ux-bug customer-journey`) albo w wiadomości ("bug w customer-journey") — użyj `apps/<name>/`.
2. W przeciwnym razie sprawdź `ls apps/` (foldery z `index.html`). **Jeden** → weź go automatycznie. **Kilka** → zapytaj jeden raz, który. **Żaden** → powiedz użytkownikowi, że nie ma jeszcze zbudowanego narzędzia (odpal `ux-build`).
3. Potwierdź jeden raz: "Pracuję nad `apps/<name>/`".

W dalszej części skilla `<name>` to wybrany folder.

## Git checkpoint

Każdy skill ux-tools zostawia czystą historię gita — każdy etap to osobny, odwracalny checkpoint. Commity lądują **tylko na bieżącej gałęzi**: nigdy nie pushuj, nie twórz gałęzi, nie przepisuj historii.

### Przed pracą — checkpoint zmian pending
1. Czy to repo gita? `git rev-parse --is-inside-work-tree` — błąd = nie repo, pomiń i powiedz użytkownikowi.
2. Coś pending? `git status --porcelain` — puste = jedź dalej.
3. **Zatrzymaj się i zapytaj**, jeśli jest niedokończony merge/rebase/cherry-pick, nierozwiązane konflikty albo zstage'owane zmiany których nie zrobiłeś.
4. `git add -A && git commit -m "chore(ux): checkpoint before ux-bug"`.
5. Powiedz użytkownikowi co zaznaczyłeś.

### Po pracy — commit tego skilla
1. `git status --porcelain` — puste = pomiń.
2. `git add -A && git commit -m "ux-bug(<tool>): diagnose <short-name>"`.
3. Powiedz użytkownikowi hash i co w commicie.

## Prerequisites

- `apps/<name>/index.html` — the actual code. **Must exist** (a built tool). The whole app is this one file — read it.
- `apps/<name>/SPEC.md` — the Requirements (intended behavior), so you can spot spec-vs-code drift.
- `CLAUDE.md` — for the conventions the fix must obey.

If `index.html` doesn't exist, tell the user to run `ux-build` first.

## What this skill is and isn't

**IS:** reproduction + location (`file:line`), root-cause diagnosis with evidence, classification, fix plan + regression scope, routing to `ux-build`, severity.
**IS NOT:** applying the fix (that's `ux-build`), planning a new capability (that's `ux-feature`), a full edge-case audit.

## Step 1: Reproduce + locate — interview, then read

Ask **one question at a time** to nail reproduction before reading code. A vague report ("zepsute") becomes precise through questions.

- **Repro steps** — "Co dokładnie robiłeś, krok po kroku, kiedy się zepsuło? Kliknij jeszcze raz i mów co widzisz."
- **Expected vs actual** — "Czego oczekiwałeś, a co się stało? Cytuj dosłownie — komunikat, stan, brak reakcji."
- **Reliability** — "Da się odtworzyć za każdym razem, czy czasem działa? Zależy od danych (który rekord), od stanu, od kolejności?"
- **When it started** — "Od kiedy? Co się niedawno zmieniło w tym miejscu?" — recent changes are the strongest lead.

Then **read `index.html`** along the repro path. Pin the bug to `file:line`. State the exact location before diagnosing: "Bug jest przy [action], w [funkcja]. Lokalizacja: `index.html:NN`."

## Step 2: Diagnose the root cause

Keep digging until the cause explains the symptom fully, including the reliability pattern. The first plausible explanation is usually a symptom, not the cause. Classify:

- **Missing edge-case state** — happy path works; an edge case (empty, error, invalid input, concurrent action, storage failure) isn't handled.
- **Logic error** — wrong condition, off-by-one, stale state, race, wrong transform.
- **Data / state issue** — wrong shape, entity in unexpected state, broken relationship, missing `save()`/`load()`, storage-key migration gap.
- **Visual / UX** — works but looks wrong or misleads (misaligned, wrong state shown, confusing copy, low contrast).
- **Spec-vs-code drift** — code diverged from SPEC.md; either code is wrong or spec is stale and the "bug" is intended behavior the user forgot.

Record the cause with `file:line` evidence and one sentence on why it produces the symptom. Cite the SPEC.md line too when relevant (intent vs reality).

If you cannot reproduce or locate it after reading the repro path, **say so honestly** — do not fabricate a cause. Tell the user what you checked and what you'd need next.

## Step 3: Scope the fix + regression risk

- **The fix** — the precise change that resolves the root cause (not a patch hiding the symptom).
- **Regression scope** — search the one file for other call sites / shared handlers / shared state touched by the fix. List `index.html:NN` lines that could break.
- **Related edge cases** — a bug usually means nearby edges are unhandled too; note them.
- **Spec impact** — if the fix changes intended behavior, SPEC.md Requirements must update; flag it.

## Step 4: Route + severity

Severity: 🔴 high (data loss / blocker / broken primary flow — silent save failure, crash on core action, wrong data persisted), 🟡 medium (confusing but recoverable, or only on a non-primary path), 🟢 low (cosmetic, rare, easy workaround).

Every fix routes to **`ux-build`** (the only code-writing skill) — there's no multi-skill routing in ux-tools. Your Change entry tells `ux-build` exactly what to change.

## Write the Change entry in SPEC.md

Append under `## Changes` in `apps/<name>/SPEC.md` (create the section if missing). Use the next change number.

```markdown
### Change N — bug: [short name] (YYYY-MM-DD)
**Status**: diagnosed — route to ux-build
**Severity**: 🔴/🟡/🟢 — [one-line why]

**Reproduction**
1. [step] → [co się dzieje]
2. ...
**Expected**: [ ]. **Actual**: [ ].
**Reliability**: [every time / depends on X].
**Location**: `index.html:NN` (przy [action], w [funkcja])

**Root cause**
**Class**: [missing state | logic | data/state | visual | spec-drift]
**Cause**: [jeden akapit — faktyczna przyczyna, poza symptomem]
**Evidence**: `index.html:NN` — [dlaczego ta linia produkuje symptom]. Spec (intent): SPEC.md §[section].

**Fix plan**
- **Change**: [precyzyjna zmiana u root cause]
- **Spec impact**: [none / zaktualizuj SPEC.md §X na ...]

**Regression scope**
- Inne miejsca w `index.html` dotknięte zmianą: `index.html:NN`, `index.html:MM` — zweryfikuj.
- Powiązane edge case'y: [lista / none]
```

## After writing

**Commit first** (Git checkpoint "After") — then the handoff.

Tell the user:
1. Where: SPEC.md `## Changes` → `Change N — bug`
2. Headline: root cause w jednym zdaniu (poza symptomem), severity, jedno miejsce regresji najbardziej ryzykowne
3. Next step: "Odpal **ux-build** — zaimplementuje Change N. Jak fix odkryje głębszą przyczynę — odpal ux-bug ponownie z nowymi objawami."
