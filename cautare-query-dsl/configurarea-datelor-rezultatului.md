# Formatarea rezultatelor

## Precizarea formatului de date

În cazul în care este nevoie ca datele să poată fi exportate în alt format decât obiectul JSON obișnuit, acest lucru poate fi precizat în cererea de căutare.

```yaml
 GET /movies/_search?format=yaml
 {
   "query":{
     "match_all": {}
   }
 }
 ```

## Selecția câmpurilor

Poți activa și/sau dezactiva datele câmpurilor din obiectul returnat în urma unei căutări. Acest lucru se face setând câmpurile pentru care nu avem nevoie de date la valoarea `false`.

```yaml
 GET /movies/_search
 {
   "_source": false,
   "query":{
     "term": {
       "title": {
         "value": "star"
       }
     }
   }
 }
 ```

 Vei obține un rezultat asemănător cu obiectul următor:

 ```json
 {
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.919734,
    "hits" : [
      {
        "_index" : "movies",
        "_type" : "_doc",
        "_id" : "135569",
        "_score" : 0.919734
      },
      {
        "_index" : "movies",
        "_type" : "_doc",
        "_id" : "122886",
        "_score" : 0.666854
      }
    ]
  }
}
```

Pentru a aduce datele unui singur câmp, îi vom da lui `"_source"` numele câmpului pentru care dorim valoarea.

```yaml
 GET /movies/_search
 {
   "_source": "title",
   "query":{
     "term": {
       "title": {
         "value": "star"
       }
     }
   }
 }
 ```

 cu un posibil rezultat asemănător cu:

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
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.919734,
    "hits" : [
      {
        "_index" : "movies",
        "_type" : "_doc",
        "_id" : "135569",
        "_score" : 0.919734,
        "_source" : {
          "title" : "Star Trek Beyond"
        }
      },
      {
        "_index" : "movies",
        "_type" : "_doc",
        "_id" : "122886",
        "_score" : 0.666854,
        "_source" : {
          "title" : "Star Wars: Episode VII - The Force Awakens"
        }
      }
    ]
  }
}
```

Pentru cazul în care ai obiecte, la `"_source"` vei da drept valoare calea către respectivul câmp.

```yaml
 GET /movies/_search
 {
   "_source": "ingrediente.nume",
   "query":{
     "term": {
       "title": {
         "value": "ciorbă"
       }
     }
   }
 }
 ```

 Pentru a obține toate proprietățile unui obiect al unui document, se poate pune wildcard-ul steluță:

 ```yaml
 GET /movies/_search
 {
   "_source": "ingrediente.*",
   "query":{
     "term": {
       "title": {
         "value": "ciorbă"
       }
     }
   }
 }
 ```

 Poți face chiar o selecție a datelor pe care dorești să le păstrezi în rezultat. Pentru a face acest lucru, vei constitui un array al câmpurilor din document pe care le dorești în setul final.

  ```yaml
 GET /movies/_search
 {
   "_source": ["ingrediente.*", "descriere"],
   "query":{
     "term": {
       "title": {
         "value": "ciorbă"
       }
     }
   }
 }
 ```

Pentru a configura deplin datele pe care dorești să le aduci, poți configura un obiect pentru proprietatea `_source`. De exemplu, vrem să aducem informații despre ingrediente, dar fără numele ingredientului.

  ```yaml
 GET /movies/_search
 {
   "_source": {
     "includes": "ingrediente.*",
     "excludes": "ingrediente.nume"
   },
   "query":{
     "term": {
       "title": {
         "value": "ciorbă"
       }
     }
   }
 }
 ```

## Precizarea dimensiunii setului

Atunci când ai nevoie doar de un subset al rezultatelor, poți preciza numărul documentelor returnate. Acest lucru poate fi util în scenarii în care avem nevoie de paginare, de exemplu. Setul returnat va conține și numărul total de documente `"total":9`, de exemplu.

Poți preciza dimensiunea drept query parameter sau poți să o introduci în corpul cererii.

```yaml
 GET /movies/_search?size=2
 {
   "_source": false,
   "query":{
     "term": {
       "title": {
         "value": "star"
       }
     }
   }
 }
 ```

sau

```yaml
 GET /movies/_search
 {
   "size": 2,
   "_source": false,
   "query":{
     "term": {
       "title": {
         "value": "star"
       }
     }
   }
 }
 ```

Numărul de rezultate dintr-un set returnat este setat din oficiu la 10.

## Setarea unui offset

```yaml
 GET /movies/_search
 {
   "size": 2,
   "from": 0,
   "_source": false,
   "query":{
     "term": {
       "title": {
         "value": "star"
       }
     }
   }
 }
 ```

## Sortarea simplă a rezultatelor

Mai întâi alegi câmpul după care dorești să faci sortarea.

```yaml
GET /movies/_search
{
  "_source": ["year"],
  "query":{
    "match": {
      "title": "star"
    }
  },
  "sort": {
    "year": {
      "order":"desc"
    }
  }
}
 ```

Atunci când dorești să faci sortarea în funcție de mai multe criterii, vei constitui un array pentru proprietatea `sort`.

```yaml
GET /movies/_search
{
  "_source": ["year"],
  "query":{
    "match": {
      "title": "star"
    }
  },
  "sort": [
    {"year": "desc"},
    {"title": "asc"}
  ]
}
 ```

## Sortarea după câmpuri cu mai multe valori

Acest lucru este foarte util pentru a ordona în funcție de un câmp care are o listă de valori așa cum este cazul aprecierilor publicului sau numărul de like-uri, etc. De exemplu, pentru un câmp care are un array cu valori ale aprecierilor independente, se va face o medie prin folosirea modului `avg ` (average),

```yaml
GET /movies/_search
{
  "_source": ["year"],
  "query":{
    "match": {
      "title": "star"
    }
  },
  "sort": [
    {
      "ratings": {
        "order": "desc",
        "mode": "avg"
        }
    }
  ]
}
 ```

## Filtre

Fitrele oferă un răspuns în funcție de concluzia pozitivă sau negativă în urma evaluării. Filtrele nu afectează scorurile privind relevanța. Acestea sunt folosite, de obicei pentru numere și date calendaristice. Elastisearch face caching la filtre.
