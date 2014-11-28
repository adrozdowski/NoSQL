# Zadania z NoSQL

Specyfikacja techniczna komputera:

| Rodzaj komponentu    | Komponent                       |
|-----------------------|---------------------------------|
| System operacyjny     | Linux Kubuntu 14.04 (64-bitowy) |
| Procesor              | Intel i5 4590                   |
| Ilość rdzeni          | 4                               |
| Ilość Ram             | 8 GB                            |
| Rodzaj dysku twardego | HDD                             |



### Zadanie 1a

#####Przygotowanie pliku
By MongoDB mogło zaimportować plik należy poddać go obróbce poniższym skryptem pochodzącym z repozytorium prowadzącego:

```sh
#! /bin/bash

if [ -z "$1" ] ; then
echo "First replaces all the \\n with spaces then replaces all the \\r with \\n"
echo "Replace header line with: _id,title,body,tags (for mongoDB)"
echo "usage: $0 input.csv output.csv"
exit 1;
fi

tr '\n' ' ' < "$1" | tr '\r' '\n' > "$2"
sed -i '$ d' "$2"

sed -i '1 c "_id","title","body","tags"' "$2" 
```

#####Import pliku do bazy MongoDB
Aby zaimportowac plik csv do mongo nalezy wpisac ponizsza komende do konsoli:
```sh
time mongoimport -c Topics --type csv --file Train.csv --headerline
```
#####Import pliku do bazy PSQL
Nalezy wejsc do konsoli psql i w niej wpisac, poniższe zapytanie aby utworzyc tabele:
```sh
CREATE TABLE Topics(
ID INT PRIMARY KEY NOT NULL,
TITLE CHAR(128) NOT NULL,
BODY CHAR(128) NOT NULL,
TAGS CHAR(128) NOT NULL,
);
```
Nastepnie należy wypełnić tabele danymi z pliku:
```sh
COPY Topics FROM '/home/Adrian/Pulpit/NoSQL/Train.csv' DELIMITER ',' CSV;
```
Do wyliczenia czasu w Postrgesie posłużyłem się komendą:
```sh
\timing
```
Czas jest podawany w milisekundach.

##Zestawiennie czasów

| Baza Danych |                    Import I                    |                    Import II                   |                   Import III                   |
|-------------|:----------------------------------------------:|:----------------------------------------------:|:----------------------------------------------:|
|   MongoDB (2.6.4)   | real,13m22.426s  user,1m58.598s  sys,0m19.360s | real,13m21.256s  user,1m58.112s  sys,0m18.101s | real,13m21.313s  user,1m58.280s  sys,0m19.112s |
| PostgreSQL  |                  964411.873ms (około 14 minut)                 |                                  963775.013ms (około 14 minut)  | 964678.256ms (około 14 minut)     
| Mongo 2.8.0 RC z WiredTiger  |                  1164411.873ms (około 15 minut)                 |                                  -  | -                                 |

Przy Imporcie II zestawiłem ze sobą wykresy obciążenia procesora i RAM-u dla MongoDB i PostrgeSQL. Z problemów wynikających z użycia nowej wersji Mongo tylko raz wykonałem import. Liczba rekordów się zgadzała.

###MongoDB (2.6.4)

![Alt text](https://raw.githubusercontent.com/adrozdowski/NoSQL/master/Images/SystemMonitor1.jpg)

###PostreSQL

![Alt text](https://raw.githubusercontent.com/adrozdowski/NoSQL/master/Images/SystemMonitor2.jpg)

###MongoDB (2.8.0 RC z wiredtiger)

![Alt text](https://raw.githubusercontent.com/adrozdowski/NoSQL/master/Images/SystemMonitor3.jpg)

Nie wiem skąd takie zużycie procesora...

### Zadanie 1b
Aby policzyc ilosc dokumentow w kolekcji w konsoli mongo wpisalem:
```sh
db.Topics.count()
```
Wynik:
```sh
6.034.195
```

### Zadanie 1c
Poniższy skrypt uruchamiamy za pomocą komendy:

```sh
time skrypt1c.js

```

Czas wykonania to:
###MongoDB (2.6.4)

```sh
real,10m35.108s 
user,9m02.879s  
sys,0m10.058s
```
###MongoDB (2.8.0 RC)

```sh
real,18m01.898s 
user,16m02.553s  
sys,0m25.112s
```
```js
var database = db.Topics.find();

var tags = {};
var records = [];

var licznikTagów = 0;
var licznikUnikalnychTagów = 0;
var licznikRekordów = 0;

database.forEach(function(dokument) {
	licnzikRekordów++;
	var oddzieloneTagi = [];
	if(dokument.tags.constructor === String) {
		oddzieloneTagi = dokument.tags.split(" ");
	} 
	else { 
		oddzieloneTagi.push(dokument.tags);
	}
dokument.tags = oddzieloneTagi;
licznikTagów = oddzieloneTagi.length + licznikTagów;

for(var i=0; i<oddzieloneTagi.length; i++) {
	if(tags[oddzieloneTagi[i]] === undefined) {
		licznikUnikalnychTagów++;  
		tags[oddzieloneTagi[i]] = 1; 
	}
}

records.push(dokument);
if (licznikRekordów % 10000 === 0 || licznikRekordów === 6034195) {
	db.Train2c.insert(records);
	records = [];
	}
});

print("Tagi:" + licznikTagów);
print("Unikalne Tagi:" + licnzikUnikalnychTagów);
```
```sh
Tagi: 17409994
Unikalne Tagi: 42048
```

### Zadanie 1d
Importujemy do Mongo bazę z ważniejszymi miastami Polski. Czynimi to za pomocą poniższego polecenia: 
```sh
mongoimport -c places < polskieMiasta.json
```
Dodajemy geoindeks do kolekcji places:
```sh

db.places.ensureIndex({loc : "2dsphere"})
```
Zapytanie 1: Miasta oddalone od Warszawy o maksymalnie o 100 km:
```sh
db.polskieMiasta.find({loc: {$near: {$geometry: {type: "Point", coordinates: [21.000366210937496, 52.231163984032676]}, $maxDistance: 100000}}}).skip(1)
```
Mapka dla zapytania 1: [Geojson1](https://github.com/adrozdowski/NoSQL/blob/master/Images/Zapytanie1.geojson)

Zapytanie 2: Poniższe zapytanie prezentuje mniejwięcej kształt województwa świętokrzyskiego:
```js
db.polskieMiasta.find({loc: {$geoWithin: {$geometry: {type: "Polygon", coordinates: [[[20.390625,50.194484528931746], [20.247802734375,50.47498691278631], 
[19.791870117187496,50.548344490674786], [ 19.8577880859375, 50.666872321810715], [19.808349609375,50.830228205617445], [19.852294921874996,51.02412130394265], [20.0445556640625,50.965346321637696],[20.0115966796875,51.193115244645874],[20.28076171875,51.26191485308454],[20.423583984375, 51.334043778789415],[20.753173828125, 51.144894309328016],[ 21.1376953125,51.138001488062564],[21.5167236328125,51.0033856925319],[21.8353271484375,51.07591977545679],[21.829833984375,50.719069112580804],[20.7421875,50.240178884797025],[20.401611328125,50.1804159214388],[20.390625,50.194484528931746]]]}}}})
```
Mapka dla zapytania 2: [Geojson2](https://github.com/adrozdowski/NoSQL/blob/master/Images/Zapytanie2.geojson)

Zapytanie 3. Szukamy miast będących na południku 22.93121337890625:
```js
db.polskieMiasta.find({loc: {$geoIntersects: {$geometry: {type: "LineString", coordinates: [[22.93121337890625, -90],[22.93121337890625, 90]]}}}})
     
```
Mapka dla zapytania 3: [Geojson3](https://github.com/adrozdowski/NoSQL/blob/master/Images/Zapytanie3.geojson)

Zapytanie 4. Szuakmy trzech miast najbliższych Ostrowcowi Świętokrzyskiemu:
```js
db.polskieMiasta.find({loc: {$near: {$geometry: {type: "Point", coordinates: [21.38711929321289, 50.9393925139039]}}}}).limit(3)
```
Mapka dla zapytania 4: [Geojson4](https://github.com/adrozdowski/NoSQL/blob/master/Images/Zapytanie4.geojson)

Zapytanie 5. Reuzltat taki sam jak w zapytaniu 2. Użycie komendy $geoIntersect (co zachodzi w interakcję z polygonem)
```js
db.polskieMiasta.find({loc: {$geoIntersects: {$geometry: {type: "Polygon", coordinates: [[[19.259033203125, 52.3923633970718], [18.1768798828125, 51.17589926990911], 
[19.7259521484375, 50.86144411058924], [20.5059814453125, 51.50532341149335], [20.23681640625, 52.1166256737882], [19.259033203125, 52.3923633970718]]]}}}})
```
Mapka dla zapytania 5: [Geojson5](https://github.com/adrozdowski/NoSQL/blob/master/Images/Zapytanie2.geojson) 

Zapytanie 6. Miasta na drodze pomiędzy Gdańskiem a Zakopanem:
```js
db.polskieMiasta.find({loc: {$geoIntersects: {$geometry: {type: "LineString", coordinates: [ [[18.655128479003906,54.34815256064472]], [19.948768615722656,49.29803885147804]]}}}})
```
Mapka dla zapytania 6: [Geojson6](https://github.com/adrozdowski/NoSQL/blob/master/Images/Zapytanie6.geojson)
