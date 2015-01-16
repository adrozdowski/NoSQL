# Zadania z NoSQL

Specyfikacja techniczna komputera:

| Rodzaj komponentu    | Komponent                       |
|-----------------------|---------------------------------|
| System operacyjny     | Linux Kubuntu 14.04 (64-bitowy) |
| Procesor              | Intel i5 4590                   |
| Ilość rdzeni          | 2                               |
| Ilość Ram             | 4 GB                            |
| Rodzaj dysku twardego | HDD                             |



### Zadanie 3 - Anagramy

Aby zaimportowac plik word_list.txt wykonujemy polecenie:
```sh
mongoimport -c words --type csv --file word_list.txt -f "word"
```
Czas importu: 0m0.998s

![alt text](https://raw.githubusercontent.com/adrozdowski/NoSQL/master/anagramyimport.png "")

Anagramy tworzymy za pomoca:
```js
var m = function() {
	emit(Array.sum(this.word.split("").sort()), this.word);
};
var r = function(key, values) {
	return values.toString();
};

db.words.mapReduce(
  m,
  r,
  {
    query: {},
    out: "anagramy"
  }
)
```
Czas mapreduce: 0m3.033s

![alt text](https://raw.githubusercontent.com/adrozdowski/NoSQL/master/anagramymap.png "")

W kolecji 'anagramy' sa pary - posortowane litery - slowa zawierajace te litery.

```sh
{ "_id" : "aaabcl", "value" : "cabala" }
{ "_id" : "aaabcn", "value" : "cabana" }
{ "_id" : "aaabcs", "value" : "casaba" }
{ "_id" : "aaabnn", "value" : "banana" }
{ "_id" : "aaabrz", "value" : "bazaar" }
{ "_id" : "aaacci", "value" : "acacia" }
{ "_id" : "aaaclp", "value" : "alpaca" }
{ "_id" : "aaacmr", "value" : "maraca" }
{ "_id" : "aaadmr", "value" : "armada" }
{ "_id" : "aaaelz", "value" : "azalea" }
{ "_id" : "aaaitx", "value" : "ataxia" }
{ "_id" : "aaajmp", "value" : "pajama" }
{ "_id" : "aaalms", "value" : "salaam" }
{ "_id" : "aaamnn", "value" : "manana" }
{ "_id" : "aaamnp", "value" : "panama" }
{ "_id" : "aaappy", "value" : "papaya" }
{ "_id" : "aaartv", "value" : "avatar" }
{ "_id" : "aabbbo", "value" : "baobab" }
{ "_id" : "aabblo", "value" : "balboa" }
{ "_id" : "aabcls", "value" : "cabals" }
Type "it" for more
> it
{ "_id" : "aabcrs", "value" : "scarab" }
{ "_id" : "aabcsu", "value" : "abacus" }
{ "_id" : "aabder", "value" : "abrade" }
{ "_id" : "aabdes", "value" : "abased" }
{ "_id" : "aabdet", "value" : "abated" }
{ "_id" : "aabdll", "value" : "ballad" }
{ "_id" : "aabdlm", "value" : "lambda" }
{ "_id" : "aabdmn", "value" : "badman" }
{ "_id" : "aabdor", "value" : "abroad,aboard" }
{ "_id" : "aabegt", "value" : "teabag" }
{ "_id" : "aabelr", "value" : "arable" }
{ "_id" : "aabelt", "value" : "ablate" }
{ "_id" : "aabelz", "value" : "ablaze" }
{ "_id" : "aabemo", "value" : "amoeba" }
{ "_id" : "aabess", "value" : "abases" }
{ "_id" : "aabest", "value" : "abates" }
{ "_id" : "aabggr", "value" : "ragbag" }
{ "_id" : "aabggs", "value" : "gasbag" }
{ "_id" : "aabgin", "value" : "baaing" }
{ "_id" : "aabgir", "value" : "airbag" }
Type "it" for more
```

### Zadanie 3 - Wikipedia

Do skonwertowania xmla to jsona stworzyłem program([link](https://github.com/klatoszewski/nosql/tree/master/konwerter)) w c++ z wykorzystaniem biblioteki([link](https://github.com/Cheedoong/xml2json)).

Kod, który użyłem do konwersji wygląda następująco:

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <string>
#include <sstream>
#include <fstream>
#include <sys/time.h>
#include <time.h>

#include "xml2json.hpp"

using namespace std;

int64_t getCurrentTime()
{
	struct timeval tval;
	gettimeofday(&tval, NULL);
	return tval.tv_sec * 1000000LL + tval.tv_usec;
}
enum Stany
{
    SZUKAM = 0,
    TWORZE = 1,

};

int main(int argc, const char *argv[])
{
int licznik = 0;
Stany stan = SZUKAM;
string znacznikStart = "<page";
string znacznikKoniec = "</page>";
string fragment = "";
ifstream fin;
fin.open("plwiki-20141209-pages-articles-multistream.xml", ios::in);
ofstream myfile;
myfile.open ("my-input-file.txt");
char my_character ;

	while (!fin.eof() )
    {
	fin.get(my_character);
	fragment += my_character;
	if(stan == SZUKAM && fragment.length() >= znacznikStart.length() && znacznikStart == fragment.substr (fragment.length()-znacznikStart.length(),znacznikStart.length()))
    {



        fragment = znacznikStart;
        stan = TWORZE;
    }
    else if(stan == TWORZE && fragment.length() >= znacznikKoniec.length() && znacznikKoniec == fragment.substr (fragment.length()-znacznikKoniec.length(),znacznikKoniec.length()))
    {
        if(licznik%1000 == 0)cout << "licznik: " << licznik << " pozycja w pliku: " << fin.tellg()/1048576 << endl;
        licznik++;
        myfile << xml2json(fragment.c_str()) << "\n";
        fragment = "";
    }

	}
    fin.close();
    myfile.close();
    cout << "Koniec\n";
	return 0;
}
```
Otrzymałem olbrzymi plik w formacie .json, który do mongo wczytałem poniższym poleceniem:

```sh
mongoimport --c wikipedia -d test --file wikipedia.json
```

Czas importu: 90m 33.665s

![alt text](https://raw.githubusercontent.com/adrozdowski/NoSQL/master/wikiimport.png "")

Skrypt:

```js
var m = function() {
    var array = this.page.revision.text.tekst.match(/([a-zA-Z]+|[^\x00-\x7F]+)+/g);
    if(array != null) array.forEach(function(word) {
        emit(word, 1);
    });
};
var r = function(key, values) {
    return Array.sum(values);
};

db.wikipedia.mapReduce(
  m,
  r,
  {
    query: {},
    out: "wikipediaSlowa"
  }
)
```
Czas mapreduce: 140m 28.112s

![alt text](https://raw.githubusercontent.com/adrozdowski/NoSQL/master/wikimap.png "")

Częstotliwość występowania słów:

![alt text](https://raw.githubusercontent.com/adrozdowski/NoSQL/master/wykres.png "")

