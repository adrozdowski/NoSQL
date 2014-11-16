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
