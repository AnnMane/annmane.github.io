---
title: "SQL & Power Query: Jak rozwiązałam kolizję ID z trzech platform streamingowych"
date: 2025-12-10
description: "Netflix, Amazon i Disney+ używają własnych, nakładających się numerycznych ID. Composite Primary Keys w Power Query rozwiązały problem elegancko."
tags: ["SQL", "Power Query", "Data Modeling", "debugging"]
---

W projekcie Global Streaming Catalog połączyłam dane z Netflixa, Amazona i Disney+. Każda platforma używa własnego systemu ID dla tytułów. Problem: **te ID nakładają się**.

Netflix ma tytuł z ID `1042`. Amazon też ma tytuł z ID `1042`. To zupełnie różne filmy.

Gdy połączyłam trzy zbiory naiwnym UNION — model nie wiedział, który `1042` to który.

## Rozwiązanie 1: Composite Primary Key

Dodaję kolumnę `platform` i łączę ją z `content_id` w composite key:

```powerquery
// W Power Query
WithKey = Table.AddColumn(Source, "composite_id",
    each [platform] & "_" & Text.From([content_id]),
    type text)
```

Teraz `netflix_1042` ≠ `amazon_1042`. ✓

## Rozwiązanie 2: Surrogate key (alternatywa)

Jeśli chcę numerycznego klucza do relacji:

```sql
-- Po unii w SQL
WITH combined AS (
    SELECT 'netflix' as platform, id, title, genre FROM netflix
    UNION ALL
    SELECT 'amazon', id, title, genre FROM amazon
    UNION ALL
    SELECT 'disney', id, title, genre FROM disney
)
SELECT
    ROW_NUMBER() OVER (ORDER BY platform, id) as surrogate_id,
    platform,
    id as original_id,
    title,
    genre
FROM combined;
```

`ROW_NUMBER()` tworzy unikalny numeryczny klucz zastępczy.

## Które podejście wybrałam

W Power BI wybrałam composite key (string) — prostszy w debugowaniu, od razu widać "netflix_1042" i wiesz o co chodzi. Surrogate key jest lepszy przy dużych wolumenach danych (int kompresuje się lepiej niż string).

Przy ~50k tytułów łącznie — string jest wystarczający.
