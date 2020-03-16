# Mappings

Înainte de a introduce date în Elasticsearch, trebuie să ai un mapping care definește structura datelor încărcate. De cele mai multe ori, Elasticsearch va deduce ce tipuri de date folosești și va crea un mapping necesar.

Un mapping constituie o definire de schemă a datelor (*schema definition*). Aceast mapping îi spune lui Elasticsearch cum să introducă datele. Fii foarte atent pentru că începând cu versiunea 6 a lui Elasticsearch, `curl` are nevoie să-i specifici headerul neapărat: `-H "Content-Type: application/json"`.

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/numeindex -d '
{
    "mappings": {
        "numeindex": {
            "properties": {
                "year": {
                    "type":"date"
                }
            }
        }
    }
}'
```

Mapping-urile se folosesc în mai multe scopuri.

## Specificarea tipurilor de date

Poți specifica tipurile de date - `text`, `keyword`, `byte`, `long`, `short`, `integer`, `float`, `double`, `boolean`, `date`.

```json
"properties": {
    "id": {
        "type": "long"
    }
}
```

## Definirea indexării câmpurilor

Dacă dorești ca un câmp să fie indexabil, poți specifica acest lucru.

```json
"properties": {
    "genre": {
        "index": "false"
    }
}
```

Dacă este `true`, acest câmp va apărea în indexul inversat (*inverted index*).

## Cum sunt analizate câmpurile

La momentul în care sunt inițiate căutările pe câmpuri, rezultatele returnate pot fi modelate chiar de la bun început, atunci când este constituit `mapping`-ul pentru respectivele documente. În următorul exemplu, sunt definite câmpurile prin indicarea tipurilor lor și apoi sunt aplicate constrângerile de regăsire impuse prin analizoare.

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
{
  "mappings": {
    "properties": {
      "id": {"type": "integer"},
      "year": { "type":"date" },
      "genre": {"type": "keyword"},
      "title": {"type": "text", "analyzer": "english"}
    }
  }
}'
```

### Tipurile câmpurilor

#### `"type":"date"`

Este un câmp care așteaptă ca fosrmatul introdus să fie de tip Date.

#### `"type":"keyword"`

Acest tip specifică faptul că la momentul căutării, va trebui să fie trimis către Elasticsearch forma exactă a cuvântului sau sintagmei a cărui câmp poartă acest tip. Atenție, este și **case-sensitive**.

#### `"type": "text"`

Acest tip este ceva mai iertător în ceea ce privește rezultatele returnate, permițând și definirea unuor analizori.

### Analizoare

Poți defini filtrul prin care operează tokenizatorul și cum sunt operate token-urile: `standard`, `whitespace`, `simple`, `english`.

```json
"properties": {
    "description": {
        "analyzer": "english"
    }
}
```

Analizoarele permit filtrarea anumitor caractere, de exemplu să convertești din HTML encoding și invers.

## Crearea unui index

Pentru a constitui un index repede, se poate folosi *curl* din linie de comandă.

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
{
  "mappings": {
    "properties": {
      "year": { "type":"date" }
    }
  }
}'
```

În cazul în care deja ai scris într-un fișier mappingul pentru date, poți introduce direct fișierul.

```bash
curl -H 'Content-Type: application/json' -XPUT 'localhost:9200/shakespeare' --data-binary @shakes-mapping.json
```

Pentru a verifica ceea ce s-a petrecut, poți iniția din linia de comandă o interogare pe `_mapping`.

```bash
curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_mapping/movie
```

Răspunsul este simplu `{"movies":{"mappings":{"movie":{"properties":{"year":{"type":"date"}}}}}}`. Dacă dorești o versiune a răspunsului într-o formă mai lizibilă, pui și elementul de interogare `pretty` în linia de comandă.

```bash
curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_mapping/movie?pretty
```

Răspunsul va fi formatat.

```json
{
  "movies" : {
    "mappings" : {
      "movie" : {
        "properties" : {
          "year" : {
            "type" : "date"
          }
        }
      }
    }
  }
}
```

## Introducerea datelor după mapping

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies/_doc/109487 -d '
{
  "genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar",
  "year": 2014
}'
```
Răspunsul trebuie să fie ceva asemănător cu: `{"_index":"movies","_type":"_doc","_id":"109487","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}`.

Caută înregistrarea cu `curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search?pretty`.

## Vizualizarea structurii unui index după ce a fost introdus

```bash
curl -H 'Content-Type: application/json' -XGET 'localhost:9200/shakespeare/?pretty'
```

## Parent - child mapping

Să presupunem că în cazul unei francize de film așa cum este Star Wars avem mai multe filme. Pentru fi mai rapid în ceea ce privește aducerea documentelor cu cât mai multe informații, fără a interoga de două ori baza, odată pentru franchise și apoi un matching pe copiii care au setată respectiva franciza, se poate proceda la o denormalizare a datelor în sensul că pentru fiecare film putem introduce și franciza și astfel, nu facem două interogări. Acesta este cazul ținerii datelor într-o formă denormalizată. Spațiul de depozitare este ieftin și nu mai trebuie făcute normalizări la sânge în defavoarea timpilor de acces și numărul de atingeri ale bazei.

Totuși, Elasticsearch permite relații parent -child, dacă se dorește realizarea unui astfel de index.

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/series -d '{
  "mappings": {
    "properties": {
      "film_to_franchise": {
        "type": "join",
        "relations": {
          "franchise": "film"
        }
      }
    }
  }
}'
```
Cu răspunsul `{"acknowledged":true,"shards_acknowledged":true,"index":"series"}`.

*Franchise* va avea drept copii *films*.

Putem introduce un set corespondent asemănător cu următorul:

```json
{ "create" : { "_index" : "series", "_id" : "1", "routing" : 1} }
{ "id": "1", "film_to_franchise": {"name": "franchise"}, "title" : "Star Wars" }

{ "create" : { "_index" : "series", "_id" : "260", "routing" : 1} }
{ "id": "260", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode IV - A New Hope", "year":"1977" , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "series", "_id" : "1196", "routing" : 1} }
{ "id": "1196", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode V - The Empire Strikes Back", "year":"1980" , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "series", "_id" : "1210", "routing" : 1} }
{ "id": "1210", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode VI - Return of the Jedi", "year":"1983" , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "series", "_id" : "2628", "routing" : 1} }
{ "id": "2628", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode I - The Phantom Menace", "year":"1999" , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "series",  "_id" : "5378", "routing" : 1} }
{ "id": "5378", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode II - Attack of the Clones", "year":"2002" , "genre":["Action", "Adventure", "Sci-Fi", "IMAX"] }
{ "create" : { "_index" : "series", "_id" : "33493", "routing" : 1} }
{ "id": "33493", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode III - Revenge of the Sith", "year":"2005" , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "series", "_id" : "122886", "routing" : 1} }
{ "id": "122886", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode VII - The Force Awakens", "year":"2015" , "genre":["Action", "Adventure", "Fantasy", "Sci-Fi", "IMAX"] }
```

Mai întâi se va introduce franciza, iar ulterior fiecare câmp cu filmele care țin de franciză, se vor introduce cu toate datele fiecăruia, dar cu specificația că în câmpul `film_to_franchise` se va introduce un câmp `parent` a cărui valoare este cea a francizei: `film_to_franchise": {"name": "film", "parent": "1"}`.

Introdu setul întreg.

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/_bulk?pretty --data-binary @series.json
```

În ceea ce privește căutarea, pentru datele care au setate cardinalități așa cum este cel al francizei cu mai multe filme, căutarea se va face folosind cuvântul cheie `has_parent`.

```bash
curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/series/_search?pretty -d '
{
  "query": {
    "has_parent": {
      "parent_type": "franchise",
      "query": {
        "match": {
          "title": "Star Wars"
        }
      }
    }
  }
}'
```

O astfel de interogare centrată pe numele francizei, va aduce toate documentele pentru care este părinte.

Aici există o notă importantă care poate fi utilă. În cazul specificării lui `parent_type` putem avea mai mulți părinți.

În cazul în care dorești să găsești un copil, interogarea se va centra pe copil.

```bash
curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/series/_search?pretty -d '
{
  "query": {
    "has_child": {
      "type": "film",
      "query": {
        "match": {
          "title": "The Force Awakens"
        }
      }
    }
  }
}'
```

Este returnat un rezultat asemănător cu următorul obiect:

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "series",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "id" : "1",
          "film_to_franchise" : {
            "name" : "franchise"
          },
          "title" : "Star Wars"
        }
      }
    ]
  }
}
```