# Customer Journey

> Structured customer journey maps — a grid of steps × dimensions — replacing the freeform Axure canvas.

## Idea

### Core concept
Single-file narzędzie do tworzenia i utrzymywania **map podróży klienta (customer journeys)** jako **ustrukturyzowanej siatki**: **kolumny = kroki** procesu, **wiersze = wymiary** (emocje, pain points, pomysły, insighty, touchpointy typu entrypoint/push). CJ są pogrupowane po **kategoriach procesów** i powiązane z **personami**. Siatka sama się układa — koniec z ręcznym przepychaniem elementów po wolnym płótnie.

### User problem
- **Ręczna praca w Axure**: CJ dziś żyją na wolnym płótnie → dużo czasu idzie na ogarnianie przestrzeni i ręczne dodawanie/przesuwanie elementów (kroków, entrypointów, pushy) zamiast na treść. Zmiana wymaga przepychania wszystkiego.
- **Trudna prezentacja zmiany**: pokazanie, jak CJ zmieni się po wprowadzeniu zmiany, jest dzisiaj żmudne i robione ręcznie.

### Target user
UX designer (autor) pracujący nad podróżami klienta w procesach finansowych (np. cash loan, account opening). Jedna osoba autorska; narzędzie osobiste, dane lokalnie.

### Core action
Otworzyć customer journey → edytować **siatkę kroków × wymiarów** (dodawać/przesuwać kroki, edytować komórki w miejscu) → mieć to od razu uporządkowane i gotowe do pokazania.

### Scope
- **MVP**:
  - Kategorie procesów (cash loan, account opening…) z licznikiem CJ + baza person.
  - Lista CJ w kategorii (osoba + krótki opis przypadku).
  - **Edytor CJ jako siatka**: kolumny = kroki (dodawanie, przenoszenie, rename, usuwanie), wiersze = wymiary (emocje, pain points, pomysły, insighty, touchpointy: entrypoint/push); edycja komórek w miejscu.
- **Later** (przyszłe Change'y):
  - Warianty: kopia CJ → wariant; CJ bazowa oznaczona jako „aktualna"; wariant = bazowa + zmiany.
  - Wersjonowanie historii edycji CJ.
  - Prezentacja zmiany (side-by-side przed/po / eksport) — doprecyzować w `ux-spec` format dostarczenia.

## Requirements
<!-- ux-spec fills this: data model, actions, screens, states, interactions, edge cases -->

## Glossary
| Term (PL) | Code name (EN) | Definition |
|-----------|----------------|------------|
| kategoria procesu | `category` | Grupa procesów biznesowych (np. cash loan, account opening) grupująca customer journeys. |
| persona | `persona` | Postać użytkownika, której dotyczy dany customer journey. |
| customer journey | `journey` | Mapa podróży klienta przez dany proces — siatka kroków × wymiarów. |
| krok | `step` | Pojedynczy etap procesu — **kolumna** w siatce CJ. |
| wymiar | `dimension` | Wiersz w siatce CJ opisujący aspekt (emocje, pain points, pomysły, insighty, touchpointy). |
| emocja | `emotion` | Wymiar: co klient czuje w danym kroku. |
| pain point | `pain-point` | Wymiar: trudność/frustracja w danym kroku. |
| pomysł | `idea` | Wymiar: propozycja ulepszenia dla kroku. |
| insight | `insight` | Wymiar: wniosek/badawcza obserwacja dla kroku. |
| touchpoint | `touchpoint` | Punkt kontaktu klienta (entrypoint, push) w kroku. |
| entrypoint | `entry-point` | Touchpoint: wejście klienta do kroku/kanalu. |
| push | `push` | Touchpoint: powiadomienie push wychodzące do klienta. |
| komórka | `cell` | Wartość w przecięciu kroku (kolumna) i wymiaru (wiersz). |
| wariant | `variant` | (Later) Kopia CJ bazowej z wprowadzonymi zmianami. |
| wersja | `version` | (Later) Zapisany stan CJ w historii edycji. |
| bazowa / aktualna | `base` / `current` | (Later) CJ oznaczona jako aktualna, od której powstają warianty. |

## Changes
<!-- ux-feature / ux-bug append Change entries here -->
