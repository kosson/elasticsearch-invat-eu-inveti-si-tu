# multi_match

Aceasta este o interogare care permite căutarea în mai multe câmpuri deodată. În exemplul de mai jos, căutăm valoarea `nero` în câmpurile `title` și `description`.

```text
GET /resursedus0/_search
{
  "query": {
    "multi_match": {
      "query": "nero",
      "fields": ["title", "description"],
      "fuzziness": 1
    }
  }
}
```

Într-o colecție reală, rularea acestei interogări aduce un posibil rezultat sub forma următoare:

```json
{
  "took" : 33,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 17,
      "relation" : "eq"
    },
    "max_score" : 6.050819,
    "hits" : [
      {
        "_index" : "resursedus0",
        "_type" : "_doc",
        "_id" : "5ec3a2ee072c8a374a970ad9",
        "_score" : 6.050819,
        "_source" : {
          "title" : "Nerva",
          "description" : "Nerva (/ˈnɜːrvə/; Marcus Cocceius Nerva Caesar Augustus;[1] 8 November 30 – 27 January 98) was Roman emperor from 96 to 98. Nerva became emperor when aged almost 66, after a lifetime of imperial service under Nero and the rulers of the Flavian dynasty. Under Nero, he was a member of the imperial entourage and played a vital part in exposing the Pisonian conspiracy of 65. Later, as a loyalist to the Flavians, he attained consulships in 71 and 90 during the reigns of Vespasian and Domitian, respectively. ",
        }
      },
      {
        "_index" : "resursedus0",
        "_type" : "_doc",
        "_id" : "619f7c14c6a48b29ec90305d",
        "_score" : 4.5258293,
        "_source" : {
          "title" : "25 noiembrie test",
          "description" : "Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.",
        }
      },
      {
        "_index" : "resursedus0",
        "_type" : "_doc",
        "_id" : "5eb0d9e5188020563a6380c1",
        "_score" : 4.3515472,
        "_source" : {
          "title" : "Nero",
          "description" : "Nero Claudius Caesar Augustus Germanicus (n. 15 decembrie 37, Anzio – d. 9[3] sau 11 iunie[4] 68, Roma) a fost din 54 e.n. până în 68 e.n. al cincilea împărat roman al dinastiei Iulio-Claudiene.",
        }
      }
    ]
  }
}
```

Rezultatele aduse sunt mult peste ceea ce am căutat strict pentru că în afara cuvântului în sine, vor fi căutate și cuvinte care încep cu silaba și dacă nu găsește, doar cu litera de început.

O altă problemă interesantă apare atunci când folosim o sintagmă. De exemplu `viermi lați` în:

```text
GET /resursedus0/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "multi_match": {
            "query": "viermi lați",
            "type": "best_fields",
            "fields": ["title", "description"]
          }
        }
      ],
      "should": [
        {
          "multi_match": {
            "query": "viermi lați",
            "type": "most_fields",
            "fields": ["tile", "description"]
          }
        }
      ]
    }
  },
  "_source": ["title", "description"]
}
```

Elasticsearch face un OR între cele două cuvinte. Căutarea se va face astfel: *caută `viermi` ORI `lați` în `must` pe câmpurile menționate*. Când cauți ori una, ori alta, vor fi aduse multe rezultate. Poți modifica acest lucru restricționând chiar la sintagma care este căutată specificând operatorul care va fi folosit între cuvintele sintagmei de căutare:

```json
  "multi_match": {
    "query": "viermi lați",
    "type": "best_fields",
    "fields": ["title", "description"],
    "operator": "and"
  }
```

Valoarea relevanței poate fi crescută pentru un anumit document dacă șirul de căutat se află într-un câmp pentru care este specificată creșterea.

```text
GET /resursedus0/_search
{
  "query": {
    "multi_match": {
      "query": "viermi",
      "type": "most_fields",
      "fields": ["title^1", "description"]
    }
  },
  "_source": [
    "title", "description"
  ]
}
```

### Tipul cross_fields

Această setare indică faptul că vei căuta în mai multe câmpuri ca și cum ar fi doar unul singur. Un lucru similar cu rezultate mai bune ar fi ca la momentul ingest-ului să aplici un `copy_to` pentru a constitui un câmp cu valorile combinate ale mai multora. Astfel, vei face interogarea unui singur câmp.

## Resurse

- [Multi-match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html)
- [Multi Match Query with elasticsearch – Influence scoring – part 3 | JÉRÉMY GACHET | 12/03/2020](https://spoon-elastic.com/all-elastic-search-post/multi-match-query-elasticsearch-scoring-part-3/)
- [Understanding Elasticsearch Combined Fields And Multi Match Queries | Alexander Reelsen | May 28, 2021](https://spinscale.de/posts/2021-05-28-understanding-elasticsearch-combined-fields-multi-match-queries.html)
