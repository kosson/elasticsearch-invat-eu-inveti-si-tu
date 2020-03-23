# Mappings

Înainte de a introduce date în Elasticsearch, trebuie să ai un mapping care definește structura datelor încărcate. Poți să te gândești la mapping ca la un plan de construcție a unui index. Fiecare index are un *mapping type* care **determină cum vor fi indexate documentele**.

De cele mai multe ori, Elasticsearch va deduce ce tipuri de date folosești și va crea un mapping necesar. Adu-ți aminte că pentru a face mapping-ul ai nevoie ca indexul să existe: `PUT /movies`, de exemplu.

Un mapping constituie o definire de schemă a datelor (*schema definition*). Aceast mapping îi spune lui Elasticsearch cum să introducă datele pentru a fi indexate. Un *mapping type* este compus din:

- Meta-fields
- Fields

Meta-fields sunt câmpuri folosite pentru a atașa metadate necesare descrierii unui document în interiorul indexului. Câmpurile sunt:

- `_index`,
- `_type`,
- `_id`,
- `_source`.

Fields (*câmpuri*) sunt proprietățile documentului în sine. Fiecare câmp va înmagazina un anumit tip de date. Cele simple sunt:

- `text`,
- `keyword`,
- `date`,
- `long`,
- `double`,
- `boolean`,
- `ip`.

Apoi pot fi obiecte care suportă structura ierarhică a unui obiect JSON precum `object` și `nested`.

Sunt și tipuri de câmpuri specializate precum `geo_point`, `geo_shape` sau `completion`.

Fii foarte atent pentru că începând cu versiunea 6 a lui Elasticsearch, `curl` are nevoie să-i specifici headerul neapărat: `-H "Content-Type: application/json"`.

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

În exemplul de mai sus, maparea propriu-zisă se face în obiectul care este valoarea lui `properties`. Trebuie indicat și faptul că pentru a introduce obiectul de mapare în Elasticsearch pentru un anumit index, se va folosi verbul `PUT`. În cazul în care aveți la îndemână Kibana, puteți introduce mapările folosind `Dev Tools`. După introducerea obiectului de mapare, o interogare cu `GET` pe endpointul `_mappings` a indexului dorit. De exemplu:

```yaml
PUT /movies/_mapping
{
  "properties": {
      "id": {"type": "integer"},
      "year": { "type":"date" },
      "genre": {"type": "keyword"},
      "title": {"type": "text", "analyzer": "english"}
  }
}
```

Pentru a observa ce mappinguri au fost făcute, se poate rula din Dev Tools (Kibana) comanda `GET /movies/_mapping`. Obiectul de răspuns poate fi similar cu:

```json
{
  "movies" : {
    "mappings" : {
      "properties" : {
        "genre" : {
          "type" : "keyword"
        },
        "id" : {
          "type" : "integer"
        },
        "title" : {
          "type" : "text",
          "analyzer" : "english"
        },
        "year" : {
          "type" : "date"
        }
      }
    }
  }
}
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

## Parametrii mapping-ului

Parametrii pot fi specificați pentru fiecare câmp indicând comportamentul Elasticsearch atunci când primește date pentru a le transforma în documente ale indexului.

### `coerce`

Acest parametru când are valoarea `true`, va *converti* valorile care seamnănă a fi numere în numere. O valoare venită ca string `"10.2"`, va fi convertită la `10.2`.

### `copy_to`

Permite crearea de câmpuri custom. Acest parametru îi specifică lui Elasticsearch să copieze o valoare într-un anumit câmp.

```json
{
  "nume": {
    "type":"text",
    "copy_to":"numecomplet"
  },
  "prenume": {
    "type":"text",
    "copy_to":"numecomplet"
  },
  "numecomplet": {
    "type":"text"
  }
}
```

### `dynamic`

Activează sau dezactivează adăugarea de câmpuri documentelor sau obiectelor imbricate în mod dinamic.

```json
{
  "mappings": {
    "default": {
      "dynamic": false,
      "properties":{
        "nume":{
          "dynamic": true,
          "properties": {}
        }
      }
    }
  }
}
```

### `norms`

Dacă `true`, va marca respectivul câmp ca fiind relevant pentru a calcula scorul de relevanță. Nu poți activa `norms` mai târziu fără a reface indexul.

### `format`

Definește formatul datelor calendaristice. Posibilele valori sunt:

- "yyyy-MM-dd",
- "epoch_millis",
- "epoch_second".

Valoarea din oficiu este "strict_date_optional_time||epoch_millis".

### `null_value`

Dacă unul din câmpuri primește o valoare `null` la venirea datelor, valoarea acestuia va fi completată cu valoarea lui `null_value`.

```json
{
  "properties": {
    "likes": {
      "type": "integer",
      "null_value":0
    }
  }
}
```

### `fields`

Acest parametru este folosit pentru a indexa câmpurile în diferite feluri.

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

Este un câmp care așteaptă ca formatul introdus să fie de tip `Date`.

#### `"type":"keyword"`

Acest tip specifică faptul că la momentul căutării, va trebui să fie trimis către Elasticsearch forma exactă a cuvântului sau sintagmei a cărui câmp poartă acest tip. Atenție, este și **case-sensitive**.

#### `"type": "text"`

Acest tip este ceva mai iertător în ceea ce privește rezultatele returnate, permițând și definirea unor analizori.

## Modificarea unui mapping

Modificarea mapărilor pentru câmpurile existente pentru care s-a făcut mapare deja, nu se mai poate face. Încercarea de a face acest lucru se va solda cu eșec. Pentru a face acest lucru, ar trebui să ștergem indexul, să facem unul noi cu noi mapări și apoi să reindexăm datele.

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



## Resurse

- [Mapping | Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping.html)
- [Removal of mapping types | Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/removal-of-types.html)