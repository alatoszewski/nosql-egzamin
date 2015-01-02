# Egzamin
W zadaniu użyłem bazy GetGlue.

## Pierwsze map reduce
Poniższe zapytanie liczy "lajki" dla poszczególnych filmów z serii Harry Potter.
W części query wyszukuje tylko te filmy które zaczynają sie od słów Harry Potter.
Użyłem do tego następującego wyrażenia regularnego: /^Harry Potter/.
Map grupuje filmy wg nazwy i dodaje pole action do kolekcji pomocniczej.
Reduce zlicza ilość wystąpień słowa "Liked" w tablicy values.