---
title: "SQL: CTE kontra podzapytanie — nie tylko czytelność"
date: 2025-10-25
description: "Większość tutoriali mówi, że CTE to 'tylko czytelniejsze podzapytanie'. To nie do końca prawda. Kiedy CTE jest szybsze i dlaczego."
tags: ["SQL", "PostgreSQL", "performance", "optimization"]
---

Słyszałam wielokrotnie: *CTE to tylko czytelna wersja podzapytania — optymalizator i tak przekształci jedno w drugie*.

W PostgreSQL to **nie zawsze prawda**.

## Przykład: raport miesięczny

Mam zapytanie, które:
1. Filtruje aktywnych użytkowników (podzapytanie)
2. Liczy ich aktywność (agregacja)
3. Rankuje wyniki

```sql
-- Wersja z podzapytaniem (zagnieżdżona)
SELECT
    u.name,
    a.action_count,
    RANK() OVER (ORDER BY a.action_count DESC) as rank
FROM users u
JOIN (
    SELECT user_id, COUNT(*) as action_count
    FROM actions
    WHERE created_at >= '2026-01-01'
    GROUP BY user_id
) a ON u.id = a.user_id
WHERE u.status = 'active';
```

```sql
-- Wersja z CTE
WITH active_users AS (
    SELECT id, name FROM users WHERE status = 'active'
),
activity AS (
    SELECT user_id, COUNT(*) as action_count
    FROM actions
    WHERE created_at >= '2026-01-01'
    GROUP BY user_id
)
SELECT
    u.name,
    a.action_count,
    RANK() OVER (ORDER BY a.action_count DESC) as rank
FROM active_users u
JOIN activity a ON u.id = a.user_id;
```

## Kluczowa różnica w PostgreSQL

Przed PostgreSQL 12 CTE był **zawsze zmaterializowany** — wykonywany raz i wynik trafiał do tymczasowej tabeli, niezależnie od tego, ile razy był referencjonowany. To mogło być szybsze LUB wolniejsze, zależnie od zapytania.

Od PostgreSQL 12 optymalizator może **inlinować** CTE jak podzapytanie — chyba że użyjesz `MATERIALIZED`:

```sql
WITH MATERIALIZED expensive_calc AS (
    SELECT ... FROM big_table WHERE ...
)
SELECT * FROM expensive_calc e1
JOIN expensive_calc e2 ON e1.id = e2.parent_id;
```

Tutaj materializacja pomaga — `expensive_calc` jest liczone raz, nie dwa razy.

## Moja zasada

- **CTE do czytelności** — zawsze, szczególnie gdy logika jest wieloetapowa
- **Podzapytanie** — gdy potrzebuję korelacji z zapytaniem zewnętrznym
- **`WITH MATERIALIZED`** — gdy ten sam CTE referencjonuję wielokrotnie i jest kosztowny

`EXPLAIN ANALYZE` przed i po — zawsze.
