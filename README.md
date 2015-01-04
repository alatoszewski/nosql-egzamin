# Egzamin
W zadaniu użyłem bazy GetGlue.

## Pierwsze map reduce
Poniższe zapytanie liczy "lajki" dla poszczególnych filmów z serii Harry Potter.
W części query wyszukuje tylko te filmy które zaczynają sie od słów Harry Potter.
Użyłem do tego następującego wyrażenia regularnego: /^Harry Potter/.
Map grupuje filmy wg nazwy i dodaje pole action do kolekcji pomocniczej.
Reduce zlicza ilość wystąpień słowa "Liked" w tablicy values.

Piersze map reduce:
```sh
db.movies.mapReduce(
  function(){emit(this.title, this.action);},
  function(key, values)
  {
    var count = 0;
    for(var i = 0; i < values.length; ++i){
      if(values[i] == "Liked")
	count++;
    }
    return count
  },
  {
    query: {title: /^Harry Potter/},
    out: "hplikes"
  }
)
```


Wynik w kolekcji hplikes:
```sh
{ "_id" : "Harry Potter", "value" : 14 }
{ "_id" : "Harry Potter Podcast", "value" : 0 }
{ "_id" : "Harry Potter and the Chamber of Secrets", "value" : 4 }
{ "_id" : "Harry Potter and the Deathly Hallows", "value" : 3 }
{ "_id" : "Harry Potter and the Deathly Hallows, Part 1", "value" : 10 }
{ "_id" : "Harry Potter and the Deathly Hallows, Part 2", "value" : 28 }
{ "_id" : "Harry Potter and the Deathly Hallows: Part I", "value" : 3 }
{ "_id" : "Harry Potter and the Deathly Hallows: Part II", "value" : 0 }
{ "_id" : "Harry Potter and the Forbidden Journey", "value" : 1 }
{ "_id" : "Harry Potter and the Goblet of Fire", "value" : 4 }
{ "_id" : "Harry Potter and the Half", "value" : 26 }
{ "_id" : "Harry Potter and the Half-Blood Prince", "value" : 2 }
{ "_id" : "Harry Potter and the Order of the Phoenix", "value" : 2 }
{ "_id" : "Harry Potter and the Prisoner of Azkaban", "value" : 8 }
{ "_id" : "Harry Potter and the Sorcerer&apos;s Stone", "value" : 1 }
{ "_id" : "Harry Potter and the Sorcerer's Stone", "value" : 3 }
{ "_id" : "Harry Potter: A Decade of Magic", "value" : 31 }
{ "_id" : "Harry Potter: The Final Chapter", "value" : 25 }
```

## Drugie map reduce
Drugie zapytanie oblicza ilość komentarzy dla każdej z kategorii filmów.
```sh
db.movies.mapReduce(
  function(){emit(this.modelName, 1);},//Musi być jeden bo funkcja reduce nie działa na jednym elemencie i zamiast liczby było by ""
  function(key, values)
  {
    return Array.sum(values)
  },
  {
    query: {comment: {$ne : ""}, modelName: {$ne : null}},//Pozbywam sie pustych komentarzy i dokumentow bez rezysera.
    out: "commod"
  }
)
```


```sh
{ "_id" : "movies", "value" : 729852 }
{ "_id" : "recording_artists", "value" : 3 }
{ "_id" : "topics", "value" : 13 }
{ "_id" : "tv_shows", "value" : 1959906 }
```

## Trzecie map reduce
Ostatnie zapytanie oblicza Top10 reżyserów wg lajków i dislajków.
```sh
db.movies.mapReduce(
  function(){emit(this.director, this.action);},
  function(key, values)
  {
    var rank = 0;
    for(var i = 0; i < values.length; ++i){
      if(values[i] == "Liked")
	rank++;
      else //if(values[i] == "Disliked")
	rank--;
    }
    return rank
  },
  {
    query: { $and: [{director: {$ne : null}} , { $or: [ { action: "Liked" }, { action: "Disliked" } ]}]},
    out: "dirrank"
  }
)
```

```sh
db.dirrank.find( { value: { $not: { $type : 2 } } } ).sort({value:-1}).limit(10)//dokumenty z value nie bedacym stringiem
{ "_id" : "tony randel", "value" : 119 }
{ "_id" : "emilio estevez", "value" : 113 }
{ "_id" : "andrew currie", "value" : 110 }
{ "_id" : "billy ray", "value" : 105 }
{ "_id" : "laurence dunmore", "value" : 105 }
{ "_id" : "paul brickman", "value" : 104 }
{ "_id" : "john mcnaughton", "value" : 103 }
{ "_id" : "jeffrey nachmanoff", "value" : 99 }
{ "_id" : "olivier dahan", "value" : 98 }
{ "_id" : "peter mcdonald", "value" : 96 }
```
