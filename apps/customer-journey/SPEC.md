# Customer Journey

> Structured customer journey maps — a grid of steps × slices, grouped into stages — replacing the freeform Axure canvas.

## Idea

### Core concept
Single-file narzędzie do tworzenia i utrzymywania **map podróży klienta (customer journeys)** jako **ustrukturyzowanej siatki**: **kolumny = kroki** procesu (grupowane w **etapy**), **wiersze = slice'y** (7 stałych wymiarów: działania, touchpointy, myśli, emocje, pain points, pomysły, insighty). CJ są pogrupowane po **kategoriach procesów** i powiązane z **personami**. Siatka sama się układa — koniec z ręcznym przepychaniem elementów po wolnym płótnie Axure.

### User problem
- **Ręczna praca w Axure**: CJ dziś żyją na wolnym płótnie → dużo czasu idzie na ogarnianie przestrzeni i ręczne dodawanie/przesuwanie elementów (kroków, entrypointów, pushy) zamiast na treść. Zmiana wymaga przepychania wszystkiego.
- **Trudna prezentacja zmiany**: pokazanie, jak CJ zmieni się po wprowadzeniu zmiany, jest dzisiaj żmudne i robione ręcznie.

### Target user
UX designer (autor) pracujący nad podróżami klienta w procesach finansowych (np. cash loan, account opening). Jedna osoba autorska; narzędzie osobiste, dane lokalnie.

### Core action
Otworzyć customer journey → edytować **siatkę kroków × slice'ów** (dodawać/przesuwać kroki i etapy, zarządzać niezależnymi elementami w komórkach) → mieć to od razu uporządkowane i gotowe do pokazania (eksport PNG).

### Scope
- **MVP**:
  - Kategorie procesów (cash loan, account opening…) z licznikiem CJ + baza person.
  - Lista CJ w kategorii (osoba + krótki opis przypadku).
  - **Edytor CJ jako siatka**: etapy grupujące kroki (kolumny — dodawanie, przenoszenie, rename, usuwanie), 7 stałych slice'ów (wiersze); komórki z wieloma niezależnymi elementami tekstowymi (drag kolejności, usuwanie osobno).
  - **Eksport całej CJ jako PNG**.
- **Later** (przyszłe Change'y):
  - Warianty: kopia CJ → wariant; CJ bazowa oznaczona jako „aktualna"; wariant = bazowa + zmiany.
  - Wersjonowanie historii edycji CJ.
  - Prezentacja zmiany (side-by-side przed/po / diff) — format dostarczenia do doprecyzowania.
  - Obrazki/screenshoty w komórkach (→ IndexedDB na blob-y).

## Requirements

### Data model
- **`category`** — proces biznesowy (np. cash loan, account opening); `name`. Wiele. Nadrzędna dla `journey`.
- **`persona`** — postać z bazy person; `name`. Wiele, współdzielone między CJ. CRUD.
- **`journey`** (customer journey) — `title`, `description` (scenariusz), `expectations`, `personaId`; należy do `category`. Wiele per kategoria.
- **`stage`** (etap) — `name`; uporządkowana lista w ramach `journey`; grupuje `step`. Np. Formularz, Obsługa po przesłaniu. (NN/g „Journey Phase".)
- **`step`** (krok) — `name`; należy do `stage`; uporządkowana. Kolumna siatki.
- **`slice`** — jeden z 7 stałych wierszy (globalnie, nie per CJ): `action`, `touchpoint`, `mindset`, `emotion`, `pain-point`, `idea`, `insight`.
- **`cell`** — przecięcie (`step`, `slice`); zawiera uporządkowaną listę `element`.
- **`element`** — pojedynczy, niezależny wpis tekstowy w `cell` (`text`); kolejność zarządzana, usuwany osobno. Tylko tekst w MVP.
- **Lifecycle**: usunięcie `category` → kaskada (journeys → stages → steps → cells → elements). Usunięcie `persona` używanej przez `journey` → zapytaj lub null + komunikat.
- **Storage**: metadane/stan → `localStorage` pod `ux-customer-journey_v1` (jeden obiekt `state`, `save()`/`load()` w `try/catch`, migracje). Brak blob-ów w MVP → bez IndexedDB. (Obrazki w komórkach = Later → wtedy IndexedDB.)

### Actions
| Action | Entity | Description |
|--------|--------|-------------|
| Create / Rename / Delete / Reorder | category | zarządzanie kategoriami procesów |
| Create / Rename / Delete | persona | baza person (współdzielona) |
| Create / Rename / Delete / Switch | journey | CJ w kategorii; ustaw personę + opis + oczekiwania |
| Add / Rename / Delete / Reorder | stage | etapy w CJ |
| Add / Rename / Delete / Reorder | step | kroki w i między etapami |
| Add / Reorder / Delete | element | niezależne elementy w komórce (drag kolejności) |
| Set sentiment | emotion cell | wartość emocji per krok (krzywa emocjonalna) |
| Export PNG | journey | render siatki → PNG (vanilla: SVG/Canvas → toBlob) |

**Rdzeń**: edycja siatki — dodawanie kroków/etapów i zarządzanie elementami w komórkach. `save()` przy każdej zmianie.

### Screens / views
- **Header**: tytuł toola + akcje „Nowa CJ" i „Eksport PNG" (aktywnej CJ).
- **Sidebar**: lista **kategorii** (z licznikiem CJ) + sekcja **person** (baza) + lista **CJ w aktywnej kategorii** (persona + krótki opis). Drag-to-reorder, hover-akcje (✎/×), inline-rename — jak w image-slicer.
- **Main panel = siatka CJ**: pasek **etapów** rozpięty nad kolumnami **kroków**; 7 wierszy **slice'ów**; komórki edytowalne w miejscu (dodaj element, drag kolejności, usuń). Wiersz `emotion` = wartość sentymentu (krzywa). Zamrożona kolumna z etykietami slice'ów; przewijanie poziome przy wielu krokach.
- **Empty state**: brak kategorii/CJ → instrukcja „Dodaj kategorię i pierwszą CJ".

### States
- **Empty**: pusta apka (brak kategorii) oraz pusta CJ (brak kroków) → komunikat + pierwsza akcja.
- **Error**: pełny `localStorage` / błąd zapisu → grzeczny komunikat (nie `alert()`); `save()` w `try/catch`.
- **Validation**: lekka — nazwy wymagane przy tworzeniu, auto-nazwa przy pustym (np. „Krok 1", „Etap 1") jak w image-slicer.

### Interactions
- **Drag & drop**: reorder kroków (w i między etapami), reorder etapów, reorder elementów w komórce.
- **Paste (Ctrl+V)**: tekst do aktywnej komórki — opcjonalne.
- **Eksport**: aktywna CJ → PNG (download przez `<a download>`; opcjonalnie copy do schowka z feedbackiem „✓ Copied" / „✗ Failed").
- **Keyboard**: Enter = zapisz inline-edycję, Escape = anuluj.
- **Live preview**: zmiany w siatce od razu widoczne.

### Edge cases
- Usunięcie kategorii → kaskada na całą zawartość.
- Usunięcie persony używanej przez CJ → zapytaj lub null + oznacz.
- Pusta CJ bez kroków → empty state siatki.
- Bardzo dużo kroków → przewijanie poziome (zamrożona kolumna slice-labels).
- Pełny `localStorage` → komunikat, nie cicha utrata.
- Aktywny indeks po usunięciu → `ensureActiveIndicesValid()` (kategoria/CJ/etap/krok nigdy poza zakresem).

## Glossary
| Term (PL) | Code name (EN) | Definition |
|-----------|----------------|------------|
| kategoria procesu | `category` | Grupa procesów biznesowych (np. cash loan, account opening) grupująca customer journeys. |
| persona | `persona` | Postać użytkownika, której dotyczy dany customer journey (baza współdzielona). |
| customer journey | `journey` | Mapa podróży klienta przez dany proces — siatka etapów/kroków × slice'ów. |
| etap | `stage` | Poziomy pas grupujący kroki (NN/g „Journey Phase"); np. Formularz, Obsługa po przesłaniu. |
| krok | `step` | Pojedynczy etap granularny — **kolumna** w siatce, należy do `stage`. |
| slice (wymiar) | `slice` | Wiersz w siatce CJ — jeden z 7 stałych pasów: `action`, `touchpoint`, `mindset`, `emotion`, `pain-point`, `idea`, `insight`. |
| działanie klienta | `action` | Slice: co klient robi w kroku. |
| myśli / pytania | `mindset` | Slice: co klient myśli, czego szuka, motywacje. |
| emocja | `emotion` | Slice: wartość sentymentu w kroku (skala) → krzywa emocjonalna. |
| pain point | `pain-point` | Slice: trudność/frustracja w kroku. |
| pomysł | `idea` | Slice: propozycja ulepszenia dla kroku (opportunity). |
| insight | `insight` | Slice: wniosek/badawcza obserwacja dla kroku. |
| touchpoint | `touchpoint` | Slice: punkt kontaktu klienta (entrypoint, push, kanał). |
| entrypoint | `entry-point` | Touchpoint: wejście klienta do kroku/kanału. |
| push | `push` | Touchpoint: powiadomienie push wychodzące do klienta. |
| komórka | `cell` | Lista elementów w przecięciu kroku (kolumna) i slice'a (wiersz). |
| element | `element` | Pojedynczy niezależny wpis tekstowy w komórce (może być wiele; kolejność zarządzana, usuwany osobno). |
| motyw | `theme` | Motyw UI: `light` (domyślnie), `dark`, `auto` (podąża za `prefers-color-scheme`); persystowany w `state.theme`. |
| scenariusz | `scenario` | Nagłówek CJ: sytuacja/cel, którego dotyczy mapa (NN/g Scenario). |
| oczekiwania | `expectations` | Nagłówek CJ: czego klient oczekuje (NN/g Expectations). |
| wariant | `variant` | (Later) Kopia CJ bazowej z wprowadzonymi zmianami. |
| wersja | `version` | (Later) Zapisany stan CJ w historii edycji. |
| bazowa / aktualna | `base` / `current` | (Later) CJ oznaczona jako aktualna, od której powstają warianty. |

## Changes

### Change 1 — feature: english UI + accessibility + light/dark theme (2026-07-27)
**Status**: planned — route to ux-build

**User goal**
Narzędzie ma być **po angielsku** (użycie profesjonalne), **dostępne** (a11y) i **jaśniejsze** — dotychczasowy motyw dark był dla użytkownika zbyt ciemny. Light jako domyślny, dark jako opcja.

**MVP scope**
- Cały UI po angielsku + `<html lang="en">`.
- **Light theme domyślnie + dark jako opcja**: przełącznik w headerze; domyślnie podąża za `prefers-color-scheme`; wybór persystowany w `state.theme`.
- **Aktualizacja design systemu w `CLAUDE.md` (sekcja 2)** na dual palette (light domyślnie + dark) z tokenami jako CSS custom properties — żeby przyszłe apki dziedziczyły light.
- **Baseline a11y**: semantyczne landmarki (`header`/`main`/`nav`), ikony-akcje jako `<button>` z `aria-label`, ARIA siatki (`role=grid`/`row`/`columnheader`/`rowheader`/`gridcell`), każdy kontrolka osiągalna i operacyjna z klawiatury, **widoczny focus ring**, kontrast interaktywnych ≥ WCAG AA.

**Later (deferred)**
- Pełna nawigacja strzałkami po komórkach siatki.
- Ogłaszanie operacji drag&drop dla czytników ekranu (live regions).

**Impact**
- **Data**: + `state.theme` (`'light' | 'dark' | 'auto'`, default `'auto'`). Pole addytywne — **bez key version bump** (`load()` przez `Object.assign` zostawi default przy braku pola w starych danych). Brak blob-ów → localStorage.
- **Actions**: `toggleTheme()` / `setTheme(v)` → `save()` + zastosuj `data-theme` na `<html>` (+ nasłuch `prefers-color-scheme` gdy `'auto'`).
- **Screens**: przycisk przełącznika motywu w headerze (☀ / ☾).
- **States**: brak nowych; edge: brak zapisanego theme → `'auto'` → resolve przez media query; zmiana system theme na żywo w trybie auto.
- **Interactions**: keyboard (toggle, focus mgmt); media query `prefers-color-scheme`.
- **Edge cases**: stary stan bez `theme` → default; kontrast w obu motywach ≥ AA.
- **Glossary**: `theme` (`light` / `dark` / `auto`).

**Build instructions for ux-build**
1. **`CLAUDE.md` sekcja 2**: zastąp pojedynczą dark paletę **dualnym systemem** — tokeny jako CSS custom properties; wariant light (domyślny na `:root`) i dark (przez `[data-theme="dark"]`, plus `@media (prefers-color-scheme: dark)` gdy `data-theme="auto"`). Light palette: bg `#f7f8fa`, panele `#ffffff #f1f3f5`, granice `#e5e7eb #d0d7de`, tekst `#1f2328`, wtórny `#57606a`, marginalny `#8c959f`, akcent primary `#2563eb` (hover `#1d4ed8`), danger `#d1242f`. Zachowaj dotychczasowe wartości dark jako wariant dark. Zaktualizuj opisy komponentów (btn, sidebar, panel, drop zone, scrollbar), żeby używały tokenów.
2. **`index.html`**: wprowadź ten sam system tokenów (CSS vars na `:root` + `[data-theme="dark"]`); `<html lang="en">`; przycisk przełącznika w headerze wołający `toggleTheme()`; boot: zastosuj `state.theme` jako `data-theme` na `<html>` + nasłuch `prefers-color-scheme` dla `'auto'`.
3. **Angielski (wszystkie stringi)**: header hint, sidebar section headers → *Categories / Personas / Customer journeys*, tooltipy ✎/×/⠿ → *Rename / Remove / Drag*, journey meta → *Title / Persona / Scenario (description) / Expectations*, `+ Etap` → *+ Stage*, `+ krok` → *+ Step*, `+ element` → *+ element*, empty states, export → *Export PNG / ✓ Copied / ✗ Failed*, licznik → *N steps*. Slice labels → **Actions, Touchpoints, Mindset, Emotions, Pain points, Ideas, Insights**. Seeded example: kategoria *Cash loan*, persona *Anna*, CJ *Cash loan application*, etap *Stage 1*, krok *Step 1*.
4. **a11y**: `<html lang="en">`; semantyczne `header`/`main`/`nav`; ikony-akcje (⠿ ✎ × + ☀/☾) jako `<button aria-label="…">`; ARIA w siatce: `#grid` → `role="grid"`, wiersze → `role="row"`, nagłówki kroków → `role="columnheader"`, etykiety slice'ów → `role="rowheader"`, komórki danych → `role="gridcell"`; widoczny `:focus-visible` ring na wszystkich interaktywnych; zachować Enter/Escape w inline-edycji.
5. **Glossary w SPEC.md**: dopisz `theme` (`light`/`dark`/`auto`) jeśli go nie ma.

### Change 2 — bug: low text contrast, esp. dark mode (2026-07-27)
**Status**: diagnosed — route to ux-build
**Severity**: 🟡 medium — treść główna (`--text`, `--text-2`) czytelna; niedoczytelne etykiety/listy/chipy/empty states. Bloker a11y dla WCAG AA.

**Reproduction**
1. Otwórz appkę, przełącz na dark (☀ → ☾).
2. Czytaj: nagłówki sekcji sidebar (CATEGORIES, PERSONAS…), panel-header, field-labels w journey-meta, empty-state hinty, slice labels, tekst list CJ, liczniki (chip).
**Expected**: cały tekst ≥ 4.5:1 (WCAG AA). **Actual**: tekst wtórny/marginalny 2.4–4.1:1 — ciężko czytać; light mode też 2.7–4.3:1 na etykietach.
**Reliability**: za każdym razem, w obu motywach; dark gorzej.
**Location**: definicje tokenów — `index.html:18-20,29` (light `:root`) i `index.html:47-49,58` (dark `[data-theme="dark"]`).

**Root cause**
**Class**: visual / a11y
**Cause**: szare tokeny dla tekstu wtórnego/marginalnego (`--text-3`, `--text-4`) oraz `--chip-text` są zbyt blisko luminancji swoich tłów — poniżej WCAG AA 4.5:1. Najgorzej dark `--text-4 #555` = 2.4:1 na panelach (nagłówki sekcji, field-labels, empty states prawie nieczytelne). `--text-3 #777` ≈ 4.0:1, `--chip-text #888` = 4.05:1. Light `--text-4 #8c959f` ≈ 2.7–3.0:1, `--text-3 #6e7781` ≈ 4.1:1. Tekst główny i elementów (`--text`, `--text-2`) przechodzi.
**Evidence** (zmierzone WGAG): dark `#555` on `#141414` = 2.47, on `#111` = 2.53; dark `#777` on `#141414` = 4.11; dark `#888`(chip) on `#2a2a2a` = 4.05; light `#8c959f` on `#fff` = 3.04, on `#f1f3f5` = 2.73; light `#6e7781` on `#f1f3f5` = 4.09. Spec intent: SPEC.md §Requirements→States („kontrast ≥ WCAG AA") + CLAUDE.md §2.1 („Kontrast ≥ WCAG AA w obu motywach").

**Fix plan**
- **Change**: podbij tokeny do wartości AA (zweryfikowane ≥4.5:1 na realnych tłach):
  - Dark (`[data-theme="dark"]`, `index.html:47-49,58`): `--text-2: #b9bfc6` (z #999), `--text-3: #9aa0a6` (z #777), `--text-4: #8a9098` (z #555), `--chip-text: #a8aeb6` (z #888).
  - Light (`:root`, `index.html:18-20`): `--text-2: #4b5563` (z #57606a), `--text-3: #5c6670` (z #6e7781), `--text-4: #626a73` (z #8c959f); `--chip-text` zostaw `#57606a` (5.4:1 OK).
  - `--text` bez zmian (`#e0e0e0` / `#1f2328` — już AA).
- **Spec impact**: none — bez zmiany zachowania; CLAUDE.md §2.1 już nakazuje AA, to doprowadza kod do zgodności.

**Regression scope**
- Zmiana tylko tokenów; dotyka każdej powierzchni używającej tych tokenów (nagłówki sekcji, panel-header, field-labels, slice labels, tekst list, chipy, empty states, hinty) — wszystkie staną się bardziej czytelne, brak regresji funkcjonalnej.
- Zaktualizować też tabelę tokenów w CLAUDE.md §2.1 (kolumny light/dark dla `--text-2/3/4`), żeby design-system-doc zgadzał się z kodem i przyszłe apki dziedziczyły tokeny AA.

**Build instructions for ux-build**
- W `index.html`: podmień 4 wartości dark (L47-49, L58) i 3 wartości light (L18-20) na wartości AA powyżej.
- W `CLAUDE.md` §2.1 tabela: zaktualizuj wiersze `--text-2/3/4` (kolumny light + dark) do nowych wartości.
- Po zmianie: prze-weryfikuj kontrast ≥4.5:1 dla realnych par token×tło. (text-4 na panel-3/chip dotyczy tylko dekoracyjnych drag-handle `aria-hidden` — zwolnione; dla pełnego marginesu text-4 → #5c6670.)
