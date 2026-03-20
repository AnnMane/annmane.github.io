---
title: "DAX: Dlaczego CALCULATE robi magię i kiedy mnie zaskoczyło"
date: 2026-03-10
description: "Context transition w DAX potrafi dać zupełnie nieoczekiwane wyniki. Pokazuję konkretny bug z projektu IT Job Market, który zajął mi dwa dni."
tags: ["Power BI", "DAX", "debugging"]
---

Pracując nad dashboardem IT Job Market, miałam miarę, która miała liczyć **średnią liczbę wymagań technologicznych per oferta pracy**. Wyglądała tak:

```dax
Avg Requirements Per Job =
DIVIDE(
    COUNTROWS(fact_requirements),
    DISTINCTCOUNT(fact_jobs[job_id])
)
```

Działała poprawnie w ogólnym KPI. Ale gdy wrzuciłam ją do tabeli podzielonej po technologiach — każda technologia pokazywała tę samą wartość globalną, zamiast wartości dla swojego kontekstu. 🤔

## Co poszło nie tak

Problem leżał w tym, jak DISTINCTCOUNT zachowuje się w row context. W tabeli wizualizacji DAX automatycznie iteruje po wierszach, ale moja miara nie respektowała filtra narzuconego przez kolumnę technologii — bo `fact_jobs` nie miała bezpośredniej relacji do kolumny filtrującej.

## Rozwiązanie: CALCULATE + context transition

```dax
Avg Requirements Per Job (Fixed) =
VAR total_jobs =
    CALCULATE(
        DISTINCTCOUNT(fact_jobs[job_id]),
        RELATEDTABLE(bridge_job_skills)
    )
RETURN
DIVIDE(COUNTROWS(bridge_job_skills), total_jobs)
```

`CALCULATE` wymusza **przejście kontekstu** — zamienia row context w filter context, co pozwala relacji "przepłynąć" przez tabelę bridge.

## Lekcja

Kiedy miara działa globalnie, ale sypie się w kontekście wiersza tabeli — prawie zawsze chodzi o context transition. Rozwiązanie to `CALCULATE` owijające tę część, która ma "widzieć" bieżący filtr.

Dwa dni debugowania = jedno zdanie lekcji.
