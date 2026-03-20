---
title: "Power Query: Normalizacja walut bez zewnętrznych API"
date: 2026-01-05
description: "W projekcie IT Job Market musiałam przeliczać EUR i USD na PLN. Pokazuję jak zrobiłam to w Power Query bez zbędnych zależności."
tags: ["Power BI", "Power Query", "ETL"]
---

W danych z projektu IT Job Market wynagrodzenia były w trzech walutach: PLN, EUR, USD. Żeby porównania miały sens, musiałam wszystko sprowadzić do PLN.

Najprostsze podejście? Hardkodować kurs. Ale to złe — dashboard starzeeje się błyskawicznie.

## Podejście 1: Tabela kursów w modelu

Stworzyłam osobną tabelę `dim_exchange_rates`:

| currency | rate_to_pln | updated_date |
|----------|-------------|--------------|
| EUR      | 4.28        | 2026-01-01   |
| USD      | 3.91        | 2026-01-01   |
| PLN      | 1.00        | 2026-01-01   |

Aktualizuję ją ręcznie przy odświeżaniu dashboardu. Nie jest automatyczne, ale transparentne.

## Power Query: dołączanie kursu i przeliczanie

```powerquery
let
    Source = Excel.CurrentWorkbook(){[Name="offers"]}[Content],

    // Dołącz tabelę kursów
    Rates = Excel.CurrentWorkbook(){[Name="exchange_rates"]}[Content],
    Merged = Table.NestedJoin(Source, "currency", Rates, "currency", "rates", JoinKind.LeftOuter),
    Expanded = Table.ExpandTableColumn(Merged, "rates", {"rate_to_pln"}),

    // Przelicz na PLN
    WithPLN = Table.AddColumn(Expanded, "salary_pln",
        each [salary_min] * [rate_to_pln], type number)
in
    WithPLN
```

## Wyodrębnianie flagi "remote"

Przy okazji, kolumna `location` zawierała tekst w stylu `"Warszawa / Zdalnie"` lub `"Remote"`. Wyciągnęłam flagę booleanową:

```powerquery
WithRemote = Table.AddColumn(WithPLN, "is_remote",
    each Text.Contains(Text.Lower([location]), "zdal")
      or Text.Contains(Text.Lower([location]), "remote"),
    type logical)
```

Proste `Text.Contains` + `Text.Lower` (żeby nie przejmować się wielkością liter).

## Wynik

Model zawiera salaries ujednolicone do PLN i czysty boolean `is_remote`. W DAX mogę od razu filtrować i agregować bez obejść.
