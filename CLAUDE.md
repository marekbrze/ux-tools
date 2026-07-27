# ux-tools

Zbiór małych, samodzielnych narzędzi dla UX designera. Każde narzędzie to **pojedynczy plik HTML** działający w przeglądarce, **offline**, bez backendu, zapisujący dane lokalnie (localStorage / IndexedDB).

---

## 1. Zasady budowy aplikacji (wymagania twarde)

Każda aplikacja w `apps/` **musi** spełniać:

### 1.1 Jeden plik, zero zależności zewnętrznych
- Cała aplikacja to jeden plik `index.html` (HTML + CSS + JS w jednym).
- **Brak build-stepu**, brak `node_modules`, brak bundlera. Otwarcie pliku w przeglądarce = działa.
- **Tylko vanilla JS** — żadnych frameworków (React/Vue/jQuery) ani bibliotek runtime'owych.
- Dopuszczalne API: wyłącznie to, co wbudowane w przeglądarkę (Canvas, IndexedDB, Clipboard, File API, drag&drop, Web Storage).
- **Brak zewnętrznych zasobów**: fonty systemowe, ikony jako inline SVG, brak requestów sieciowych. Aplikacja musi działać offline i bez internetu.

### 1.2 Persystencja danych
- **Metadane / mały stan** → `localStorage` jako jeden obiekt JSON pod kluczem z wersją, np. `ux-<tool>_v1`. Wersja w kluczu umożliwia migracje.
- **Duże blob-y** (obrazy, pliki) → `IndexedDB`. Nigdy nie trzymaj dużych dataURL w `localStorage` (limit ~5 MB).
- Pojedynczy obiekt `state` jako źródło prawdy; funkcje `save()` / `load()` z `try/catch` (localStorage może rzucać przy przepełnieniu lub trybie prywatnym).
- Migracje: przy `load()` obsługuj migrację ze starych/legacy kluczy, a potem czyść stare dane.
- Każda zmiana stanu wywołuje `save()`; start aplikacji wywołuje `await load()` przed pierwszym renderem.
- Defensywne `ensureActiveIndicesValid()` po każdej operacji usuwania, żeby aktywny indeks nigdy nie wyszedł poza zakres.

### 1.3 Architektura kodu (konwencje z image-slicer)
- Jeden obiekt `state` na górze skryptu.
- Helpery dostępu: `current*()` (np. `currentProject()`, `currentImage()`).
- Operacje CRUD jako funkcje: `create*`, `add*`, `switch*`, `rename*`, `remove*`, `move*` — każda woła `save()` + `render()`.
- Renderowanie podzielone na niezależne funkcje `render*()` (`renderProjectList`, `renderEditor`, ...), koordynowane przez jeden `render()`.
- Budowa DOM imperatywnie (`document.createElement` + `appendChild`) z dokładaniem handlerów eventów, nie `innerHTML` z stringów (bezpieczeństwo + wydajność przy interakcjach).
- ID generowane przez licznik `nextId` w stanie (stabilne identyfikatory do drag&drop i persystencji).

### 1.4 Brak sekretów / brak backendu
- Aplikacja nie wysyła danych nigdzie. Wszystko zostaje na urządzeniu użytkownika. Zero kluczy API, zero zewnętrznych endpointów.

---

## 2. Warstwa wizualna (design system)

Wszystkie aplikacje współdzielą **spójny design system** (light domyślnie + dark) wywodzący się z `image-slicer`, żeby zbiór narzędzi czytał się jako jeden produkt. Kolory realizowane są **wyłącznie przez tokeny CSS** (custom properties) — nigdy przez zakodowane wartości w komponentach.

### 2.1 Tokeny i motywy (light domyślnie + dark)

Tokeny definiowane na `:root` (light) i nadpisywane w `[data-theme="dark"]`. Aktywny motyw ustawiany przez atrybut `data-theme` na `<html>` (resolved `'light'` / `'dark'`); wybór persystowany w `state.theme` (`'light'` | `'dark'` | `'auto'` — `auto` podąża za `prefers-color-scheme` i aktualizuje się na żywo). **Domyślny motyw aplikacji to `light`.**

| Token | Light | Dark | Rola |
|------|-------|------|------|
| `--bg` | `#f6f8fa` | `#111` | tło aplikacji (`body`) |
| `--panel` | `#ffffff` | `#141414` | tło sidebar/sekcji |
| `--panel-2` | `#f1f3f5` | `#161616` | panel-header, etykiety wierszy |
| `--panel-3` | `#eaecef` | `#1a1a1a` | header aplikacji, etapy |
| `--inset` | `#ffffff` | `#1a1a1a` | tło inputów |
| `--border` | `#d0d7de` | `#2a2a2a` | bordery |
| `--border-soft` | `#e5e7eb` | `#1e1e1e` | separatory komórek |
| `--text` | `#1f2328` | `#e0e0e0` | tekst główny |
| `--text-2` | `#57606a` | `#999` | tekst wtórny |
| `--text-3` | `#6e7781` | `#777` | treść list |
| `--text-4` | `#8c959f` | `#555` | etykiety sekcji, empty states |
| `--accent` / `--accent-hover` | `#2563eb` / `#1d4ed8` | `#3b82f6` / `#2563eb` | primary, aktywne elementy, focus |
| `--danger` | `#d1242f` | `#f43f5e` | akcje destrukcyjne |
| `--chip` / `--chip-text` | `#eaecef` / `#57606a` | `#2a2a2a` / `#888` | liczniki, znaczniki |
| `--cell` / `--cell-hover` | `#ffffff` / `#f6f8fa` | `#111` / `#161616` | komórki siatki |
| `--scrollbar` | `#c8d0d8` | `#2a2a2a` | thumb scrollbara |

W komponentach używaj **wyłącznie `var(--token)`**. Kontrast interaktywnych elementów ≥ WCAG AA w obu motywach. Widoczny `:focus-visible` ring (`outline: 2px solid var(--accent)`). Przełącznik motywu w headerze (`☀`/`☾`/`◐`), cykl `light → dark → auto`.

### 2.2 Typografia
- Font: `-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif` (systemowy — zero ładowania).
- Header `h1`: `15px / 600`.
- Etykiety sekcji / panel-header: `10–11px`, `700`, `uppercase`, `letter-spacing 0.08–0.1em`, kolor `var(--text-4)`.
- Treść list: `12px`. Przyciski: `12–13px / 500`.

### 2.3 Komponenty UI (zachować spójne)
- **Przyciski**: `.btn`, warianty `.btn-primary` (filled accent), `.btn-ghost` (transparent + border), rozmiar `.btn-sm`. Ikony-akcje (✎ × ⠿ + ☀) to prawdziwe `<button>` z `aria-label`.
- **Sidebar** (`<aside aria-label="…">`) z listami: drag-to-reorder, hover-reveal akcji, inline rename, miniatury.
- **Panel**: `.panel-header` (etykieta + licznik `.count`) + `.panel-body` (scrollowalne).
- **Drop zone / empty state**: dashed border `2px`, ikona SVG, opis, akcent na hover/drag-over.
- **Custom scrollbars**: cienkie (`4–6px`), thumb `var(--scrollbar)`.
- **Resize handle** panelu (np. sidebar) z persystencją szerokości w stanie (`role="separator"`).

---

## 3. Wzorce interakcji (UX) — domyślnie w każdej aplikacji

- **Drag & drop** plików do okna (upload) + reorder elementów w listach (z wskaźnikiem `drag-over-above`/`below`).
- **Paste z clipboard** (Ctrl+V) — gdzie ma to sens (obrazy, tekst).
- **Inline rename**: dwuklik na nazwie LUB ikona ✎; commit na Enter/blur, anulowanie na Escape.
- **Hover-reveal akcji**: ikony edycji/usuwania widoczne dopiero na hoverze elementu (mniej szumu wizualnego).
- **Live preview / natychmiastowa reakcja** na zmiany (np. canvas re-renderowany w czasie przeciągania).
- **Eksport**: Download (anchor + `download`) oraz Copy to clipboard (`navigator.clipboard`) z feedbackiem (przycisk → "✓ Copied" na ~1,5 s, "✗ Failed" przy błędzie).
- **Keyboard**: Enter = zatwierdź, Escape = anuluj w polach; focus management przy inline-edycji.
- **Empty states**: każdy panel ma stan "brak danych" z instrukcją, co zrobić.
- **Defensywność**: `try/catch` wokół storage, sanityzacja nazw plików (`safeFilename`), clampowanie indeksów i wartości.

---

## 4. Struktura repozytorium

```
ux-tools/
├── CLAUDE.md                      # ten dokument — konwencje i wymagania
├── apps/
│   └── <nazwa-narzedzia>/
│       ├── index.html             # cała aplikacja (single-file)
│       └── README.md              # (opcjonalnie) co robi, screenshot
└── .github/
    └── workflows/deploy.yml       # deploy apps/ na GitHub Pages
```

- Każde narzędzie ma własny folder `apps/<nazwa>/index.html` — dzięki temu URL na Pages jest czysty (`.../ux-tools/apps/<nazwa>/`) i jest miejsce na README/zasoby, a `index.html` serwuje się domyślnie. Sama aplikacja nadal jest single-file.
- Nazwy folderów: `kebab-case`, krótkie i opisowe (np. `color-picker`, `aspect-ratio`, `contrast-checker`).

---

## 5. Tworzenie nowej aplikacji — checklista

1. Utwórz folder `apps/<nazwa>/index.html`.
2. Skopiuj boilerplate: `<!DOCTYPE html>`, head z viewportem, `<style>` z design systemem (paleta z pkt 2), struktura header + main + sidebar + panele.
3. Zdefiniuj `state`, `save()`, `load()` (z wersjonowanym kluczem `ux-<tool>_v1`) i boot IIFE.
4. Zbuduj CRUD (`create/add/switch/rename/remove`) + `render()`.
5. Dodaj wzorce z pkt 3 adekwatne do narzędzia (drag&drop, paste, eksport, empty states).
6. Przetestuj: odśwież → stan się przywraca; usuń dane → empty state; przepełnienie → IndexedDB zamiast localStorage dla blobów.
7. Dodaj wpis do `apps/README.md`.

---

## 6. Hosting / deploy

Aplikacje publikowane są statycznie przez GitHub Pages (workflow `.github/workflows/deploy.yml` wgrywający katalog `apps/`), identycznie jak w `image-slicer`. Każda apka dostępna pod:
`https://<user>.github.io/ux-tools/apps/<nazwa>/`

---

## 7. Konwencje nazewnictwa

- Foldery aplikacji, klucze storage, ID w HTML: `kebab-case`.
- Funkcje JS: `camelCase`, prefiksy `render*` / `add*` / `remove*` / `switch*` / `rename*` / `move*` / `ensure*`.
- Zmienne stanu: `camelCase` (`activeProject`, `sidebarWidth`, `nextImageId`).
- Elementy DOM: `id` w `kebab-case`, referencje w JS jako `camelCase` (np. `#btn-upload` → `btnUpload`).

---

## 8. Proces i skille

Narzędzia budujemy przez zestaw 5 skilli w `.claude/skills/` (wywoływane jako `/ux-<name>`). Adaptacja Twojego repo `proto` do skali single-file toola — lżejsza (jeden SPEC.md, jeden skill implementujący, brak ADR-ów i wieloskillowego routingu).

**Nowe narzędzie (od iskry):**
```
ux-idea   → apps/<name>/SPEC.md (sekcja Idea + Glossary)   [docs, brak kodu]
ux-spec   → SPEC.md (sekcja Requirements)                  [docs, brak kodu]
ux-build  → apps/<name>/index.html                          [JEDYNY skill piszący kod]
```

**Żywe narzędzie (cecha / błąd):**
```
ux-feature → wpis "Change — feature" w SPEC.md (plan MVP, scope wpływu) → route do ux-build
ux-bug     → wpis "Change — bug" w SPEC.md (repro, root cause file:line, fix plan, regresja) → route do ux-build
ux-build   → implementuje najnowszy pending Change w index.html
```

Każdy skill: wywiad jedno-pytaniowy po polsku, angielskie nazwy kodowe, **git checkpoint** (commit przed + po = odwracalny etap). Pełne instrukcje w każdym `SKILL.md`.

**Wskazanie narzędzia:** `ux-spec`/`ux-build`/`ux-bug`/`ux-feature` pracują nad istniejącym narzędziem — podajesz je jako argument (`/ux-build customer-journey`) lub w zdaniu; przy jednym istniejącym narzędziu skill wybiera je sam, przy wielu pyta. `ux-idea` tworzy nowe, więc nazwa powstaje w wywiadzie (możesz ją zasugerować w argumencie). Pojemność dokumentu per apka to **jeden `apps/<name>/SPEC.md`** z sekcjami: Idea, Requirements, Glossary, Changes.
