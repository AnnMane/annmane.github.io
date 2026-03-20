---
title: "Python: Refactor słownika na dataclass — kiedy warto i jak"
date: 2025-11-22
description: "Przez miesiąc używałam dict do przechowywania stanu gry w Minesweeper. Refactor na dataclass zmienił wszystko."
tags: ["Python", "refactoring", "clean code"]
---

W pierwszej wersji Minewsweepera `Cell` wyglądała tak:

```python
cell = {
    'is_mine': False,
    'is_revealed': False,
    'is_flagged': False,
    'adjacent': 0
}
```

Działało. Ale przy każdym `cell['is_revealed']` traciłam czas na myślenie, czy literówka. Brak autocomplete. Brak type hints. Brak `.` dostępu.

## Refactor na dataclass

```python
from dataclasses import dataclass, field

@dataclass
class Cell:
    is_mine:     bool = False
    is_revealed: bool = False
    is_flagged:  bool = False
    adjacent:    int  = 0
```

Teraz:
- `cell.is_revealed` zamiast `cell['is_revealed']`
- IDE podpowiada pola
- Typy są jawne
- `repr` z pudełka: `Cell(is_mine=False, is_revealed=True, ...)`

## Kiedy NIE warto

Dataclass to overhead przy małych, tymczasowych strukturach. Jeśli tworzysz i wyrzucasz tysiące obiektów w pętli — `__slots__` lub namedtuple są szybsze:

```python
from dataclasses import dataclass

@dataclass(slots=True)  # Python 3.10+
class Cell:
    is_mine:     bool = False
    is_revealed: bool = False
    is_flagged:  bool = False
    adjacent:    int  = 0
```

`slots=True` redukuje zużycie pamięci o ~30–40% przy dużych tablicach obiektów.

## Wynik w Minesweeper

Plansza Expert to 16×30 = 480 komórek. Każda jako `Cell` z `slots=True` zamiast dicts. Mierzalne? Przy 480 obiektach — nie bardzo. Ale kod jest znacznie czytelniejszy i łatwiejszy do testowania.

Refactor trwał 20 minut i był wart każdej sekundy.
