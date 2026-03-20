---
title: "Power BI: Dlaczego nie używam flat table i jak zaprojektować Star Schema"
date: 2026-01-30
description: "Flat table to najczęstszy błąd początkujących w Power BI. Pokazuję konkretny przykład z projektu i jak Star Schema poprawia i wydajność, i DAX."
tags: ["Power BI", "Data Modeling", "Star Schema"]
---

Kiedy zaczynałam z Power BI, wrzucałam wszystko do jednej tabeli. To jest złe. Oto dlaczego i co zamiast tego.

## Problem z flat table

Wyobraź sobie tabelę ofert pracy:

| job_id | title | company | skill_1 | skill_2 | skill_3 | salary |
|--------|-------|---------|---------|---------|---------|--------|
| 1 | Backend Dev | Acme | Python | SQL | Docker | 12000 |
| 2 | Analyst | Corp | SQL | Excel | — | 8000 |

Chcę policzyć, ile ofert wymaga SQL. Muszę szukać w `skill_1`, `skill_2`, `skill_3` — albo mieć 10 kolumn dla 10 technologii. Koszmar.

## Star Schema: wymiary + fakty

```
dim_jobs (job_id, title, company, salary_min, salary_max)
dim_skills (skill_id, skill_name, category)
bridge_job_skills (job_id, skill_id)   ← many-to-many!
```

`bridge_job_skills` to **tabela bridge** — rozwiązuje relację many-to-many (jedna oferta wymaga wielu umiejętności, jedna umiejętność pojawia się w wielu ofertach).

## Konfiguracja relacji w Power BI

W Power BI Desktop:
- `dim_jobs` ←→ `bridge_job_skills` (one-to-many po `job_id`)
- `dim_skills` ←→ `bridge_job_skills` (one-to-many po `skill_id`)
- Kierunek filtra: **oba kierunki** na bridge table

## DAX staje się prostszy

```dax
-- Liczba ofert wymagających SQL
Jobs With SQL =
CALCULATE(
    DISTINCTCOUNT(dim_jobs[job_id]),
    dim_skills[skill_name] = "SQL"
)
```

Bez bridge table to byłby potwór z wieloma SEARCH/FIND.

## Bonus: wydajność

Star Schema to mniejszy model. Power BI kompresuje kolumny w kolumnowym magazynie danych — wąskie tabele faktów z liczbami kompresują się świetnie. Mój model z 2400 ofert i 40 umiejętnościami waży mniej niż 1 MB.
