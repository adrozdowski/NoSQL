### Zadanie 2
Do wykonania posłużyłem się bazą zamieszczoną na stronie prowadzącego. Baza nosi nazwę GetGlue. Najpierw skonstruowałem zapytania w MongoDB, a następnie przepisałem je do Pymongo.

W celu zainicjwalizowania bazy w pymongo należy dopisać dwie linie na samym początku kodu:

```sh
from pymongo import MongoClient
db = MongoClient().test
```
###Zapytanie 1

Agragacja ma policzyć i zwrócić 10 reżyserów, którzy mają na swoim koncie najmniejszą liczbę filmów

#MongoDB
```sh
db.movies.aggregate(
    { $match: {"modelName": "movies" || "tv_shows"  } },
    { $group: {_id: {"dir": "$director", id: "$title"}, count: {$sum: 1}} },
    { $group: {_id: "$_id.dir" , count: {$sum: 1}} },
    { $sort: {count: 1} },
    { $limit : 10}
    );
```

#pymongo
```sh
db.movies.aggregate(
    { "$match": {"modelName": "movies" || "tv_shows"  } },
    { "$group": {"_id": {"dir": "$director", "id": "$title"}, "count": {"$sum": 1}} },
    { "$group": {"_id": "$_id.dir" , "count": {"$sum": 1}} },
    { "$sort": {"count": 1} },
    { "$limit" : 10}
    );
```
Wynik przedstawiony w postaci tabelki:

|      Reżyser      | Ilość fimów |
|:-----------------:|:-----------:|
| lou pizarro       |      1      |
| c.s. drury        |      1      |
| émile reynaud     |      1      |
| paul grimault     |      1      |
| sasuke            |      1      |
| Charles Sturridge |      1      |
| dave edwards      |      1      |
| perri peltz       |      1      |
| Tate Taylor       |      1      |
| steven pimlott    |      1      |

Wynik przedstawiony w postaci wykresu kołowego:

![Alt text](https://raw.githubusercontent.com/adrozdowski/NoSQL/master/Images/Zadanie2Zapytanie1.jpg)

  


###Zapytanie 2


Agregacja odpowiada na pytanie, jakie jest 5 najmniej popularnych filmów i przedstawień TV?
#MongoDB
```sh
db.movies.aggregate(
    { $match: {"modelName": "movies" || "tv_shows"  } },
    { $group: {_id: "$title", count: {$sum: 1}} },
    { $sort: {count: 1} },
    { $limit : 5}
    );	
  ```  
#pymongo
```sh
db.movies.aggregate(
    { "$match": {"modelName": "movies" || "tv_shows"  } },
    { "$group": {"_id": "$title", "count": {"$sum": 1}} },
    { "$sort": {"count": 1} },
    { "$limit" : 5}
    );	
```
Wynik przedstawiony w postaci tabelki:

|             Nazwa filmu             | Ilość wystąpień |
|:-----------------------------------:|:---------------:|
| The Wagons Roll at Night            |        1        |
| Hamam                               |        1        |
| Yaji and Kita: The Midnight Pilgrim |        1        |
| Government Girl                     |        1        |
| sasuke                              |        1        |

Wynik przedstawiony w postaci wykresu kołowego:

![Alt text](https://raw.githubusercontent.com/adrozdowski/NoSQL/master/Images/Zadanie2Zapytanie2.jpg)

###Zapytanie 3
5 najbardziej aktywnych użytkowników
#MongoDB
```sh
db.movies.aggregate({$group:{_id: "$userId", count:{$sum: 1}}},{$sort:{count: -1}},{$limit: 5});
```
#pymongo
```sh

db.movies.aggregate({"$group":{"_id": "$userId", "count":{"$sum": 1}}},{"$sort":{"count": -1}},{"$limit": 5});
```
Wynik przedstawiony w postaci tabelki:

|   Użytkownik  | Aktywność (ilość wpisów) |
|:-------------:|:------------------------:|
| LukeWilliamss |          696782          |
|   demi_konti  |           68137          |
|    bangwid    |           59261          |
|    zenofmac   |           56233          |
|  agentdunham  |            691           |

Wynik przedstawiony w postaci wykresu kołowego:

![Alt text](https://raw.githubusercontent.com/adrozdowski/NoSQL/master/Images/Zadanie2Zapytanie3.jpg)
###Zapytanie 4
Największa ilość pozytywnych wpisów.

#MongoDB
```sh
db.movies.aggregate( { $match: { "action": "Liked" }},{ $match: { "comment": {$ne: ""} } }, { $group: { _id: "$title", count: {$sum: 1} } }, { $sort: { count: -1 } }, { $limit: 10 } );

```
#pymongo
```sh
db.movies.aggregate( { "$match": { "action": "Liked" }},{ "$match": { "comment": {"$ne": ""} } }, { "$group": { "_id": "$title", "count": {"$sum": 1} } }, { "$sort": { "count": -1 } }, { "$limit": 10 } );

```

Wynik przedstawiony w postaci tabelki:

|                Nazwa filmu                | Ilość pozytywnych wpisów |
|:-----------------------------------------:|:------------------------:|
| Lord of the Rings: The Return of the King |            815           |
|                 Fight Club                |            725           |
|                  Iron Man                 |            722           |
|                Pulp Fiction               |            721           |
|                The Hangover               |            691           |
|               X-Men", "count              |            667           |
|               Monsters, Inc.              |            644           |
|                 Braveheart                |            638           |
|             Kill Bill: Vol. 1             |            631           |
|                   WALL-E                  |            624           |

Wynik przedstawiony w postaci wykresu kołowego:

![Alt text](https://raw.githubusercontent.com/adrozdowski/NoSQL/master/Images/Zadanie2Zapytanie4.jpg)

###Uwaga! 

W czasie używania funkcji agregujących w Mongo 2.8.0 napotkałem na kilka problemów. Rozwiązania na szczęcie znalazłem stronach internetowych. Chociaż nie jestem pewien czasów jakie otrzymałem w przypadku Mongo 2.8.0 RC. Znacznie róźnią się od "gorszej" wersji. Wyniki zapytań były identyczne jak w przypadku Mongo 2.6.4 i PyMongo.

Czas działania agregacji (Podałem tylko czas rzeczywisty):

| Zapytania   |   Mongo 2.6.4   |     PyMongo     |  Mongo 2.8.0 RC wiredtiger |
|-------------|:---------------:|:---------------:|:----------------:|
| Zapytanie 1 | real,65m08.312s | real,69m24.881s | real,90m11.756s  |
| Zapytanie 2 | real,70m22.559s | real,75m45.714s | real,111m25.483s |
| Zapytanie 3 | real,69m46.713s | real,75m22.741s | real,112m12.649s |
| Zapytanie 4 | real,71m43.012s | real,77m57.998s | real,113m09.652s |




