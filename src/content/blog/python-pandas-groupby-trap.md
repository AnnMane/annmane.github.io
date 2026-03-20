---
title: "Python: Pułapka groupby().apply() — straciłam godzinę na jeden znak"
date: 2026-01-18
description: "groupby().apply() w Pandas potrafi zachowywać się inaczej niż myślisz, gdy włączone jest include_groups. Historia prawdziwego bugu i czego mnie nauczyła."
tags: ["Python", "Pandas", "debugging"]
---

Analizowałam dane o ofertach pracy i chciałam dla każdej kategorii wziąć top 5 ofert według wynagrodzenia. Klasyczne zastosowanie `groupby().apply()`:

```python
def top5(group):
    return group.nlargest(5, 'salary')

result = df.groupby('category').apply(top5)
```

Kod działał. Wynik wyglądał poprawnie. Ale kiedy sprawdziłam sumy — coś nie grało. Niektóre wartości były zduplikowane.

## Co się stało

W nowszych wersjach Pandas (≥2.0) `groupby().apply()` domyślnie **dołącza kolumnę grupującą** do przekazywanego DataFrame wewnątrz funkcji. Efekt: `nlargest` operowało na DataFrame z dodatkową kolumną `category`, która zaburzała indeks i powodowała dziwne duplikaty przy reindeksacji.

Komunikat był w FutureWarning, który zignorowałam. Oczywiście.

## Rozwiązanie

```python
# Opcja 1: jawnie wyłącz include_groups
result = df.groupby('category').apply(top5, include_groups=False)

# Opcja 2: prostszy refactor bez apply
result = (
    df.sort_values('salary', ascending=False)
      .groupby('category')
      .head(5)
)
```

Opcja 2 jest szybsza i bardziej czytelna. `groupby().head(n)` istnieje właśnie do tego.

## Lekcja

Nigdy nie ignoruj `FutureWarning` w Pandas. Zazwyczaj opisuje dokładnie co się zmieni i kiedy. Teraz przy każdym projekcie zaczynam od `import warnings; warnings.filterwarnings('error', category=FutureWarning)` — wymusza to zajęcie się każdym ostrzeżeniem od razu.
