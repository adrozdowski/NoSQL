# Zadania z NoSQL

Specyfikacja techniczna komputera:

| Rodzaj komponentu    | Komponent                       |
|-----------------------|---------------------------------|
| System operacyjny     | Linux Kubuntu 14.04 (64-bitowy) |
| Procesor              | Intel i5 4590                   |
| Ilość rdzeni          | 4                               |
| Ilość Ram             | 8 GB                            |
| Rodzaj dysku twardego | HDD                             |



### Zadanie 3 - ANAGRAMY

#####Wczytanie pliku
Plik [word_list.txt](https://github.com/adrozdowski/NoSQL/blob/master/word_list.txt) wczytujemy poniższym poleceniem. 
```sh
mongoimport -c words --type csv --file word_list.txt -f "word"
```


Wszystkie anagramy otrzymujemy za pomocą poniższego kodu:
```js
db.words.mapReduce(
  function(){emit(Array.sum(this.word.split("").sort()), this.word);},
  function(key, values) {return values.toString()},
  {
    query: {},
    out: "anagramy"
  }
)
```

W kolekcji anagramy znajdują się pary (posortowane litery - słowa, które zawierają te litery oddzielone przecinkami).
