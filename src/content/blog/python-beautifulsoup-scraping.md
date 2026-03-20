---
title: "Python: Jak scrapowalem 2400 ofert pracy (i co mnie prawie zatrzymało)"
date: 2026-02-14
description: "Za kulisami projektu IT Job Market — od pierwszego requestu do 2400 ofert, przez rate limiting, dynamiczny HTML i oczyszczanie danych."
tags: ["Python", "BeautifulSoup", "scraping", "ETL"]
---

Projekt IT Job Market zaczął się od prostego pytania: *jakich technologii faktycznie wymagają firmy w Polsce?* Nie ankieta, nie subiektywna opinia — surowe dane z ogłoszeń.

Plan był prosty. Wykonanie — mniej.

## Problem 1: Dynamiczny HTML

Pierwsza strona z ogłoszeniami ładowała dane przez JavaScript. `requests` + `BeautifulSoup` zwracało pusty div.

Rozwiązanie: szukam endpointów API w DevTools → Network tab → XHR. Większość nowoczesnych serwisów ogłoszeniowych ma ukryte JSON API. Znalazłam je, wyciągałam dane bezpośrednio — bez Selenium.

```python
import requests

def fetch_offers(page: int) -> list[dict]:
    url = "https://api.example.com/offers"
    params = {"page": page, "limit": 50, "category": "IT"}
    r = requests.get(url, params=params, headers={"User-Agent": "Mozilla/5.0"})
    r.raise_for_status()
    return r.json()["offers"]
```

## Problem 2: Rate limiting

Po ~200 requestach zaczęłam dostawać 429. Dodałam `time.sleep` z losowym jitterem:

```python
import time, random

time.sleep(random.uniform(1.2, 2.8))
```

Nie jest szybkie. Ale działa i nie wygląda jak bot.

## Problem 3: Oczyszczanie wynagrodzeń

Pole salary wyglądało tak: `"8 000 – 15 000 PLN netto"`, `"€3,500/month"`, `"do negocjacji"`.

```python
import re

def parse_salary(raw: str) -> tuple[float | None, float | None, str]:
    raw = raw.replace('\xa0', '').replace(' ', '')
    numbers = re.findall(r'\d+(?:[.,]\d+)?', raw)
    if not numbers:
        return None, None, 'unknown'
    lo = float(numbers[0].replace(',', '.'))
    hi = float(numbers[1].replace(',', '.')) if len(numbers) > 1 else lo
    currency = 'EUR' if '€' in raw else 'PLN'
    return lo, hi, currency
```

Konwersję EUR→PLN robiłam już w Power Query, żeby zachować aktualny kurs w dashboardzie.

## Wynik

2,428 ofert, ~40 unikalnych technologii, pełen pipeline ETL → CSV → Power BI.
