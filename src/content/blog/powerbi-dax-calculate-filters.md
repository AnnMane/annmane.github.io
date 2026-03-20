---
title: "DAX: CALCULATE z wieloma filtrami — AND czy OR?"
date: 2025-11-08
description: "CALCULATE z wieloma argumentami filtrowymi to nie zawsze to, co myślisz. Konkretny przykład, który mnie zaskoczył przy analizie wynagrodzień."
tags: ["Power BI", "DAX", "debugging"]
---

Chciałam policzyć oferty pracy, które **jednocześnie** wymagają Pythona i mają wynagrodzenie powyżej 10 000 PLN. Napisałam:

```dax
Python Senior Jobs =
CALCULATE(
    COUNTROWS(dim_jobs),
    dim_skills[skill_name] = "Python",
    dim_jobs[salary_min] > 10000
)
```

Wynik: 47. Brzmi rozsądnie.

Ale potem napisałam to samo inaczej:

```dax
Python Senior Jobs v2 =
CALCULATE(
    COUNTROWS(dim_jobs),
    FILTER(dim_jobs, dim_jobs[salary_min] > 10000),
    dim_skills[skill_name] = "Python"
)
```

Wynik: 52. Inaczej. 😳

## Dlaczego?

Wiele argumentów filtrowychw `CALCULATE` to **AND** między nimi — ale każdy filtr osobno **zastępuje** istniejący filtr na tej kolumnie z zewnętrznego kontekstu.

Kluczowa różnica:
- `CALCULATE(X, filter1, filter2)` — filtr1 i filtr2 są niezależnymi tabelami filtrów
- Gdy przekazujesz wyrażenie booleanowe (`kolumna = "wartość"`) — CALCULATE opakowuje to w `FILTER(ALL(tabela), warunek)`, co **usuwa** poprzednie filtry z tej tabeli

Innymi słowy: boolean filter w CALCULATE = `KEEPFILTERS` wyłączony.

## Rozwiązanie: KEEPFILTERS

```dax
Python Senior Jobs (Correct) =
CALCULATE(
    COUNTROWS(dim_jobs),
    KEEPFILTERS(dim_skills[skill_name] = "Python"),
    KEEPFILTERS(dim_jobs[salary_min] > 10000)
)
```

`KEEPFILTERS` mówi DAX: *nie zastępuj istniejących filtrów, dodaj swój do nich*.

## Zasada którą stosuję

Kiedy nie jestem pewna zachowania `CALCULATE` z booleanami — używam `FILTER` z `KEEPFILTERS`. Verbose, ale przewidywalny.
