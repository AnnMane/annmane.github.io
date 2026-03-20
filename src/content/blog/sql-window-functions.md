---
title: "SQL: Jak przestałam używać podzapytań i pokochałam okna"
date: 2026-02-28
description: "Window functions zmieniły sposób, w jaki piszę SQL. Pokazuję trzy wzorce, które zastąpiły mi skomplikowane podzapytania."
tags: ["SQL", "PostgreSQL", "performance"]
---

Przez długi czas pisałam SQL tak jak większość początkujących — jak tylko chciałam porównać wiersz z poprzednim lub policzyć running total, sięgałam po podzapytanie:

```sql
SELECT
    job_title,
    salary,
    (SELECT AVG(salary) FROM jobs j2 WHERE j2.category = j1.category) as avg_in_category
FROM jobs j1;
```

Działa. Ale przy 100k wierszach — błaga o litość.

## Window functions — 3 wzorce które używam najczęściej

### 1. Running total
```sql
SELECT
    date,
    revenue,
    SUM(revenue) OVER (ORDER BY date) as running_total
FROM sales;
```

### 2. Ranking w grupie
```sql
SELECT
    name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank
FROM employees;
```
Chcę tylko liderów każdego działu? Owijam w CTE i filtruję `WHERE dept_rank = 1`.

### 3. Porównanie z poprzednim wierszem (LAG)
```sql
SELECT
    month,
    users,
    LAG(users) OVER (ORDER BY month) as prev_month,
    users - LAG(users) OVER (ORDER BY month) as growth
FROM monthly_stats;
```

## Kiedy to mnie uderzyło

Przy analizie danych z KPMG musiałam pokazać, jak wartości zmieniają się w czasie relative do poprzedniego okresu. Podzapytanie byłoby koszmarem — `LAG` zajął jedną linię.

`PARTITION BY` to `GROUP BY` dla okna. `ORDER BY` mówi jak "przesuwać" okno. Tyle.
