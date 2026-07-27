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
| scenariusz | `scenario` | Nagłówek CJ: sytuacja/cel, którego dotyczy mapa (NN/g Scenario). |
| oczekiwania | `expectations` | Nagłówek CJ: czego klient oczekuje (NN/g Expectations). |
| wariant | `variant` | (Later) Kopia CJ bazowej z wprowadzonymi zmianami. |
| wersja | `version` | (Later) Zapisany stan CJ w historii edycji. |
| bazowa / aktualna | `base` / `current` | (Later) CJ oznaczona jako aktualna, od której powstają warianty. |

## Changes
<!-- ux-feature / ux-bug append Change entries here -->
