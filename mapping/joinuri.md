## Parent - child mapping

Să presupunem că în cazul unei francize de film așa cum este Star Wars avem mai multe filme. Pentru fi mai rapid în ceea ce privește aducerea documentelor cu cât mai multe informații, fără a interoga de două ori baza, odată pentru franchise și apoi un matching pe copiii care au setată respectiva franciza, se poate proceda la o denormalizare a datelor în sensul că pentru fiecare film putem introduce și franciza și astfel, nu facem două interogări. Acesta este cazul ținerii datelor într-o formă denormalizată. Spațiul de depozitare este ieftin și nu mai trebuie făcute normalizări la sânge în defavoarea timpilor de acces și numărul de atingeri ale bazei.

Totuși, Elasticsearch permite relații parent-child, dacă se dorește realizarea unui astfel de index. Limitările join-urilor este că toate documentele trebuie să stea în același index. Nu poți face join-uri pe documente din indexuri diferite. Documentele părinte și cele copil trebuie să fie în același shard. Nu poți avea decât un singur câmp de join per index. Un document poate avea un singur părinte. Join-urile taxează performața pentru că au nevoie de resurse de calcul alocate.

Poți folosi join-urile în cazul în care ai documente care nu se modifică prea mult, de exemple rețete ca părinte și ingrediente drept copii. Adu-ți mereu aminte că Elasticsearch nu este o bază de date relațională.

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

Mai întâi se va introduce franciza, iar ulterior fiecare câmp cu filmele care țin de franciză, se vor introduce cu toate datele fiecăruia, dar cu specificația că în câmpul `film_to_franchise` se va introduce un câmp `parent` a cărui valoare este cea a francizei: `film_to_franchise": {"name": "film", "parent": "1"}`. Mai există un element care apare, acesta fiind `"routing":1`. Raouting-ul indică calea către shard-ul în care este stocat documentul cu un anumit id. Routing se dovedește util atunci când sunt indexate documente noi, dar și când acestea sunt căutate. Comportamentul din oficiu este să fie luat drept valoare de rutare, id-ul documentului care este prelucrat de o funcție de hashing. Documentele părinte și documentele copil trebuie să stea pe același shard.

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

### Părinți multipli pentru join-uri

Mai întâi de toate setezi mapping-ul pentru indexul părinților. Apoi creezi vreo doi părinți. Apoi copii. La routing, menționezi id-ul părintelui cu care legi copilul.

```yaml
PUT /unit
{
  "mappings": {
    "properties": {
      "legatura_campuri": {
        "type": "join",
        "relations": {
          "unit":"user"
        }
      }
    }
  }
}

PUT /unit/_doc/1
{
  "institutie": "Inspectoratul Școlar București",
  "legatura_campuri":"unit"
}

PUT /unit/_doc/2
{
  "institutie": "Casa Corpului Didactic Focșani",
  "legatura_campuri":"unit"
}

PUT /unit/_doc/3
{
  "nume": "Ioana Postelnicu",
  "email": "i.posi@telos.com",
  "creat_la": "2020/02/22",
  "admin": true,
  "legatura_campuri": {
    "name": "user",
    "parent": 1,
    "routing": 1
  }
}

PUT /unit/_doc/3
{
  "nume": "Jean Martin",
  "email": "jeanm@telos.com",
  "creat_la": "2020/02/23",
  "admin": false,
  "legatura_campuri": {
    "name": "user",
    "parent": 2,
    "routing": 2
  }
}
```

Ca să extragi documentele legate de un părinte, vei iniția un query asemănător cu:

```yaml
GET /unit/_search
{
  "query": {
    "parent_id": {
      "type": "user",
      "id": 1
    }
  }
}
```

Pentru stabilirea relației și generarea datelor se va constitui un query de tip `parent_id`, un obiect a cărei primă proprietate este `type` ce poartă numele copiilor așa cum a fost definit în relație la momentul mapping-ului. A doua proprietate va fi `id`, care este id-ul părintelui.

Ce faci în cazul în care nu cunoști id-ul părintelui? Sau poate dorești să introduci condiții suplimentare. În acest caz se va folosi proprietatea `has_parent` și implică niște condiții pe care un părinte trebuie să le întruneacă pentru a putea să-i găsim copiii.

```yaml
GET /unit/_search
{
  "query": {
    "has_parent": {
      "parent_type": "unit",
      "score": true,
      "query": {
        "term": {
          "institutie": {
            "value": "Inspectoratul Școlar București"
          }
        }
      }
    }
  }
}
```

Pentru toate documentele copil, din oficiu va fi impusă valoarea 1. Dacă adăugăm opțiunea `"score": true`, la acest scor se va adăuga scorul părintelui.

Dacă dorești să extragi documentele părinte în baza unor criterii pe care copiii trebuie să le satisfacă, se poate folosi proprietatea `has_child`. Propriu-zis vei căuta structurile părinte de care țin copiii cu anumite caracteristici.

```yaml
GET /unit/_search
{
  "query": {
    "has_child": {
      "parent_type": "user",
      "score": true,
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "creat_la": {
                  "gte": "2020/02/22"
                }
              }
            }
          ],
          "should": {
            "term": {
              "admin": true
            }
          }
        }
      }
    }
  }
}
```

Dacă ai adăuga și proprietățile `min_children` și `max_children` în obiectul lui `has_child`, poți limita numerele copiilor pe care unitățile le-ar putea avea. Dacă numărul de unități nu au useri între limitele setate de min și max children, nu vor apărea în rezultate.

## Relații pe mai multe niveluri

În cazul în care ai o ierarhie de posibili părinți, de exemplu școală->[clasa1,clasa2]->elev, poți modela relații și în aceste scenarii.

```yaml
PUT /unitate
{
  "mappings": {
    "properties": {
      "uneste_campuri": {
        "type": "join",
        "relations": {
          "scoala": ["clasa","atelier"],
          "clasa": "elev"
        }
      }
    }
  }
}

POST /unitate/_doc/1?refresh
{
  "nume": "Scoala Generală Nr. 12 „George Enea”",
  "uneste_campuri": "scoala"
}

PUT /unitate/_doc/2?routing=1&refresh
{
  "nume": "Clasa I A",
  "uneste_campuri": {
    "name": "clasa",
    "parent": 1
  }
}

PUT /unitate/_doc/3?routing=1
{
  "nume": "Ionela Panait",
  "varsta": 9,
  "uneste_campuri": {
    "name": "elev",
    "parent": 2
  }
}
```

Observă faptul că la adăugarea unui elev, routing-ul a fost pus pe 1, fapt care indică bunicul unității în care va sta pentru a indica shard-ul corect, Observă că la fel face și scoala, indică părintele direct. Putem spune că înregistrările care au rădăcina comună, trebuie să indice prin `routing` id-ul corect al acestuia.
Proprietatea `parent` va indica id-ul unității din care face parte, în cazul nostru, clasa 1 A (`clasa1a`).

Să mai creăm o scoală cu o clasă:

```yaml
PUT /unitate/_doc/4?refresh
{
  "nume": "Scoala Generală Nr. 119 „Ecaterina Teodoroiu”",
  "uneste_campuri": "scoala"
}

PUT /unitate/_doc/5?routing=1&refresh
{
  "nume": "Clasa I A",
  "uneste_campuri": {
    "name": "clasa",
    "parent": 4
  }
}

PUT /unitate/_doc/6?routing=1&refresh
{
  "nume": "Adrian Valentin",
  "varsta": 9,
  "uneste_campuri": {
    "name": "elev",
    "parent": 5
  }
}
```

Am creat o școală nouă care are id-ul 4. În această școală există clasa I A și în această clasă este un elev.

Să căutăm unul dintre elevi pentru a vedea în ce unitate școlară se află.

```yaml
GET /unitate/_search
{
  "query": {
    "has_child": {
      "type": "clasa",
      "query": {
        "has_child": {
          "type": "elev",
          "query": {
            "match": {
              "nume": "Adrian Valentin"
            }
          }
        }
      }
    }
  }
}
```

Răspunsul este similar cu:

```json
{
  "took" : 7,
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
        "_index" : "unitate",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 1.0,
        "_source" : {
          "nume" : "Scoala Generală Nr. 119 „Ecaterina Teodoroiu”",
          "uneste_campuri" : "scoala"
        }
      }
    ]
  }
}
```

Pentru a vedea și cum poți obține date din relațiile directe (`inner_hits`), mai adăugăm doi elevi.

```yaml
PUT /unitate/_doc/3?routing=1&refresh
{
  "nume": "Ionică Vasilescu",
  "varsta": 13,
  "sex": "M",
  "uneste_campuri": {
    "name": "elev",
    "parent": 2
  }
}

PUT /unitate/_doc/3?routing=1&refresh
{
  "nume": "Fănel Aglutinel",
  "varsta": 13,
  "sex": "M",
  "uneste_campuri": {
    "name": "elev",
    "parent": 5
  }
}
```

Ceea ce dorim să aflăm este câți elevi care satisfac cerințele interogării pot fi găsiți și in care clasă sunt aceștia. Modelăm interogarea astfel:

```yaml
GET /unitate/_search
{
  "query": {
    "has_child": {
      "type": "elev",
      "inner_hits": {},
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "varsta": {
                  "gte": 10,
                  "lte": 20
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "sex": {
                  "value": "M"
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

Și obținem un răspuns similar cu:

```json
{
  "took" : 74,
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
        "_index" : "unitate",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "nume" : "Clasa I A",
          "uneste_campuri" : {
            "name" : "clasa",
            "parent" : 4
          }
        },
        "inner_hits" : {
          "elev" : {
            "hits" : {
              "total" : {
                "value" : 1,
                "relation" : "eq"
              },
              "max_score" : 1.0,
              "hits" : [
                {
                  "_index" : "unitate",
                  "_type" : "_doc",
                  "_id" : "3",
                  "_score" : 1.0,
                  "_routing" : "1",
                  "_source" : {
                    "nume" : "Fănel Aglutinel",
                    "varsta" : 13,
                    "sex" : "M",
                    "uneste_campuri" : {
                      "name" : "elev",
                      "parent" : 5
                    }
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

Pentru a căuta subdiviziunile unui părinte, dacă acestea există și cum se numesc, vom modela o interogare cu `has_parent`, precum în exemplul:

```yaml
GET /unitate/_search
{
  "query": {
    "has_parent": {
      "parent_type": "scoala",
      "query": {
        "term": {
          "nume": "119"
        }
      }
    }
  }
}
```

Obiectul returnat este similar cu:

```json
{
  "took" : 5,
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
        "_index" : "unitate",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "nume" : "Clasa I A",
          "uneste_campuri" : {
            "name" : "clasa",
            "parent" : 4
          }
        }
      }
    ]
  }
}
```

Adăugând opțiunea `inner_hits`, ne este oferită toată ierarhia care a fost stabilită.

```yaml
GET /unitate/_search
{
  "query": {
    "has_parent": {
      "parent_type": "scoala",
      "inner_hits": {},
      "query": {
        "term": {
          "nume": "119"
        }
      }
    }
  }
}
```

Vom obține un rezultat similar cu:

```json
{
  "took" : 26,
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
        "_index" : "unitate",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "nume" : "Clasa I A",
          "uneste_campuri" : {
            "name" : "clasa",
            "parent" : 4
          }
        },
        "inner_hits" : {
          "scoala" : {
            "hits" : {
              "total" : {
                "value" : 1,
                "relation" : "eq"
              },
              "max_score" : 1.2222548,
              "hits" : [
                {
                  "_index" : "unitate",
                  "_type" : "_doc",
                  "_id" : "4",
                  "_score" : 1.2222548,
                  "_source" : {
                    "nume" : "Scoala Generală Nr. 119 „Ecaterina Teodoroiu”",
                    "uneste_campuri" : "scoala"
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

Pentru un termen mai generic precum `scoala`, am fi obținut un adevărat arbore al relațiilor. Cu cât termenul este mai specific, cu atît vom obține doar relațiile directe ale celui găsit cu părintele său.

## Resurse

- [Parent ID query | Elasticsearch Reference | Query DSL | Joining queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-parent-id-query.html)
